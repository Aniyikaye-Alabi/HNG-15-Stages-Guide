# How to Contribute to Zedu (zedu-hng `dev`) — Team Guide

This guide shows two ways to get your change into the main Zedu project:

- **Way 1: The Crude Method.** Use git commands and GitHub directly. For people who are comfortable in a terminal.
- **Way 2: The AI Method.** Use an AI agent (Claude Code, Cursor, Copilot, etc.) and good prompts to do everything. For people who don't want to type git commands.

Both ways end the same: **one clean Pull Request (PR) into `zedu-hng/zedu-fe`, branch `dev`.**

Everything here comes from a real example: **PR #43, "feat: add contributors page"**
https://github.com/zedu-hng/zedu-fe/pull/43
That PR first had messy history, the wrong route folder and a wrong branch name. We fixed all three. This guide is built from what we learned, so you can skip those mistakes.

---

## IMPORTANT: Pull from `zedu-hng` dev regularly

> **Always pull the latest `zedu-hng` `dev` before you start work, and again from time to time while you work.**
> Other teams merge new code into `dev` every day. If you work on old code, your PR will clash with theirs and the reviewers will send you back.
> Do this by the git commands (Section 3 to start, Section 8 to update, never by force-push) or by prompting your AI agent (Prompts 3 and 7 in Way 2).

Why: in PR #43 a reviewer asked **"Did you pull from dev?"** Other teams had already created their own route folders. Our page was in the wrong place because we were on old code.

---

## Part A: What everyone must know first

### A1. The three places the code lives

| Nickname | Address | What it is |
|----------|---------|-----------|
| `upstream` | `github.com/zedu-hng/zedu-fe` | The **main project**. All PRs go here, into branch `dev`. You cannot push to it directly. |
| `origin` | `github.com/zedu-condor/zedu-fe` | **Our team's copy** (a "fork"). You clone this and push your branches here. |
| (your own) | `github.com/<your-username>/zedu-fe` | Your **personal fork**. Use it only if you don't have write access to `zedu-condor`. |

```
Your computer --push--> zedu-condor (our fork) --Pull Request--> zedu-hng (main project, branch dev)
```

### A2. Why we clone from `zedu-condor` `staging`

Our team's `staging` branch is kept **equal to `zedu-hng`'s `dev`**. So cloning `staging` gives you the same code as the main project. (Part C explains how the maintainer keeps it in sync, and Sections 3 and 8 show how you stay up to date yourself. To be safe, always start your own branch from `upstream/dev`, not from `staging`.)

### A3. The rules reviewers check

| Rule | What it means |
|------|---------------|
| **Branch name format** | `type/ticketnumber-short-description`, for example `feat/task-2-zedu-condor-contributors-page`. See A4. |
| **One ticket, one PR** | Each PR is for one ticket and one person. No combined PRs. |
| **Clean history** | Ideally ONE commit on top of the latest `dev`. No merge commits. Get it clean BEFORE your first push. |
| **No force-push, ever** | Never use `git push --force` or `--force-with-lease`. Once pushed, a branch only moves forward. Need a redo? Make a fresh `-v2` branch and a new PR (Section 8). |
| **Commit message format** | `type(team-name): description`, for example `feat(condor): add contributors page`. Types: `feat`, `fix`, `docs`, `refactor`, `chore`, `test`, `perf`, `security`. |
| **Only the files for your ticket** | No drive-by changes. |
| **Your own folder** | Contributor pages go in your team's own route folder: `src/app/(homepage)/contributors/<team-name>/`. Other teams did this (heron, sparrow, zedu-kestrel, zedu-osprey). |
| **Protected files: do not change** | `.github/`, `AGENTS.md`, `CONTRIBUTING.md`, tooling config. A PR touching them fails unless a reviewer adds the `config-change-approved` label. |
| **Never commit secrets** | No `.env` files, keys, or credentials. |
| **Use `pnpm`** | Never `npm` or `yarn`. |

### A4. Branch name rule (this exact check failed on PR #43)

The automatic check:

```
^(feat|fix|test|docs|refactor|chore|perf|security)/([A-Za-z]+-)?[[:digit:]]+-[a-z[:digit:]][a-z[:digit:]-]*$
```

In plain English:

```
feat/task-2-zedu-condor-contributors-page
 |     |  | |
 |     |  | +-- short description (lowercase, numbers, dashes)
 |     |  +---- ticket number (digits)
 |     +------- optional ONE word + dash (like "task-")
 +------------- type
```

- GOOD: `feat/task-2-zedu-condor-contributors-page`, `fix/245-login-redirect`
- BAD: `feat/stage-task-2-...` (two words before the number), `Feat/My-Page` (capitals), `my-branch` (no type)

Test your name before using it:

```bash
echo "feat/task-2-my-change" | grep -E '^(feat|fix|test|docs|refactor|chore|perf|security)/([A-Za-z]+-)?[[:digit:]]+-[a-z[:digit:]][a-z[:digit:]-]*$' && echo MATCH
```

---

# Way 1: The Crude Method (git + GitHub)

## Section 1: One-time setup

You need: a GitHub account, **Git**, **Node.js 20 or newer**, and **pnpm** (`npm install -g pnpm`). Ask the team lead to give your GitHub account **write access** to `zedu-condor/zedu-fe` (accept the email invitation). Without it, `git push` fails with a 403 error. In that case use your own fork (Section 9).

## Section 2: Clone our team's `staging`

```bash
git clone -b staging https://github.com/zedu-condor/zedu-fe.git
cd zedu-fe
```

- `-b staging` means "download the staging branch".
- Add the main project as a second remote called `upstream`:

```bash
git remote add upstream https://github.com/zedu-hng/zedu-fe.git
git remote -v        # you should see origin (zedu-condor) and upstream (zedu-hng)
```

Install and configure:

```bash
pnpm install
cp env.example .env      # Windows PowerShell: Copy-Item env.example .env
```

Fill `.env` with your team's values (ask your team lead). **Never commit `.env`**. It is already ignored by git.

Run the app to check your setup:

```bash
pnpm dev                 # opens at http://localhost:3000
```

## Section 3: Get the latest `dev` and make your branch (EVERY TIME you start a ticket)

```bash
git fetch upstream                                   # download latest from zedu-hng (changes nothing yet)
git switch -c feat/task-2-your-change upstream/dev   # new branch, starting from the latest zedu-hng dev
```

Why `upstream/dev` and not `staging`: it is the source of truth, always the freshest. This avoids the "did you pull from dev?" review comment.

## Section 4: Make your change

1. Find where other teams put their work, so you match the pattern:
   ```bash
   git ls-tree -r --name-only upstream/dev | grep -i contributors
   ```
2. Add your files **only inside your own folder**, for example
   `src/app/(homepage)/contributors/zedu-condor/page.tsx`
   with data in `.../zedu-condor/_lib/contributors.ts`.
3. Preview it at `http://localhost:3000/contributors/zedu-condor` (run `pnpm dev`).
4. Run the checks the PR will run:
   ```bash
   pnpm check-format     # Prettier (fix with: pnpm format)
   pnpm check-lint       # ESLint   (fix with: pnpm lint:fix)
   pnpm check-types      # TypeScript
   pnpm build            # full production build
   ```

## Section 5: Commit with only your files

```bash
git status --short                                        # see what changed
git add "src/app/(homepage)/contributors/zedu-condor"     # add ONLY your files, never "git add ."
git commit -m "feat(contributors): add Zedu-Condor team contributors page" -m "Ticket: task-2"
```

- Do **not** add `AGENTS.md` even if it shows as modified. The dev server edits it by itself.
- Windows may show `LF will be replaced by CRLF`. That is a harmless warning.

## Section 6: Push your branch

```bash
git push -u origin feat/task-2-your-change
```

If you see `403 Permission denied`: you don't have write access yet. Ask the team lead for it, or use your own fork (Section 9).

## Section 7: Open the Pull Request

GitHub prints a link after the push. Or open this one yourself (replace the branch name):

```
https://github.com/zedu-hng/zedu-fe/compare/dev...zedu-condor:feat/task-2-your-change?expand=1
```

Settings that matter:

- **Base repository:** `zedu-hng/zedu-fe`
- **Base branch:** `dev`
- **Head:** `zedu-condor:feat/task-2-your-change`
- **PR title:** Conventional Commits format, for example `feat: add contributors page` or prefarably `feat(condor): add contributors page`. The title is checked too, and becomes the final commit message.

Fill in the description template. This is the one we used for PR #43 (copy and edit):

```
## Ticket
- **Ticket ID:** task-2
- **Ticket title:** Contributors page

## Team lead
@<team lead's GitHub handle>

## What changed
Added the Zedu-Condor team contributors page at /contributors/zedu-condor, in its own route folder like the other teams. Lists 10 contributors with initials, name, handle and role, plus a call to action. Rebuilt on the latest dev as one commit.

## Why
To show the people on the Zedu-Condor team on a public contributors page, without clashing with other teams' routes.

## How to test
1. Pull the branch and run `pnpm dev`.
2. Open http://localhost:3000/contributors/zedu-condor.
3. Check each card shows initials, full name, handle and role.
4. Shrink the window to phone width and confirm the cards stack in one column.
5. Open http://localhost:3000/contributors/heron and confirm other teams' pages still load.

## What to expect
A page headed "The People Behind Zedu" with 10 contributor cards.

## Test evidence
- Tested against: local setup (page is static, no API calls)
- Tests: none added (static page, no logic). Prettier, ESLint and type check pass.

## Mandatory checks
- [x] Atomic: one ticket, 2 files, about 140 lines
- [x] Feature flag: N/A, small static page
- [x] Database / API contract: no changes
- [x] Preview: I verified the change in the fork build
- [x] Protected files: not changed

## Screenshots / recording
<add a screenshot>

## AI usage
<one line on how AI was used, if it was>
```

After you open it:

1. Ask your team lead to review and **Approve** first.
2. The **fork build** may not start by itself. In `zedu-condor/zedu-fe` go to **Actions, then PR build, then Run workflow** on your branch, or comment `/fork-build` on the PR.
3. Read the bot comments (PR review bot, CodeRabbit) and fix what they flag.

## Section 8: After the PR is open

### THE RULE: never force-push (no `--force`, no `--force-with-lease`)

We were told not to force-push. Force-pushing rewrites history that is already on GitHub. It can wipe out other people's work and confuse reviewers. So **once a branch is pushed, it only ever moves forward: you add commits, you never rewrite them.**

Clean history is something you do **before** your first push, or by starting a **fresh branch** (see below). It is never done by force.

> Note: an earlier version of this guide told you to use `git rebase` followed by `git push --force-with-lease`. That was wrong for our rules. Ignore it.

### Make your history clean BEFORE the first push (this is safe)

Before you push for the first time, nothing is on GitHub yet, so you may tidy your local commits freely. No force is needed, because there is nothing to overwrite.

```bash
git fetch upstream
git rebase upstream/dev                 # only while the branch is NOT pushed yet
# several local commits and want one? squash them:
git reset --soft upstream/dev           # keeps ALL your file changes, removes the commit history
git commit -m "feat: your clean message" -m "Ticket: task-2"
git push -u origin feat/task-2-your-change     # first push, a normal push
```

### After the PR is open: how to make changes WITHOUT force

**1. Reviewer asked for a fix (the normal case).** Just add a new commit and push normally:

```bash
git add "<the file you fixed>"
git commit -m "fix: use link semantics for contributors CTA" -m "Ticket: task-2"
git push                                 # a normal push, no force
```

Reviewers **squash-merge** PRs, which means all your commits become ONE commit on `dev` anyway. Extra small fix commits are fine.

**2. Do I have to update my branch with the latest `dev`?** Usually **no**. If GitHub shows "This branch has no conflicts with the base branch", leave it alone. Reviewers merge it as it is. Do not press **Update branch** and do not run `git merge staging` or `git merge dev`. Those add merge commits, which is exactly what the reviewer complained about in PR #43 ("that commit history is giving PTSD").

**3. GitHub says there are conflicts, or the reviewer says your branch is out of date or has messy history.** Use the **fresh branch method**. It needs no force, because it makes a NEW branch:

```bash
git fetch upstream
git switch -c feat/task-2-your-change-v2 upstream/dev       # new branch from the newest dev
git checkout feat/task-2-your-change -- "<your folder>"     # copy ONLY your files from the old branch
git add "<your folder>"
git commit -m "feat: your clean message" -m "Ticket: task-2"
git push -u origin feat/task-2-your-change-v2               # a normal push of a new branch
```

Then:

1. Open a **new PR** from the `-v2` branch into `zedu-hng` `dev` (Section 7). Use the same title and description.
2. On the old PR, comment: "Replaced by #<new PR number>, rebuilt on the latest dev" and **close** it.
3. Optional tidy-up: delete the old branch with `git push origin --delete feat/task-2-your-change`. This deletes a whole branch, it does not rewrite history, so it is not a force-push.

Check that `-v2` still matches the branch name rule: `feat/task-2-your-change-v2` does (letters, numbers and dashes after the ticket number).

Only the **same ticket number** matters. Keep `task-2` in the name.

> Why this works: the old branch is left untouched, and a new branch is just a normal push. Nothing is overwritten, so nothing can be lost.

**Real example:** PR #43 started messy (8 commits, 2 merge commits, wrong folder). We cleaned it up by rewriting the branch with a force-push. Under the "no force-push" rule, the right way would have been to open a new `-v2` branch and a new PR, as shown above.

### Undoing a file to how it was

```bash
git checkout upstream/dev -- path/to/file        # restore one file to match dev
```

### Common problems

| Problem | Cause | Fix |
|---------|-------|-----|
| `403 Permission denied` on push | No write access to `zedu-condor` | Get access from the team lead, or use your own fork (Section 9) |
| "Branch doesn't match `<type>/<ticket-id>-<short-description>`" | Bad branch name | Rename: `git branch -m old new`, push the new name, delete the old (`git push origin --delete old`) |
| PR shows files you didn't change | Wrong base, or you merged `staging` into your branch | Rebuild the branch from `upstream/dev` (Section 3) and copy your files in |
| `git push` says "rejected" or "non-fast-forward" or "fetch first" | The remote branch has commits you don't have (someone else pushed, or GitHub added a merge) | **Do not force.** Run `git fetch origin`, look with `git log origin/<branch>`, then `git pull --no-rebase origin <branch>` to bring them in, and push normally. If it is a mess, use the fresh `-v2` branch method (Section 8) |
| `AGENTS.md` shows as modified | `pnpm dev` regenerates it | Don't add it. To drop it: `git checkout AGENTS.md` |
| Reviewer: "did you pull from dev?" | Your branch started from old code | Use the fresh `-v2` branch method (Section 8). Start from `upstream/dev` and copy your files in. No force needed |
| PR "Protected files" check fails | You changed `CONTRIBUTING.md`, `.github/`, `AGENTS.md` or tooling config | Revert those files: `git checkout upstream/dev -- <file>` |

## Section 9: If you don't have write access to `zedu-condor` (use your own fork)

1. On GitHub open `https://github.com/zedu-condor/zedu-fe` and click **Fork**, then choose your account.
2. Add it as a remote and push there:
   ```bash
   git remote add fork https://github.com/<your-username>/zedu-fe.git
   git push -u fork feat/task-2-your-change
   ```
3. Open the PR with this link (replace the names):
   ```
   https://github.com/zedu-hng/zedu-fe/compare/dev...<your-username>:feat/task-2-your-change?expand=1
   ```

---

# Way 2: The AI Method (prompts only, no git commands to type)

For people who prefer not to use git by hand. You need an **AI coding agent that can run commands on your computer**, such as Claude Code, Cursor (agent mode) or GitHub Copilot agent. A plain chat AI (the website ChatGPT, for example) can only explain. It can't run the steps for you.

### How to use this

1. Open the agent **in an empty folder** where you want the project.
2. Paste **Prompt 0** first (the rules). Then paste the other prompts **one at a time**, in order. Wait for each to finish and **read what the AI says** before moving on.
3. Replace the `<<...>>` parts with your own details.
4. If the AI asks permission to run a command, read it. Say **no** if it mentions **any kind of force** (`--force`, `-f` or `--force-with-lease`), `git add .`, deleting things, `.env` or `AGENTS.md`.

### Prompt 0: Give the AI the rules (always first)

```
You are helping me contribute to the Zedu frontend. Follow these rules for the whole session:

REPOS
- Main project (upstream): https://github.com/zedu-hng/zedu-fe  -- all PRs go into branch "dev"
- Our team fork (origin): https://github.com/zedu-condor/zedu-fe
- Always start new work from the latest upstream dev, never from old code.

RULES
1. Branch names must match: ^(feat|fix|test|docs|refactor|chore|perf|security)/([A-Za-z]+-)?[0-9]+-[a-z0-9][a-z0-9-]*$
   Example: feat/task-2-zedu-condor-contributors-page. Test the name before using it.
2. One ticket per branch. Keep the PR to ONE clean commit on top of the latest upstream/dev. No merge commits.
3. Never merge dev or staging into my branch. Only rebase onto upstream/dev while my branch has NOT been pushed yet. After it is pushed, never rewrite it: to update it, make a fresh branch from upstream/dev (name ending in -v2), copy my files over, and push it as a new branch.
4. Commit messages use Conventional Commits: "type: description", with the ticket as a trailer line "Ticket: <id>".
5. Stage files by exact name. NEVER run "git add ." or "git add -A".
6. NEVER change or commit: .github/, AGENTS.md, CONTRIBUTING.md, tooling config, .env files, keys or secrets.
7. NEVER force-push. That means never "git push --force", never "-f", and never "--force-with-lease", on any branch, even if I ask casually. Only normal "git push". If a push is rejected, stop, explain why in plain English, and offer the safe options. Never delete or rewrite remote history.
8. Use pnpm, never npm or yarn.
9. Put contributor pages only in my team's own folder: src/app/(homepage)/contributors/<team-name>/
10. Before every git command that changes things, tell me in plain English what it will do.
11. Always show me "git status" and the list of files changed before each commit.
12. If you are unsure, stop and ask me. Do not guess.

Reply "Understood" and summarise the rules in 3 lines.
```

### Prompt 1: Download the project

```
Clone https://github.com/zedu-condor/zedu-fe (branch "staging") into a folder called zedu-fe.
Then add a second remote named "upstream" pointing to https://github.com/zedu-hng/zedu-fe.git.
Run "git remote -v" and show me the result. Then install dependencies with pnpm and copy env.example to .env.
Do not fill in or commit any secrets. Tell me which values in .env I need to get from my team lead.
```

### Prompt 2: Run the app to check it works

```
Start the development server with "pnpm dev" and tell me the URL.
If there are errors, explain them in simple English and suggest the fix before changing anything.
```

### Prompt 3: Create my branch from the latest dev (always do this before starting a ticket)

```
Run "git fetch upstream".
Then create a new branch from the latest upstream/dev called <<feat/task-2-short-description>>.
Check the branch name against the rule from my rules message first and tell me if it fails.
Show me "git status" and "git log --oneline -3" afterwards.
```

### Prompt 4: Make my change (describe what you want)

```
My ticket is <<ticket id and title>>.
Look at how other teams did the same kind of work first: list the files under src/app/(homepage)/contributors on upstream/dev and tell me the pattern they followed.
Then, for my team (<<team name>>), create my page inside its own folder: src/app/(homepage)/contributors/<<team-name>>/page.tsx, with the data in a _lib/contributors.ts file in the same folder.
My team members are:
<<Full name | handle | role>>
<<Full name | handle | role>>
Match the style of the existing pages. Use a Next.js <Link> for any button that goes to another page (never a <button> inside a <Link>). Do not touch any file outside my folder.
When done, show me the list of files you created or changed.
```

### Prompt 5: Check everything works, then commit and push

```
Run these checks and show me the result of each: pnpm check-format, pnpm check-lint, pnpm check-types, pnpm build.
If format fails, run "pnpm format" on only my files and re-check.
Then show me "git status".
Add ONLY the files in my folder by exact path (not "git add ."), make ONE commit with this message:
  <<feat(contributors): add <<Team>> contributors page>>
with a second line: "Ticket: <<task-2>>".
Show me "git log --oneline -3" and "git diff upstream/dev --stat" so I can confirm only my files are in it.
Do not commit AGENTS.md or .env.
If everything looks right, push the branch with: git push -u origin <my branch name>.
If the push says 403 Permission denied, stop and tell me. Do not try other ways around it.
```

### Prompt 6: Raise the Pull Request

```
I need to open a Pull Request from zedu-condor/zedu-fe branch <<my branch>> into zedu-hng/zedu-fe branch "dev".
If you have the GitHub CLI (gh) logged in, create it with the title "<<feat: add <<Team>> contributors page>>".
If you cannot create it yourself, give me:
 1. the link: https://github.com/zedu-hng/zedu-fe/compare/dev...zedu-condor:<<my branch>>?expand=1
 2. a fully filled-in PR description using the Zedu PR template (.github/pull_request_template.md in the repo), with these headings: Ticket, Team lead, What changed, Why, How to test, What to expect, Test evidence, Mandatory checks, Screenshots, AI usage.
Fill in everything you can from my changes. Leave a clear <placeholder> where you need info from me (ticket ID, team lead handle, screenshot).
The "How to test" must use the real URL of my page.
```

### Prompt 7: Stay in sync with `zedu-hng` dev (use regularly!)

```
Bring my work up to date with the latest zedu-hng dev, WITHOUT creating merge commits and WITHOUT any force-push.
1. Run "git fetch upstream".
2. Tell me how many commits my branch is behind upstream/dev, and whether my branch has already been pushed (check with "git ls-remote origin <my branch>").
3. If my branch has NOT been pushed yet: run "git rebase upstream/dev". If there are conflicts, show me each conflicting file and explain in simple English what each side changed. Ask me before choosing. Then run the checks (pnpm check-format, check-lint, check-types) and show me.
4. If my branch HAS been pushed: do NOT rebase and do NOT force-push. Instead use the fresh branch method:
   a. create a new branch "<my branch>-v2" from upstream/dev;
   b. copy ONLY my own folder from the old branch ("git checkout <old branch> -- <my folder>");
   c. run the checks, make ONE commit (message: <<type: description>>, trailer "Ticket: <<id>>"), and push the new branch normally with "git push -u origin <my branch>-v2";
   d. give me the PR link for the new branch, and a short comment I can post on the old PR saying it is replaced by the new one.
5. Before any push, tell me exactly what it will do and wait for my OK.
```

### Prompt 8: A reviewer asked me to clean up my history

```
A reviewer says my branch history is messy. Fix it WITHOUT any force-push, using a fresh branch:
1. Run "git fetch upstream".
2. Create a new branch "<my branch>-v2" from upstream/dev.
3. Copy ONLY my own folder from my old branch: "git checkout <my branch> -- <my folder>".
4. Show me "git diff upstream/dev --stat" so I can see only my files are included.
5. Make ONE commit with the message "<<type: description>>" and the trailer "Ticket: <<id>>", then push the new branch normally ("git push -u origin <my branch>-v2").
6. Give me the PR link for the new branch and a short comment to post on the old PR saying it is replaced and can be closed.
Do not touch the old branch. Do not use any force option.
```

### Prompt 9: A reviewer left comments

```
Here are the review comments on my PR:
<<paste the comments>>
Explain each one in simple English. Then make the fixes (only in my files), run the checks, commit as a NEW small commit (never amend or rewrite a commit that is already pushed), and show me the diff before pushing with a normal "git push".
```

### Prompt 10: When something goes wrong

```
Something went wrong. Here is the exact error message:
<<paste the error>>
Do NOT run any fix yet. First explain in simple English what the error means and what caused it.
Then give me 2 safe options with the risks of each, tell me which you recommend, and wait for me to choose.
```

### Tips for good prompting

- **Be specific.** Give the ticket ID, team name, branch name and who is in the list.
- **One step at a time.** Do not paste all prompts at once.
- **Make the AI explain.** Add "explain in simple English before running" whenever you are unsure.
- **Always verify.** After any push, open the PR's **"Files changed"** tab on GitHub. It should show only your files. (In PR #43 we found `CONTRIBUTING.md` in the PR this way.)
- **Never let the AI** touch `.github/`, `AGENTS.md`, `CONTRIBUTING.md`, `.env`, or run any force-push (`--force`, `-f`, `--force-with-lease`) or `git add .`.

---

# Part C: Keeping `staging` in sync (for the team lead / maintainer)

Our `staging` is meant to equal `zedu-hng` `dev`. It drifts as other teams merge. To re-sync it:

**Safe check first (changes nothing):**

```bash
git fetch origin
git fetch upstream
echo "behind by: $(git rev-list --count origin/staging..upstream/dev)"
echo "ahead by:  $(git rev-list --count upstream/dev..origin/staging)"
```

- If **ahead is 0** the sync is a normal fast-forward, with no force needed:
  ```bash
  git push origin upstream/dev:refs/heads/staging
  ```
- If **ahead is more than 0**, `staging` has commits that `dev` doesn't (for example old PR merges). **Do not force-push `staging`.** Use the safe way, which is a merge through a PR on our fork:
  ```bash
  git fetch origin
  git fetch upstream
  git switch -c sync/staging-with-dev origin/staging     # a working branch, so staging itself is not touched
  git merge upstream/dev                                  # brings the newest zedu-hng dev in
  # if there are conflicts, resolve them (accept dev's version of the files), then:
  git add <resolved files>
  git commit                                              # finishes the merge
  git push -u origin sync/staging-with-dev                # a normal push of a new branch
  ```
  Then on GitHub open a PR in **`zedu-condor/zedu-fe`** with base `staging` and head `sync/staging-with-dev`, get it approved, and merge it. Everything is a normal push. Nothing is overwritten.
  The cost: `staging` keeps its extra old commits plus one merge commit, so its history is not as tidy. That is fine, because **contributors never branch from `staging`**: they branch from `upstream/dev` (Section 3), so `staging`'s history never reaches a PR.
  Check afterwards that `dev` is fully inside `staging`: `git rev-list --count origin/staging..upstream/dev` should print `0`.

**Our history on this:** on 2026-10-04 `staging` had 9 commits that `dev` lacked and was 13 behind. At that time it was reset to `dev` (`cb68e93`) using a force-push, and the old state was saved as `backup/staging-2026-10-04`. Under the "no force-push" rule, do **not** repeat that. Use the merge-through-a-PR method above. The backup branch can stay or be deleted by the maintainer.

**Rule to avoid drift:** do not commit or merge anything into our `staging` except these sync merges. Send all real work to `zedu-hng` `dev` through PRs. If `staging` never gets its own commits, the "ahead" number stays `0` and syncing is always the easy fast-forward.

---

# Quick Cheat Sheet

**Way 1 (commands), the whole flow:**

```bash
git clone -b staging https://github.com/zedu-condor/zedu-fe.git && cd zedu-fe
git remote add upstream https://github.com/zedu-hng/zedu-fe.git
pnpm install && cp env.example .env

git fetch upstream
git switch -c feat/task-N-short-description upstream/dev

# ...make your change in your own folder, then:
pnpm check-format && pnpm check-lint && pnpm check-types && pnpm build
git add "<your folder>"
git commit -m "feat: your message" -m "Ticket: task-N"
git push -u origin feat/task-N-short-description

# open: https://github.com/zedu-hng/zedu-fe/compare/dev...zedu-condor:feat/task-N-short-description?expand=1
```

**Stay up to date (no force):** `git fetch upstream`. Not pushed yet? `git rebase upstream/dev`. Already pushed? Don't rewrite it: make a fresh branch `git switch -c <branch>-v2 upstream/dev`, copy your folder over with `git checkout <old-branch> -- "<your folder>"`, commit once, push normally, and open a new PR (Section 8).

**Way 2 (AI):** Prompt 0 (rules), then 1 (clone), 2 (run), 3 (branch), 4 (change), 5 (check, commit, push), 6 (PR), then 7 (sync) from time to time.

**Golden rules:** fresh `dev` first. One ticket, one clean commit. Right branch name. Only your own folder. Never touch protected files. **Never force-push (no `--force`, no `--force-with-lease`).** Check "Files changed" before asking for review.
