# Reading tests: how each answer might be phrased

Test cases send answer values, so they never show how Felspar's evaluator reads the wording an agent sends. A pack can pass `check_pack` and still hold half its live cases because "launched in September" or "shipped it back on the 3rd" doesn't read as the answer its definition means. Reading tests find those misses before the pack is proposed. Run them once `check_pack` is clean; `check_readings` refuses a draft with blocking findings (`readings_blocked`).

## 1. Write phrasing variants

For each answer of each choice question (number and date questions are read exactly and need none), write a few phrasings as the job's evidence would say them: a ticket note, an email line, a contract clause, a status field. Give each the answer a person would pick (`expect`).

| Shape | What it tests | Example: a refund job's "What happened to the item?" |
| --- | --- | --- |
| Plain | The definition's own words | "The customer returned the jacket." (`returned`) |
| Indirect | The meaning without the key words | "Label used, parcel scanned at the depot." (`returned`) |
| Past-tense or dated | A finished event with a date, not the present tense the definition may assume | "Sent it back on the 3rd, tracking attached." (`returned`) |
| Near miss | Wording that looks like this answer but belongs to another | "Customer printed a return label but still has the item." (`not_returned`) |
| Fragment | A heading, status line or one-word field | "Status: RMA received" (`returned`) |

- **Near misses come from the definition's exclusions** and from neighbouring answers. Every answer gets at least one, with `expect` set to the answer it really belongs to.
- **Rotate the shapes, not the facts.** Two phrasings that differ by one word test the same thing.
- **Stay synthetic.** Invented wording, no real names, companies or identifying numbers. A real record's excerpt is never written as a variant; see [Testing on real evidence](#5-testing-on-real-evidence).
- **Caps:** at most 8 per answer (per question when `expect` is left out) and 96 per check, the draft's `readingTests` included. Pick the phrasings most likely to miss rather than filling the cap.

## 2. Read them

Call `check_readings` with the `source_object_id`, the checked draft and the phrasings as `variants` (`{ id, question, text, expect, label }`; label each with its shape, such as "dated" or "near miss"). The draft's own `readingTests` are read too. No wording comes back, only your IDs, so keep your own list to match them.

| `result` | What it means | What to do |
| --- | --- | --- |
| `as_expected` | Read as the answer a person would pick, above the bar | Keep it if it covers a shape the others don't |
| `different` | Read clearly as another answer (`picked`) | A miss: fold it back |
| `held` | A live case would be held for a person. `reason`: `below_bar` (leaned toward `leaning`; `confidence` says `borderline` or `low`), `cannot_tell`, `tie`, or `looks_like_instructions` | A miss, never a pass. For `looks_like_instructions`, check the phrasing reads like real evidence before treating it as a gap |
| `read` | No `expect` given | Show the person what it was read as and ask which answer it should get |
| `not_read` | Not read now (`evaluation_off`, `limit_reached` or `not_answered`) | Untested, never a pass. Say which phrasings weren't read; read them again later |

A `status` of `incomplete` means some phrasings weren't read. Report each miss in plain words: the phrasing's shape and the answer it should get, and what it was read as or leaned toward ("the dated return leaned toward 'returned' but not clearly enough to decide"). Never mention cost or usage; evaluation is included and has a daily limit.

## 3. Fold misses back into the document

A miss means the definition doesn't cover that wording. Fix the meaning, not the test, and fix it where meaning lives: the document.

| What the miss shows | The fix to suggest |
| --- | --- |
| Plain, indirect or fragment held | An **example** in the definition, in the document's own voice |
| Past-tense or dated held | A **sharper meaning**: say whether a finished or dated event counts ("A return counts once the parcel is scanned, whenever that was") |
| Near miss read as this answer | An **exclusion** on this answer, and an example on the answer it belongs to |
| `tie`, or `different` between two answers | A **sharper meaning** that says what tells the two apart |
| The document doesn't settle which answer is right | An **owner decision**: name it, with suggested wording, and keep the miss open |

1. Look for a line the document already has that covers it (a nearby example, a definition elsewhere) and quote it first. Otherwise draft the sentence, following the edit loop in `drafting-decision-packs`: paste-ready text for a job with no release, a change request for a released one.
2. When the edit is in, `add_current_version`, re-quote the draft from the new text, `check_pack`, then `check_readings` again with the same phrasings, and report what changed.
3. **Never** change a phrasing's `expect` to what it was read as, reword a phrasing until it passes, write a meaning into the draft the document doesn't say, or ask for a lower bar. A miss still open at proposal time goes in the report as a gap, with the phrasing's shape and what it was read as.

One worked example from sales qualification: under "in progress: building, piloting or running an AI project now", "Our AI underwriting assistant is live for every broker" and "We launched AI-assisted underwriting to all brokers in September" were held. A fix to suggest for the document: "A project counts as in progress once it is live for customers or staff, including one launched earlier this year."

## 4. Keep the useful ones as `readingTests`

Phrasings you wrote go in the draft as `readingTests: [{ id, question, text, expect }]`, so a later version of the pack, or of Felspar's evaluator, reads them again. Keep every miss once it's fixed, one of each shape per answer and every near miss; drop near-duplicates. At most 8 per answer. They are synthetic, like test cases.

## 5. Testing on real evidence

Felspar reads whatever an agent brings, not the owner's wording, so made-up phrasings aren't enough. The person's own assistant (you) often already has the connections the job's evidence comes from: a ticketing system, a CRM, email, a contract repository, and their own skills for querying them. Use that access, but only with their agreement, and keep their data theirs.

### Notice the sources

From the draft's `evidenceKinds`, `whereToLook` and question meanings, name the systems the job will read ("this job reads support tickets and refund emails"). Check which ones your own tools and connections already reach, without being told; checking means looking at what tools you have, never opening a record. For any you can't reach, ask once: "Do you have an MCP or other connection to these, or a skill for querying them?" Never ask for credentials, keys or passwords, and never connect, install or authorize anything yourself.

### Explain first, then ask

Before any record is read, say in plain words:

- **Why it matters:** "Your agents will send Felspar real tickets and emails, written however people write them. If the pack's definitions don't match that wording, Felspar holds those cases for a person instead of deciding. Testing on a few real examples now finds those gaps before your team does."
- **What happens:** which source, about how many records ("about 20 recent tickets"), which question each sample tests, and that you read them through their own connection.
- **What Felspar sees and keeps:** Felspar reads only the short excerpt for each question, once, to check how it would decide. Nothing from a record goes in the pack, its test cases or the proposal. What's kept is an anonymized lesson ("a ticket saying the parcel was 'sent back last week', with no tracking, was held"), never the record.
- **What they control:** they can pick or exclude sources, lower the sample size, review the excerpts before Felspar reads them, or skip this step. Skipping is fine: the pack is then tested only on made-up phrasings, and you say so in the report.

Then ask one question, with the total and the per-question split: "OK to test on about 20 recent support tickets, about 10 for each of the two questions?" Ask about any source you can't reach after they answer, so each reply has one question. **Read nothing until they say yes.** Not a search, not a peek at one record, not a count. A yes covers the sources and sample size you named; anything larger asks again.

### Query and verify, one question at a time

For each choice question:

1. **Pull a bounded sample** of recent records of the job's evidence kinds through your own tools and the person's query skills: about 10 per question by default, never more than 20.
2. **Extract the excerpt** that would answer the question, as an agent would at decision time, leaving out names, email addresses and account or order numbers the answer doesn't depend on. Skip records with nothing relevant.
3. **Read each excerpt** with `check_readings` as `variants`, with `expect` set to the answer you think a person would pick (left out when you can't tell), in batches within the caps. Record what it picked, how sure it was and whether it held.
4. **Sort the results:** read as intended, held, or read as a different answer than a person would pick. Ask the person which answer the held and misread ones should get.
5. **Turn each miss into a suggested document edit** (an example, a near miss, an exclusion or a sharper meaning) for the owner to confirm, as in [step 3](#3-fold-misses-back-into-the-document). Never edit the pack directly: the fix lands in the document, then the pack is redrafted from it.
6. **Read the same sample again** after the owner's edits, and report what changed.

### Walk them through it

One step at a time, with a short progress line: "Read 10 tickets for 'what happened to the item': 7 read as intended, 2 held, 1 read as a different answer." Show each miss anonymized, next to the answer the person thinks it should get and the document edit that would fix it. Close with what was read (sources and number of records), what was learned, and what was kept.

### Privacy rules

- **Nothing from a real record is copied verbatim** into the draft, its `readingTests` or test cases, a suggested document edit, a change request, a proposal, a resume card or anything Felspar stores. Excerpts go only in `check_readings` `variants`, which are read once and kept nowhere.
- **Anonymize and paraphrase before keeping anything.** Replace names, companies, email addresses, phone numbers, order, ticket, deal, claim or account numbers, amounts and dates that identify a case or a person, and any phrase unique to one record. Keep the lesson ("a message saying they're 'exploring options' with no competitor named"), not the record. A fixed miss becomes a synthetic reading test written from that lesson.
- **Show excerpts only to the person** who agreed, and only anonymized unless they asked to review them first.
- **Say exactly what you read:** which sources and how many records, including when it was fewer than agreed. Never show model cost, tokens or usage counts.
- **Off-limits sources stay off.** If the person says a source is off-limits or holds regulated data (health, financial or children's records, for example), skip it and say which questions remain tested only on made-up phrasings.

## Next actions

| Situation | Next step | Who |
| --- | --- | --- |
| `check_readings` refuses with `readings_blocked` | The draft has blocking findings (check_pack lists them in warnings): close each one, taking its fixes in order, then check_readings again. | You |
| A phrasing is held or read differently | Suggest the example, exclusion or sharper meaning for the document; read again after the edit | You, then the owner |
| The document doesn't settle which answer is right | Raise it as an owner decision with suggested wording | The owner |
| Some phrasings `not_read` | Report them as untested; read them again later | You |
| The job's sources are reachable | Explain, then ask "OK to test on about N recent …?" | The person |
| A source isn't reachable | Ask once whether they have a connection or query skill for it | The person |
| The person skips real evidence, or a source is off-limits | Say which questions are tested only on made-up phrasings | You |
| Misses fixed | Keep the phrasings as `readingTests`, then propose | You |
