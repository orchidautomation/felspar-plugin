---
name: connecting-company-agents
description: Registers a company agent with Felspar and gets it working - chooses its jobs, picks Latest or a pinned release for each, and gives the agent's builder the MCP or REST setup. Use this skill when someone wants an agent, bot, Dust, n8n, Eve, Claude, ChatGPT or a script to use their company's decisions from Felspar, or asks how to connect, pin, update, rotate or revoke an agent's access, or how a connected agent should pick its job, which case ID to send, or what to do with an escalation. Not for making decisions inside a running agent; Felspar's agent tools carry that method.
compatibility: Agents are registered, given jobs and revoked by the workspace owner on the Felspar web page (Workspace → Company agents); no tool changes an agent. Keys are shown there once.
metadata:
  author: felspar
  version: "0.3.0"
---

# Connecting company agents

Each company agent is its own identity in Felspar: its own key, the jobs it may run and an accountable person. It can run decisions; it can never approve, release or read review material.

## Workflow

1. **Name the agent and its jobs.** One agent per deployed system (for example "Outbound Researcher"), so it can be revoked on its own.
2. **Choose Latest or pinned for each job.** Default to **Latest**: the agent gets each approved release automatically. Pin only for a contractual promise, a regulated decision that needs a validated version, or a trial of a new release on one agent first. Also choose its environment: **production** (the default) runs what production serves; a **staging** key runs only what is released to staging, to try a change with a test copy of the agent first. Staging keys follow Latest only, and a key's environment never changes: register a new key to move it.
3. **Register the agent on the Felspar web page** (Workspace → Company agents) with its jobs and Latest or pinned for each. The key is shown once. Tell the person to store it in their secret manager; never paste it into chat.
4. **Hand over the setup:** give the builder [the agent setup reference](references/agent-setup.md). Everything job-specific comes from the job's brief (`get_job_requirements`, or `GET /v0/jobs/{job}/brief`) on the release the agent runs, so there is nothing per job to hand over.
5. **Check it works:** have the builder run one decision and confirm it shows up for that agent.

## Gotchas

- **Send excerpts, not records.** Felspar keeps what an agent sends for each question, so the builder's agent sends the passages that answer and nothing a decision doesn't need. Felspar doesn't redact personal data. What it stores, for how long and who else sees it: [Security, privacy and data](https://docs.felspar.ai/security).
- **Batch agents** that decide many cases in one run should keep one release for the whole run, so a release landing mid-run doesn't mix results.
- **Revoking** an agent stops it within a minute, including its MCP connection. Revoke a key that may have leaked rather than waiting.
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
| Key may have leaked | Revoke it now and issue a new one | An owner |
| Setup handed over | The builder runs one decision; confirm it shows up for that agent | The builder |
| The agent's code acts on decisions | Verify each receipt before acting (see the setup reference) | The builder |
