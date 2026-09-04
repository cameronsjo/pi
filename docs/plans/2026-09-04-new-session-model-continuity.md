# New-Session Model Continuity

## Goal

Keep Pi's active model and thinking level stable when an extension replaces the current runtime with `ctx.newSession()`, and align Cameron's local OpenAI Codex context metadata with the endpoint's enforced limit so compaction occurs before overflow.

## Requirements

- `ctx.newSession()` MUST preserve the source runtime's selected model when that model remains available in the replacement runtime.
- `ctx.newSession()` MUST preserve the source runtime's thinking level, clamped through the replacement model's supported levels.
- Existing session replacement behavior MUST remain unchanged when the source model is unavailable after runtime reconstruction.
- The affected session's parent/child model transition MUST be covered by a regression test.
- Cameron's `openai-codex` configuration MUST use the generated catalog's 272,000-token context limit rather than overriding it to 1,050,000.
- Verification MUST establish the effective context metadata and exercise the runtime regression test.

## Chosen Approach

Capture the active model identity and thinking level before teardown, then ask the newly created `AgentSession` to select the equivalent model through its replacement `ModelRegistry`; if resolution fails, retain the replacement runtime's normal fallback selection and surface existing diagnostics rather than making session creation fail.

Remove Cameron's complete `openai-codex.modelOverrides` block because every entry only inflates `contextWindow`; relying on the generated model catalog avoids another stale local copy of endpoint limits.

## Alternatives Declined

- Fix only `pi-plan-mode`: model loss is caused by generic `ctx.newSession()` runtime reconstruction and affects every extension using that API.
- Add a model argument to `ctx.newSession()`: preserving current runtime state is the expected default and should not burden every caller.
- Hard-code GPT model limits in another local file: the generated catalog already reports the endpoint's 272,000-token limit and should remain authoritative.

## Checklist

- [ ] Add a failing runtime regression test for model and thinking continuity.
- [ ] Preserve active model and thinking level across `AgentSessionRuntime.newSession()`.
- [ ] Update public documentation and changelog if the behavior contract needs clarification.
- [ ] Remove the stale local OpenAI Codex context overrides.
- [ ] Run focused tests, type diagnostics, and effective-model validation.
