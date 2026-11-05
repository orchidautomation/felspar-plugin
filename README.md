# Felspar plugin

Felspar turns your company's policy documents into tested, approved **Decision Packs** that every agent follows, and keeps them current as the policy changes.

This plugin is for the people who run Felspar for a company: turning policy into packs, testing them, reviewing changes, and connecting agents. It adds Felspar's skills and connector to Claude, Codex, ChatGPT and other clients that read [Agent Skills](https://agentskills.io/specification). The first time a Felspar tool runs, you sign in with your Felspar login.

## Install

**Claude Code**

```sh
claude plugin marketplace add orchidautomation/felspar-plugin
claude plugin install felspar@felspar
```

**Codex**

```sh
codex plugin marketplace add orchidautomation/felspar-plugin
codex plugin add felspar@felspar
```

**ChatGPT (workspace owners and admins):** download `felspar-plugin-<version>.zip` from the [latest release](https://github.com/orchidautomation/felspar-plugin/releases/latest) and upload it in the admin console under **Plugins**.

**Any other MCP client:** connect to `https://mcp.felspar.ai/mcp`.

## What's included

| Skill | Use it to |
| --- | --- |
| `planning-agent-jobs` | Plan which decisions in a team's work need a pack and which to build first |
| `preparing-policy-documents` | Write or strengthen a policy so a pack can be drafted from it |
| `drafting-decision-packs` | Turn a connected policy document into a proposed Decision Pack |
| `writing-pack-test-cases` | Write and maintain a pack's test cases |
| `reviewing-pack-changes` | See what a proposed change would do before anyone approves it |
| `triaging-source-changes` | Work through edited policy documents |
| `approving-and-releasing-packs` | Get a pack ready for an approver to release on the Felspar web page |
| `connecting-company-agents` | Register an agent and give its builder the setup |

Approving and releasing a pack, and managing agent keys, are always a person's own click on the Felspar web page, never a tool.

## Learn more

- [felspar.ai](https://felspar.ai)
- [Documentation](https://docs.felspar.ai)
- [Examples](https://github.com/orchidautomation/felspar-examples)
- Support: [felspar.ai/support](https://felspar.ai/support)

This repository is published from Felspar's own source; changes made here are not accepted. Copyright © 2026 Felspar. All rights reserved. See [LICENSE](LICENSE).
