# TRACE

**Trust Reasoning for AI in Cybersecurity Environments**

> *Should I trust this AI system, and can I defend that decision?*

A framework for security professionals who don't build models but are increasingly asked to trust them.

---

## Why This Framework Exists

AI behavioral controls are not security controls.

Every "rule" given to an AI system — through training, alignment, or system prompts — is a *probabilistic norm*, not a hard interlock. There is no kernel-level enforcement preventing a model from violating its instructions. Instructions shape the probability distribution of model output; they don't gate it. Compliance with stated behavior sits on a gradient:

| Layer | Enforcement | Example |
|---|---|---|
| Trained-in values | Strongest. Baked into weights. | Refusal of severe harms. |
| Constitutional / RLHF behaviors | Stronger than prompts; still probabilistic. Jailbreak research keeps finding the edges. | Honesty, source-attribution, uncertainty acknowledgment. |
| System prompt instructions | Moderate. Reliable in most cases, breakable adversarially. | "Never save these things to memory." "Don't guess URLs." |
| Style preferences | Weak. Trivially overridable. | "No emojis." "Tight responses." |

*If your defense relies on the model agreeing to follow a rule, you have not built a security control. You have built a strong suggestion. Your actual security boundary lives elsewhere.*

TRACE exists because *somewhere else* needs to be defined, evaluated, and defensible. The five dimensions are the questions that survive when the model's compliance does not.

---

## What TRACE Is

TRACE is a method for **calibrating reliance** on an AI system — not a yes/no gate.

The human-factors literature has studied trust in automation for decades. Lee and See (2004) define *appropriate* trust by three properties: **calibration** (your trust matches the system's actual capability), **resolution** (your trust distinguishes between the things it does well and the things it does badly), and **specificity** (your trust tracks the right component at the right time). Most AI failures in security operations are calibration failures in one of two directions:

| Failure | What it looks like | Cost |
|---|---|---|
| **Over-trust** (automation bias) | Acting on the alert because the AI said so; skipping the verification step the tool was supposed to speed up | Missed attacks, bad HR decisions, unexplained quarantines |
| **Under-trust** (disuse) | Switching off a working detector after one bad week; routing around the tool | Paying for capability you don't use; analysts drowning in the volume the tool was bought to absorb |

TRACE's five dimensions are the questions whose answers move your trust toward where it should be. The output is not a score. It is a **knowledge map**: for each dimension, what you know, what you partly know, and what you don't — and therefore how much weight the system's output can bear.

*Reference: Lee, J. D., & See, K. A. (2004). Trust in automation: Designing for appropriate reliance. Human Factors, 46(1), 50–80.*

---

## The Five Dimensions

| | Dimension | Core Question |
|:---:|:---|:---|
| **T** | **Training** | What was this trained on? Is the foundation sound? |
| **R** | **Reliability** | How does this fail? How would I know? |
| **A** | **Attack Surface** | How could an adversary manipulate this? |
| **C** | **Context** | Does this fit *my* environment? |
| **E** | **Explainability** | Can I justify this decision to stakeholders? |

**One cross-cutting question** applies to every dimension when the system is an LLM or an agent: *which of these controls still holds if the model does exactly what the attacker asks?* If the answer is "none," the control you are evaluating is a behavioral control, and the security boundary is somewhere you have not looked yet.

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

**The base-rate arithmetic**, because every vendor deck skips it:

| | |
|---|---|
| Events per day | 1,000,000 |
| True attacks among them | 10 |
| Vendor's claimed accuracy | 99% |
| False alarms per day (1% of 999,990) | ~10,000 |
| Chance any given alert is real | **~0.1%** |

This is not a new problem. Axelsson showed in 1999 that the false-alarm rate, not the detection rate, is the limiting factor for intrusion detection, and that the rate required for a usable system may be "unattainably low." AI detection inherits the same arithmetic with a new vendor logo. Ask for the false-positive rate at *your* base rate, or treat the accuracy claim as marketing.

*Reference: Axelsson, S. (2000). The base-rate fallacy and the difficulty of intrusion detection. ACM Transactions on Information and System Security, 3(3), 186–205. (Earlier version: ACM CCS 1999.)*

---

## A — Attack Surface

**How could an adversary manipulate this system?**

| Threat | Example |
|--------|---------|
| Evasion | Crafted inputs that bypass detection |
| Poisoning | Corrupting training data to shape behavior |
| Prompt injection | Hijacking LLM context (direct or indirect) |
| Extraction | Stealing model weights or training data |
| Supply chain | Compromised models, plugins, dependencies — including the tools and MCP servers an agent executes |
| Output handling | Model output rendered or fetched by a client (Markdown images, links, HTML) becomes an exfiltration channel; nothing AI-specific about the hole, but the model is what fills it |

**For LLM applications and agents, "is it vulnerable to prompt injection?" is the wrong question** — the answer is always yes. Ask the architectural questions instead:

| Ask | Why |
|---|---|
| Does the design remove a leg of the *lethal trifecta* — access to private data, exposure to untrusted content, a channel to communicate out? | A system with all three is exploitable by design. Removing one leg is a security control; "the model is trained to refuse" is not. |
| Which published design pattern does it implement — Dual LLM (a privileged model acts, a quarantined model reads), code-then-execute (CaMeL), action-selector, plan-then-execute? Or none? | Patterns constrain what injected text can *do*. Filters only constrain what it can *say*. |
| Where does untrusted content enter the model's context, and is it tagged with its source? | Provenance is what lets you answer "what preceded this action?" after the fact. |

See: [MITRE ATLAS](https://atlas.mitre.org) | [OWASP Top 10 for LLMs](https://genai.owasp.org/llm-top-10/) | [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | Willison, "The lethal trifecta" (2025) | Debenedetti et al., "Defeating Prompt Injections by Design" (CaMeL, arXiv:2503.18813) | Beurer-Kellner et al., "Design Patterns for Securing LLM Agents against Prompt Injections" (arXiv:2506.08837)

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

Explainability is three different things, needed by three different audiences. Evaluate them separately:

| | Question | Who needs it | For LLMs |
|---|---|---|---|
| **Evidence** | What happened, in order, with artifacts? Can I reconstruct it? | IR, auditors | Available if logged. This is where the weight sits. |
| **Explanation** | Why did the model produce this output? | Analysts, model owners | Mostly unavailable. A model's stated reasoning is output, not reasoning. |
| **Justification** | Can I defend this decision to someone who is not an engineer? | Leadership, legal, HR, regulators | Built from Evidence, not Explanation. |

| Scenario | The Question |
|----------|--------------|
| AI quarantines an email | What did it see, and what did it do? Can the analyst see the inputs? |
| AI flags an employee | Can you explain this to HR and legal using evidence, not the model's narrative? |
| AI misses an attack | Can you reconstruct what it saw and what it skipped? |
| Audit time | Is there a trail, and does it survive the agent's own access? |

**Explanation is also an oracle.** Whatever the system exposes to help an analyst understand a verdict — scores, confidence, feature weights, verbose logs — is equally available to an attacker tuning inputs against it. Every Explanation feature is an Attack Surface entry; decide who gets to see it. (See the [Cylance case study](guides/case-studies/cs-evasion-cylance.md).)

**On "the reasoning":** for language models, the chain of thought a model emits does not reliably reflect the computation that produced its answer. Treating it as evidence of intent is a mistake; it is a rationalization that may or may not be faithful. Build Justification on Evidence. *(Chen et al., "Reasoning Models Don't Always Say What They Think," 2025, arXiv:2505.05410; Korbak et al., "Chain of Thought Monitorability," 2025, arXiv:2507.11473.)*

**Regulatory anchor (as of November 2026):** the EU AI Act's high-risk logging and transparency duties (Articles 12–13) were deferred by the Digital Omnibus, Regulation (EU) 2026/1744, to December 2, 2027 for stand-alone Annex III systems and August 2, 2028 for Annex I embedded systems. In the US, NIST IR 8596 (Cyber AI Profile, preliminary draft) maps AI logging outcomes onto CSF 2.0. Check both before writing "regulators" into a requirement.

*If you can't explain it, it's a liability regardless of accuracy.*

---

## When the Answer Is Unknown

For most commercial AI products — and every closed frontier model — the honest answer to T is "unknown." Training data provenance, representativeness and poisoning exposure cannot be inspected by the buyer. Applied literally, the pyramid in the [Visual Guide](docs/TRACE-visual-guide.md) would then mark every layer above T as suspect, and TRACE would reject everything. That is not useful, and it is not what the framework means.

**The substitution rule:** when a dimension cannot be inspected, it is inferred from behavior. You cannot audit T; you can test R and A empirically, on your data, at your base rates. Behavioral evidence stands in for provenance you cannot see.

| What you have | What it supports |
|---|---|
| T unknown + strong R evidence **on your own data** (your base rates, your edge cases, your red team) | Accept with conditions. Name the conditions: monitoring, drift checks, re-test cadence. |
| T unknown + R evidence **supplied by the vendor only** | Needs more information. Vendor metrics are a claim, not a measurement. |
| T unknown + R unknown | Reject, or run it as an experiment with no production dependency. You have no basis for calibration. |
| T known and poor (stale, unrepresentative, poisonable) + strong R evidence | Accept with a short re-test cycle. The foundation is weak; watch for the cracks to reach the output. |

"Unknown" is a finding, not a gap in your evaluation. Write it down in the knowledge map. The failure is not having an unknown; it is treating an unknown as a yes.

---

## Agentic AI

AI agents — systems that take autonomous actions — require deeper evaluation.

The five dimensions still apply, but agents add:
- **Action logging** — What did it do?
- **Attribution** — AI action vs. human action?
- **Delegation** — On whose authority? Its own standing permissions, or a named user's?
- **Permission scope** — What *could* it access?
- **Rollback** — Can you undo it?

See **[TRACE for Agentic AI](guides/TRACE-agentic-ai.md)** for forensics, logging requirements, and vendor questions, and **[TRACE for Agent Runtime Monitoring](guides/TRACE-agent-runtime-monitoring.md)** for detection-side controls and the social-engineering-at-the-endpoint threat model.

---

## Apply TRACE

| Situation | Time | Focus |
|-----------|------|-------|
| Acting on an AI alert | 5 min | R, C, E — Can I trust this output? → [SOC guide](guides/use-cases/soc-alert-triage.md) |
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
| [Visual Guide](docs/TRACE-visual-guide.md) | Pyramid diagram for teaching and workshops |
| [Framework Crosswalk](docs/TRACE-crosswalk.md) | Where each TRACE dimension lands in NIST AI RMF, CSF 2.0 (via the Cyber AI Profile), MITRE ATLAS and OWASP |
| [Case Studies](guides/case-studies/) | TRACE applied retrospectively to public incidents from the MITRE ATLAS catalog |
| [Use-Case Guides](guides/use-cases/) | Situation-specific walkthroughs; SOC alert triage published, IR and GRC planned |
| [Agentic AI Supplement](guides/TRACE-agentic-ai.md) | Extended evaluation guidance for AI agents — forensics, logging, policy enforcement |
| [Agent Runtime Monitoring](guides/TRACE-agent-runtime-monitoring.md) | Detection-side controls — social engineering threat model, model-as-judge, behavioral baselining |
| [Skills for AI Assistants](skills/) | Prompts to apply TRACE with Claude, Cursor, and other AI tools |

## Project Roadmap

- [x] Core framework definition (five dimensions, use cases)
- [x] TRACE Quick Reference Card
- [x] Agentic AI supplement (forensics, logging, policy enforcement)
- [x] Agent runtime monitoring supplement (detection, social engineering threat model)
- [x] AI assistant skills for practical application
- [x] Framework crosswalk (NIST AI RMF, CSF 2.0, ATLAS, OWASP)
- [x] Retrospective case studies from the ATLAS catalog
- [x] Use-case guide: SOC alert triage
- [ ] Use-case guides: IR, GRC
- [ ] Practitioner field-testing and iteration

## Contributing

TRACE is a community project. If you're a security practitioner applying TRACE in your work, encountering edge cases, or finding gaps, open an issue or start a discussion. Real-world feedback from people using this in operational environments is more valuable than theoretical refinement.

### How to break this framework

The most useful contribution is a counterexample. TRACE claims five dimensions are enough to locate any AI trust failure. Prove it wrong:

- **An incident that doesn't fit.** Walk a real AI failure through the five dimensions (the [case studies](guides/case-studies/) show the format). If the root cause lands in none of them, or only by forcing it, open an issue titled "Doesn't fit: ...". That is evidence the dimensions are not exhaustive.
- **A question with no resolution.** If a TRACE question gets the same answer for every system you evaluate, it is not helping you calibrate. Name it.
- **A decision TRACE got wrong.** You ran TRACE, accepted or rejected a system, and reality disagreed. What did the framework fail to ask?

Confirmations are welcome; refutations are more valuable.

## Author

[Melissa Bischoping](https://www.linkedin.com/in/mbischoping/) — Director, Secure Operations & Enablement at Siemens; security researcher, speaker, and SANS Technology Institute Board member. TRACE grew out of work on AI literacy for cybersecurity professionals, driven by the observation that our industry is adopting AI faster than it's developing the judgment to evaluate it.

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/). You are free to share and adapt this material for any purpose, including commercial, as long as you give appropriate credit.
