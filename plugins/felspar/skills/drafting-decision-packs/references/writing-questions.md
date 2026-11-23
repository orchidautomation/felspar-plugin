# Turning policy into questions

Felspar decides a case from small facts that a record states, and from your pack's rules, which do every comparison, count and combination. Write the rules as if/then lines first ([the if/then method](if-then-method.md)); this page turns each `if` part into a question. Write each question so that a record either states its answer or doesn't. `check_pack` warns when a question asks for more (`judgment_question`, `limit_in_question`, `math_in_question`, `compound_question`, `agent_conclusion`, `overlapping_answers`); `check_readings` shows how real wording is read.

## The method, one paragraph at a time

1. **Split the paragraph into the smallest facts a record can state.** A date, an amount, a count, a status, a type, a named document. "Loans above $25,000 need two years of verified employment" is three facts: the loan amount, when employment started, and whether the employment record says it was verified.
2. **Word each fact as "what does the record state about X".** The question's meaning and every answer describe what text says ("the employer's letter confirms the start date"), never whether something is true, adequate or likely. An answer is never an opinion, a risk level or a verdict.
3. **Leave "is it stated?" to Felspar.** When no record states a fact, Felspar leaves that question open and the case follows `whenUnresolved`. So never add "unknown", "not stated" or "none" answers, and never fold "if stated" into an answer. Add a `noneOfThese` answer only for text that clearly states something outside the list.
4. **Numbers and dates are their own questions.** An amount, a count, a percentage or a date is a `number` or `date` question. The limit goes in the rule (`{ "loan_amount": { "gt": 25000, "quote": "above $25,000" } }`), never in a question or answer.
5. **Put the arithmetic in the pack.** A ratio, total, difference, cap or "within N days of" is worked out by the pack: `compute` for sums, differences, products and `min` for "up to N", a date bound for windows (`{ "question": "received_on", "days": 30 }`, `{ "decisionDate": -730 }` or `{ "monthsFromDecisionDate": -12 }`), a comparison of two facts (`{ "fact": "limit" }`), `countsBackFrom` for dated counts. Ask for the parts the record states, never for the result.
6. **Put the logic in the rules.** "And" is several keys in one rule's `when`. "Or" is a list of answers, or two rules to one outcome. "Unless" and "except" are an exception rule named in `except`. "At least two of these" is scoring with a band. Precedence the document states is rule order or `except`.
7. **Ask a question only where it can change the outcome.** Gate it with `askOnlyWhen` on an earlier recorded fact, never on a judgment.
8. **Keep agent conclusions out.** Questions, answers and examples describe the source record (a form, a letter, a log), never what an agent, assistant or reviewer concluded about it. Tell agents to send excerpts from the record itself.

Each answer still needs its definition, examples, inclusions, exclusions and an edge case quoted from the document (`thin_answer` otherwise), and no two answers may fit one text.

## Worked examples

### Lending

> Personal loans above $25,000 require two years of verified employment and a debt-to-income ratio no higher than 40%, unless a co-signer with a credit score of at least 700 is on the application.

Not this: one question, "Does the applicant qualify for a large loan?", answered _qualifies_ / _does not qualify_. It asks for the decision itself, hides two limits and an exception, and an agent's summary ("applicant looks solid") reads as an answer.

Instead:

| Fact | Question |
| --- | --- |
| `loan_amount` | number, USD: the amount on the application |
| `employment_start` | date: the start date the employment record states |
| `employment_verification` | choice: _employer-confirmed_ (a letter or call log from the employer) or _self-reported_ (only the applicant's form) |
| `monthly_debt`, `monthly_income` | numbers, USD, as the credit report and pay stubs state them |
| `debt_limit` | computed: `{ "mul": ["monthly_income", 0.4] }` |
| `co_signer` | choice: _co-signer listed_ / _sole applicant_, from the application's signature block |
| `co_signer_score` | number, asked only when `co_signer` is _co-signer listed_ |

Rules: `large-loan-refer` when `loan_amount` is over 25,000, with two exceptions that win over it: `large-loan-eligible` when verification is _employer-confirmed_, `employment_start` is on or before `{ "decisionDate": -730 }` and `monthly_debt` is at most `{ "fact": "debt_limit" }`, and `co-signer-eligible` when `co_signer_score` is at least 700. A missing credit score or pay stub holds the case instead of failing it. Ask the owner to state two years as 730 days.

### Healthcare

> Lumbar spine MRI is covered without prior authorization when the patient has completed at least six weeks of conservative therapy, or when the referral documents a red-flag symptom such as bowel or bladder dysfunction.

Not this: "Is the MRI medically necessary?" (a judgment), or answers such as _therapy over six weeks_ (a limit inside an answer).

Instead: `imaging_ordered` (choice: _lumbar MRI_, and `noneOfThese` for other imaging); `therapy_status` (choice: _completed_ / _in progress_, as the therapy notes state); `therapy_start` and `therapy_end` (dates); `referral_symptom` (choice of the red-flag symptoms the document lists, plus _pain or stiffness only_, which says what the referral lists instead of what it lacks), asked only when `therapy_status` is _in progress_. Rules: `therapy-route` when `therapy_status` is _completed_ and `therapy_end` is at least 42 days after `therapy_start`; `red-flag-route` for any red-flag answer. Both lead to the same outcome, which is how the document's "or" is written.

### Freight

> Each carrier shall, upon receipt in writing of a proper claim [...] acknowledge the receipt of such claim in writing to the claimant within 30 days after the date of its receipt by the carrier unless the carrier shall have paid or declined such claim in writing within 30 days of the receipt thereof.

Not this: "Was the claim acknowledged on time?" with _on time_ / _late_. "On time" is a judgment, the 30 days sit inside the question, and the "unless" is a second rule folded into the same answer.

Instead: `claim_form` (choice: _written claim_ / _phone or verbal only_); `received_on` (date, as stamped on the claim); `acknowledged_on` (date of the written acknowledgment); `disposition` (choice: _paid_ / _declined_ / _still open_, as the claim file states it); `disposed_on` (date, asked only when `disposition` is _paid_ or _declined_). Rules: `acknowledged-in-time` when `acknowledged_on` is on or before `{ "question": "received_on", "days": 30 }`; `disposed-in-time`, for a _paid_ or _declined_ disposition with `disposed_on` on or before the same bound, leads to the same outcome; `acknowledgment-overdue` when the acknowledgment came after it and no disposition did. A claim file with no acknowledgment date is held for a person, which is what the policy needs.

### Support

> Annual-plan refunds are approved automatically within 14 days of purchase if fewer than 3 seats have been used; otherwise the request goes to billing.

Not this: "Is the customer eligible for an automatic refund?", or an answer _used fewer than 3 seats_.

Instead: `plan_type` (choice: _annual_ / _monthly_), `purchased_on` (date), `seats_used` (number, unit seats, with `unitWords`). Rules: `auto-refund` when `plan_type` is _annual_, `purchased_on` is on or after `{ "decisionDate": -14 }` and `seats_used` is under 3; `to-billing` for annual plans otherwise. Test 13, 14 and 15 days, and 2 and 3 seats.

### Insurance

> Claims over $5,000, or from a policyholder with two or more prior claims in the 12 months before the loss, go to a senior adjuster.

Instead of "Is this a high-risk claim?": `claim_amount` (number, USD), `loss_date` (date) and `prior_claims` (number, `countsBackFrom` `loss_date` within 12 months, as the document states it). Two rules lead to `senior-adjuster`.

## Wording each answer

- **Describe what the text says, positively.** Not "no referral is recorded" but "the referral lists only pain or stiffness". When one record settles a fact, name it ("the verification field shows N or is blank"); an answer that needs every record to be silent holds cases the document would decide.
- **Say whose event it is and whether it happened.** "Lost their job" also matches "my husband lost his job" unless an exclusion says otherwise.
- **Put hedges in exclusions.** "Possible", "suspected", "pending inspection" and "per caller, unconfirmed" go in the exclusions of the answer they would otherwise be read as.
- **At most six answers, and no two that fit one text.** More usually means two facts in one question (`many_answers`).
- **Write examples the way records are written:** lowercase, shorthand, a bare status line. Never copy wording from a reading test or eval.
- **Never put IDs in the wording.** Only the text is read.
- **Quote the sentence that asks the question** as its `meaning`, not a heading.
- **A source limit goes in `evidenceKinds`,** not `whereToLook`: Felspar refuses an excerpt sent under a kind the question doesn't list (`source_limit_not_enforced`).
- **When the document says which record wins,** declare `prevails` with its withdrawal check, as `get_pack_guide` shows. Never pick a winner the document doesn't state.

When a quote itself carries the problem (a limit or an "unless" in the sentence the question must quote), quote a narrower passage, or propose a document line that states the fact on its own through the edit loop.

## Test both sides

Before proposing, `check_readings` each answer with plain, indirect, dated and near-miss phrasings, and text from the wrong person or with a hedge. A phrasing that is held or read as another answer is a wording fix in the document, never a lower bar.
