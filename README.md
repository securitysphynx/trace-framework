# TRACE: Trust Reasoning for AI in Cybersecurity Environments

TRACE is a lightweight evaluation framework for cybersecurity professionals who need to make informed decisions about AI systems, AI-generated output, and AI-related claims in their daily work.

It was built for the people who don't build models but are increasingly asked to trust them: SOC analysts acting on AI-generated alerts, security engineers evaluating AI-powered products, GRC teams writing AI policy, and incident responders investigating AI system failures.

## Status

**Work in progress.** TRACE is under active development. The core framework (five evaluation dimensions) is stable. Supporting materials, use-case guides, and practitioner resources are being developed. Contributions, feedback, and real-world testing are welcome.

## The Problem

Security professionals are adopting AI tools faster than they're developing the judgment to evaluate them. We have mature methodologies for evaluating firewall rules, triaging vulnerabilities, and assessing organizational risk. We don't have an equivalent for the question practitioners face every day: *should I trust this AI system, and can I defend that decision?*

Vendor marketing says "trust us." Academic literature on adversarial ML is rigorous but inaccessible to most practitioners. The gap between those two poles is where real security decisions get made, and right now people are making them without a structured approach.

TRACE fills that gap.

## The Framework

TRACE provides five evaluation dimensions. They apply to any AI system, any AI-generated output, or any AI-related claim a security professional encounters.

### T — Training

**Core question:** What was this system trained on, and is that foundation sound?

What you're evaluating: data provenance, representativeness, freshness, labeling quality, bias, poisoning risk, and data supply chain integrity. A model is only as trustworthy as the data it learned from. If you don't know what that data is, you don't know what the model actually learned.

### R — Reliability

**Core question:** How does this system fail, and how would I know?

What you're evaluating: error rates (precision, recall, false positive/negative tradeoffs), failure modes, silent degradation, data drift, concept drift, threshold sensitivity, and base rate performance. A model that is 99% accurate can still flood your SOC with false positives if the base rate of real attacks is low enough. Reliability is not about accuracy numbers; it's about whether the failure modes are acceptable for your operational context.

### A — Attack Surface

**Core question:** How could an adversary manipulate this system?

What you're evaluating: evasion attacks, data poisoning, prompt injection (direct and indirect), model extraction, training data extraction, supply chain compromise, and adversarial inputs. AI systems are not just tools; they are attack surfaces. An adversary who understands your detection model can craft inputs that bypass it. An adversary who can influence your training data can shape what the model learns. Security professionals must threat-model AI systems with the same rigor they apply to any other infrastructure.

Relevant frameworks: [MITRE ATLAS](https://atlas.mitre.org) (adversarial threat landscape for AI systems), [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) (2025 edition).

### C — Context

**Core question:** Does this system's design match my operational environment?

What you're evaluating: deployment fit, data compatibility, workflow integration, team capacity, edge cases specific to your environment, and human-in-the-loop requirements. A model trained on one network does not necessarily generalize to yours. A tool designed for a 50-person SOC may not work for a team of 3. Context is where most AI deployments fail silently: the model works, just not here.

### E — Explainability

**Core question:** Can I understand and justify this system's decisions?

What you're evaluating: output transparency, audit trails, the ability to explain decisions to analysts, leadership, legal teams, and regulators. If an AI system quarantines a legitimate email, blocks a production connection, or flags an employee for investigation, someone has to explain why. If you can't, the system is a liability regardless of its accuracy.

### Agentic AI: Beyond the Five Dimensions

The five TRACE dimensions apply to any AI system. But AI agents — systems that take autonomous actions like writing files, calling APIs, or executing commands — create additional evaluation requirements that traditional AI doesn't.

When evaluating agents, TRACE still applies, but you need deeper treatment of forensic and operational concerns: action logging, attribution, session boundaries, permission scope, and rollback capabilities.

See **[TRACE for Agentic AI](guides/TRACE-agentic-ai.md)** for the full supplement.

## How to Use TRACE

TRACE scales to the situation. It is not a compliance checklist or a formal assessment methodology. It is a structured way of thinking that adapts to the decision in front of you.

**Quick evaluation (5 minutes).** An AI system generates an alert. Before you act on it, mentally run TRACE: Do I know what this model was trained on? Do I understand its error rate? Could an attacker have influenced the input? Does this fit my environment? Can I explain my action if asked? If any answer is "no" or "I don't know," that informs how much weight you give the output.

**Product evaluation (hours to days).** Your organization is evaluating an AI-powered security tool. TRACE structures the evaluation: request training data documentation (T), ask for performance metrics including false positive/negative rates and drift monitoring (R), threat-model the AI components using ATLAS (A), assess fit for your specific environment and team (C), evaluate whether the tool's decisions are auditable and explainable (E).

**Policy development.** Your GRC team is writing an AI acceptable use policy. TRACE provides the five categories of risk that the policy must address. Each dimension maps to policy requirements: data governance (T), performance monitoring and SLAs (R), security controls for AI systems (A), deployment approval criteria (C), and transparency and audit requirements (E).

**Incident investigation.** An AI system missed an attack, flagged innocent users, or produced unexpected output. TRACE becomes a diagnostic framework: Was the training data compromised or unrepresentative (T)? Did the model's reliability degrade (R)? Was the system subject to adversarial manipulation (A)? Was there a context mismatch between the model's design and the deployment (C)? Can you reconstruct what the system did and explain it to stakeholders (E)?

## What TRACE Is Not

- Not a maturity model. There are no levels to achieve.
- Not a certification framework. There is no audit.
- Not a replacement for MITRE ATLAS, NIST AI RMF, or OWASP Top 10 for LLMs. Those are detailed, domain-specific resources. TRACE is the mental model that helps you know when to reach for them and what questions to ask when you get there.
- Not a vendor evaluation scorecard. It does not produce a numerical score. It produces better questions.

## Resources

| Resource | Purpose |
|----------|---------|
| [Quick Reference Card](docs/TRACE-quick-reference.md) | One-page TRACE summary |
| [Agentic AI Supplement](guides/TRACE-agentic-ai.md) | Extended evaluation guidance for AI agents |
| [Skills for AI Assistants](skills/) | Prompts to apply TRACE with Claude, Cursor, and other AI tools |

## Project Roadmap

- [x] Core framework definition (five dimensions, use cases)
- [x] TRACE Quick Reference Card
- [x] Agentic AI supplement (forensics, logging, policy enforcement)
- [x] AI assistant skills for practical application
- [ ] Use-case guides with worked examples (SOC, IR, GRC)
- [ ] Practitioner field-testing and iteration

## Contributing

TRACE is a community project. If you're a security practitioner applying TRACE in your work, encountering edge cases, or finding gaps, open an issue or start a discussion. Real-world feedback from people using this in operational environments is more valuable than theoretical refinement.

## Author

[Melissa Bischoping](https://www.linkedin.com/in/mbischoping/) — Security research leader, speaker, and SANS Technology Institute Board member. TRACE grew out of work on AI literacy for cybersecurity professionals, driven by the observation that our industry is adopting AI faster than it's developing the judgment to evaluate it.

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You are free to share and adapt this material for any purpose, including commercial, as long as you give appropriate credit.
