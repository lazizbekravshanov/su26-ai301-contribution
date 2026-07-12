# Rust Contributions Handoff Log

Last updated: 2026-07-12 16:56 EDT

## Purpose

This file is the handoff record for the next two CodePath AI301 contributions. A new agent should
read this file first, verify the live GitHub state, and continue from the **Next Actions** section.

Do not start implementation until the `git-cliff` maintainer confirms that the relevant issue is
available and agrees with the proposed scope.

## Selected Project

**Repository:** [`orhun/git-cliff`](https://github.com/orhun/git-cliff)  
**Stack:** Rust CLI application and library workspace  
**Contribution order:** issue #1182 first, then issue #412

Why this project was selected:

- It is Rust-first and both issues appear in `AI301 Issue Candidates.xlsx`.
- The repository is active. External pull requests were merged as recently as July 12, 2026.
- Setup is conventional Cargo tooling, with stable Rust for builds/tests and nightly Rust for
  formatting.
- Its current `CONTRIBUTING.md` contains no explicit ban on AI-assisted contributions.
- Working in one repository lets us reuse setup and build a maintainer relationship.

Ruma was investigated and rejected even though it had technically suitable issues: Ruma's
`CONTRIBUTING.md` explicitly bans LLM-generated code, documentation, issues, and comments, which
conflicts with this AI301 workflow.

## Issue 1: Search for `.config/cliff.toml`

**Issue:** [`orhun/git-cliff#1182`](https://github.com/orhun/git-cliff/issues/1182)  
**Title:** Support searching in `.config` folder  
**Labels:** `feature/request`, `good first issue`  
**Live state when checked:** Open, no current assignee, no linked or unlinked open PR found

### Claim caveat

This issue has a stale soft claim. In June 2025, issue author and contributor `@joshka` wrote:

> I'll definitely get to this though.

There has been no visible implementation activity since then, but we must ask before taking over.

### Comment posted

Claim/scope question posted on July 12, 2026:

<https://github.com/orhun/git-cliff/issues/1182#issuecomment-4952729917>

The comment:

- asks `@joshka` whether he still plans to implement it;
- asks `@orhun` for permission and scope confirmation;
- proposes distinguishing explicit `--config` from automatic discovery;
- proposes adding `.config/cliff.toml` while preserving existing precedence;
- proposes regression tests;
- explicitly avoids broad Clap/`--init` refactoring unless requested.

### Current technical understanding

Relevant files on current `main`:

- `git-cliff/src/args.rs`
  - `Opt::config` is a `PathBuf`.
  - Clap gives it `default_value = DEFAULT_CONFIG`.
  - This makes an omitted `--config` and an explicitly supplied `--config cliff.toml` difficult to
    distinguish.
- `git-cliff/src/lib.rs`
  - Loads the selected config and contains the current default/global/embedded fallback behavior.
- `git-cliff-core/src/lib.rs`
  - Defines `DEFAULT_CONFIG` as `"cliff.toml"`.
- `git-cliff-core/src/embed.rs`
  - Separately embeds the default config and built-in example configs.

The issue was initially described as a few-line search-path change. `@joshka` later found that it
touches config argument semantics and possibly overlaps with cleanup of the embedded config types.
`@orhun` agreed that config/argument handling could improve but wanted further discussion about
Clap usage.

### Tentative acceptance criteria

These are proposals only; update them after maintainer feedback:

- With no explicit `--config`, `git-cliff` discovers `.config/cliff.toml`.
- Existing config search behavior and precedence remain backward compatible.
- An explicit `--config <PATH>` always wins and is never silently replaced by discovery.
- Existing embedded/default fallback behavior remains intact.
- Tests cover root `cliff.toml`, `.config/cliff.toml`, explicit paths, and precedence.
- Avoid unrelated CLI redesign.

## Issue 2: User-Defined Templates

**Issue:** [`orhun/git-cliff#412`](https://github.com/orhun/git-cliff/issues/412)  
**Title:** Add flags and options to support user-defined templates  
**Labels:** `feature/request`, `good first issue`  
**Live state when checked:** Open, no current assignee, no comments, no active linked or unlinked PR

The issue references merged PR #370, which added initialization from built-in templates. That PR is
historical context, not an implementation of #412.

### Comment posted

Claim/scope question posted on July 12, 2026:

<https://github.com/orhun/git-cliff/issues/412#issuecomment-4952729922>

The proposed scope in the comment:

- add `--templates-dir <PATH>`;
- support `GIT_CLIFF_TEMPLATES_DIR`;
- add `--list-templates`;
- list built-in and user-provided templates;
- let a user template override a built-in template with the same name;
- add CLI/template-resolution tests and documentation.

The comment says this is planned as a follow-up after #1182 and that implementation will wait for
scope confirmation. The issue is from 2023, so the maintainer must confirm that the interface is
still desired.

### Current technical understanding

Relevant files on current `main`:

- `git-cliff/src/args.rs`
  - `--init [CONFIG]` is represented as `Option<Option<String>>`.
  - There is currently no `--templates-dir` or `--list-templates`.
- `git-cliff/src/lib.rs`
  - `init_config()` chooses either a named built-in template or the embedded default.
- `git-cliff-core/src/embed.rs`
  - `EmbeddedConfig` embeds `config/cliff.toml`.
  - `BuiltinConfig` embeds files under `examples/`.
  - `BuiltinConfig::get_config()` normalizes a missing `.toml` extension and reads only embedded
    templates.

### Tentative acceptance criteria

These are proposals only; update them after maintainer feedback:

- A user can supply a template directory through CLI and environment variable.
- `--list-templates` reports both built-in and user templates deterministically.
- User templates override built-ins with matching names, if the maintainer confirms that
  precedence.
- Existing `--init` behavior remains backward compatible.
- Missing directories, unreadable files, duplicate names, and extension normalization have clear
  behavior and tests.
- Documentation explains discovery and precedence.

## Repository Contribution Requirements

From the current `git-cliff` contribution guide:

1. Discuss changes with the owner before implementation.
2. Use Rust 1.85.1 or later for stable builds/tests.
3. Install nightly Rust for formatting.
4. Fetch upstream tags because tests require them.
5. Follow Conventional Commits.
6. Add or update tests.
7. Before submitting, run:

```sh
cargo test
cargo clippy --tests --verbose -- -D warnings
cargo +nightly fmt --all -- --check --verbose
```

If snapshot tests change:

```sh
env UPDATE_EXPECT=1 cargo test
```

The repository asks contributors to update documentation and complete its PR template.

## Next Actions

1. Check both issue threads for replies:

```sh
gh issue view 1182 --repo orhun/git-cliff --comments
gh issue view 412 --repo orhun/git-cliff --comments
```

2. For #1182:
   - If `@joshka` is still working on it, do not duplicate the work; ask whether another issue is
     preferable.
   - If `@joshka` releases it and `@orhun` approves the scope, begin #1182.
   - If the maintainer requests a broader config/Clap refactor, stop and write a plan before coding.

3. For #412:
   - Wait for `@orhun` to confirm that the old interface is still wanted.
   - Do not implement it in parallel with #1182 unless the maintainer explicitly wants both moving
     at once.
   - Prefer finishing or at least opening #1182 before beginning #412.

4. After approval, set up the target repository in a separate folder, likely
   `~/pet/git-cliff`, with the fork as `origin` and `orhun/git-cliff` as `upstream`.

5. Establish a clean baseline before changing code:
   - record Rust/Cargo versions;
   - fetch tags;
   - run the relevant focused tests;
   - run `cargo test`;
   - inspect any baseline failures before attributing them to new work.

6. Use test-driven development for each behavior change:
   - add a focused failing regression/feature test;
   - verify it fails for the intended reason;
   - implement the smallest approved change;
   - run focused and full verification.

7. Keep one issue per branch and one PR per issue.

## Communication and AI Notes

- Be transparent that this work is part of CodePath AI301 when appropriate.
- The student remains responsible for understanding and reviewing every submitted line.
- Record AI assistance and independent verification in the main contribution journal.
- Re-check the repository's contribution and AI policies before each PR because policies can
  change.
- Do not post additional comments, claim assignments, or open PRs without reviewing the exact
  text and current GitHub state first.

## Status Summary

- [x] Filtered the CodePath list for Rust candidates.
- [x] Live-checked claims, assignees, comments, and linked PRs.
- [x] Selected `orhun/git-cliff`.
- [x] Selected #1182 as the first contribution.
- [x] Selected #412 as the follow-up contribution.
- [x] Posted claim/scope comments on both issues.
- [ ] Receive availability and scope confirmation for #1182.
- [ ] Receive scope confirmation for #412.
- [ ] Set up the local `git-cliff` fork/clone.
- [ ] Reproduce and plan #1182.
- [ ] Implement, verify, and submit #1182.
- [ ] Reproduce and plan #412.
- [ ] Implement, verify, and submit #412.
