---
name: railspilot-progress-report
description: Generate monthly client progress reports for RailsPilot. Uses gh CLI to list the user's merged PRs, applies feature sizing rules, and drafts a numbered-list email. Use when asked for a monthly progress report, client delivery summary, or "progress report for [Month]".
---

# RailsPilot monthly progress report

Generate a monthly progress report email by analyzing the user's merged PRs for the target month.

Read `template.md` in this skill directory for the canonical output format before drafting.

## Context

RailsPilot is an AI-augmented Rails development service. Tasks use GitHub issue/PR numbering and optional project tags in titles (e.g. `[Epic]`, `[Athena]`).

**Data source: `gh` CLI primary, git fallback.**

- Primary: `gh` with `author:$ME` where `ME` comes from `gh api user --jq .login` (not `git config user.name`).
- Fallback: when `gh` is unavailable or unauthenticated, use git commands below with `--author="$(git config user.email)"`.
- Do not call Jira, Asana, Linear, or any PM API. Do not browse the web to resolve task titles.
- Derive issue links from PR bodies only (`Closes #N`, `Fixes #N`, or a trailing `https://github.com/.../issues/N` URL).

## Step 1: Gather data

Run from the project root. Replace `YYYY-MM` with the target month.

```bash
ME="$(gh api user --jq .login)"

gh pr list --state merged \
  --search "merged:YYYY-MM-01..YYYY-MM-31 author:$ME" \
  --limit 100 \
  --json number,title,mergedAt,additions,deletions,changedFiles,author,body,url
```

For each PR, extract the linked issue from the body:

- `Closes #N` / `Fixes #N` / `Resolves #N`
- Trailing issue URL in the body

Collect loose commits only for dev-improvement slots (not headline features):

```bash
git log --author="$(git config user.email)" \
  --after="YYYY-MM-01" --before="YYYY-MM+1-01" \
  --pretty=format:"%h %s" --no-merges
```

### Git fallback (offline / no `gh` auth)

```bash
AUTHOR="$(git config user.email)"

git log --author="$AUTHOR" --merges \
  --after="YYYY-MM-01" --before="YYYY-MM+1-01" \
  --pretty=format:"%h %s"

git log --author="$AUTHOR" \
  --after="YYYY-MM-01" --before="YYYY-MM+1-01" \
  --pretty=format:"%h %s" --no-merges
```

## Step 2: Authorship filter

Only include work authored by `$ME`.

1. Filter with `author:$ME` in the `gh pr list --search` query.
2. When formatting, assert each PR's `.author.login == $ME`. Drop any that fail.
3. Exclude PRs the user did not author (co-authored-only, merged on behalf of others, review merges).
4. Print an internal **Excluded (other authors)** list (PR number, title, author login) for transparency. Never include this in the client email.

## Step 3: Categorize and size features

Apply these mechanical rules before drafting.

### Split (rare)

Split a PR into two numbered feature slots only when it spans clearly distinct subsystems **and** is large:

- Rough guide: > ~800 changed lines (`additions + deletions`), or
- > ~20 files with separable concerns.

Otherwise: **one PR = one slot**.

### Consolidate (common for small work)

Merge into a single numbered slot (with sub-bullets) or into the dev-improvements item when a PR is:

- Docs-only, single-file, or trivial
- Rough guide: ≤ ~100 changed lines **and** ≤ 2 files
- Title starts with `Document`, `Bump`, `Archive`, `Chore`, or similar housekeeping prefix

Canonical consolidate example: a docs-only PR (+85/−0, 1 file) belongs in "Smaller items", not as a headline feature.

### De-duplicate overlapping work

When multiple PRs describe the same feature iteration (e.g. several AI-session PRs with overlapping titles), merge into **one** numbered slot with sub-bullets per PR. Do not give each iteration its own headline slot.

### No double-counting

Before listing any loose commit as a dev improvement:

1. Confirm it is not already contained in a counted PR (`gh pr list --search "<sha>"` or inspect PR commits).
2. If the commit is part of a merged PR in the report period, it is already represented — do not list it again.

### Categories

- **Features**: user-facing functionality, new capabilities, UX improvements
- **Bugfixes**: corrections to existing behavior (own slot when significant; otherwise sub-bullet)
- **Dev improvements**: CI, monitoring, error tracking, developer tooling — group as the last numbered item unless individually significant

Count honestly. Optionally state the feature count in one closing sentence; do not inflate or undercount.

## Step 4: Draft the email

Follow `template.md`. Ask the user for client name and month if not provided.

### Tone

- Professional but warm. Name features, reference PR/issue numbers, state what was built.
- No vague language ("various improvements", "two tasks deployed").
- Forward planning: derive from PR bodies, branch names, or commit cues (`Follow-up`, `TODO`, `Next:`, `WIP`). If nothing is inferrable, use `[Needs input: next priorities]`.

### What NOT to do

- Do not call PM APIs or browse the web for task details
- Do not include other authors' PRs in the client email
- Do not guess issue numbers — only use IDs from PR bodies
- Do not leave forward planning vague ("a task in the backlog")

## Step 5: Self-verification

Before presenting the draft, confirm each item:

- [ ] Every listed PR is authored by `$ME` (re-check `.author.login`)
- [ ] Every PR `mergedAt` falls within the target month (print dates internally)
- [ ] Each issue link was resolved from a PR body, not guessed
- [ ] No commit is double-listed (already inside a counted PR)
- [ ] Small/docs PRs are consolidated, not promoted to headline features
- [ ] Carryover items from the prior month that shipped this month are included and noted if helpful

Present the verified draft for user review before sending.

## Usage

From the project root:

```
Generate the RailsPilot progress report for [Month Year]
```
