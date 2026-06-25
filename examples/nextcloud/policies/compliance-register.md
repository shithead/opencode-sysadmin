# Compliance Register (A.18 — ISO 27001 Annex A.18)

## Zweck
Nachweis der Einhaltung gesetzlicher, vertraglicher und regulatorischer Anforderungen fuer das Nextcloud-Setup.

---

## 1. Anwendbare Vorschriften & Standards

| Vorschrift | Relevanz | Gueltig seit | Verantwortlich |
|------------|----------|-------------|----------------|
| DSGVO (GDPR) | Personenbezogene Daten in Nextcloud (EU) | 25.05.2018 | [ ] |
| BDSG (neu) | Nationales Datenschutzrecht (DE) | [ ] | [ ] |
| IT-Sicherheitsgesetz 2.0 | KRITIS (falls anwendbar) | [ ] | [ ] |
| ISO 27001 | Informationssicherheits-Managementsystem | [ ] | [ ] |
| TTDSG | Telekommunikation-Telemedien-Datenschutz (DE) | [ ] | [ ] |
| [ ] | [ ] | [ ] | [ ] |

---

## 2. DSGVO-Compliance

### DPIA (Datenschutz-Folgenabschaetzung)

| Aspekt | Status |
|--------|--------|
| DPIA erforderlich? | [ ] |
| DPIA durchgefuehrt am | [ ] |
| DPIA-Dokument | [ ] |
| Risiken identifiziert | [ ] |
| Mitigation-Massnahmen | [ ] |
| Freigabe DSB | [ ] |

### Auftragsverarbeitung (AVV)

| Auftragsverarbeiter | Dienst | AVV vorhanden? |
|-------------------|--------|----------------|
| Hosting-Provider | Server-Hosting | [ ] |
| Backup-Storage-Provider | Backup-Ziel | [ ] |
| [ ] | [ ] | [ ] |

### Betroffenenrechte (DSGVO Art. 15-21)

| Recht | Nextcloud-Funktion | Status |
|-------|-------------------|--------|
| Auskunft (Art. 15) | [ ] | [ ] |
| Berichtigung (Art. 16) | [ ] | [ ] |
| Loeschung (Art. 17) | [ ] | [ ] |
| Einschraenkung (Art. 18) | [ ] | [ ] |
| Datenuebertragbarkeit (Art. 20) | [ ] | [ ] |
| Widerspruch (Art. 21) | [ ] | [ ] |

### Loeschkonzept

| Datenkategorie | Loeschfrist | Automatisiert? | Verantwortlich |
|---------------|-------------|---------------|----------------|
| User-Daten (vom Nutzer geloescht) | [ ] | [ ] | [ ] |
| Inaktive User-Accounts | [ ] | [ ] | [ ] |
| Gelöschte Dateien (Trashbin) | [ ] (Nextcloud default: 30d) | Ja (Nextcloud intern) | Nextcloud |
| Versionen | [ ] (Nextcloud default: [ ]) | Ja (Nextcloud intern) | Nextcloud |
| Log-Dateien | max-file 3 (json-file Rotation) | Ja (Docker) | Docker |
| Backups | 7 Tage Retention | Ja (backup.sh Rotation) | Backup-Operator |
| Keycloak-User | [ ] | [ ] | [ ] |

### Datenkategorien in Nextcloud

| Kategorie | Beispiele | Besonders schutzbeduerftig? |
|-----------|----------|---------------------------|
| Stammdaten | Name, E-Mail, Username | [ ] |
| Inhaltsdaten | Hochgeladene Dateien, Kalender, Kontakte | [ ] |
| Meta-Daten | Zugriffslogs, Shares, Aktivitaeten | [ ] |
| Auth-Daten | OIDC-Tokens, Sessions | [ ] |
| [ ] | [ ] | [ ] |

---

## 3. Lizenz-Compliance

### Open-Source-Lizenzen der Setup-Komponenten

| Komponente | Lizenz | Lizenztext/Link | Compliance-Anforderung |
|------------|--------|-----------------|----------------------|
| Nextcloud | AGPL-3.0 | https://github.com/nextcloud/server/blob/master/COPYING | Quellcode auf Anfrage bereitstellen; Aenderungen offenlegen |
| PostgreSQL | PostgreSQL License (BSD-aehnlich) | https://www.postgresql.org/about/licence/ | Copyright-Hinweis beibehalten |
| Redis | BSD-3-Clause | https://redis.io/docs/about/license/ | Copyright-Hinweis beibehalten |
| Traefik | MIT | https://github.com/traefik/traefik/blob/master/LICENSE | Copyright-Hinweis beibehalten |
| Keycloak | Apache-2.0 | https://github.com/keycloak/keycloak/blob/main/LICENSE.txt | Copyright-Hinweis + NOTICE beibehalten |
| Docker Engine | Apache-2.0 | [ ] | [ ] |
| Debian (Host) | GPL + diverse | [ ] | [ ] |

### Lizenz-Compliance-Verpflichtungen

| Verpflichtung | Erfuellt? | Nachweis |
|---------------|-----------|----------|
| AGPL-3.0 Quellcode-Bereitstellung (Nextcloud) | [ ] Keine Eigenentwicklung → nicht anwendbar | [ ] |
| Copyright-Hinweise in Dokumentation | [ ] | [ ] |
| Lizenztexte im Release-Paket | [ ] | [ ] |
| SBOM (Software Bill of Materials) | [ ] | [ ] |

### Nextcloud-Apps (zusätzlich installierte)

| App | Lizenz | Geprueft? |
|-----|--------|-----------|
| [ ] | [ ] | [ ] |
| [ ] | [ ] | [ ] |

---

## 4. Vertragliche Anforderungen

| Vertragspartner | Vertragsgegenstand | Compliance-Anforderung | Status |
|-----------------|-------------------|----------------------|--------|
| [ ] (Hosting) | Server-Bereitstellung | [ ] | [ ] |
| [ ] (Domain) | Domain-Registrierung | [ ] | [ ] |
| [ ] (Backup-Storage) | Backup-Speicher | [ ] | [ ] |

---

## 5. Audit-Requirements

### Interne Audits

| Audit | Frequenz | Letztes Audit | Naechstes Audit | Verantwortlich |
|-------|----------|---------------|-----------------|----------------|
| ISO 27001 ISMS | [ ] | [ ] | [ ] | [ ] |
| DSGVO-Compliance | [ ] | [ ] | [ ] | [ ] |
| Lizenz-Compliance | [ ] | [ ] | [ ] | [ ] |
| Sicherheits-Review | [ ] | [ ] | [ ] | [ ] |

### Externe Anforderungen

| Anforderung | Nachzuweisen gegenueber | Frequenz |
|-------------|------------------------|----------|
| [ ] | [ ] | [ ] |

---

## 6. Dokumentationspflichten

| Dokument | Speicherort | Version | Letzte Aktualisierung |
|----------|-------------|---------|----------------------|
| Asset Inventory (A.8) | examples/nextcloud/policies/asset-inventory.md | [ ] | [ ] |
| Access Control Policy (A.9) | examples/nextcloud/policies/access-control-policy.md | [ ] | [ ] |
| Cryptographic Policy (A.10) | examples/nextcloud/policies/cryptographic-policy.md | [ ] | [ ] |
| Operations Procedure (A.12) | examples/nextcloud/policies/operations-procedure.md | [ ] | [ ] |
| Secure Development Policy (A.14) | examples/nextcloud/policies/secure-development-policy.md | [ ] | [ ] |
| Incident Response Plan (A.16) | examples/nextcloud/policies/incident-response-plan.md | [ ] | [ ] |
| Business Continuity Plan (A.17) | examples/nextcloud/policies/business-continuity-plan.md | [ ] | [ ] |
| Compliance Register (A.18) | examples/nextcloud/policies/compliance-register.md | [ ] | [ ] |
| Verarbeitungsverzeichnis (DSGVO Art. 30) | [ ] | [ ] | [ ] |
| Technische & Organisatorische Massnahmen (TOM) | [ ] | [ ] | [ ] |

---

## 7. Non-Compliance & Abweichungen

| ID | Beschreibung | Risiko | Mitigation | Deadline | Status |
|----|-------------|-------|------------|----------|--------|
| [ ] | [ ] | [ ] | [ ] | [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
