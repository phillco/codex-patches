# Validation

Base: `818f1cca8ccf8899f0f4d59336baebaccf358eed`.

Steering + Heads Up patched source tree: `3ab9f1d4318942b569ffec405642841fc91e720b`.

- The steering and Heads Up patches apply sequentially to a fresh index at the base; the resulting tree exactly matches the isolated extraction.
- The optional Stop-hook patch applies independently to the same base; its seven Python tests pass.
- Rust formatting and source whitespace checks pass. Scoped `just fix` passed for TUI, core, app-server, app-server-protocol, and config; its only edit was an unrelated upstream unused import, excluded from the patches.
- The expanded-view config schema was regenerated and its property compared against the extracted schema. Experimental app-server exports were regenerated successfully for the capture-control RPC.
- An independent review covered feature isolation, draft preservation, expanded-setting propagation, cancellation/disable races, and public-artifact scope. No blocking findings remain.
- Focused final tests on the extracted series: 68 passed. Two supplemental steering rejection-recovery tests also passed.
- App-server protocol suite: 302 passed, 1 skipped, on the fork. Its protocol source is identical to the extracted series.
- The updated fork release built successfully. The signed installed package passed CLI startup, Code Mode JavaScript execution, and matching voice runtime initialization/close checks; the prior package is retained for rollback.

The initial native extraction passed all 53 focused Heads Up tests. Subsequent build checks caught missing configuration initializers in TUI local settings and the sample app; the delivered patches include those fixes.

Focused validation covers Heads Up, pending-start steering, rejection recovery, and propagation into local TUI preferences:

```sh
just test -p codex-tui -p codex-core -p codex-app-server -p codex-app-server-protocol \
  -E 'test(heads_up) | test(pending_start) | test(before_turn_started) | test(start_rejection) | test(local_load_preserves_defaults)' \
  --test-threads 4
just test -p codex-tui -p codex-core -p codex-app-server -p codex-app-server-protocol \
  -E 'test(unsupported_legacy_permissions)' --test-threads 4
just test -p codex-app-server-protocol
```

Tests use the fork's Studio build runner with Rust 1.95.0 on macOS. The build runner is validation infrastructure and is not included in the patches. Full workspace tests, Windows/Linux builds, and interactive UI acceptance were not performed. Rendered UI behavior has snapshot coverage.

## Background command wakeups

- Patch `0003` applies independently to `BASE`; its resulting tree exactly matches the isolated extraction: `0fca39fa77d38479913812dfd30a4abd89d244a4`.
- All three patches also apply sequentially: resulting tree `3213b91dbf05468250e96d05570deabdb037d0a5`.
- The patch contains nine core implementation/test files, with no Heads Up dependency or new configuration/dependencies.
- Rust formatting and whitespace checks pass. An independent review found no blocking concurrency or cancellation issues.
- Focused standalone tests: 26 sleep-related tests and 3 completion-readiness tests passed (29 total).
- Scoped core lint passed. Its sole automatic edit removed an unrelated upstream unused import; that edit is excluded from this patch.
- The implementation matches the already installed fork build `88549bed1cd6`; extraction does not change the running fork.

```sh
just test -p codex-core -E 'test(sleep) | test(unified_exec::process_manager_tests)' --test-threads 4
# The first filter did not match the process-manager module name; run its new cases explicitly.
just test -p codex-core -E 'test(background_completion_wakes_waiter_once)' --test-threads 4
just fix -p codex-core
```

The completion tests cover pending and active sleeps, one-time delivery, initial exec eligibility, and collected/foreground exclusions. Command-execution coverage is Unix-only; Windows behavior and the full workspace suite were not validated for this patch.
