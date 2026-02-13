---
name: pr
description: >
  Use when the user wants to create a GitHub pull request from the current branch,
  open a PR, or push and create a PR with a structured description.
  Triggers: "create pr", "open pr", "pull request", "gh pr create", "create pull request".
argument-hint: "[optional PR title or --base <branch>]"
user-invocable: true
allowed-tools: Bash(git:*), Bash(gh:*), Read, Grep, Glob
---

# Create PR

Announce: "I'm using the pr skill to open a GitHub pull request from the current branch."

## Step 1: Gather Context

Run in parallel:
- `git branch --show-current` (head branch name)
- `git remote` (list remotes to find the default, usually `origin`)
- `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null` (detect default branch, e.g. `origin/main`)
- `git status --short` (check for uncommitted changes)
- `gh auth status 2>&1` (verify gh CLI is installed and authenticated)

**If `gh` is not installed or not authenticated:** inform the user that the `gh` CLI is required and must be authenticated (`gh auth login`). Stop.

**If this is not a git repository:** inform the user and stop.

## Step 2: Determine Base Branch

Determine the default branch:

1. `git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null` — extract branch name (e.g. `origin/main` → `main`).
2. If that fails: `git remote set-head origin --auto 2>/dev/null` and retry step 1.
3. If still unresolved: fall back to `main` if `origin/main` exists, then `master` if `origin/master` exists.
4. If nothing works: ask the user (see fallback below).

If `$ARGUMENTS` contains `--base <branch>`, use that as the base branch instead.

**If no base branch can be determined:**

```
AskUserQuestion(
  header: "Base branch",
  question: "Could not detect the default branch. Which branch should the PR target?",
  options: [
    "main" -- Use main as the base branch,
    "master" -- Use master as the base branch
  ]
)
```

## Step 3: Validate Branch

**If the current branch IS the base branch:** inform the user that they are on the base branch and cannot create a PR from it. Stop.

**If there are uncommitted changes:** warn the user that there are uncommitted changes that will not be included in the PR, but proceed.

## Step 4: Collect Commits

Run:
- `git merge-base origin/<base> HEAD` to find the divergence point.
- `git log --format="%h%x09%s%x09%b" <merge-base>..HEAD` for each commit's short SHA, subject, and body.
- `git diff --stat origin/<base>..HEAD` for a file change summary.

**If no commits are found between the base and HEAD:** inform the user there are no new commits to include in a PR. Stop.

Also scan commit messages for:
- **JIRA ticket IDs**: patterns like `[A-Z]+-[0-9]+` (e.g. `PROJ-1234`).
- **URLs**: any `https://` links (RFCs, docs, incidents).

## Step 5: Build PR Title and Body

### Title

Determine the PR title using this priority:
1. If `$ARGUMENTS` provides a title (text that is not a `--base` flag), use it.
2. If the branch name contains a ticket ID (e.g. `feat/PROJ-1234-add-widget`), derive a title from it by cleaning up slashes and hyphens into readable text.
3. If there is a single commit, use its subject as the title.
4. Otherwise, synthesize a concise title from the commit subjects: identify the primary theme across commits, then write a single phrase capturing the overall change (e.g., commits "Add user model", "Add auth middleware", "Add login endpoint" → "Add user authentication").

### Body

Construct the PR body using this template. **Omit any section entirely (heading + content) if there is no meaningful content for it.**

```
## 📎 Documentation

- [RFC]({URL})
- [JIRA]({URL})

## 🎯 Motivation

- {why this change is needed}

## 📋 Summary

- {what changed and how}
```

Section order is always: Documentation → Motivation → Summary. Rules:

- **Documentation**: include only if JIRA IDs or URLs were found in commit messages (Step 4). If none found, omit entirely.
- **Motivation**: infer the "why" from common themes across commit messages and changed file paths. Omit if obvious from the title.
- **Summary**: summarize what changed using bullet points. Group related changes logically (e.g. "Added endpoint X", "Refactored module Y", "Updated tests for Z").
- If all three sections are omitted, the body is empty.
- The body must be valid markdown.
- Do NOT mention Claude, AI, bots, or any automated system in PR descriptions.

## Step 6: Push and Create PR

Check if the branch has an upstream remote:
- Run `git rev-parse --abbrev-ref @{upstream} 2>/dev/null`
- If no upstream exists, push the branch: `git push -u origin HEAD`
- If upstream exists, check if local is ahead: `git status` should show up-to-date or ahead. If ahead, push with `git push`.

Create the pull request using a heredoc to preserve markdown formatting:

```bash
gh pr create --base <base> --head <head> --title "<title>" --body "$(cat <<'EOF'
<constructed body>
EOF
)"
```

After the PR is created, report the PR URL to the user.

## Error Handling

- **`gh` not installed or not authenticated**: inform the user to install and authenticate the `gh` CLI. Stop.
- **Not a git repository**: inform the user and stop.
- **On the base branch**: inform the user they need to be on a feature branch. Stop.
- **No diverging commits**: inform the user there are no new commits for a PR. Stop.
- **Default branch not detected**: follow the Default Branch Detection procedure in Step 2, then ask the user if all fallbacks fail.
- **Push failure**: report the push error. Do NOT force-push. Let the user decide how to proceed.
- **PR already exists**: if `gh pr create` fails because a PR already exists for this branch, report the existing PR URL using `gh pr view --web` or `gh pr view --json url`. Let the user decide whether to update it.
- **Network or API failure**: report the error from `gh`. Let the user retry.
