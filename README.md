# MetaRun skills

App Store missions for coding agents. Each skill teaches your agent a
complete workflow over [MetaRun](https://metarun.dev)'s hosted tool server:
the same 300-tool registry that powers its dashboard, AI operator, MCP
server, and CLI.

The part that makes these different from other App Store automation: **your
Apple keys never touch the machine running the agent.** Credentials live
envelope-encrypted on MetaRun's servers; the agent holds a revocable token,
and every write goes preview → confirm token → apply, recorded in change
history and mostly one-tap revertible.

## The skills

| Skill | The mission |
| --- | --- |
| `release-handoff` | The build is uploaded (fastlane, asc, Xcode Cloud) → attach it, clear compliance, sweep readiness, fix blockers, submit for review; and after a rejection, read App Review's verdict, fix, resubmit |
| `worldwide-pricing` | One base price → 175 defensible local prices via PPP/Big Mac/Spotify indexes, staged and confirmed |
| `localize-listing` | The agent translates, MetaRun stages: name, subtitle, description, keywords, What's New, per locale |
| `review-inbox` | Triage reviews, draft public replies (approved one by one), mine complaint/praise themes and keyword candidates |
| `aso-audit` | Keyword ranks, metadata lint, rival diffs, Apple's measured search analytics, and answer-engine visibility |
| `store-media` | Screenshots, app preview videos and creative assets through Apple's Asset Library: upload once, place on the listing, custom product pages, events or tests, order, verify (iPhone Duo included) |

## Install

This repo is one plug-in for several agents: the skills plus MetaRun's
hosted MCP server (`https://metarun.dev/api/mcp`). On first use the agent
opens MetaRun in your browser to sign in; nothing to paste.

Claude Code (plugin marketplace):

```bash
claude plugin marketplace add metarun/metarun-skills
claude plugin install metarun-skills@metarun
```

Then run `/mcp` in Claude Code and choose metarun to sign in.

Xcode 27 (agent plug-in): Settings, Intelligence, Plug-ins, Add Plug-in, Add
from URL, then paste `https://github.com/metarun/metarun-skills` and click
Sign in next to MetaRun. One-click: `xcode://agent-plugin-clone?repo=https%3A%2F%2Fgithub.com%2Fmetarun%2Fmetarun-skills`

Cursor: install from the repo URL (`.cursor-plugin/plugin.json`).

Gemini CLI (extension):

```bash
gemini extensions install https://github.com/metarun/metarun-skills
```

Any skills-capable agent (skills only, no server):

```bash
npx skills add metarun/metarun-skills
```

Setup guides for each agent: https://metarun.dev/agents

## Setup

You need a MetaRun account with your App Store Connect key connected (Pro,
Studio, or the 14-day trial). Agents sign in through the browser. For scripts
and CI, use a personal access token instead (Settings, MCP):

```bash
npx metarun login
```

Or wire the server with a token: `claude mcp add --transport http metarun
https://metarun.dev/api/mcp --header "Authorization: Bearer <token>"`.

## What runs, and where data goes

- **MetaRun's MCP server** (`https://metarun.dev/api/mcp`, declared in
  `.mcp.json`): every tool call goes there. Agents sign in with OAuth; your
  App Store Connect key stays envelope-encrypted on MetaRun's servers, which
  call Apple's App Store Connect API on your behalf.
- **The `metarun` CLI** (`npx metarun`, the npm package `metarun`): skills
  use it when the MCP server isn't connected. It calls the same server with a
  personal access token from `npx metarun login` or `METARUN_TOKEN`.
- **Local image uploads**: when a screenshot exists only on your machine,
  `studio_create_upload` returns a short-lived signed URL on MetaRun's file
  storage (Supabase Storage) and a `curl` command that PUTs the file there;
  `studio_finalize_upload` then hands back an https URL for the upload tool.
- Nothing else: the skills send no data to any other service. Applied
  changes are kept in your MetaRun change history for your plan's retention
  period. Privacy policy: https://metarun.dev/privacy

## Design rules these skills follow

- **Schemas are never embedded.** Skills name tools; parameters are fetched
  live (`npx metarun tools <name> --json`), so input changes never stale a
  skill.
- **Tool references are CI-enforced.** Every tool name in every skill is
  asserted against the live registry in MetaRun's build; a rename cannot
  ship without updating the skill.
- **Writes are never assumed.** Previews are shown, confirm tokens are
  passed deliberately, and submission requires an explicit acknowledgment.

## License

MIT
