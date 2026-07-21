# Combined Feedback Checklist

A single, running checklist distilled from every piece of maintainer / reviewer / mentor
feedback across all contribution cycles. The idea (courtesy of a Slack mentor): review my
own work against this list **before** each submission, so I keep making *new* mistakes
instead of repeating old ones.

Each item is tagged with where it came from. New feedback gets added here as a checkbox,
not just noted once and forgotten.

---

## Phase I — Choosing & claiming an issue

- [ ] **Read the *entire* issue thread before claiming**, not just the title/first post. A contributor may have quietly spoken for it. _(Cycle 2: my claim-detection missed "happy to work on **these**" — someone already had #961.)_
- [ ] **Verify claimability three ways**: no assignee, no linked/closing PR, *and* no recent "I'll take this" commenter. _(Cycle 1: first two picks were already claimed.)_
- [ ] **Confirm the project is actively maintained** — sample maintainer reply latency across the issue tracker; make "active" a measured criterion, not a vibe. _(Cycle 2.)_
- [ ] **Re-verify the issue against the current default branch before committing to it.** Old issues rot: the fix may be done, obsolete, or moved. _(Cycle 3 #829: the JAX half was obsolete and `detect_singletons` had migrated numba→Rust — following the issue literally would have shipped dead code.)_
- [ ] **Read `CONTRIBUTING.md` first** — for LLM-policy bans, sibling-checkout/test requirements, and PR conventions. _(Cycle 1: sibling repo needed for `just test-ts`; Cycle 5: rejected Ruma because its CONTRIBUTING bans LLM-authored code.)_

## Phase II — Reproduce & plan

- [ ] **Reproduce with a *natural* trigger**, not a contrived one — it makes the root cause obvious and the regression test honest. _(Cycle 2: built an always-treated-unit panel as a natural repro.)_
- [ ] **Find the root cause, name the specific file/function** — don't stop at the symptom. _(All cycles; UMPIRE.)_
- [ ] **Verify external contracts by reading/executing, not assuming.** Confirm wire formats and API shapes against reality before wiring anything. _(Cycle 1: proved `disabled=True` → `options:{"disabled":true}` by running the deserialize path; Cycle 4: verified MiniMax chat vs. T2A endpoints/auth before coding.)_
- [ ] **Find an in-repo "Match" pattern** — an analogous existing implementation to mirror, so the fix looks native. _(All cycles.)_
- [ ] **Front-load design questions** in the claim/PR before writing code, and keep the offer open in the PR. _(Cycle 2: asked "hard error vs. graceful?" up front.)_

## Phase III — Build

- [ ] **Write the failing test first (TDD), watch it go red→green.** _(Cycles 1–5.)_
- [ ] **Run the *full* suite, not just the sub-suite I've been iterating on.** A test in a sibling module can encode an assumption my change breaks. _(Cycle 4: `test_team_scoped_services` in `experiments/test_models.py` hard-coded the service list and slipped past me because I'd only run the `service_providers` suite — @snopoke caught it in CI.)_
- [ ] **Keep the main diff atomic; ship unrelated fixes as a separate PR.** _(Cycle 2: split the mypy fixes into #1369 — merged same day, and it turned #1368's CI green.)_
- [ ] **No unrelated churn in the diff** — no reformatting, no commented-out code. **A formatter/linter warning is a *claim*, not a verdict** — check whether the repo intentionally excludes those paths. _(Cycle 4: `ruff format` "would reformat" a migration, but `migrations/` is in the repo's ruff `exclude`; reformatting would have added churn the project avoids.)_
- [ ] **Descriptive commit messages** (Conventional Commits), regular cadence, no `wip`/`fix`/`asdf`.
- [ ] **When adding to an enum/registry, regenerate the derived artifacts** (schemas, generated files) surgically. _(Cycle 4: adding a provider drifted `api-schemas/export.yml`; regenerated with `--api-version export` only.)_

## Phase IV — Submit & iterate

- [ ] **Fact-check the PR description against the source** — especially maintainer names/handles. AI-assisted drafts hallucinate attributions. _(Cycle 1: self-review caught me crediting the wrong maintainer before it went public.)_
- [ ] **PR body: WHY before WHAT, multiple paragraphs**, acceptance-criteria checklist, and before/after evidence (test output/logs for backend, screenshot for UI).
- [ ] **Close keyword on the *completing* PR only.** For a multi-PR feature, earlier PRs say "Part of #X"; the last one carries "Closes #X". _(Cycle 4: #3800 "Part of #2979", #3801 "Closes #2979" — issue auto-closed at the right moment.)_
- [ ] **Plan multi-artifact collisions up front.** Numbered artifacts (migrations, lockfiles, codegen manifests) can conflict even when git merges cleanly — flag it at open-time so the second-to-merge is a one-liner. _(Cycle 4: flagged the `0063` Django migration collision on both PRs; when #3800 merged, the fix was one renumber commit.)_
- [ ] **On a fork, rebase onto `upstream/<default>`, never the stale `origin/<default>`.** _(Cycle 4: `origin/main` lacked the just-merged PR; rebasing there would have "resolved" a conflict against week-old main and re-collided.)_
- [ ] **After force-pushing to a reviewed PR, prove nothing else changed** — `git range-diff` and say so in the comment, so the reviewer doesn't re-review from scratch. _(Cycle 4: showed all six reviewed commits replayed byte-identical.)_
- [ ] **Approvals are perishable** — every push after an approval dismisses it. Batch changes; expect to ask for a re-approve. _(Cycle 4: #3801's two approvals were auto-dismissed by my fixes.)_
- [ ] **@mention the reviewer to surface the PR** — external contributors can't `gh pr edit --add-reviewer` (404). _(All cycles.)_
- [ ] **Review the maintainer's own commits on my PR** — a good comment on my *own* PR can still improve the result. _(Cycle 3: my carryforward point became @s3alfisc's final commit before merge; Cycle 2: endorsed @leostimpfle's refactor.)_
- [ ] **Diagnose a red CI check before reacting** — is it red for *me* or red for *everyone*? Check the same job on `main`. _(Cycle 5: git-cliff "Test suite" = Codecov-upload flake, "Links" = GitLab 403s — both fail intermittently on `main` too, not my code.)_

## Cross-cutting

- [ ] **No AI attribution** in commits or PRs (own every line). _(Project rule, all repos.)_
- [ ] **Evidence before assertions** — a failing→passing test, a wire format proven by execution, a diff audited line by line. The throughline across every cycle.

---

_Started 2026-07-21 after Cycles 1–5 (6 PRs merged). Append new feedback as checkboxes; never delete — a struck-through lesson is still a lesson._
