# When Felspar refuses

A refusal is the next step. Each tool's description, the refusal's `fix`, `get_pack_guide` (When Felspar refuses) and this page say the same thing.

| Refusal (tools) | Next step |
| --- | --- |
| `needs_definitions` (check_pack, propose_pack) | The draft has blocking findings (check_pack lists them in warnings): close each one, taking its fixes in order. A quote or draft fix needs only check_pack again. For a job with no release, send the accepted lines as pending_lines and check_pack again. For a released job, never edit the document: list_change_requests, read_change_request to build on your open one, open_change_request with each line as a change (quote the passage it follows, then repeat it with the line after it), its reason and the revised draft, then test it with check_readings, preview_change and run_evals given its change_request_id. If the document already holds a newer version than the release, add_current_version, quote its lines and check_pack again. Nothing was proposed. |
| `tests_failed` (check_pack, propose_pack) | A test case doesn't return what the document says (tests lists each). Fix the rule or question the document decides; never change a case's expected outcome to what the draft returns. |
| `draft_not_ready` (open_change_request) | The draft has blocking findings (check_pack lists them in warnings): close each one, taking its fixes in order. Add the lines to the change request's changes, quote them in the revised draft and open_change_request again. Nothing was stored. |
| `change_request_quote_only_on_staging` (open_change_request) | This text is only in the release staging serves: open the change request with base_environment: staging. details.problems names each change production's version doesn't hold. |
| `readings_blocked` (check_readings) | The draft has blocking findings (check_pack lists them in warnings): close each one, taking its fixes in order, then check_readings again. |
| `preview_reinterpret_blocked` (preview_change) | The draft has blocking findings (check_pack lists them in warnings): close each one, taking its fixes in order, then preview_change with reinterpret: true again. A preview without reinterpret still runs. |
| `evals_blocked` (run_evals) | The draft has blocking findings (check_pack lists them in warnings): close each one, taking its fixes in order, then run_evals again. |
| `eval_candidate_not_ready` (run_evals) | The candidate doesn't build (status says invalid or tests_failed). Run check_pack on it, fix each problem at its path, then run_evals again. |
| `release_reader_fixed` (run_evals) | Later releases of a job keep its reader, which details.reader names. Run evals or release again without reader, or with that one. |
| `readings_not_checked` (propose_pack) | Run check_readings on this exact draft (the same source_object_id, draft and pending_lines you propose) with phrasings of each answer listed in answers: plain, indirect, dated and a near miss that belongs to another answer. A reading not_read doesn't count: read it again. Fix a miss in the document and redraft, then propose_pack the draft you read; misses don't refuse a proposal, its owner reviews them. |
| `job_retired` (propose_pack) | The workspace owner retired this job. Don't call it again: use the job named in replacement (check it with list_jobs), or, if replacement is null, tell the person the job is retired and stop. |
| `source_read_failed` (add_source, add_current_version) | Show the safe returned next step. Consent/reconnect is the person's click. For `no_access` on a Google Doc, give the person the returned `pickUrl` (or Choose a Google Doc on the Felspar Sources page) to choose it there (their click; owning the Doc or having an agent create it is not enough), wait for them, then call the same tool again; only if Google doesn't offer it there, ask the document owner for access. A provider failure is not a captured source. Never use another person's connection or invent source IDs. |
| `source_access_required` (read_pack, read_source, add_current_version) | Your own connection can't read a document behind this object. Connect or reconnect it on the Felspar Sources page, or ask the document's owner to share it. |
| `pending_lines_invalid` (check_pack, check_readings, run_evals, propose_pack) | A pending line can't be placed on the document (details.problems names each line by index and code): anchor it after a passage that appears exactly once, or at a non-blank line N, as one sentence with no newline, and send the same pending_lines again. |
| `pending_lines_not_proposable` (POST /v0/drafts/propose) | Pending lines never become an ordinary proposal: send them with propose_pack, which opens a change request on the cached document for a job with no release. |
| `evals_gate_blocked` (release, stage or promote) | A release needs the job's eval set to mark gate: true on at least one case expecting an outcome, one expecting a hold (review, missing or stale) and one expecting a conflict (details.floorMissing names each kind still missing: decided, held or conflict; add a case of each kind it names and mark it gate: true with save_evals), and the newest eval run of the set's current version against what you are releasing (for a proposal, a run of the exact draft it was made from on the same source counts too) to bring each marked case back as expected; a case expecting a hold comes back as expected only when held for that reason. details.cases names each case by what was expected and what happened, when you may read that run (not_run: the run didn't send it, or ran an earlier version of the set). Fix the pack until each case comes back as expected, never a case's expected outcome to match the pack, run_evals again on the same proposal or change request, then release again. Suggested bars can't be accepted on that release (details.acceptedBars). To promote a release staged before the set changed, propose its pack again, run_evals on that proposal, release it to staging again, then promote. |
| `job_routing_missing` (release or stage) | An agent picks a job by its purpose and when to use it, so a release needs both (details.missing names each still missing: purpose, useWhen). In the pack draft, quote the passage that says why the job exists and what it resolves as "purpose" and at least one passage that says when to apply it as "useWhen"; if the document doesn't say, suggest wording for the owner to add. Then check_pack again, propose the pack and release the new proposal. Nothing was released. |
| `pending_lines_need_google_document` (propose_pack) | Pending lines are written only to one connected Google Doc: ask the person to add the accepted lines to the document, add_current_version, quote them and check_pack again without pending_lines. |
| `pending_lines_need_unreleased_job` (propose_pack) | This job has a release, so its changes go through a change request: open_change_request with each line as a change, its reason and the revised draft. |
| `draft_not_held` (check_pack, check_readings, preview_change, run_evals, propose_pack) | Felspar holds no draft under this draft_id in this workspace. Send the whole draft with its source_object_id to check_pack, then use the draft_id it returns. |
| `draft_expired` (check_pack, check_readings, preview_change, run_evals, propose_pack) | This draft_id has expired: Felspar holds a draft for 24 hours after check_pack returned its draft_id. Send the whole draft with its source_object_id to check_pack again, then use the new draft_id. |
| `draft_id_conflict` (check_pack, check_readings, preview_change, run_evals, propose_pack) | Send draft_id or draft, not both. With draft_id, leave source_object_id out or send the source the draft was checked against; a patch goes only with draft_id, on check_pack. |
| `draft_patch_failed` (check_pack) | One patch operation can't apply to the held draft (index names it, problem says why). Nothing was applied and the held draft is unchanged. Fix that operation's path (a JSON Pointer into the draft as check_pack last held it) and check_pack again with the same draft_id and patch, or send the whole draft. |

Blocking findings, each in `warnings` with `severity: "blocking"`:

| Finding | Next step |
| --- | --- |
| `missing_definition` | Quote the passage that defines it. |
| `thin_answer` | Take its fixes in order: a sentence of the document at its line, another answer's quote, or the owner question with its recommended default. |
| `answers_share_passage` | Inspect conflicts for the exact meaning fields and source lines, then follow the finding's fix. Keep the document's meaning and required lists; ask the owner when it does not distinguish the answers. |
| `missing_none_of_these` | Add an answer for text that clearly says something outside this list, marked noneOfThese: true. |
| `unrouted_none_of_these` | Add a rule for it, quoting where the document says it leads; if the document doesn't say, suggest wording for the owner to add. |
| `untested_threshold` | Add a case at the last value that meets it and one at the first that doesn't, with the outcome the document gives each. |
| `untested_band_edge` | Add cases at the last total in the band and the first outside it, with the outcome the document gives each. |
| `coverage_missing` | Add coverage: each material passage of the document, quoted, with its categories and what became of it (represented, needs_review or unsupported), starting with the passages it names. |
| `uncovered_passage` | Add the passage at its line to coverage with its categories and what became of it: represented, needs_review or unsupported. |
| `thin_number_or_date` | Take its fixes in order: a sentence of the document at its line, or the owner question with its recommended default. |

`coverage_missing`, `uncovered_passage` and `thin_number_or_date` refuse only a job's first release. On a revision of a released job they are warnings: report them in the revision's gap list and close what the revision can.

Wording warnings, each with its `fix` (see [turning policy into questions](writing-questions.md)):

| Warning | Next step |
| --- | --- |
| `judgment_question` | Ask what the record states, and let rules combine stated facts. |
| `limit_in_question` | Ask for the value as a number or date question and put the limit in the rule. |
| `math_in_question` | Ask for each value the record states; let the pack compute, count and compare. |
| `compound_question` | One question per fact; put the "and", "or", "unless" in rules and exceptions. |
| `agent_conclusion` | Ask what the source record says; an agent's conclusion is never evidence. |
| `adverse_outcome_unmarked` | An outcome reads as going against the case but isn't marked adverse, so it acts without a person. Add `"adverse": true` if the document wants a person to see it first; if it really should act unattended, leave it and record that in the gap report. |
| `overlapping_answers` | Give each answer text only it describes, or split the question. |
| `undeclared_related_record` | A question or a `subjects` passage names a record other than the case's own (the policy a new one replaces, a parent account). Add the relationship its `fix` suggests, with the document's own definition of that record, and set `about` on the questions about that record, before any eval case is marked `gate: true`. If it is the case's own record, leave it and say so in the gap report. |
| `no_reading_tests` | Keep the phrasings you read with `check_readings` for that question in `readingTests`, each with the answer a person would pick. |
| `job_routing_missing` | The job has no `purpose` or no `useWhen` (`path` names which), so an agent that may run several jobs can't pick it, and its release will be refused. Quote both from the document; if it doesn't say, suggest wording for the owner to add. |
| `release_no_eval_set`, `release_case_missing` | Save the eval set with a marked (`gate: true`) case of each kind the warning names: an outcome, a hold, a conflict. |
| `decided_share_below_bar` | The newest `run_evals` of this draft decided under 80% of the cases expecting an outcome as expected. `read_eval_run` it, look at the held and wrong cases and fix the wording through the edit loop, then run again; never change a case's expected outcome to pass. It never blocks a release. |
| `release_evals_not_run`, `release_case_not_as_expected` | `run_evals` with this draft; fix the pack until each marked case comes back as expected, never the case. After `propose_pack`, `run_evals` on the proposal. |
| `release_case_outcome_unknown` | The case expects an outcome the pack doesn't have: correct the case with the job's approver, or add the outcome if the document has it. |
| `release_case_unreachable` | No rule reads a question, so no case can be held or conflict. If the document makes the decision depend on evidence, add that question to the rule's `when`; otherwise raise it with the job's approver. |
| `release_case_source_refused` | Send the excerpt the way its kind's source rule takes it (its `source.url`, or as the system's record in its ID form), then `run_evals` again. |

Fix the other warnings (`absence_question`, `duplicate_question`, `many_answers`, `date_in_answer`, `uncovered_passage`, `no_held_case`, `no_stale_case`, `source_class_not_quoted`, `source_class_quote_not_found`, `source_limit_not_enforced`, `state_not_confirmable`) where the document lets you; report the rest as gaps.
