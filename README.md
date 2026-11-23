# Auditor di Sicurezza CI/CD & Rilevatore di Fuga di Secret | TAVARDT Italia

[![Licence](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![CI/CD Security](https://img.shields.io/badge/CI%2FCD-Security%20Hardened-00d26a.svg)](README.md)
[![Language](https://img.shields.io/badge/Language-Italiano-green.svg)](README.md)
[![Engineering](https://img.shields.io/badge/TAVARDT-Cybersecurity-5b3cf5.svg)](https://ag.tavardt.com/it/sicurezza-informatica/)

Auditor statico difensivo e linter di sicurezza per pipeline di integrazione e rilascio continuo (GitHub Actions). Rileva configurazioni ad alto rischio di supply chain, trigger vulnerabili a pull request esterne, permessi eccessivi su GITHUB_TOKEN e fuga di secret aziendali.

## Applicazione Web Interattiva

Accedi allo strumento interattivo per verificare i tuoi manifesti YAML in tempo reale direttamente dal browser:

- [https://tavardt-italia.github.io/ci-cd-security-auditor-it/](https://tavardt-italia.github.io/ci-cd-security-auditor-it/)

---

## 1. Panoramica dell'Auditor di Sicurezza CI/CD

Le pipeline di automazione e integrazione continua sono diventate il bersaglio principale degli attacchi di supply chain. Gli aggressori sfruttano credenziali sviluppatore compromesse e trigger mal configurati per eseguire codice arbitrario sui runner di build ed esfiltrare variabili protette, chiavi cloud (AWS, GCP, Azure) e secret di rilascio.

L'Auditor di Sicurezza CI/CD TAVARDT analizza i manifesti YAML in modalità passiva e deterministica, verificando ogni direttiva rispetto ai controlli dell'OWASP Top 10 per la sicurezza CI/CD e al framework SLSA.

---

## 2. Vettori e Vulnerabilità Verificati

| Vettore di Rischio | Scenario nella Pipeline CI/CD | Impatto Tecnico |
| :--- | :--- | :--- |
| **pull_request_target Non Sicuro** | La pipeline viene eseguita nel contesto del branch principale ed effettua il checkout del codice PR (`ref: ${{ github.event.pull_request.head.sha }}`). | Esecuzione remota di codice (RCE) immediata con accesso completo a tutti i secret di produzione del repository. |
| **Permessi write-all Eccessivi** | La pipeline specifica `permissions: write-all` oppure omette il blocco restrittivo a livello radice. | Il token eredita l'autorizzazione a modificare tag, sovrascrivere release e distribuire artefatti compromessi. |
| **Iniezione di Comandi Shell in run:** | Variabili di contesto generate da utenti esterni (es: `${{ github.event.issue.title }}`) sono concatenate direttamente nei blocchi shell. | Consente il breakout di sintassi e l'esecuzione di comandi malevoli direttamente nel sistema operativo del runner. |
| **Azioni Collegate a Tag Mutabili** | Le istruzioni `uses:` fanno riferimento a tag mutabili come `@v3` o `@main` anziché a commit SHA immutabili. | Espone la pipeline a rischi di supply chain poisoning in caso di compromissione dell'account del manutentore. |
| **Esfiltrazione e Credenziali Hardcoded** | Presenza di chiavi private in chiaro o istruzioni `curl` che trasmettono secret verso server esterni. | Compromissione immediata dell'infrastruttura cloud e violazione delle direttive di conformità e governance. |

---

## 3. Guida all'Installazione ed Esecuzione Locale (CLI)

Lo scanner opera in ambiente terminale senza dipendenze esterne, sfruttando esclusivamente i moduli standard di Python 3.

### Clonare il Repository

```bash
git clone https://github.com/TAVARDT-Italia/ci-cd-security-auditor-it.git
cd ci-cd-security-auditor-it
```

### Verificare un Singolo File YAML

```bash
python scanner.py .github/workflows/deploy.yml
```

### Verificare l'Intera Cartella con Output JSON

```bash
python scanner.py .github/workflows/ --json
```

---

## 4. Blindatura Professionale e Supporto Tecnico

Se l'audit ha evidenziato vulnerabilità nelle pipeline, secret esposti o la necessità di una governance Zero Trust con i provider cloud:

- **Portale Istituzionale:** [TAVARDT Italia](https://ag.tavardt.com/it/sicurezza-informatica/)
- **Centro Strumenti:** [Strumenti Difensivi TAVARDT](https://tools.tavardt.com/it/)
- **Consulenza con l'Ingegneria:** [Supporto Tecnico e Risposta agli Incidenti](https://ag.tavardt.com/it/contato/)

---

## Licenza

Distribuito sotto licenza open source MIT. Consulta il file [`LICENSE`](LICENSE) per ulteriori dettagli.  
Copyright (c) 2026 TAVARDT Italia.
