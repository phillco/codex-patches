# Validation

Base: `818f1cca8ccf8899f0f4d59336baebaccf358eed`.

Final patched source tree: `7b173b99ef09837335d2eda6bd708012d3619e85`.

- Both patches apply sequentially to a fresh index at the base; the resulting tree exactly matches the isolated extraction.
- The optional Stop-hook patch applies independently to the same base; its seven Python tests pass.
- Rust formatting and source whitespace checks pass.
- The expanded-view config schema was regenerated and its property compared against the extracted schema. Experimental app-server exports were regenerated successfully for the capture-control RPC.
- An independent review covered feature isolation, draft preservation, expanded-setting propagation, cancellation/disable races, and public-artifact scope. No blocking findings remain.
- Focused final tests: pending.
- App-server protocol suite: pending.

The initial native extraction passed all 53 focused Heads Up tests. Subsequent build checks caught missing configuration initializers in TUI local settings and the sample app; the delivered patches include those fixes.

Focused validation covers Heads Up, pending-start steering, rejection recovery, and propagation into local TUI preferences:

```sh
just test -p codex-tui -p codex-core -p codex-app-server -p codex-app-server-protocol \
  -E 'test(heads_up) | test(pending_start) | test(before_turn_started) | test(start_rejection) | test(local_load_preserves_defaults)' \
  --test-threads 4
just test -p codex-app-server-protocol
```

Tests use the fork's Studio build runner with Rust 1.95.0 on macOS. The build runner is validation infrastructure and is not included in the patches. Full workspace tests, Windows/Linux builds, and interactive UI acceptance were not performed. Rendered UI behavior has snapshot coverage.
