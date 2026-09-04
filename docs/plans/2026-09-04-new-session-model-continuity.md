# New-Session Model Continuity

## Goal

Keep Pi's active model and thinking level stable when an extension replaces the current runtime with `ctx.newSession()`, and align Cameron's local OpenAI Codex context metadata with each model's verified limit so compaction occurs before overflow.

## Requirements

- `ctx.newSession()` MUST preserve the source runtime's selected model when that model remains available in the replacement runtime.
- `ctx.newSession()` MUST preserve the source runtime's thinking level, clamped through the replacement model's supported levels.
- Existing session replacement behavior MUST remain unchanged when the source model is unavailable after runtime reconstruction.
- The affected session's parent/child model transition MUST be covered by a regression test.
- Cameron's `openai-codex` configuration MUST keep Sol's verified 1,050,000-token override while leaving GPT-5.5 at the generated 272,000-token limit.
- Verification MUST establish the effective context metadata and exercise the runtime regression test.

## Chosen Approach

Capture the active model identity and thinking level before teardown, then ask the newly created `AgentSession` to select the equivalent model through its replacement `ModelRegistry`; if resolution fails, retain the replacement runtime's normal fallback selection and surface existing diagnostics rather than making session creation fail.

Narrow Cameron's `openai-codex.modelOverrides` block to Sol only because the live Sol session confirms its 1,050,000-token window, while the failed child session proves GPT-5.5 must retain the generated 272,000-token limit.

## Alternatives Declined

- Fix only `pi-plan-mode`: model loss is caused by generic `ctx.newSession()` runtime reconstruction and affects every extension using that API.
- Add a model argument to `ctx.newSession()`: preserving current runtime state is the expected default and should not burden every caller.
- Hard-code unverified GPT model limits in another local file: the generated catalog remains authoritative except for Sol's independently verified 1,050,000-token window.

## Status

Implementation is committed and pushed to `cameronsjo/pi` branch `fix/new-session-model-continuity`. The local model configuration is active. A clean two-file PR branch is pushed as `fix/new-session-model-continuity-pr` at `58cc440b7`; upstream submission remains deferred because `earendil-works/pi` requires maintainer `lgtm` and the local non-owned-repository guard blocked `gh pr create`.

## Next Step

Cameron should obtain upstream maintainer `lgtm`, then open the prepared comparison at <https://github.com/earendil-works/pi/compare/main...cameronsjo:pi:fix/new-session-model-continuity-pr?expand=1>.

## Checklist

- [x] Add a runtime regression test for model and thinking continuity.
- [x] Preserve active model and thinking level across `AgentSessionRuntime.newSession()`.
- [x] Confirm no public API documentation change is needed; upstream `CONTRIBUTING.md` requires maintainers to add changelog entries.
- [x] Remove stale OpenAI Codex context overrides while retaining Sol's verified 1,050,000-token override.
- [x] Run focused tests, type diagnostics, and effective-model validation.
