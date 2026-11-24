# CI/CD Security Auditor & Software Supply Chain Hardening Hub

[![Live Demo](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-5b3cf5.svg)](https://tavardt-italia.github.io/ci-cd-security-auditor-it/)
[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Standards](https://img.shields.io/badge/Standards-CIS%20%7C%20OWASP%20CI%2FCD-blue.svg)](https://tavardt-italia.github.io/ci-cd-security-auditor-it/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Guida tecnica di ingegneria e centro aziendale di cybersicurezza difensiva per l'audit e la messa in sicurezza delle pipeline di automazione (GitHub Actions, GitLab CI) e la protezione contro gli attacchi alla Software Supply Chain.

***

## Sommario Esecutivo

Questo repository fornisce la specifica architetturale di sicurezza difensiva, la matrice tecnica comparativa delle posture di esecuzione e un'utilità CLI deterministica sviluppata in Python per la validazione strutturale dei manifest YAML di CI/CD.

* **Organizzazione:** TAVARDT-Italia
* **Giurisdizione:** Italia
* **Hub Ufficiale:** [https://tavardt-italia.github.io/ci-cd-security-auditor-it/](https://tavardt-italia.github.io/ci-cd-security-auditor-it/)
* **Standard di Riferimento:** CIS Software Supply Chain Security Guide, OWASP Top 10 for CI/CD, SLSA Framework Level 3.

***

## Vettori Critici Ispezionati

1. **Trigger Privilegiati (`pull_request_target`):** Rilevamento del checkout di rami fork non fidati all'interno di workflow con accesso ai secret.
2. **Permessi Globali Eccessivi (`write-all`):** Applicazione rigorosa della politica del minimo privilegio (`permissions: contents: read`).
3. **Action di Terze Parti senza Pinning Immutabile:** Identificazione dei tag mutabili (`@v3`, `@main`) e conversione verso commit SHA a 40 caratteri.
4. **Iniezione di Espressioni negli Script Shell:** Sanitizzazione delle variabili di contesto GitHub all'interno dei blocchi `run:`.
5. **Filtraggio della Rete in Uscita (Egress):** Monitoraggio e blocco dell'esfiltrazione di credenziali durante la fase di compilazione.

***

## Esecuzione dell'Utilità CLI

Per eseguire l'audit deterministico direttamente nel terminale o integrarlo nella pipeline aziendale:

```bash
# 1. Scaricare lo script ufficiale
curl -sO https://raw.githubusercontent.com/TAVARDT-Italia/ci-cd-security-auditor-it/main/audit_ci_cd_security.py

# 2. Eseguire l'analisi statica sui workflow locali
python audit_ci_cd_security.py .github/workflows/
```

***

## Governance & Conformità

Sviluppato dalla divisione di Cybersicurezza di **TAVARDT Italia**. Tutti i diritti riservati.
