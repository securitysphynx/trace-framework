# TRACE Framework Crosswalk

Where each TRACE dimension lands in the frameworks your GRC, SOC and engineering teams already report against. TRACE is not a replacement for any of these; this table is how you know which one to open next.

**Scope note:** mappings are at the function or category level and are indicative, not authoritative. Refine them with your own GRC team against the current text of each framework. Version anchors are as of November 2026.

---

## The crosswalk

| TRACE | NIST AI RMF 1.0 (function) | NIST CSF 2.0 via Cyber AI Profile (IR 8596, draft) | MITRE ATLAS (tactics, techniques, mitigations) | OWASP LLM Top 10 (2025) | OWASP Agentic Top 10 (2026) |
|---|---|---|---|---|---|
| **T — Training** | MAP (context, data provenance); MEASURE (data quality, representativeness) | IDENTIFY (ID.AM asset inventory incl. models and data; ID.RA supply-chain risk); GOVERN (GV.SC supply chain) | Training Data Poisoning (AML.T0020); Supply Chain Compromise (AML.T0010), including AI Agent Tool (AML.T0010.005); Publish Poisoned AI Artifacts (AML.T0115) | LLM03 Supply Chain; LLM04 Data and Model Poisoning | ASI04 Agentic Supply Chain Vulnerabilities |
| **R — Reliability** | MEASURE (performance, error rates, drift); MANAGE (monitoring, response to degradation) | DETECT (DE.CM continuous monitoring, DE.AE adverse-event analysis); IDENTIFY (ID.IM improvement) | Impact: Erode AI Model Integrity (AML.T0031); Evade AI Model (AML.T0015) as the adversarial driver of false negatives | LLM09 Misinformation; LLM10 Unbounded Consumption | ASI08 Cascading Failures |
| **A — Attack Surface** | MAP (threat identification); MEASURE (security and resilience testing); MANAGE (risk treatment) | PROTECT (PR.AA identity and access, PR.PS platform security, PR.DS data security); DETECT (DE.CM) | The whole matrix, Reconnaissance through Impact. For agents: AI Agent Tool Poisoning (AML.T0110), Publish Poisoned AI Artifacts: AI Agent Tools (AML.T0115.002), AI Agent Tool Invocation (AML.T0053); mitigations M0026–M0032 | LLM01 Prompt Injection; LLM02 Sensitive Information Disclosure; LLM05 Improper Output Handling; LLM06 Excessive Agency; LLM07 System Prompt Leakage; LLM08 Vector and Embedding Weaknesses | ASI01 Agent Goal Hijack; ASI02 Tool Misuse; ASI03 Identity and Privilege Abuse; ASI05 Unexpected Code Execution; ASI06 Memory and Context Poisoning; ASI07 Insecure Inter-Agent Communication; ASI10 Rogue Agents |
| **C — Context** | GOVERN (policies, roles, risk tolerance); MAP (intended use, deployment context, impacted parties) | GOVERN (GV.OC organizational context, GV.RR roles and responsibilities, GV.RM risk management strategy); PROTECT (PR.AA least privilege for agents) | Mitigations: Manage AI Agent Service Access (M0026), Manage User-Delegated AI Agent Access (M0027), Enforce Access Controls for AI Agent Tools (M0028), Human In-the-Loop for AI Agent Actions (M0029) | LLM06 Excessive Agency | ASI03 Identity and Privilege Abuse; ASI09 Human-Agent Trust Exploitation |
| **E — Explainability** (Evidence / Explanation / Justification) | GOVERN (transparency and accountability); MEASURE (explainability and interpretability measures, logging) | DETECT (DE.AE event analysis); RESPOND (RS.AN incident analysis, RS.MA incident management); GOVERN (GV.OV oversight) | Case-study catalog (what evidence investigators had); no direct technique mapping. Evidence layer standards: OCSF `ai_operation` profile and `delegation` attribute; OpenTelemetry GenAI conventions (not yet Stable) | No direct entry. LLM09 Misinformation is the closest for Justification | ASI10 Rogue Agents (detection and attribution); ASI09 for Justification to human principals |

---

## How to read it

- **A GRC reader** starts from the CSF column. The Cyber AI Profile (NIST IR 8596) is the document that lets AI-specific outcomes sit inside the CSF 2.0 structure you already report against; it is still a preliminary draft, so cite it as direction, not requirement.
- **A SOC or detection engineer** starts from the ATLAS column. TRACE-A tells you to open the matrix; ATLAS tells you which techniques and which mitigations. The agent mitigations M0026–M0032 are, in effect, a control checklist for TRACE-C and the permission-scope questions in the [Agentic AI supplement](../guides/TRACE-agentic-ai.md).
- **An AppSec reader** starts from the OWASP columns. Use the IDs in requirements documents (see the `trace-vendor` skill's REQ-A-004) so vendors and auditors are answering the same question.
- **An auditor or investigator** starts from the E row. Note that the frameworks are thinnest here: none of them tells you what a complete forensic record for an AI agent looks like. The [Agentic AI supplement](../guides/TRACE-agentic-ai.md) proposes one.

---

## What is not in this table

- **ISO/IEC 42001** (AI management systems) maps to TRACE-C and TRACE-E at the management-system level; it is a certification standard and TRACE is not, so the mapping is one-way.
- **EU AI Act** high-risk obligations (Articles 9–15) map onto T (data governance, Art. 10), R (accuracy and robustness, Art. 15), A (cybersecurity, Art. 15), C (human oversight, Art. 14) and E (logging and transparency, Arts. 12–13). Their application dates were deferred by Regulation (EU) 2026/1744 to December 2027 (Annex III) and August 2028 (Annex I). Treat them as the direction of travel for E.
- **OWASP Agent Control Standard (ACS)** and the **NIST NCCoE agent identity and authorization** work define the runtime-control and identity layers that TRACE's agent questions evaluate. They are specifications, not risk frameworks, so they appear in the supplements rather than here.

---

## Version anchors

| Framework | Version used | Date |
|---|---|---|
| NIST AI RMF | 1.0, with Generative AI Profile (NIST AI 600-1) | 2023 / 2024 |
| NIST CSF | 2.0 | February 2024 |
| NIST Cyber AI Profile | IR 8596, initial preliminary draft | December 16, 2025 |
| MITRE ATLAS | content release 2026.10 | October 28, 2026 |
| OWASP Top 10 for LLM Applications | 2025 edition | 2025 |
| OWASP Top 10 for Agentic Applications | 2026 edition | December 9, 2025 |
| OCSF | 1.9.0 | August 3, 2026 |

---

*Part of the [TRACE Framework](../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Last updated: November 2026.*
