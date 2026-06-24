# AGENTS.md — opencode-systemadministration

## What this is
A prompt-template orchestration system for IT program setups (Nextcloud, Keycloak, Mattermost, Odoo, n8n, etc.) with Docker/K8s + Traefik/Nginx/HAProxy. **Not a software project** — no build, test, lint, or codegen. The "code" is German-language agent prompts in `agents/` and JSON schemas in `schemas/`.

## Architecture: 6-Phase Pipeline

```
Phase 1: Spec-Agent       → Gate 1 → Spec valid?
Phase 2: Blueprint-Agent  → Gate 2 → Blueprint complete?
Phase 3: Validation + Security + Ops (parallel) → Gate 3 → Technically validated?
Phase 4: ISO-Gap-Agent    → Gate 4 → No critical gaps?
Phase 5: ISO-Policy-Agent → Gate 5 → All policies valid?
Phase 6: ISO-Audit-Agent  → Gate 6 → Audit-ready?
```

Phases 1-3 are the setup pipeline. Phases 4-6 (ISO27001) are optional and gated — each requires the previous phase to be green.

The entrypoint is `agents/orchestrator.md`. The orchestrator dispatches subagents via OpenCode's `task` tool.

## Critical Agent Conventions

### ALL agents MUST output ONLY JSON — no markdown, no code fences, no explanations.
The single most-common mistake: agents wrapping JSON in ```json``` fences or adding prose. Every agent template ends with: **"Gib NUR das JSON-Objekt zurueck, kein Markdown, keine Erklaerungen."**

### Secrets: NEVER plaintext
All agent output must use `${VAR}` placeholders for secrets. No hardcoded passwords, tokens, or keys in any config, template, or blueprint.

### Agent language: German
All agent prompts are written in German. Agent names and roles use German terms (Setup-Assembler, Execution-Blueprint, etc.). Output fields use English/generated names but descriptions and instructions are German.

## Subagents (8 total)

| Agent | File | Phase | Output Schema |
|-------|------|-------|---------------|
| Spec-Agent (Setup-Assembler) | `agents/spec-agent.md` | 1 | `schemas/spec.schema.json` |
| Blueprint-Agent | `agents/blueprint-agent.md` | 2 | `schemas/blueprint.schema.json` |
| Validation-Agent | `agents/validation-agent.md` | 3 | `schemas/validation-result.schema.json` |
| Security-Agent | `agents/security-agent.md` | 3 | `schemas/validation-result.schema.json` |
| Ops-Agent | `agents/ops-agent.md` | 3 | `schemas/validation-result.schema.json` |
| ISO-Gap-Agent | `agents/iso-gap-agent.md` | 4 | `schemas/validation-result.schema.json` |
| ISO-Policy-Agent | `agents/iso-policy-agent.md` | 5 | `schemas/validation-result.schema.json` |
| ISO-Audit-Agent | `agents/iso-audit-agent.md` | 6 | `schemas/iso-audit-report.schema.json` |

Phase 3 agents run **in parallel** (single message, multiple task tool calls).

## Gate Logic (from orchestrator)

- Each gate checks required fields and completeness after its phase
- On failure: describe the deficiency, retry the agent with feedback — **max 2 retries**, then ask the user
- Gate 3 (Phase 3): aggregate all three validator results into a combined table (categories A-G, S1-S5, O1-O6)

## Data Flow

All intermediate artifacts are saved as JSON in `examples/<program>/`:
```
examples/nextcloud/
  input.json              ← User input
  spec-output.json        ← Phase 1 output
  blueprint-output.json   ← Phase 2 output
  validation-output.json  ← Phase 3: Validation agent result
  security-output.json    ← Phase 3: Security agent result
  ops-output.json         ← Phase 3: Ops agent result
```

The orchestrator reads prior outputs and passes them to subsequent agents as JSON in the prompt.

## Schema Conventions

- All schemas: JSON Schema draft 2020-12 (`$schema: "https://json-schema.org/draft/2020-12/schema"`)
- `$id` uses `https://opencode-systemadministration/schemas/<name>.schema.json`
- Schemas and agent templates are in-sync by design — agents reference the schema they must conform to
- The `validation-result.schema.json` is shared by Validation, Security, Ops, ISO-Gap, and ISO-Policy agents (uses `patternProperties` for flexible category names)

## Key Files

| File | Purpose |
|------|---------|
| `basisplanung.txt` | Original requirements document (German). Source of truth for what the system should do. |
| `agents/orchestrator.md` | Main chat agent. Entrypoint for users. Contains the full pipeline definition and gate logic. |
| `agents/spec-agent.md` | Phase 1 — build target specification from user input |
| `agents/blueprint-agent.md` | Phase 2 — create implementation artifacts (compose YAML, etc.) |
| `schemas/input.schema.json` | User input contract (programmes, environment, frontdoor) |
| `schemas/spec.schema.json` | Spec agent output contract |
| `schemas/blueprint.schema.json` | Blueprint agent output contract |
| `schemas/validation-result.schema.json` | Shared validation output contract |
| `schemas/iso-audit-report.schema.json` | Audit evidence report contract |

## Onboarding: First Dry-Run

To test a new program:
1. Create `examples/<program>/input.json` following `schemas/input.schema.json`
2. Run the orchestrator: pass the input, it dispatches Phase 1-3 sequentially
3. Fix any issues found by the validation agents, re-run until all gates green
4. Optionally continue to ISO pipeline (Phase 4-6)

**No build/test commands exist** — the system is tested by running the pipeline end-to-end on example inputs.

## OpenCode Integration

All agents are registered in `opencode.json`. The orchestrator is the `default_agent` (`mode: primary`). All 8 subagents are `hidden: true` (invoked only by the orchestrator via `task` tool, not by users via `@`).

```json
// openencode.json — agent registration
"agent": {
  "orchestrator":    { "mode": "primary",  "prompt": "{file:./agents/orchestrator.md}" },
  "spec-agent":      { "mode": "subagent", "prompt": "{file:./agents/spec-agent.md}",      "hidden": true },
  "blueprint-agent": { "mode": "subagent", "prompt": "{file:./agents/blueprint-agent.md}", "hidden": true },
  "validation-agent":{ "mode": "subagent", "prompt": "{file:./agents/validation-agent.md}","hidden": true },
  "security-agent":  { "mode": "subagent", "prompt": "{file:./agents/security-agent.md}",  "hidden": true },
  "ops-agent":       { "mode": "subagent", "prompt": "{file:./agents/ops-agent.md}",       "hidden": true },
  "iso-gap-agent":   { "mode": "subagent", "prompt": "{file:./agents/iso-gap-agent.md}",   "hidden": true },
  "iso-policy-agent":{ "mode": "subagent", "prompt": "{file:./agents/iso-policy-agent.md}","hidden": true },
  "iso-audit-agent": { "mode": "subagent", "prompt": "{file:./agents/iso-audit-agent.md}", "hidden": true }
}
```

`default_agent` is `orchestrator` — OpenCode starts with the pipeline coordinator.
`instructions: ["AGENTS.md"]` loads this file as additional context.
The `.gitignore` excludes `basisplanung.txt` (reference document, not shipped).
