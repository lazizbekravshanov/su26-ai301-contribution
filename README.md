# Contribution README

**Name:** Lazizbek Ravshanov
**Program:** CodePath AI301, Summer 2026
**Status:** Phase I Complete

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

*To be completed.*

### Understanding the Issue

### Reproduction Process

### Solution Approach

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
