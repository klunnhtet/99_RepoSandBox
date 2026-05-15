# Git Conflicts — What Happens & How to Survive

---

## Table of Contents

- [What Is a Conflict?](#what-is-a-conflict)
- [Conflict Type 1 — Push Rejected](#conflict-type-1--push-rejected)
- [Conflict Type 2 — Pull / Merge Conflict](#conflict-type-2--pull--merge-conflict)
- [Conflict Type 3 — Rebase Conflict](#conflict-type-3--rebase-conflict)
- [Conflict Type 4 — Stash Conflict](#conflict-type-4--stash-conflict)
- [Reading Conflict Markers](#reading-conflict-markers)
- [Resolving Step by Step](#resolving-step-by-step)
- [Prevention Cheatsheet](#prevention-cheatsheet)

---

## What Is a Conflict?

A conflict = **two people (or two timelines) edited the same line differently**.
Git cannot decide which version is correct — it stops and asks you.

```
Person A edited line 5:   "color: red"
Person B edited line 5:   "color: blue"
                               ▲
                          Git: "I don't know. You decide."
```

**Conflicts can happen during:**

```
git pull      ← remote has changes you don't have
git merge     ← combining two branches
git rebase    ← replaying commits on new base
git stash pop ← restoring stashed work
git cherry-pick
```

---

## Conflict Type 1 — Push Rejected

### What happened

```
Timeline:

t1   Both you and teammate start from same commit
     remote  ── A ── B
     local   ── A ── B
                      ▲ same starting point

t2   Teammate pushes first
     remote  ── A ── B ── C (teammate's commit)

t3   You try to push YOUR commit
     local   ── A ── B ── D (your commit)

     git push → ❌ REJECTED

     remote  ── A ── B ── C  ← remote is ahead
     local   ── A ── B ── D  ← your version diverged
```

### Error you see

```
! [rejected] main -> main (non-fast-forward)
error: failed to push some refs
hint: Updates were rejected because the tip of your current
hint: branch is behind its remote counterpart.
```

### What to do

```
Option A — Pull then push (safe, creates merge commit)
─────────────────────────────────────────────────────
git pull
# resolve conflicts if any
git push

Result:
  remote ── A ── B ── C ──── M (merge commit)
                         \  /
  local  ── A ── B ── D ─/


Option B — Rebase then push (clean, linear history)
─────────────────────────────────────────────────────
git pull --rebase
# resolve conflicts if any
git push

Result:
  remote ── A ── B ── C ── D'
                            ▲ your commit replayed on top
```

> ⚠️ Never `git push --force` on a shared branch. Overwrites teammates' work.

> 💡 `git push --force-with-lease` is safer — fails if remote has new commits you haven't seen.

---

## Conflict Type 2 — Pull / Merge Conflict

### What happened

```
Timeline:

t1   Same starting point
     remote  ── A ── B
     local   ── A ── B

t2   Both sides edit the SAME file
     remote  ── A ── B ── C   (C edits login.js line 10)
     local   ── A ── B ── D   (D also edits login.js line 10)

t3   git pull → Git fetches C, tries to merge C + D

     ── A ── B ── C
                   \
                    ── D  ← same line edited → ❌ CONFLICT
```

### Error you see

```
Auto-merging login.js
CONFLICT (content): Merge conflict in login.js
Automatic merge failed; fix conflicts and then commit the result.
```

### What Git does to your file

```
<<<<<<< HEAD               ← your local version starts here
const url = "/api/v2/login"
=======                    ← divider
const url = "/api/v1/login"
>>>>>>> origin/main        ← remote version ends here
```

### States during conflict

```
git status output:

  Unmerged paths:
    (use "git add <file>..." to mark resolved)
    both modified: login.js    ← needs your decision
```

### Fix flow

```
t3   Conflict detected
      │
t4   Open login.js, find all <<<<<<< markers
      │
t5   Edit the file — keep what's correct, delete ALL markers
      │
      │   <<<<<<< HEAD                 ← DELETE
      │   const url = "/api/v2/login"  ← KEEP (or edit)
      │   =======                      ← DELETE
      │   const url = "/api/v1/login"  ← DELETE (or keep)
      │   >>>>>>> origin/main          ← DELETE
      │
      │   Result: const url = "/api/v2/login"
      │
t6   git add login.js       ← mark as resolved
      │
t7   git commit             ← Git uses auto-generated merge message
      │
t8   git push               ← done ✅
```

> 💡 Use a merge tool to see conflicts side by side:
>
> ```bash
> git mergetool          # opens configured tool
> git mergetool --tool=vimdiff
> ```

> ⚠️ After resolving, always run your tests before committing.
> You may have kept the wrong version and broken something.

---

## Conflict Type 3 — Rebase Conflict

### What happened

```
Timeline:

t1   Feature branch from B
     main     ── A ── B ── C ── D   (D is new, added while you worked)
                      \
     feature          E ── F        (your work)

t2   git rebase main

     Git replays E then F on top of D

     ── A ── B ── C ── D
                        \
                    tries E' → ❌ CONFLICT (E touches same lines as D)
```

### Error you see

```
CONFLICT (content): Merge conflict in auth.js
error: could not apply e3a1b2c... Add token validation
hint: Resolve all conflicts manually, mark them with
hint: "git add/rm <conflicted_files>", then run
hint: "git rebase --continue".
```

### Rebase conflict is per-commit

```
Replaying E' → conflict → you resolve → git rebase --continue
                                                      │
                                              Replaying F' → conflict → resolve → continue
                                                                                      │
                                                                               Rebase done ✅
```

### Fix flow

```
t2   Conflict on commit E'
      │
t3   Resolve the conflict in the file (same as merge conflict)
      │
t4   git add auth.js
      │
t5   git rebase --continue     ← NOT git commit
      │
      ├── more conflicts? → repeat t3-t5
      │
t6   All commits replayed → ✅
      │
t7   git push --force-with-lease origin feature
```

### Abort if things get messy

```bash
git rebase --abort    # returns you to exactly before rebase started
                      # 100% safe to use anytime during rebase
```

> 💡 Rebase conflicts = one conflict per commit being replayed.
> A rebase of 5 commits can give you up to 5 separate conflict rounds.

> ⚠️ Do NOT `git commit` during rebase. Use `git rebase --continue`.

---

## Conflict Type 4 — Stash Conflict

### What happened

```
Timeline:

t1   You stash unfinished work
     git stash       → work saved to stash stack

t2   You pull / switch branches / make commits

t3   git stash pop  → Git tries to re-apply stashed changes

     If the file changed between t1 and t3 on the same lines → ❌ CONFLICT
```

### Error you see

```
Auto-merging index.js
CONFLICT (content): Merge conflict in index.js
The stash was NOT removed due to conflicts.
```

### Stash state after conflict

```
git status:

  Unmerged paths:
    both modified: index.js

Stash is still in the stack (not popped):
  git stash list → stash@{0}: WIP on main: abc1234 your message
```

### Fix flow

```
t3   Conflict detected after stash pop
      │
t4   Resolve conflict in index.js (same process)
      │
t5   git add index.js
      │
t6   git commit  OR  just continue working (no commit needed if just restoring work)
      │
t7   git stash drop    ← manually remove the stash since it wasn't auto-dropped
```

> 💡 `git stash pop` = apply + drop stash (but drop is skipped on conflict)
> `git stash apply` = apply only, never drops — safer if you expect conflicts

---

## Reading Conflict Markers

```
Full anatomy of a conflict block:

  <<<<<<< HEAD                      ← YOUR version label
  const timeout = 5000;             ← your code
  ||||||| merged common ancestors   ← ORIGINAL (only with diff3 style)
  const timeout = 3000;             ← what it looked like before both edits
  =======                           ← divider
  const timeout = 8000;             ← THEIR code
  >>>>>>> origin/main               ← THEIR version label


Enable diff3 style (shows original too — highly recommended):
  git config --global merge.conflictstyle diff3
```

### Decision guide

```
Situation                          → Action
──────────────────────────────────────────────────────
Your version is correct            → keep yours, delete their block
Their version is correct           → keep theirs, delete your block
Both are needed                    → manually combine both into one block
Neither — rewrite from scratch     → delete both blocks, write new code
```

---

## Resolving Step by Step

### Universal fix flow (works for merge, pull, rebase, stash)

```
CONFLICT DETECTED
       │
       ▼
┌──────────────────────────────────────────┐
│  git status                              │
│  → see which files are "both modified"   │
└──────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────┐
│  Open each conflicted file               │
│  Find ALL <<<<<<< markers                │
│  Edit to keep correct code               │
│  Delete ALL markers (<<<<, ====, >>>>)   │
└──────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────┐
│  git add <resolved-file>                 │
│  Repeat for every conflicted file        │
└──────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────┐
│  Merge/Pull?  → git commit               │
│  Rebase?      → git rebase --continue    │
│  Stash?       → git stash drop           │
└──────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────┐
│  Run tests                               │
│  git push                                │
└──────────────────────────────────────────┘
       │
       ▼
      ✅ Done
```

### Verify no markers left before committing

```bash
# Search for leftover conflict markers in all files
grep -r "<<<<<<" .
grep -r "=======" .
grep -r ">>>>>>>" .

# If output is empty → you're clean ✅
```

---

## Prevention Cheatsheet

```
Rule                                         Why
──────────────────────────────────────────────────────────────────
Pull before you start working               You start from latest code
git pull upstream main / git fetch          Reduces divergence gap
Work on feature branches, not main          main stays clean and pullable
Keep PRs small and focused                  Less overlap with teammates
Communicate when touching shared files      Avoid stepping on each other
Commit often                                Smaller diffs = smaller conflicts
Rebase feature branch on main regularly     Conflicts caught early, one at a time
Use .gitattributes for binary files         Prevents unmergeable binary conflicts
```

### Conflict escape hatches

```bash
# During merge — abort everything, go back to before
git merge --abort

# During rebase — abort everything, go back to before
git rebase --abort

# During cherry-pick — abort
git cherry-pick --abort

# Nuclear: throw away ALL local changes and match remote exactly
git fetch origin
git reset --hard origin/main
```

> ⚠️ `reset --hard` deletes all local uncommitted work permanently.
> Use only when you truly want to discard everything.

---

## Quick Reference

```
Error                              Cause                    Fix
──────────────────────────────────────────────────────────────────────────────
non-fast-forward push rejected     remote ahead of local    git pull then push
CONFLICT (content)                 same line edited twice   resolve markers, add, commit
refusing to merge unrelated hist.  no common ancestor       git pull --allow-unrelated-histories
Already up to date (but wrong)     wrong remote/branch      check git remote -v
CONFLICT during rebase             commit clashes with base  resolve, git rebase --continue
Stash conflict on pop              file changed since stash  resolve, git stash drop
```

---

_Notes by KunTsuKe | Last updated May 2026_
