# Contribution README

**Name:** Lazizbek Ravshanov
**Program:** CodePath AI301, Summer 2026
**Status:** Phase II Complete

---

## Project

**Project:** [marimo-team/marimo-lsp](https://github.com/marimo-team/marimo-lsp)
**My fork:** https://github.com/lazizbekravshanov/marimo-lsp

marimo-lsp is the language server and official VS Code extension for marimo, an open-source reactive Python notebook. It bridges three runtimes — the VS Code notebook UI, a Python LSP server, and the marimo kernel — so marimo notebooks run natively inside VS Code.

---

## Phase I: Issue Selection

### Issue

[marimo-team/marimo-lsp #154: Extension runs cells marked as disabled](https://github.com/marimo-team/marimo-lsp/issues/154)

Labels: `bug`, `good first issue`

### Why I Chose This Issue

Cells marked `@app.cell(disabled=True)` execute anyway when run from the VS Code extension, while marimo's own frontend respects the flag. The core maintainer confirmed disabled cells "aren't currently supported" in the extension, called the fix achievable, added it to the roadmap, and explicitly wrote "we'd welcome a contribution." I verified before claiming it: no assignee, no linked pull requests, and the repository is exceptionally responsive — PRs were merged the same day I checked, and maintainers replied to recent issues within minutes to a few hours.

### Issue Selection Checklist Notes

I ran this issue through the Phase I selection checklist. My notes:

1. **I understand the problem.** In one sentence: running a cell marked `@app.cell(disabled=True)` from the VS Code extension executes it, when it should be skipped. "Fixed" looks like: the extension respects the disabled cell config the same way marimo's own frontend does, with a regression test (`just test`) covering it.
2. **Scope fits 3 to 4 weeks.** The change is contained to the cell-run path between the extension's `KernelManager` and the LSP server's command handler, which `ARCHITECTURE.md` documents in detail. No marimo core changes are required.
3. **Matches my skills.** Python (the LSP server) and TypeScript (the extension), with an unusually thorough `ARCHITECTURE.md` mapping the exact message flow I need to trace.
4. **Active and claimable.** No assignee, no linked PR, no competing claim comments. I measured the repository before choosing: PRs merged daily and maintainers replied to the last ten issues within minutes to a few hours.
5. **Helpful context.** The core maintainer triaged the issue and put it on the roadmap; `ARCHITECTURE.md` documents the three LSP channels between extension, server, and kernel; marimo's own docs specify the intended disabled-cell behavior to match.
6. **Clear setup documentation.** `CONTRIBUTING.md` gives a quick start (open in VS Code, press F5), the exact tooling (`uv`, `pnpm`, `just`), a pinned `.marimo-version` for the side-by-side marimo checkout, and `just lint` / `just test` commands. marimo also runs an active Discord community for contributor questions.

All six checks passed.

### Risks I Noted

The repository moves fast (multiple merges per day), so the code around my fix may shift while I work; I will rebase frequently and open a draft PR early. The maintainers may also prefer the fix at the extension (TypeScript) layer or the LSP server (Python) layer, so I asked in my claim comment to surface that preference before I write code.

---

## Phase II: Reproduce and Plan

### Understanding the Issue

**Problem.** A marimo cell authored as `@app.cell(disabled=True)` must not execute, but the VS Code extension runs it normally. marimo's own frontend respects the flag; the extension ignores it.

**Root cause (confirmed by reading and running the code).** Every execution request in the extension is assembled by `extension/src/lib/extractExecuteCodeRequest.ts`. Its loop skips only cells without a stable id, then pushes every remaining cell's code and id into the `execute-cells` request — it never checks the disabled config. This one function feeds **both** execution paths: `extension/src/kernel/NotebookControllerFactory.ts` (line 74) and `extension/src/kernel/SandboxController.ts` (line 55).

**Where `disabled` lives (verified by execution, not guesswork).** I ran the LSP server's own deserialize path (`MarimoConvert.from_py(...).to_ir()` → `dataclasses.asdict`) on a notebook containing a disabled cell. The cell arrives as `options: {"disabled": true}`, and `NotebookSerializer.ts` copies `cell.options` verbatim into the VS Code cell's `metadata.options`. So at the point of the bug the flag is readable as `metadata.options.disabled === true` — `CellMetadata.ts` already models `options` as `Record<string, unknown>`, so no schema change is required.

**Expected behavior:** disabled cells are excluded from the execution request. **Current behavior:** they are included and run.

### Reproduction Process

#### Environment Setup

OS: macOS (Darwin 25.5.0, Apple Silicon). Tools: node v25.6.1, git 2.50.1, uv 0.11.20, pnpm 11.5.3, just 1.52.0 (the last three installed via `brew install uv pnpm just`).

```sh
cd ~/pet
git clone https://github.com/lazizbekravshanov/marimo-lsp.git
cd marimo-lsp
git remote add upstream https://github.com/marimo-team/marimo-lsp.git
git fetch upstream && git checkout main && git merge --ff-only upstream/main
uv sync                      # Python deps (~5s)
pnpm -C extension install    # TS deps (~10s)
just test-ts                 # FAILED at first — see below
```

Two real issues hit and fixed (total setup ~15 minutes):

1. First `just test-ts` run failed: 8 of 40 test files errored with `Cannot find package '@marimo-team/smart-cells'`. Cause: `extension/package.json` links `@marimo-team/smart-cells`, `frontend`, and `openapi` from a side-by-side `marimo` checkout (`link:../../marimo/...`), so the unit tests **do** require the sibling clone described in CONTRIBUTING.md. Fix: `git clone --branch 0.23.9 --depth 1 https://github.com/marimo-team/marimo.git` next to `marimo-lsp` (0.23.9 = the pin in `.marimo-version`), then re-run `pnpm -C extension install`.
2. Second run still failed: `Tsconfig not found` for smart-cells (its tsconfig extends the workspace package `@marimo-team/tsconfig`) and `Cannot find package 'html-react-parser'` from `marimo/frontend`. Cause: the marimo workspace's own dependencies were not installed. Fix: `pnpm install` at the marimo repo root (~2m17s).

Baseline after fixes: `just test-ts` → **Test Files 40 passed (40); Tests 429 passed | 1 skipped (430)**.

#### Steps to Reproduce

**A. Unit-level reproduction (deterministic, the core evidence).** I wrote `extension/src/lib/__tests__/extractExecuteCodeRequest.test.ts` (mirroring the style of `getCellExecutableCode.test.ts`): it builds two mock cells — one normal, one with `metadata.options = { disabled: true }` — and asserts the returned `cellIds` exclude the disabled cell.

1. `cd ~/pet/marimo-lsp && git checkout fix-issue-154-disabled-cells`
2. `just test-ts src/lib/__tests__/extractExecuteCodeRequest.test.ts`
3. Expected: 3/3 tests pass. Actual: the issue-154 case fails — run twice, identical failure both times:

```
AssertionError: expected [ 'cell-enabled', 'cell-disabled' ] to not include 'cell-disabled'
Tests  1 failed | 2 passed (3)
```

The two passing sanity cases (enabled cells included; id-less cells skipped) isolate the failure to exactly the missing disabled check.

**B. Manual end-to-end reproduction (user-facing).**

1. `cd ~/pet/marimo-lsp && code .`
2. Press **F5** ("Run Extension") to launch the Extension Development Host.
3. In the dev host, create a marimo notebook `repro154_mo.py` with two cells: `x = 1`, and a cell whose decorator is `@app.cell(disabled=True)` containing `print("RAN — bug if you see this")`.
4. Open it as a marimo notebook, pick a Python environment, and **Run All**.
5. Expected: the disabled cell is skipped (no output). Actual: it prints `RAN — bug if you see this`.

*Observed result:* **[placeholder — paste output/screenshot after running the F5 repro]**

#### Reproduction Evidence

- Working branch with the failing regression test: https://github.com/lazizbekravshanov/marimo-lsp/tree/fix-issue-154-disabled-cells (commit `21986be`)
- The committed test fails on current `main` code by design; it becomes the regression guard once the Phase III fix lands.

### Solution Approach

#### Analysis

The disabled config survives the whole pipeline (marimo file → LSP deserialize → `NotebookSerializer` → cell `metadata.options`) and is simply never consulted at the single choke point where execution requests are built. Fixing that one function fixes both execution paths at once.

#### Proposed Solution

Skip disabled cells inside `extractExecuteCodeRequest`'s loop, exposed through a typed getter on `MarimoNotebookCell` so the read is centralized and matches the existing `isStale` pattern.

#### Implementation Plan (UMPIRE)

**Understand:** disabled cells must not be sent to `execute-cells`; today `extractExecuteCodeRequest` includes them because its loop only checks for a missing stable id.

**Match:** `MarimoNotebookCell` already derives flags from metadata the same way (`isStale` reads `metadata.state`; `id` reads `metadata.stableId`), and the loop already uses the skip idiom (`if (Option.isNone(cell.id)) continue;`). The regression test mirrors the established vitest pattern in `extension/src/lib/__tests__/`.

**Plan:**
1. `extension/src/schemas/MarimoNotebookDocument.ts` — add an `isDisabled` getter on `MarimoNotebookCell` reading `metadata.options?.disabled === true` (defaults to false). No schema change needed.
2. `extension/src/lib/extractExecuteCodeRequest.ts` — add `if (cell.isDisabled) continue;` to the loop. This fixes both call sites (`NotebookControllerFactory.ts:74`, `SandboxController.ts:55`) in one place. Alternative — filtering at each call site — rejected: it duplicates logic and any future call site would silently re-introduce the bug.
3. Include the one-line fix for a latent bug that becomes load-bearing after filtering: the `if (codes.length === 0)` branch calls `Option.none()` without `return`, falling through to `Option.some({codes: [], cellIds: []})`. Once disabled cells are filtered, a selection of only disabled cells reaches this branch, so it must actually return `Option.none()` (the controller already handles that case by logging "Empty execution request" and stopping). This will be called out explicitly in the PR description.
4. Files touched: `extractExecuteCodeRequest.ts`, `MarimoNotebookDocument.ts`, plus the already-committed test file.
5. Open question posted on #154 (no maintainer reply yet): enforce at the extension layer or the LSP server layer. Recommendation: extension layer — it is where the request is assembled, and the maintainer framed this as a missing extension feature. If the maintainer prefers the server layer, the regression test still stands and the plan adapts.

**Implement:** branch https://github.com/lazizbekravshanov/marimo-lsp/tree/fix-issue-154-disabled-cells (currently test-only; the fix lands in Phase III).

**Review:** run `just fix` and `just lint` before committing; install hooks with `uvx pre-commit install`; clear commit message; PR description links #154; per the repo's stated philosophy, no lint/type opt-outs.

**Evaluate:** the regression test flips to green with the fix; full `just test-ts` stays green (baseline: 40 files, 429 passed / 1 skipped); the manual F5 repro shows the disabled cell skipped. Integration suite (`just test-vscode`): run once before opening the PR as a sanity check, but no new integration test — the bug and fix are fully exercised at the unit seam where the request is constructed.

---

## Phase III: Build

*To be completed.*

### Testing Strategy

### Implementation Notes

---

## Phase IV: Submit and Iterate

*To be completed.*

### Pull Request

### Summary

### Maintainer Feedback Log

---

## AI Tool Usage Log

I am responsible for every line in my pull request. This table logs how I used AI tools, per course policy.

| Date | Tool | What I used it for | What I verified myself |
|---|---|---|---|
| 2026-06-10 | Claude | Scoring candidate repositories from the course issue list for maintainer responsiveness (time-to-first-reply, merge cadence) and screening issues for existing claims, assignees, and linked PRs | I read issue #154 and the maintainer's comment, `CONTRIBUTING.md`, and `ARCHITECTURE.md` directly to confirm scope, setup steps, and that no assignee or pull request exists |
| 2026-06-10 | Claude Code | Phase II: setting up the dev environment (debugging the side-by-side marimo checkout requirement), tracing the execution code path, confirming the `options.disabled` wire format by running the LSP deserialize path, writing the failing regression test, and drafting the UMPIRE plan | I reviewed every quoted file (`extractExecuteCodeRequest.ts`, both call sites, `CellMetadata.ts`, `MarimoNotebookDocument.ts`, `NotebookSerializer.ts`, `api.py`) and ran the baseline suite and the regression test myself (twice) to confirm the green baseline and the consistent failure |
