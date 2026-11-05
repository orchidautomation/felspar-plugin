---
name: planning-agent-jobs
description: Turns "we want an agent for this team" into a plan of Felspar jobs - lists the decisions in the work, marks which need a Decision Pack and which stay with the agent, splits or merges them so each pack makes one decision, names each job's case, outcomes, owner and source document, says how the agent moves between jobs, and picks the one to build first. Use this skill when someone wants an agent for a team, department or workflow, asks whether something should be one pack or several, whether packs talk to each other, or whether a policy or pack is doing too much, even if they don't say 'Felspar'. Not for writing a policy (preparing-policy-documents), drafting one pack (drafting-decision-packs), tests (writing-pack-test-cases) or registering an agent (connecting-company-agents).
compatibility: Works without tools by interviewing. With the Felspar connector (list_jobs, list_sources) it checks which jobs and documents already exist. A person signed in as a workspace member, approver or owner.
metadata:
  author: felspar
  version: "0.1.0"
---

# Planning agent jobs

A Decision Pack makes **one decision about one kind of case**: a fixed set of outcomes, one accountable owner, one policy that decides it. "Can this refund be approved?" is a pack. "Run our support team" is not; it is a handful of decisions plus a lot of work that isn't deciding anything.

Your job is to turn the work someone wants an agent to do into a short plan: which decisions need a pack, which don't, how they connect, and which to build first. The plan is a recommendation; the person decides.

## Workflow

- [ ] 1. **Name the work.** In two or three lines: which team or workflow, what one case looks like as it moves through it (an invoice, a claim, a ticket, a request), who does it today, and which agent will run it. If Felspar is connected, call `list_jobs` and `list_sources` so you build on jobs and documents that already exist.
- [ ] 2. **Check whether it's already one decision.** If the request is one question with one set of outcomes ("which refund requests do we approve?"), say so, skip the plan, and hand off: `preparing-policy-documents` if nothing usable is written, otherwise `drafting-decision-packs`.
- [ ] 3. **List the decisions.** Walk one case through the work, step by step. At each step ask: is something decided here, with a fixed set of possible outcomes, that a policy should govern? Write each one as a question in the team's words. Also list the steps that aren't decisions (writing, looking things up, taking an action), so the person sees what stays with the agent.
- [ ] 4. **Sort and group.** Use [job boundaries](references/job-boundaries.md): mark each step **pack** or **not a pack**, then apply the split and keep-together tests to the decisions. Each group that remains is one job.
- [ ] 5. **Describe each job.** For each: the decision in one sentence, what one case is, when the agent runs it, its outcomes, the owner (a role), the source document and its state, what it is **not for** (naming its sibling jobs), and what happens next for each outcome. A document's state is **ready**, **thin**, **missing** or **not read yet**: read it with `read_source` if it's connected, otherwise mark it not read yet and say so. Rules that live only in someone's head are a **missing** document with an expert available, not a reason to skip the pack. Show it as one table (see [Output](#output)).
- [ ] 6. **Say how the agent moves between jobs.** Felspar decides each job on its own; it doesn't pass one job's result to another. The agent, or the builder's code, runs the jobs in order: it picks a job by its purpose, use-when and not-for, and uses each outcome or escalation to choose what happens next. Write that order as plain steps.
- [ ] 7. **Pick the first job.** Weigh what a wrong call costs and how often the job runs first; a job every case passes through often matters most. Then prefer a clear owner and something to build from now: a usable document, or an expert who can walk through cases (`preparing-policy-documents` runs that interview). Say why, and what you'd build second.
- [ ] 8. **Hand off.** Ask the person to confirm or change the plan. Then hand the first job to `preparing-policy-documents` (thin or missing document, or rules only in someone's head) or `drafting-decision-packs` (a usable document), with the job's one-line decision, outcomes, owner and not-for.

## Output

One table, then the order the agent runs things, then the recommendation.

| # | Decision | One case is | Runs when | Outcomes | Owner | Document | Pack? | Not for |
|---|---|---|---|---|---|---|---|---|
| 1 | Does this invoice need a hold before payment? | One vendor invoice | Every invoice | pay, hold for procurement, hold and verify | AP lead | AP controls doc (ready) | Pack | Coding to the ledger (job 2) |
| 2 | Which ledger account does this invoice go to? | One invoice line | After job 1 says pay | one of the chart of accounts, review | Controller | Coding guide (thin) | Pack | Holds (job 1) |
| – | Write the vendor email | – | – | – | – | – | Not a pack: the agent writes it | – |

Under the table, list any assumptions you made (an outcome you guessed, an owner you weren't sure of) as "correct me" lines; they aren't extra questions. Then: "The agent runs job 1 on every invoice. On **pay**, it runs job 2; on either **hold**, it stops and hands the invoice to the hold queue." Then: "Start with job 1: …"

## Gotchas

- **Never invent a decision or a rule.** The list comes from how the team works today. If you think a decision is missing, ask.
- **Owners are roles,** never names, tools or channels.
- **One department is never one pack.** If a single job would decide several different things, it's several jobs.
- **Don't promise what Felspar doesn't do.** It doesn't chain jobs, run steps, send messages or route work. Those belong to the agent and the builder's code.
- **Not every decision needs a pack.** A choice nobody owns a policy for, or one with no real consequence, can stay with the agent. Say so, with the reason.
- **Rotate the domain.** Use the team's own vocabulary and examples from their work, not sales by default.
- **A plan changes nothing.** No job exists until a pack is drafted, approved and released.

## Next actions

End every reply with where the person is and the **one** next thing: a question with your recommended answer, or an action with who does it. Blockers first.

| Situation | Next action |
| --- | --- |
| The work isn't clear yet | Ask them to walk you through one recent case from start to finish. |
| It's already one decision | Say so and hand off to `preparing-policy-documents` or `drafting-decision-packs`. |
| A step might or might not be a decision | Ask: are there a fixed set of outcomes, and does a policy say which one? |
| Two candidate jobs might be one | Apply the keep-together test; ask the one question that settles it (same case? same owner? same outcomes?). |
| A job has no owner | Ask who would be accountable for its rules (a role); until then it's a suggestion. |
| A job's document is missing or thin | Mark it, and plan `preparing-policy-documents` for it. |
| An existing pack does more than one decision | Show which questions and outcomes belong to which decision, and suggest splitting it into separate jobs. |
| The plan is shown | Ask them to confirm or change it, with the first job you recommend. |
| The plan is confirmed | Hand the first job to `preparing-policy-documents` or `drafting-decision-packs`. |

Example ending: "Five steps, two decisions worth a pack. Next: start with invoice holds? I'd suggest yes, because a wrong payment can't be pulled back and the AP controls doc is already written."
