# SolCon Prompt for v0 (Finisher)

Patch only. Preserve existing UI/UX.

## File handling rule (critical)

These files are contract. You must copy them byte-for-byte into the project using
Move(operation="copy") from user_read_only_context, NOT by reading and rewriting.

Do NOT use WebFetch-and-Write to reconstruct these files. Do NOT rewrite them from memory.
If files were attached directly as chat attachments, use Move to copy them.
If files were cloned via WebFetch, you MUST re-read the written output and verify
the critical values listed below before continuing.

After copying or writing each reference file, immediately re-read it and print:
- The exact value of `DEFAULT_LOCK_MS` (must be `300`)
- The exact body of `normalizeUserId` (must be `.trim()` only, no `.toLowerCase()`)
- The `listenForNative` callback signature (must be `changeUserFn(userId, detail)`)

If any value does not match, stop and fix it before proceeding.

## Source of truth

- Use only files in this same uploaded finisher folder.
- Ignore any `solcon-*` folders already present inside the target app.
- If duplicate guidance exists, this file wins.

## Reference fidelity rule

- Treat reference files as contract, not inspiration.
- Do not invent extra normalization, lock values, or event-forwarding paths.
- If output behavior differs from reference files, output is invalid.

Reference:
- `./demo_bridge_entry.js` -> `lib/bridge-entry.ts`
- `./sync_state_reference.js` -> `lib/sync-state.ts`
- custom event helper -> `lib/track-event.ts`

## Execution rule for custom events

- Route all custom events through one `trackEvent(name, properties)` helper.
- The helper must call `braze.logCustomEvent` only.
- Do not add `DemoBridge.logEvent` or `DemoBridge.logCustomEvent` calls in app code.

## Must enforce

1) One bridge entrypoint file, one import path across the app (no mixed bridge modules), native callback signature `changeUserFn(userId, detail)`.
2) One sync-state writer for identity changes, one shared default user constant for first session + Braze init.
3) Preserve user IDs exactly as received (case-sensitive). `.trim()` only. Never `.toLowerCase()`.
4) `DEFAULT_LOCK_MS = 300` and `manualLockMs = 300`. No other value. Not 3000. Not 30. Exactly 300.
5) One identity action = one bridge session write path.
6) Dynamic configId (prop or env variable, never hardcoded string).
7) One custom-event helper only (no parallel provider/context event path).
8) Canonical event forwarding: `trackEvent -> braze.logCustomEvent` only.
9) Starter continuity: no DemoBridge custom-event forwarding in finisher mode.
10) Identity ownership: all web-origin identity writes flow through one `setUser()` path only.

## Identity write rule (hard-fail if violated)

All Braze identity writes (`braze.changeUser`, `braze.openSession`) and bridge identity writes
must be owned by one path: the sync state machine `setUser` callback.

Init, manual switches, and native-origin updates must flow through `applyIncomingSync` into that owner path.

Do not call `braze.changeUser` directly in UI components, providers, or listeners outside that owner path.

## Hard-fail (reject output immediately if found)

- `toLowerCase()` anywhere in identity/sync codepaths
- `DEFAULT_LOCK_MS` or `manualLockMs` set to anything other than `300`
- native listener callback that drops `detail` payload
- identity change path that writes bridge session twice or mixes multiple bridge modules
- `DemoBridge.logEvent` or `DemoBridge.logCustomEvent` event forwarding in app code
- direct `braze.changeUser`/`braze.openSession` calls outside the single identity owner path

## Preflight gate (must pass before writing any app code)

Before editing any application file, print this report:

```
PREFLIGHT REPORT
================
Prompt filename: SOLCON_PROMPT_V0.md
DEFAULT_LOCK_MS in sync-state file: [print exact value from file]
normalizeUserId body: [print exact function body from file]
listenForNative signature: [print exact callback pattern from file]
toLowerCase in identity path: yes/no
Mixed bridge imports: yes/no
Legacy prompt filenames referenced: yes/no
```

If any value is wrong, stop and fix the reference file before continuing.

## Post-write verification (must pass before reporting done)

After all edits are complete, grep the entire project and confirm:
- Zero matches for `toLowerCase(` in any file under `lib/sync-state` or `lib/bridge-entry`
- The string `300` appears as the lock value (not `3000`, not `30`)
- `braze.changeUser` appears only in the single identity owner file
- `window.DemoBridge` appears only in the single bridge entry file
- No files reference `_NEW` in any prompt filename

Print the verification results. If any check fails, fix it before reporting done.

## Cleanup

After patching is complete, remove all `solcon-*` instruction/reference folders
from the project to prevent re-read drift in future sessions.
