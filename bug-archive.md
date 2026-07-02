# Bug Archive — marimo-team/marimo-lsp candidates (verified 2026-06-10)

Future contribution candidates, investigated and code-verified locally (checkout at `~/pet/marimo-lsp`, marimo `0.23.9` sibling at `~/pet/marimo`). All were unassigned with **no linked PRs** as of 2026-06-10. Re-verify claims/PRs before starting any of these.

Excluded after triage: #532 (maintainer already fixing), #353 (open PR #464), #585 (lives in marimo frontend, not this repo).

---

## 1. #581 — Error message shows raw cell UUID instead of cell number (easiest)

- **Issue:** https://github.com/marimo-team/marimo-lsp/issues/581
- **Root cause (verified):** `extension/src/lib/errors.ts` — `prettyErrorMessage(error, cellIdMapper?)` only applies `cellIdMapper` in the `multiple-defs` case; the `exception`, `ancestor-stopped`, `ancestor-prevented`, and `strict-exception` cases interpolate the raw kernel UUID.
- **Fix:** in those four cases, resolve `cellIdMapper?.(error.raising_cell / error.blamed_cell) ?? rawId` before formatting. The mapper is already passed at the call site (`extension/src/kernel/ExecutionRegistry.ts:629,717`) and `createCellNavigationLink` (`extension/src/types.ts`) already renders clickable `cell-N` links.
- **Signal:** maintainer mscolnick commented "Yes this is something we can fix with a hyperlink." String-formatting change + unit tests.

## 2. #351 — Auto-reload button shows "off" on startup

- **Issue:** https://github.com/marimo-team/marimo-lsp/issues/351
- **Root cause (verified):** `extension/src/config/ConfigContextManager.ts:50-68` — the `marimo.config.runtime.auto_reload` context key (drives which toolbar button is visible via package.json `when` clauses) is set from a stream that zips config changes with the active notebook; at startup the active notebook is `Option.none()` and the handler defaults it to `"off"`, rendering the wrong button until real config arrives.
- **Fix:** don't write a default on `Option.none()` — `Stream.filter(Option.isSome)` before the `setContext` tap (leave the key unset until known).

## 3. #491 — Toggling on_cell_change auto/lazy crashes before first run

- **Issue:** https://github.com/marimo-team/marimo-lsp/issues/491
- **Root cause (verified):** `src/marimo_lsp/api.py:327-335` — `update_configuration` requires an existing session (sessions are only created by `run`); with no session it returns `{"success": False, "error": "No session found"}`, which the TS `MarimoConfigResponseSchema` (expects `{config}`) fails to decode → "Failed to toggle…". Log line "Could not find cached data for notebook" matches.
- **Fix:** Python side — when no session exists, persist config via the default config-manager path (what `session.config_manager.save_config` wraps) and skip the kernel control request; the kernel picks up config when the session is created. Check with maintainer first: manzt framed it as "delay deserialize until the LSP has started," a different layer.

## 4. #531 — "Open as marimo notebook" deletes an unsaved file

- **Issue:** https://github.com/marimo-team/marimo-lsp/issues/531
- **Root cause (verified):** `extension/src/commands/openAsMarimoNotebook.ts` — runs `vscode.openWith(uri, NOTEBOOK_TYPE)` then closes the text tab with no `isDirty`/`isUntitled` handling; declining VS Code's save prompt discards the only copy of the buffer and the notebook deserializes from the empty/missing on-disk file. Data loss.
- **Fix:** guard the command — if `doc.isDirty || doc.isUntitled`, `doc.save()` first and abort with a clear message if the user cancels. Precedent for untitled-notebook handling exists in `extension/src/statusbar/MarimoStatusBar.ts` (`openUntitledNotebookDocument`).

## 5. #591 — Undefined-variable squiggle desync (NOT easy — parked)

- **Issue:** https://github.com/marimo-team/marimo-lsp/issues/591
- **Finding:** marimo-lsp's own diagnostics (`src/marimo_lsp/_rules.py`) implement only `multiple_definitions` and `cycles` — no undefined-variable rule exists here, so the squiggles likely come from the managed **ty** language server; the fix is about feeding cross-cell context to ty (possibly upstream). The underscore-variable half of the report is intentional marimo semantics (`is_local` filtering in `marimo/_ast/compiler.py:359`).
- **Verdict:** research project, not a quick fix. Revisit only with maintainer guidance.

---

**Plan:** finish #154 (fix + PR) first; then #581 is the recommended contribution #2 (maintainer-blessed, smallest, analysis ready).

---

**Update 2026-07-01 (Cycle 2 selection):** #154 done (PR #603 merged 2026-06-24). **#581 selected as Cycle 2** — re-verified today: still open, unassigned, zero cross-referenced PRs (checked #581/#351/#491/#531 timelines); the four raw-UUID branches in `errors.ts` are unchanged on upstream `main` (`157ab4a`). Caveat: `ExecutionRegistry.ts` was heavily refactored (~400 lines) since the June analysis — re-map the `cellIdMapper` call sites in Phase II. New candidate for Cycle 3: @manzt's invited follow-up from PR #603 (surface disabled state in UI + enable/disable commands) — no issue filed yet.

**Update 2026-07-02 (selection revised): #491 is the Cycle 2 pick.** Re-ranked the bench by maintainer responsiveness before claiming: #491 is the only candidate where the reviewing CODEOWNER (@manzt) personally posted a diagnosis, vs. #581's single month-old contributor comment. Re-verified all five candidates the same day: every one still open, unassigned, zero cross-referenced PRs, zero competing claim comments (full comment threads pulled). #491 root cause re-confirmed on upstream `main` `157ab4a`: `update_configuration` moved to `src/marimo_lsp/api.py:353` but the no-session error branch is intact, and `MarimoConfigurationService.updateConfig` (`extension/src/config/MarimoConfigurationService.ts`) still decodes with `MarimoConfigResponseSchema` (requires `config`), so the error shape fails to decode. New detail: `extension/src/config/schemas.ts` now also defines `MarimoConfigUpdateResponseSchema` (`success`/`config`/`error?`) which models the failure shape but is unused by `updateConfig` — a natural hook for the TS half of the fix. #581 returns to the top of the bench for Cycle 3 consideration alongside the PR #603 follow-up.

**Update 2026-07-02 (later, final): Cycle 2 moved off marimo-lsp entirely — the pick is [py-econometrics/pyfixest #1244](https://github.com/py-econometrics/pyfixest/issues/1244)** (did2s `ValueError` when first-stage fixed effects can't be estimated), selected from the course's official issue list; the full elimination trail is in the journal README's Cycle 2 Phase I section. Everything above stays valid as the marimo-lsp bench: #581, #491 (both fully re-verified available 2026-07-02, root causes confirmed on current upstream), #531, #351, and the invited PR #603 follow-up are all Cycle 3 candidates.
