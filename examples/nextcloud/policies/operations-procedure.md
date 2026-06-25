# Operations Procedure (A.12 — ISO 27001 Annex A.12)

## Zweck
Standardisierte Betriebsablaeufe fuer das Nextcloud-Setup: Deployment, Backup, Recovery, Update, Change-Management.

---

## 1. Deployment-Prozedur (8 Steps — idempotent)

### Pre-Deployment
- [ ] Host-System geprueft (Debian x86_64, Docker installiert)
- [ ] .env-Datei mit allen ${VAR}-Secrets vorhanden
- [ ] Netzwerk-Konnektivitaet zu Traefik und externen Diensten geprueft
- [ ] Backup vor Deployment (falls Upgrade)

### Deploy-Steps

| Step | Aktion | Befehl/Tool | Idempotent? |
|------|--------|-------------|-------------|
| 1 | Docker-Netz(e) erstellen | `docker network create [netz_name]` | Ja |
| 2 | Volumes erstellen (5x) | `docker volume create <name>` | Ja |
| 3 | Postgres starten (Healthcheck: pg_isready) | `docker compose up -d postgres` | Ja |
| 4 | Redis starten (ACL-basiert) | `docker compose up -d redis` | Ja |
| 5 | Nextcloud starten (Port 80 intern) | `docker compose up -d nextcloud` | Ja |
| 6 | Traefik-Routen aktivieren | Traefik labels / dynamic config | Ja |
| 7 | Keycloak OIDC-Client registrieren | Keycloak Admin Console / API | [ ] |
| 8 | Smoketest (Login, Upload, Health) | healthcheck-monitor.sh | Ja |

### Post-Deployment
- [ ] healthcheck-monitor.sh erfolgreich
- [ ] TLS-Zertifikat valide (cloud.example.com, auth.example.com)
- [ ] OIDC-Login via Keycloak getestet
- [ ] Backup-Job konfiguriert

---

## 2. Backup-Plan

### Backup-Script: backup.sh (pg_dump + rsync)

| Komponente | Methode | Ziel | Retention |
|------------|---------|------|-----------|
| Postgres-DB | pg_dump (custom format) | [ ] | 7 Tage |
| Volumes (html, config, apps) | rsync | [ ] | 7 Tage |
| Redis-Daten | Nicht gesichert (Cache) | N/A | N/A |
| Traefik-Konfiguration | [ ] | [ ] | [ ] |

### Backup-Zeitplan

| Backup-Typ | Frequenz | Zeitfenster | Verantwortlich |
|------------|----------|-------------|----------------|
| Vollbackup | [ ] (Empfehlung: taeglich) | [ ] | Backup-Operator / Cron |
| Log-Backup | json-file Rotation | max-size 10m, max-file 3 | Docker |
| Pre-Upgrade-Backup | Vor jedem Update | Manuell | System-Administrator |

### Restore-Test

| Test-Frequenz | Verantwortlich | Dokumentation |
|---------------|----------------|---------------|
| [ ] (Empfehlung: monatlich) | [ ] | [ ] |

---

## 3. Rollback-Prozedur (5 Steps)

| Step | Aktion | Befehl/Tool |
|------|--------|-------------|
| 1 | Container stoppen | `docker compose stop nextcloud` |
| 2 | Postgres-Restore | `pg_restore <backup_file>` |
| 3 | Volume-Restore | `rsync <backup_dir>/ <volume_mount>/` |
| 4 | Vorheriges Image-Tag setzen | `compose.yaml` Tag aendern |
| 5 | Container neustarten + Smoketest | `docker compose up -d` + healthcheck-monitor.sh |

---

## 4. Update-Strategie

| Komponente | Update-Typ | Verfahren | Ruellback-Plan? |
|------------|-----------|-----------|-----------------|
| Nextcloud | Major (z.B. 31→32) | Dokumentiert, Pre-Upgrade-Backup | Ja (pg_restore) |
| Postgres | Major (z.B. 16→17) | Dokumentiert, pg_dump vorher | Ja |
| Redis | Patch | Image-Tag aktualisieren | Ja |
| Traefik | Patch | Image-Tag aktualisieren | Ja |
| System-Packages (Host) | [ ] | [ ] | [ ] |

### Update-Ablauf (Nextcloud Major)

| Step | Aktion | Verantwortlich |
|------|--------|----------------|
| 1 | Ankuendigung & Change-Request | [ ] |
| 2 | Backup (pg_dump + rsync) | Backup-Operator |
| 3 | Wartungsmodus aktivieren | System-Administrator |
| 4 | Image-Tag aktualisieren | System-Administrator |
| 5 | Container neustarten + Upgrade ausfuehren | System-Administrator |
| 6 | Smoketest & Funktionspruefung | [ ] |
| 7 | Wartungsmodus deaktivieren | System-Administrator |
| 8 | Dokumentation des Updates | [ ] |

---

## 5. Change-Management

### Change-Klassifikation

| Typ | Beispiele | Genehmigung erforderlich? |
|-----|----------|--------------------------|
| Standard-Change | Patch-Update, Backup-Restore-Test | [ ] |
| Normal-Change | Minor-Config-Aenderung, User-Provisioning | [ ] |
| Emergency-Change | Sicherheits-Patch, Incident-Response | [ ] |
| Major-Change | Nextcloud Major-Upgrade, Postgres Major | [ ] |

### Change-Request-Template

| Feld | Inhalt |
|------|--------|
| Change-ID | [ ] |
| Beschreibung | [ ] |
| Begruendung | [ ] |
| Betroffene Assets | [ ] |
| Risikobewertung | [ ] |
| Rollback-Plan | [ ] |
| Genehmiger | [ ] |
| Geplantes Fenster | [ ] |
| Durchfuehrender | [ ] |
| Testergebnis | [ ] |
| Status | [ ] (Requested / Approved / In Progress / Completed / Failed) |

---

## 6. Monitoring & Alerting

| Komponente | Check | Tool | Alerting |
|------------|-------|------|----------|
| nextcloud-app | HTTP 200 + Login | healthcheck-monitor.sh | Webhook / Mail |
| nextcloud-postgres | pg_isready | healthcheck-monitor.sh | Webhook / Mail |
| nextcloud-redis | PING | healthcheck-monitor.sh | Webhook / Mail |
| Traefik TLS | Zertifikat-Ablauf | [ ] | [ ] |
| Host-Disk | Speicherplatz | [ ] | [ ] |
| Host-CPU/RAM | Auslastung | [ ] | [ ] |

### Alert-Eskalation

| Level | Bedingung | Aktion | Reaktionszeit |
|-------|-----------|-------|---------------|
| Critical | Service down | Webhook + Mail | [ ] |
| Warning | Zertifikat < 7 Tage | Mail | [ ] |
| Info | Update verfuegbar | [ ] | [ ] |

---

## 7. Logging-Konfiguration

| Container | Log-Driver | max-size | max-file | Zugriff |
|-----------|-----------|----------|----------|---------|
| nextcloud-app | json-file | 10m | 3 | `docker logs` |
| nextcloud-postgres | json-file | 10m | 3 | `docker logs` |
| nextcloud-redis | json-file | 10m | 3 | `docker logs` |
| Traefik | [ ] | [ ] | [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
