# TRACE Quick Reference Card

**Trust Reasoning for AI in Cybersecurity Environments**

When you encounter an AI system, an AI-generated output, or an AI-related claim, run TRACE.

---

## T — Training

What was this trained on? Is the data representative, recent, and uncompromised?

*Ask:* Where did the training data come from? How old is it? Could it have been poisoned or biased? Does the vendor disclose their data sources?

---

## R — Reliability

How does this fail? What are the error rates? How would I detect silent degradation?

*Ask:* What is the false positive rate? The false negative rate? What happens when the data distribution shifts? Is there drift monitoring? What does "99% accurate" actually mean given the base rate? (1M events, 10 real, 99% accurate: ~10,000 false alarms a day, and roughly 1 alert in 1,000 is real.)

---

## A — Attack Surface

How could an adversary manipulate this? What is the AI-specific threat model?

*Ask:* Can an attacker craft inputs to evade detection? Can they poison the training pipeline? If this is an LLM, does the design remove a leg of the lethal trifecta (private data, untrusted content, outbound channel), and which design pattern constrains injected text? What supply chain risks exist in the model or its dependencies?

*Reference:* MITRE ATLAS (atlas.mitre.org), OWASP Top 10 for LLM Applications and OWASP Top 10 for Agentic Applications (genai.owasp.org)

---

## C — Context

Does this fit my environment, my data, my team, and my workflow?

*Ask:* Was this built for an environment like mine? Does it handle the edge cases I see (multi-language, legacy systems, hybrid networks)? Does my team have the capacity to handle the output? What's the fallback if this tool is unavailable?

---

## E — Explainability

Can I understand and justify this system's decisions to my team, my leadership, and my regulators? Three parts: **Evidence** (what happened, reconstructable), **Explanation** (why; for LLMs usually a rationalization, not a record), **Justification** (defensible from evidence alone).

*Ask:* When this flags something, can the analyst see what it saw and did? Can I reconstruct the decision for an incident report without relying on the model's own narrative? Does this meet audit and regulatory transparency requirements? If I had to defend this decision in a meeting, could I?

---

## When the Answer Is "Unknown"

For closed models, T is usually unknowable. Do not stop there: behavioral evidence substitutes for provenance. Test R and A on your own data, at your base rates. T unknown + strong R evidence on your data = accept with conditions. T unknown + vendor metrics only = needs more information. T and R both unknown = no basis to calibrate. Write "unknown" in the knowledge map; it is a finding.

---

## Apply TRACE To:

| Situation | Depth | Primary Dimensions |
|-----------|-------|--------------------|
| Acting on an AI-generated alert | 5-minute mental check | R, C, E |
| Evaluating a vendor product | Structured evaluation (hours) | All five |
| Writing AI acceptable use policy | Policy framework | All five |
| Investigating an AI system failure | Forensic diagnostic | T, R, A first; then C, E |
| Triaging a new AI claim or headline | Quick gut check | T, R |
| Evaluating an AI agent deployment | Structured evaluation | All five + [Agent Supplement](../guides/TRACE-agentic-ai.md) |
| Auditing an agent's monitoring before an incident | Structured evaluation | R, A, E + [Runtime Monitoring](../guides/TRACE-agent-runtime-monitoring.md) |

---

TRACE by Melissa Bischoping | CC BY 4.0 | github.com/SecuritySphynx/trace-framework
