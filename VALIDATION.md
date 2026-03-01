## SolCon Finisher Validation (v0)

1) No session spam while idle.
2) Web switch and native switch each apply once (no bounce/duplicates).
3) Native callback forwards `detail` through unchanged.
4) User IDs are case-preserved (no lowercasing in identity flow).
5) Lock window is 300ms.
6) Custom events route through one helper and forward correctly (no parallel event API path).
7) Browser fallback does not crash without bridge.
8) Direct `window.DemoBridge` calls exist only in the single chosen bridge entry file.

Reject if found:
- `toLowerCase()` in identity codepaths.
- `DEFAULT_LOCK_MS` or `manualLockMs` not equal to `300`.
- native callback that ignores `detail`.
- `DemoBridge.logEvent` or `DemoBridge.logCustomEvent` in event flow
- mixed bridge imports (starter + finisher bridge modules together)
- legacy prompt filename references
- pack drift (using app-embedded `solcon-*` folders instead of uploaded finisher files)
- direct `braze.changeUser`/`braze.openSession` calls outside the single identity owner path
- extra event-forwarding path beyond `trackEvent -> braze.logCustomEvent`

Required validation evidence:
- Report which prompt filename was used (`SOLCON_PROMPT_V0.md` expected).
- Report the final `DEFAULT_LOCK_MS` value found in app code.
- Report whether `normalizeUserId` preserves case (`trim` only).
- Report where Braze identity writes occur (must be one owner path only).
- Report whether any native event forwarding API is used from app code (must be `no`).
