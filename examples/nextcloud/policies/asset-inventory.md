# Asset Inventory (A.8 — ISO 27001 Annex A.8)

## Scope
Nextcloud-Setup (Docker, Debian x86_64), inkl. Traefik-Reverse-Proxy, Keycloak-OIDC-IdP, Postgres, Redis.

## Hardware Assets

| Asset | Typ | Host/Node | CPU | RAM | Disk | Netzwerk | Status |
|-------|-----|-----------|-----|-----|------|----------|--------|
| Debian Host | Physisch/VM | [ ] | x86_64 | [ ] | [ ] | [ ] | Produktiv |

## Container Assets

| Container | Image | Tag | Port(s) | User | Netz | Status |
|-----------|-------|-----|---------|------|------|--------|
| nextcloud-app | nextcloud | 31.0.2-apache | 80 (intern) | 33:33 (www-data) | [ ] | Produktiv |
| nextcloud-postgres | postgres | 16.6-alpine | 5432 (intern) | [ ] | [ ] | Produktiv |
| nextcloud-redis | redis | 7.4-alpine | 6379 (intern) | [ ] | [ ] | Produktiv |

## Volume Assets

| Volume | Pfad (Host) | Mount (Container) | Groesse | Backup? | Status |
|--------|-------------|-------------------|---------|---------|--------|
| nextcloud-html | [ ] | /var/www/html | [ ] | Ja (rsync) | Produktiv |
| nextcloud-config | [ ] | [ ] | [ ] | Ja (rsync) | Produktiv |
| nextcloud-apps | [ ] | [ ] | [ ] | Ja (rsync) | Produktiv |
| postgres-data | [ ] | /var/lib/postgresql/data | [ ] | Ja (pg_dump) | Produktiv |
| redis-data | [ ] | /data | [ ] | Nein (cache) | Produktiv |

## Domain / DNS Assets

| Domain | Zweck | TLS | Provider | Status |
|--------|-------|-----|----------|--------|
| cloud.example.com | Nextcloud Frontend | ACME (Traefik), TLS 1.2+ | [ ] | Produktiv |
| auth.example.com | Keycloak OIDC-IdP | ACME (Traefik), TLS 1.2+ | [ ] | Produktiv |

## Secret Assets (alle via ${VAR}-Platzhalter)

| Secret | Umgebungsvariable | Rotation | Zugriff | Speicherort |
|--------|-------------------|----------|---------|-------------|
| DB_PASSWORD | ${DB_PASSWORD} | [ ] | nextcloud-app Container | Docker Secret / .env |
| NEXTCLOUD_ADMIN | ${NEXTCLOUD_ADMIN} | [ ] | nextcloud-app Container | Docker Secret / .env |
| NEXTCLOUD_ADMIN_PASSWORD | ${NEXTCLOUD_ADMIN_PASSWORD} | [ ] | nextcloud-app Container | Docker Secret / .env |
| OIDC_CLIENT_ID | ${OIDC_CLIENT_ID} | [ ] | nextcloud-app Container | Docker Secret / .env |
| OIDC_CLIENT_SECRET | ${OIDC_CLIENT_SECRET} | [ ] | nextcloud-app Container | Docker Secret / .env |
| REDIS_PASSWORD | ${REDIS_PASSWORD} | [ ] | nextcloud-app + nextcloud-redis Container | Docker Secret / .env |

## Netzwerk-Topologie (vereinfacht)

```
Internet → Traefik (:443) → nextcloud-app (:80)
                          → Keycloak
           nextcloud-app → nextcloud-postgres (:5432)
           nextcloud-app → nextcloud-redis (:6379)
```

## Asset Owner

| Asset-Gruppe | Owner (Rolle) | Kontakt |
|--------------|---------------|---------|
| Container & Volumes | [ ] | [ ] |
| Domains & TLS | [ ] | [ ] |
| Secrets | [ ] | [ ] |
| Host-System | [ ] | [ ] |

## Bewertung (CIA)

| Asset | Confidentiality | Integrity | Availability | Klassifikation |
|-------|----------------|-----------|--------------|----------------|
| nextcloud-app | [ ] | [ ] | [ ] | [ ] |
| nextcloud-postgres | Hoch | Hoch | Hoch | Kritisch |
| nextcloud-redis | [ ] | [ ] | [ ] | [ ] |
| Volumes (alle) | Hoch | Hoch | Hoch | Kritisch |
| Domains | [ ] | [ ] | [ ] | [ ] |
| Secrets | Sehr hoch | Sehr hoch | [ ] | Kritisch |
| Host-System | [ ] | [ ] | [ ] | [ ] |

---
*Letzte Aktualisierung: [ ]*  
*Review-Turnus: [ ]*
