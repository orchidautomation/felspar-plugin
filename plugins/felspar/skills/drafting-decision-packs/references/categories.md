# The eleven categories

Every pack maps its source into these categories. Use the exact keys. Mapping a passage organizes it for review; it doesn't by itself make Felspar enforce it.

| Key                      | Ask of the source                                        |
| ------------------------ | -------------------------------------------------------- |
| `actors`                 | Who decides, acts, owns or receives this?                |
| `decision_criteria`      | What conditions select an outcome, including exceptions? |
| `source_signals`         | Which observations and sources does this job use?        |
| `needs_and_requirements` | What facts, definitions and prerequisites are required?  |
| `evidence_and_proof`     | What supports a claim, and when is evidence eligible?    |
| `boundaries`             | What must never happen, or needs approval?               |
| `output_contracts`       | What must a produced result contain or exclude?          |
| `routing_jobs`           | Which job and destination apply?                         |
| `gaps`                   | What is unknown, contradictory or undecided?             |
| `evaluations`            | Which checks and examples define acceptable behavior?    |
| `subjects`               | What is one case about, and which records relate to it?  |

## Tag what the passage actually says

- "For a trial, return pilot_offer" is `decision_criteria` plus `output_contracts`. Add `routing_jobs` only when it names a job or destination.
- A definition, inclusion or exclusion is `needs_and_requirements`. "Synthetic excludes anonymized production records" defines a term; it doesn't ban anything. Add `boundaries` only for an explicit ban or approval condition.
- Evidence eligibility ("use records younger than 30 days") is `evidence_and_proof`, plus `source_signals` when it names sources. It becomes an `evaluations` item only with an expected behavior ("a 31-day-old record must come back as missing evidence").
- "The message must never guarantee returns" is `output_contracts` plus `boundaries`. If Felspar can't enforce that part, keep it as a gap.
- Missing authority, contradictory clauses and unclear owners are `gaps`. Name the conflict; don't rewrite the source as if it said something else.

- "A ticket is one request a customer sends to the help desk" is `subjects`: it says what a case is. "Each contract is filed under one matter" is `subjects` too: a relationship. Who reviews or approves is still `actors`; the job itself is still `routing_jobs`.
- "A held ticket goes to human review" is `routing_jobs` plus `gaps`: declare `human-review` as an escalation.

Example: "Only use the 18% result for limited pilots; never guarantee it" touches `evidence_and_proof`, `needs_and_requirements` and `boundaries`. Mapping it only as a reusable claim loses meaning.
