# TRACE Investigate

A skill for investigating AI-related security incidents through the TRACE framework.

## Skill Definition

```yaml
name: trace-investigate
description: Incident analysis for AI system failures OR adversary use of AI in attacks
trigger: When investigating an AI system failure, or when adversaries used AI tools in an attack
output: Structured incident analysis with root cause mapping to TRACE dimensions
```

## Modes of Operation

This skill covers two distinct investigation scenarios:

1. **Mode 1: AI System Failure** — Your AI system missed an attack, flagged innocent users, or behaved unexpectedly. TRACE helps diagnose what went wrong.

2. **Mode 2: Adversary Use of AI** — An adversary used AI tools during an attack. They may have lived off the land using your enterprise AI tools, deployed their own AI capabilities, or leveraged AI to craft attacks. TRACE helps you understand what happened and what evidence exists.

---

## Mode 1: AI System Failure Investigation

You are helping a security practitioner investigate an AI system failure or unexpected behavior. Something went wrong — an AI system missed an attack, flagged innocent users, produced bad output, or behaved unexpectedly. Your role is to systematically diagnose what happened using the five TRACE dimensions.

### Investigation Flow

**Step 1: Establish the incident**

Ask:
- What happened? Describe the failure or unexpected behavior.
- When did it occur? When was it discovered?
- What was the impact? (missed detection, false action, operational disruption)
- What AI system was involved?
- Is this a one-time event or a pattern?

**Step 2: Apply TRACE as a diagnostic framework**

Work through each dimension to identify potential root causes.

---

#### T — Training: Was the foundation compromised?

Investigate:
- Was the failure related to something outside the model's training data?
- Could training data have been poisoned or biased in a way that caused this?
- For retrieval systems: Was the retrieval corpus compromised or incomplete?
- Was there a data freshness issue? (e.g., model didn't know about new attack technique)

Questions to answer:
- [ ] Was the scenario represented in training data?
- [ ] Any evidence of training data issues contributing to failure?
- [ ] For RAG systems: Was retrieval corpus verified?

---

#### R — Reliability: Did the system degrade?

Investigate:
- Did the system's performance metrics change before the incident?
- Was there data drift — input distribution different from training?
- Was there concept drift — the underlying patterns changed?
- Did the system hit an edge case or boundary condition?
- Were there signs of degradation that went unnoticed?

Questions to answer:
- [ ] Was drift monitoring in place? Did it alert?
- [ ] What were the confidence scores on the failed output?
- [ ] Was this a known failure mode or a new one?

---

#### A — Attack Surface: Was this adversarial?

Investigate:
- Could an attacker have crafted input to cause this failure? (evasion)
- Is there evidence of prompt injection? (direct or indirect)
- Could an adversary have influenced training data or retrieval corpus?
- Was the AI system used as a pivot point for lateral movement or exfiltration?
- Does this map to any MITRE ATLAS techniques?

Questions to answer:
- [ ] Was input anomalous in ways suggesting adversarial crafting?
- [ ] For LLM systems: Was there potentially malicious content in context?
- [ ] Could this be an attack vs. a bug?

---

#### C — Context: Was there a deployment mismatch?

Investigate:
- Was the system operating outside its intended use case?
- Was there a mismatch between the system's design and your environment?
- Did human operators miss something the system surfaced?
- Did the human-in-the-loop process fail?
- Was the system being asked to do something it wasn't designed for?

Questions to answer:
- [ ] Was the system used within its documented scope?
- [ ] Did environmental factors contribute? (scale, data types, edge cases)
- [ ] Did the human oversight process work as designed?

---

#### E — Explainability: Can we reconstruct what happened?

Investigate:
- Do we have logs of what the system did and why?
- Can we trace the decision chain that led to the failure?
- For agents: Do we have complete action logs?
- Can we explain to stakeholders what went wrong?
- Are there forensic gaps that prevent full reconstruction?

Questions to answer:
- [ ] Do we have sufficient logs to reconstruct the incident?
- [ ] Can we explain the failure to leadership/legal/regulators?
- [ ] What evidence is missing that we wish we had?

---

**Step 3: Identify root cause(s)**

Based on the investigation, identify which TRACE dimension(s) contributed:

```markdown
## Root Cause Analysis

### Primary Cause
[Which dimension? What specifically failed?]

### Contributing Factors
[Other dimensions that played a role]

### Evidence
[What logs, data, or artifacts support this conclusion?]

### Unknowns
[What couldn't we determine? What evidence is missing?]
```

**Step 4: Recommend remediation**

For each root cause, recommend specific fixes:

| Root Cause | Remediation | Priority |
|------------|-------------|----------|
| Training gap | [specific action] | |
| Reliability issue | [specific action] | |
| Attack surface | [specific action] | |
| Context mismatch | [specific action] | |
| Explainability gap | [specific action] | |

**Step 5: Identify systemic improvements**

Beyond fixing this incident, what should change?

- Monitoring gaps to close
- Logging to add
- Process changes needed
- Vendor conversations to have
- Policy updates required

---

## Mode 1 Usage Examples

**User:** "Our AI-powered malware detection missed a ransomware sample that hit three endpoints yesterday."

**Assistant:** Begins incident scoping, then investigates through each TRACE dimension — was this in training data? did reliability degrade? was it adversarially crafted to evade? was there a deployment mismatch? can we reconstruct what the system saw?

**User:** "The AI assistant we deployed for IT helpdesk started giving users instructions to disable security controls."

**Assistant:** Focuses on A (prompt injection?) and E (can we see what led to this?), then works through other dimensions.

---

## Mode 2: Adversary Use of AI Investigation

You are helping a security practitioner investigate an incident where adversaries used AI tools as part of their attack. This could be:

- **Living off the land:** Adversary abused enterprise AI tools already deployed in your environment (Copilot, Claude, internal chatbots, AI assistants)
- **Adversary-deployed AI:** Attacker brought their own AI tools or used external AI services during the attack
- **AI-assisted attack techniques:** AI was used to craft phishing, generate malware variants, automate reconnaissance, or accelerate attack stages

TRACE helps you understand what AI capabilities were involved and what forensic evidence exists.

### Investigation Flow

**Step 1: Establish the AI involvement**

Ask:
- How was AI involved in this incident? (victim's tools abused, attacker tools, AI-generated content)
- What's the evidence of AI involvement? (logs, artifacts, behavioral indicators)
- Which AI systems or services were accessed or used?
- What was the timeline of AI-related activity?

**Step 2: Apply TRACE for adversary AI investigation**

---

#### T — Training: What did the adversary's AI know?

If the adversary used your AI systems:
- Did they access systems with sensitive training data or RAG corpora?
- Could they have extracted information about your environment from AI responses?
- Did they poison or manipulate any training data or retrieval sources?

If the adversary brought their own AI:
- What capabilities did their AI tools have?
- Were AI-generated artifacts (phishing, code, content) used in the attack?
- Can you identify the AI system used? (writing style, metadata, tool signatures)

Questions to answer:
- [ ] What AI systems did the adversary interact with?
- [ ] What information could AI systems have disclosed?
- [ ] Any evidence of training data or RAG manipulation?

---

#### R — Reliability: Did AI tools behave as expected during the attack?

Investigate:
- Did your AI security tools perform as expected during the intrusion?
- Were there detection gaps that AI systems should have caught?
- Did the adversary exploit reliability weaknesses (edge cases, confidence thresholds)?
- Were AI-generated attack artifacts effective? (indicates attacker AI reliability)

Questions to answer:
- [ ] Did AI security controls fire appropriately?
- [ ] Were there reliability gaps the adversary exploited?
- [ ] How effective were adversary AI-generated artifacts?

---

#### A — Attack Surface: How was AI weaponized?

Investigate:
- **Prompt injection:** Did the adversary manipulate AI systems via malicious prompts?
- **Tool abuse:** Did they use AI tool capabilities for unintended purposes (reconnaissance, data exfiltration)?
- **API abuse:** Were AI APIs accessed with stolen credentials or from unexpected locations?
- **AI as C2:** Was an AI system used as a command-and-control channel? (SesameOp-style)
- **Supply chain:** Did compromised AI tools or plugins enable access?

MITRE ATLAS techniques to check:
- AI Agent Context Poisoning (persistent manipulation of agent context)
- Exfiltration via AI Agent Tool Invocation (using "write" tools to leak data)
- LLM Prompt Injection for code execution or data access

Questions to answer:
- [ ] Was prompt injection used?
- [ ] Were AI tool capabilities abused?
- [ ] Did AI systems enable lateral movement or exfiltration?

---

#### C — Context: Why was AI available to the adversary?

Investigate:
- What AI tools were accessible from the compromised position?
- Were AI systems appropriately segmented and access-controlled?
- Did the adversary pivot through AI tools to reach other systems?
- What permissions did compromised AI tools have?
- Was there human oversight on AI actions during the attack window?

Questions to answer:
- [ ] What AI capabilities were in the adversary's blast radius?
- [ ] Were AI permissions scoped appropriately?
- [ ] Did AI tools enable access that wouldn't otherwise exist?

---

#### E — Explainability: What forensic evidence exists?

Investigate:
- **Logs:** What AI interaction logs exist? (prompts, responses, tool calls, API access)
- **Attribution:** Can you distinguish adversary AI usage from legitimate use?
- **Session boundaries:** Can you isolate adversary sessions from normal activity?
- **Reasoning traces:** For agents, do you have action logs and decision chains?
- **Gaps:** What evidence is missing? What can't you reconstruct?

Questions to answer:
- [ ] Do AI logs capture adversary activity?
- [ ] Can you build a timeline of AI-related adversary actions?
- [ ] What questions can't you answer due to logging gaps?

---

**Step 3: Build the adversary AI timeline**

```markdown
## Adversary AI Activity Timeline

| Time | AI System | Action | Evidence | Gaps |
|------|-----------|--------|----------|------|
| | | | | |

### AI Systems Accessed
[List all AI systems the adversary interacted with]

### AI-Enabled Capabilities
[What could the adversary do via AI that they couldn't do otherwise?]

### Forensic Gaps
[What AI activity couldn't we reconstruct?]
```

**Step 4: Assess AI-specific impact**

- What information did AI systems disclose to the adversary?
- Did AI tools amplify the attack (speed, scale, sophistication)?
- Were AI systems used to maintain persistence or evade detection?
- What's the blast radius of AI tool permissions the adversary accessed?

**Step 5: Recommend AI-specific hardening**

| Finding | Remediation |
|---------|-------------|
| Adversary accessed enterprise AI | Review AI tool access controls, segmentation |
| No logs of AI interactions | Implement AI audit logging |
| AI disclosed sensitive info | Review what AI systems can access/retrieve |
| AI tools had broad permissions | Apply least privilege to AI service accounts |
| Couldn't attribute AI sessions | Implement session tracking, user attribution |

---

## Mode 2 Usage Examples

**User:** "We found evidence the attacker used our internal Copilot to search for credentials and map out our infrastructure."

**Assistant:** Focuses on E (what logs exist from Copilot?), C (why could they access Copilot from that position?), and A (did they use prompt injection or just normal queries?). Builds a timeline of Copilot interactions during the attack window.

**User:** "The phishing emails in this campaign look AI-generated. How do we investigate?"

**Assistant:** Focuses on T (can we identify the AI system used?), A (what techniques suggest AI involvement?), and recommends artifact analysis approaches for AI-generated content.

**User:** "An attacker used our Claude Code deployment to write scripts that exfiltrated data."

**Assistant:** Full TRACE investigation — T (what could Claude access?), R (did it behave as expected?), A (prompt injection or legitimate-looking requests?), C (why did Claude have those permissions?), E (do we have complete action logs?).

---

*Part of the [TRACE Framework](../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
