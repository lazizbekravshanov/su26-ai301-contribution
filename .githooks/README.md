# Git hooks

Version-controlled hooks for this repo. They are **not** active until you point
git at this directory (git does not auto-enable tracked hooks):

```sh
git config core.hooksPath .githooks
```

Run that once per clone. To verify it is active: `git config core.hooksPath`
should print `.githooks`.

## `commit-msg`

Enforces [Conventional Commits](https://www.conventionalcommits.org/) on every
commit's subject line:

```
<type>[optional scope][!]: <description>
```

- **types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`,
  `build`, `ci`, `chore`, `revert`
- optional `(scope)` and optional `!` (breaking change)
- a non-empty description is required

Git's own auto-generated subjects (`Merge …`, `Revert …`, `fixup! …`,
`squash! …`, `Reapply …`) are allowed through. Subjects longer than 72
characters produce a non-blocking warning.

Dependency-free (POSIX `sh`) — no Node, commitlint, or husky required. In a
Node project the equivalent would be commitlint + husky; here a native hook
keeps this docs repo free of a toolchain.

To bypass in an emergency: `git commit --no-verify` (avoid for normal work).
