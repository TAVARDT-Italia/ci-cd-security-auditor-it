<p align="center">
  <a href="https://ag.tavardt.com/it/">
    <img src="https://ag.tavardt.com/img/tavardt-banner.png" alt="TAVARDT" width="1024" />
  </a>
</p>

# Auditor di Sicurezza CI/CD e Hardening della Catena di Fornitura del Software

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-italia.github.io/ci-cd-security-auditor-it/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-italia.github.io/ci-cd-security-auditor-it/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guida aziendale di ingegneria difensiva, standard di immunizzazione delle pipeline di automazione e utility deterministica da riga di comando per la protezione della catena di fornitura del software in GitHub Actions e piattaforme moderne di CI/CD.

***

## Quadro Esecutivo: Lo Scenario di Minaccia nelle Pipeline CI/CD

Le pipeline di Integrazione Continua e Distribuzione Continua (CI/CD) rappresentano l'infrastruttura centrale dello sviluppo software contemporaneo. Avendo accesso a chiavi di rilascio, privilegi di scrittura sul codice sorgente e canali di rete verso ambienti produttivi, le automazioni sono diventate il bersaglio principale degli attacchi mirati alla catena di fornitura.

Quando i team di ingegneria accelerano i tempi di rilascio trascurando i controlli perimetrali, i workflow ereditano vulnerabilità strutturali di elevata gravità:
* Trigger di compilazione eseguono codice non verificato proveniente da fork all'interno di ambienti privilegiati.
* Dipendenze e action di terze parti impiegano tag mutabili suscettibili di compromissione dell'account del creatore.
* I token di esecuzione ereditano privilegi globali di scrittura estesi all'intero repository.
* Gli esecutori (runners) operano senza restrizioni sul traffico di rete in uscita, consentendo l'esfiltrazione di segreti.

Questo repository definisce uno standard corporativo di difesa in profondità. Unisce modelli di configurazione pronti per la produzione a un analizzatore statico deterministico in Python (`audit_ci_cd_security.py`) concepito per operare senza trasmissione di telemetria.

***

## Superfici Critiche di Attacco nelle Pipeline di Automazione

### 1. La Criticità Strutturale del Trigger `pull_request_target`
I flussi avviati da `pull_request_target` vengono eseguiti nel contesto privilegiato del branch principale e accedono direttamente ai segreti protetti:
* **Meccanismo della Vulnerabilità:** Qualsiasi workflow che associ questo trigger all'estrazione del commit della pull request (`github.event.pull_request.head.sha`) introduce codice fornito da terzi nel runner privilegiato.
* **Impatto Operativo:** Soggetti esterni possono alterare gli script di compilazione, eseguire comandi arbitrari ed esfiltrare credenziali aziendali senza approvazione preventiva.
* **Direttiva Difensiva:** Non eseguire mai il checkout di codice non verificato sotto `pull_request_target`. Isolare i test preliminari in eventi ordinari di tipo `pull_request`.

### 2. Action Esterne prive di Blocco con Hash Commit SHA
L'invocazione di action di terze parti mediante tag di versione (quali `@v3` o `@main`) espone a rischi diretti di manomissione a monte:
* **Meccanismo della Vulnerabilità:** I tag in Git sono puntatori mutabili che possono essere riposizionati qualora l'account dello sviluppatore originario venga violato.
* **Impatto Operativo:** Un commit ostile introdotto in una action diffusa contamina simultaneamente e automaticamente migliaia di pipeline aziendali.
* **Direttiva Difensiva:** Fissare tutte le action esterne esclusivamente tramite l'identificatore SHA a 40 caratteri, corredato da un commento che indichi la versione corrispondente.

### 3. Token con Privilegi Eccessivi (`permissions: write-all`)
Le pipeline operano frequentemente ereditando le autorizzazioni predefinite impostate a livello di organizzazione:
* **Meccanismo della Vulnerabilità:** L'assenza di un blocco esplicito `permissions:` attribuisce al token diritti completi di scrittura su repository, pacchetti e rilasci.
* **Impatto Operativo:** Una dipendenza compromessa durante la fase di compilazione può manomettere rami protetti o pubblicare artefatti corrotti.
* **Direttiva Difensiva:** Dichiarare imperativamente `permissions: contents: read` all'inizio del file, limitando i privilegi di scrittura a singoli job isolati che ne necessitino formalmente.

### 4. Assenza di Restrizioni sul Traffico in Uscita degli Esecutori
I runner di compilazione dispongono solitamente di connettività internet in uscita senza limiti:
* **Meccanismo della Vulnerabilità:** Script dannosi eseguiti durante il build possono aprire connessioni remote e trasmettere variabili di ambiente a server esterni.
* **Direttiva Difensiva:** Applicare filtri sul traffico in uscita (egress filtering), consentendo la comunicazione unicamente con registri e destinazioni autorizzate.

***

## Modelli di Hardening per Ambienti di Produzione

Il repository fornisce modelli tecnici collaudati per l'adozione immediata in ambienti aziendali:

### 1. Workflow di Sicurezza per GitHub Actions (`hardened-workflow.yml`)
Applica il principio del minimo privilegio, il blocco tramite commit SHA e l'isolamento dei segreti:

```yaml
name: Enterprise Hardened Pipeline

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  audit-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Hardened Checkout
        uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
        with:
          persist-credentials: false

      - name: Configurazione Sicura di Python
        uses: actions/setup-python@0a5c61591373683505ea898e09a3ea4f39ef2b9c # v5.0.0
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Esecuzione dell Analisi Statica
        run: |
          python audit_ci_cd_security.py .github/workflows/
```

### 2. Script di Verifica del Blocco delle Action (`action-pinning.sh`)
Scansiona i manifesti di configurazione e verifica l'adozione di hash SHA inalterabili:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Esamina i workflow e valida l utilizzo di hash SHA a 40 caratteri
find .github/workflows -type f \( -name "*.yml" -o -name "*.yaml" \) | while read -r workflow; do
  echo "[*] Audit del workflow: ${workflow}"
  grep -E "uses: [a-zA-Z0-9_\-\.\/]+@[a-f0-9]{40}" "${workflow}" || echo "[!] Action non bloccate rilevate"
done
```

### 3. Filtraggio delle Connessioni di Rete in Uscita (`step-security-harden.yml`)
Blocca connessioni esterne non previste originate dall'ambiente di esecuzione:

```yaml
- name: Proteggere Connessioni in Uscita
  uses: step-security/harden-runner@63c24ba6bd7ba022e95695ff85de572c04a18142 # v2.7.0
  with:
    egress-policy: block
    allowed-endpoints: >
      github.com:443
      api.github.com:443
      pypi.org:443
      files.pythonhosted.org:443
```

***

## Utility Deterministica di Audit CLI

Gli ambienti browser applicano politiche di isolamento che impediscono l'accesso al file system locale e l'ispezione diretta dei file di configurazione. Per consentire valutazioni tecniche riproducibili e prive di telemetria, TAVARDT ha realizzato un analizzatore statico in Python per l'impiego nei terminali degli sviluppatori e nelle pipeline di CI/CD.

### Funzionamento dell'Analizzatore Statico

L'utility (`audit_ci_cd_security.py`) esegue quattro verifiche deterministiche:
* **Audit dei Trigger:** Esamina le dichiarazioni di evento per individuare utilizzi a rischio di `pull_request_target`.
* **Verifica del Minimo Privilegio:** Rileva impostazioni `write-all` o l'assenza di clausole restrittive per i token.
* **Controllo del Blocco delle Dipendenze:** Valida le istruzioni `uses:` tramite espressioni regolari per accertare l'uso di hash SHA a 40 caratteri.
* **Scansione Segreti in Chiaro:** Analizza le istruzioni da terminale e le variabili per identificare credenziali esposte.

### Guida all'Uso

Scaricare ed eseguire l'utility direttamente sui manifesti del repository:

```bash
# 1. Scaricare lo script ufficiale
curl -sO https://raw.githubusercontent.com/TAVARDT-Italia/ci-cd-security-auditor-it/main/audit_ci_cd_security.py

# 2. Eseguire l analisi statica sui workflow locali
python audit_ci_cd_security.py .github/workflows/
```

***

## Domande Frequenti di Architettura (FAQ Tecnico)

### 1. Perché il trigger pull_request_target è considerato una delle vulnerabilità più gravi in GitHub Actions?
L'evento `pull_request_target` opera nel contesto con privilegi elevati del branch principale e dispone di accesso diretto ai segreti di produzione. Qualora il flusso esegua il checkout del codice proposto da un fork esterno, utenti malintenzionati possono eseguire comandi arbitrari nel runner e sottrarre credenziali aziendali.

### 2. Come previene la manomissione della catena di fornitura il blocco tramite commit SHA a 40 caratteri?
I tag come `@v3` o `@main` sono puntatori Git modificabili che possono essere riposizionati se l'account del mantentore viene violato. Un hash di commit SHA a 40 caratteri è crittograficamente immutabile, garantendo l'esecuzione esclusiva del codice formalmente revisionato.

### 3. Quale rischio operativo comporta l'omissione della dichiarazione dei permessi nei file di workflow?
In assenza di una dichiarazione esplicita tramite `permissions:`, il token del runner eredita i privilegi predefiniti dell'organizzazione, che frequentemente includono ampi poteri di scrittura. Un componente malevolo può sfruttare tale token per manomettere rami protetti o pubblicare artefatti compromessi.

### 4. Come impedisce la fuga di credenziali il filtraggio del traffico in uscita (egress filtering)?
Il filtraggio in uscita limita le connessioni di rete del runner a una lista restrittiva di destinazioni ufficiali verificate. Se una dipendenza compromessa tenta di avviare una sessione remota o inviare variabili di ambiente verso un server esterno, la connessione viene terminata istantaneamente a livello di rete.

### 5. L'analizzatore statico in Python richiede connettività internet o privilegi di amministratore?
No. L'utility funziona in modo del tutto passivo e offline, leggendo i file YAML locali direttamente dal disco senza trasmettere telemetria e senza richiedere privilegi di superutente sul sistema operativo.

***

## Normative di Riferimento e Standard Aziendali

Architettura difensiva allineata alle linee guida OWASP Top 10 CI/CD Security Risks, al framework SLSA e ai controlli di integrità del software raccomandati dall'Agenzia per la Cybersicurezza Nazionale (ACN).

***

## Link Utili

* **Portale Aziendale:** https://ag.tavardt.com/it/
* **Suite di Diagnostica:** https://tools.tavardt.com/it/
* **Contatto Tecnico:** https://ag.tavardt.com/it/contatto/

***

© 2026 TAVARDT. Tutti i diritti riservati.
