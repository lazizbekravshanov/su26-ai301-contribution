# PR: Skip cells marked disabled when building the execute-cells request (#154)

> Archived description for marimo-team/marimo-lsp **PR #603**, issue #154.
> Repo has no PR template; this follows the structure of recent merged PRs.

Closes #154.

## Why

Cells authored as `@app.cell(disabled=True)` are executed by the VS Code extension, even
though marimo's own frontend respects the flag. @manzt confirmed in #154 that disabled cells
"aren't currently supported" in the extension and welcomed a contribution.

**Root cause:** `extension/src/lib/extractExecuteCodeRequest.ts` assembles the `execute-cells`
request for **both** execution paths (`NotebookControllerFactory` and `SandboxController`), but
its loop only skips cells without a stable id — it never consults the disabled config. The flag
*does* survive the pipeline (marimo file → LSP deserialize → `NotebookSerializer` → cell
`metadata.options.disabled`); I confirmed the wire format by running the server's own
deserialize path, where a disabled cell arrives as `options: { "disabled": true }`.

## What

- Add an `isDisabled` getter on `MarimoNotebookCell` (`metadata.options?.disabled === true`),
  mirroring the existing `isStale` getter. No schema change — `CellMetadata` already types
  `options` as `Record<string, unknown>`.
- Skip disabled cells in `extractExecuteCodeRequest`, fixing both call sites at the single
  choke point (rather than per-call-site, which would let a future call site silently
  reintroduce the bug).
- Fix a latent bug the filter exposes: the `if (codes.length === 0)` branch called
  `Option.none()` without `return`ing, so a selection of only-disabled cells would have sent an
  empty `execute-cells` request. It now returns `Option.none()`, which the controller already
  handles by stopping ("Empty execution request").

## Evidence

`extension/src/lib/__tests__/extractExecuteCodeRequest.test.ts` (4 tests):

- `excludes cells disabled via @app.cell(disabled=True) (issue #154)` — **fails before**
  (`expected [...'cell-disabled'] to not include 'cell-disabled'`), **passes after**.
- `returns none when every selected cell is disabled` — covers the empty-request branch.
- 2 sanity cases (enabled cells included; id-less cells skipped).

Full `just test-ts` green (466 passed / 1 skipped, +4 from this PR), `just lint` (ruff + `ty` +
TS) clean, `just test-vscode` integration suite green.

## Open question

Enforce at the **extension layer** (where the request is assembled, as here) or the **LSP
server layer**? I chose the extension layer since that's where the request is built and this
was framed as a missing extension feature — happy to move it if you'd prefer the server side.
