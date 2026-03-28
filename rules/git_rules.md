# Git & GitHub Rules

> **Purpose:** Version control strategy, branching model, commit conventions, tagging,
> and GitHub integration rules for projects managed by the Agnostic Orchestrator Agent.
>
> **Scope:** All projects with `version_control.enabled: true` in `config.json`.
> **Condition:** These rules are IGNORED if `version_control.enabled` is `false` or absent.

---

## 1. Branching Model

| Branch              | Purpose                              | Created From | Merges Into        |
| :------------------ | :----------------------------------- | :----------- | :----------------- |
| `main`              | Production-ready, stable releases    | —            | —                  |
| `develop`           | Integration branch for next release  | `main`       | `main`             |
| `feature/<name>`    | New feature development              | `develop`    | `develop`          |
| `fix/<name>`        | Bug fix                              | `develop`    | `develop`          |
| `hotfix/<name>`     | Urgent fix on production             | `main`       | `main` + `develop` |
| `release/<version>` | Release preparation & stabilization  | `develop`    | `main` + `develop` |

### Rules:
- **Never commit directly to `main` or `develop`.**
- Every change goes through a feature/fix branch → Pull Request → merge.
- Delete branches after merge.
- Branch names: lowercase, hyphens, max 50 chars. e.g., `feature/add-user-auth`.
- **Repositories without `develop`:** If no `develop` branch exists, feature/fix branches
  are created from and merged into `version_control.default_branch` (typically `main`).
  The agent SHOULD create `develop` from `default_branch` only if the user explicitly
  requests the gitflow model.

---

## 2. Commit Conventions (Conventional Commits)

Every commit message MUST follow this format:

```
<type>(<scope>): <short description>

[optional body]

[optional footer(s)]
```

### Allowed Types:

| Type       | Description                                          | Version Bump |
| :--------- | :--------------------------------------------------- | :----------- |
| `feat`     | A new feature                                        | MINOR        |
| `fix`      | A bug fix                                            | PATCH        |
| `docs`     | Documentation changes only                          | none         |
| `style`    | Formatting, missing semicolons, etc. (no logic)     | none         |
| `refactor` | Code restructuring without feature/fix change        | none         |
| `perf`     | Performance improvement                              | PATCH        |
| `test`     | Adding or updating tests                             | none         |
| `build`    | Changes to build system or dependencies              | none         |
| `ci`       | Changes to CI/CD configuration                      | none         |
| `chore`    | Maintenance tasks, dependency updates                | none         |
| `revert`   | Reverts a previous commit                            | PATCH        |

### Breaking Changes:
- Append `!` after the type/scope to signal a breaking change: `feat!: remove legacy API`.
- Or include `BREAKING CHANGE:` in the footer.
- Breaking changes trigger a **MAJOR** version bump.

### Examples:

```
feat(auth): add OAuth2 login support

fix(api): handle null response from database

feat!: drop support for Node.js 14

BREAKING CHANGE: Node.js 14 is no longer supported. Upgrade to Node.js 18+.

docs(readme): update installation instructions

chore(deps): upgrade express to 4.18.2
```

### Rules:
- Short description: max 72 characters, imperative mood, no period at end.
- Scope: optional, represents the affected module or component.
- Body: explain the **why**, not the **what**. Wrap at 100 characters.

---

## 3. Versioning (SemVer)

All projects follow [Semantic Versioning 2.0.0](https://semver.org/): `MAJOR.MINOR.PATCH`.

### Version Bump Rules (derived from commit types since last release):

| Condition                                                  | Bump    | Example           |
| :--------------------------------------------------------- | :------ | :---------------- |
| Any commit with `BREAKING CHANGE` or `!` suffix            | MAJOR   | `1.2.3` → `2.0.0` |
| Any `feat` commit (and no breaking change)                 | MINOR   | `1.2.3` → `1.3.0` |
| Any `fix`, `perf`, or `revert` commit (and no feat/break)  | PATCH   | `1.2.3` → `1.2.4` |
| Only `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore` | none | version unchanged |

### Algorithm (applied at STEP 7):

1. Collect all commits on the working branch since the last tag.
2. Determine the highest applicable bump level (MAJOR > MINOR > PATCH > none).
3. Increment the appropriate version component; reset lower components to `0`.
4. Store the resulting version in `VAR_CURRENT_VERSION` in `session_state.json`.

### Initial Version:
- If no tag exists in the repository, start at `0.1.0` (first working release with minimal
  features). Use `0.0.1` only for pre-release commits that are not yet considered functional.

---

## 4. Tagging

- Tags MUST be annotated: `git tag -a v<VERSION> -m "Release v<VERSION>"`.
- Tag format: `v<MAJOR>.<MINOR>.<PATCH>` — e.g., `v1.3.0`.
- **Never delete or move a published tag.**
- Tags are pushed explicitly: `git push origin v<VERSION>`.
- A tag must only be created on the `main` branch after a successful merge.

---

## 5. Pull Requests

- Every feature/fix/hotfix branch MUST be merged via a Pull Request.
- PR title must follow Conventional Commits format (same as a commit message).
- PR description MUST include:
  - **What:** summary of changes.
  - **Why:** motivation or linked issue.
  - **How to test:** steps to verify the change.
- At least one approval is required before merging (when team size > 1).
- All CI checks must pass before merge.
- Use **squash merge** for feature/fix branches to keep `develop` history clean.
- Use **merge commit** (no squash) when merging `release/*` or `hotfix/*` into `main`.

---

## 6. CI/CD Artifacts

- Every push to `main` MUST trigger a CI/CD pipeline.
- CI pipelines MUST:
  1. Run all automated tests.
  2. Build the Docker image(s).
  3. Tag the image with the version: `<image-name>:<version>` and `<image-name>:latest`.
  4. Push the image to the configured container registry.
- Artifacts (binaries, packages, images) MUST be versioned and traceable to a git tag.
- Pipeline configuration files live in `.github/workflows/` (GitHub Actions) or equivalent.
- **Never deploy from a branch other than `main`** (or the configured `default_branch`).

---

## 7. Repository Hygiene

- Keep `.gitignore` up to date — never commit secrets, build artifacts, or `node_modules`.
- Store secrets in environment variables or a secrets manager — never in the repository.
- The `CHANGELOG.md` (or `changelog.md`) is updated automatically at STEP 7 before tagging.
- Stale branches (merged > 30 days ago) should be deleted.
- Use `git rebase` to keep feature branches up to date with `develop` before opening a PR.
