# opencode-systemadministration

Ein KI-Agenten-System fuer IT-Systemadministration — Docker/K8s-Setups mit Traefik/Nginx/HAProxy.

**6-Phasen-Pipeline**: Spec → Blueprint → Technische Validierung → ISO27001 Gap → Policy → Audit

## Quickstart

```bash
git clone <repo-url> opencode-systemadministration
cd opencode-systemadministration
opencode
```

Der Orchestrator-Agent ist der `default_agent` — du startest direkt in der Pipeline.

## Commands

| Command | Phase | Beschreibung |
|---------|-------|-------------|
| `/setup <programm>` | 1-3 | Setup-Pipeline: Spec → Blueprint → Validate |
| `/iso <programm>` | 4-6 | ISO27001-Pipeline: Gap → Policy → Audit |
| `/validate <programm>` | 3 | Nur Validierung auf bestehendem Setup |
| `/iso-generate <programm>` | — | Policy-Templates aus Gap-Report generieren |

## Beispiel

```bash
# Neues Programm einrichten
/setup mattermost
# oder mit existierendem Input:
cp examples/mattermost/input.json .

# ISO27001-Check
/iso nextcloud

# Policy-Templates generieren lassen
/iso-generate nextcloud
```

## Struktur

```
opencode.json              # Agent-Registrierung + Commands
agents/                    # Prompt-Templates (9 Agenten)
  orchestrator.md          # Chat-Agent, koordiniert die Pipeline
  spec-agent.md            # Phase 1: Soll-Spezifikation
  blueprint-agent.md       # Phase 2: Docker/K8s-Artefakte
  validation-agent.md      # Phase 3: Konsistenz (A-G)
  security-agent.md        # Phase 3: Security (S1-S5)
  ops-agent.md             # Phase 3: Betrieb (O1-O6)
  iso-gap-agent.md         # Phase 4: ISO-Lueckenanalyse
  iso-policy-agent.md      # Phase 5: Policy-Dokumentation
  iso-audit-agent.md       # Phase 6: Audit-Evidenz
  iso-generate-agent.md    # Policy-Template-Generator
schemas/                   # JSON-Schemas (5 Dateien)
examples/                  # Beispiel-Setups
  nextcloud/               #   Nextcloud (Docker + Traefik)
  keycloak/                #   Keycloak (Docker + Traefik)
```

## Programme

Unterstuetzt: Nextcloud, Keycloak, Mattermost, Odoo, n8n, Ansible, sowie Linux-Services und jede Docker/K8s-Workload.

## ISO27001

Die Pipeline unterstuetzt ISO27001-Zertifizierung in 3 Stufen:

1. **Gap-Analyse**: 15 Annex-A-Kontrollen auf Luecken pruefen
2. **Policy-Check**: 8 Policy-Dokumente auf Existenz & Aktualitaet
3. **Audit-Evidence**: 12 Kategorien mit konkreten Nachweisen (Logs, Configs, Git)

## Lizenz

GPLv3 — siehe [LICENSE](LICENSE)
