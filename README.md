# MetaRun skills

App Store missions for coding agents. Each skill teaches your agent a
complete workflow over [MetaRun](https://metarun.dev)'s hosted tool server —
the same 244-tool registry that powers its dashboard, AI operator, MCP
server, and CLI.

The part that makes these different from other App Store automation: **your
Apple keys never touch the machine running the agent.** Credentials live
envelope-encrypted on MetaRun's servers; the agent holds a revocable token,
and every write goes preview → confirm token → apply, recorded in change
history and mostly one-tap revertible.

## The skills

| Skill | The mission |
| --- | --- |
| `release-handoff` | The build is uploaded (fastlane, asc, Xcode Cloud) → attach it, clear compliance, sweep readiness, fix blockers, submit for review |
| `worldwide-pricing` | One base price → 175 defensible local prices via PPP/Big Mac/Spotify indexes, staged and confirmed |
| `localize-listing` | The agent translates, MetaRun stages: name, subtitle, description, keywords, What's New, per locale |
| `review-inbox` | Triage reviews, draft public replies (approved one by one), mine complaint/praise themes and keyword candidates |
| `aso-audit` | Keyword ranks, metadata lint, rival diffs, Apple's measured search analytics, and answer-engine visibility |

## Install

Claude Code (plugin marketplace):

```bash
claude plugin marketplace add metarun/metarun-skills
claude plugin install metarun-skills@metarun
```

Agent-agnostic (skills installer):

```bash
npx skills add metarun/metarun-skills
```

## Setup

The skills drive MetaRun's tools, so you need an account with your App Store
Connect key connected, and a personal access token (Settings → MCP):

```bash
npx metarun login
```

Or wire the MCP server directly: `claude mcp add metarun
https://metarun.dev/api/mcp -t http -H "Authorization: Bearer <token>"`.

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
