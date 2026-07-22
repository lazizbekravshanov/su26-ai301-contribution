# Reflections — What I Actually Worked On

A plain-language walkthrough of each open-source contribution: what the problem was,
which part of the software it lives in, what I actually changed, and how it ended up.
This is the "explain it to a friend" version — the formal phase-by-phase write-ups are
in [README.md](README.md).

At a glance: **6 pull requests merged and 2 in review**, across four projects and three
languages (TypeScript, Python, Rust).

---

## Cycle 1 — marimo-lsp: stop running "disabled" notebook cells

**Project.** [marimo-team/marimo-lsp](https://github.com/marimo-team/marimo-lsp) — the
language server and official VS Code extension for [marimo](https://marimo.io), a reactive
Python notebook. It's the glue between three moving parts: the VS Code notebook UI, a Python
language server, and the marimo kernel that actually runs your code.

**The problem ([#154](https://github.com/marimo-team/marimo-lsp/issues/154)).** marimo lets
you mark a cell as disabled — `@app.cell(disabled=True)` — meaning "don't run this." But when
you hit "run all" inside VS Code, the extension ran the disabled cells anyway.

**What part of the software.** The **execute-cells request builder** in the VS Code extension —
the small piece of TypeScript that turns your "run these cells" click into a request sent to
the marimo kernel. The kernel and language server were already correct; the bug was that the
extension never filtered out disabled cells before asking the kernel to run them.

**What I actually did.**
- Added an `isDisabled` check on the cell object (it reads `metadata.options?.disabled === true`
  from the notebook document) and a single `if (cell.isDisabled) continue;` so disabled cells
  are skipped when building the run request.
- While in there, I found and fixed a second, unrelated latent bug: a missing `return` that
  let an empty-request code path fall through when it shouldn't.
- Before writing any of it, I proved the exact data shape by *running* the deserialize path
  rather than guessing — confirming that `disabled=True` really arrives as `options: {"disabled": true}`.
- Files: `extension/src/lib/extractExecuteCodeRequest.ts` and `extension/src/schemas/MarimoNotebookDocument.ts`.

**The interesting bit.** I traced the marimo kernel end-to-end to confirm it already skips
disabled cells during *automatic* (reactive) re-runs — so only the *explicit* "run" path needed
the guard. That's what told me the fix belonged in the extension layer, not the server.

**Outcome.** [PR #603](https://github.com/marimo-team/marimo-lsp/pull/603) — approved and merged
on the first review pass by the maintainer (@manzt), closing #154.

---

## Cycle 2 — pyfixest: turn a cryptic crash into a clear error

**Project.** [py-econometrics/pyfixest](https://github.com/py-econometrics/pyfixest) — a fast
Python library for fixed-effects regressions, widely used by economists. Mixed Python/Rust
(a compiled core for speed).

**The problem ([#1244](https://github.com/py-econometrics/pyfixest/issues/1244)).** The `did2s`
estimator (a two-stage difference-in-differences method) sometimes crashed with an opaque
`ValueError` about mismatched array shapes — the kind of error that tells you *something*
broke but not *what*.

**What part of the software.** The two-stage DiD estimator itself, `pyfixest/did/did2s.py`.

**What I actually did.**
- Root-caused it: if a unit is treated the whole time, the first regression stage only sees
  part of the data, so one of the "fixed effect" categories never gets an estimate. That
  produces `NaN` predictions, which makes two internal arrays different lengths, which finally
  blows up as an unhelpful broadcast error several steps later. The *real* problem was the
  unestimable category — not the array math where it happened to crash.
- Built a natural reproduction (a synthetic panel with an always-treated unit) so the failure
  was honest, not contrived.
- Added a guard right after the first-stage prediction that detects the unestimable categories
  and raises a clear error **naming which ones**, so the user knows exactly what's wrong.
- Wrote the regression test first (red → green) in `tests/test_errors.py`.

**Outcome.** [PR #1368](https://github.com/py-econometrics/pyfixest/pull/1368) merged. The
maintainer (@leostimpfle) liked the fix and took it the last mile himself — moving the warning
into a shared prediction helper so it protects more code paths, not just `did2s`. I also spotted
and fixed two unrelated type-checker errors that were blocking the project's CI, shipped as a
separate small [PR #1369](https://github.com/py-econometrics/pyfixest/pull/1369) that merged the
same day.

---

## Cycle 3 — pyfixest: make invisible fast-code visible to coverage

**Project.** Same as Cycle 2 (pyfixest), but this one is about **testing infrastructure**, not
a bug in the library's behavior.

**The problem ([#829](https://github.com/py-econometrics/pyfixest/issues/829)).** pyfixest speeds
up hot loops with `numba`, which compiles Python functions to machine code. The catch: once
compiled, those functions no longer run as normal Python, so the code-coverage tool can't "see"
them — they showed up as untested even though tests exercised them.

**What part of the software.** Continuous-integration configuration — a new test task, the
weekly CI workflow (`.github/workflows/extended_tests.yaml`), and the coverage config
(`codecov.yml`). No library behavior changed.

**What I actually did.**
- Added a test run with `NUMBA_DISABLE_JIT=1`, which tells numba to *skip* compilation and run
  the functions as ordinary Python — so the coverage tool can trace them. Verified it worked:
  one numba module went from 17% to 100% coverage.
- Since this run is slower, wired it into the weekly "extended" CI rather than every push.
- Widened it to cover two more numba modules the first pass missed (`find_collinear_variables_nb`,
  `nested_fixef_nb`).
- Corrected the issue's scope first: it was 16 months old, and part of it (the "JAX" half) was
  obsolete, plus one function had since been rewritten in Rust — so I left those out instead of
  writing dead code.

**Outcome.** [PR #1385](https://github.com/py-econometrics/pyfixest/pull/1385) merged by
@s3alfisc, closing #829. He finished the CI wiring himself, and a point I raised while reviewing
his commit (that regular pull-request builds might now under-report numba coverage) became his
final commit before merging. A genuinely collaborative close.

---

## Cycle 4 — open-chat-studio: add a new AI provider (chat + voice)

**Project.** [dimagi/open-chat-studio](https://github.com/dimagi/open-chat-studio) — a Django
web platform for building and running chatbots, with pluggable AI providers.

**The problem ([#2979](https://github.com/dimagi/open-chat-studio/issues/2979)).** A feature
request: let teams use **MiniMax** (an AI company offering both a chat model and a
text-to-speech service) inside Open Chat Studio. At the time there was no way to configure it.

**What part of the software.** The **provider system** — the code that lets the platform talk to
different AI vendors: `service_providers/`, the LLM service layer, `speech_service.py`, the model
seed data, and database migrations. The maintainer asked me to split it into two pull requests.

**What I actually did.**
- **Chat ([PR #3800](https://github.com/dimagi/open-chat-studio/pull/3800)).** MiniMax's chat API
  is OpenAI-compatible, so I wired it through the existing OpenAI-style path (the same one Groq
  and Perplexity use), pointed it at MiniMax's endpoint, and seeded the current MiniMax models
  with a data migration so existing installs pick them up.
- **Voice ([PR #3801](https://github.com/dimagi/open-chat-studio/pull/3801)).** The text-to-speech
  API is *not* OpenAI-shaped — it's a custom endpoint that needs a `GroupId` query parameter plus
  a bearer token, and it returns the audio hex-encoded. I wrote a `MinimaxSpeechService` that
  handles that contract (decoding the audio, surfacing API errors cleanly) and seeded MiniMax's
  built-in system voices.
- Before writing anything, I confirmed both real API shapes so the code matched reality, not
  assumptions.

**The interesting bit.** Because it was two pull requests touching the same area, both added a
database migration with the same number — which git merges without complaint but Django refuses.
I flagged the collision up front, so when the first PR merged, fixing the second was a one-line
renumber instead of a scramble.

**Outcome.** Both merged by @SmittieC (chat 2026-07-16, voice 2026-07-17); issue #2979 closed
automatically. MiniMax now works in Open Chat Studio as both a chat and a voice provider.

---

## Cycle 5 — git-cliff: custom templates and cleaner config discovery (Rust)

**Project.** [orhun/git-cliff](https://github.com/orhun/git-cliff) — a popular command-line tool
(written in Rust) that generates changelogs from git history.

### Issue [#412](https://github.com/orhun/git-cliff/issues/412) — user-defined templates

**The problem.** `git cliff --init` sets up a starter config from a handful of *built-in*
templates. Users wanted to supply their **own** template directory too.

**What part of the software.** The template-embedding layer (`git-cliff-core/src/embed.rs`) and
the CLI argument handling (`git-cliff/src/args.rs`, `lib.rs`, `main.rs`), plus docs.

**What I actually did.** Added a `--templates-dir <PATH>` flag (and a `GIT_CLIFF_TEMPLATES_DIR`
environment variable), a `--list-templates` flag to see what's available, and made a user's own
templates override the built-ins when names collide. Built test-first (10 tests).
[PR #1583](https://github.com/orhun/git-cliff/pull/1583) — complete and green, in review.

### Issue [#1182](https://github.com/orhun/git-cliff/issues/1182) — config discovery

**The problem.** A request to have git-cliff also look for its config in a `.config/` folder.

**What I actually did.** First, I checked current `main` and found this was **already
implemented** upstream — so instead of duplicating it, I posted a note flagging it for closure.
The maintainer (@orhun) then asked for a related cleanup: make the `--config` option a proper
"optional" value (instead of a hardcoded default) and route through automatic discovery when it's
omitted. I did that — when you don't pass `--config`, git-cliff now searches the project folder
(and up the tree) and then your user config directory; passing `--config` still overrides
everything. Updated the docs too. [PR #1584](https://github.com/orhun/git-cliff/pull/1584) — in
review.

**Where these stand.** Both git-cliff PRs are finished on my side, pass their tests, and are
waiting on the maintainer's review.

---

## What I take away from all of this

- **The real work is usually the diagnosis, not the code.** The hardest parts were figuring out
  *why* something broke (an unestimable category, a JIT'd function coverage can't see, a
  migration clash git can't detect) — the actual change was often small once the cause was clear.
- **Fixes belong at the layer that owns the problem.** Cycle 1's guard went in the extension, not
  the kernel, because the kernel was already right. Knowing where *not* to change things mattered
  as much as the change itself.
- **Good open source is collaborative.** On two of the pyfixest contributions the maintainer took
  my work the last mile, and my input still improved the final result. That's a healthy ending,
  not a sign the original was wrong.
