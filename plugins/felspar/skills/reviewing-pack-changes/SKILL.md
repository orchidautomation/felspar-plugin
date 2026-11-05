---
name: reviewing-pack-changes
description: Explains what a proposed change to a Felspar Decision Pack would do before anyone approves it - backtests it against the current release, explains which decisions flip or become unknown, flags meaning changes and names the agents affected. Use this skill when someone asks what a policy or pack change would do, wants a backtest or a version comparison, or is preparing an approval, even if they only ask whether a change is safe. It never approves; approval is approving-and-releasing-packs.
compatibility: Requires the Felspar connector (list_jobs, read_pack, read_proposal_review, list_saved_results, read_saved_result, preview_change, check_readings, backtest, read_source, read_change_request, read_evals, list_eval_runs, read_eval_run, run_evals) and a person signed in as a workspace member, approver or owner.
metadata:
  author: felspar
  version: "0.3.1"
---

# Reviewing pack changes

The approver needs to see consequences, not diffs. Your job is a short, truthful summary of what changes for real cases and real agents.

## Workflow

1. **Identify the change:** which proposal, against which release. `list_jobs` shows the job's current release and open proposals; `read_pack` with and without `proposal_id` gives both versions as drafts, so you can name each changed question, answer, rule and outcome by its ID. Read the changed source passages with `read_source`. For an existing proposal, `read_proposal_review` returns what its reviewers see in one call: the changes against the release it was built on, whether it is the job's first pack or has gone stale, the cases its tests decide differently, saved backtests and previews by ID, and who may approve it.
2. **Check for a saved result first.** Call `list_saved_results` with the job ID and `proposal_id`: it lists the backtests people already ran on this proposal and the preview of the draft it came from, each with who ran it and when. If one fits (same proposal, compared with the release that is current now, over the cases you need), `read_saved_result` it instead of running a new one, and cite it: "Priya ran a backtest on this proposal on 27 Oct 2026, 14:05 UTC; here's what it showed." Run a new one when none fits, the current release has changed since, or the person asks for a fresh run or for re-reading.
3. **Backtest:** otherwise call `backtest` for the proposal against the current release over recent saved cases. Report counts: changed, unchanged, unresolved. The run is kept for the workspace (`savedResultId`), so the next reviewer can read it. Without `execution_ids` it picks the cases the proposal can change (those that asked a question it changes, undecided and escalated first); cases that asked none of them count as `unchanged` without being re-decided.
4. **Explain the flips.** Group changed decisions by cause (for example "all 14 flipped because the size floor moved from 200 to 100") and show two or three examples.
5. **Check for meaning changes.** If a definition's wording changed (what counts as "active evaluation"), past evidence must be re-read under the new meaning. Tell the person how many quotes re-reading covers and whether it is within today's limit (the backtest's `estimate`): "re-reads N quotes with Felspar's evaluator; within today's limit". When `estimate.exact` is false, say "up to N quotes": some are read only if another answer asks for them. Run it only if they say so. When the diff has `readerChanged`, Felspar will read this job's evidence with a newer reader, so some cases that went to review may now decide, even where the pack's text is unchanged. Say so, and offer `reinterpret` to see which. The review summary says it too.
6. **A new question changes what agents must send.** When the proposal adds a question its rules need, agents must start sending it; until they do, every run that needs it goes undecided. Say that first, next to breaking changes. Saved cases recorded before it are `needs_new_answer` (`newQuestions` names it) until their kept quotes are read: offer re-reading with `reinterpret: true`, which decides the cases those quotes answer. Cases still undecided afterwards are `unresolved` with reason `new_question_unanswered`: say how many ("12 cases would go undecided until agents send the refund answer") and that this is what those runs will do after release.
7. **Lead with breaking changes.** If the backtest has `breaking`, say each one plainly first ("removes escalation risk-review") with the number of agents on the job: customer code maps escalation ids, subject types and relationship names, so a removal or rename breaks it. Cases whose escalation changes are `changed` even when the outcome is the same; `escalationChanged` counts them, and also cases the proposal leaves undecided (they hand off to the unresolved path but count as `unresolved`), so it can exceed `changed`.
8. **Check evidence kinds too.** If a question's `evidenceKinds` changed, say so with the breaking changes, even when `breaking` is empty: agents still sending the old kind are refused until they change, and the current release can't be compared on those cases.
9. **Run the job's eval set.** `read_proposal_review` shows the newest run against the proposal (`evals.run`) and whether the set changed since (`isStale`); without a current run, `run_evals` with the `proposal_id`. Lead with `newMisses`: cases the current release reads as expected and the proposal doesn't, each with what it now does and why. A job with no set (`evals.setVersion` null) gets one from `writing-pack-test-cases` before release.
10. **Name who is affected** from the backtest's agents: agents following the job on Latest get the new version when an owner releases it (approval alone changes nothing); pinned agents don't.
11. **Suggest confidence bars only from the person's own labels.** When the person tells you, case by case, the answer a saved case's quotes establish for a question (or that they don't settle it), pass those as `labels` to `backtest`. Its `calibration` suggests a bar per question from those cases (held to a higher standard for a question marked `highConsequence`), or says why there is none: `too_few_cases` (at least 10 labeled cases), `coverage_gap` (every label is the same answer) or `needs_more_evidence_or_rewrite`. Never label a case from your own reading or Felspar's. Nothing changes here: the person who releases ticks a suggested bar on the release page, and unticked questions keep their bar.
12. **Write the summary** using [the review template](assets/review-summary.md).

## Before a proposal exists

When the person is still revising a draft, call `preview_change` (job ID, the new `sourceObjectId`, the draft) instead of `backtest`: it compares the draft with the current release without changing any pack. Check `list_saved_results` (job ID, `kind: "preview"`) first: a preview of the same draft (`draft.draftDigest`, same `sourceObjectId`) already shows it; read it and say who ran it and when. Present its `diff` and `cases` as Markdown tables (before | after | why), `breaking` entries first, with its `backtest` counts, and ask "OK with these changes?" before anyone calls `propose_pack`. Each added question the draft's rules need appears under `breaking` as "agents must start sending ...": present it with the breaking changes. The current pack's own test cases marked `needsNewAnswer` only lack that answer in the test; saved cases that go undecided are real changes: shown `changed`, with `status: "unresolved"` (how the counts count them) and `open`, each question still open and why (`unclear`, `no_quotes`, `waits_on_question`, `not_read`, `quotes_disagree`, `not_accepted`); name those. If quotes need re-reading, say how many (up to, when `estimate.exact` is false) and whether today's limit covers them (its `estimate`), and run it with `reinterpret: true` only on a yes.

## A change request, before the document changes

A change request proposes lines for a released job's document and carries a revised draft; the document doesn't change until it is approved. To see whether its wording works, call `preview_change` with the job ID and `change_request_id` (no `source_object_id` or draft): Felspar builds the request's draft on the document with its changes applied, in memory only, and compares it with the release the request is based on. Its `savedCases` lists each saved case now and with the change: the new outcome, or the questions it is still held on and why. A changed definition needs re-reading: say how many quotes (`estimate`) and run it again with `reinterpret: true` only on a yes. Re-reading spends the request's own allowance (`allowance.percentLeft`), and quotes already re-read for it are not read again. Name each case the request was meant to fix and whether it now decides; a case still held means the wording isn't sharp enough yet. `check_readings` with `change_request_id` reads the request's `readingTests` and any phrasings you send against the proposed wording, the same way. Both are refused once the request is closed or out of date (`change_request_closed`, `change_request_out_of_date`): open a new one from the current release. Each such preview is kept like a draft's: check `list_saved_results` with `change_request_id` first.

## Gotchas

- **Re-reading shares one allowance per proposal.** `allowance.percentLeft` is what is left across all of this proposal's backtests; quotes already re-read for it are reused. Past it, `reinterpret` is refused with `backtest_allowance_used`: backtest again without it. Report the share left, never a count.
- **Number and date changes need no re-reading.** They reuse the recorded answers. Say so; it builds trust in the numbers.
- **Felspar's costs are internal.** Never discuss Felspar's internal costs or usage mechanics; if asked, say evaluation is included and has a daily limit.
- **A saved result is a snapshot.** Say who ran it and when, and which release it compared with. `casesWithheld` counts cases that have since left the 30-day case history; the counts stay as they were. If you can't read one (`source_access_required`), you need access to the document behind it, as running it would.
- **Unknown stays unknown.** Cases whose evidence can't be re-read (excerpts not kept, or older than 30 days) are reported as unknown, never guessed.
- **A backtest describes past cases only.** It doesn't predict volume, revenue or every future case.
- **Don't approve.** This skill prepares the decision; approval belongs to an approver using `approving-and-releasing-packs`.
- **Pinned agents are a feature.** Name them so the owner can decide when to move them.

## Next actions

End every reply with where the person is and the one next thing: a question with your recommended answer, or an action, and who does it. Blockers first. Common situations:

| Situation | Next step | Who |
| --- | --- | --- |
| The backtest lists a `breaking` change | Say it first, with the agents on the job; the owner decides whether to keep it | The owner |
| A change request should fix held cases | `preview_change` with its `change_request_id`; on a yes, again with `reinterpret`; name each case it was meant to fix and whether it now decides | You, then the person |
| A change request's preview still holds a case | Suggest sharper wording, then the person opens the request again (it replaces their earlier one) | You, then the person |
| Definitions changed and cases need re-reading | Say how many quotes and whether today's limit covers them; run `backtest` with `reinterpret` only if they agree | The person |
| The proposal adds a question | Say agents must start sending it and which cases go undecided until they do; offer re-reading kept quotes with `reinterpret` (how many quotes, from `estimate`) | The person, then the owners of the agents |
| Many decisions flip | Group them by cause and ask whether each flip is intended | The owner |
| Cases stay unresolved | Name the open question behind them and why (a case's `open`, or the backtest's `reason`); it's usually a document gap for `drafting-decision-packs`, or a new question agents must start sending | The owner |
| Someone already ran a backtest or preview that fits | Read it with `read_saved_result` and cite who ran it and when; offer a fresh run only if the release changed since | You |
| You need the runs behind an eval count | `list_eval_runs` for the job or proposal, then `read_eval_run` for the cases, what they sent and what each side read; `compare_with` for what changed between two runs | You |
| The eval run lists `newMisses` | Say each first, with what it now does; it is usually a definition the document should sharpen | The owner |
| The job has no eval set | Write one with `writing-pack-test-cases` and run it before release | You |
| No saved cases to backtest | Say so; the review rests on the pack's test cases | You |
| The review is clean | Hand off to `approving-and-releasing-packs` with the summary | An approver |
| Pinned agents won't get the release | List them so the owner can move them deliberately | The owner |

## Decision diagrams

`read_pack` returns the canonical Mermaid graph and its dashboard link. `preview_change` returns `graphs.current` and `graphs.candidate`. Show those exact returned diagrams when explaining the full path or a proposed change; never redraw the topology from prose. A saved decision’s dashboard view uses its original release and verified receipt. If a graph is unavailable, report that state instead of inventing a path.
