---
name: exec-github-cli
description: >-
  Uses GitHub CLI (gh) for auth, repo, PR, issue, run, and release operations.
  Use when the user mentions gh, GitHub CLI, creating/reviewing PRs, listing
  issues, checking workflow runs, or managing releases from the terminal.
---

# GitHub CLI (gh)

## Auth

```bash
gh auth status
gh auth login        # interactive: choose GitHub.com, HTTPS, browser
gh auth refresh -s <SCOPE>   # add scopes such as workflow, repo, read:org
```

Tokens live in the local OS keychain; never commit `GH_TOKEN` or paste tokens in chat.

For CI, set `GH_TOKEN=<TOKEN>` as a secret—do not hardcode.

## Repo

```bash
gh repo view <OWNER>/<REPO>
gh repo clone <OWNER>/<REPO>
gh repo create <REPO> --private --source=. --remote=origin --push
```

## Pull requests

```bash
gh pr list --state open
gh pr view <PR_NUMBER> --comments
gh pr checkout <PR_NUMBER>
gh pr create --base <BASE_BRANCH> --title "<TITLE>" --body "<BODY>"
gh pr diff <PR_NUMBER>
gh pr review <PR_NUMBER> --approve            # or --request-changes -b "<MSG>"
gh pr merge <PR_NUMBER> --squash --delete-branch
```

## Issues

```bash
gh issue list --state open --label <LABEL>
gh issue view <ISSUE_NUMBER>
gh issue create --title "<TITLE>" --body "<BODY>" --label <LABEL>
gh issue close <ISSUE_NUMBER>
```

## Actions / workflow runs

```bash
gh run list --workflow=<WORKFLOW_FILE> --limit 10
gh run view <RUN_ID> --log
gh run watch <RUN_ID>
gh run rerun <RUN_ID> --failed
```

## Releases

```bash
gh release list
gh release create <TAG> --title "<TITLE>" --notes "<NOTES>"
gh release upload <TAG> <ASSET_PATH>
```

## Agent workflow

1. `gh auth status` before any write operation.
2. Use `gh pr view --json` / `gh run view --log` to gather context instead of pasting URLs into chat.
3. For destructive actions (`pr merge`, `release create`, `repo delete`), confirm intent with the user first.

## Troubleshooting

| Symptom | Action |
|---------|--------|
| `gh: command not found` | Install via `brew install gh` or platform package manager |
| 401 / token expired | `gh auth login` or `gh auth refresh` |
| Missing scope (e.g. workflow) | `gh auth refresh -s workflow,repo` |
| Wrong account | `gh auth switch` |
