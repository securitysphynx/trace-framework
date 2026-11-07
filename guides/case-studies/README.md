# TRACE Case Studies

These are worked examples of TRACE applied after the fact to publicly documented AI security
incidents from the [MITRE ATLAS](https://atlas.mitre.org) case-study catalog. Each one also doubles
as a test of the framework: if an incident does not land cleanly in one of the five dimensions, its
validity note says so.

| Case | ATLAS ID | Era | Dimension that broke | File |
|---|---|---|---|---|
| Bypassing Cylance's AI Malware Detection | AML.CS0003 | 2019, classic ML evasion | Attack Surface, rooted in Training | [cs-evasion-cylance.md](cs-evasion-cylance.md) |
| EchoLeak: Zero-Click Prompt Injection Targeting M365 Copilot | AML.CS0059 | 2025, LLM assistant indirect prompt injection | Context (permission scope versus input trust), exploited through Attack Surface | [cs-injection-echoleak.md](cs-injection-echoleak.md) |
| Poisoned Postmark MCP Server Email Exfiltration | AML.CS0053 | 2025, AI agent tool supply chain | Attack Surface (supply chain), with Context setting the blast radius | [cs-agent-postmark-mcp.md](cs-agent-postmark-mcp.md) |

Technique IDs in every case study are taken verbatim from ATLAS release 2026.10. Earlier ATLAS data
files use different IDs for some steps.

## What the three cases say about the framework

| Finding | Case |
|---|---|
| Explainability features can double as an attacker's oracle. TRACE has no place to record that tension. | Cylance |
| Output handling (what the client does with model output) is not an AI-specific category, but it is where the exfiltration happened. | EchoLeak |
| TRACE has no dimension for the provenance of code an agent executes. T covers training data provenance only. | Postmark MCP |

## Contributing a case study

Use a case from the [ATLAS case-study catalog](https://github.com/mitre-atlas/atlas-data) and follow
the structure of the existing files: title with ATLAS ID; a one-paragraph dated summary linking the
ATLAS entry and the primary report; **What happened** (numbered steps, each tagged with its ATLAS
technique ID exactly as ATLAS gives it); **TRACE walk** (a five-row table: Dimension, what the
defender knew or could have known at the time marked known, partial or unknown, and what the
incident revealed); **Which dimension broke**; **Would TRACE have surfaced it** (quote the TRACE
question, and say so if TRACE would not have caught it); **Validity note** (name anything that did
not fit; do not force a fit); and **Sources** with a HIGH or MED confidence tag per line. Every
factual claim must trace to the ATLAS entry or a linked public report. If a detail cannot be
verified, leave it out or mark it unverified.

*Part of the [TRACE Framework](../../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*

