# Validation

Base: `818f1cca8ccf8899f0f4d59336baebaccf358eed`.

Final patched source tree: `3ab9f1d4318942b569ffec405642841fc91e720b`.

- Both patches apply sequentially to a fresh index at the base; the resulting tree exactly matches the isolated extraction.
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
