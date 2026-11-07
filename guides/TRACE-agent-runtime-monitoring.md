# TRACE for Agent Runtime Monitoring

A supplement to the [TRACE framework](../README.md) for monitoring AI agents *as they operate* — detection-side controls that complement the forensic and logging guidance in [TRACE for Agentic AI](TRACE-agentic-ai.md).

## Why a Separate Guide

The agentic AI supplement focuses on **forensics**: what should be logged, how logs should be structured, how tamper-evidence works, what artifacts an investigator needs after the fact.

This guide focuses on **detection**: how to monitor an agent in real time, identify when it's been manipulated, and surface anomalous behavior before it becomes an incident.

The two are complementary. Forensics tells you *what happened*. Detection tells you *something is happening*. Mature agent security needs both.

---

## The Threat Model: Social Engineering at the Endpoint

Prompt injection, jailbreaking, and indirect manipulation of AI agents are best understood as **social engineering attacks against a non-human endpoint**. The attacker constructs inputs designed to convince the model to act against the principal's interest. The model is the mark.

What's familiar from the human side:
- The attack vector is persuasion via crafted content
- The exploit succeeds when the target deviates from intended behavior under influence
- Defenses at the principal level (training, awareness, skepticism) are necessary but never sufficient

What changes when the principal is a model:

| Defense Affordance | Human Targets | LLM Agents |
|---|---|---|
| Post-incident self-report ("I felt weird about that call") | Yes | No. No persistent self across sessions. |
| Behavioral baseline observable over time | Yes — UEBA, identity, access patterns | Weak. General-purpose agents have no stable baseline. |
| Internal alarms (gut feeling, hesitation) | Yes | No. Chain-of-thought is not faithful — emitted reasoning does not reliably reflect actual computation. |
| Friction / time delay before high-impact action | Yes — sleep on it, talk to colleague | No. Agents act on the timescale of inference. |
| Out-of-band verification (call back, ask coworker) | Yes | Only if explicitly architected into the system. |
| Forensic witness post-incident | Yes — the human remembers | No — only the I/O log persists; the model has no continuity. |

*The asymmetry that defines the problem: with humans, the victim is itself a forensic asset. With LLMs, the victim preserves no evidence of having been compromised. The only forensic record is what was logged externally. This is the gap detection has to fill.*

---

## The Missing Witness Problem

Traditional social engineering investigations begin with the human victim. They remember the call, preserve the email, recall what felt off. The witness is unreliable but present.

Agent compromise has no witness. Each session starts with no memory of the last (unless persistence is explicitly architected, which is itself an attack surface). The model cannot reliably introspect on whether it was deceived. Self-reports from the model about *why* it took an action are themselves susceptible to having been part of the manipulation, or to post-hoc rationalization that doesn't reflect actual processing.

**Implication for monitoring:** if the agent can't be a witness against itself, the monitoring system has to *be* the witness. Every input, every action, every output, every approval gate, every anomaly — these have to be captured externally, because there is no internal observer to fall back on.

---

## Detection by Pipeline Location

Defenses sort cleanly by where in the agent pipeline they intervene. A working monitoring stack should layer multiple of these — none is sufficient alone.

### Input Boundary

**Provenance tagging.** Each chunk of input is tagged by source: user-typed, tool A response, tool B response, retrieved document, memory recall. The tags travel with the content through the model and into the audit log. After-the-fact question: "what content from what source preceded this anomalous action?"

**Injection-detection classifiers.** Smaller, separate models scan tool and retrieved content for instruction-like patterns. Pattern-matching catches the trivial attacks ("ignore previous instructions"); embedding-based classification catches medium-effort variants. Sophisticated, content-aware injection still gets through. Vendors in this space as of April 2026 included Lakera Guard, Microsoft Prompt Shield, NVIDIA NeMo Guardrails and Protect AI; the list churns, the limitation does not.

**Spotlighting / data-marking.** Untrusted content is wrapped with deliberate boundaries the model is trained to respect. Improves the model's ability to distinguish *data being read* from *instruction being received*. Anthropic and others have published technique variants.

*Red flag: tool/retrieved content enters the model context with the same trust level as user input.*

### Action Boundary

**Capability minimization.** The single highest-leverage defense. The agent cannot misuse a tool it does not have. Every tool added is attack surface. Violated constantly in practice.

**Action-tier authorization.** Tools are classified by impact tier:
- Reads → log only
- Writes → log plus sample audit
- Sends, spends, deletes → explicit human consent in real time, with the user seeing the actual proposed action

The user sees what the agent intends to do *before* it happens. Compare to standard OS privilege models: the kernel doesn't trust user-mode applications to be benign, and doesn't have to.

**Cooling-off windows and rate limits.** An agent calling `send_email` two hundred times in a minute is not normal. Threshold-based circuit breakers catch automated-exploit scenarios where the attacker uses the agent as a force multiplier.

*Red flag: agent has standing authority to take irreversible actions without per-action approval gates.*

### Output Boundary

**Output-side DLP.** A classifier sits between model output and any sensitive sink. Pattern-matches for credentials, PII, internal data, suspicious URLs. This is regular DLP technology, applied to a new egress point.

**Honeytokens / canaries.** Plant decoy strings — fake API keys, fake customer records, fake passwords — in places the agent can read but should never need to output. Appearance in any output channel = confirmed exfil, with high signal and low false-positive rate. Borrowed directly from insider-threat detection. Cheap to deploy, hard for an attacker to know they're there.

*Red flag: model outputs go to sensitive destinations (databases, email, API endpoints) without an intervening filter.*

### Decision Boundary

**Model-as-judge.** A second model — narrower scope, ideally different training distribution — evaluates each proposed action against stated user intent. *"Given the user said X, is calling tool Y with these arguments consistent with that intent?"* Anthropic's Constitutional Classifiers, OpenAI's content filtering, and most enterprise agent platforms do versions of this.

| Strength | Weakness |
|----------|----------|
| Attacker has to compromise both the working model and the judge | The judge is also persuadable, often through similar attack vectors |
| Judge can have a narrower context window and be cheaper | Judge sees the same potentially-poisoned input |
| Different training distribution = different inductive biases | Judge model may inherit shared training-data vulnerabilities |

**Multi-model voting.** Ensemble of judges with different lineage must agree before an action proceeds. Costly at scale; appropriate for highest-impact action tiers.

*Red flag: high-impact actions are gated only by the working agent's own self-assessment.*

### Behavioral Boundary — UEBA for Agents

**Action-sequence baselining.** Profile the agent's normal tool-call distribution, sequence patterns, content topics. Statistical deviation is the signal.

| Agent Type | Baselining Difficulty |
|---|---|
| Narrow agent (customer support bot, IT runbook) | Tractable. Normal behavior is constrained. |
| Domain agent (security analyst, support engineer) | Workable with sufficient observation period. |
| General-purpose agent (assistant, copilot) | Hard. Baseline is "do whatever the user asks," which has no stable shape. |

**Cross-session correlation.** Same agent identity, multiple sessions — are actions consistent with the user's profile and history? The closest direct translation of insider threat detection patterns: legitimate credentials, plausible-looking actions, sudden shift in behavior.

*Red flag: no behavioral baseline exists; every session is treated as independent and unjudgable.*

### Model-Internals Boundary (Research Grade)

**Interpretability-based monitoring.** Sparse autoencoders, feature-activation analysis, circuit-level tracing. The goal: detect when "comply with adversarial instruction," "deceive user," or similar internal features fire. Anthropic and OpenAI both have ongoing programs.

Status: not real-time, not cheap, not deployable as a SIEM rule today. Significance: the only technique on this list that even tries to monitor the manipulation itself rather than its inputs and outputs. This is where the next leap forward in agent security will come from.

---

## The Three Monitoring Layers

Detection capability today sorts into three maturity layers:

| Layer | What It Sees | Status | Coverage Gap |
|---|---|---|---|
| **Action-level** | What the agent did — tool calls, file accesses, API requests, outputs | Mature. EDR-grade telemetry, OS hooks, network capture. | Sees actions, not intent. Cannot tell legitimate from manipulated when the action itself looks plausible. |
| **Behavioral-trace** *(emerging)* | Probable decision pathways reconstructed from I/O history plus model-as-judge analysis | Emerging. No vendor owns this layer at scale. | Best-effort. Inferential, not direct. |
| **Decision-level** | The agent's actual reasoning process — was it persuaded, was deception suppressed, did manipulation succeed | Research-grade. Not deployable. | Currently invisible at production scale. |

The honest position: most production agent deployments today operate with *only the action-level layer*. Sophisticated deployments add behavioral-trace inference. Decision-level monitoring is a research frontier.

The strategic gap is the middle layer. Action-level monitoring is mature. Decision-level monitoring is research-grade. Behavioral-trace monitoring — using the agent's I/O history plus model-as-judge analysis to reconstruct probable decision pathways and flag suspicious ones — is unclaimed territory in the vendor landscape, and it's the layer that will define enterprise-grade agent security over the next 24 months (prediction dated April 2026; score it in April 2028).

---

## What a Working Monitoring Stack Looks Like

Practitioner-grade monitoring for an agent in production:

1. **Centralized session logging** with full I/O capture and provenance tagging *(see TRACE-agentic-ai for log schema)*
2. **Pre-execution injection detection** on retrieved and tool-channel content
3. **Tool-call gating by impact tier**, with human-in-the-loop for high-impact actions
4. **Post-execution output filtering** for sensitive patterns and honeytoken hits
5. **Behavioral baselining** with deviation alerting
6. **Asynchronous model-as-judge layer** doing semantic intent-consistency checks
7. **Continuous red-team evaluation** running in parallel — robustness as a tracked metric, not a state
8. **Incident response playbook** including session replay and counterfactual analysis ("re-run the session without the suspect content; does behavior diverge?")

The defensive posture is the one already understood from human-side defense: *assume compromise is possible at any session, minimize blast radius of any single agent action, require out-of-band confirmation for high-impact decisions, instrument the I/O exhaustively.* Least privilege, separation of duties, two-person integrity, forensic logging — translated into machine-trust terms.

---

## Questions to Ask About Runtime Monitoring

Add these to vendor assessments and internal architecture reviews:

1. Is provenance tagged on input content, distinguishing user-typed from tool/retrieved?
2. Is there an injection-detection layer on tool/retrieved content before it enters model context?
3. Are tools classified by impact tier, and what's the gating model for each tier?
4. Is there an output-side filter independent of model compliance?
5. Are honeytokens deployed in agent-accessible data sources?
6. Is there a model-as-judge or co-monitoring layer evaluating proposed actions?
7. What behavioral baseline exists for this agent? How is deviation detected?
8. What happens when the monitoring layer flags something? Block? Alert? Human review?
9. Can the agent's monitoring be disabled by the agent itself, or by the operator?
10. How are monitoring failures detected? (Who watches the watchers?)

---

## Red Flags Checklist

Warning signs that an agent deployment lacks meaningful runtime monitoring:

- [ ] No distinction between user input and tool/retrieved content in the model context
- [ ] No injection-detection layer; tool output enters the model raw
- [ ] All tools granted equal authority; no tier-based gating
- [ ] High-impact actions (send, spend, delete) execute without per-action human approval
- [ ] Outputs go to sensitive destinations with no intervening filter
- [ ] No honeytokens in agent-accessible data
- [ ] No secondary model evaluating proposed actions
- [ ] No behavioral baseline; every session evaluated in isolation
- [ ] Monitoring layer can be disabled by the agent or its operator
- [ ] No defined process for what happens when monitoring fires
- [ ] "The model is trained to refuse adversarial instructions" cited as the primary defense

---

## What Monitoring Cannot Do

A few honest limits worth stating explicitly:

**Detection is not prevention.** Even a well-instrumented stack will miss novel attacks. Monitoring lowers the failure rate; it doesn't reach zero. Plan for compromise as a possible outcome, not a hypothetical.

**Chain-of-thought is not a monitoring artifact.** Published research shows the reasoning a model emits does not reliably reflect the actual computation that produced its output. Treating CoT traces as evidence of intent is a mistake. They are *output*, not *reasoning*.

**Model-as-judge is persuadable.** Adding a second model raises attack cost but does not eliminate the threat. Sophisticated indirect injection can target the judge as well as the working agent. The defense compounds; it does not multiply.

**Standardized agent telemetry does not yet exist at scale.** Until there is a *Sysmon-for-agents* — a universal event schema for tool-call-attempted, input-classified, output-filtered, and the rest — detection ecosystems will fragment along vendor lines. This is the field-shaping problem of the next 24 months (from April 2026). Candidates have since appeared: OCSF's `ai_operation` profile with the 1.9.0 `delegation` attribute, OpenTelemetry's GenAI conventions (still unstable), and OWASP's Agent Control Standard hooks. None is deployed at scale yet.

---

## Summary

Agent runtime monitoring is **insider threat program design translated into machine-trust terms**. The threat actor has legitimate credentials, takes plausible-looking actions, and operates from a trusted seat. The defenses — least privilege, separation of duties, behavioral baselining, two-person integrity, exhaustive I/O instrumentation — translate cleanly. The new wrinkle is the missing witness: the agent has no conscience to backstop the controls, no post-incident memory, no internal alarm. The monitoring system has to *be* the witness, because nothing else can.

| Detection Layer | Coverage |
|---|---|
| What the agent did | Mature — instrument like an endpoint |
| Why the agent did it | Research-grade — interpretability is the frontier |
| Whether the agent was manipulated | Emerging — model-as-judge plus behavioral-trace inference |

Build like the model will eventually be compromised. In any given session, there is a non-zero probability it already has been. The monitoring stack is what tells you which session that was.

---

## References

- [TRACE for Agentic AI](TRACE-agentic-ai.md) — companion guide focused on forensics, logging, and policy enforcement
- [MITRE ATLAS](https://atlas.mitre.org) — adversarial threat techniques for AI systems; the agent-authority mitigations M0026–M0032 (access management, human-in-the-loop, restricting tool invocation on untrusted data, memory hardening, segmentation) are the control-side counterpart to the detections in this guide
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) — 2026 edition (ASI01–ASI10); the social-engineering-at-the-endpoint model here is ASI01 Goal Hijack, ASI06 Memory and Context Poisoning and ASI09 Human-Agent Trust Exploitation
- [OCSF](https://schema.ocsf.io/) — Open Cybersecurity Schema Framework; `ai_operation` profile (v1.8.0) plus `ai_agent` object and `delegation` attribute (v1.9.0) for attributing an action to the authority it ran under
- [OpenTelemetry GenAI Semantic Conventions](https://github.com/open-telemetry/semantic-conventions-genai) — dedicated repository; not yet marked Stable, attribute names still changing
- [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework)
- [NIST IR 8596, Cyber AI Profile](https://csrc.nist.gov/pubs/ir/8596/iprd) — initial preliminary draft, December 2025; maps AI security outcomes onto CSF 2.0 Detect and Respond functions

---

*This guide is part of the TRACE framework by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Last updated: November 2026.*
