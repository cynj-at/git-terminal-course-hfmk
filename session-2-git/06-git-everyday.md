# 06 — The everyday workflow

**Goals:** start or clone a repository, commit changes, look at the history and differences, and sync with a remote.

## Starting a repository

Either **start locally**:

```bash
mkdir my-project && cd my-project
git init
```

…or **copy an existing one** from a server (this also downloads the whole history):

```bash
git clone git@<your-host>:<username>/<repo>.git
cd <repo>
```

## The core loop

```bash
git status                  # 1. what changed?
git diff                    # 2. show the exact changes (not yet staged)
git add notes.md            # 3. stage a file
git add .                   #    ...or everything in this folder and below
git diff --staged           # 4. what will go into the commit?
git commit -m "Add rehearsal notes for Tuesday"   # 5. save the snapshot
git push                    # 6. upload to the remote
```

And to get the changes other people (or you on another computer) pushed:

```bash
git pull
```

> 💡 **Rhythm:** `status` → `add` → `status` → `commit`. Run `git status` before and after each step until it becomes a habit.

## Good commit messages

- Short summary line (≤ ~50 characters), written as a command: *"Add intro section"*, *"Fix typo in bar 12"*.
- Explain **why**, not just what. The *what* is in the diff.
- One commit = one logical change. "Fix bar 12 and reformat everything and add new piece" is three commits.

`git commit` without `-m` opens your editor for a longer message: the first line is the summary, then an empty line, then details.

## Looking at the history

```bash
git log                         # full history (q to quit)
git log --oneline               # one line per commit
git log --oneline --graph --all # with branches drawn as a graph
git show a1b2c3d                # what exactly changed in this commit?
git show HEAD                   # ...in the latest commit
```

## Ignoring files: `.gitignore`

Some files should never be committed: temporary files, build output, large audio exports, passwords. List them in a file called `.gitignore` in the project root:

```gitignore
# macOS / Windows junk
.DS_Store
Thumbs.db

# exports and temporary files
*.wav
*.tmp
build/

# secrets
.env
```

Ignored files no longer show up in `git status`. Commit the `.gitignore` itself.

> ⚠️ Never commit passwords, tokens or private keys. Even if you delete them later, **they stay in the history.**

## Remotes

```bash
git remote -v                                       # which remotes exist?
git remote add origin git@<your-host>:<user>/<repo>.git   # connect a local repo to a server
git push -u origin main                             # first push: -u remembers origin/main as the default
git push                                            # afterwards: just push
```

> ▶️ **Exercise checkpoint: S1 — My first repository** · [open exercise](exercises-session-2.md#s1--my-first-repository)
>
> ▶️ **Exercise checkpoint: S2 — The daily routine** · [open exercise](exercises-session-2.md#s2--the-daily-routine)

## Old commands you'll see in tutorials

Older tutorials use `git checkout` for almost everything. Git split it into two clearer commands:

| Old | New (use these) |
|---|---|
| `git checkout <branch>` | `git switch <branch>` |
| `git checkout -b <new-branch>` | `git switch -c <new-branch>` |
| `git checkout -- <file>` | `git restore <file>` |
| `git reset HEAD <file>` | `git restore --staged <file>` |

`checkout` still works, so don't be confused when you see it.

One `checkout` trick you'll see a lot: getting a **single file** from another commit or branch without touching the rest.

```bash
git checkout <commit-or-branch> -- <file>         # old way: changes the file AND stages it
git restore --source=<commit-or-branch> <file>    # new way: only changes the file in your folder
```

## Try it

→ Exercises **S1** and **S2** in [exercises-session-2.md](exercises-session-2.md).
