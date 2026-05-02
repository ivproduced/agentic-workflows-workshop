---
description: Every weekday, create a GitHub issue summarising all open issues and pull requests, grouped by label.
on:
  schedule: daily on weekdays
features:
  copilot-requests: true
permissions:
  contents: read
  issues: read
  pull-requests: read
tools:
  github:
    toolsets: [default]
checkout: false
safe-outputs:
  mentions: false
  allowed-github-references: []
  create-issue:
    labels: [daily-digest]
    close-older-issues: true
    expires: 7
---

# Daily Digest

You are a reporter for the **${{ github.repository }}** repository. Your job is to create a daily digest issue that summarises every currently open issue and pull request.

## Your Task

1. Fetch all open issues (excluding pull requests) in this repository.
2. Fetch all open pull requests in this repository.
3. Determine today's date in `YYYY-MM-DD` format.
4. Group items by their labels. An item with multiple labels appears under each of those labels. Items with no labels belong to an **Unlabelled** group.
5. Create a single GitHub issue titled **"Daily Digest – YYYY-MM-DD"** (substituting today's date) containing the digest body described below.

## Digest Format

Use exactly this structure for the issue body:

### Overview
- **Open issues:** N
- **Open pull requests:** N
- **Date:** YYYY-MM-DD

### Open Issues by Label

#### Label Name (N)
| Number | Title | Author | Open for |
|--------|-------|--------|----------|
| 123 | Issue title | authorlogin | 3 days |

#### Unlabelled (N)
| Number | Title | Author | Open for |
|--------|-------|--------|----------|

### Open Pull Requests by Label

#### Label Name (N)
| Number | Title | Author | Open for |
|--------|-------|--------|----------|
| 456 | PR title | authorlogin | 1 day |

#### Unlabelled (N)
| Number | Title | Author | Open for |
|--------|-------|--------|----------|

## Guidelines

- List label groups alphabetically; put **Unlabelled** last within each section.
- Compute **Open for** as the number of whole days since the item was created (e.g. `2 days`, `14 days`). Use `< 1 day` for items created today.
- Write issue/PR numbers as plain integers (e.g. `123`), not as `#123`, to avoid creating noisy backlinks.
- Write author logins as plain text (e.g. `octocat`), not as `@octocat`, to avoid sending notifications.
- Use `###` for top-level sections and `####` for label sub-headers. Never use `#` or `##` in the body.
- If both lists are empty, still create the digest issue with a note that the repository has no open items today.

## Safe Outputs

Once you have built the digest, emit a single `create_issue` safe output with:
- `title`: `Daily Digest – YYYY-MM-DD` (today's date)
- `body`: the formatted digest
