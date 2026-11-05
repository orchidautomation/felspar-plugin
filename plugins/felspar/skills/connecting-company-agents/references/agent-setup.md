# Agent setup to hand to a builder

Give the builder three things: the MCP address, the agent's key (through their secret manager, never chat), and these instructions.

## MCP clients

- **Claude Code:** `claude mcp add --transport http felspar <mcp-url> --header "Authorization: Bearer $FELSPAR_AGENT_KEY"`
- **Other MCP clients** (Codex, Dust, n8n, Eve): add a Streamable HTTP server at the MCP address with the header `Authorization: Bearer <agent key>`.

The agent sees `list_jobs`, `get_job_requirements`, `get_job_decision` and `check_output`. It needs no Felspar skill: the tools carry the method, and `get_job_requirements` carries everything job-specific.

Over MCP, `get_job_requirements` returns a compact brief whose `input` lists the arguments and a short `facts` example. `get_job_decision` returns a compact result: the outcome, why, what is still open, the outputs, any escalation, the context capsule, `explanation.evidenceFreshUntil` and the full receipt, which is everything the code that acts and the SDK action gate need. `detail: "full"` returns the REST result unchanged, for code that inspects a decision. The agent sends no execution ID: calling again with the same arguments within the same hour returns the decision already made, and a `blocked` result means that first call is still running. The system that acts on a decision verifies the receipt over REST (`POST /v0/executions/{id}/verify`) with the same key before acting. When the pack has output checks, the agent checks what it wrote with `check_output` before sending it, and the SDK action gate can require that pass (`outputCheck` on a tool rule).

## REST

Every request carries `Authorization: Bearer <agent key>`.

```text
GET  /v0/jobs
GET  /v0/jobs/{job}/brief
POST /v0/jobs/{job}/decisions
     { "mode": "facts", "executionId": "<new UUID; reuse to retry>", "jobId": "<job>",
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

## What to tell the builder

- If the agent wasn't told which job to run, it calls `list_jobs` and picks the job whose purpose, use-when and not-for fit; if none fits, it says so instead of forcing one. Don't hard-code job routing in its instructions.
- Identify each case the way the brief's `identity` says: the record's own ID (a ticket number, a contract ID, a web domain) as `entityId`; the named values under `identity` for a fields job; nothing for a content job; nothing on the first decision of an issued job, then the `case.id` the result returned, for every retry, output check and action on that case. An issued ID belongs to the agent that got it; another's is refused. For a question about a related record, send that record's ID under `related` and quote that record.
- Send the source's exact words for each question, never a label. `source.id` is the page URL exactly as fetched (query included) or the record's own ID; `source.observedAt` is a date and time with `Z` or an offset, such as `2026-10-21T10:00:00Z`.
- Words a person gave the agent are the evidence: quote them as given with `source: { class, manual: true, locator }`, naming the source they came from in `locator`, with no `id` or `observedAt` (Felspar stamps the time). Never cite a page the agent didn't read. A job whose brief has `manualFacts: { allowed: false }` refuses them (`manual_facts_refused`).
- Act only on Felspar's outcome; treat "needs evidence" as unresolved.
- When a result (or an `attempt_limit_reached` refusal) carries `escalation`, report `escalation.id` exactly and hand the case off that way; don't ask again, decide yourself, or invent an escalation. The builder's code maps each id to its action (a review queue, a message, another job); Felspar returns the id and stops.
- Verify the receipt before a consequential action.
