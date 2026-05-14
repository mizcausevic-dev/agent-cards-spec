# Agent Cards

[![Validate examples](https://github.com/mizcausevic-dev/agent-cards-spec/actions/workflows/validate.yml/badge.svg)](https://github.com/mizcausevic-dev/agent-cards-spec/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

A draft specification for **Agent Cards** — declarative documents that disclose what an AI agent is, what it can do, what it refuses, and how it has been evaluated.

HuggingFace gave models a way to disclose themselves through model cards. Agents need the same — but the disclosure surface is *different*. Agents have **tool surfaces, refusal behavior, memory persistence, and deployment posture** that models alone do not. An Agent Card is the document that makes those properties machine-readable, auditable, and comparable across agents.

## The three pillars

| Pillar | What it does |
|---|---|
| **Disclose** | A declarative capability surface — what models, what tools, what context window, what memory model |
| **Constrain** | A refusal taxonomy and a list of categories that require human-in-the-loop approval |
| **Audit** | Eval results, deployment posture, and an incident-response surface |

## Why not just rely on model cards?

A model card documents an LLM. An agent is a system *built on top of* an LLM, with tools, prompts, and policies layered in. The agent's capability surface is not the model's capability surface.

Two examples of agent properties that model cards cannot express:

- An agent built on Claude Opus may be configured to refuse a category the base model would not refuse, because of a safety prompt or a policy layer
- An agent's tool surface (e.g. "can write to your AWS account") is determined by integration choices, not by the model

## Why not just a README?

READMEs are unstructured. An Agent Card is parseable: a CI pipeline can validate it, a registry can index it, a procurement team can compare two agents side-by-side without reading paragraphs of prose.

## Quickstart

1. Pick an `agent.id` (kebab-case, unique within your org or product line).
2. Author an Agent Card document conforming to [`agent-card.schema.json`](agent-card.schema.json). Start from [examples](examples/).
3. Validate with any JSON Schema 2020-12 validator.
4. Publish at `https://<your-product>/.well-known/agents/<agent-id>.json` (preferred) or alongside the agent's source.
5. Reference Tool Cards ([mcp-tool-card-spec](https://github.com/mizcausevic-dev/mcp-tool-card-spec)) and Prompt Provenance records ([prompt-provenance-spec](https://github.com/mizcausevic-dev/prompt-provenance-spec)) where applicable.

## Files in this repo

- [`SPEC.md`](SPEC.md) — full v0.1 specification
- [`agent-card.schema.json`](agent-card.schema.json) — JSON Schema (draft 2020-12)
- [`examples/`](examples/) — reference documents for a customer-support agent, an SRE incident agent, and a sandboxed eval agent

## Status

**v0.1 draft.** Issues and pull requests welcome.

## License

MIT-licensed. The specification text, JSON Schema, and example documents in this repository may be freely implemented, extended, redistributed, or incorporated into commercial or non-commercial products with attribution. Reference implementations of this spec (such as [mcp-kinetic-gain](https://github.com/mizcausevic-dev/mcp-kinetic-gain)) are licensed separately under AGPL-3.0.

## Kinetic Gain Protocol Suite

A family of nine open specifications for the answer-engine and agent era. Each spec is a self-contained JSON document format with its own JSON Schema and reference examples; together they compose into an end-to-end account of entity, agent, prompt, tool, citation, EdTech disclosure, and incident reporting. Single landing: [`kinetic-gain-protocol-suite`](https://github.com/mizcausevic-dev/kinetic-gain-protocol-suite).

| Spec | What it does |
|---|---|
| [AEO Protocol](https://github.com/mizcausevic-dev/aeo-protocol-spec) | Entity declaration at `/.well-known/aeo.json` — authoritative claims, citation preferences, audit hooks |
| [Prompt Provenance](https://github.com/mizcausevic-dev/prompt-provenance-spec) | Versioned, lineaged, reviewable LLM prompt records |
| **[Agent Cards](https://github.com/mizcausevic-dev/agent-cards-spec)** (this) | Declarative agent capability and refusal disclosure |
| [AI Evidence Format](https://github.com/mizcausevic-dev/ai-evidence-format-spec) | Structured citations that travel with LLM-generated claims |
| [MCP Tool Cards](https://github.com/mizcausevic-dev/mcp-tool-card-spec) | Per-tool disclosure layered on Model Context Protocol servers |
| [AI Tutor Cards](https://github.com/mizcausevic-dev/ai-tutor-card-spec) | EdTech-specialized agent disclosure (vendor-side) |
| [Student AI Disclosure](https://github.com/mizcausevic-dev/student-ai-disclosure-spec) | Student-side disclosure attached to submitted work |
| [Classroom AI AUP](https://github.com/mizcausevic-dev/classroom-ai-aup-spec) | District / school / course AI policy (third leg of the EdTech trio) |
| [AI Incident Card](https://github.com/mizcausevic-dev/ai-incident-card-spec) | Post-incident disclosure for AI agents — references this spec via `affected.agent_card_uris[]` |
| [Clinical AI Disclosure](https://github.com/mizcausevic-dev/clinical-ai-disclosure-spec) | **HealthTech vertical** — vendor disclosure for healthcare AI (HIPAA / FDA / SaMD). References this spec via `agent_card_uri`. |

### Related testing artifact

| Repo | What it does |
|---|---|
| [`prompt-injection-bench`](https://github.com/mizcausevic-dev/prompt-injection-bench) | 30-attack corpus + Python harness for prompt injection. Every record carries an `agent_card_refusal_categories` back-ref to this spec's `refusal_taxonomy[].category` — grep your declared categories against the corpus to test whether your stated commitments hold under attack. |

---

**Connect:** [LinkedIn](https://www.linkedin.com/in/mirzacausevic/) · [Kinetic Gain](https://kineticgain.com) · [Medium](https://medium.com/@mizcausevic/) · [Skills](https://mizcausevic.com/skills/)
