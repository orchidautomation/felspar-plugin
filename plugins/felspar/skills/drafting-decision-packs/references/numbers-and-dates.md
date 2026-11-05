# Numbers, dates, thresholds and scoring

Read this before writing a number or date question, or any rule that compares one. They need the pack draft format `felspar.pack-draft/v1` (preview); every v0 draft is still valid. `get_pack_guide` has the exact syntax.

## Questions

- **A number is a number question, never answer options.** Felspar reads it exactly from the agent's excerpt, with no model. Give its unit: a currency code (`"USD"`), `"percent"`, or the noun a count is of (`"claims"`, `"seats"`).
- **A count needs its unit words**, the document's own words that name it next to a number (`"unitWords": ["claims", "prior claims"]`). An excerpt saying "3 policies" is then refused for a claims question instead of misread. Each word must appear in the document.
- **A date is a date question.** Agents send what their source says; Felspar reads ISO dates, written month names and numeric dates.
- **Conventions are the document's.** If the policy writes `1.234,56` or day-first dates, set `numberStyle` or `dateOrder`. Without `dateOrder`, `03/04/2026` is refused as ambiguous. Set `timeZone` (IANA) if decisions near midnight depend on the local date; UTC otherwise.

## Thresholds

- **Quote the threshold where the document states it,** in the condition's `quote`: `{ "prior_claims": { "lt": 3, "quote": "fewer than 3 prior claims" } }`. Felspar checks that the quote states 3, in the question's unit.
- **Match the wording to the operator.** "fewer than", "under", "before" are `lt`; "at most", "no later than" are `lte`; "more than", "over", "after" are `gt`; "at least", "or more", "on or after" are `gte`. A contradiction blocks. Wording that doesn't say ("within", "up to", "between", "by") is a warning: ask the owner whether the bound itself counts, and record the answer as a gap if they can't say.
- **A range is two bounds in one condition** (`"gte": 10000, "lte": 50000`), with each side's inclusivity explicit.
- **Relative dates** compare with another date question plus days (`{ "question": "delivered_on", "days": 14 }`) or with the decision date (`{ "decisionDate": -30 }`). Months and years are refused: a month is not a fixed number of days, so ask the owner for the days. Business days are not supported yet.
- **Don't invent a threshold.** A number the document doesn't state is an owner decision: list it as a gap.

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
