---
name: approving-and-releasing-packs
description: Prepares a Felspar approver to approve and release a Decision Pack on the Felspar web page - confirms the exact proposal reviewed, shows what changes and which agents are affected, and hands the person the review link to approve and release it themselves, to staging first or straight to production. Use this skill when someone asks to approve, release, ship, publish, stage, promote, roll back or roll out a pack or policy change for their agents. Not for reviewing impact without approving (reviewing-pack-changes).
compatibility: Requires the Felspar connector, signed in as the person. Approving, releasing, promoting and rolling back happen only on the Felspar web page, as that person's own click; no tool can do any of them, and members and agents cannot either.
metadata:
  author: felspar
  version: "0.3.0"
---

# Approving and releasing packs

Approval says "this is what the company means". Release makes it what agents follow. Both are consequential: a release can be superseded, never edited. That is why they are never tools: the person approves and releases on the Felspar web page themselves. Your job is to make sure they know exactly what they are about to approve, then send them there.

## Sequence

1. **Identify the exact proposal** the person means (its ID from `propose_pack`). If they haven't seen a review summary, produce one first with `reviewing-pack-changes`.
2. **Show the consequences** in two or three lines from `backtest`: decisions that change (old outcome to new), cases still unresolved, and the agents that follow this job and will get the release.
3. **Hand them the review link** that `propose_pack` returned, and say what they'll do there: approve the proposal, then release it. Only the job's approvers can: name them from `owners` in `list_jobs`, `read_pack` or `propose_pack` (a workspace owner can also act as a recorded fallback).
4. **Suggested confidence bars,** if a backtest of the proposal suggested any, appear on the release page as boxes to tick. Tell the person which questions have one and what each would change (the backtest's `calibration`): they tick the ones they accept on the release click; unticked questions keep the bar they were proposed with.
5. **Staging first, when it's a change worth trying.** Each job has a staging and a production environment. On the approval page the person can click **Release to staging** instead: production keeps its release and only staging agents (registered with a staging key) run the new one. When it behaves as expected, they click **Promote staging's release to production** on the job's page; production then runs exactly the release staging ran, nothing rebuilt. Read where each environment stands with `read_environments`.
6. **A change request can go to staging before the document changes.** On the job's page an approver releases an open change request's candidate (the document with its changes applied) to staging. Promotion waits until the document holds exactly that text: the person edits the document to match, you run `add_current_version`, then they promote. If promotion is refused, the page and `read_environments` say which changes are still missing. A first draft's change request (from `propose_pack` with `pending_lines`, for a job with no release) shows each line and its reason on the job's page: the approver stages it, then **Write to Google and promote** writes the lines into the Google Doc with their own Google connection, reads it back and promotes. If the document changed where a line goes, nothing is written and the request goes back to its author as out of date.
7. **Afterwards,** if they ask, confirm with `read_environments` and `backtest` that the release is what agents now use.

## Gotchas

- **Never claim to approve or release.** No tool does it. If someone asks you to "just approve it", explain that it is their click on the web page, by design, and give them the link.
- **Nobody approves their own proposal while another approver can.** `read_proposal_review` says whether the person may approve it (`approval.canApprove`, with the reason); if they proposed it, share the review link with another of the job's approvers.
- **You can't help someone approve for someone else.** If the person isn't one of the job's approvers (a member, or `not_job_approver` on the web page), name the people in the job's `owners` and share the link with them; don't look for another way.
- **"Source access required"** on the web page means the approver's own Google access to the policy document is missing or expired. They need access to approve, which is deliberate.
- **Approve what was reviewed.** If the proposal changed since the review, review it again first.
- **Never claim to stage, promote or roll back.** No tool does those either. `read_environments` only reads; tell the person which button to press on the job's page.
- **A staging key never runs production's release, and a production key never runs staging's.** If someone's test agent isn't seeing the staged change, check it was registered with a staging key (`connecting-company-agents`).
- **Rolling back** is **Roll back production** on the job's page: production goes back to a release it served before, on agents' next call.
- **Release takes effect when it's made.** Scheduling a release for a later date isn't available yet; if asked, say so and suggest releasing at that time.

## Next actions

End every reply with where the person is and the one next thing: a question with your recommended answer, or an action, and who does it. Blockers first. Common situations:

| Situation | Next step | Who |
| --- | --- | --- |
| No review summary yet | Produce one with `reviewing-pack-changes` first | You |
| The proposal changed since it was reviewed | Review it again before approving | You, then the approver |
| The person can't approve | Name the job's approvers from `owners.summary` and share the review link with them | The person |
| "Source access required" on the web page | The approver restores their own Google access to the document | The approver |
| Ready to approve | Open the review link, approve, then release to staging or production | The approver |
| Staged and tried | Promote staging's release on the job's page | The approver |
| A staged change request won't promote (`document_not_updated`) | Edit the document to match the missing changes, then `add_current_version` | The person, then you |
| Released | Connect or update agents with `connecting-company-agents`; tell pinned agents' owners | An owner |
