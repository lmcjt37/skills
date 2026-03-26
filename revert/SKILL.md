---
name: revert
description: Revert a previously merged pull request by creating a fresh branch from the target base branch, applying the correct git revert flow, pushing the branch, and opening a GitHub PR. Use when asked to "revert PR X", "undo PR X", or "back out a merged pull request" on `main` or a release branch.
---

# Revert Merged PR

## When to Use This Skill

- A developer wants to revert a PR that has already been merged
- The revert should be delivered as a new branch and PR
- The revert may need to be applied to more than one base branch, such as `main` and a release branch

## Inputs To Confirm

- The PR URL or PR number
- The target base branch to revert onto, such as `main` or `release/v4.81.0`
- The desired revert branch name, if the user already has a naming convention

If any of those are missing and cannot be inferred safely, ask.

## Workflow

### Step 1: Check repository state

Make sure the worktree is clean before switching branches:

```bash
git status --short --branch
```

If there are unrelated local changes, stop and ask before proceeding.

### Step 2: Inspect the merged PR

Use GitHub metadata to confirm:

- The PR is merged
- The original title
- The original Jira ticket link, if present
- The base branch of the original PR

You also need the merged commit SHA that landed on the target branch.

Useful commands:

```bash
git fetch origin main
git log origin/main --oneline --grep='#6807\|SB-3767'
git show --no-patch --pretty=raw <commit-sha>
```

Useful GitHub tool:

- `mcp__github__pull_request_read` with `method: "get"`

### Step 3: Determine the correct revert shape

Check whether the PR landed as:

- A normal or squash commit: one parent only
- A merge commit: two parents

Rule:

- Single-parent commit: use `git revert <commit-sha>`
- Merge commit: use `git revert -m 1 <commit-sha>`

Do not guess. Inspect the commit object first.

### Step 4: Create the revert branch from the target base

Fetch the target branch, switch to it, and branch from the current remote tip:

```bash
git fetch origin <target-branch>
git checkout <target-branch>
git checkout -b <revert-branch-name>
```

Verify the commit being reverted is actually reachable from the target branch:

```bash
git merge-base --is-ancestor <commit-sha> HEAD; echo $?
```

Expect `0`. If not, stop and reassess.

### Step 5: Apply the revert

Revert the exact merged commit:

```bash
git revert --no-edit <commit-sha>
```

Or, for a merge commit:

```bash
git revert --no-edit -m 1 <commit-sha>
```

If conflicts appear:

- Resolve them deliberately
- Preserve unrelated later changes on the target branch
- Continue with `git revert --continue`

Never use destructive reset commands to force through the revert.

### Step 6: Verify the result

Confirm:

- The revert commit was created
- The worktree is clean
- The changed files match expectations

Useful commands:

```bash
git status --short --branch
git show --stat --format=short HEAD
git diff --name-only HEAD^ HEAD
```

If the revert removed tests or docs that were introduced by the original PR, that is expected.

### Step 7: Push the branch

```bash
git push -u origin <revert-branch-name>
```

### Step 8: Open the PR

Create a PR back to the target base branch using the repository template structure:

```md
## Jira ticket
[SB-3767](https://economist.atlassian.net/browse/SB-3767)

## References
- Reverts [#6807](https://github.com/EconomistDigitalSolutions/mobile-liskov/pull/6807)

## What's new
- Reverts the `#6807` changes from `<target-branch>`
- Restores the previous behaviour that existed before that PR

## Impact areas
- Describe the feature or flow affected, not the file list

## After
```

Use a revert-focused title, for example:

- `Revert [SB-3767] Native Topic offline view states`
- `Revert [SB-3767] Native Topic offline view states on main`

Useful GitHub tool:

- `mcp__github__create_pull_request`

## Notes

- If the user wants the same revert on multiple branches, repeat the process independently for each branch.
- Prefer reverting the merged commit rather than manually undoing files.
- Keep the user informed of the exact branch name, revert commit SHA, and PR URL you created.
- If you did not run tests or linting, state that explicitly in the final handoff.
