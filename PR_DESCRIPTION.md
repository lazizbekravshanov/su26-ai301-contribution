# PR: Skip cells marked disabled when building the execute-cells request (#154)

> Draft PR description for marimo-team/marimo-lsp, issue #154.
> Repo has no PR template; this follows the structure of recent merged PRs.

Closes #154.

## Problem

Cells authored as `@app.cell(disabled=True)` are executed by the VS Code
extension, even though marimo's own frontend respects the flag. (@mscolnick
confirmed in #154 that disabled cells "aren't currently supported" in the
extension yet and welcomed a contribution.)

## Root cause

`extension/src/lib/extractExecuteCodeRequest.ts` assembles the `execute-cells`
request for **both** execution paths — `NotebookControllerFactory.ts` and
`SandboxController.ts` — but its loop only skips cells without a stable id; it
never consults the disabled config.

The flag does survive the pipeline: marimo file → LSP deserialize →
`NotebookSerializer` → cell `metadata.options.disabled`. I confirmed the wire
format by running the server's own deserialize path
(`MarimoConvert.from_py(...).to_ir()` → `dataclasses.asdict`) on a notebook with
a disabled cell — it arrives as `options: { "disabled": true }`.

## Fix

- Add an `isDisabled` getter on `MarimoNotebookCell`
  (`metadata.options?.disabled === true`, defaults to `false`), mirroring the
  existing `isStale` getter. No schema change — `CellMetadata` already models
  `options` as `Record<string, unknown>`.
- Skip disabled cells in `extractExecuteCodeRequest`, fixing both call sites at
  the single choke point where the request is built.
- Fix a latent bug that becomes load-bearing once disabled cells are filtered:
  the `if (codes.length === 0)` branch called `Option.none()` **without
  returning**, so it fell through to `Option.some({ codes: [], cellIds: [] })`.
  A selection of only-disabled cells now reaches that branch, so it must
  actually `return Option.none()` (the controller already handles that by
  logging "Empty execution request" and stopping).

## Tests

`extension/src/lib/__tests__/extractExecuteCodeRequest.test.ts`:

- **Regression:** a `{ disabled: true }` cell is excluded from `codes`/`cellIds`
  (fails before the fix, passes after).
- **Edge case:** a selection of only disabled cells returns `Option.none()`.
- Plus the two pre-existing sanity cases (enabled cells included; id-less cells
  skipped).

Full `just test-ts` stays green (44 files, 462 passed / 1 skipped); `just lint`
(ruff + `ty` + TS check) clean.

## Open question

Should this be enforced at the **extension layer** (where the request is
assembled, as done here) or at the **LSP server layer**? I went with the
extension layer since that's where the request is built and the issue was framed
as a missing extension feature — happy to move it if you'd prefer the server
side.
