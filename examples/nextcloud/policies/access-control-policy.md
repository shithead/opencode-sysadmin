# Access Control Policy (A.9 — ISO 27001 Annex A.9)

## Zweck
Regelung der Zugriffskontrolle auf das Nextcloud-System und alle zugehoerigen Komponenten.

## Rollen & Berechtigungen

### System-Rollen (Infrastruktur)

| Rolle | Zugriff | Scope | Authentifizierung |
|-------|---------|-------|-------------------|
| System-Administrator | SSH auf Host, Docker-Commands, alle Volumes | Gesamtes Host-System | SSH-Key + [ ] |
| DB-Administrator | PostgreSQL Superuser, pg_dump/pg_restore | nextcloud-postgres | Passwort + [ ] |
| Nextcloud-Admin | Nextcloud Admin-UI, occ CLI | nextcloud-app Container | NEXTCLOUD_ADMIN_PASSWORD |
| Backup-Operator | backup.sh Ausfuehrung, Zugriff auf Backup-Dir | Host + Backup-Storage | [ ] |
| Monitoring | healthcheck-monitor.sh Logs lesen, Status-Endpunkte | Alle Container | [ ] |

### Applikations-Rollen (Nextcloud via Keycloak OIDC)

| Rolle | Beschreibung | Quelle |
|-------|-------------|--------|
| OIDC-User | Standard-Nutzer via Keycloak-Authentifizierung | Keycloak IdP (auth.example.com) |
| [ ] | Weitere Nextcloud-interne Gruppen/Rollen | Nextcloud |
| [ ] | Gast/Externer Zugriff (falls konfiguriert) | Nextcloud |

## Authentifizierungsverfahren

| Verfahren | Verfuegbarkeit | Erzwungen? |
|-----------|---------------|------------|
| Passwort (lokal) | Ja (Nextcloud-Admin) | [ ] |
| OIDC (Keycloak) | Ja (auth.example.com) | [ ] |
| MFA via Keycloak | Verfuegbar | [ ] |
| SSH-Key | Ja (Host-Zugriff) | [ ] |

## MFA-Policy (ueber Keycloak)

- **MFA verfuegbar**: Keycloak unterstuetzt TOTP, WebAuthn, [ ]
- **Erzwungen fuer**: [ ] (Admin-Rollen), [ ] (alle User), [ ] (externe Zugriffe)
- **Recovery**: [ ]

## Passwort-Policy

| Parameter | Vorgabe |
|-----------|---------|
| Mindestlaenge | [ ] (Empfehlung: >= 12) |
| Komplexitaet | [ ] |
| Ablauf | [ ] (Empfehlung: 90 Tage) |
| History | [ ] |
| Account-Sperre nach | [ ] Fehlversuchen |

## Netzwerk-Zugriffskontrolle

| Quelle | Ziel | Port | Erlaubt? | Bedingung |
|--------|------|------|----------|-----------|
| Internet | Traefik | 443 | Ja | TLS 1.2+, ACME-valides Zertifikat |
| Internet | Traefik | 80 | [ ] | HTTP→HTTPS Redirect |
| Internet | Host direkt | SSH (22) | [ ] | Nur Management-Netze |
| nextcloud-app | nextcloud-postgres | 5432 | Ja | Internes Docker-Netz |
| nextcloud-app | nextcloud-redis | 6379 | Ja | ACL-basiert (REDIS_PASSWORD) |
| [ ] | [ ] | [ ] | [ ] | [ ] |

## Rate-Limiting

| Service | Typ | Average | Burst | Implementierung |
|---------|-----|---------|-------|-----------------|
| Nextcloud Frontend | Rate-Limit | 10 req/s | 20 req/s | Traefik Middleware |
| Keycloak Auth | [ ] | [ ] | [ ] | [ ] |
| Admin-Zugriff | [ ] | [ ] | [ ] | [ ] |

## Zugriffsueberpruefung

- **Review-Turnus**: [ ] (Empfehlung: quartalsweise)
- **Verantwortlich**: [ ]
- **De-/Provisioning bei Rollenwechsel**: [ ]
- **Offboarding-Prozess**: [ ]

## Logging & Audit

| Ereignis | Log-Quelle | Retention |
|----------|-----------|-----------|
| SSH-Logins | /var/log/auth.log | [ ] |
| Nextcloud-Logins | nextcloud.log | json-file, max-size 10m, max-file 3 |
| Keycloak-Auth-Events | Keycloak Audit Log | [ ] |
| Docker-API-Zugriffe | [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
