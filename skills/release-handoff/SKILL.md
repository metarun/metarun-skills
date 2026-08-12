---
name: release-handoff
description: >-
  Finish an iOS release after the build is uploaded. When a build has been
  uploaded to App Store Connect by any tool (fastlane, asc, Xcode, Xcode
  Cloud, CI) and the user wants it shipped: attach the build to the version,
  clear export compliance, run the readiness sweep, fix what blocks, and
  submit for review through MetaRun. Trigger phrases: "ship it", "submit the
  build", "finish the release", "send 2.4 to review", "the build is up".
---

# Release handoff: from uploaded build to submitted release

You are completing a release through MetaRun. The build side (compile, sign,
upload) is already done by whatever tool the project uses; your job is
everything after, and you never need App Store Connect open in a browser.

## Ground rules (read once, apply always)

- **Transport.** Use MetaRun's tools via the MCP server (`metarun` tools in
  your tool list) if connected. Otherwise use the CLI: `npx metarun run
  <tool> '<json>'`. Same tools, same behavior. If neither is set up, run
  `npx metarun login` (needs a token from metarun.dev Settings -> MCP).
- **Schemas are live, never guessed.** Before calling a tool with arguments
  you have not used in this session, fetch its input schema:
  `npx metarun tools <tool-name> --json` (or read it from the MCP listing).
  Do not invent parameter names from memory.
- **Every write is a preview/apply pair.** The preview returns a diff and a
  `confirmToken`; the apply requires that token and fails if the store
  changed in between (then: re-preview, show the user, re-apply). Never
  skip showing the user what a preview returned before applying it.
- **Resolve ids, never guess them.** Version ids come from
  `apple_list_versions`, build ids from `apple_list_builds`. A bundle id the
  user gives you ("com.example.app") is the only id you may take on faith.

## The mission

### 1. Locate the app and the editable version

- `apple_list_apps` -> find the app by bundle id or name.
- `apple_list_versions` -> the newest versions carry state. You need the one
  editable version (states like PREPARE_FOR_SUBMISSION). Apple allows only
  ONE editable version at a time.
- No editable version? Create one: `apple_preview_create_version` ->
  confirm with the user -> `apple_apply_create_version`. Wrong version
  number on an existing draft? Renumber with `apple_preview_set_version_string`
  (never delete-and-recreate; Apple permits one draft).

### 2. Find the processed build

- `apple_list_builds` -> newest first, with processing state and encryption
  flags. The build you want must be state PROCESSED/VALID. If it is still
  processing, tell the user and wait or check back; do not attach an
  unprocessed build.
- If `usesNonExemptEncryption` is unset, the build sits on "Missing
  Compliance" and cannot ship: run `apple_preview_declare_build_encryption`
  (most apps using only HTTPS/standard crypto answer both questionnaire
  flags false = exempt), confirm, `apple_apply_declare_build_encryption`.
  This one is not revertible; say so when you show the preview.

### 3. Attach the build to the version

- `apple_preview_set_version_build` with the version id and build id ->
  show the diff -> `apple_apply_set_version_build` with the confirmToken.

### 4. Run the readiness sweep

- `apple_preview_submit_for_review` is the sweep: it blocks with a
  per-language checklist of everything Apple still requires. Read the result
  as a work list, not an error.
- Common blockers and their fixes:
  - Missing What's New (updates only): `apple_preview_set_version_localization`
    per language. On a FIRST release Apple has no What's New field at all;
    do not try to set it.
  - Missing privacy policy URL: `apple_preview_set_app_info_localization`.
  - Missing description/keywords/support URL per language:
    `apple_preview_set_version_localization`.
  - An earlier submission still in Apple's queue:
    `apple_preview_cancel_review_submission` (confirm with the user first;
    it withdraws a live submission).
- First submission only: the sweep will carry an App Privacy warning. App
  Privacy has NO App Store Connect API; it is the one step that must be done
  once by hand in the browser. Tell the user plainly; do not hunt for a tool
  that does not exist.

### 5. Release options (ask, do not assume)

If the user has not said how it should release, ask once: automatic on
approval, manual, or phased. Set via `apple_preview_set_release_options` /
`apple_apply_set_release_options`. After approval, `apple_apply_manual_release`
releases a manually-held version and `apple_apply_phased_release` manages a
staged rollout.

### 6. Submit

- Re-run `apple_preview_submit_for_review` until it returns clean (warnings
  may remain; blocks may not).
- Show the user exactly what will be submitted.
- `apple_apply_submit_for_review` requires the confirmToken AND
  `acknowledge: true`. The acknowledge flag is the user's explicit yes to an
  outward action: never set it without their clear go-ahead in this
  conversation.

### 7. Close the loop

Report what was submitted and remind the user: every change you applied is
in MetaRun's change history, attributed and revertible (except the noted
exceptions), and review-state changes will show on the Releases deck.

## Beyond this mission

MetaRun's surface is much larger than this workflow: pricing across 175
territories, metadata and translation, screenshot rendering and publishing,
ASO and answer-engine visibility, reviews, TestFlight. Discover it live with
`npx metarun tools <filter>` (for example `tools pricing`, `tools aso`);
never assume a capability is missing before searching for it.
