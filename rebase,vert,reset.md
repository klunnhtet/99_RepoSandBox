# Git Reset, Revert & Rebase — Deep Dive Notes

> Personal reference notes with diagrams, timelines, tips, and cautions.

---

## Table of Contents

- [git reset](#git-reset)
- [git revert](#git-revert)
- [git rebase](#git-rebase)
- [Quick Comparison](#quick-comparison)
- [When to Use What](#when-to-use-what)

---

## git reset

### What it does

Moves the `HEAD` pointer (and optionally your staging area and working files) **backwards** to a previous commit. It **rewrites history** — the commits after the target are gone from the branch.

### Timeline — Before Reset

```
t1 ──── t2 ──── t3 ──── t4
                         ▲
                        HEAD (main)
```

### Timeline — After `git reset t2`

```
t1 ──── t2
         ▲
        HEAD (main)

t3 and t4 are now unreachable (not deleted from disk yet, but gone from branch)
```

---

### The Three Modes

```
Working Directory   Staging Area (Index)   Commit History
──────────────────────────────────────────────────────────
      KEPT                KEPT              REWOUND        ← --soft
      KEPT               CLEARED            REWOUND        ← --mixed  (default)
     CLEARED             CLEARED            REWOUND        ← --hard
```

---

### Mode 1: `--soft`

```bash
git reset --soft <commit>
```

**What changes:**

- HEAD moves back ✅
- Staging area: unchanged (your changes stay staged)
- Working files: unchanged

**Timeline:**

```
Before:
  t1 ──── t2 ──── t3
                   ▲ HEAD

After git reset --soft t2:
  t1 ──── t2
           ▲ HEAD

  ┌──────────────────────────────┐
  │ Staging Area                 │
  │  + changes from t3 (staged)  │  ← still here, ready to re-commit
  └──────────────────────────────┘
```

**Use case:** Made a bad commit message, or want to squash the last N commits into one.

```bash
# Example: undo last commit but keep changes staged
git reset --soft HEAD~1
git commit -m "Better commit message"
```

---

### Mode 2: `--mixed` (default)

```bash
git reset <commit>
# same as
git reset --mixed <commit>
```

**What changes:**

- HEAD moves back ✅
- Staging area: cleared ✅
- Working files: unchanged

**Timeline:**

```
Before:
  t1 ──── t2 ──── t3
                   ▲ HEAD

After git reset --mixed t2:
  t1 ──── t2
           ▲ HEAD

  ┌──────────────────────────────┐
  │ Working Directory            │
  │  + changes from t3 (unstaged)│  ← still here, but NOT staged
  └──────────────────────────────┘

  Staging Area: empty
```

**Use case:** You committed too early and want to re-stage selectively.

```bash
# Undo last commit, unstage everything
git reset HEAD~1

# Now cherry-pick what to stage
git add src/feature.js
git commit -m "Focused commit"
```

---

### Mode 3: `--hard`

```bash
git reset --hard <commit>
```

**What changes:**

- HEAD moves back ✅
- Staging area: cleared ✅
- Working files: **deleted / reverted** ✅

**Timeline:**

```
Before:
  t1 ──── t2 ──── t3
                   ▲ HEAD

After git reset --hard t2:
  t1 ──── t2
           ▲ HEAD

  Working Directory = exactly as it was at t2
  All changes from t3 are GONE from disk
```

> ⚠️ **CAUTION: --hard is destructive.**
> Uncommitted changes in your working directory are permanently deleted.
> There is no undo unless you have a backup or use `git reflog`.

---

### Useful Reset Shortcuts

```bash
git reset HEAD~1        # go back 1 commit (mixed)
git reset HEAD~3        # go back 3 commits (mixed)
git reset <hash>        # go back to a specific commit
git reset --hard HEAD   # throw away all uncommitted changes (nuclear clean)
```

---

### 💡 Tips

> **Tip 1:** `git reflog` is your safety net after a hard reset.
> Even after `--hard`, Git keeps commits in reflog for ~30 days.
>
> ```bash
> git reflog
> git reset --hard HEAD@{2}   # recover to 2 steps ago
> ```

> **Tip 2:** Use `HEAD~N` notation instead of copy-pasting hashes.
> `HEAD~1` = 1 commit before HEAD. `HEAD~3` = 3 commits before HEAD.

> **Tip 3:** `git reset HEAD <file>` unstages a single file without touching anything else.

---

### ⚠️ Cautions

> **Caution 1:** Never reset commits that have already been pushed to a shared remote.
> Other people's local branches are based on those commits.
> If you reset and force push, their history breaks.

> **Caution 2:** `--hard` deletes uncommitted work permanently.
> Always `git stash` or `git commit` first if you have work you care about.

---

---

## git revert

### What it does

Creates a **new commit** that undoes the changes from a previous commit. The original commit stays in history — nothing is rewritten.

This is the **safe** alternative to reset for shared/remote branches.

### Timeline — Before Revert

```
t1 ──── t2 ──── t3 ──── t4
                          ▲ HEAD
```

### Timeline — After `git revert t2`

```
t1 ──── t2 ──── t3 ──── t4 ──── t5
                                  ▲ HEAD

t5 = new commit that undoes the changes introduced in t2
t2 still exists in history — nothing was deleted
```

---

### Basic Usage

```bash
# Revert a specific commit by hash
git revert <commit-hash>

# Revert the last commit
git revert HEAD

# Revert without opening the editor (auto commit message)
git revert HEAD --no-edit

# Revert but don't auto-commit (stage only)
git revert HEAD --no-commit
```

---

### Reverting Multiple Commits

```
Timeline:
  t1 ──── t2 ──── t3 ──── t4
                            ▲ HEAD

Goal: undo t2 and t3

git revert t3   →   t1 ── t2 ── t3 ── t4 ── t5 (undoes t3)
git revert t2   →   t1 ── t2 ── t3 ── t4 ── t5 ── t6 (undoes t2)

Note: revert in REVERSE ORDER (newest first) to avoid conflicts
```

```bash
# Revert a range (exclusive..inclusive) — creates one commit per revert
git revert t2..t4
```

---

### What "undoing" means

```
t2 added these lines to login.js:
  + const token = generateToken();
  + saveToCache(token);

t5 (revert of t2) removes those lines:
  - const token = generateToken();
  - saveToCache(token);

The revert commit is the mirror image of the original.
```

---

### 💡 Tips

> **Tip 1:** Prefer `git revert` over `git reset` for anything already pushed.
> Revert keeps history intact — no force push needed, no teammates affected.

> **Tip 2:** Use `--no-commit` to combine multiple reverts into one tidy commit.
>
> ```bash
> git revert --no-commit t3
> git revert --no-commit t2
> git commit -m "Revert broken login changes"
> ```

> **Tip 3:** Revert works on merge commits too, but needs a `-m` flag.
>
> ```bash
> git revert -m 1 <merge-commit-hash>
> # -m 1 means "keep the first parent's version"
> ```

---

### ⚠️ Cautions

> **Caution 1:** Reverting a revert is valid and sometimes necessary.
> If you reverted a feature and later want it back, `git revert <revert-commit>` brings it back cleanly.

> **Caution 2:** Reverting a merge commit can be tricky.
> It removes the feature from the branch but the branch's history still shows it was merged.
> Re-merging the same branch later won't bring the feature back — you'd need to revert the revert first.

> **Caution 3:** Conflicts can still happen during revert.
> If code around the reverted lines has changed since, Git may not be able to apply the reverse patch cleanly.

---

---

## git rebase

### What it does

Takes your commits and **replays them on top of a different base commit**.
The result looks like your work branched off from a different point than it actually did.

It **rewrites commit hashes** — the content is the same but the ancestry changes.

### Timeline — Before Rebase

```
          feature branch
          ↓
t1 ──── t2 ──── t5 ──── t6
         \
          t3 ──── t4
          ▲
          feature branched off t2
```

### Timeline — After `git rebase main`

```
t1 ──── t2 ──── t5 ──── t6
                          \
                           t3' ──── t4'
                           ▲
                           feature now based on t6 (tip of main)

t3' and t4' are NEW commits — same content, different hashes
```

---

### Standard Rebase Workflow

```bash
# You are on feature branch
git checkout feature

# Replay your feature commits on top of latest main
git rebase main
```

**Step by step, what Git does internally:**

```
Step 1: Find common ancestor
        common ancestor = t2

Step 2: Save your commits as patches
        patch-A = diff(t2 → t3)
        patch-B = diff(t3 → t4)

Step 3: Reset feature branch to tip of main (t6)

Step 4: Apply patches one by one
        apply patch-A → t3'
        apply patch-B → t4'

Result: t3' and t4' are on top of t6
```

---

### Interactive Rebase — `git rebase -i`

The most powerful tool for cleaning up commit history before merging.

```bash
git rebase -i HEAD~3    # interactive rebase of last 3 commits
git rebase -i <hash>    # interactive rebase from a specific commit
```

**The editor opens showing:**

```
pick t2a1b3c Added login form
pick t3d4e5f Fixed typo
pick t4f5g6h WIP dont review this

# Commands:
# pick   = keep commit as-is
# reword = keep commit, change message
# edit   = pause to amend the commit
# squash = merge into previous commit (keeps both messages)
# fixup  = merge into previous commit (discard this message)
# drop   = delete this commit entirely
```

**Common interactive rebase use cases:**

```
squash — combine WIP commits into one clean commit
──────────────────────────────────────────────────
Before:
  t1 ── t2(Added login) ── t3(fix typo) ── t4(fix test) ── t5(final)

Change to:
  pick   t2 Added login
  squash t3 fix typo
  squash t4 fix test
  squash t5 final

After:
  t1 ── t2'(Added login - single clean commit)


reword — fix a commit message
──────────────────────────────────────────────────
  pick   t2 Added login
  reword t3 fix typo       ← editor opens for just this message


drop — delete a commit entirely
──────────────────────────────────────────────────
  pick t2 Added login
  drop t3 Accidentally committed password     ← gone
  pick t4 Added tests
```

---

### Rebase vs Merge — Visual Comparison

```
SCENARIO: feature branch behind main by 2 commits (t5, t6)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MERGE approach:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t1 ── t2 ── t5 ── t6 ──────────── M (merge commit)
              \                  /
               t3 ──── t4 ──────

Result: non-linear history, preserves when things actually happened
        creates a merge commit (M)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
REBASE approach:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

t1 ── t2 ── t5 ── t6 ── t3' ── t4'

Result: linear history, looks like feature was always built on t6
        no merge commit
```

---

### Handling Rebase Conflicts

```
During rebase, if a conflict occurs:

  t1 ──── t2 ──── t5 ──── t6
                            \
                         ❌ t3' ← conflict here

Git pauses and tells you:
  "CONFLICT (content): Merge conflict in login.js"

You:
  1. Open the file, resolve the conflict markers
     <<<<<<< HEAD       (t6 version)
     ...
     =======
     ...
     >>>>>>> t3         (your version)

  2. Stage the resolved file:
     git add login.js

  3. Continue the rebase:
     git rebase --continue

  4. Or abort entirely (go back to before rebase):
     git rebase --abort
```

---

### 💡 Tips

> **Tip 1:** Always rebase on an updated branch.
>
> ```bash
> git fetch origin
> git rebase origin/main    # use the remote version, not local
> ```

> **Tip 2:** `git rebase --abort` is always safe.
> If anything goes wrong during rebase, `--abort` returns you to exactly where you started. No damage done.

> **Tip 3:** Use interactive rebase to clean up before opening a PR.
> Squash WIP commits, fix typos in messages, drop accidental commits.
> Reviewers will thank you.

> **Tip 4:** After rebasing, your local and remote histories have diverged.
> You'll need force push to update the remote:
>
> ```bash
> git push --force-with-lease    # safer than --force, checks no one else pushed
> ```

---

### ⚠️ Cautions

> **Caution 1:** Never rebase a branch that others are working on.
> Rebase rewrites commit hashes. Anyone else based on your old commits will have a broken history when you force push.
> **Golden rule: only rebase local/private branches.**

> **Caution 2:** `--force` vs `--force-with-lease`
>
> ```bash
> git push --force               # overwrites remote no matter what
> git push --force-with-lease    # fails if remote has new commits you haven't seen
> ```
>
> Always prefer `--force-with-lease` — it protects against overwriting teammates' pushes.

> **Caution 3:** Interactive rebase changes commit hashes of ALL commits after the edit point.
> Even commits you marked `pick` (unchanged content) get new hashes.

---

---

## Quick Comparison

```
┌─────────────┬──────────────┬─────────────────┬────────────────────┐
│             │ git reset    │ git revert       │ git rebase         │
├─────────────┼──────────────┼─────────────────┼────────────────────┤
│ What it does│ Moves HEAD   │ Adds a new undo  │ Replays commits on │
│             │ backwards    │ commit           │ a new base         │
├─────────────┼──────────────┼─────────────────┼────────────────────┤
│ Rewrites    │ Yes          │ No               │ Yes                │
│ history?    │              │                  │                    │
├─────────────┼──────────────┼─────────────────┼────────────────────┤
│ Safe for    │ No           │ Yes ✅           │ No                 │
│ shared      │              │                  │                    │
│ branches?   │              │                  │                    │
├─────────────┼──────────────┼─────────────────┼────────────────────┤
│ Data loss   │ Possible     │ No               │ Possible if        │
│ risk?       │ (--hard)     │                  │ misused            │
├─────────────┼──────────────┼─────────────────┼────────────────────┤
│ Creates new │ No           │ Yes              │ No (rewrites       │
│ commit?     │              │                  │ existing ones)     │
└─────────────┴──────────────┴─────────────────┴────────────────────┘
```

---

## When to Use What

```
Situation                                        → Command
──────────────────────────────────────────────────────────────────────
Fix last commit message (not pushed)             → git reset --soft HEAD~1
Unstage files from last commit (not pushed)      → git reset HEAD~1
Throw away last commit + all changes (not pushed)→ git reset --hard HEAD~1
Undo a commit that's already pushed/shared       → git revert <hash>
Clean up messy commits before PR                 → git rebase -i HEAD~N
Bring feature branch up to date with main        → git rebase main
Combine multiple WIP commits into one            → git rebase -i (squash)
Delete a specific old commit from history        → git rebase -i (drop)
Recover from a bad hard reset                    → git reflog + git reset
```

---

## 🔖 Quick Reference Cheatsheet

```bash
# ── RESET ──────────────────────────────────────────────
git reset --soft HEAD~1          # undo commit, keep staged
git reset HEAD~1                 # undo commit, unstage (default)
git reset --hard HEAD~1          # undo commit + delete changes
git reset --hard HEAD            # discard all uncommitted changes
git reflog                       # see all recent HEAD positions (lifesaver)

# ── REVERT ─────────────────────────────────────────────
git revert HEAD                  # revert last commit (opens editor)
git revert HEAD --no-edit        # revert last commit (auto message)
git revert <hash> --no-commit    # stage revert without committing
git revert -m 1 <merge-hash>     # revert a merge commit

# ── REBASE ─────────────────────────────────────────────
git rebase main                  # rebase current branch onto main
git rebase -i HEAD~3             # interactive rebase last 3 commits
git rebase --continue            # after resolving conflict
git rebase --abort               # cancel rebase, return to original state
git push --force-with-lease      # safe force push after rebase
```

---

_Notes by KunTsuKe | Last updated May 2026_
