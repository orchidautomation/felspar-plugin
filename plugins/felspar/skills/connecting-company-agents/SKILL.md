---
name: connecting-company-agents
description: Registers a company agent with Felspar and gets it working - chooses its jobs, picks Latest or a pinned release for each, and gives the agent's builder the MCP or REST setup. Use this skill when someone wants an agent, bot, Dust, n8n, Eve, Claude, ChatGPT or a script to use their company's decisions from Felspar, or asks how to connect, pin, update, rotate or revoke an agent's access, or how a connected agent should pick its job, which case ID to send, or what to do with an escalation. Not for making decisions inside a running agent; Felspar's agent tools carry that method.
compatibility: Requires personal sign-in for the Felspar connector (list_jobs, list_agents, read_job_activity, list_decisions, read_decision). Owners register agents, change jobs, revoke and rotate keys on the Felspar web page; no MCP tool changes an agent. Keys are shown there once.
metadata:
  author: felspar
  version: "0.3.1"
---

# Connecting company agents

Each company agent is its own identity in Felspar: its own key, the jobs it may run and an accountable person. It can run decisions; it can never approve, release or read review material.

## Workflow

Use only tools advertised by the connected server. A missing tool is unavailable, not permission to invent an action or change credentials.

1. **Identify the agent and its jobs.** `list_agents` shows existing registrations, assignments, selected releases, last use and revocation state; follow `next.after` for more. Reuse the returned identity and management link, never request a key. For a new system (for example "Warehouse Router"), the owner registers its own agent so it can be revoked separately.
2. **Choose Latest or pinned for each job.** Default to **Latest**: the agent gets each approved release automatically. Pin only for a contractual promise, a regulated decision that needs a validated version, or a trial of a new release on one agent first. Also choose its environment: **production** (the default) runs what production serves; a **staging** key runs only what is released to staging, to try a change with a test copy of the agent first. Staging keys follow Latest only, and a key's environment never changes: register a new key to move it.
3. **Have the owner use the Felspar management page.** For a new agent, register it under Workspace → Company agents with its jobs and Latest or pinned for each. For an existing agent, use its returned management link to adjust only what the person requested. A new key is shown once; the person stores it in their secret manager, never in chat.
4. **Hand over the setup:** give the builder [the agent setup reference](references/agent-setup.md). Everything job-specific comes from the job's brief (`get_job_requirements`, or `GET /v0/jobs/{job}/brief`) on the release the agent runs, so there is nothing per job to hand over.
5. **Check it works:** have the builder run one decision. With personal sign-in, use `list_decisions` filtered to the returned agent/job and `read_decision` with its `resultId` to inspect the observed result. `read_job_activity` shows assigned agents and retained 7/30-day outcome counts. Report unavailable or denied evidence honestly; these reads are not a way to execute a decision with a person token.

## Gotchas

- **Send excerpts, not records.** Felspar keeps what an agent sends for each question, so the builder's agent sends the passages that answer and nothing a decision doesn't need. Felspar doesn't redact personal data. What it stores, for how long and who else sees it: [Security, privacy and data](https://docs.felspar.ai/security).
- **Batch agents** that decide many cases in one run should keep one release for the whole run, so a release landing mid-run doesn't mix results.
- **Rotation is the owner's click.** On the returned management page, ordinary rotation preserves the agent and its jobs and keeps the previous key valid for 15 minutes. The owner installs the replacement in the agent's secret manager before the overlap ends. For a leaked key, use compromised rotation: all previous keys stop immediately, even if replacement issuance fails. Do not wait out an overlap or register another agent merely to rotate. Revoking the agent remains a separate action that stops it. Never accept or echo a key in chat.
- **A 403 for a job** means the agent isn't registered for that job; the owner adds the job to the agent on the Workspace page.
- **A retired job refuses every call** with `job_retired`, pinned agents included, and leaves `list_jobs`. The refusal names the `replacement` job when the owner gave one: move the agent to it on the Workspace page. Only the owner retires or restores a job, on the web page (Jobs).
- **Pinned agents don't get fixes.** List them when a release goes out, and move them deliberately.
- **No hard-coded routing.** An agent with several jobs calls `list_jobs` and picks by each job's purpose, use-when and not-for; if none fits, it says so.
- **Escalations are typed.** A result or refusal may carry `escalation.id`; the agent reports it exactly and hands off, and the builder's code maps each id to an action. Felspar returns the id and stops.

## Next actions

End every reply with where the person is and the one next thing: a question with your recommended answer, or an action, and who does it. Blockers first. Common situations:

| Situation | Next step | Who |
| --- | --- | --- |
| No released pack for the job yet | Release one first with `approving-and-releasing-packs` | An approver |
| Agent not registered | Register it under Workspace → Company agents on the web page; store the key in a secret manager | An owner |
| A 403 for a job | Add the job to the agent on the Workspace page | An owner |
| `job_retired` | Move the agent to the `replacement` job on the Workspace page, or ask the owner to restore the job | An owner |
| Key may have leaked | Use compromised rotation on the agent's management page: immediate cutoff, then install the replacement in its secret manager | An owner |
| Routine key replacement | Ordinary rotation on the management page; install the new key before the 15-minute overlap ends | An owner |
| Setup handed over | The builder runs one decision; inspect the retained result with the person's decision reads | The builder, then you |
| The agent's code acts on decisions | Verify each receipt before acting (see the setup reference) | The builder |
