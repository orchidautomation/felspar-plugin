---
name: writing-pack-test-cases
description: Writes and maintains test cases for a Felspar Decision Pack - one per outcome, both sides of every threshold, each exception and missing evidence - preferably from real past decisions, plus reading tests that check how Felspar reads different phrasings of each answer, on made-up wording and, with the person's agreement, a small sample of their real records, and agent-shaped eval sets of realistic multi-excerpt cases run against every candidate before release. Use this skill when someone wants to add, fix or review a pack's tests or examples, turn past decisions into tests, test how real wording is read, or check that a policy's edge cases are covered, including when a backtest or a live decision looks wrong or real cases keep being held. Not for drafting the pack's rules (drafting-decision-packs) or approving it.
compatibility: Requires the Felspar connector (list_jobs, read_pack, read_source, add_current_version, check_pack, check_readings, propose_pack, read_evals, save_evals, run_evals, list_eval_runs, read_eval_run) and a person signed in as a workspace member, approver or owner.
metadata:
  author: felspar
  version: "0.3.3"
---

# Writing pack test cases

A test case is evidence for one case plus the outcome the policy says it should get. Tests are how a reviewer trusts a pack, and how the next change is checked against today's behavior. Write them so each one would catch a real mistake.

## Coverage checklist

- [ ] Every outcome has at least one case.
- [ ] Every threshold has a case at the last value that meets it and one at the first that doesn't, including business-day windows (the last business day inside, the day after) and comparisons with another question (the amount equal to the limit, and one cent above) (day 14 and day 15 for "within 14 days" when day 14 counts; $500 and $500.01 for "up to $500"). `check_pack` blocks a proposal until both exist, and its `boundaries` show what the pack decides at each edge.
- [ ] Every passage the draft's `coverage` marks `represented` by a rule has a case that rule decides; a `needs_review` or `unsupported` passage gets no case until the owner settles it (for a refund policy's "managers may waive the restocking fee", no case expects a waived fee).
- [ ] Every exception has a case where it applies and one where it almost does. A case where an exception applies expects the exception's rule: in a claims pack where `"water-damage"` names `"except": ["flood-zone"]`, a flood-zone water-damage claim expects `{ "outcome": "adjuster-visit", "rule": "flood-zone" }`. Never write a case that only a contradiction `check_pack` reported would decide: that is the owner's gap.
- [ ] In a scored pack, every case states its total (`expect.score`), and every score band has cases at the nearest totals inside and outside its edge.
- [ ] Held cases are required: at least one where evidence is missing and the pack must not decide, and one with stale evidence when the document limits evidence age. When one question has its own age (`evidence.maxEvidenceAgeDays`), the stale case is about that question: in a claims pack whose adjuster estimate may be 14 days old, `{ "claim_amount": { "stale": 500 } }` expects `{ "held": "stale" }`. Add a conflict case where two sources can disagree. `check_pack` warns (`no_held_case`) when a pack has none.
- [ ] Where answer options classify text, at least one case has text that should pass and one that should fail.
- [ ] Real past decisions the person supplies are included, each with what actually happened.
- [ ] When the policy names an escalation for an outcome, a case with that outcome asserts it (`expect.escalation`).
- [ ] A case about a particular record gives its ID as the job's subject writes it (`subject: { id }`, lowercase for a case-insensitive subject).
- [ ] The job has an eval set built to break it: raw, messy records mapped to each requirement key the way an agent would, with separate excerpts per key, every outcome, at least one expected hold and one conflict, all made up. Never one blob, and never a policy example reused as a case. Blind where your client allows it. See [eval sets](references/eval-sets.md).
- [ ] Every case from the owner's stress table is covered, reworded, with the owner's answer as its expected result.
- [ ] Every confirmed case from a case interview (`preparing-policy-documents`) is in the eval set, paraphrased, with the outcome the expert confirmed.
- [ ] Every answer of every choice question has reading tests: plain, indirect, past-tense or dated, and a near miss that belongs to another answer, read with `check_readings` and every miss folded back.

## Held cases

A held case gives one fact as a held reading and expects `held` instead of an outcome and a rule. Pick the vertical's own example; these rotate:

```json
{ "id": "claim-date-missing", "why": "<quote: a claim needs the incident date>",
  "facts": { "claim_type": "water-damage", "incident_on": { "missing": true } },
  "expect": { "held": "missing" } }
{ "id": "old-headcount", "why": "<quote: evidence older than 30 days doesn't count>",
  "facts": { "employees": { "stale": 240 } }, "expect": { "held": "stale" } }
{ "id": "two-regions", "why": "<quote: the region is the one on the signed order>",
  "facts": { "region": { "conflict": ["emea", "apac"] } }, "expect": { "held": "conflict" } }
```

A number or date conflict needs two readings the pack decides differently, one on each side of a threshold: readings every threshold decides alike agree, so the case decides and `check_pack` refuses it. `{ "review": <value> }` holds a reading its reader flagged, expecting `{ "held": "review" }`. A held case that a rule decides fails in `check_pack` with a `problem` naming the rule: the rule decides without the fact it needs. Fix the rule, not the expectation.

## Workflow

1. Read the pack's questions (the draft in progress, or `read_pack` for a released job) and the source passages they quote (`read_source`).
2. Ask the person for real past cases first; they find gaps made-up cases miss. Turn each into an anonymized case that keeps what it teaches, and write made-up cases for coverage the real ones lack. If `preparing-policy-documents` ran a case interview, start from its confirmed cases: they are real decisions the expert walked through and confirmed. Give each one an ID starting `confirmed-`, paraphrase it with invented names and numbers, and use the confirmed outcome as the expected result. A case the expert marked as a past exception expects what the policy says now, and its `why` says it was an exception.
3. Write each case's evidence as the source would say it (an email line, a ticket note, a contract clause, a tracking record), with its source and date, never as a label.
4. State the expected outcome from the policy, independently of what the draft currently returns. Add the case to the pack draft's test cases.
5. Run the tests with `check_pack`. For each failure, decide with the person whether the rule or the expectation is wrong. Fix that one; never weaken a test to make it pass.
6. Test the wording: first run `check_pack`. A `thin_answer` finding (fewer than 3 examples with one written the way a real record is, 2 exclusions, 2 inclusions or 1 edge case on a deciding question) is the next step: close it from its `fixes` in the document before any reading, since `check_readings` refuses a draft with blocking findings. Then write phrasing variants for each answer, read them with `check_readings`, fold each miss back into the document, and keep the useful ones as `readingTests`. Offer to test on a small sample of the person's real records, explaining first and reading nothing until they agree. Read [reading tests](references/reading-tests.md) first.
7. Write the job's eval set from raw records mapped per requirement key, and `save_evals` it; `run_evals` against the draft (or the proposal or change request), report each case as a decision, and fold each miss back into the document, never into the case. A wrong decision blocks release. Reading tests never replace this step. Read [eval sets](references/eval-sets.md) first.
8. When the tests pass and the readings are clean, or each miss left is listed as a gap, `propose_pack` so the new cases go to review with the pack. For a released job, `preview_change` first and ask the person whether they're OK with it, as `drafting-decision-packs` does.

## Reading tests and real evidence

Test cases send answer values; reading tests send wording, the way agents will. The rules that never bend:

- **A held or misread phrasing is a miss, never a pass.** Fix the definition in the document (an example, an exclusion or a sharper meaning, from the document's own words where it has them) or raise it to the owner. Never change a phrasing's expected answer to what it was read as, reword it until it passes or ask for a lower bar.
- **Explain, then ask, before reading any real record.** Say why it matters, which source and how many records, what Felspar sees and keeps, and that they can narrow or skip it; then ask "OK to test on about N recent …?". Nothing is searched, opened or counted until they say yes. Never ask for credentials or connect anything.
- **Real records stay theirs.** Excerpts go only to `check_readings`, read once and kept nowhere. Nothing from a record, verbatim or identifying (names, companies, emails, ticket, order, deal or account numbers), goes in the draft, its test cases or reading tests, a suggested edit or a proposal; keep the anonymized lesson instead. Every real miss becomes a suggested document edit, never a direct pack edit.

## Gotchas

- **A past exception stays an exception.** If a real case was approved against the written policy, mark it as an expected exception and tell the owner; don't rewrite the rule to match it.
- **Dates are relative to the case.** Set each case's evidence date to when the source said it; a "14 days after delivery" case needs both dates. In a pack draft, give numbers as numbers and dates as `YYYY-MM-DD`; a pack that compares with the decision date ("in the last 30 days") gives every case a `decisionDate`.
- **Tests stay synthetic.** The proposal keeps the draft, so a real past case becomes a paraphrased case with invented names, companies and numbers: keep what it teaches, not who it was about. The same holds for reading tests.
- **A confirmed case's outcome comes from the expert, not the draft.** If the pack decides it differently, the rule or the wording is wrong, or the expert's answer needs a second look with them; never change the expectation to match.
- **The draft's own output is not the expected outcome.** Copying what the pack returns into the expectation makes the test pass by definition.

## Next actions

End every reply with where the person is and the one next thing: a question with your recommended answer, or an action, and who does it. Blockers first. Common situations:

| Situation | Next step | Who |
| --- | --- | --- |
| An outcome has no case | Write one from a real past case, or a synthetic one if there is none | You, with the person |
| A threshold lacks a case on one side | Add the last value that meets it and the first that doesn't | You |
| No held case (`no_held_case`) | Add a missing-evidence case, and a stale one if the document limits evidence age | You |
| A held case was decided | Find which fact the deciding rule should need; fix the rule with the person | The person |
| A case is dated outside the business-day calendar (`case_outside_calendar`) | Date it inside the calendar's years, or ask the owner for the next year's holidays | You, or the owner |
| A test fails | Decide with the person whether the rule or the expectation is wrong; fix that one | The person |
| A real past case contradicts the policy | Mark it an expected exception and tell the owner | The owner |
| The policy doesn't say what a case should get | It's a gap: hand it to `drafting-decision-packs` as an owner decision | The owner |
| A case interview handed over confirmed cases | Paraphrase each as a `confirmed-` eval case with the expert's outcome; add made-up cases only for what they don't cover | You |
| No real cases supplied | Ask for two or three recent ones before writing synthetic cases | The person |
| A reading test is held or read as another answer | Suggest an example, exclusion or sharper meaning for the document; read again after the edit | You, then the owner |
| The job's evidence sources are reachable | Explain, then ask "OK to test on about N recent …?" before reading anything | The person |
| A source can't be reached, is off-limits or the person skips | Say which questions are tested only on made-up phrasings | You |
| `check_pack` reports a `thin_answer` | Close it from its `fixes` in the document before writing reading tests | You, then the owner |
| Every answer lacks reading tests | Write the four shapes for each answer and run `check_readings` | You |
| The job has no eval set | Write one from its evidence kinds: every outcome, a hold and a conflict; `save_evals`, then `run_evals` | You |
| `run_evals` lists `newMisses` | Report them first; trace each to the definition or rule and suggest the document edit | You, then the owner |
| An eval case comes back `refused` | Fix the case (a question or source class the pack doesn't take), not the pack | You |
| Tests pass and readings are clean | `propose_pack` so the cases go to review with the pack | You |
