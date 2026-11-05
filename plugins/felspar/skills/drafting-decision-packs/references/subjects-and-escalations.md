# Subjects, relationships and escalations

A pack says what each case is about and where a case goes when it can't be decided. Both are the document's own words, quoted like every other meaning. Felspar never assumes a vertical: a case is a ticket only because the document says so.

## The job first

`purpose` quotes why the job exists and what it resolves. `useWhen` and `notFor` quote when the document says to use it and when not. `owner` quotes the accountable role if the document names one; leave it out otherwise. An agent that wasn't told which job to run picks one from these with `list_jobs`.

## Subjects

Declare each kind of record the document talks about once, and name the job's own in `subject`.

| Field | What to write |
| --- | --- |
| `id` | The record's name in the document, such as `ticket`, `contract`, `candidate`, `company`. |
| `definition` | The quote that says what the record is. |
| `identifier` | How its ID is written: "help desk ticket number", "contract ID", "web domain". |
| `example` | One ID, written as Felspar keeps it (lowercase when matching is case-insensitive). |
| `match` | `exact` (default) or `case-insensitive` for web domains and email addresses. |

Felspar normalizes the case ID by the job subject's `match` before it counts attempts, keeps history or writes a receipt, so `Jo@Example.com` and `jo@example.com` are one case.

## Identity: how each case is identified

Not every case has an ID of its own. A subject's `identity` says how a case gets one:

- `provided` (default when the subject has an `identifier`): the record's own ID, written as its `identifier`, `example` and `match` say (a ticket number, a contract ID, a web domain).
- `fields`: built from named values the job collects, such as a candidate's email and the requisition: `{ "mode": "fields", "fields": [ { "name", "identifier", "example", "match" } ] }`. Felspar normalizes each value by its own match rule and hashes them in name order, so the order an agent sends them in never matters.
- `content`: the text being judged is the case, such as a marketing claim or a draft email: `{ "mode": "content", "question": "<the question whose excerpt is that text>" }`. The same text is the same case.
- `issued`: there is no ID (an anonymous chat inquiry). Felspar issues one on the first decision; the agent reuses it for retries, output checks and actions. It is the default for a v2 subject without an identifier, and for a v2 job without a subject.

Pick the mode from what the document says identifies a case; ask the person when it doesn't say. A related record is always sent by its own ID, so a subject used as a relationship target needs an `identifier` and `example`.

## Relationships

`relationships: [{ name, subject, cardinality, required, definition }]` on a subject links it to another declared subject.

- `cardinality`: `one` (a ticket has one customer) or `many` (a contract's affiliates). For `many`, agents send each ID in a list and say which one each excerpt comes from.
- `required: true` when the document says a decision needs that record. A question `about` a required relationship with no record or no evidence leaves the case as a gap naming it ("missing: counterparty"); Felspar never decides without it.
- `definition` quotes what the relationship is.
- A question whose evidence comes from the related record says `about: "<name>"`. The result records which related IDs supplied evidence for which questions.

## Escalations

`escalations: [{ id, quote }]` lists each hand-off the document names. Name the kind of review (`human-review`, `legal-review`, `risk-review`), never a tool, channel or person: the company's own code decides what each id does. Point at them with `escalate` on an outcome, on `whenUnresolved` (`{ "quote", "escalate" }`) and on `attempts`. A test case may assert `expect.escalation`.

Customer code maps escalation ids to actions. When you revise a pack, keep every escalation id, subject type and relationship name the current release has. `check_pack`, `propose_pack` and `backtest` list any removal or rename under `breaking`, with how many agents run the job; report each one to the person.

## Worked shapes

Quotes are elided; take each from the document.

**Support desk** (`triage-ticket`): subjects `ticket` (exact, "48213") and `customer` (exact, "cus-9f2"); `ticket` → `customer` (one, optional); a plan question `about: "customer"`; escalations `human-review` (unresolved, attempts) and `incident-review` (the on-call outcome).

**Legal NDA review** (`review-nda`): subjects `contract` (exact, "CTR-2026-0117"), `company` (case-insensitive, "northwind.com") and `matter` (exact, "M-4410"); `contract` → `counterparty`: company (one, required), `affiliates`: company (many), `matter`: matter (one); a governing-law question `about: "counterparty"`, a sanctions question `about: "affiliates"`; escalations `legal-review` and `compliance-review`.

**Recruiting** (`screen-candidate`): subject `candidate` identified by `fields` (email, case-insensitive, and requisition number); escalation `hiring-manager-review`.

**Marketing claim check** (`check-claim`): subject `claim` identified by its `content` (the claim text sent for the competitor question).

**Code change review** (`review-change`): subjects `change` (exact, "api#4412", repository plus change number), `person` (case-insensitive, "dev@example.com") and `area` (exact, "billing"); `change` → `author`: person (one, required), `areas`: area (many); a sensitivity question `about: "areas"`; escalations `security-review`, `second-approver` and `human-review` (unresolved, attempts).

**Anonymous chat inquiry** (`route-inquiry`): no subject; v2 issues each case an ID.

**Sales qualification** (`qualify-company`): one subject `company`, case-insensitive by web domain; escalation `sales-review` for the review outcome and the unresolved path.

If the document doesn't say how a record is identified, which records a decision needs, or who reviews a held case, ask the person and list it as a gap; never invent one.
