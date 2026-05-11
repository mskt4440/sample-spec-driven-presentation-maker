# Maintenance Notes

This file is fork-specific (`mskt4440/sample-spec-driven-presentation-maker`) and lives on the
`develop` branch. It is not intended to be merged upstream.

## Repository layout

- `origin` — `https://github.com/mskt4440/sample-spec-driven-presentation-maker.git` (this fork)
- `upstream` — `https://github.com/aws-samples/sample-spec-driven-presentation-maker.git`
- Working branch: `develop` (contains local extensions on top of `upstream/main`)
- Skill symlink: `~/.local/share/skills/pptx-maker/` symlinks into `skill/` of this repo, so any
  edit on `develop` is immediately visible to Claude Code's skill resolver.

## Sync procedure: pull upstream/main into develop

Run this whenever upstream releases new commits.

### 1. Fetch and inspect

```
git -C <repo> fetch upstream
git -C <repo> log --oneline develop..upstream/main
git -C <repo> rev-list --count develop..upstream/main
git -C <repo> rev-list --count upstream/main..develop
```

If `develop..upstream/main` is empty, no sync is needed.

### 2. Merge (do not rebase)

Past history uses merge commits, not rebase, to preserve the `develop` line. Keep that style.

```
git -C <repo> switch develop
git -C <repo> merge upstream/main --no-ff -m "Merge remote-tracking branch 'upstream/main' into develop"
```

### 3. Resolve conflicts

Local extensions on `develop` that are likely to clash with upstream changes:

| Area                                                  | Local extension                                                                                                       |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `skill/sdpm/checks/font_size.py`                      | Token discipline check (this fork shipped a similar check earlier; upstream later added a more complete version)      |
| `skill/references/workflows/create-new-2-compose.md`  | Token Discipline section: global default style resolution order (`~/.kiro/local/pptx-assets/styles/`), drift analysis |

General conflict policy:

- For **code** (`*.py`): prefer upstream's implementation when it supersedes the local one
  (upstream tends to add edge cases like directory-input support, slug attribution).
- For **workflow docs** (`skill/references/workflows/*.md`): keep local-extension paragraphs
  (Resolution order, drift analysis) and absorb upstream's wording improvements (e.g. switching
  from `presentation.json` to `slide JSON`).

### 4. Verify

Run the full test suite from the **repo root** (not `--project skill`) so the root `pyproject.toml`
dev group resolves `boto3` and other shared deps:

```
uv run pytest tests/
```

Expected: ~150 passed, possibly some `skipped`. `test_authz.py` requires `boto3` from the root
project, not the skill subproject — running with `uv run --project skill pytest` will fail on
those four tests with `ModuleNotFoundError: No module named 'boto3'`.

### 5. Push to origin

`develop` is tracked on `origin/develop` as a backup of local extensions. Push the merge commit:

```
git -C <repo> push origin develop
```

The first push (when the remote branch did not yet exist) used `git push -u origin develop` to
create the upstream tracking; subsequent syncs only need `git push`.

## History

- `2026-05-11` — Merged upstream commits 2e2aba8 (`feat: enforce fontSize token discipline`),
  a943f2c (`fix: support directory input for fontSize token check`),
  b68758a (`fix: escape % in argparse help string for Python 3.14`). Conflicts in
  `skill/sdpm/checks/font_size.py` and `skill/references/workflows/create-new-2-compose.md`
  were resolved per the policy above. Merge commit: `b89e60f`.
