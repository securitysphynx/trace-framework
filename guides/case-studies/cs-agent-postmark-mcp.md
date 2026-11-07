# Poisoned Postmark MCP Server Email Exfiltration (ATLAS AML.CS0053)

In September 2025 an unknown actor used a namesquatted npm package, `postmark-mcp`, to copy every
email sent through it to an address they controlled. Postmark, an email delivery service, had never
published its MCP server to npm; its official server lives under the ActiveCampaign organisation
on [GitHub](https://github.com/ActiveCampaign/postmark-mcp). The actor published functional copies
first, then, per [Postmark's
statement](https://postmarkapp.com/blog/information-regarding-malicious-postmark-mcp-package/) of 25
September 2025, built trust over 15 versions before adding a backdoor in version 1.0.16 that
silently BCC'd outgoing mail to an external address. [ATLAS
AML.CS0053](https://atlas.mitre.org/studies/AML.CS0053) records it as an incident, dated 17 to 25
September 2025, reported by Koi Research, and notes the package had passed 1,000 downloads per week
before the switch.
[Snyk](https://snyk.io/blog/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails/) identified
the BCC address as `phan@giftshop.club`.

## What happened

Technique IDs are from the ATLAS 2026.10 release of AML.CS0053. (The legacy ATLAS.yaml file maps
step 3 to AML.T0104 and step 6 to AML.T0110; release 2026.10 replaced both with the IDs below.)

1. **Defense Evasion, AML.T0073 Impersonation.** The actor published a legitimate copy of the server
   as `postmark-mcp` on npm. Postmark had not registered the name, so the namesquat succeeded.
2. **Resource Development, AML.T0017 Develop Capabilities.** The actor modified the server so the
   email tool added their address to the BCC line of every message.
3. **Resource Development, AML.T0115.002 AI Agent Tools** (sub-technique of Publish Poisoned AI
   Artifacts). The malicious version was published to npm.
4. **Defense Evasion, AML.T0109 AI Supply Chain Rug Pull.** Because users had already adopted the
   clean versions, the update avoided the scrutiny new tools receive.
5. **Initial Access, AML.T0010.005 AI Agent Tool.** Organisations that upgraded to 1.0.16 received
   the backdoored tool through their normal dependency path.
6. **Persistence, AML.T0110.001 Implementation** (sub-technique of AI Agent Tool Poisoning). Once
   configured into an organisation's agents, the poisoned server stayed in place.
7. **Execution, AML.T0011.002 Poisoned AI Agent Tool.** When users asked their agent to send mail,
   the agent called the tool and the malicious code ran.
8. **Exfiltration, AML.T0086 Exfiltration via AI Agent Tool Invocation.** The full content of each
   email went to the actor via BCC.
9. **Impact, AML.T0048 External Harms.** Exposed mail could include transactional messages (client
   private data) and promotional messages (client lists).

What the agent saw versus what happened:

| Layer | What it recorded |
|---|---|
| User prompt | "Send this email to X" |
| Agent reasoning and tool call | A correct call to the email tool, addressed to X |
| MCP server code (below the agent) | The same call, plus a hard-coded BCC to the actor |
| Mail delivery | Two recipients. Postmark advised customers to check email logs for suspicious activity. |

## TRACE walk

| Dimension | What the defender knew or could have known at the time | What the incident revealed |
|---|---|---|
| **T** Training | **Known, and irrelevant.** Nothing about the model or its training data was involved. | The model was not attacked. It made correct tool calls throughout. The agentic guide's T questions (tool-use training, RAG corpus) have nothing to say here. |
| **R** Reliability | **Known.** The agent and tool worked as expected; emails were delivered. | The tool kept working perfectly, which is what made the backdoor invisible. There was no degradation to detect at the agent layer. |
| **A** Attack Surface | **Partial.** Defenders could know the package was not published by Postmark: the official server was on GitHub, and the npm publisher was not Postmark. Whether anyone checked is unknown. | A supply chain compromise of an agent tool, delivered as a routine version bump. The adversary needed no access to the model, the prompt or the victim's network. |
| **C** Context | **Known.** Each team knew it had given an agent an email-sending tool, and that sent email cannot be recalled. | Permission scope was the blast radius: every message sent through the tool. Agent tools installed by individual users from a public registry sat outside whatever review the organisation applied to the agent itself. |
| **E** Explainability | **Partial.** Agent-side logs could show what the agent asked for. | Those logs would explain the agent's decisions correctly and still miss the exfiltration, because the malicious behaviour lived in tool code below the reasoning layer. Explaining the agent did not explain the incident. |

## Which dimension broke

The failure lived in **Attack Surface**, in the supply chain row: "Compromised models, plugins,
dependencies." No AI component misbehaved. A third-party package that an agent was configured to
trust was replaced by a malicious version under a name the real vendor had never claimed, and teams
pulled the update the same way they pull any dependency.

**Context** determined the damage. An email tool is an exfiltration channel by design, and the
agentic guide already names it as irreversible ("file modifications might be recoverable, sent
emails are not"). The Context failure is governance: the agent was evaluated, if at all, as an AI
system, while the tool it called was treated as ordinary plumbing.

## Would TRACE have surfaced it

Partly. The README's Attack Surface table lists the threat class directly:

> *Supply chain | Compromised models, plugins, dependencies*

And the agentic guide's Context section asks:

> *Scope creep: Agents tend to accumulate capabilities over time. Is there governance around what
> the agent is allowed to do?*

A team that asked those questions about each MCP server would have had grounds to check who
published `postmark-mcp` and to pin versions. That check would have caught the namesquat, since
Postmark had never published to npm. TRACE would not have caught the rug pull itself through its
agent-specific controls. The forensic questions (action logging, reasoning traces, attribution) and
the "irreversible actions without approval gates" red flag all operate at the agent layer. A human
approving "send this email to X" would have approved, correctly, and the BCC would still have been
added. The controls that work here are software supply chain controls (publisher verification,
pinning, update review), which TRACE points to but does not supply.

## Validity note

This incident fit TRACE badly in one specific way. T, R and E were effectively empty because the
model was never the target; the whole incident sat in a single Attack Surface row plus Context.
TRACE has no dimension for the provenance of the code an agent executes. T asks about the provenance
of training data, and the agentic guide asks about tool-use training, but neither asks "who
published this tool and what changed in this version." The same attack would have worked against a
non-AI integration using the same package, which raises a fair question about whether TRACE should
treat agent tools as their own evaluation object.

## Sources

- HIGH: MITRE ATLAS, AML.CS0053, release 2026.10, read in full from the [atlas-data
  repository](https://github.com/mitre-atlas/atlas-data). Web view:
  https://atlas.mitre.org/studies/AML.CS0053
- HIGH: Postmark, "Security Alert: Malicious 'postmark-mcp' npm Package Impersonating Postmark," 25
  September 2025. Statement from the impersonated vendor.
  https://postmarkapp.com/blog/information-regarding-malicious-postmark-mcp-package/
- MED: Snyk, "Malicious MCP Server on npm postmark-mcp Harvests Emails," 25 September 2025. Used
  only for the BCC address. Its version labels are inconsistent (code labelled 1.0.18 next to a
  `package.json` showing 1.0.14), so version claims here follow Postmark and ATLAS.
  https://snyk.io/blog/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails/
- Not read: Koi Research, "First Malicious MCP in the Wild: The Postmark Backdoor That's Stealing
  Your Emails," the reporter ATLAS credits.
  https://www.koi.ai/blog/postmark-mcp-npm-malicious-backdoor-email-theft redirected to an unrelated
  Palo Alto Networks product page in November 2026.

*Part of the [TRACE Framework](../../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*

