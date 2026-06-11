# yasqat Release Checklist — Detailed Reference

## Phase 1 — Branch setup

```bash
git checkout main
git pull origin main
git checkout -b dev
```

If `dev` already exists:
```bash
git branch -l dev  # check if it exists
# Ask user: reuse existing dev or delete and recreate?
git branch -D dev && git checkout -b dev  # recreate
# OR
git checkout dev && git merge main        # reuse and sync
```

## Phase 2 — Development

Ask the user: "Has development work already been completed, or do you need to
implement changes first?"

If work is done, proceed. If not, implement the requested changes on the `dev`
branch before continuing.

## Phase 3 — Changelog

### Gather commits since last release

```bash
# Find the latest release tag
git describe --tags --abbrev=0
# e.g. v0.3.1

# List commits since that tag
git log v0.3.1..HEAD --oneline --no-merges
```

### Categorize changes

Use these categories (match existing `CHANGELOG.md` style):

| Category | When to use |
|---|---|
| **Breaking changes** | API removals, signature changes, return type changes |
| **New features** | New public functions, classes, parameters |
| **Bug fixes** | Corrections to incorrect behavior |
| **Performance** | Speed or memory improvements |
| **Removals** | Deleted code, deprecated features removed |
| **Documentation** | Doc-only changes (usually not in changelog) |
| **Internal** | Refactoring, test improvements (usually not in changelog) |

### Write changelog entry

Prepend to `CHANGELOG.md` (keep existing entries below):

```markdown
## X.Y.Z (YYYY-MM-DD)

### Category

- Description of change with `code_references` where helpful.
```

### Mirror to docs

Update `docs/changelog.qmd` — same content but preserve the YAML frontmatter:

```yaml
---
title: "Changelog"
toc: false
---
```

Then the same markdown content as `CHANGELOG.md` (minus the `# Changelog` H1).

## Phase 4 — Version bump

### Read current version

```bash
grep '^version' pyproject.toml
# version = "0.3.1"
```

### Determine next version (semver, pre-1.0)

| Change type | Bump | Example |
|---|---|---|
| Breaking API changes | Minor | 0.3.1 → 0.4.0 |
| New features (non-breaking) | Minor | 0.3.1 → 0.4.0 |
| Bug fixes, performance, docs | Patch | 0.3.1 → 0.3.2 |

Post-1.0, breaking changes bump major instead.

### Apply version bump

Edit `pyproject.toml`:
```toml
version = "X.Y.Z"
```

Only this one location — `yasqat.__version__` reads from `importlib.metadata`
at runtime, so no other files need updating.

## Phase 5 — README update

Only modify `README.md` if new user-facing features were added. Target the
`## Features` bullet list. Add new items matching the existing style:

```markdown
- **Feature name**: Brief description
```

Do not rewrite existing bullets or reorganize.

## Phase 6 — Quality gates

Run in order, stop on first failure:

```bash
# 1. Lint
uv run ruff check src/ tests/

# 2. Format check
uv run ruff format --check src/ tests/

# 3. Tests
uv run pytest
```

### Common fixes

- **Lint errors**: `uv run ruff check --fix src/ tests/`
- **Format errors**: `uv run ruff format src/ tests/`
- **Test failures**: Investigate and fix, do not skip

## Phase 7 — Docs update

1. Verify `docs/changelog.qmd` matches `CHANGELOG.md` content (done in Phase 3)
2. If new modules were added to `src/yasqat/`, check:
   - `docs/_quarto.yml` → `sidebar` and `quartodoc.sections` for new entries
   - Create new `.qmd` files in `docs/api/` if needed
3. Optional: `quarto preview docs/` for local verification

## Phase 8 — Commit and PR

```bash
# Stage all release changes
git add pyproject.toml CHANGELOG.md docs/changelog.qmd README.md
# Also stage any other changed files (new modules, docs, tests)
git status  # review what's staged

git commit -m "release: vX.Y.Z"
git push -u origin dev
```

Create PR:
```bash
gh pr create --title "Release vX.Y.Z" --body "$(cat <<'EOF'
## Summary
- Bump version to X.Y.Z
- <key changes from changelog>

## Changelog
<paste changelog section>

## Checklist
- [ ] Tests pass
- [ ] Lint clean
- [ ] Changelog updated
- [ ] Docs updated
EOF
)"
```

## Phase 9 — Merge and sync

```bash
# After user approves the PR
gh pr merge <PR_NUMBER> --merge
git checkout main
git pull origin main
```

## Phase 10 — Tag and release

```bash
git tag vX.Y.Z
git push origin vX.Y.Z

gh release create vX.Y.Z --title "vX.Y.Z" --notes "$(cat <<'EOF'
<changelog section content>
EOF
)"
```

## Phase 11 — Publish to PyPI

### Option A: CI publishing (preferred)

Pushing the `vX.Y.Z` tag triggers `.github/workflows/publish.yml` which:
1. Runs tests
2. Builds with `uv build --no-sources`
3. Publishes with `uv publish` (OIDC or token-based)

Monitor: `gh run list --workflow=publish.yml --limit=1`

### Option B: Local publishing

```bash
uv build --no-sources
uv publish --token $PYPI_TOKEN
```

The token must be available. Ask the user if they have it set as an environment
variable or need to provide it.

### Verify

```bash
# Wait a minute for PyPI to index, then:
pip index versions yasqat
# Or check https://pypi.org/project/yasqat/
```

## Phase 12 — Cleanup

```bash
# Delete local dev branch
git branch -d dev

# Delete remote dev branch (confirm with user first)
git push origin --delete dev
```

Verify docs deployment:
```bash
gh run list --workflow=deploy-docs.yml --limit=1
```

The docs workflow triggers automatically on push to `main`.
