# 05 — How git thinks

**Goals:** understand the few ideas that explain (almost) everything git does: the three areas, commits, branches and `HEAD`.

## Why version control?

Without git:
```
thesis.docx
thesis_v2.docx
thesis_v2_final.docx
thesis_v2_final_REALLY_final_corrected.docx
```

With git there's **one** `thesis` folder, plus a complete history: who changed what, when and why. You can go back to any point, try things out without risk, and work with others on the same files.

Git works best with **text files** (code, Markdown, LaTeX, MusicXML, LilyPond, config files…). Binary files (Word, audio, images) can be stored, but git can't show *what* changed inside them.

## The three areas

```
  Working directory          Staging area              Repository
  (your files)               (the "index")             (the history, in .git/)
 ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
 │ edit files here │ ────► │ prepared for    │ ────► │ commits,        │
 │                 │  git  │ the next commit │  git  │ saved forever   │
 │                 │  add  │                 │ commit│                 │
 └─────────────────┘       └─────────────────┘       └─────────────────┘
          ▲                                                   │
          └────────────── git restore / git switch ───────────┘
```

1. **Working directory:** the normal files you see and edit.
2. **Staging area:** a "shopping cart" for the next commit. `git add` puts changes in it. This lets you commit only *some* of your changes.
3. **Repository:** the saved history, a hidden `.git` folder in your project. **Don't touch `.git` by hand, and never delete it** unless you mean to throw away the whole history.

`git status` tells you, at any time, what is in which area. **When in doubt: `git status`.**

## Commits = snapshots

A commit is a **snapshot of the whole project** at a point in time, plus:
- author and date,
- a **message** explaining *why*,
- a pointer to its **parent** (the previous commit).

Each commit has a unique ID (a *hash*) like `a1b2c3d4e5…`. Usually the first 7 characters are enough: `a1b2c3d`.

```
  A ◄── B ◄── C          (each commit points back to its parent)
```

## Branches are just pointers

A branch is a **movable label** on a commit. Nothing more. When you commit, the current branch moves forward to the new commit.

```
              main
               ▼
  A ◄── B ◄── C
```

Create a branch `feature` and commit twice on it:

```
              main
               ▼
  A ◄── B ◄── C ◄── D ◄── E
                          ▲
                       feature
```

Switch back to `main` and commit there: the history **forks**:

```
                    F   ◄── main
                   /
  A ◄── B ◄── C ◄─┤
                   \
                    D ◄── E   ◄── feature
```

A **merge** brings the two lines together again with a *merge commit* that has two parents.

## HEAD — "you are here"

`HEAD` points to the branch you're currently on (and so to its latest commit). `git switch <branch>` moves `HEAD` to another branch and updates your files to match.

`HEAD~1` means "one commit before HEAD", `HEAD~2` "two before", and so on. You'll use that when undoing things.

## Local and remote

```
  Your computer                                 GitHub / GitLab / Codeberg
 ┌──────────────────────────┐   git push     ┌──────────────────────────┐
 │ repository (.git)        │ ─────────────► │ "origin"                 │
 │ main, feature, ...       │ ◄───────────── │ main, ...                │
 └──────────────────────────┘   git pull     └──────────────────────────┘
```

- Your local repo is **complete**. You can commit, branch and look at the history offline.
- `origin` is the conventional name of the remote copy.
- Nothing goes to the server until you `push`. Nothing comes from it until you `pull` (or `fetch`).

## Takeaways

1. **`git status`** is your best friend.
2. Changes go *working directory → `add` → staging → `commit` → repository → `push` → remote*.
3. A branch is a pointer, so branches are cheap. Use them.
4. Almost everything that was **committed** can be recovered. Uncommitted changes are much easier to lose.
