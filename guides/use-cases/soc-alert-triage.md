# TRACE for the SOC: Acting on an AI-Generated Alert

A use-case guide for the [TRACE framework](../../README.md). Scenario: an alert produced or enriched by an AI system has landed in your queue, and you have about five minutes to decide what to do with it.

This is the fastest TRACE use case and the one most analysts hit daily. It uses three of the five dimensions — **Reliability, Context, Explainability** — because Training and Attack Surface are evaluated once, when the tool is bought, not at 02:00 on every alert. (If you have never evaluated the tool, that is a finding too; see [Escalating to a full evaluation](#escalating-to-a-full-evaluation).)

---

## The Five-Minute Card

Work top to bottom. Each row has a pass condition and a flag condition. Write down what you find; the result is a one-line knowledge map, not a score.

| Step | Ask | Pass | Flag |
|---|---|---|---|
| **R1** Base rate | What is this detector's *measured* precision for this alert type, at our volume? Not the vendor's accuracy figure — the number from our own tuning log. | A number exists and it is from our environment | No number, or only the vendor's. Treat the alert as unweighted until verified. |
| **R2** Drift | Has this alert type's false-positive pattern changed recently (new source onboarded, model update, rule tweak)? | No known change | A change shipped this week. Expect noise. |
| **C1** Population fit | Is the flagged asset, user or flow inside the population this detector was built for and tuned on? | Yes | Edge case: new hire, contractor, OT segment, legacy host, non-English mail, travelling exec. Edge cases are where AI alerts fail quietly. |
| **C2** Capacity | If this is real, can we act on it in the window that matters? | Yes, with a named owner | No. Then the question is escalation, not triage. |
| **E1** Evidence | What did the system *see*? Does the alert carry the inputs and artifacts (headers, hashes, URLs, logs, the actual access events), or only a verdict and a narrative? | Artifacts attached | Verdict and narrative only. You are being asked to trust the model's word. |
| **E2** Reconstruct | Can I rebuild the finding from the artifacts without the model's explanation? | Yes | No. The explanation is doing the work the evidence should do. |
| **E3** Justify | Can I write the ticket's justification line from E1 alone, in a sentence a non-engineer would accept? | Yes | No. Do not escalate on narrative. |

**Cross-cutting, for any alert where a language model wrote part of it:** *which of these findings still holds if the model was manipulated or simply wrong?* Everything in E1 holds. Nothing in the narrative does.

### Decision

| Knowledge map | Decision |
|---|---|
| R known, C fits, E has artifacts | **Act.** Standard response for the alert class. |
| R known, C fits, E narrative-only | **Verify first.** Pull the artifacts yourself before acting. Note the gap; it is a tool finding, not just an alert finding. |
| R unknown, anything else | **Verify, and open a tuning task.** You cannot weight this alert. The detector has no measured precision in your environment, which is a Reliability finding about the tool. |
| C edge case | **Verify with the owner of the edge.** Call the help desk, the OT lead, the exec's assistant. Edge cases are also where the real attacks hide; "it's a new hire" cuts both ways. |
| E fails and the subject is a person | **Do not escalate to HR, legal or management on the model's narrative.** Gather evidence first. See [Worked alert 2](#worked-alert-2-ueba-flags-an-employee). |

---

## The Base-Rate Worksheet

Do this once per detector, not per alert. Keep the result where the triage analyst can see it.

| | Your number | Example |
|---|---|---|
| Events the detector scores per day | | 1,000,000 |
| True positives per day (from incident history; if none, estimate low) | | 10 |
| Vendor's claimed detection rate (true positive rate) | | 99% |
| Vendor's claimed false positive rate (if they will give it; if not, write "refused") | | 1% |
| Expected false alarms per day = (events − true positives) × FP rate | | ~10,000 |
| Expected true alerts per day = true positives × detection rate | | ~10 |
| **Precision = true alerts ÷ all alerts** | | **~0.1%** |

If the vendor will not give you a false-positive rate, measure it: take the last 200 alerts of this type, record how many were real, and that fraction is your R1 number. It is almost always lower than the deck implied. This is Axelsson's base-rate fallacy from 1999 arriving in your queue with a new logo; see the README's [Reliability section](../../README.md#r--reliability).

---

## Worked Alert 1: Email Gateway Quarantines a Message

*Hypothetical. Names and domains are invented.*

**The alert.** An AI-based email security gateway quarantines an inbound message to the finance team, classified "Business Email Compromise — payment redirection," confidence 0.94. The alert attaches: full headers, the sending domain (`acme-invoices-llc.com`, registered 11 days ago), a screenshot of the body, the extracted bank details, and a note that the display name matches a real supplier contact while the reply-to does not.

**The walk.**

| Step | Finding | Status |
|---|---|---|
| R1 | Tuning log says this alert class ran at 71% precision over the last quarter (142 of 200 confirmed). | Known |
| R2 | No model or rule change this month. | Known |
| C1 | Finance inbox, external sender, English, invoice context. Squarely in the tuned population. | Known |
| C2 | Playbook exists: hold, notify recipient, verify with supplier by phone. Owner: the on-shift analyst. | Known |
| E1 | Headers, domain age, reply-to mismatch, bank details. The artifacts are present and the analyst can see them. | Known |
| E2 | Yes. Domain age plus reply-to mismatch plus new bank details is a BEC finding with or without the model's label. | Known |
| E3 | "Inbound invoice from an 11-day-old lookalike domain with a mismatched reply-to and new bank details, held pending supplier verification." A manager accepts that sentence. | Known |

**Decision: act.** The model's 0.94 is not why. The artifacts are why; the model found them faster than a human would have. That is the correct division of labour, and the alert design makes it possible by shipping evidence with the verdict.

**What to record.** Nothing new about the tool. Add the outcome to the precision log.

---

## Worked Alert 2: UEBA Flags an Employee

*Hypothetical. Names are invented.*

**The alert.** A user and entity behaviour analytics platform raises "Insider risk — anomalous data access," risk score 87/100, on an employee in the engineering organisation. The alert body is a paragraph generated by the platform's language-model summariser: *"The user has exhibited a pattern consistent with data staging behaviour, accessing a significantly elevated volume of repositories outside their normal working profile and at unusual hours. This activity is indicative of potential exfiltration preparation."* No access events are attached. A "view details" link opens a dashboard of the same score over time.

**The walk.**

| Step | Finding | Status |
|---|---|---|
| R1 | No precision log exists for this alert class. The platform was onboarded four months ago; nobody has reconciled its insider-risk alerts against outcomes. The vendor deck said "industry-leading accuracy." | **Unknown** |
| R2 | A connector for the source-code platform was added two weeks ago, so the "normal profile" baseline includes at most two weeks of that data source. | **Flag** |
| C1 | HR system shows the employee moved teams 19 days ago. A team move means new repositories, new hours (new team is in another time zone), and a baseline that describes the old job. | **Edge case** |
| C2 | If real, the response is an HR and legal process with a named owner. That exists. But it is irreversible for the employee once started. | Known, high cost of error |
| E1 | No artifacts. A score and a narrative. The dashboard shows the score, not the events behind it. | **Flag** |
| E2 | Not from what is attached. The analyst would need to pull repository access logs independently. | **Flag** |
| E3 | The only sentence available is the model's. "Indicative of potential exfiltration preparation" is a conclusion, not evidence, and it is the sentence that would reach HR. | **Flag** |

**Decision: verify; do not escalate.** Three independent reasons, any one sufficient: R is unknown so the score carries no calibrated weight; C says the baseline describes a job the person no longer has; E says there is nothing to act on except a paragraph a model wrote. Pull the actual access events. Compare them with the new team's repositories. If they match the new role, close the alert and record the reason. If they do not, you now have evidence and can escalate on it.

**What to record.** Three tool findings, which is more than most alerts yield:

1. **R:** this alert class has no measured precision. Open a tuning task: reconcile the last 90 days of insider-risk alerts against outcomes.
2. **C:** the platform does not consume HR change events (team moves, role changes, leave). Ask the vendor whether it can; if not, add a triage step that checks HR for every person-subject alert.
3. **E:** the alert ships a narrative instead of evidence. Ask the vendor to attach the top contributing events to every person-subject alert. Until then, set a rule: no person-subject alert leaves the SOC on narrative alone.

The README's E table has this exact row — *AI flags an employee: can you explain this to HR and legal using evidence, not the model's narrative?* — because this is the alert that ends careers when it is wrong.

---

## When the AI Is the Analyst

A growing share of SOC tooling puts a language model in the analyst's seat: a copilot that summarises an incident, drafts the ticket, proposes the containment step, or correlates alerts into a story. The five-minute card still applies, with one change of emphasis: **the model's output is the alert.**

| Ask | Why |
|---|---|
| Does every claim in the summary link back to a specific event, log line or artifact? | A summary that cites nothing is a narrative. Summaries invent indicators; a fabricated hash or IP in a ticket propagates to blocklists. |
| Is the proposed action reversible, and does the copilot know? | "Isolate host" and "disable account" are not equivalent to "add to watchlist." The model does not carry the cost of being wrong. |
| What did the copilot *not* look at? | Context windows are finite. A summary of 40 alerts built from the first 25 is confidently incomplete. Ask what was in scope. |
| Did untrusted content touch the model before it wrote this? | If the copilot read the phishing email, the attacker's text was in its context. A summary that says "this message appears benign" may be repeating the message's own instruction. See the [Runtime Monitoring guide](../TRACE-agent-runtime-monitoring.md). |

Rule of thumb from the README: a model's stated reasoning is output, not reasoning. Use the copilot to find the evidence faster. Do not let it be the evidence.

---

## Escalating to a Full Evaluation

Triage findings accumulate. When the same tool finding recurs, stop triaging it and evaluate the tool:

| Pattern | Trigger |
|---|---|
| R unknown for an alert class after 30 days in production | Run the base-rate worksheet and the `trace-evaluate` skill's Reliability section. Decide whether to keep the alert class on. |
| C edge cases are a majority of an alert class | The detector was tuned for a population you do not have. Context failure; see [README C](../../README.md#c--context). |
| E narrative-only across a product | Vendor requirement at renewal: evidence attached to every alert (see `trace-vendor`, REQ-E-001). |
| An alert class is tuned off because of noise | Record it. A silenced detector is a Reliability finding the next incident review will ask about. |
| Any alert where the question "was the model manipulated?" cannot be answered | Attack Surface review of the tool, not of the alert. |

The tool's knowledge map — the five tri-state rows, kept per product and updated from triage — is what you bring to the renewal conversation. It is also what you bring to the post-incident review when the question is "why did we trust this?"

---

## Summary

- Three dimensions in five minutes: **R** (is this detector's precision known here?), **C** (is this subject in the tuned population?), **E** (is there evidence, or a narrative?).
- **Act** on artifacts, never on confidence scores or model prose.
- **Person-subject alerts** never leave the SOC on narrative alone.
- Every triage leaves a **tool finding** or confirms none. Keep them; they are the evaluation you did not have time to run.

---

*Part of the [TRACE Framework](../../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Last updated: November 2026.*
