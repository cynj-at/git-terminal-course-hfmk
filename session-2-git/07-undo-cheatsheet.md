# 07 — "Oops!" — undoing things

**Goals:** for each typical mistake, know which command fixes it and how dangerous that command is.

## The golden rule

> **Has it been pushed (shared with others)?**
> - **No** → you may rewrite history (`amend`, `reset`).
> - **Yes** → don't rewrite. Add a new commit that undoes the change (`revert`).

Rewriting history that others already have causes them a lot of pain (and `git push` refuses it unless you force it).

## Decision table

| Situation | Command | Danger |
|---|---|---|
| Changed a file, want the last committed version back | `git restore <file>` | ⚠️ uncommitted changes in that file are **lost** |
| Staged a file by accident (`git add`) | `git restore --staged <file>` | ✅ safe, the change stays in your file |
| A committed file should be ignored, but `.gitignore` has no effect | `git rm --cached <file>`, then commit | ✅ the file stays on your disk, git stops tracking it |
| Typo in the **last** commit message (not pushed) | `git commit --amend -m "New message"` | ✅ local only |
| Forgot a file in the **last** commit (not pushed) | `git add <file>` then `git commit --amend --no-edit` | ✅ local only |
| A commit **already pushed** is wrong | `git revert <hash>` | ✅ safe, creates a new "undo" commit |
| Last commit(s) not pushed, want to **redo** them | `git reset --soft HEAD~1` | ✅ changes stay staged |
| ...and also unstage them | `git reset HEAD~1` (= `--mixed`) | ✅ changes stay in your files |
| ...and **throw them away completely** | `git reset --hard HEAD~1` | ⚠️ commits gone from the branch (rescue: `reflog`); uncommitted changes **lost forever** |
| Want one file as it was in an older commit | `git restore --source=<hash> <file>` | ⚠️ overwrites the current file |
| One specific commit from another branch is needed here | `git cherry-pick <hash>` | ✅ creates a copy of that commit |
| "I broke everything and lost commits!" | `git reflog`, then `git reset --hard HEAD@{n}` | 🛟 the safety net |

> ▶️ **Exercise checkpoint: S3 — I messed up a file / staged the wrong thing** ·[open exercise](exercises-session-2.md#s3--i-messed-up-a-file--staged-the-wrong-thing)
>
> ▶️ **Exercise checkpoint: S4 — Typo in my commit, and I forgot a file** · [open exercise](exercises-session-2.md#s4--typo-in-my-commit-and-i-forgot-a-file)

## `revert` vs. `reset`

```
reset:    A ◄── B ◄── C(bad)        →   A ◄── B                    (C disappears, history rewritten)
revert:   A ◄── B ◄── C(bad)        →   A ◄── B ◄── C ◄── C'       (C' undoes C, history kept)
```

`revert` is always safe for shared branches. Everyone can just `pull`.

> ▶️ **Exercise checkpoint: S5 — A bug went out to everyone** · [open exercise](exercises-session-2.md#s5--a-bug-went-out-to-everyone)

## `reset` — the three modes

`git reset <target>` moves the current branch back to `<target>`. The mode decides what happens to the changes from the commits you "removed":

```
before:   A ◄── B ◄── C ◄── D     main, HEAD on D

git reset --soft  HEAD~2   →  main on B; changes from C+D are STAGED      (ready to commit again)
git reset --mixed HEAD~2   →  main on B; changes from C+D are in your FILES, unstaged
git reset --hard  HEAD~2   →  main on B; changes from C+D are GONE        (files look like B)
```

Memory aid: **soft** keeps everything, **mixed** keeps the files, **hard** keeps nothing.

## The safety net: `reflog`

Git records every place `HEAD` has been, including commits that no branch points to anymore:

```bash
git reflog
```
```
3f2a1bc HEAD@{0}: reset: moving to HEAD~2
9d8e7f6 HEAD@{1}: commit: Add coda
5c4b3a2 HEAD@{2}: commit: Add bridge
...
```

To go back to the state before the reset:

```bash
git reset --hard HEAD@{1}       # or use the hash: git reset --hard 9d8e7f6
```

The reflog is local and keeps entries for about 90 days by default. **Committed work is almost never truly lost.** Uncommitted work, however, can be lost. So: commit often.

> ▶️ **Exercise checkpoint: S6 — My last commits are a mess / I deleted too much** · [open exercise](exercises-session-2.md#s6--my-last-commits-are-a-mess-and-i-deleted-too-much)

## Detached HEAD — what is that?

If you switch to a **commit** instead of a branch (`git switch --detach a1b2c3d`, or the old `git checkout a1b2c3d`), `HEAD` points directly at the commit. Git calls this *detached HEAD*. That's fine for **looking around** in the past. If you want to keep commits you make there, create a branch: `git switch -c rescue-branch`. To go back: `git switch main`.

## Branches and merging

A branch is a separate line of work (see [05](05-git-concepts.md#branches-are-just-pointers)). The commands:

```bash
git branch                       # list branches, * = the one you're on
git switch -c feature            # create a branch "feature" and switch to it
git switch main                  # go back to main
git merge feature                # bring feature's commits INTO the branch you're on (here: main)
git branch -d feature            # delete a branch that is merged
git log --oneline --graph --all  # see all branches as a graph
```

**Direction matters:** first `git switch` to the branch that should *receive* the changes, then `git merge <other>`.

A merge ends in one of two ways:
- **Fast-forward:** `main` had no new commits since the branch started, so `main` simply moves forward. No new commit.
- **Merge commit:** both sides have new commits. Git joins them with a new commit that has two parents. An editor opens for its message: just save and close.

### When both sides changed the same lines: a merge conflict

Git can't decide for you, so it stops and marks the spot in the file:

```
<<<<<<< HEAD
Venue: Chamber Music Hall      ← your version (the branch you're on)
=======
Venue: Great Hall              ← the version from the branch you're merging in
>>>>>>> new-venue
```

1. `git status` lists the files with conflicts (*"both modified"*).
2. Open each file and make it look the way it should: one side, the other, or a mix. **Delete all three marker lines.**
3. `git add <file>` marks it as resolved.
4. `git commit` finishes the merge.

Panic button: `git merge --abort` puts everything back to how it was before the merge.

## Practice

> ▶️ **Exercise checkpoint: S7 — Working on a branch, and a merge conflict** · [open exercise](exercises-session-2.md#s7--working-on-a-branch-and-a-merge-conflict)

> 🏠 **Optional, at home:** [S8 — cherry-pick](exercises-session-2.md#s8---i-need-just-that-one-commit) and [S9 — shell config in git](exercises-session-2.md#s9---my-shell-config-in-git-oops-with-my-password-in-it).
>
> 💪 **More practice:** [exercises-extra.md](exercises-extra.md): merging (X0), merge conflicts and special situations like a rejected push, `stash`, commits on the wrong branch, or a cherry-pick with a conflict (X1–X9).
>
> 🏁 **End of session 2.** Keep the [cheatsheet](../cheatsheet.md) handy.
