# Business Continuity Plan (A.17 — ISO 27001 Annex A.17)

## Zweck
Sicherstellung der Geschaeftskontinuitaet des Nextcloud-Services im Falle von Stoerungen, Ausfaellen oder Desastern.

**Hinweis**: Single-Host Deployment — keine native Hochverfuegbarkeit. Wiederherstellung ueber Backup/Restore.

---

## 1. Business Impact Analysis (BIA)

### Kritische Geschaeftsprozesse

| Prozess | Abhaengigkeit von Nextcloud | Kritikalitaet | Max. tolerable Ausfallzeit |
|---------|----------------------------|---------------|---------------------------|
| [ ] | [ ] | [ ] | [ ] |
| [ ] | [ ] | [ ] | [ ] |
| [ ] | [ ] | [ ] | [ ] |

### Service-Abhaengigkeiten

| Service | Intern/Extern | Kritisch? | Auswirkung bei Ausfall |
|---------|--------------|-----------|----------------------|
| Docker Engine | Intern (Host) | Ja | Alle Container down |
| nextcloud-app | Intern (Container) | Ja | Nextcloud nicht erreichbar |
| nextcloud-postgres | Intern (Container) | Ja | Nextcloud ohne DB-Funktion |
| nextcloud-redis | Intern (Container) | [ ] | Performance-Degradation |
| Traefik | Intern (Container) | Ja | Kein Routing/TLS |
| Keycloak (auth.example.com) | [ ] | [ ] | OIDC-Login nicht moeglich |
| ACME (Let's Encrypt) | Extern | [ ] | Nach [ ] Tagen kein TLS |
| Internet (ISP) | Extern | Ja | Kein Zugriff von aussen |
| Strom/Hardware | Physisch | Ja | Host down => Alles down |

---

## 2. RTO / RPO

### Recovery Objectives

| Service | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) | Begruendung |
|---------|------------------------------|-------------------------------|-------------|
| Nextcloud (Gesamt) | [ ] (Empfehlung: < 4h) | [ ] (Empfehlung: < 24h) | Single-Host Restore |
| Postgres-DB | [ ] | [ ] | pg_restore aus Backup |
| Nextcloud-Volumes | [ ] | [ ] | rsync aus Backup |

### Backup-RPO-Deckung

| Backup-Frequenz | RPO abgedeckt? | Luecke |
|-----------------|---------------|--------|
| Taeglich (empfohlen) | RPO 24h | Daten aelter als 24h verloren |
| [ ] | [ ] | [ ] |

---

## 3. Single-Host Deployment — Risiken & Mitigation

| Risiko | Impact | Mitigation |
|--------|--------|------------|
| Host-Hardware-Fehler | Totalausfall | Backup extern lagern; [ ] Ersatz-Host bereithalten |
| Host-OS-Korruption | Totalausfall | [ ] Rebuild-Prozedur dokumentiert |
| Docker-Daemon-Ausfall | Alle Container down | [ ] Monitoring + Auto-Restart |
| Netzwerk-Ausfall | Nicht erreichbar | [ ] Failover-Netz / Provider-Wechsel |
| Ransomware / Loeschung | Datenverlust | Offsite-Backups; [ ] Immutable Backups |
| Fehlkonfiguration | Teilausfall | Change-Mgmt + Rollback (5 Steps) |

---

## 4. Disaster Recovery Plan (DR)

### DR-Trigger

| Bedingung | Aktion |
|-----------|--------|
| Host nicht erreichbar > [ ] min | DR-Plan aktivieren |
| Datenkorruption festgestellt | DR-Plan aktivieren |
| Ransomware entdeckt | DR-Plan aktivieren + Incident-Response |

### DR-Prozedur (Wiederaufbau auf neuem Host)

| Step | Aktion | Tool / Datei | Dauer (Schaetzung) |
|------|--------|-------------|-------------------|
| 1 | Neuen Host provisionieren (Debian x86_64) | [ ] (Ansible / Cloud-Init / manuell) | [ ] |
| 2 | Docker installieren | `apt install docker-ce` | [ ] |
| 3 | compose.yaml aus Repository deployen | git clone / scp | [ ] |
| 4 | .env mit Secrets bereitstellen | [ ] (Vault / 1Password / sicherer Kanal) | [ ] |
| 5 | Volumes anlegen | `docker volume create` (5x) | [ ] |
| 6 | Postgres-Restore | `pg_restore <neuestes_backup>` | [ ] |
| 7 | Volume-Restore (html, config, apps) | `rsync <backup>/ <volume_mount>/` | [ ] |
| 8 | Container starten | `docker compose up -d` | [ ] |
| 9 | Smoketest (healthcheck-monitor.sh) | healthcheck-monitor.sh | [ ] |
| 10 | DNS auf neue Host-IP umstellen | [ ] | [ ] |
| 11 | TLS-Zertifikate erneuern (ACME) | Traefik auto | [ ] |
| **Gesamt (geschätzt)** | | | **[ ] min/h** |

### DR-Szenario: Nur-DB-Wiederherstellung

| Step | Aktion | Tool |
|------|--------|------|
| 1 | Nextcloud in Maintenance-Mode | `occ maintenance:mode --on` |
| 2 | pg_restore | `pg_restore -d nextcloud <backup>` |
| 3 | Maintenance-Mode deaktivieren | `occ maintenance:mode --off` |
| 4 | Smoketest | healthcheck-monitor.sh |

---

## 5. DR-Test-Plan

| Test-Art | Frequenz | Verantwortlich | Letzter Test | Ergebnis |
|----------|----------|----------------|-------------|----------|
| Tabletop (Walkthrough) | [ ] (Empfehlung: jaehrlich) | [ ] | [ ] | [ ] |
| Backup-Restore (DB+Volumes) | [ ] (Empfehlung: quartalsweise) | [ ] | [ ] | [ ] |
| Full-DR-Test (neuer Host) | [ ] (Empfehlung: jaehrlich) | [ ] | [ ] | [ ] |

---

## 6. Kommunikation waehrend Continuity-Event

| Stakeholder | Wer informiert? | Kanal | Vorlage? |
|-------------|-----------------|-------|----------|
| IT-Leitung | [ ] | [ ] | [ ] |
| Enduser | [ ] | [ ] | [ ] |
| Externe Dienstleister | [ ] | [ ] | [ ] |

---

## 7. Kritische Dokumentation & Zugriff

| Dokument/Artefakt | Speicherort | Zugriff auch bei Primary-Ausfall? |
|-------------------|-------------|----------------------------------|
| compose.yaml | Git-Repository | [ ] |
| .env (Secrets) | [ ] | [ ] |
| Backup-Storage-Pfad | [ ] | [ ] |
| DR-Prozedur (dieses Dokument) | [ ] | [ ] |
| Incident-Response-Plan | [ ] | [ ] |
| Kontaktliste | [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
