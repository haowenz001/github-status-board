# Source repository template

Copy the **contents** of this directory into the root of your code repository:

```text
.github/workflows/ci.yml
.github/ci-gates.json
scripts/gate/gate.mjs
test/cases/case-catalog.yaml
test/example.test.mjs
package.json
```

Then:

1. Replace the `build`, `lint` and `test` scripts in `package.json` with your real commands. If you use Python, Go or another stack, edit `.github/ci-gates.json` and remove the Node setup/install steps from `.github/workflows/ci.yml`.
2. Fill `test/cases/case-catalog.yaml`. A `ready` case needs `file`, `owner` and a `// @case <id>` marker in the file. Mark unfinished cases `todo`; use `skipped` with `until` and `reason` only when necessary.
3. Open a PR. The `CI` workflow runs `build` and `pr-ready`; on merge queue it runs `pr-merge`; on main push it runs `build`; on schedule it runs `daily` (observe only); on release it runs `release`.
4. Enable branch protection and require the check **`CI / gate`**. Use `CI / gate` in the merge queue too.

The workflow uploads:

- `ci-gates-reports`: the raw lane reports (`build.json`, `pr-ready.json`, ...).
- `ut-results`: optional JUnit/Playwright reports matching the globs in `ci.yml`. The status board reads this artifact so test counts appear automatically.

The status board's default gate pattern includes `^CI$` and `CI Gates`, so keep the workflow named `CI`. If you rename it, update `board-config.json` → `workflows.gate`.

This template is deliberately minimal. It does not install a test framework, linter or build tool; those remain your project's choices.
