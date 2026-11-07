# Bypassing Cylance's AI Malware Detection (ATLAS AML.CS0003)

On 18 July 2019, researchers at Skylight Cyber published ["Cylance, I Kill
You!"](https://skylightcyber.com/2019/07/18/cylance-i-kill-you/), showing that appending a fixed set
of strings taken from an online game to a malicious executable caused Cylance's AI-based antivirus
to score it as benign. The bypass worked on all of the top 10 malware families for May 2019 and on
83.59% of a 384-file malware sample, rising to 88.54% when the strings were appended repeatedly. An
update on 7 September 2019, tied to a BSides Sydney talk, named the game (Rocket League) and stated
the bypass still worked on the home edition (Cylance SmartAV); the vendor told Skylight the
enterprise edition (CylancePROTECT) was fixed, which Skylight could not verify. MITRE records this
as an exercise in [ATLAS AML.CS0003](https://atlas.mitre.org/studies/AML.CS0003), dated 7 September
2019 (the update date, not the original post).

## What happened

Technique IDs are from the ATLAS 2026.10 release of AML.CS0003.

1. **Reconnaissance, AML.T0000 Search Open Technical Databases.** The researchers read public talks
   and Cylance patent filings describing the detector, which disclosed that it was an ensemble of
   models.
2. **AI Model Access, AML.T0047 AI-Enabled Product or Service.** They had ordinary access to the
   shipping product.
3. **Discovery, AML.T0063 Discover AI Model Outputs.** They enabled the product's verbose logging,
   which exposed per-file scores and details of reputation scoring and model ensembling. Skylight
   reports a one-byte-modified Mimikatz scoring -852.
4. **Resource Development, AML.T0017.000 Adversarial AI Attacks.** Using those scores, they reverse
   engineered which attributes moved reputation up or down and found a secondary mechanism that
   could override the core model. Skylight describes it as comparing a file's feature vector against
   pre-calculated centroids: "Effectively, it tries to find if the executable is very similar to
   ones that were added to the model's white/black list." One centroid was named after an online
   game.
5. **AI Attack Adaptation, AML.T0043.003 Manual Modification.** They extracted strings from the
   game's main executable (about 5 MB, later cut to about 60 KB) and concatenated them onto malware.
6. **Defense Evasion, AML.T0015 Evade AI Model.** Because the secondary model's positive verdict
   overrode the primary, the modified malware was scored as benign. Mimikatz moved from -852 to 999.

| Test set (Skylight) | Result |
|---|---|
| Top 10 malware, May 2019 | All 10 moved from negative to positive scores ("100%") |
| 384 malicious files, strings appended once | 83.59% bypassed detection; average score -920 to 630 |
| 384 malicious files, strings appended repeatedly | 88.54% marked benign; average score rose to 750 |

## TRACE walk

| Dimension | What the defender knew or could have known at the time | What the incident revealed |
|---|---|---|
| **T** Training | **Unknown.** Customers had no visibility into which files built the model or its override reference set. Patents and talks described architecture, not data. | The override reference set carried a strong bias toward one game's executable. Skylight calls it "a peculiar bias towards a specific game." How the centroids were built is not described in the source (unverified). |
| **R** Reliability | **Partial.** Vendor detection rates were public marketing claims; no customer could measure the false negative rate against adaptive input. | A single, cheap, universal transformation flipped the verdict on most malware, and nothing in the product signalled that it was being fooled. The failure was silent. |
| **A** Attack Surface | **Partial.** Evasion of ML classifiers was a known research topic in 2019, so the threat class was knowable. The specific override path was not. | The second model was a bypass that sat above the first. Attacking the override was easier than attacking the core classifier, and verbose logging gave the attacker a scoring oracle on their own machine. |
| **C** Context | **Partial.** Buyers knew whether this product was their only malware control or one layer among several. | Where it was the sole control, one string list removed endpoint malware detection. Skylight's own conclusion was that pure-ML products need verification by established techniques as defence in depth. |
| **E** Explainability | **Known.** The verbose log mode was a product feature that showed scores. | The same score output that let an analyst see why a file was judged was the instrument the attackers used to tune the bypass. |

## Which dimension broke

The failure lived in **Attack Surface**, rooted in **Training**. The core classifier was not the
weak point. The product carried a second, list-like mechanism whose positive verdict overrode the
model, and that mechanism encoded a bias the attackers could reach with plain string concatenation.
That is an Attack Surface failure (an evasion path with an override that had more authority than the
detector it protected) whose root cause sat in what the system had learned to trust.

**Reliability** broke as a consequence: the system failed silently and universally rather than
degrading gracefully on hard samples. A customer watching detection volumes would have seen nothing.

## Would TRACE have surfaced it

Partly. The Attack Surface question in the [quick reference](../../docs/TRACE-quick-reference.md) is
the one that applies:

> *Can an attacker craft inputs to evade detection?*

And the README's Reliability row:

> *Silent degradation is worse than loud failure*

Asked before deployment, those questions point a buyer at the right conversation: "Does any
component override the core model's verdict, and has that override been red-teamed?" and "Is this
our only malware control?" A vendor answer of "the model is robust" is a red flag under TRACE's own
framing. TRACE would not have found the Rocket League centroid. No buyer-side questionnaire would.
What TRACE would have done is force the Context decision (layer it, do not rely on it alone), which
is the control that would have limited impact.

| TRACE question | Would it have flagged this? |
|---|---|
| A: craft inputs to evade detection | Yes, as a class. The specific override was not discoverable by a buyer. |
| R: how would I know it is failing | Yes. The honest answer was "you would not," which is a finding. |
| T: does the vendor disclose data sources | Yes, but the answer would have been no, with nothing to act on. |
| C: what's the fallback if this tool is unavailable | Yes. This is where a defender could actually reduce exposure. |

## Validity note

One aspect did not fit cleanly: the verbose scoring output is an **Explainability** feature and also
an **Attack Surface** oracle. TRACE treats explainability as something to maximise, and has no place
to record that the same transparency helps an attacker tune inputs. Everything else landed in a
dimension without forcing.

## Sources

- HIGH: MITRE ATLAS, AML.CS0003, release 2026.10, read in full from the [atlas-data
  repository](https://github.com/mitre-atlas/atlas-data). Web view:
  https://atlas.mitre.org/studies/AML.CS0003
- HIGH: Skylight Cyber, "Cylance, I Kill You!", 18 July 2019, with 7 September 2019 update.
  https://skylightcyber.com/2019/07/18/cylance-i-kill-you/
- Not used: "Statements from Skylight Cyber CEO", the second ATLAS reference.
  https://www.security7.net/news/the-new-cylance-vulnerability-what-you-need-to-know redirects to a
  page that returned HTTP 404 in November 2026.

*Part of the [TRACE Framework](../../README.md) by [Melissa Bischoping](https://www.linkedin.com/in/mbischoping/). Licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).*

