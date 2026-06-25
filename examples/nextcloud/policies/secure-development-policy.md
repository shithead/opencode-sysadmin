# Secure Development Policy (A.14 — ISO 27001 Annex A.14)

## Zweck
Sicherstellung, dass Konfiguration, Deployment-Artefakte und Skripte des Nextcloud-Setups nach sicheren Entwicklungsprinzipien erstellt und gewartet werden.

**Hinweis**: Dieses Setup liefert keine Eigenentwicklung im klassischen Sinne. Diese Policy behandelt das IaC- und Configuration-Management (compose.yaml, Shell-Skripte, Dockerfiles) als "Entwicklungsartefakte".

---

## 1. Image-Management

### Image-Tag-Policy

| Container | Image | Tag-Strategie | Status |
|-----------|-------|---------------|--------|
| nextcloud-app | nextcloud | Fixiert: 31.0.2-apache | Produktiv |
| nextcloud-postgres | postgres | Fixiert: 16.6-alpine | Produktiv |
| nextcloud-redis | redis | Fixiert: 7.4-alpine | Produktiv |
| Traefik (Reverse-Proxy) | traefik | [ ] (Empfehlung: fixiert) | [ ] |
| Keycloak | [ ] | [ ] (Empfehlung: fixiert) | [ ] |

### Image-Update-Prozess

| Step | Aktion | Verantwortlich |
|------|--------|----------------|
| 1 | Neue Version im Changelog pruefen | [ ] |
| 2 | Security-Advisories pruefen (CVE) | [ ] |
| 3 | Image in Staging/Test-Umgebung deployen | [ ] |
| 4 | Smoketest + Integrationstest | [ ] |
| 5 | compose.yaml Tag aktualisieren | [ ] |
| 6 | Deployment in Produktion (gemaess Change-Mgmt) | [ ] |

### Verbotene Tags

- `latest` — nicht erlaubt (unbestimmte Version)
- `:dev`, `:nightly`, `:master` — nicht erlaubt
- Tags ohne Digest-Pinning (optional: `image@sha256:...` fuer zusaetzliche Sicherheit)

---

## 2. Secret-Management

### Grundsaetze

- **KEINE** Secrets im Code (compose.yaml, Skripte, Config-Dateien)
- **AUSSCHLIESSLICH** `${VAR}`-Platzhalter mit Substitution zur Laufzeit
- Secrets nie in Git committen (.gitignore fuer .env)
- Keine Default-Werte fuer Secrets in compose.yaml

### Verwendete Secrets

| Variable | Verwendung | In compose.yaml? | In Backup? |
|----------|-----------|-----------------|------------|
| `${DB_PASSWORD}` | Postgres-Verbindung | Ja (Platzhalter) | [ ] |
| `${NEXTCLOUD_ADMIN}` | Admin-Benutzername | Ja (Platzhalter) | [ ] |
| `${NEXTCLOUD_ADMIN_PASSWORD}` | Admin-Passwort | Ja (Platzhalter) | [ ] |
| `${OIDC_CLIENT_ID}` | Keycloak OIDC Client | Ja (Platzhalter) | [ ] |
| `${OIDC_CLIENT_SECRET}` | Keycloak OIDC Secret | Ja (Platzhalter) | [ ] |
| `${REDIS_PASSWORD}` | Redis ACL | Ja (Platzhalter) | [ ] |

### .env-Datei

| Aspekt | Vorgabe |
|---------|---------|
| Datei-Berechtigung | 0600 |
| Owner | [ ] (Empfehlung: root) |
| In Git? | Nein (.gitignore) |
| Im Backup? | [ ] |
| Verteilung | [ ] |

---

## 3. Dependency-Management

### Container-Images als Dependencies

| Aspekt | Verfahren |
|---------|-----------|
| Registry | Docker Hub ([ ] alternativ private Registry) |
| Image-Verifikation | [ ] (Docker Content Trust / Cosign / Signaturen) |
| Vulnerability-Scanning | [ ] (Trivy / Grype / Docker Scout) |
| SBOM | [ ] |

### Base-Image-Sicherheit

| Image | Base | User | CVE-Check |
|-------|------|------|-----------|
| nextcloud | Apache + PHP | 33:33 (www-data) | [ ] |
| postgres | Alpine Linux | [ ] | [ ] |
| redis | Alpine Linux | [ ] | [ ] |

### Skript-Dependencies

| Skript | Abhaengigkeiten | Version geprueft? |
|--------|----------------|-------------------|
| backup.sh | pg_dump, rsync, bash | [ ] |
| healthcheck-monitor.sh | curl, bash | [ ] |
| [ ] | [ ] | [ ] |

---

## 4. Code-Review & Qualitaetssicherung

### Artefakte unter Versionskontrolle

| Artefakt | Repository | Review-Pflicht? |
|----------|------------|-----------------|
| compose.yaml | [ ] | [ ] |
| backup.sh | [ ] | [ ] |
| healthcheck-monitor.sh | [ ] | [ ] |
| .env.example (ohne Secrets) | [ ] | [ ] |
| Traefik-Konfiguration | [ ] | [ ] |

### Shell-Skript-Standards

- `set -euo pipefail` in allen Skripten
- Keine unquoted variables
- Keine eval/exec mit ungeprueftem Input
- [ ] ShellCheck-Linting vor Commit

---

## 5. Testumgebung

| Aspekt | Status |
|--------|--------|
| Separate Staging-Umgebung vorhanden? | [ ] |
| UAT vor Produktions-Deployment? | [ ] |
| Automatisierte Integrationstests? | [ ] |
| Last-/Performance-Tests? | [ ] |

---

## 6. Secure-Configuration-Baseline

### Nextcloud

| Einstellung | Vorgabe | Status |
|-------------|---------|--------|
| HSTS | Via Traefik | Aktiviert |
| CSP (Content-Security-Policy) | [ ] | [ ] |
| CORS | [ ] | [ ] |
| File-Upload-Limit | [ ] | [ ] |
| Brute-Force-Schutz | [ ] | [ ] |
| App Store (Zugriff) | [ ] | [ ] |

### Postgres

| Einstellung | Vorgabe | Status |
|-------------|---------|--------|
| `password_encryption` | scram-sha-256 | [ ] |
| `log_connections` | [ ] | [ ] |
| `log_disconnections` | [ ] | [ ] |

### Redis

| Einstellung | Vorgabe | Status |
|-------------|---------|--------|
| ACL aktiviert | Ja (REDIS_PASSWORD) | Produktiv |
| `protected-mode` | [ ] | [ ] |
| Command-Renaming | [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
