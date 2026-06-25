# Incident Response Plan (A.16 — ISO 27001 Annex A.16)

## Zweck
Definierter Ablauf zur Erkennung, Reaktion und Wiederherstellung bei Sicherheitsvorfaellen im Nextcloud-Setup.

---

## 1. Incident-Klassifikation

| Severity | Beispiele | Reaktionszeit | Eskalation |
|----------|----------|---------------|------------|
| Critical (P1) | Kompletter Service-Ausfall, Datenverlust, Ransomware, aktiver Angriff | [ ] (Empfehlung: < 1h) | [ ] |
| High (P2) | Teilausfall (DB/Redis), OIDC nicht erreichbar, Zertifikat abgelaufen | [ ] | [ ] |
| Medium (P3) | Performance-Degradation, Verzoegerungen, einzelner Container down | [ ] | [ ] |
| Low (P4) | Log-Anomalien, fehlgeschlagene Login-Versuche, Warnungen | [ ] | [ ] |

---

## 2. Detection (Erkennung)

### Monitoring-Quellen

| Quelle | Typ | Intervall | Alerting |
|--------|-----|-----------|----------|
| healthcheck-monitor.sh | Active Check (alle Container) | [ ] | Webhook + Mail |
| Docker-Healthchecks | pg_isready (Postgres) | Container-intern | `docker ps` status |
| Traefik-Logs | Access/Error | — | [ ] |
| Nextcloud-Logs | nextcloud.log | — | json-file rotation |
| Postgres-Logs | PostgreSQL Log | — | json-file rotation |
| Host-System | Syslog, auth.log | — | [ ] |

### Detection-Regeln

| Rule ID | Bedingung | Severity | Alert-Kanal |
|---------|-----------|----------|-------------|
| DET-001 | healthcheck-monitor meldet DOWN | Critical | Webhook + Mail |
| DET-002 | TLS-Zertifikat < 7 Tage gueltig | High | Mail |
| DET-003 | Disk > [ ]% belegt | High | [ ] |
| DET-004 | Mehr als [ ] fehlgeschlagene Logins/5min | Medium | [ ] |
| DET-005 | Unbekannte IP auf SSH | Medium | [ ] |
| DET-006 | [ ] | [ ] | [ ] |

---

## 3. Response-Team

| Rolle | Name | Kontakt | Verfuegbarkeit |
|-------|------|---------|----------------|
| Incident Manager | [ ] | [ ] | [ ] |
| System-Administrator | [ ] | [ ] | [ ] |
| Security-Officer | [ ] | [ ] | [ ] |
| Kommunikation / PR | [ ] | [ ] | [ ] |
| Backup-Operator | [ ] | [ ] | [ ] |

---

## 4. Incident-Response-Prozess

### Phase 1: Identifikation & Triage

| Step | Aktion | Verantwortlich | Tool/Quelle |
|------|--------|----------------|-------------|
| 1 | Alert empfangen & validieren | Incident Manager | Webhook / Mail |
| 2 | Severity klassifizieren | Incident Manager | Klassifikation (s.o.) |
| 3 | Incident-Ticket eroeffnen | Incident Manager | [ ] |
| 4 | Response-Team benachrichtigen | Incident Manager | [ ] |

### Phase 2: Containment (Eindaemmung)

| Step | Aktion | Verantwortlich |
|------|--------|----------------|
| 1 | [ ] (z.B. kompromittierten Container stoppen) | System-Administrator |
| 2 | Netzwerk-Zugriff einschraenken (falls noetig) | System-Administrator |
| 3 | Beweise sichern (Logs, Volumes, DB-Snapshot) | System-Administrator |
| 4 | Betroffene Secrets rotieren (falls kompromittiert) | Security-Officer |

### Phase 3: Analyse & Investigation

| Step | Aktion | Verantwortlich |
|------|--------|----------------|
| 1 | Logs analysieren (Container, Host, Traefik) | Security-Officer |
| 2 | Root-Cause ermitteln | Security-Officer |
| 3 | Betroffene Assets identifizieren | System-Administrator |
| 4 | Auswirkungen dokumentieren | Security-Officer |

### Phase 4: Recovery (Wiederherstellung)

| Step | Aktion | Verantwortlich | Tool |
|------|--------|----------------|------|
| 1 | Entscheidung: Rollback oder Fix | Incident Manager | — |
| 2a | **Rollback**: pg_restore + rsync (5-Step-Rollback) | System-Administrator | pg_restore, rsync |
| 2b | **Fix**: Korrigierte Konfiguration deployen | System-Administrator | docker compose |
| 3 | Smoketest (healthcheck-monitor.sh) | System-Administrator | healthcheck-monitor.sh |
| 4 | Service-Staus verifizieren | Incident Manager | — |
| 5 | Monitoring wieder aktivieren | System-Administrator | — |

### Phase 5: Post-Incident

| Step | Aktion | Verantwortlich |
|------|--------|----------------|
| 1 | Post-Mortem-Report erstellen | Incident Manager |
| 2 | Lessons-Learned dokumentieren | Alle |
| 3 | Preventive-Massnahmen definieren | Security-Officer |
| 4 | Policy/Playbook aktualisieren | Security-Officer |
| 5 | Incident-Ticket schliessen | Incident Manager |

---

## 5. Rollback als Recovery (5-Step-Rollback)

Gemaess Operations Procedure (A.12):

| Step | Aktion | Tool |
|------|--------|------|
| 1 | Container stoppen | `docker compose stop nextcloud` |
| 2 | Postgres-Restore | `pg_restore <backup_file>` |
| 3 | Volume-Restore | `rsync <backup_dir>/ <volume_mount>/` |
| 4 | Vorheriges Image-Tag setzen | compose.yaml editieren |
| 5 | Container neustarten + Smoketest | `docker compose up -d` + healthcheck-monitor.sh |

---

## 6. Kommunikationsplan

| Stakeholder | Wann informieren? | Kanal | Vorlage? |
|-------------|-------------------|-------|----------|
| Interne IT-Leitung | P1, P2 Incidents | [ ] | [ ] |
| Datenschutzbeauftragter | Bei Personen-Daten betroffen | [ ] | [ ] |
| Enduser | Bei Service-Ausfall > [ ] h | [ ] | [ ] |
| Aufsichtsbehoerde | Bei meldepflichtigem DSGVO-Vorfall (72h) | [ ] | [ ] |
| Oeffentlichkeit/PR | Nach Freigabe | [ ] | [ ] |

---

## 7. Incident-Tracking

| Incident-ID | Datum | Severity | Beschreibung | Status | Post-Mortem |
|-------------|-------|----------|-------------|--------|-------------|
| INC-001 | [ ] | [ ] | [ ] | [ ] | [ ] |

---

## 8. Beweissicherung (Forensics)

| Artefakt | Methode | Retention nach Incident |
|----------|---------|------------------------|
| Container-Logs | `docker logs` + json-file | [ ] |
| Host-Logs | /var/log/* | [ ] |
| DB-Snapshot | pg_dump zum Incident-Zeitpunkt | [ ] |
| Volume-Snapshot | rsync des aktuellen Zustands | [ ] |
| Netzwerk-Capture | [ ] (tcpdump) | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*  
*Getestet am: [ ]*
