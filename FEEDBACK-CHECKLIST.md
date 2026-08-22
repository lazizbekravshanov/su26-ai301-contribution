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
- [ ] **When a data label looks physically wrong, check the authoritative source document, and say so on the PR instead of quietly working around it.** _(Cycle 6: the raw EIA sheet labels petcoke stocks "thousand barrels", but petcoke is a solid; @zaneselvans settled it with the EIA-923 form instructions (short tons), @aesharpe escalated it to EIA, and on 2026-08-05 EIA confirmed their own label was wrong. Raising the contradiction fixed the upstream data source, not just our reading of it.)_

## Phase III — Build

- [ ] **Write the failing test first (TDD), watch it go red→green.** _(Cycles 1–5.)_
- [ ] **Run the *full* suite, not just the sub-suite I've been iterating on.** A test in a sibling module can encode an assumption my change breaks. _(Cycle 4: `test_team_scoped_services` in `experiments/test_models.py` hard-coded the service list and slipped past me because I'd only run the `service_providers` suite — @snopoke caught it in CI.)_
- [ ] **Keep the main diff atomic; ship unrelated fixes as a separate PR.** _(Cycle 2: split the mypy fixes into #1369 — merged same day, and it turned #1368's CI green.)_
- [ ] **No unrelated churn in the diff** — no reformatting, no commented-out code. **A formatter/linter warning is a *claim*, not a verdict** — check whether the repo intentionally excludes those paths. _(Cycle 4: `ruff format` "would reformat" a migration, but `migrations/` is in the repo's ruff `exclude`; reformatting would have added churn the project avoids.)_
- [ ] **Descriptive commit messages** (Conventional Commits), regular cadence, no `wip`/`fix`/`asdf`.
- [ ] **When adding to an enum/registry, regenerate the derived artifacts** (schemas, generated files) surgically. _(Cycle 4: adding a provider drifted `api-schemas/export.yml`; regenerated with `--api-version export` only.)_
- [ ] **Keep shared test mocks aligned with the real API — use `vi.spyOn()` for test-only call tracking, not a new mock-only property.** _(Cycle 9: @manzt asked me to drop a `saveCalls` counter I had added to the shared `TestVsCode` mock and spy on the real `save()` method instead.)_
- [ ] **Check the return value of an async platform call that can fail, instead of assuming success.** _(Cycle 9: @manzt asked me to stop instead of proceeding when `document.save()` returns `false`, and to add a test for that path.)_
- [ ] **Install the repo's pre-commit hooks right after cloning, before the first commit.** A missing hook shows up as mysterious diffs in generated files that the reviewer notices before you do. _(Cycle 6: @zaneselvans spotted a mis-sorted row in the dbt seed CSV; `pixi run prek install` fixed the sort and exposed two rows an earlier un-hooked commit had displaced.)_

## Phase IV — Submit & iterate

- [ ] **Fact-check the PR description against the source** — especially maintainer names/handles. AI-assisted drafts hallucinate attributions. _(Cycle 1: self-review caught me crediting the wrong maintainer before it went public.)_
- [ ] **PR body: WHY before WHAT, multiple paragraphs**, acceptance-criteria checklist, and before/after evidence (test output/logs for backend, screenshot for UI).
- [ ] **Close keyword on the *completing* PR only.** For a multi-PR feature, earlier PRs say "Part of #X"; the last one carries "Closes #X". _(Cycle 4: #3800 "Part of #2979", #3801 "Closes #2979" — issue auto-closed at the right moment.)_
- [ ] **Plan multi-artifact collisions up front.** Numbered artifacts (migrations, lockfiles, codegen manifests) can conflict even when git merges cleanly — flag it at open-time so the second-to-merge is a one-liner. _(Cycle 4: flagged the `0063` Django migration collision on both PRs; when #3800 merged, the fix was one renumber commit.)_
- [ ] **On a busy upstream, re-check mergeability while the PR waits.** A green, mergeable PR can silently go CONFLICTING hours later when `main` moves. _(Cycle 6: pudl landed three commits the same afternoon and flipped #5431 to CONFLICTING; caught it on a routine status check, not from any notification. It happened twice in 24 hours.)_
- [ ] **If a release ships while the PR is pending, move the release note out of the shipped version.** Leaving it there claims a released version contains a feature it does not. Find the commit that opened the previous upcoming-release section and copy that pattern exactly, then tell the maintainer where you put it. _(Cycle 6: PUDL cut v2026.8.0 while #5431 waited; the entry moved to a new `v2026.9.x` section matching `1fc32eb5f4`.)_
- [ ] **A pre-commit hook failing en masse in seconds is an environment problem, not a code problem.** Check the env the hook runs in before believing the failure count. _(Cycle 6: pudl's `unit-tests` hook reported 929 errors in 5 seconds purely because `PUDL_INPUT`/`PUDL_OUTPUT` were absent from the commit shell; every hook passed once they were exported.)_
- [ ] **On a fork, rebase onto `upstream/<default>`, never the stale `origin/<default>`.** _(Cycle 4: `origin/main` lacked the just-merged PR; rebasing there would have "resolved" a conflict against week-old main and re-collided.)_
- [ ] **After force-pushing to a reviewed PR, prove nothing else changed** — `git range-diff` and say so in the comment, so the reviewer doesn't re-review from scratch. _(Cycle 4: showed all six reviewed commits replayed byte-identical.)_
- [ ] **Approvals are perishable** — every push after an approval dismisses it. Batch changes; expect to ask for a re-approve. _(Cycle 4: #3801's two approvals were auto-dismissed by my fixes.)_
- [ ] **@mention the reviewer to surface the PR** — external contributors can't `gh pr edit --add-reviewer` (404). _(All cycles.)_
- [ ] **Review the maintainer's own commits on my PR** — a good comment on my *own* PR can still improve the result. _(Cycle 3: my carryforward point became @s3alfisc's final commit before merge; Cycle 2: endorsed @leostimpfle's refactor.)_
- [ ] **Closing a stalled PR with an explicit reopen offer keeps the door open; a quiet close does not.** _(Cycle 5: I closed git-cliff #1583/#1584 after three weeks of silence, each with a note that the branch stays on my fork and I would gladly reopen. Twelve days later @orhun reopened both himself and reviewed one, without needing to ask me for anything.)_
- [ ] **When a maintainer reports a defect, reproduce it before fixing it, and separate what you broke from what was already there.** _(Cycle 5: the `--workdir` regression was genuinely mine, but the "silent fallback" he paired it with is upstream behavior on main; only my docs sentence was wrong. Fixing both as if both were regressions would have changed long-standing behavior without asking.)_
- [ ] **A new warning can be correct and still fire in the wrong place; check every path that reaches it.** _(Cycle 5: the "config file not found" warning I added for a missing `--config` path also fired for `--config detailed`, a built-in configuration name, printing a missing-file warning immediately before the built-in loaded successfully. The eager filesystem lookup was pre-existing and silent; adding a warning turned a latent ordering flaw into visible noise.)_
- [ ] **Diagnose a red CI check before reacting** — is it red for *me* or red for *everyone*? Check the same job on `main`. _(Cycle 5: git-cliff "Test suite" = Codecov-upload flake, "Links" = GitLab 403s — both fail intermittently on `main` too, not my code.)_
- [ ] **A CLA-assistant check that fails after already signing is usually stale, not a real re-ask.** Some bots don't automatically re-scan a new commit; a `recheck` comment on the PR forces it. _(Cycle 9: `CLAAssistant` flipped red on `9818afc` after I'd already signed on marimo-lsp #644; every other check stayed green.)_
- [ ] **When the maintainer asks for a local run of the CI-equivalent suite, run it, and verify an environment limitation actually exists before citing one.** _(Cycle 6: I declined `pixi run pytest-ci` citing a missing ETL data environment; @zaneselvans pointed out my own materialize proved the setup existed and that the tests download their own data.)_
- [ ] **A maintainer's own merge can bump a transitive dependency and silently break a generated artifact I already committed (a migration, a lockfile) — that's their break, not mine, but it's still mine to fix.** Don't hand-edit the regenerated artifact against a guess; regenerate it with the real tool against a real environment. _(Cycle 6: @zaneselvans's 2026-08-08 merge of `main` bumped Alembic to 1.19, which changed how `CHECK` constraints autogenerate and broke the fuel-stocks migration; he flagged it and apologized, and it needs `pixi run alembic revision --autogenerate` against a live dev DB to regenerate correctly. Resolved 2026-08-10: @aesharpe regenerated it herself rather than wait, which is why the revision id changed from `0636821e0069` to `de2fb0442a2a`. Regenerating, not patching, is what the tool does, and the new id is the evidence of it.)_
- [ ] **A PR can be blocked by checks that are absent, not failing.** A fork pull request whose workflows sit at `action_required` shows no red X at all, just nothing, and the PR reads `BLOCKED` while looking approved and healthy. Check for it with `gh api repos/OWNER/REPO/commits/SHA/check-suites` and compare the run count against a merged PR, rather than reading the PR page. _(Cycle 7: mteb #5026 sat four days with two approvals and zero check runs on its head commit; the gate is per push, so an earlier push clearing it proves nothing about the current one.)_
- [ ] **Whether a push dismisses approvals is a repo setting, so check it instead of assuming.** The Cycle 4 "approvals are perishable" lesson is real but not universal, it depends on the branch protection rule, which is knowable before deciding whether to hold a fix. _(Cycle 7: held @Samoed's naming note for six days to protect two approvals; when the fix eventually went up, mteb kept both approvals because it does not dismiss stale reviews. The hold cost time and bought nothing.)_
- [ ] **Re-verify a message's premise at send time, not at draft time.** A drafted comment carries the world state of when it was written, and on an active PR that expires fast. One live-state check immediately before posting is cheap insurance. _(Cycle 7: a nudge asking a maintainer to release the mteb CI run was drafted, approved, and about to go out when a last check showed the run had been released an hour earlier and all 15 checks had passed. Posting it would have asked two maintainers to do something already done. Retired unposted.)_
- [ ] **A decision recorded only in prose cannot bind anything that runs unattended.** If a hold matters, enforce it where the action happens, or do not delegate the action. _(Cycle 7: the journal recorded a deliberate hold on the mteb naming fix; the cloud routine's own prompt told it to push maintainer-requested changes, so it pushed. The note was never a lock. Both AI301 watch routines were retired 2026-08-12 for this reason.)_

- [ ] **A result that exists only in a background process's log is not yet a result.** Read it into the record while the process is alive, or expect to lose it. _(Cycle 7: a three-task verification ran as a background job; the session ended, the job was stopped with no completion record, and its log went with it. The two scores I had read out survived. The third had not, and re-establishing it cost a 22-minute re-run, which returned exactly the same number. Correct all along, unshowable for hours.)_
- [ ] **Silence from a maintainer means queued, not unwanted, and closing is not the same as giving up.** Close with an explicit offer to reopen and leave the branch on your fork. _(Cycle 5: closed git-cliff #1583 and #1584 after three weeks of silence, each with a note saying I would gladly reopen. @orhun reopened both himself twelve days later and reviewed one. The one sentence in the closing comment is what made that possible without him having to ask.)_

- [ ] **Before editing a shared clone, check `git status` and file mtimes, not just your memory of the tree.** A second agent or session can be mid-edit; interleaved writes corrupt both. _(Cycle 5, 2026-08-21: two sessions started on the same git-cliff review. The clean tree had grown 119 uncommitted insertions and the same file hashed differently 25 seconds apart. Catching it before writing is the only reason nothing was lost. Same hazard as Cycle 4's concurrent-session incident.)_
- [ ] **When a flag accepts both a path and a name, "the path does not exist" is not an error condition on its own.** Rule out the name first. _(Cycle 5: `--config` takes built-in template names too, and a name is never a file, so an unconditional not-found warning fired on every valid `--config keepachangelog`. Fixed by deferring the filesystem check until `--config-url` and the built-ins are ruled out.)_
- [ ] **A path fix that works on absolute input is not finished until it is tested on relative input.** `ancestors()` on a relative path walks only that path's own components, then stops. _(Cycle 5: the `--workdir` discovery fix passed for absolute paths and still failed for relative ones, which is the common case and what the repo's own fixture uses. `current_dir.join(workdir)` handles both, since joining an absolute path replaces.)_

## Cross-cutting

- [ ] **No AI attribution** in commits or PRs (own every line). _(Project rule, all repos.)_
- [ ] **Evidence before assertions** — a failing→passing test, a wire format proven by execution, a diff audited line by line. The throughline across every cycle.

---

_Started 2026-07-21 after Cycles 1–5 (6 PRs merged). Append new feedback as checkboxes; never delete — a struck-through lesson is still a lesson._
