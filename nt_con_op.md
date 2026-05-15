# Git Open Source Contribution Workflow

> How to contribute to an open source project the right way —
> from forking to getting your Pull Request merged.

---

## Table of Contents

- [The Big Picture](#the-big-picture)
- [Step 1 — Fork the Original Repo](#step-1--fork-the-original-repo)
- [Step 2 — Clone to Your PC](#step-2--clone-to-your-pc)
- [Step 3 — Add Upstream Remote](#step-3--add-upstream-remote)
- [Step 4 — Pull Latest from Upstream](#step-4--pull-latest-from-upstream)
- [Step 5 — Create a Feature Branch & Make Changes](#step-5--create-a-feature-branch--make-changes)
- [Step 6 — Push to Your Remote Repo](#step-6--push-to-your-remote-repo)
- [Step 7 — Open a Pull Request](#step-7--open-a-pull-request)
- [Step 8 — Keeping Your Fork Up to Date](#step-8--keeping-your-fork-up-to-date)
- [Full Cheatsheet](#full-cheatsheet)

---

## The Big Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                        GITHUB (Remote)                          │
│                                                                 │
│  ┌─────────────────────┐          ┌────────────────────────┐   │
│  │   Original Repo     │  fork    │   Your Forked Repo     │   │
│  │  (e.g. facebook/    │ ──────►  │  (yourname/react)      │   │
│  │   react)            │          │                        │   │
│  │                     │◄──────── │                        │   │
│  │  upstream/main      │    PR    │  origin/main           │   │
│  └─────────────────────┘          └────────────┬───────────┘   │
│           ▲                                    │               │
└───────────┼────────────────────────────────────┼───────────────┘
            │ pull (upstream)                    │ push (origin)
            │                                    │
     ┌──────┴────────────────────────────────────▼──────┐
     │                   YOUR PC (Local)                 │
     │                                                   │
     │   clone → create branch → edit → commit → push   │
     └───────────────────────────────────────────────────┘
```

**Three actors in this workflow:**

```
upstream  =  the original open source repo  (you don't own this)
origin    =  your fork on GitHub            (you own this)
local     =  your cloned copy on your PC   (you own this)
```

---

## Step 1 — Fork the Original Repo

### What is a fork?

A fork is a **full copy of someone else's repo placed under your GitHub account**.
You can do whatever you want to it without affecting the original.

### Timeline

```
t1  Original repo exists on GitHub
    facebook/react  [main] ── c1 ── c2 ── c3
                                           ▲ HEAD

t2  You click "Fork" on GitHub

t3  GitHub creates an exact copy under your account
    yourname/react  [main] ── c1 ── c2 ── c3
                                           ▲ HEAD
    
    Both repos are identical at t3.
    From here they can diverge independently.
```

### How to do it

```
1. Go to the original repo on GitHub
   e.g. https://github.com/facebook/react

2. Click the "Fork" button (top right)

3. Choose your GitHub account as the destination

4. GitHub creates: https://github.com/yourname/react
```

> 💡 **Tip:** You only fork once per project. All future work goes through your fork.

> 💡 **Tip:** The fork button also shows how many people have forked that repo.
>  A high fork count = active community = good project to contribute to.

---

## Step 2 — Clone to Your PC

### What is cloning here?

You are cloning **your fork** (not the original) to your local machine.
This gives you a working copy you can edit.

### Timeline

```
t3  Your fork exists on GitHub
    yourname/react  ── c1 ── c2 ── c3
                                    ▲ origin/main

t4  You run git clone

t5  A full copy now exists on your PC
    local/react     ── c1 ── c2 ── c3
                                    ▲ HEAD (main)
    
    Git automatically sets "origin" = your GitHub fork URL
```

### Command

```bash
# Clone YOUR fork (not the original)
git clone https://github.com/yourname/react.git

# Move into the folder
cd react
```

### What you have now

```
Remotes configured automatically:
  origin  →  https://github.com/yourname/react.git   ✅ (set by git clone)
  
  upstream  →  (nothing yet — you need to add this manually)
```

> 💡 **Tip:** Clone with SSH instead of HTTPS if you have SSH keys set up.
> It avoids password prompts every time you push.
> ```bash
> git clone git@github.com:yourname/react.git
> ```

> ⚠️ **Caution:** Don't clone the original repo directly.
> If you do, you won't have push access to it (you're not a maintainer).
> Always clone your own fork.

---

## Step 3 — Add Upstream Remote

### Why do you need upstream?

Your fork is already connected to `origin` (your GitHub fork).
But you also need a connection to the **original repo** so you can pull in
new changes that other contributors merge while you're working.

Without upstream, your fork slowly gets out of date.

### Timeline

```
t5  Local repo has one remote:
    origin  →  yourname/react  (your fork)
    
t6  You add upstream:
    origin    →  yourname/react       (your fork)
    upstream  →  facebook/react       (original repo)
    
    Nothing downloaded yet — just a URL registered.
```

### Command

```bash
# Add the original repo as "upstream"
git remote add upstream https://github.com/facebook/react.git

# Verify both remotes are registered
git remote -v
```

### Expected output

```
origin    https://github.com/yourname/react.git (fetch)
origin    https://github.com/yourname/react.git (push)
upstream  https://github.com/facebook/react.git (fetch)
upstream  https://github.com/facebook/react.git (push)
```

> 💡 **Tip:** The name "upstream" is just a convention — you could name it anything.
> But everyone uses "upstream" so stick with it for clarity.

> 💡 **Tip:** You can check your remotes anytime with:
> ```bash
> git remote -v
> ```

---

## Step 4 — Pull Latest from Upstream

### Why pull before making changes?

The original repo may have received new commits since you forked.
If you start working on an outdated base, your PR will conflict with main.

Always sync first.

### Timeline

```
t3  You forked  →  c1 ── c2 ── c3
                                ▲ your local main

t6  Meanwhile, original repo got new commits:
    facebook/react  →  c1 ── c2 ── c3 ── c4 ── c5
                                               ▲ upstream/main

t7  You fetch and merge upstream:
    your local main  →  c1 ── c2 ── c3 ── c4 ── c5
                                               ▲ HEAD
    
    Now you're in sync with the original repo.
```

### Commands

```bash
# Fetch all new commits from upstream (doesn't change your files yet)
git fetch upstream

# Merge upstream's main into your local main
git merge upstream/main

# Or do both in one step:
git pull upstream main
```

### Also update your GitHub fork

```bash
# Push the updated main to your GitHub fork too
git push origin main
```

```
After this step:

  facebook/react  [upstream/main]  ── c1 ── c2 ── c3 ── c4 ── c5
                                                               ▲
  yourname/react  [origin/main]    ── c1 ── c2 ── c3 ── c4 ── c5
                                                               ▲
  local           [main]           ── c1 ── c2 ── c3 ── c4 ── c5
                                                               ▲
  All three are in sync ✅
```

> 💡 **Tip:** Use `git fetch upstream` + `git merge` instead of `git pull upstream main`
> when you want to inspect what's coming before merging.
> ```bash
> git fetch upstream
> git log upstream/main --oneline   # preview new commits
> git merge upstream/main           # then merge
> ```

> ⚠️ **Caution:** Don't make changes directly on your local `main` branch.
> Keep `main` clean and always in sync with upstream.
> Do all your work on a separate feature branch (next step).

---

## Step 5 — Create a Feature Branch & Make Changes

### Why a separate branch?

- Keeps `main` clean and always sync-able with upstream
- You can have multiple PRs in progress at the same time (one per branch)
- Easy to discard if the PR gets rejected

### Timeline

```
t7  You are on main, synced with upstream
    main  ── c1 ── c2 ── c3 ── c4 ── c5
                                      ▲ HEAD

t8  Create a new branch for your feature
    git checkout -b fix/button-typo

    main         ── c1 ── c2 ── c3 ── c4 ── c5
                                             \
    fix/button-typo                           ▲ HEAD (new branch, same point)

t9  You make changes and commit

    main         ── c1 ── c2 ── c3 ── c4 ── c5
                                             \
    fix/button-typo                           c6 ── c7
                                                     ▲ HEAD

t10 Meanwhile upstream may get more commits (c8, c9)
    You don't have those yet on your feature branch.
```

### Commands

```bash
# Create and switch to a new branch
git checkout -b fix/button-typo

# Make your changes in the code editor...

# Stage your changes
git add .

# Commit with a clear message
git commit -m "fix: correct typo in Button component label"
```

### Branch naming conventions

```
Type        Format                  Example
──────────────────────────────────────────────────────
Feature     feature/<name>          feature/dark-mode
Bug fix     fix/<name>              fix/login-crash
Docs        docs/<name>             docs/update-readme
Refactor    refactor/<name>         refactor/auth-module
Chore       chore/<name>            chore/update-deps
```

### Good commit message format

```
<type>: <short description>

fix: correct typo in Button label
feat: add dark mode toggle
docs: update contributing guide
refactor: extract auth logic into helper

Bad examples:
  "fixed stuff"
  "wip"
  "asdfgh"
  "update"
```

> 💡 **Tip:** One branch = one purpose.
> Don't mix a bug fix and a new feature in the same branch/PR.
> Maintainers want to review focused, small changes.

> 💡 **Tip:** Commit early and often. You can always squash later with interactive rebase.
> ```bash
> git rebase -i HEAD~3   # squash last 3 commits into one clean commit
> ```

> ⚠️ **Caution:** Never commit directly to `main`.
> If you do, syncing with upstream later becomes painful.

---

## Step 6 — Push to Your Remote Repo

### What you're doing

Pushing your feature branch from your local PC to your GitHub fork (`origin`).
The original repo is not touched at all yet.

### Timeline

```
t9  Local state:
    local  [fix/button-typo]  ── c1 ── ... ── c5 ── c6 ── c7
                                                            ▲ HEAD

t10  After push:
    yourname/react [origin/fix/button-typo]  ── c5 ── c6 ── c7
                                                              ▲
    
    facebook/react [upstream/main]           ── c5  (unchanged, doesn't know about your work)
```

### Commands

```bash
# Push your branch to your fork
git push origin fix/button-typo

# If it's your first push on this branch, set tracking:
git push -u origin fix/button-typo

# Future pushes on the same branch (tracking already set):
git push
```

### What "-u" does

```
git push -u origin fix/button-typo

  -u = --set-upstream

  Links your local branch  →  origin/fix/button-typo
  After this, plain "git push" knows where to go.
  You only need -u once per new branch.
```

> 💡 **Tip:** Push frequently, even if your PR isn't ready.
> Your work is backed up on GitHub and others can see your progress.
> Use "Draft PR" on GitHub if you don't want review yet.

> ⚠️ **Caution:** If you rebased your branch after already pushing,
> you'll need to force push:
> ```bash
> git push --force-with-lease origin fix/button-typo
> ```
> `--force-with-lease` is safer than `--force` — it fails if someone else pushed to your branch.

---

## Step 7 — Open a Pull Request

### What is a PR?

A Pull Request is a **request to the original repo's maintainers** to review
your changes and pull them into their codebase.

It is a GitHub UI concept, not a Git command.

### Timeline

```
t10  After your push, GitHub shows a banner:
     "fix/button-typo had recent pushes — Compare & pull request"

t11  You open the PR:
     base:  facebook/react  main        ← where you want your changes to go
     head:  yourname/react  fix/button-typo  ← your changes

t12  Maintainers review your code
     They may:
       - Approve and merge ✅
       - Request changes 🔁 (you edit, commit, push again — PR updates automatically)
       - Close without merging ❌

t13  If approved:
     facebook/react main ── c1 ── ... ── c5 ── c6 ── c7
                                                        ▲ your commit is now in the original!
```

### PR checklist before submitting

```
 ✅  Branch is up to date with upstream main
 ✅  Code follows the project's style guide (check CONTRIBUTING.md)
 ✅  Tests pass locally
 ✅  Commit messages are clean (squash WIPs if needed)
 ✅  PR title is clear and descriptive
 ✅  PR description explains WHAT and WHY (not just how)
 ✅  Related issue is linked (e.g. "Closes #142")
```

### Good PR description template

```markdown
## What does this PR do?
Fixes a typo in the Button component label ("Submitt" → "Submit").

## Why?
Reported in issue #142. Affects all users of the Button component.

## How to test?
1. Run the app
2. Navigate to any form with a Button
3. Confirm label reads "Submit"

Closes #142
```

### What happens when maintainers request changes

```
t12  Maintainer comments: "Please also update the snapshot test"

t13  You make more changes locally on the same branch:
     git add .
     git commit -m "fix: update snapshot test for Button"
     git push

t14  PR is automatically updated — no need to open a new PR
     The new commit appears in the PR thread
```

> 💡 **Tip:** Link the related GitHub issue in your PR.
> "Closes #142" auto-closes the issue when your PR is merged.

> 💡 **Tip:** Keep PRs small. A PR touching 5 files gets reviewed fast.
> A PR touching 50 files sits in the queue for weeks.

> ⚠️ **Caution:** If upstream/main gets new commits while your PR is open,
> you may need to rebase your branch before it can be merged.
> ```bash
> git fetch upstream
> git rebase upstream/main
> git push --force-with-lease origin fix/button-typo
> ```

> ⚠️ **Caution:** Don't close and reopen a PR after changes.
> Just push new commits to the same branch — the PR updates automatically.

---

## Step 8 — Keeping Your Fork Up to Date

### Why this matters

While you were working, the original repo kept receiving commits from other contributors.
Your fork on GitHub is now behind. You need to sync regularly.

### Timeline

```
t0  You forked:
    upstream  ── c1 ── c2 ── c3
    origin    ── c1 ── c2 ── c3   (same)

t5  Time passes. Other contributors merged work into upstream:
    upstream  ── c1 ── c2 ── c3 ── c4 ── c5 ── c6
    origin    ── c1 ── c2 ── c3                      ← 3 commits behind

t6  You sync:
    git fetch upstream
    git checkout main
    git merge upstream/main
    git push origin main

    upstream  ── c1 ── c2 ── c3 ── c4 ── c5 ── c6
    origin    ── c1 ── c2 ── c3 ── c4 ── c5 ── c6   ← in sync again ✅
    local     ── c1 ── c2 ── c3 ── c4 ── c5 ── c6   ← in sync again ✅
```

### Commands

```bash
# Fetch latest from upstream
git fetch upstream

# Switch to your local main
git checkout main

# Merge upstream changes into local main
git merge upstream/main

# Push updated main to your GitHub fork
git push origin main
```

### Also update your feature branch (if PR is still open)

```bash
# Switch to your feature branch
git checkout fix/button-typo

# Rebase your feature branch on top of updated main
git rebase main

# Force push to update your fork's feature branch
git push --force-with-lease origin fix/button-typo
```

> 💡 **Tip:** Sync your fork before starting any new contribution.
> Make it a habit: "before I code, I sync."

> 💡 **Tip:** GitHub now has a "Sync fork" button in the UI.
> It does the same thing as fetch + merge upstream, but only for the main branch.
> Still good to know the Git commands for feature branches.

---

## Full Cheatsheet

```
THE STORY IN ONE FLOW
──────────────────────────────────────────────────────────────────────

  [GitHub] facebook/react ──fork──► [GitHub] yourname/react
                                              │
                                           git clone
                                              │
                                              ▼
                                      [Local] your PC
                                              │
                              git remote add upstream (facebook/react)
                                              │
                              git pull upstream main  (sync)
                                              │
                              git checkout -b fix/button-typo
                                              │
                              (edit code, git add, git commit)
                                              │
                              git push -u origin fix/button-typo
                                              │
                                              ▼
                                [GitHub] yourname/react
                                     fix/button-typo branch
                                              │
                                        Open Pull Request
                                              │
                                              ▼
                                [GitHub] facebook/react ← PR reviewed & merged ✅
```

```bash
# ── STEP 1: FORK ───────────────────────────────────────────
# Done on GitHub UI — click "Fork"

# ── STEP 2: CLONE ──────────────────────────────────────────
git clone https://github.com/yourname/react.git
cd react

# ── STEP 3: ADD UPSTREAM ───────────────────────────────────
git remote add upstream https://github.com/facebook/react.git
git remote -v                              # verify both remotes

# ── STEP 4: SYNC WITH UPSTREAM ─────────────────────────────
git fetch upstream
git checkout main
git merge upstream/main
git push origin main                       # update your fork too

# ── STEP 5: CREATE BRANCH & MAKE CHANGES ───────────────────
git checkout -b fix/button-typo
# ... edit code ...
git add .
git commit -m "fix: correct typo in Button label"

# ── STEP 6: PUSH TO YOUR FORK ──────────────────────────────
git push -u origin fix/button-typo

# ── STEP 7: OPEN PULL REQUEST ──────────────────────────────
# Done on GitHub UI
# base: facebook/react main
# head: yourname/react fix/button-typo

# ── STEP 8: KEEP FORK UPDATED (repeat anytime) ─────────────
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Update feature branch if PR still open:
git checkout fix/button-typo
git rebase main
git push --force-with-lease origin fix/button-typo
```

---

## Key Terms Summary

```
Term          │ Meaning
──────────────┼──────────────────────────────────────────────────────
fork          │ Your copy of someone else's repo on GitHub
origin        │ Your fork's URL (set automatically when you clone)
upstream      │ The original repo's URL (you add this manually)
clone         │ Download a repo to your local PC
PR            │ Pull Request — ask maintainers to accept your changes
base branch   │ Where your changes go (upstream main)
head branch   │ Where your changes come from (your fork's feature branch)
fetch         │ Download remote changes without applying them
merge         │ Apply downloaded changes to your branch
rebase        │ Replay your commits on top of an updated base
```

---

*Notes by KunTsuKe | Last updated May 2026*