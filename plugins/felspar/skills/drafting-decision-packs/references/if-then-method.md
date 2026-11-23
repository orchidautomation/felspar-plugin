# Writing each rule as if/then

Felspar's drafting method: every rule in the pack is one sentence of the form **if these stated facts, then this outcome**. The facts come from records, the reader reads them, and the pack's own rules do every comparison. Use it on every paragraph that decides something, alongside [turning policy into questions](writing-questions.md).

## The method

1. **Write the document's rule as if/then, in its own words.** "If the customer has an active bankruptcy filing, then refer the case to the legal team." Each `if` part becomes a question; the `then` part becomes an outcome. A sentence that hides several rules ("…provided the account is not more than 60 days past due and no deferral has been granted in the last 12 months") is several if/then lines, one per way the case can end.
2. **Make each `if` part a fact a record states.** A status, a type, an amount, a count, a date. Never the decision itself ("eligible", "qualifies", "entitled"): `check_pack` warns `judgment_question` on those words.
3. **The reader answers in two steps.** First whether the records state an answer at all, then which answer. So a question never needs "unknown", "not stated" or "none" answers: when no record states the fact, the question stays open.
4. **Only the records count.** The reader answers from what the records the agent sent say, read the way the staff who use that system read them, never from what usually happens or from what a record leaves out.
5. **Numbers, dates and counts are read exactly.** The reader only says which figure, date or count in the records is the one the question asks about; Felspar reads its value. Ask for the parts a record states and let the pack work out the rest: `compute` with `add`, `sub`, `mul` or `min` (a computed number may use one declared before it), date bounds, and comparisons with another fact.
6. **Put the limit in the rule, quoted.** `{ "days_past_due": { "gt": 60, "quote": "more than 60 days past due" } }`. Felspar checks that the quote states the number.
7. **Order the rules when the document does.** "Apply these checks in this order, and the first that applies decides the case" is `"ruleOrder": "first-match"`: the first rule that matches decides.
8. **Say what happens when a fact is missing.** A rule that needs a fact no record states can't match, and when no rule can decide, the case is held: the result names each open question, asks the agent for the record `whereToLook` points to, and the case follows `whenUnresolved`. With `openQuestions`, a case is still decided when the missing facts can't change the outcome.
9. **Mark the outcomes that go against the case.** An outcome with `"adverse": true` (a decline, a denial, a referral) goes to a person instead of acting, unless the document says it may act unattended.

## Worked example: a payment deferral

> A customer may defer up to two monthly payments when they report a financial hardship, such as job loss or a medical emergency, provided the account is not more than 60 days past due and no deferral has been granted in the last 12 months. Deferrals totalling more than $2,000 require a supervisor's approval. If the customer has an active bankruptcy filing, do not offer a deferral; refer the case to the legal team.

As if/then lines, in the order the document applies them:

| If | Then |
| --- | --- |
| the bankruptcy search lists an open case | refer to legal (adverse) |
| a hardship is reported and the account is more than 60 days past due | decline (adverse) |
| a hardship is reported and a deferral was granted in the last 12 months | decline (adverse) |
| a hardship is reported and the deferral total is more than $2,000 | supervisor approval |
| a hardship is reported and the deferral total is $2,000 or less | approve |

The facts each line reads:

| Fact | Question | Where a record states it |
| --- | --- | --- |
| `hardship_reason` | choice: _job loss_, _medical emergency_, _other hardship_ (`noneOfThese`) | the call note |
| `bankruptcy_status` | choice: _active filing_ / _no active filing_, `highConsequence` | the bankruptcy search |
| `days_past_due` | number, a count with `unitWords` | the account screen |
| `monthly_payment` | number, USD | the account screen |
| `last_deferral_on` | date | the account screen |
| `payments_requested` | number: how many payments the call note asks to defer | the call note |
| `payments_deferred` | computed: `{ "min": ["payments_requested", 2] }` | never asked |
| `deferral_total` | computed: `{ "mul": ["payments_deferred", "monthly_payment"] }` | never asked |

"In the last 12 months" is a calendar-month bound on the decision date: `{ "last_deferral_on": { "gte": { "monthsFromDecisionDate": -12 }, "quote": "…12 months…" } }`. "Up to two" is `min`, and the total is worked out from it, never asked.

The document didn't say everything the pack needs, so the owner added a short "Applying the rule" section through the edit loop: the order the checks apply in, what "the last 12 months" counts from, how the deferred payments and the total are worked out, what a total of exactly $2,000 gets, and what happens when a fact is missing. Propose those lines; never assume them.

**How a case decides.** A call note says "Customer lost her job on Oct 3. Asking to skip the November and December payments." The account screen shows 34 days past due, a $1,150 monthly payment and a last deferral on 2025-04-10, and the bankruptcy search found no filings. The reader finds job loss and two payments in the note, and the screen's figures and date; the pack works out 2 × $1,150 = $2,300, and the first matching rule sends it to a supervisor. The result names the reader, each fact with the words it came from, and every rule that matched with the deciding one marked. Its receipt replays with no model.

**How a case holds.** A call note that only says "Member called about the car loan." states no hardship, so `hardship_reason` stays open, no rule matches and the case is held: the result asks for the hardship reason and quotes where to look ("Look for the hardship reason in the call note.").

## Counting what a record lists

When a question counts things, the reader says how many the record asks for or describes ("skip the November and December payments" is 2). When a record lists dated entries or writes a total, Felspar reads or counts them exactly. Test both: a note that names the months, and one that writes the number.

## Eval cases send what the agent looked at

An eval case is a request an agent would send. When the agent looked at a record that doesn't hold the answer (an account screen without the field, a note that never mentions it), the case still sends that record, and Felspar holds the case as it would a live one. A case that sends nothing at all for a question the pack asks on every case is refused before anything is read (`required_evidence_missing`); `save_evals` and `check_pack` warn `eval_case_evidence_missing`, naming the case and the question.

## Checks that catch a rule written wrong

- `judgment_question`: a question asks for the decision ("eligible", "qualifies", "entitled", "on time").
- `compound_question`: one question asks two things ("and without", "but not").
- `limit_in_question`, `math_in_question`: a limit or a sum sits in the question instead of the rule.
- `decided_share_below_bar`: the newest eval run decided fewer than 80% of the cases expecting an outcome as expected. It never blocks a release; fix the wording of the held and wrong cases.
