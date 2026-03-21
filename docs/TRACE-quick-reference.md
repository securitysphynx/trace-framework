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

*Ask:* What is the false positive rate? The false negative rate? What happens when the data distribution shifts? Is there drift monitoring? What does "99% accurate" actually mean given the base rate?

---

## A — Attack Surface

How could an adversary manipulate this? What is the AI-specific threat model?

*Ask:* Can an attacker craft inputs to evade detection? Can they poison the training pipeline? If this is an LLM, is it vulnerable to prompt injection? What supply chain risks exist in the model or its dependencies?

*Reference:* MITRE ATLAS (atlas.mitre.org), OWASP Top 10 for LLM Applications (genai.owasp.org)

---

## C — Context

Does this fit my environment, my data, my team, and my workflow?

*Ask:* Was this built for an environment like mine? Does it handle the edge cases I see (multi-language, legacy systems, hybrid networks)? Does my team have the capacity to handle the output? What's the fallback if this tool is unavailable?

---

## E — Explainability

Can I understand and justify this system's decisions to my team, my leadership, and my regulators?

*Ask:* When this flags something, can the analyst see why? Can I reconstruct the decision for an incident report? Does this meet audit and regulatory transparency requirements? If I had to defend this decision in a meeting, could I?

---

## Apply TRACE To:

| Situation | Depth | Primary Dimensions |
|-----------|-------|--------------------|
| Acting on an AI-generated alert | 5-minute mental check | R, C, E |
| Evaluating a vendor product | Structured evaluation (hours) | All five |
| Writing AI acceptable use policy | Policy framework | All five |
| Investigating an AI system failure | Forensic diagnostic | T, R, A first; then C, E |
| Triaging a new AI claim or headline | Quick gut check | T, R |

---

TRACE by Melissa Bischoping | CC BY 4.0 | github.com/SecuritySphynx/trace-framework
