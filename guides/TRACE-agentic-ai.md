# TRACE for Agentic AI

A supplement to the [TRACE framework](../README.md) for evaluating AI agents — systems that take autonomous actions, not just produce outputs.

## Why Agents Are Different

Traditional AI systems generate outputs: a classification, a score, a recommendation. Someone acts on that output.

Agents act directly. They write files, call APIs, execute commands, send messages, modify databases. The distinction is not academic — it creates fundamentally different risk profiles and forensic obligations.

**The core problem:** We currently treat AI agents like sophisticated humans with no audit trail. They can do anything within their permission scope, but there's no standardized way to know what they did, when, or why.

When a firewall blocks traffic, you have logs. When an employee accesses a file, you have access logs. When an AI agent does either, you might have... nothing. Or scattered logs across multiple systems. Or logs the agent could theoretically modify.

This guide extends TRACE to address the forensic and operational challenges unique to agentic AI.

## Forensic Evaluation Questions

When evaluating any AI system that interacts with your environment, and especially agentic systems, ask these questions:

### Core Questions (Any AI System)

**1. Logging — Are system interactions logged?**

Where are logs stored? What format? Are prompts/inputs captured, or only outputs? If the system makes API calls to an external model, is that traffic logged?

*Red flag:* "Logs are stored locally on the instance" with no forwarding to centralized logging.

**2. Attribution — Can you distinguish AI actions from human actions?**

When something happens in your environment, can you tell if an AI did it? Do AI-initiated actions have distinct user accounts, service principals, or metadata tags? Or does the AI act as a human user?

*Red flag:* Agent runs under a shared service account with no additional attribution metadata.

**3. Retention — How long are logs/outputs preserved?**

What's the retention policy? Does it match your compliance and investigation timeline requirements? If logs roll over after 7 days, you can't investigate an incident discovered on day 8.

*Red flag:* Retention controlled by the AI vendor with no customer visibility or control.

**4. Correlation — Can you join AI activity to your SIEM and other logs?**

Do the logs have timestamps and identifiers that correlate to your existing telemetry? Can you build a unified timeline that includes AI actions alongside network, endpoint, and identity events?

*Red flag:* Proprietary log format with no standard export or integration path.

### Agent-Specific Questions (Deeper Layer)

**5. Action completeness — Are ALL agent actions logged?**

Not just the final output, but every intermediate action: file reads, API calls, tool invocations, subprocess execution. An agent that "summarizes your documents" might read 50 files to produce one summary. Are those 50 reads logged?

*Red flag:* Only final outputs are logged; intermediate actions are ephemeral.

**6. Immutability — Can the agent tamper with its own logs?**

If the agent has write access to the logging system, it could theoretically modify or delete evidence of its own actions. This isn't about malicious AI — it's about forensic integrity if the agent is compromised or manipulated.

*Red flag:* Agent has write access to its own log storage location.

**7. Session boundaries — Are agent sessions clearly delineated?**

Can you identify when an agent session started and ended? What triggered it? If an agent runs continuously, are there logical boundaries (conversation IDs, task IDs) that segment activity?

*Red flag:* No session concept; all activity appears as one continuous stream.

**8. Permission scope — What could the agent access?**

Not just what it did access, but what it *could* have accessed. If investigating an incident, you need to know the blast radius. Can you audit what permissions the agent had at any point in time? Are permission changes logged?

*Red flag:* Agent has broad standing permissions with no scoping or audit trail.

**9. Reasoning traces — Can you see why the agent took actions?**

Beyond what it did, can you see the chain of reasoning? This is distinct from Explainability (E) in TRACE — E asks "can I understand the decision?" while this asks "is the decision process preserved for later review?"

*Red flag:* Only actions are logged; reasoning/planning steps are not captured.

**10. Rollback capability — Can you undo agent actions?**

If an agent took a bad action (whether through error, drift, or adversarial manipulation), can you reverse it? This depends on what the agent can do — file modifications might be recoverable, sent emails are not.

*Red flag:* Agent takes irreversible actions with no confirmation gates.

## Agent-Specific TRACE Considerations

The five TRACE dimensions still apply to agents, but with additional considerations:

### T — Training (for Agents)

Beyond the base model's training data, ask:
- What tool-use examples shaped its behavior? How was it trained to use your specific tools?
- If the agent uses retrieval (RAG), what's in the retrieval corpus? Can that corpus be poisoned?
- Are there fine-tuning or RLHF steps specific to agentic behavior? What were the objectives?

### R — Reliability (for Agents)

Agent-specific failure modes:
- **Runaway actions:** Does the agent have guardrails against excessive or repeated actions?
- **Context window limits:** What happens when a long task exceeds context? Does it hallucinate history?
- **Tool-use failures:** If a tool call fails, does it retry indefinitely? Fail gracefully? Escalate?
- **State management:** How does the agent track what it's already done? Can it get confused about state?

### A — Attack Surface (for Agents)

Agent-specific threats:
- **Tool injection:** Can an attacker manipulate inputs to cause the agent to misuse tools?
- **Indirect prompt injection:** Can malicious content in retrieved documents, emails, or other inputs hijack the agent?
- **Lateral movement:** If compromised, what can the agent access? Is it a pivot point?
- **Privilege escalation:** Can the agent be tricked into requesting or using elevated permissions?
- **Data exfiltration:** Can an attacker use the agent to extract sensitive information?

Resources: [MITRE ATLAS](https://atlas.mitre.org), [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)

### C — Context (for Agents)

Agent-specific fit questions:
- **Supervision capacity:** Does your team have the bandwidth to review agent actions? What's the human-in-the-loop model?
- **Approval workflows:** For high-risk actions, is there a gate? Who approves? How quickly?
- **Incident response:** If the agent causes an incident, who owns response? Is there a kill switch?
- **Scope creep:** Agents tend to accumulate capabilities over time. Is there governance around what the agent is allowed to do?

### E — Explainability (for Agents)

Beyond decision explainability:
- Can you explain not just a single decision, but a sequence of actions?
- If the agent took 20 steps to complete a task, can you articulate the logic of the whole sequence?
- When the agent fails, can you explain to stakeholders what it was trying to do and where it went wrong?

## Questions to Ask Vendors

When evaluating agent-based products, add these to your vendor assessment:

1. Where are action logs stored? Can we forward them to our SIEM?
2. What's logged — just outputs, or all intermediate actions and reasoning?
3. Can the agent modify its own logs?
4. How do we identify agent actions in our existing logs (distinct user, metadata)?
5. What's the retention policy? Can we control it?
6. If we need to investigate an incident, what artifacts will we have?
7. Is there a session/conversation ID that segments activity?
8. Can we audit what permissions the agent had at any point in time?
9. What's the kill switch? How quickly can we halt agent activity?
10. What irreversible actions can the agent take? What gates exist?

## Red Flags Checklist

Warning signs that an agent deployment may not support investigation:

- [ ] No centralized logging — logs live only on agent instances
- [ ] Agent runs under shared/generic service account
- [ ] No retention policy or vendor-controlled retention
- [ ] Proprietary log format with no export
- [ ] Only final outputs logged, not intermediate actions
- [ ] Agent has write access to its own log location
- [ ] No session boundaries or correlation IDs
- [ ] Broad standing permissions without audit
- [ ] No reasoning traces preserved
- [ ] Irreversible actions without approval gates

## Policy Enforcement: From Optional to Mandatory

Identifying forensic gaps is necessary but not sufficient. The harder question: **how do you enforce logging when the agent operator could simply disable it?**

### The Maturity Model: Lessons from OS-Level Enforcement

Every major operating system has evolved from "logging is possible" to "logging is enforceable." AI agents need to follow the same path.

| Maturity Level | Windows | macOS | Linux | AI Agents (Current) |
|----------------|---------|-------|-------|---------------------|
| **1. Optional logging** | Early PowerShell | Basic syslog | Optional auditd | Most frameworks today |
| **2. Centralized policy** | Group Policy, Intune | MDM profiles | Ansible/Puppet + auditd rules | Does not exist |
| **3. Tamper-resistant storage** | Event Log (SYSTEM-owned) | Unified Logging (protected) | journald with sealing, remote syslog | Rare |
| **4. Constrained execution** | Constrained Language Mode, WDAC | TCC, App Sandbox | SELinux/AppArmor, seccomp | Emerging |
| **5. Signed/attested runtime** | Code signing, Secure Boot | Notarization, SIP | IMA, Secure Boot | Does not exist |

**Platform-specific enforcement mechanisms:**

- **Windows:** Group Policy enforces PowerShell Script Block Logging; WDAC controls what code can execute; Event Log is SYSTEM-owned and tamper-resistant
- **macOS:** MDM profiles enforce logging configuration; Unified Logging (`os_log`) writes to protected storage; Endpoint Security framework provides kernel-level visibility; TCC controls app permissions
- **Linux:** auditd rules can be centrally managed and made immutable (`-e 2`); SELinux/AppArmor enforce mandatory access control; eBPF enables deep observability; journald supports log sealing with FSS

Most agent deployments today are at Level 1 across all platforms. The industry needs to reach Level 3-5 for enterprise readiness.

### What Existing OS Telemetry Can (and Can't) Capture

**Audit trail vs. forensic evidence:** These terms overlap but aren't identical. An *audit trail* is a contemporaneous record of actions, typically structured and queryable. *Forensic evidence* is broader — any artifact that helps reconstruct what happened, including logs, memory dumps, file metadata, network captures, and more.

For AI agents, the question is: what can existing OS-level instrumentation actually tell us?

**Linux (eBPF and auditd):**

| Hook/Mechanism | What It Captures | AI Agent Relevance |
|----------------|------------------|-------------------|
| `execve` tracepoints | Process execution with arguments | Agent spawning subprocesses, tool invocations |
| File operation tracepoints | open, read, write, unlink | Files the agent accessed or modified |
| Network socket hooks | connect, accept, sendmsg | API calls to model providers, lateral network access |
| LSM hooks (via eBPF) | Security-relevant decisions | Permission checks the agent triggered |
| Tetragon, Falco, Sysdig | Higher-level policies on syscalls | "Alert when this process writes to /etc" |

**Windows (ETW, kernel callbacks):**

| Hook/Mechanism | What It Captures | AI Agent Relevance |
|----------------|------------------|-------------------|
| Process creation callbacks | Process start with command line | Agent spawning tools |
| ETW providers (Microsoft-Windows-Kernel-*) | File, registry, network, process events | Comprehensive syscall-level visibility |
| AMSI | Script content before execution | If agent executes PowerShell/VBScript |
| WFP (Windows Filtering Platform) | Network connections | API traffic, lateral connections |
| File system filter drivers | All file operations | EDR-level file access logging |

**macOS (Endpoint Security, OpenBSM):**

| Hook/Mechanism | What It Captures | AI Agent Relevance |
|----------------|------------------|-------------------|
| Endpoint Security framework | Process, file, network events | Kernel-level visibility, similar to eBPF |
| OpenBSM / auditd | Syscall-level audit trail | Traditional audit logging |
| Network Extension | Network filter capability | Inspecting agent network traffic |
| TCC database | Permission grants | When agent was granted file/camera/mic access |

**The gap: OS hooks see actions, not reasoning**

OS-level telemetry tells you:
- The agent process read these 50 files
- It made HTTPS connections to api.anthropic.com
- It spawned `git commit` as a subprocess
- It wrote to `/var/log/app.log`

OS-level telemetry does NOT tell you:
- What prompt the user gave the agent
- Why the agent decided to read those files
- What the agent was planning to do next
- Whether the agent was manipulated by prompt injection

**This is the critical gap.** You can reconstruct *what happened* from OS telemetry (forensic evidence of actions). You cannot reconstruct *why it happened* (the reasoning chain, the decision process).

For full forensic capability, you need both:
1. **OS-level action logs** — what the agent actually did in the environment
2. **Agent-level reasoning logs** — what the agent was thinking, planning, deciding

Today, #1 is achievable with existing tools. #2 depends entirely on the agent framework's logging — and that's what's often missing or bypassable.

### Why Agent Logging is Hard to Enforce

Current agent architectures have a fundamental problem: **the agent operator controls the logging configuration.**

| Enforcement Layer | Who Controls It | Bypass Difficulty |
|-------------------|-----------------|-------------------|
| Agent-level hooks | Operator/user | Trivial — disable the hook |
| SDK/runtime config | Agent developer | Easy — fork or patch the SDK |
| API-level logging | API provider | Hard — but you don't own the logs |
| Infrastructure logging | Enterprise IT | Medium — requires infra access |
| Hardware attestation | Platform vendor | Very hard — but doesn't exist yet |

If the person running the agent can also configure it, they can disable logging. This is the pre-Group Policy era for AI agents.

### What Enterprises Can Do Today

Until the industry matures, defense-in-depth:

**1. Separation of duties**
- The team operating agents should not control logging configuration
- Logging config should be owned by security/IT, deployed via infrastructure-as-code
- Treat agent config like you treat EDR config — operators can't disable it

**2. Network-layer capture**
- Proxy all agent API traffic through an enterprise gateway
- Log requests/responses at the network layer, independent of agent config
- This captures API calls even if the agent's internal logging is disabled

**3. Demand API-level audit feeds from vendors**
- API providers (Anthropic, OpenAI, etc.) log all API calls on their side
- Enterprise contracts should include access to those logs
- "You log it" is not enough — "we get the logs" is the requirement

**4. Infrastructure-level monitoring**
- EDR sees what processes agents spawn, files they touch, network connections
- CASB/SASE sees API traffic to AI providers
- Identity systems see what the agent's service principal accessed
- Correlate these with agent session IDs

**5. Immutable log destinations**
- Forward agent logs to append-only storage (S3 with Object Lock, immutable blob storage)
- The agent should not have credentials to modify or delete logs
- Separate log ingestion credentials from log deletion credentials

### What the Industry Needs to Build

For AI agents to reach enterprise maturity, we need:

**Centralized policy enforcement**
- The equivalent of Group Policy for AI agents
- Enterprise IT defines: what tools are allowed, what must be logged, what requires approval
- Policy is enforced at a layer the agent can't bypass

**Attested agent runtimes**
- Signed agent configurations that can be verified
- The agent can prove it hasn't been modified to bypass logging
- Similar to Secure Boot — the runtime attests its integrity

**Standardized audit event schemas**
- Common format for agent action logs (like CEF/LEEF for security events)
- Enables cross-vendor correlation and SIEM integration
- Industry bodies (OWASP, MITRE, NIST) should drive this

**Provider-side audit feeds**
- API providers offer enterprise audit log access as a standard feature
- Logs include: prompts, responses, tool calls, token usage, session metadata
- Customer-owned, exportable, with configurable retention

**Constrained execution modes**
- "High-security mode" that restricts agent capabilities
- Analogous to PowerShell Constrained Language Mode
- Fewer tools available, mandatory approval gates, enhanced logging

### Questions to Ask About Enforcement

Add these to vendor assessments:

1. Can we enforce logging configuration via centralized policy, or only per-agent config?
2. If an operator disables logging, would we know?
3. Do you offer an enterprise audit feed for API-level logs?
4. Can the agent's service principal delete or modify its own logs?
5. Do you support signed/attested agent configurations?
6. Is there a constrained execution mode for sensitive environments?
7. What's your roadmap for enterprise policy enforcement?

### The Goal: Logging That Can't Be Turned Off

The end state we need:

- **Logging happens at layers the agent doesn't control** (API provider, network infrastructure, platform)
- **Logs are tamper-evident** (append-only, cryptographically signed, or stored where agent has no write access)
- **Policy is centrally enforced** (enterprise IT controls what agents can do and what gets logged)
- **Attestation is possible** (agents can prove their logging configuration hasn't been bypassed)

Until we get there, treat agent logging as you would any high-privilege system: trust but verify, defense in depth, and assume the logs you have may be incomplete.

## Summary

Agents are not just AI systems that happen to take actions. They are a distinct category with forensic requirements traditional AI doesn't create.

The core TRACE framework helps you evaluate any AI system. This supplement adds the questions you need when that AI system can act autonomously in your environment:

- **What can it do?** (permission scope)
- **What did it do?** (action logging, completeness)
- **Why did it do it?** (reasoning traces)
- **Can I prove it?** (immutability, attribution)
- **Can I undo it?** (rollback)

Until the industry develops better standards for agent observability, security teams should treat agent forensics with the same rigor they apply to any other high-privilege system component.

---

*This guide is part of the TRACE framework by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*
