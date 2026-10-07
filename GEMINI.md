# MetaRun

The `metarun` MCP server operates App Store Connect for the user: versions and
builds, App Review submissions, release controls, listing text in every
language, screenshots, prices in 175 storefronts, subscriptions and offers,
customer reviews, TestFlight, and ASO keyword tracking.

How to use it well:

- Read before you write. List the user's apps (`apple_list_apps`) and the
  version you are working on before staging anything.
- Every change is two calls. A `*_preview_*` tool returns the before and
  after plus a `confirmToken`; the matching `*_apply_*` tool needs that token
  and refuses if App Store Connect changed in between. Show the user the
  preview and wait for their approval before applying.
- Submitting for review also needs `acknowledge: true` on the apply.
- App Privacy answers and App Review's messages have no App Store Connect API.
  Do not search for a tool that writes them; `apple_build_privacy_answers`
  builds the answers from PrivacyInfo.xcprivacy files for the user to enter.
- The user's Apple key never reaches you. If a tool says the key is missing,
  point them to https://metarun.dev/settings/platforms.

Setup guide: https://metarun.dev/gemini-cli
