# Job boundaries: one pack, one decision

Use this to decide what a pack covers, when to split it into several and when to keep it whole. The tests tell you what to ask; the person decides.

## What one pack is

A pack answers **one question about one kind of case**:

- **One kind of case.** Every case the agent sends is the same kind of thing: one invoice, one claim, one leave request.
- **A fixed set of outcomes.** Approve, decline, review. Hold, pay. Remove, label, leave up. Every case ends in one of them, or is held for a person.
- **One accountable owner.** The person (a role) who approves changes to its rules.
- **One policy that decides it.** One document, or one section of a document, is the authority.

A long policy with many thresholds and exceptions can still be one pack. Size isn't the test; the decision is.

## Not a pack

These stay with the agent:

- **Writing:** the reply, the summary, the email, and their tone and style.
- **Looking things up:** finding a record, reading an account history, searching a knowledge base.
- **Taking actions:** sending, booking, filing, updating a system.
- **Steps where nothing is decided:** collecting documents, formatting, scheduling.
- **Choices no policy owns** and that cost little if they're wrong. If nobody would sign off on the rules, there's no pack to write. Rules that live only in an expert's head are different: that decision has an owner and a missing document, so it's a pack, written down first with `preparing-policy-documents`.

A pack can still *govern* an action: the agent asks Felspar first ("may this refund be issued?") and acts only on the outcome. The decision is the pack; the action stays with the agent.

## Split into separate jobs when

Any one of these is enough to split.


1. **The case changes.** A refund request and the customer's account are different cases; deciding about one isn't deciding about the other.
2. **The owner changes.** Each pack has one owner who approves its changes. If finance owns one part and support owns another, that's two packs.
3. **One decision only matters after another,** and needs its own evidence. "Is this claim covered?" then "Does it qualify for fast-track payment?" If the second only runs when the first says yes, and asks for different facts, split it.
4. **The parts change at different speeds.** If the price rules change monthly and the eligibility rules yearly, one pack means re-reviewing everything every month.
5. **Different agents need different parts.** If the intake agent needs only the first half and the payment agent only the second, they're two jobs.

## Keep it in one job when

- It's the same case, the same owner and one set of outcomes, however many rules lead there. All three must hold: two kinds of case under one owner and one document are still two jobs.
- The "second decision" is really an exception or a threshold of the first ("approve, unless over 5,000 EUR, then manager review").
- Splitting would make the agent send the same facts twice for one answer.

## Signs one job is really several

- Its outcomes answer different questions (approve or decline *and* which team to route to).
- Its questions fall into groups that never affect the same outcome. When Felspar's pack check reports `separate_decisions`, that's what it found.
- The agent would gather evidence from many unrelated systems for one case.
- The owner can't say in one sentence what it decides.
- Its "not for" is empty because it tries to cover everything.

## How jobs connect

Felspar decides each job on its own. Jobs don't call each other, and Felspar doesn't pass one job's result to the next. The connection lives in the agent:

- **Choosing a job.** An agent with several jobs picks one by each job's purpose, use-when and not-for. Make each job's not-for name its siblings ("not for coding invoices to the ledger; that is the invoice-coding job") so the agent never picks the wrong one.
- **What happens next.** Each outcome, and each escalation, has a next step the builder's code carries out: run another job, hand the case to a person, stop. Write those steps in the plan.
- **Shared definitions.** When two jobs quote the same passage of the same document, an edit to it is flagged for both. Keep shared definitions in one place in the document rather than copying them.

## Worked plans

These show the shape. Every company's decisions, owners and words will differ.

**Insurance: claims intake.** One case is one claim.

| Decision | Outcomes | Owner | Pack? |
|---|---|---|---|
| Is this loss covered by the policy? | covered, not covered, adjuster review | Claims manager | Pack |
| Does a covered claim qualify for fast-track payment? | fast-track, standard handling | Claims manager | Same owner and case, but it only runs after "covered" and needs its own evidence (amount, photos, prior claims): its own pack |
| Is there a fraud signal that needs the special investigations team? | refer, don't refer | SIU lead | Different owner: its own pack |
| Write the acknowledgement to the policyholder | – | – | Not a pack |

The agent runs coverage first; on **covered** it runs fast-track and the fraud check; on **adjuster review** it stops and hands the claim over.

**People: leave requests.** One case is one leave request.

| Decision | Outcomes | Owner | Pack? |
|---|---|---|---|
| Is this leave request approved? (notice, balance, blackout dates, leave type) | approve, decline, manager review | HR lead | One pack: sick, vacation and carer's leave are categories of the same decision, not separate jobs |
| Look up the employee's balance | – | – | Not a pack: a lookup |
| Update the HR system | – | – | Not a pack: an action, taken only on **approve** |

**Trust and safety: reported posts.** One case is one reported post.

| Decision | Outcomes | Owner | Pack? |
|---|---|---|---|
| Remove, label or leave up this post? | remove, label, leave up, specialist review | Policy lead | Pack |
| Does the account get a strike? | strike, warning, none | Enforcement lead | Different case (the account, not the post) and owner: its own pack |
| Draft the notice to the poster | – | – | Not a pack |
