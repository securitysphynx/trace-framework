# EchoLeak: Zero-Click Prompt Injection Targeting M365 Copilot (ATLAS AML.CS0059)

EchoLeak was a zero-click indirect prompt injection in Microsoft 365 Copilot: a single crafted
email, sent by an external attacker with no access to the tenant, could cause Copilot to pull
sensitive data from the victim's mailbox and files into its answer and send it to an
attacker-controlled server without the victim opening the email or clicking anything. Aim Labs (Aim
Security) found it and reported it to Microsoft MSRC in January 2025; Microsoft deployed a
server-side fix in May 2025 with no customer action required, and published
[CVE-2025-32711](https://www.cve.org/CVERecord?id=CVE-2025-32711) (CVSS 3.1 base score 9.3) on 11
June 2025, the same day Aim Labs disclosed. Timeline per the [EchoLeak case-study
paper](https://arxiv.org/abs/2509.10540) (Reddy and Gujral, September 2025). MITRE records it as an
exercise in [ATLAS AML.CS0059](https://atlas.mitre.org/studies/AML.CS0059), which states it was
remediated with no evidence of exploitation in the wild.

## What happened

Technique IDs are from the ATLAS 2026.10 release of AML.CS0059. Steps are in ATLAS order.

1. **AI Attack Adaptation, AML.T0065 LLM Prompt Crafting.** The researchers wrote instructions
   designed to evade Microsoft's indirect prompt injection (XPIA) classifier, read like ordinary
   business content, and tell Copilot not to mention the source email.
2. **AI Attack Adaptation, AML.T0066 Retrieval Content Crafting.** The instructions were embedded in
   business-like email text likely to be retrieved during a later, unrelated Copilot request.
3. **Resource Development, AML.T0079 Stage Capabilities.** The researchers staged a web endpoint to
   receive outbound requests carrying encoded data.
4. **Initial Access, AML.T0093 Prompt Infiltration via Public-Facing Application.** They sent the
   email to the target's Microsoft 365 inbox.
5. **Defense Evasion, AML.T0068 LLM Prompt Obfuscation.** The prompt was phrased as benign business
   text rather than an obvious injection.
6. **Persistence, AML.T0070 RAG Poisoning.** The email was ingested automatically into the content
   Copilot retrieves from.
7. **Execution, AML.T0051.002 Triggered.** When the user later asked Copilot something, it retrieved
   the email into context and the injection ran.
8. **Collection, AML.T0085.000 RAG Databases.** The instructions caused Copilot to gather sensitive
   information the user could reach: emails, files, project details.
9. **Defense Evasion, AML.T0067 LLM Trusted Output Components Manipulation.** The output used
   reference-style Markdown links and images, which the link redaction filter did not strip, and
   avoided attributing the content to the email.
10. **Exfiltration, AML.T0077 LLM Response Rendering.** Copilot rendered a Markdown image whose URL
    encoded the data. The client fetched the image automatically, so no click was needed.
11. **Exfiltration, AML.T0025 Exfiltration via Cyber Means.** To get past the content security
    policy, the image request was routed through an allowlisted Microsoft Teams URL preview
    endpoint, which fetched the attacker URL on the client's behalf.
12. **Impact, AML.T0048 External Harms.** Against a real user, the result would be disclosure of
    confidential business data.

Four separate controls were bypassed in sequence:

| Control | How it was bypassed (per ATLAS and the paper) |
|---|---|
| XPIA prompt injection classifier | Instructions phrased as a normal business request, not "ignore previous instructions" |
| Link redaction | Reference-style Markdown instead of inline links |
| User click as a gate | Markdown image, fetched automatically by the client |
| Content security policy | Request relayed through an allowlisted Teams preview endpoint |

## TRACE walk

| Dimension | What the defender knew or could have known at the time | What the incident revealed |
|---|---|---|
| **T** Training | **Partial.** Customers knew Copilot was a general-purpose LLM, with no visibility into how it was trained to treat retrieved text. | Training was not where this broke. The model did what LLMs do with instructions in context: it followed them. No step in ATLAS turns on training data. |
| **R** Reliability | **Unknown.** No customer-visible signal existed for "the assistant followed an instruction from an email." | The failure was silent from the user's side: the answer looked normal, the email was not cited, and the leak was an image load. Detection would have depended on Microsoft's telemetry, not the tenant's. |
| **A** Attack Surface | **Partial.** Indirect prompt injection against retrieval-augmented assistants was publicly documented before 2025 (ATLAS AML.CS0020, Bing Chat, 2023; AML.CS0035, Slack AI, 2024), and Microsoft had an XPIA classifier in place against it. Whether those defences held was unknown to customers. | Every layer had a bypass, and they chained. The classifier, the redaction filter and the CSP were each defeated by a format-level trick, not by a novel model attack. |
| **C** Context | **Known.** Tenants knew Copilot could read whatever the user could read, and that external email lands in the same mailbox. | That combination is the vulnerability. Untrusted external content and privileged internal data shared one context window, and the assistant's reach was the user's full M365 scope. The paper's mitigations lead with prompt partitioning and provenance-based access control. |
| **E** Explainability | **Partial.** Answers could attribute retrieved content; ATLAS notes the attack had to suppress that attribution. Whether a tenant could reconstruct which items shaped an answer is not described in the sources. | The injected instructions told Copilot to suppress attribution to the email, so the one artefact that would have explained the answer was removed by the attack. |

## Which dimension broke

The failure lived in **Context**: specifically, the permission scope of the assistant relative to
the trust level of its inputs. Copilot was designed to retrieve from the user's whole Microsoft 365
estate, and that estate includes mail from anyone on the internet. Once untrusted and trusted
content share a context window and the model can emit content that the client fetches, an injection
that survives filtering becomes an exfiltration channel. The defender knew both facts at deployment
time. What they did not have was a way to separate them.

**Attack Surface** is where the exploit ran, and it is the dimension a vendor would point to. But
each Attack Surface control (classifier, redaction, CSP) was a probabilistic or pattern filter
layered on top of an architecture that allowed the leak. The README's opening claim applies
directly: "If your defense relies on the model agreeing to follow a rule, you have not built a
security control."

## Would TRACE have surfaced it

Yes, as a risk class, and a customer could have asked the right questions. From the [agentic
guide](../TRACE-agentic-ai.md), under A for agents:

> *Indirect prompt injection: Can malicious content in retrieved documents, emails, or other inputs
> hijack the agent?*

> *Data exfiltration: Can an attacker use the agent to extract sensitive information?*

And forensic question 8:

> *Not just what it did access, but what it could have accessed.*

Asked of M365 Copilot before rollout, the honest answers were "yes, emails are retrieved," "the
defence is a classifier," and "everything the user can read." TRACE would have flagged that as an
unresolved Attack Surface risk with a Context-sized blast radius. It would not have found the
reference-style Markdown or the Teams proxy; those are vendor-side implementation flaws, and the fix
was server-side with no customer action. TRACE's practical output here is a deployment decision
(scope, sensitivity labelling, which users get it), not a patch.

## Validity note

Two things did not fit cleanly. First, the exfiltration channel (Markdown image auto-fetch through
an allowlisted proxy) is conventional web application security; TRACE files it under Attack Surface,
but nothing about it is AI-specific, and the A dimension's threat table does not list output
handling as a category. Second, TRACE assumes the evaluator can act on what it finds; here every
control that mattered was in the vendor's hands, so the walk ends at "accept the risk or limit the
scope," which TRACE supports but does not name.

## Sources

- HIGH: MITRE ATLAS, AML.CS0059, release 2026.10, read in full from the [atlas-data
  repository](https://github.com/mitre-atlas/atlas-data). Web view:
  https://atlas.mitre.org/studies/AML.CS0059
- HIGH: CVE-2025-32711 record, CNA Microsoft, published 11 June 2025, read via the CVE Services API.
  https://www.cve.org/CVERecord?id=CVE-2025-32711
- MED: Reddy and Gujral, "EchoLeak: The First Real-World Zero-Click Prompt Injection Exploit in a
  Production LLM System," arXiv:2509.10540, 6 September 2025. Read in full; it is a case study built
  from public information, and the authors did not reproduce the attack. Source of the timeline.
  https://arxiv.org/abs/2509.10540
- Not read: Aim Labs original disclosure (https://www.aim.security/lp/aim-labs-echoleak-m365)
  returned HTTP 403. Cato Networks analysis listed by ATLAS
  (https://www.catonetworks.com/blog/breaking-down-echoleak/) returned no content. Microsoft's
  advisory (https://msrc.microsoft.com/update-guide/vulnerability/CVE-2025-32711) is a JS
  application and was not read.

*Part of the [TRACE Framework](../../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*

