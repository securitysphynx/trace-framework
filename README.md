# TRACE

**Trust Reasoning for AI in Cybersecurity Environments**

> *Should I trust this AI system, and can I defend that decision?*

A framework for security professionals who don't build models but are increasingly asked to trust them.

---

## The Five Dimensions

| | Dimension | Core Question |
|:---:|:---|:---|
| **T** | **Training** | What was this trained on? Is the foundation sound? |
| **R** | **Reliability** | How does this fail? How would I know? |
| **A** | **Attack Surface** | How could an adversary manipulate this? |
| **C** | **Context** | Does this fit *my* environment? |
| **E** | **Explainability** | Can I justify this decision to stakeholders? |

---

## T — Training

**What was this system trained on, and is that foundation sound?**

| Evaluate | Why It Matters |
|----------|----------------|
| Data provenance | You can't trust what you can't trace |
| Representativeness | Models learn from data — if your threats aren't in it, neither is detection |
| Freshness | Yesterday's training data misses today's techniques |
| Poisoning risk | Adversaries target training pipelines |

---

## R — Reliability

**How does this system fail, and how would I know?**

| Evaluate | Why It Matters |
|----------|----------------|
| Error rates (FP/FN) | "99% accurate" means nothing without base rates |
| Failure modes | Silent degradation is worse than loud failure |
| Drift monitoring | Models rot — is anyone watching? |
| Threshold sensitivity | Small config changes, big outcome swings |

---

## A — Attack Surface

**How could an adversary manipulate this system?**

| Threat | Example |
|--------|---------|
| Evasion | Crafted inputs that bypass detection |
| Poisoning | Corrupting training data to shape behavior |
| Prompt injection | Hijacking LLM context (direct or indirect) |
| Extraction | Stealing model weights or training data |
| Supply chain | Compromised models, plugins, dependencies |

See: [MITRE ATLAS](https://atlas.mitre.org) | [OWASP Top 10 for LLMs](https://genai.owasp.org/llm-top-10/)

---

## C — Context

**Does this system's design match my operational environment?**

| Evaluate | The Reality |
|----------|-------------|
| Deployment fit | Built for enterprise? You're a 3-person team. |
| Data compatibility | Trained on cloud telemetry? You're hybrid. |
| Team capacity | Can you actually act on the output volume? |
| Edge cases | Your environment has quirks the vendor never saw |

*Context is where most AI deployments fail silently: the model works, just not here.*

---

## E — Explainability

**Can I understand and justify this system's decisions?**

| Scenario | The Question |
|----------|--------------|
| AI quarantines an email | Why? Can the analyst see the reasoning? |
| AI flags an employee | Can you explain this to HR and legal? |
| AI misses an attack | Can you reconstruct what it saw? |
| Audit time | Is there a trail? |

*If you can't explain it, it's a liability regardless of accuracy.*

---

## Agentic AI

AI agents — systems that take autonomous actions — require deeper evaluation.

The five dimensions still apply, but agents add:
- **Action logging** — What did it do?
- **Attribution** — AI action vs. human action?
- **Permission scope** — What *could* it access?
- **Rollback** — Can you undo it?

See **[TRACE for Agentic AI](guides/TRACE-agentic-ai.md)** for forensics, logging requirements, and vendor questions.

---

## Apply TRACE

| Situation | Time | Focus |
|-----------|------|-------|
| Acting on an AI alert | 5 min | R, C, E — Can I trust this output? |
| Evaluating a vendor | Hours | All five — Structured assessment |
| Writing AI policy | Days | All five — Map to requirements |
| Investigating an AI failure | Varies | Diagnostic — Which dimension broke? |
| Investigating adversary AI use | Varies | A, E — What happened? What's the evidence? |

---

## What TRACE Is Not

| | |
|---|---|
| Not a maturity model | No levels to achieve |
| Not a certification | No audit |
| Not a replacement for ATLAS/RMF/OWASP | TRACE tells you *when* to reach for those |
| Not a scorecard | No numbers — better questions |

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
