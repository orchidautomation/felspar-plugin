# Editing the source

Every quote in a pack comes from one exact version of the document. A change to a released job's policy is proposed as a change request on that version; the document changes only after a person approves it. This page is how to write and propose those lines.

## Writing a suggested edit

- **Use the document's voice.** Match its headings, tense and terms. If it says "requester", don't write "user".
- **One rule per sentence.** "A refund request more than 30 days after delivery is declined" quotes cleanly; a paragraph of three rules doesn't.
- **Exact numbers, in days.** "within 30 days", never "within a month" (Felspar refuses months and years).
- **Definitions in three parts:** what it is, what it doesn't include (the near miss), and an example. "A sensitive system is one that holds customer payment data. It does not include internal wikis. For example, the billing console."
- **Say "something else" out loud.** Most documents never say what happens to a case that fits no category. Suggest a sentence for it.
- **Unique wording.** A quote must appear exactly once in the document. If the sentence you suggest repeats an existing one word for word, vary it.
- **Mark the owner's decisions.** When wording settles something the person hasn't decided, say so: "Suggested; needs the security lead's decision."

## Placing it

Give an anchor the person can find in seconds: the heading, and the sentence the new text goes after. Put related edits in one block per section, in document order, so one pass through the document applies them all.

## Proposing it

The document is the only record of policy. A change you propose is a draft until a person approves it, and you never edit the document yourself, even when your client has a tool that could.

- **The job has a release: open a change request.** `open_change_request` with:
  - `changes`: each `{ quote, replacement, reason }`. The quote is copied exactly from `read_source` on the source the job's release quotes (`list_jobs` names it) and must appear there exactly once; quote more of the passage if it repeats. To add lines, quote the sentence they follow and repeat it with the new lines after it. An empty replacement removes the passage. Changes may not overlap.
  - `draft`: the revised pack from `read_pack`, edited to quote the text as it reads with your changes applied. Felspar builds and checks it against that text without storing it, and returns the diff, the cases decided differently and the saved-case counts.
  - `definitions`: `{ id, reason }` for each question or answer whose definition the draft adds or changes.
  Refusals name each change by its index (`quote_not_found`, `quote_not_unique`, `edits_overlap`, `edit_unchanged`); fix those and open it again, which replaces your earlier one.
- **Before opening, check what's open.** `list_change_requests` shows the job's open requests and their conflicts (the same rule, definition or passage). Show the person the other request rather than opening a competing one silently; `comment_on_change_request` adds to its discussion.
- **Edits someone made directly in the document** arrive as change requests of their own (origin `outside_edit`), with who edited it when Google says. Treat them like any other: read, triage, discuss.
- **The job has no release yet.** There is nothing to propose on: the person adds the lines to the document themselves. Give the exact wording and place, and ask them to say "done" when it's saved.
- Approving, releasing, rejecting and withdrawing are a person's click on the Felspar web page. No tool does them; never say a change request is approved or live.

## Picking it up (job not yet released)

1. Call `add_current_version` with the document's current `sourceObjectId`. It reads the document now; there is no need to wait for `list_sources` to show `changed`.
2. If `isNewVersion` is false, the edit isn't in the document Felspar can read yet: the person may still be typing, or saved a different document. Ask, then try once more.
3. `read_source` the new `sourceObjectId`, and draft and propose from it from now on.
4. Keep the draft. Quotes from passages that didn't change still match; re-point any that moved, add the new passages, and run `check_pack` again.
5. Tell the person what the edit fixed and what's next.

## What not to do

- Don't edit the document, or insert text into it, with any tool. Propose the lines in a change request instead.
- Don't quote wording that isn't in the document yet in `check_pack` or `propose_pack`; they will refuse it, and the pack would claim the document says something it doesn't. Only a change request's draft quotes proposed text.
- Don't paraphrase a gap away by stretching an existing sentence to mean more than it says.
- Don't summarize what a change "really means" beyond what Felspar's triage and diff say; read both quotes to the person instead.
