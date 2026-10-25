# Codex patches

Focused functionality patches for [Codex CLI](https://github.com/openai/codex), extracted from a working fork. Patches target the exact commit in [`BASE`](BASE); they are not rebased onto current upstream.

| Patch | Behavior | Requires |
| --- | --- | --- |
| [Pending-turn steering](patches/0001-pending-turn-steering.patch) | Enter steers a submitted turn even before `TurnStarted` arrives; Tab queues. Preserves input when starting or steering fails. | Base |
| [Heads Up](patches/0002-heads-up.patch) | Optional independent conversation reviewer, persistent notes, full explanations, known topics, editable discussion drafts, and direct send. Includes `tui.heads_up_expanded`. | Steering patch |
| [Background command wakeups](patches/0003-background-command-wakeups.patch) | Ends `clock.sleep` early when a background command finishes, returning its session ID and status. | Base; independent of the other patches |
| [Review ranges](patches/0004-review-ranges.patch) | Adds Git commit ranges and JJ revsets to `codex review`, with optional focus instructions. | Base; independent of the other patches |

The native Heads Up patch includes core and app-server integration, TUI rendering, generated API/schema artifacts, tests, and snapshots. It retains the parent's prompt prefix and disables tool execution for the review. Reviews run separately from the main turn, with bounded retries and output; results enter the conversation only through an explicit user action.

## Base version

These patches target an **upstream development snapshot**, not a tagged release:

- **Exact base:** [`818f1cca8ccf8899f0f4d59336baebaccf358eed`](https://github.com/openai/codex/commit/818f1cca8ccf8899f0f4d59336baebaccf358eed), dated September 10, 2026 — “Remove `repo_url` from skill invocation analytics events (#44586).”
- **Nearest ancestor CLI release tag:** `rust-v0.143.0-alpha.10`; the base is 2,816 commits beyond that tag according to `git describe`. The upstream workspace itself declares version `0.0.0`.
- **Source fork build version:** `0.162.0-alpha.13`. This is the fork’s package version, **not** the upstream patch target.

Use the exact commit recorded in [`BASE`](BASE) when applying the series; applicability to other versions has not been verified.

## Apply

From a clean Codex checkout at `818f1cca8ccf8899f0f4d59336baebaccf358eed`, with this repository cloned alongside it:

```sh
git switch --detach 818f1cca8ccf8899f0f4d59336baebaccf358eed
git apply --check ../codex-patches/patches/0001-pending-turn-steering.patch
git apply ../codex-patches/patches/0001-pending-turn-steering.patch
git apply --check ../codex-patches/patches/0002-heads-up.patch
git apply ../codex-patches/patches/0002-heads-up.patch
git apply --check ../codex-patches/patches/0003-background-command-wakeups.patch
git apply ../codex-patches/patches/0003-background-command-wakeups.patch
git apply --check ../codex-patches/patches/0004-review-ranges.patch
git apply ../codex-patches/patches/0004-review-ranges.patch
```

[`series`](series) records this order. Steering, background command wakeups, and review ranges can each be used independently. Heads Up depends on steering so “Send to agent” also steers during the pending-start interval.

Build Codex using its normal build instructions after applying the patches.

## Heads Up settings

Toggle reviews with `/heads-up`, or enable them in `config.toml`:

```toml
[features]
heads_up = true

[tui]
heads_up_expanded = true
```

`heads_up_expanded` defaults to `false`. Setting it to `true` opens each new finding directly in the full explanation view. Restart/resume the TUI after changing this startup setting.

Turning reviews off stops prompt capture, releases the retained review snapshot, and cancels an in-flight review. Switching threads releases capture for the previous thread.

Notes survive new prompts. “Discuss with agent” appends quoted, editable text to the composer; “Send to agent” sends the full note while preserving the draft. “Knew this” remembers the topic across sessions; dismissal hides the current note. Known/seen topics are stored under Codex home.

## Background command wakeups

With this patch, completion wakeups are **automatic whenever the agent calls `clock.sleep`**. There is no opt-in parameter or configuration switch. The agent chooses whether to sleep; background commands do not start a new turn or resume an idle agent on their own.

A pending or newly completed background `exec_command` can end the sleep early. The result includes the command’s session ID and exit code, or a failure/timeout status. The agent retrieves output with `write_stdin`. Each completion wakes at most one sleep; collecting the final result or removing the process discards its pending notification. Commands that finish inline do not leave a wakeup behind. Normal timer expiry and interruption by new user input remain supported.

## Review ranges

```sh
codex review --range main..HEAD "Focus on concurrency and rollback"
codex review -R HEAD~3..HEAD
codex review --rev 'main..@-' "Check compatibility"
cat checklist.txt | codex review -r 'main..@-' -
```

`--range` / `-R` reviews a nonempty Git `BASE..HEAD` range (base exclusive). The base must be an ancestor of the head; open-ended and three-dot ranges are rejected. `--rev` / `-r` accepts one contiguous, linear run of JJ commits in a Git-colocated repository. Merge commits and disconnected or gapped JJ selections are rejected. Resolution uses normal JJ snapshot semantics, so `--rev @` includes current working-copy edits. The resolved commit IDs stay fixed for the review.

Both selectors accept optional focus instructions, including stdin via `-`, with no added focus-length limit. The generated instructions contain immutable endpoints and a commit count rather than copying all commit subjects. Existing review targets remain available, with new `-u` and `-b` aliases for `--uncommitted` and `--base`. Target selectors cannot be combined. `--title` is unavailable with `--range` or `--rev`; it remains available for single-commit reviews.

Shell completions are generated from the CLI through `codex completion fish`. In JJ repositories, reviewer shell commands receive `JJ_GIT_OK=1` for compatibility with Git-blocking agent wrappers.

## Optional legacy Stop-hook prototype

[`optional/heads-up-stop-hook.patch`](optional/heads-up-stop-hook.patch) preserves the earlier Stop-hook prototype and its local marketplace registration. **It is an alternative to native Heads Up, not an additional dependency; skip it when applying the native Heads Up pair.** It applies independently to the base and is not part of `series`. It reviews only the final response in an isolated `codex exec` process, without the native explanation/actions UI. Use it as an alternative; enabling the native reviewer with `/heads-up` disables the matching local plugin to avoid duplicate reviews. If enabling through config instead, disable the plugin separately.

## Validation

See [`VALIDATION.md`](VALIDATION.md) for the exact checks and limitations of this extraction.
