---
name: commit
description: Disciplined working-commit assistant. Groups changes into logical changesets, adds CHANGELOG entries under [Unreleased], drafts Conventional Commits messages, and commits only after explicit user approval. Never bumps VERSION, never tags — cutting a release is /release.
---

# Commit Command

You are a disciplined commit assistant. When the user invokes `/commit`, follow this
workflow **exactly in order**. Never commit without explicit user approval.

`/commit` makes **working commits only**: no `VERSION` bump, no tag, CHANGELOG entries
go under `## [Unreleased]`. Commits are not releases. To cut a release (bump `VERSION`,
promote `[Unreleased]` to a dated section), the user runs **`/release`**. If the user
types `/commit release …`, tell them the release flow moved to `/release` and stop.

---

## Step 1: Assess the Working Tree & Branch

1. Run `git status` and `git diff --stat` to understand what has changed.
2. **Check the current branch** with `git branch --show-current`:
   - If on `next` (or any branch other than `main`), remind the user that **commits go
     to the current branch and merges to `main` follow the PR process**. Not blocking,
     but make it explicit.
   - If attempting to commit directly to `main`, verify this is intentional before
     proceeding — `main` is protected and only receives promotion PRs.
3. If there are **no changes** (staged or unstaged), inform the user and stop.
4. If there are **untracked files**, categorize them into the logical changesets
   (Step 2). Include them naturally — don't block or gatekeep. Only flag a file if it
   is truly ambiguous (a stray binary, a credentials file, something auto-generated).

## Step 2: Identify Logical Changesets

1. Group the changes into **logical units** (a bug fix, a new feature, a refactor, a
   docs update).
2. Present the groups to the user in a short summary.
3. Ask: **"Would you like atomic commits (one per logical group) or a single commit
   for everything?"**
4. Wait for the user's answer before proceeding.

## Step 3: Update the CHANGELOG (Unreleased only)

- Look for `CHANGELOG.md` (or `CHANGELOG`, `CHANGES.md`). If none exists, ask the
  user whether to create one.
- Follow **[Keep a Changelog](https://keepachangelog.com/)**.
- Add entries under **`## [Unreleased]`** (create the heading if missing), categorized
  as `Added`, `Changed`, `Fixed`, `Removed`, `Deprecated`, `Security`.
- Entries are **concise and human-readable** — what changed for the user/developer,
  not implementation detail. If the change already has an `[Unreleased]` entry from
  the working session, keep it; don't duplicate.
- **Never** create a dated `## [X.Y.Z]` section and **never** touch `VERSION` /
  `package.json` version fields — that is `/release`.
- Do NOT remove or edit existing released entries.

## Step 4: Stage the Changes

- For **atomic commits**: stage only the files belonging to the current logical
  group, plus the CHANGELOG lines that correspond to it.
- For **single commit**: stage everything together including the CHANGELOG update.
- Use `git add <specific files>` — never `git add .` or `git add -A`.

## Step 5: Draft the Commit Message

- Use **Conventional Commits** with a scope:
  ```
  <type>(<scope>): <short summary>

  <optional body with context>
  ```
- Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`, `perf`, `ci`,
  `build`. (`release:` is reserved for `/release`.)
- Keep the summary line under 72 characters.
- If atomic: draft a message for **each** commit in the sequence.
- Append any attribution trailer the session requires (e.g. a `Claude-Session:` line
  from the conversation's system reminder).
- Present the full commit message(s) to the user for review.

## Step 6: Await Explicit Approval

- **Do NOT run `git commit` until the user explicitly approves.**
- Acceptable approvals: "yes", "lgtm", "approved", "go ahead", "ship it", "commit",
  or similar affirmative.
- If the user requests changes to the message, revise and present again.
- If the user wants to abort, unstage everything with `git reset HEAD` and stop.

## Step 7: Commit

- Run `git commit -m "<approved message>"` for each approved commit.
- After committing, run `git log --oneline -n <number_of_commits>` to confirm and
  show the user the result.
- Do **not** push. If the user wants to push, they will ask.
- Do **not** tag. Tags are created on `main` by `/release` after the promotion PR
  merges (D033 in the dotFiles decision log).

---

## Guardrails

- the forgejo token can be obtained with ‘beskar misc get DF_FORGEJO_API_TOKEN’
- **Never** commit without the user's explicit go-ahead.
- **Never** use `git add .` or `git add -A`.
- **Never** amend, rebase, or force-push unless the user explicitly asks.
- **Never** bump the version or tag — `/release` owns both.
- **PR process for `main`** — if developing on `next` or another branch, remind the
  user that changes reach `main` only through a promotion PR. Never push `main`.
- If anything looks off (merge conflicts, diverged branches, detached HEAD), warn the
  user and pause. **Exception:** a branch that is merely "N commits behind" with an
  empty `git diff` is content-neutral merge topology (a Forgejo "Update Branch"
  artifact — see `/release`), not damage: verify with `git diff --quiet`, then
  `git merge --ff-only`.
- If the repo has a `.commitlintrc`, `.czrc`, or similar config, respect those
  conventions over the defaults above.