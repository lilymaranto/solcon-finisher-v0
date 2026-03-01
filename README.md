## SolCon Finisher v0

Use this after starter wiring and feature/UI build are done.

Quick invocation:
- `Use the instructions in SOLCON_PROMPT_V0.md to make this PWA compatible with our mobile app.`

Use:
- `SOLCON_PROMPT_V0.md`
- `demo_bridge_entry.js`
- `sync_state_reference.js`
- `VALIDATION.md`

Intent:
- Patch only, no UX redesign.
- Harden identity/session sync only; keep custom events on Braze-only path (`trackEvent -> braze.logCustomEvent`).
