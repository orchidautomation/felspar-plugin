# Agent setup to hand to a builder

Give the builder three things: the MCP address, the agent's key (through their secret manager, never chat), and these instructions.

## MCP clients

- **Claude Code:** `claude mcp add --transport http felspar <mcp-url> --header "Authorization: Bearer $FELSPAR_AGENT_KEY"`
- **Other MCP clients** (Codex, Dust, n8n, Eve): add a Streamable HTTP server at the MCP address with the header `Authorization: Bearer <agent key>`.

The agent sees `list_jobs`, `get_job_requirements`, `get_job_decision` and `check_output`. It needs no Felspar skill: the tools carry the method, and `get_job_requirements` carries everything job-specific.

Over MCP, `get_job_requirements` returns a compact brief whose `input` lists the arguments and a short `facts` example. `get_job_decision` returns a compact result: the outcome, why, the release's `reader`, what is still open, the outputs, any escalation, the context capsule, `explanation.evidenceFreshUntil` and the full receipt, which is everything the code that acts and the SDK action gate need. `detail: "full"` returns the REST result unchanged, for code that inspects a decision, adding the answers it used with the exact words each rests on (`facts`) and every rule that matched (`matchedRules`, the deciding one marked). The agent sends no execution ID: an identical call on the same release within the same UTC hour returns the stored decision and isn't counted again (after a new release, it is decided again on that release), so resending one never changes the result, and a `blocked` result means that first call is still running. The system that acts on a decision verifies the receipt over REST (`POST /v0/executions/{id}/verify`) with the same key before acting. When the pack has output checks, each check that states the case's answer carries it as `answer`: the agent writes that settled value (on its labelled line for a number or date), never one from its own notes or another case, and states none when the answer isn't settled. The agent checks what it wrote with `check_output` before sending it, and the SDK action gate can require that pass (`outputCheck` on a tool rule).

## REST

Every request carries `Authorization: Bearer <agent key>`.

```text
GET  /v0/jobs
GET  /v0/jobs/{job}/brief
POST /v0/jobs/{job}/decisions      (Idempotency-Key: <your ID for this decision>)
     { "case": "<the case's ID>", "shadow"?: true,
       "records": [{ "kind": "<a source class the brief lists>", "id"?: "<record ID>", "fetchedAt"?: "<when fetched>",
                     "content": "<the record's text>" | { <the record's JSON> } }],
       "fields"?: [{ "question": "<question key>", "value": <typed value>, "source": { "class": "<a fields.kinds kind>", "id", "observedAt" } }] }
     or the older "evidence": [{ "kind", "text": "..." | "toolCall": { "name", "input" }, "id"?: "<source ID>" }] instead of records
     → every answer, refusals included, carries decision: { action: allow | flag | block, effect: allow | flag | block | null, reason, rule, nextStep,
       missing, escalation: { id, trigger, text }, resultId, receiptUrl, open: [{ key, label }], toProceed }
     or { "mode": "facts", "executionId": "<new UUID; reuse to retry>", "jobId": "<job>",
       "entityId": "<the case's ID, as the brief's subject says>",
       "related"?: { "<relationship>": "<its ID>" | ["<ID>", ...] },
       "facts": { "<question>": { "text": "...", "source": { ... }, "about"?: "<related ID>" } } }
POST /v0/jobs/{job}/output-checks
     { "executionId": "<the decision's executionId>", "text": "<the exact text>", "output"?: "<output id>" }
GET  /v0/jobs/{job}/output-checks/{checkId}
POST /v0/executions/{executionObjectId}/verify
     { "receipt": { ... }, "expectedReceiptDigest": "<receipt.receiptDigest>", "deliveredCapsule": <result.context> | null }
```

The job's brief carries the job's questions and this request's format (`input.jsonSchema` over REST).

Records are the default request (`decisions.create` in the TypeScript SDK): the case, each record whole as its system returned it, labelled by kind, and the job's clean fields (sent only from a system key). Felspar files each record under every question that takes its kind, so a pack that adds a question needs no code change. Send `Idempotency-Key` (a UUID you store before sending) so a resend is the same decision in any hour. Over MCP, `get_job_decision` takes the same `records` and `fields` in place of `facts`. Send records or `evidence`, never both (`evidence_and_records`).

Evidence by kind, the older form of the same request, suits a hook or script that sees the same kinds of evidence every time, such as a coding agent's tool call and its change record: it labels each piece by kind and Felspar files it under every question that accepts that kind, so a pack that adds a question needs no code change. A coding agent's call goes as `toolCall`, as its runtime reports it; Felspar reads it, so the hook builds no text. The hook acts on `decision.action` alone: `block` covers every refusal and anything undecided. It reports by `decision.status`, so an outage never reads as the policy's block: `decided` is the policy's outcome, `held` waits on evidence or a person, `failed` means nothing was decided (say so, and don't present it as the policy stopping the call), `processing` means the same call is still being decided (resend it unchanged). It needs no execution ID (the same body in the same hour, on the same release, returns the same result) and no release digest; `shadow: true` records a call the hook won't act on. Send facts by question when the agent gathers evidence question by question, or for a question about a related record.

## What to tell the builder

- If the agent wasn't told which job to run, it calls `list_jobs` and picks the job whose purpose, use-when and not-for fit; if none fits, it says so instead of forcing one. Don't hard-code job routing in its instructions.
- Identify each case the way the brief's `identity` says: the record's own ID (a ticket number, a contract ID, a web domain) as `entityId`; the named values under `identity` for a fields job; nothing for a content job; nothing on the first decision of an issued job, then the `case.id` the result returned, for every retry, output check and action on that case. An issued ID belongs to the agent that got it; another's is refused. For a question about a related record, send that record's ID under `related` and quote that record.
- Send each record's field or line whole, in the source's exact words. A form row, a status line or a list of "label: value" parts is one excerpt; don't split it into its parts, so a part that qualifies or overrides another (standing water written beside "Active water: N", a damage note beside a theft label) is sent with it. A typed value goes with the passage that states it, at most 2,000 characters; send the whole record as text in its own excerpt.
- Send the source's exact words for each question, never a label. `source.id` is the page URL exactly as fetched (query included) or the record's own ID; `source.observedAt` is when the agent read the source (never a date printed in it or when the record last changed), a date and time with `Z` or an offset, such as `2026-10-21T10:00:00Z`. Evidence sent without it is accepted where the API allows, but never counts as fresh, so its question holds. Each question's `evidencePolicy.freshness` in the compact brief says how recent that reading must be.
- Send everything found in one request: every question, conditional ones (`appliesWhen`) included, and always the subject's own record. Quote the record's own words, negative or uncertain ones included; where nothing states an answer, leave that question out, never the agent's own "not found" or a guess. Give each passage its own locator. The compact brief's `matters` says what the policy checks about each question, and `answer.format` how its records write a number or date.
- When a record the agent tried fails to load (not found, access denied, a server error, a timeout), call `get_job_decision` with it under each question it was read for as `{ notLoaded: "not_found" | "access_denied" | "error" | "timeout" | "other", source }`, with no text: never quote the error page's words, and don't leave the record out. Over REST evidence by kind, send the piece as `{ kind, notLoaded, id }`. Such a question holds for review, and its retry (`reload_record`) says to load the record again and send what it says, or to follow `nextStep` or the escalation if it still won't load. Felspar reads a short text that is only a failed load, such as `404 Not Found` or an empty page, the same way.
- Attempts count per case over 24 hours, across every agent key in the environment, and start again after each decision. A retry must close a question or quote a place no earlier request quoted; rewording a passage from the same place isn't progress, and a retry without progress hands the case off.
- Words a person gave the agent are the evidence: quote them as given with `source: { class, manual: true, locator }`, naming the source they came from in `locator`, with no `id`, and `observedAt` only when the agent knows when the person gave them: without a time they never count as fresh. Never cite a page the agent didn't read. A job accepts them only when its brief has `manualFacts: { allowed: true }`; otherwise they are refused (`manual_facts_refused`), so quote the system record instead.
- Act only on Felspar's outcome; treat "needs evidence" as unresolved.
- When a result carries `retry`, each entry's `action` and `fix` say what to do for that question; do that, reading where the fix says. Felspar keeps the excerpts of the agent's held results on the case since its last decision, for 24 hours, and decides again on all of them, so a retry sends only new passages (a refused request keeps nothing); `notUsed` numbers the excerpts of the request just sent. When it says to ask again unchanged, send the same excerpts, leaving none out. Keep an excerpt that points to a different answer. When excerpts disagree, read each of those records again from its source first; when two records disagree and both are true, don't ask again: follow `nextStep`, or hand off through the escalation the fix names. When it says to follow `nextStep`, don't reword the evidence or ask for a policy change to get a decision. When it says to fix `observedAt`, send the time the source was read.
- A result carries `escalation` only when asking again can't help: the decided outcome hands off, the case has used its attempts, or an excerpt reads as instructions to an AI or a question can't be read from text. When a result (or an `attempt_limit_reached` refusal) carries `escalation`, report `escalation.id` exactly and hand the case off that way; don't ask again, decide yourself, or invent an escalation. The builder's code maps each id to its action (a review queue, a message, another job); Felspar returns the id and stops.
- On a `decision` from evidence by kind, code that gates an action acts on `action`. An agent that reports or routes the decision reads `effect`: `null` means Felspar decided an outcome that declares no effect of its own, so do its `nextStep` rather than treating it as a block. Tell the person what the rest says, in its own words:
  - `reason`: why. Relay it; don't restate it.
  - `nextStep`: what the policy says to do now. Do that. After a block, don't retry the same action another way.
  - `open` and `toProceed`: when Felspar couldn't decide, say which facts it couldn't settle (`open[].label`), and either send the evidence `toProceed` names or ask the person to decide.
  - `escalation.text`: who takes it from here. Tell the person; Felspar doesn't send it anywhere.
  - `resultId` and `receiptUrl`: keep them with whatever is done, so the person can look the decision up.
- Verify the receipt before a consequential action.
