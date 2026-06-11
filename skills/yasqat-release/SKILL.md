---
name: yasqat-release
description: >
  End-to-end release workflow for the yasqat Python package — from dev branch
  creation through PyPI publishing. Use when asked to "release yasqat", "publish
  to pypi", "bump version", "prepare a release", "cut a release", "ship it",
  "deploy new version", or any variation of releasing/publishing yasqat. Also
  triggers on "update changelog", "version bump", or "create release PR" in the
  yasqat project context.
---

# yasqat Release Workflow

Guided, sequential release process for yasqat. Each phase has a **checkpoint**
where the user must confirm before proceeding. Never skip a checkpoint.

See [references/checklist.md](references/checklist.md) for the full step-by-step
checklist with commands and edge cases.

## Quick reference

| Item | Location / Command |
|---|---|
| Version | `pyproject.toml` → `version = "X.Y.Z"` |
| Changelog | `CHANGELOG.md` (repo root) |
| Docs changelog | `docs/changelog.qmd` |
| Tests | `uv run pytest` |
| Lint | `uv run ruff check src/ tests/` |
| Format | `uv run ruff format --check src/ tests/` |
| Docs preview | `quarto preview docs/` |
| Local publish | `uv publish --token $PYPI_TOKEN` |
| CI publish | Push tag `vX.Y.Z` → `.github/workflows/publish.yml` |
| CI docs | Push to `main` → `.github/workflows/deploy-docs.yml` |

## Workflow phases

### Phase 1 — Branch setup

Create and checkout `dev` branch from `main`. If `dev` already exists locally,
confirm with user whether to reuse or recreate it.

### Phase 2 — Development

Implementation work happens here. This phase may already be done when the skill
is invoked — ask the user.

### Phase 3 — Changelog

Compare commits since last release tag (`git log v<last>..HEAD --oneline`).
Categorize changes and prepend a new version section to both `CHANGELOG.md` and
`docs/changelog.qmd`. Follow existing format — see checklist for category
conventions and file structure details.

**Checkpoint: Show drafted changelog, ask for approval.**

### Phase 4 — Version bump

Read current version from `pyproject.toml`. Propose next version based on
semver rules (see checklist). Update `version` field in `pyproject.toml`.

**Checkpoint: Confirm version number with user.**

### Phase 5 — README update

Update `README.md` only if new features warrant mention. Touch only the Features
section. Skip if no user-facing features were added.

**Checkpoint: Show diff if changed, confirm.**

### Phase 6 — Quality gates

Run all checks — stop on first failure:

```bash
uv run ruff check src/ tests/
uv run ruff format --check src/ tests/
uv run pytest
```

Fix issues before proceeding.

**Checkpoint: Report results to user.**

### Phase 7 — Docs update

Verify `docs/changelog.qmd` was updated in Phase 3. Check if new modules need
adding to `docs/_quarto.yml` sidebar or `quartodoc` sections.

**Checkpoint: Confirm docs are ready.**

### Phase 8 — Commit and PR

Stage, commit, push, and create PR. Include changelog excerpt in PR body.

**Checkpoint: Show PR URL, wait for user approval.**

### Phase 9 — Merge and sync

After user approves: merge PR, checkout main, pull.

**Checkpoint: Confirm merge succeeded.**

### Phase 10 — Tag and release

Create git tag and GitHub release with changelog as notes.

### Phase 11 — Publish to PyPI

CI auto-publishes on tag push via `.github/workflows/publish.yml`. If user
prefers local publishing, use `uv publish` with token.

**Checkpoint: Confirm package is live on PyPI.**

### Phase 12 — Cleanup

Delete `dev` branch (local and remote, with user confirmation). Verify docs
deployed automatically.
