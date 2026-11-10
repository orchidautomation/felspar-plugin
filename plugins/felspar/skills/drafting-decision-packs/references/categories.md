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

## Worked example: coding agent actions

One coding agent policy, mapped. Each row says what the policy holds in that category, in the policy's own terms.

| Category | What it holds for coding agent actions |
| --- | --- |
| `subjects` | One action an agent is about to take. Its case ID is the tool call's own ID; the session, the repository and the change record are related records. |
| `actors` | The developer the agent works for; whoever approves change records, who must not be the requester; the reviewer who sees flagged actions; the policy owner. |
| `decision_criteria` | The rules: "Commands that only change the local working copy are allowed." "A production change with an approved record is allowed and flagged for the change review." |
| `source_signals` | The tool call itself (command, file path, working directory) and the change record looked up for it, with its valid-until date sent on its own. |
| `needs_and_requirements` | The definitions: local working copy, production, irreversible, security control, outside the company, and an approved record: approved by someone other than the requester, naming the system, environment and kind of change the action makes. An approval for staging does not cover production. |
| `evidence_and_proof` | "The branch name only says which record to look up. It is not approval." "If the record can't be found or read, there is no approval." "Still valid: its valid-until date is on or after the day of the call." |
| `boundaries` | Never delete outside the project folder, print or move credentials, or send company data to an outside service. No one approves their own action. |
| `output_contracts` | `allow`, `allow-and-flag` or `block`, with the deciding rule quoted and what's missing: "an approved change record is required." |
| `routing_jobs` | One job, "check a coding agent action"; escalations for change approval, security review and incident review. |
| `gaps` | "If it is unclear what a command does or where it lands, block it." Anything the policy leaves open, such as whether staging needs approval, goes to the owner. |
| `evaluations` | Cleaning build output is allowed; deleting outside the project folder is blocked; a production change whose record's valid-until date has passed is blocked, and so is one approved only by its requester or only for staging. |

The valid-until date is a date question compared with the decision date, not wording to judge: see [numbers-and-dates.md](numbers-and-dates.md). Rotate the systems and examples to the team's own; this example is not a template for their rules.
