# Numbers, dates, thresholds and scoring

Read this before writing a number or date question, or any rule that compares one. They need the pack draft format `felspar.pack-draft/v1` (preview); every v0 draft is still valid. `get_pack_guide` has the exact syntax.

## Questions

- **A number is a number question, never answer options.** Felspar reads it exactly from the agent's excerpt, with no model. Give its unit: a currency code (`"USD"`), `"percent"`, or the noun a count is of (`"claims"`, `"seats"`).
- **A count needs its unit words**, the document's own words that name it next to a number (`"unitWords": ["claims", "prior claims"]`). An excerpt saying "3 policies" is then refused for a claims question instead of misread. Each word must appear in the document.
- **A count is a whole number; a unit of measure may be fractional.** Hours, minutes, km, miles, kg, litres, square metres and other units of time on the clock, length, weight, volume or area take unit words too and read "7.5 hours" as 7.5. Set `"integer": true` if the policy only allows whole ones. Days, weeks, months and years are counts.
- **A count of dated entries names its anchor date.** When the policy counts entries before another date ("claims in the 12 months before the loss"), add `"countsBackFrom": { "question": "loss_date", "withinMonths": 12 }` to the count, naming the date question it counts back from. Leave `withinMonths` out when the policy counts every earlier entry. Such a count goes to a person for review unless Felspar has enabled it for the release.
- **A date is a date question.** Agents send what their source says; Felspar reads ISO dates, written month names and numeric dates.
- **Expiry, deadlines and validity are date questions too.** A record saying "approved, valid until 2026-10-01" is an approval plus a date: ask for the date (`valid_until`, type `date`) and compare it with the decision date (`{ "valid_until": { "gte": { "decisionDate": 0 }, "quote": ... } }`). Tell agents to send the bare date for it (`2026-10-01`) when it is an established date in their record. A plainly stated "valid until 2026-10-01" reads on a release using date reader v3; a release using date reader v2 holds that phrase. Both hold "valid until 2026-10-01, subject to approval". A date does not establish approval: require approval in its own question and require both in the permitting rule. Do not strip conditions from evidence to obtain a date. Keep "valid until", "expires", "still valid" and dates out of an answer's meaning, inclusions, exclusions, examples and edge cases; `check_pack` warns `date_in_answer` when one is there.
- **Conventions are the document's.** If the policy writes `1.234,56` or day-first dates, set `numberStyle` or `dateOrder`. Without `dateOrder`, `03/04/2026` is refused as ambiguous. Set `timeZone` (IANA) if decisions near midnight depend on the local date; UTC otherwise.

## Thresholds

- **Quote the threshold where the document states it,** in the condition's `quote`: `{ "prior_claims": { "lt": 3, "quote": "fewer than 3 prior claims" } }`. Felspar checks that the quote states 3, in the question's unit.
- **Match the wording to the operator.** "fewer than", "under", "before" are `lt`; "at most", "no later than" are `lte`; "more than", "over", "after" are `gt`; "at least", "or more", "on or after" are `gte`. A contradiction blocks. Wording that doesn't say ("within", "up to", "between", "by") is a warning: ask the owner whether the bound itself counts, and record the answer as a gap if they can't say.
- **A range is two bounds in one condition** (`"gte": 10000, "lte": 50000`), with each side's inclusivity explicit.
- **Relative dates** compare with another date question plus days (`{ "question": "delivered_on", "days": 14 }`) or with the decision date (`{ "decisionDate": -30 }`). Months and years are refused: a month is not a fixed number of days, so ask the owner for the days. Business days are not supported yet.
- **Compare two facts with `{ "fact": ... }`**: `{ "requested_amount": { "lte": { "fact": "approved_limit" }, "quote": ... } }`. Both must be numbers in the same unit, or both dates. On a job with `openQuestions`, a case with one side still missing is decided when either side gives the same outcome. Older packs write `{ "question": ... }`; it compares the same way but holds whenever a side is missing.
- **Don't invent a threshold.** A number the document doesn't state is an owner decision: list it as a gap.

## Numbers the document works out

- **Never ask agents for arithmetic.** When the document says a total is a quantity times a unit price, or a sum of fees, ask for each part as its own number question and add a computed question: `{ "key": "order_total", "type": "number", "unit": "USD", "meaning": ..., "compute": { "mul": ["quantity", "unit_price"] } }`. `compute` takes `add`, `sub` (exactly two) or `mul`, each item a number question's key or a constant.
- **Units must fit.** A sum or difference keeps one unit. A product keeps one question's unit, and every other question in it is a count. Keep quantity and unit price separate questions.
- **A computed number is never asked and never missing silently.** Agents send its parts; when one is missing the case holds, never computed as if it were 0. Its parts can't be asked only sometimes (`askOnlyWhen`). It takes no `minimum`, `maximum` or `integer`: set those on its parts, so a misread part holds the case.
- **Use it in rules only,** like any number: a threshold, or a comparison with another fact. It can't gate another question, be scored, or be stated by an output check: state its parts instead.
- **Test cases give its parts, not the result.** Put cases on both sides of its thresholds through the parts (3 × $19.99 is $59.97: one case at a $59.97 limit, one at $59.96).

## Output checks

- **A stated number or date names its line.** An output check that states a number or date question's answer gives a `label`, and the written output must state the value on that line: `"label": "Refund amount"` for `Refund amount: €50`. Only that line counts, so a fee or another date in the text never stands in for it. Give each stated value its own label; `check_pack` asks for a missing or repeated one.

## Tests

- **Every threshold needs two cases:** the last value that meets it and the first that doesn't, one whole number, one day or one cent apart (2 and 3 for "fewer than 3"). `propose_pack` is blocked until both exist. Take each case's expected outcome from the document.
- **A pack that compares with the decision date gives every case a `decisionDate`.**
- **Read `boundaries` in the check result:** what the pack decides at each bound and one step either side. Show the owner any edge whose outcome surprises you.

## What agents' evidence does

Report these to the person; they shape what agents must send.

- Another currency is refused, never converted. A number with no currency marker is read in the question's currency and recorded as assumed.
- An estimate ("about 480") is not exact. A bound or range ("over 500", "200-500") decides only when it settles every threshold on the question; otherwise the case is held and the agent is asked for the exact figure.

## Scoring

- **Use a `scoring` block only when the document adds up points.** Each criterion quotes its meaning; each contribution quotes its points and its conditions. Contributions of one criterion must not overlap.
- **Write decisions as score bands and gates.** A band is a rule with a quoted `score` bound ("40 points or more" is `gte: 40`); a gate (`gate: true`) decides on facts and wins over any score ("hold any restricted use, whatever the score").
- **Pin each case's total** (`expect.score`) from the document's own arithmetic, and a criterion's points (`expect.points`) when the document spells them out. A missing fact an active criterion needs withholds the total and holds the case, gates included: tell the owner, since a test case can't expect that.
- **Test both sides of every band edge** at the nearest totals the criteria can reach. Report totals no band covers and bands no total reaches to the owner as gaps.
