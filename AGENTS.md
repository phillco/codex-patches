# Rules for this repository

This is a public repository of patches against an upstream snapshot. Everything committed here is published.

## Generated artifacts

- Do not include regenerated binary or compressed schema exports (for example the app-server `*.json.zst` precomputed exports) in patches. Consumers regenerate them after applying: `just write-app-server-schema` and `just write-app-server-schema --experimental`. Say so in the patch's README row.
- Hand-readable generated files (JSON schemas, snapshots, TypeScript) that the change alters stay in the patch.

## Adding a patch

- Target the commit in `BASE`; verify with `git apply --check` against a fresh index at `BASE`, then in series order.
- Add the patch to `series`, `README.md` (behavior, requirements, dependencies), `SHA256SUMS` and `VALIDATION.md`. State plainly what was and was not tested.
- Patches are independent unless the README says otherwise.
