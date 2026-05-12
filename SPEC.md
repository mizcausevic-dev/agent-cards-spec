# Agent Cards v0.1 — Specification

**Status:** Draft
**Version:** 0.1.0
**Editor:** Miz Causevic
**License:** AGPL-3.0 (this document, schema, and examples). Implementations are unrestricted.

RFC 2119 keywords (**MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, **MAY**) apply throughout.

---

## 1. Scope

This specification defines a JSON document format for disclosing an AI agent's capability surface, refusal behavior, evaluations, and deployment posture. The audience is buyers, security reviewers, procurement teams, and downstream operators who need machine-readable disclosure.

The spec does **not** standardize the agent's runtime, the LLM API, the prompt format, or the tool calling convention. Agent Cards reference those by URI.

## 2. Terminology

- **Agent** — a software system that combines one or more LLMs, one or more tools, and a control loop to accomplish tasks on behalf of a user or another system.
- **Tool surface** — the set of tools the agent is permitted to call.
- **Refusal category** — a class of inputs or outputs the agent declines.
- **Memory persistence** — whether the agent retains state between sessions: `none`, `session`, `persistent`.
- **Deployment environment** — `production`, `staging`, `sandbox`, or `experimental`.

## 3. The three pillars

### 3.1 Disclose

The Agent Card **MUST** describe:
- Which LLMs the agent uses and in what role (planner, executor, critic, etc.)
- The complete tool surface
- The maximum context window the agent is configured to use
- The agent's memory model

These are declarative; the spec does not require runtime enforcement, but consumers **MAY** treat undeclared tools or models as policy violations.

### 3.2 Constrain

Every Agent Card **MUST** include a `refusal_taxonomy` listing the categories the agent declines, even if the list is empty. An empty list signals "no documented refusals," which itself is meaningful disclosure.

Every Agent Card **SHOULD** include `safety_posture.human_in_loop_required` — the categories where the agent will not act without human approval.

### 3.3 Audit

Every Agent Card **SHOULD** include `evaluations` — one or more entries linking to eval suite results. The spec does not embed scores; it references them by URI.

Production deployments **SHOULD** additionally include `safety_posture.incident_response_uri` — a working surface for reporting agent misbehavior.

## 4. Document structure

### 4.1 `agent_card_version` (required)

A semver string. **MUST** be `"0.1"` for documents conforming to this draft.

### 4.2 `agent` (required)

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | yes | Stable kebab-case identifier. |
| `name` | string | yes | Display name. |
| `version` | string | yes | Semver. |
| `provider` | string | yes | Org or vendor that operates the agent. |
| `homepage` | URI | no | Product page or docs. |
| `description` | string | yes | One-paragraph summary of what the agent does. |

### 4.3 `capabilities` (required)

| Field | Type | Required | Description |
|---|---|---|---|
| `primary_purpose` | string | yes | One-sentence statement of the agent's job. |
| `models_used` | array of object | yes | List of models. Each has `name`, optional `provider`, and `role` (`planner` / `executor` / `critic` / `router` / `judge`). |
| `tools` | array of object | yes | List of tools. Each has `name`, optional `mcp_tool_card_uri`, and `side_effects` (`read` / `mutating` / `external` / `destructive`). |
| `max_context_tokens` | integer | yes | Hard cap on context window the agent uses. |
| `memory_persistence` | enum | yes | `none` / `session` / `persistent`. |
| `autonomy_level` | enum | yes | `assistive` / `supervised` / `autonomous`. See §5. |
| `prompts_used` | array of URI | no | Links to Prompt Provenance records ([prompt-provenance-spec](https://github.com/mizcausevic-dev/prompt-provenance-spec)). |

### 4.4 `refusal_taxonomy` (required)

An array (possibly empty) of refusal entries:

| Field | Type | Required | Description |
|---|---|---|---|
| `category` | string | yes | Short identifier (`personal_data_extraction`, `financial_advice`, etc.). |
| `behavior` | enum | yes | `refuse_silently` / `refuse_and_explain` / `escalate_to_human` / `redirect_to_alternative`. |
| `example_prompts` | array of string | no | Example inputs that trigger this category. |

### 4.5 `evaluations` (optional but recommended)

| Field | Type | Required | Description |
|---|---|---|---|
| `suite` | string | yes | Suite name. |
| `result_uri` | URI | yes | Detailed results. |
| `metrics` | object | no | Free-form metrics object. |
| `ran_at` | datetime | yes | ISO 8601 UTC. |

### 4.6 `deployment` (required)

| Field | Type | Required | Description |
|---|---|---|---|
| `environment` | enum | yes | `production` / `staging` / `sandbox` / `experimental`. |
| `rate_limits` | object | no | Free-form rate-limit description. |
| `uptime_sla` | string | no | E.g. `99.9%`. |
| `regions` | array of string | no | Deployment regions. |

### 4.7 `safety_posture` (required)

| Field | Type | Required | Description |
|---|---|---|---|
| `human_in_loop_required` | array of string | yes | Categories requiring human approval before action. |
| `audit_log_uri` | URI | no | Where call logs are retained. |
| `incident_response_uri` | URI | no | Where misbehavior can be reported. |
| `disclosure_uri` | URI | no | Public disclosure / changelog. |

## 5. Autonomy levels

| Level | Meaning |
|---|---|
| `assistive` | Agent suggests; humans always act. |
| `supervised` | Agent acts, but actions are reversible and subject to human review within a short window. |
| `autonomous` | Agent acts without per-action review; actions may be irreversible. Strong audit and incident-response surfaces required. |

`autonomous` agents **MUST** declare `safety_posture.incident_response_uri`.

## 6. Conformance levels

| Level | Requirements |
|---|---|
| **Level 1 — Disclose** | Schema-valid document with all required fields. |
| **Level 2 — Constrain** | Level 1, plus a non-empty `refusal_taxonomy` OR an explicit `"declared_no_refusals": true` note in `description`. |
| **Level 3 — Audit** | Level 2, plus `evaluations` with at least one entry whose `ran_at` is within the last 90 days. |

## 7. Security and privacy considerations

- **Truthful disclosure.** An Agent Card is a publisher's representation. Consumers **MUST NOT** treat the card as ground truth for runtime safety; it is a disclosure document, not an enforcement mechanism.
- **Tool surface leakage.** Declaring tools publicly may aid attackers. Providers **MAY** withhold the Agent Card from the public web; in that case, distribute it to authenticated consumers only.
- **PII in `example_prompts`.** Example refusal-triggering prompts **MUST NOT** contain real personal data, real secrets, or real customer content.

## 8. Relationship to existing work

| Standard | Relationship |
|---|---|
| **HuggingFace Model Cards** | Companion. Model card describes a model; Agent Card describes a system built on one or more models. |
| **MCP Tool Cards** ([mcp-tool-card-spec](https://github.com/mizcausevic-dev/mcp-tool-card-spec)) | Companion. Agent Cards reference Tool Cards via `mcp_tool_card_uri`. |
| **Prompt Provenance** ([prompt-provenance-spec](https://github.com/mizcausevic-dev/prompt-provenance-spec)) | Companion. Agent Cards reference Prompt Provenance records via `capabilities.prompts_used`. |
| **AEO Protocol** ([aeo-protocol-spec](https://github.com/mizcausevic-dev/aeo-protocol-spec)) | An organization's AEO declaration MAY reference Agent Card URIs under `aeo:operates`. |

## 9. Open questions

- **Versioning the refusal taxonomy.** Should refusal categories be a shared registry, or per-agent?
- **Tested vs declared tool surface.** Should the spec distinguish between tools the agent *can* call and tools the agent has *been tested calling*?
- **Multi-agent systems.** Should a system of agents (e.g. a planner + N executors) be one card or N linked cards? Current draft assumes one card per externally-addressable agent.
