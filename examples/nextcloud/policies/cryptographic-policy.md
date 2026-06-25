# Cryptographic Policy (A.10 — ISO 27001 Annex A.10)

## Zweck
Festlegung kryptografischer Standards fuer das Nextcloud-Setup inkl. Traefik-TLS, Datenbankverschluesselung und Secret-Management.

## TLS-Konfiguration (Traefik)

### Mindeststandards

| Parameter | Vorgabe | Status |
|-----------|---------|--------|
| TLS Minimalversion | 1.2 | Produktiv |
| TLS Maximalversion | [ ] (Empfehlung: 1.3) | [ ] |
| Cipher Suites | ECDHE+AES-GCM priorisiert | Produktiv |
| Key Exchange | ECDHE (Perfect Forward Secrecy) | Produktiv |
| HSTS | Aktiviert (Strict-Transport-Security) | Produktiv |
| HSTS max-age | [ ] (Empfehlung: >= 31536000) | [ ] |
| HSTS includeSubDomains | [ ] | [ ] |
| HSTS preload | [ ] | [ ] |

### Bevorzugte Cipher-Liste (Traefik)
```
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
[ ] Weitere gemaess Security-Audit
```

## Zertifikatsmanagement

| Aspekt | Verfahren |
|---------|-----------|
| Aussteller | ACME (Let's Encrypt / [ ]) |
| Validierung | [ ] (HTTP-01 / DNS-01) |
| Auto-Rotation | ACME automatisch (Traefik) |
| Rotations-Intervall | 60 Tage (ACME default, Erneuerung bei < 30 Tage) |
| Wildcard-Zertifikate | [ ] |
| Private-Key-Algorithmus | RSA [ ] Bit (Empfehlung: >= 2048) oder ECDSA |
| Private-Key-Speicherung | Traefik acme.json ([ ]) |

## Hash-Algorithmen

| Einsatzzweck | Algorithmus | Status |
|--------------|-------------|--------|
| Zertifikat-Signatur | SHA-256+ | Produktiv |
| Passwort-Hashing (Nextcloud) | [ ] (bcrypt / argon2id) | [ ] |
| Passwort-Hashing (Keycloak) | [ ] (PBKDF2 / bcrypt / argon2) | [ ] |
| Datenintegritaet (Backup) | [ ] (SHA-256 / SHA-512) | [ ] |

## Datenverschluesselung

### Data at Rest

| Daten | Verschluesselt? | Verfahren |
|-------|----------------|-----------|
| Nextcloud-Volumes (html, config, apps) | [ ] | [ ] (LUKS / eCryptfs / app-seitig) |
| Postgres-Daten | [ ] | [ ] |
| Redis-Daten | [ ] | N/A (Cache, keine sensiblen Daten) |
| Backup-Target | [ ] | [ ] (GPG / LUKS / rsync-over-SSH) |
| Host-Disk | [ ] | [ ] (LUKS dm-crypt) |

### Data in Transit

| Verbindung | Verschluesselt? | Verfahren |
|------------|----------------|-----------|
| Client → Traefik | Ja | TLS 1.2+ |
| Traefik → nextcloud-app | [ ] | Internes Docker-Netz ([ ] TLS / plain) |
| nextcloud-app → postgres | [ ] | Internes Docker-Netz ([ ] TLS / plain) |
| nextcloud-app → redis | [ ] | ACL-basiert (REDIS_PASSWORD) |
| nextcloud-app → Keycloak | [ ] | TLS via Domains |
| Backup-Transfer | [ ] | [ ] |

## Secret-Management

| Aspekt | Verfahren |
|---------|-----------|
| Secrets im Code | Verboten — NUR via ${VAR}-Platzhalter |
| Secrets in Configs | ${VAR}-Substitution zur Laufzeit |
| Secrets auf Disk | .env-Datei (0600), Docker Secrets, [ ] |
| Secret-Rotation | [ ] (keine automatische Rotation implementiert) |
| .env-Datei Berechtigung | [ ] (Empfehlung: 0600, owner root) |
| .env im Backup | [ ] (Explizit ausgeschlossen / verschluesselt) |

## Schluesselmanagement

| Schluessel-Typ | Lebenszyklus | Speicherort |
|---------------|-------------|-------------|
| TLS Private Keys | ACME auto (90 Tage) | Traefik acme.json |
| DB Passwort (DB_PASSWORD) | [ ] | .env |
| Admin Passwort (NEXTCLOUD_ADMIN_PASSWORD) | [ ] | .env |
| OIDC Client Secret | [ ] | .env |
| Redis Passwort | [ ] | .env |
| SSH Host Keys | Manuell / auto-generiert | /etc/ssh/ |

## Kryptografische Bibliotheken & Abhaengigkeiten

| Komponente | Verwendete Bibliothek | Version | Patch-Strategie |
|------------|----------------------|---------|-----------------|
| Traefik TLS | Go crypto/tls | Image-gebunden | Traefik-Update |
| Nextcloud Verschl. | OpenSSL via PHP | Image-gebunden | Nextcloud-Update |
| Keycloak TLS | Java JCE/JSSE | Image-gebunden | Keycloak-Update |
| Postgres SSL | OpenSSL | Image-gebunden | Postgres-Update |

## Verbotene Algorithmen

| Algorithmus | Grund |
|-------------|-------|
| SSLv3, TLS 1.0, TLS 1.1 | Veraltet, unsicher |
| RC4, 3DES, DES | Unsicher |
| MD5, SHA-1 | Kollisionsanfaellig |
| [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
