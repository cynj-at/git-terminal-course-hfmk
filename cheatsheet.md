# Cheatsheet — Terminal & Git

Everything from the course on a few pages. The links (→) lead to the chapter with the full explanation.

**Placeholders:** `<file>`, `<dir>`, `<branch>`, `<hash>`, `<url>`: replace them, including the brackets.

---

## 1. Terminal basics → [01](session-1-terminal/01-terminal-basics.md)

| Command | Does |
|---|---|
| `echo $SHELL` | your login shell (`/bin/zsh`, `/usr/bin/bash` …) |
| `echo $0` | the shell running right now |
| `chsh -s $(which zsh)` | make zsh your default shell (macOS/Linux, not Git Bash) |
| `whoami`, `date` | who am I, what time is it |
| `man <cmd>` | manual (`q` quits; not in Git Bash) |
| `<cmd> --help` | short help (Linux, Git Bash) |
| `clear` | clear the screen |

**Anatomy:** `ls -l -a Documents` = command · options (`-la` combined, `--all` long form) · argument. File names with spaces need quotes: `"My Documents"`.

## 2. Navigating and files → [02](session-1-terminal/02-navigation-files.md)

| Command | Does |
|---|---|
| `pwd` | where am I? |
| `ls` · `ls -l` · `ls -a` · `ls -la ~` | list · details · incl. hidden · both, for home |
| `cd <dir>` · `cd ..` · `cd ~` (or `cd`) · `cd /` · `cd -` | into · up · home · root · back to previous |
| `mkdir <dir>` · `mkdir -p a/b/c` | create a folder · with all missing parents |
| `touch <file>` | create an empty file |
| `echo "text" > <file>` · `>>` | write into a file (overwrites!) · append |
| `cat <file>` · `less <file>` | print a file · page through it (`q` quits) |
| `head -n 3 <file>` · `tail -n 3 <file>` | first / last 3 lines |
| `cp a b` · `cp -r dirA dirB` | copy a file · a folder |
| `mv a b` · `mv a <dir>/` | rename · move |
| `rm <file>` · `rm -r <dir>` · `rmdir <dir>` | delete · delete a folder with content · delete an empty folder. ⚠️ **No trash can.** |
| `rm -i` | safety net (interactive deletion) |
| `find . -name "*.txt"` | find files below this folder |

**Paths:** absolute starts at `/` or `~` (`/home/alex/notes.txt`, `~/notes.txt`) · relative starts where you are (`Documents/notes.txt`, `../Music`) · `.` here · `..` parent · `~` home.
**Windows (Git Bash):** `C:` is `/c/`, home is `/c/Users/<you>`.

**Wildcards:** `*` any text (`*.wav`) · `?` exactly one character (`track-0?.mp3`).

**Pipes and redirection:**

| | |
|---|---|
| `cmd1 \| cmd2` | output of cmd1 becomes input of cmd2 |
| `cmd > f` · `cmd >> f` | output into a file · append |
| `grep text f` | lines containing "text" |
| `grep text f \| wc -l` | count them |
| `sort f` | sorted |
| `history \| grep git` | search your command history |

**Keys:**

| Key | Does |
|---|---|
| `Tab` (twice: show options) | complete names and commands |
| `↑` / `↓` | previous / next command |
| `Ctrl+R` | search history |
| `Ctrl+C` | cancel the running command / the current line |
| `Ctrl+L` | clear the screen |
| `Ctrl+A` / `Ctrl+E` | start / end of line |
| `q` | quit `less`, `man`, `git log` |

**Stuck?** `q`, then `Ctrl+C`. In Vim: `Esc`, `:q!`, `Enter` (quit without saving) or `:wq` (save and quit). In nano: `Ctrl+O`, `Enter` (save), `Ctrl+X` (quit).

## 3. Shell configuration → [03](session-1-terminal/03-shell-config.md)

| | zsh | bash |
|---|---|---|
| config file | `~/.zshrc` | `~/.bashrc` (Git Bash: loaded via `~/.bash_profile`) |
| reload | `source ~/.zshrc` | `source ~/.bashrc` |

```bash
alias ll='ls -la'                         # alias: no spaces around =
alias gs='git status'
mkcd() { mkdir -p "$1" && cd "$1"; }      # function: $1 = first argument
backup() { cp "$1" "$1.$(date +%Y%m%d%H%M%S).bak"; }
export EDITOR=nano                        # variable for other programs
export PATH="$HOME/bin:$PATH"             # also search ~/bin for programs

# load all files from ~/.config/shell/ (works in bash and zsh)
for f in ~/.config/shell/*.sh; do
  [ -r "$f" ] && source "$f"
done

# only use a tool if it's installed
if command -v eza >/dev/null; then alias ll='eza -l'; fi
```

| Command | Does |
|---|---|
| `alias` · `type <name>` | list all aliases · what is `<name>`? (alias, function, program, built-in) |
| `echo $PATH` · `which <cmd>` | search path · which program runs |
| `setopt interactivecomments` | (zsh) allow `# comments` when pasting commands |

**Good functions:** `local` variables · `${1:-default}` for defaults · check the input and `return 1` · always quote `"$1"`.
**Don't** reuse names that exist (`type <name>` first). In zsh, never name a variable `path`.

**fzf** (fuzzy finder, after `source <(fzf --zsh)` / `eval "$(fzf --bash)"`):

| Key | Does |
|---|---|
| `Ctrl+R` | fuzzy search your whole history |
| `Ctrl+T` | pick a file and insert its path |
| `Alt+C` | pick a subfolder and `cd` into it (macOS: Option must act as Meta) |
| `**` + `Tab` | fuzzy completion, e.g. `cd **<Tab>` |
| `Ctrl+G` | your own command snippets (needs the snippet widget from 03) |

## 4. Git and SSH setup (once per computer) → [04](session-1-terminal/04-git-setup.md)

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.org"      # same as on your git host (or its noreply address)
git config --global init.defaultBranch main
git config --global core.editor "nano"                # or "code --wait"
git config --global pull.rebase false
git config --list --show-origin                       # check

ls ~/.ssh                                             # key already there?
ssh-keygen -t ed25519 -C "you@example.org"            # create a key pair (Enter = default location)
ssh-add ~/.ssh/id_ed25519                             # load it into the agent
cat ~/.ssh/id_ed25519.pub                             # PUBLIC key (.pub!) → host settings → SSH keys
ssh -T git@github.com                                 # test (or git@gitlab.com, git@codeberg.org); answer "yes"
```

- Agent on **Windows (Git Bash)**: `eval "$(ssh-agent -s)"` first. **macOS**: `ssh-add --apple-use-keychain ~/.ssh/id_ed25519` plus `UseKeychain yes` in `~/.ssh/config`.
- Copy the key to the clipboard: macOS `pbcopy < ~/.ssh/id_ed25519.pub` · Git Bash `clip < ~/.ssh/id_ed25519.pub`.
- ⚠️ **Never** share or upload `id_ed25519` (without `.pub`). That's the private key.
- SSH URL: `git@github.com:<user>/<repo>.git` · HTTPS URL: `https://github.com/<user>/<repo>.git`.
- Alternative for GitHub with HTTPS: `gh auth login`.

## 5. How git thinks → [05](session-2-git/05-git-concepts.md)

```
working directory ──git add──► staging area ──git commit──► repository (.git) ──git push──► remote (origin)
                                                                              ◄──git pull───
```

- **Commit** = snapshot of the whole project + author, date, message, parent. ID = hash (`a1b2c3d`).
- **Branch** = a movable label on a commit. **HEAD** = "you are here".
- `HEAD~1` = one commit before HEAD, `HEAD~2` = two before.
- `origin` = the usual name for the remote copy. Nothing is sent before `push`; nothing arrives before `pull`.

## 6. Everyday git → [06](session-2-git/06-git-everyday.md)

| Command | Does |
|---|---|
| `git init` | turn the current folder into a repository |
| `git clone <url>` | copy a repository (with its whole history) |
| `git status` | **what's going on?** Always allowed, always useful |
| `git diff` · `git diff --staged` | changes not staged yet · staged changes |
| `git add <file>` · `git add .` | stage a file · stage everything here and below |
| `git commit -m "Message"` | commit the staged changes |
| `git commit` | commit with a longer message in the editor |
| `git commit -am "Message"` | stage all *tracked* modified files + commit (not new files) |
| `git log` · `git log --oneline` | history · one line per commit |
| `git log --oneline --graph --all` | all branches as a graph |
| `git show <hash>` · `git show HEAD` | what changed in a commit · in the latest one |
| `git show --stat HEAD` | which files changed |
| `git remote -v` | list remotes |
| `git remote add origin <url>` | connect a remote (once per repo) |
| `git remote set-url origin <url>` | change the URL (e.g. HTTPS → SSH) |
| `git push -u origin main` | first push of a branch |
| `git push` · `git pull` | upload · download and merge |

**The rhythm:** `status` → `add` → `status` → `commit` → `push`. `git pull` before you start, and before you push.

**Commit messages:** short summary (≤ 50 characters) as a command ("Add intro", "Fix typo in bar 12"). One commit = one logical change.

**`.gitignore`** (in the project root, commit it too):
```gitignore
.DS_Store
*.wav
*.tmp
build/
.env
```
Only affects files git **doesn't track yet**. Already tracked: `git rm --cached <file>`, then commit.
⚠️ Never commit passwords, tokens or keys: they stay in the history.

**Old and new commands:**

| Old (still works) | New |
|---|---|
| `git checkout <branch>` | `git switch <branch>` |
| `git checkout -b <branch>` | `git switch -c <branch>` |
| `git checkout -- <file>` | `git restore <file>` |
| `git reset HEAD <file>` | `git restore --staged <file>` |
| `git checkout <commit> -- <file>` (also stages) | `git restore --source=<commit> <file>` |
| `git checkout <hash>` | `git switch --detach <hash>` |

## 7. Branches and merging → [07](session-2-git/07-undo-cheatsheet.md#branches-and-merging)

| Command | Does |
|---|---|
| `git branch` | list branches (`*` = current) |
| `git branch -a` | including the branches on the remote |
| `git switch <branch>` | go to a branch |
| `git switch -c <branch>` | create a branch and go there |
| `git push -u origin <branch>` | first push of a new branch |
| `git merge <branch>` | merge `<branch>` **into the branch you're on** |
| `git merge --no-ff <branch>` | always create a merge commit |
| `git branch --merged` · `--no-merged` | which branches are (not) merged |
| `git branch -d <branch>` | delete a merged branch (safe) |
| `git branch -D <branch>` | delete it even if not merged ⚠️ |
| `git branch -m <new-name>` | rename the current branch (e.g. `master` → `main`) |

**Merge results:** *fast-forward* (main just moves forward, no new commit) · *merge commit* (two parents, editor opens: save and close).

### Merge conflicts

```
<<<<<<< HEAD
your version (the branch you're on = "ours")
=======
the incoming version (the branch you merge in = "theirs")
>>>>>>> other-branch
```

1. `git status` → which files? (*both modified*)
2. Edit each file, **delete all marker lines**
3. `git add <file>`
4. `git commit` (after a cherry-pick: `git cherry-pick --continue`)

| Situation | Command |
|---|---|
| cancel the merge | `git merge --abort` |
| take your version of a file completely | `git checkout --ours <file>` (or `git restore --ours <file>`) |
| take their version completely | `git checkout --theirs <file>` |
| binary file (audio, image, PDF) | no markers: pick `--ours` or `--theirs` |
| *modify/delete* conflict | keep the file: `git add <file>` · delete it: `git rm <file>` |
| find leftover markers | `grep -n "<<<<<<<\|>>>>>>>\|=======" <file>` |

⚠️ During a **rebase**, `ours` and `theirs` are swapped.

## 8. Undoing things → [07](session-2-git/07-undo-cheatsheet.md)

> **Golden rule:** already pushed? → `revert` (new commit). Not pushed yet? → `amend` / `reset` are fine.

| Situation | Command | |
|---|---|---|
| discard changes in a file (not committed) | `git restore <file>` | ⚠️ gone for good |
| unstage a file | `git restore --staged <file>` | ✅ change stays in the file |
| fix the last commit's message (not pushed) | `git commit --amend -m "New message"` | ✅ |
| add a forgotten file to the last commit (not pushed) | `git add <file>`, `git commit --amend --no-edit` | ✅ |
| undo a pushed commit | `git revert <hash>` (`--no-edit` skips the editor) | ✅ safe |
| undo commits, keep changes staged | `git reset --soft HEAD~1` | ✅ |
| undo commits, keep changes in files | `git reset HEAD~1` (= `--mixed`) | ✅ |
| undo commits and throw changes away | `git reset --hard HEAD~1` | ⚠️ rescue via `reflog` |
| one file as it was in an older commit | `git restore --source=<hash> <file>` | ⚠️ overwrites the file |
| stop tracking a file but keep it | `git rm --cached <file>` (folder: `-r`) | ✅ |
| copy one commit from another branch | `git cherry-pick <hash>` | ✅ new hash |
| cherry-pick conflict | resolve, `git add`, `git cherry-pick --continue` (or `--abort`) | |
| find "lost" commits | `git reflog` → `git reset --hard HEAD@{n}` or `git branch <name> <hash>` | 🛟 |
| committed on `main` instead of a branch (not pushed) | `git branch <new>`, `git reset --hard HEAD~<n>`, `git switch <new>` | |

**reset modes:** `--soft` keeps everything (staged) · `--mixed` keeps the files · `--hard` keeps nothing.

**Detached HEAD** (`git switch --detach <hash>`): fine for looking around. To keep commits made there: `git switch -c <branch>` *before* you leave. Back: `git switch main`.

### Stash: put unfinished work aside

| Command | Does |
|---|---|
| `git stash` | put tracked changes aside, working directory is clean |
| `git stash -u` | also new (untracked) files |
| `git stash list` | what's in the stash |
| `git stash pop` | take the newest entry back (and remove it) |
| `git stash apply` | take it back but keep it in the stash |

## 9. Working with others

| Situation | What to do |
|---|---|
| `push` rejected: *fetch first* / *non-fast-forward* | `git pull`, resolve conflicts if any, `git push`. ⚠️ **Never** `git push --force` on shared branches |
| *Need to specify how to reconcile divergent branches* | `git config --global pull.rebase false`, pull again |
| remote repo was created **with** a README | `git pull --allow-unrelated-histories`, then push. Next time create it empty |
| *Your local changes would be overwritten* (on `switch` / `pull`) | commit or `git stash` first |
| password/token was pushed | treat it as public: revoke it at the service. Cleaning history (`git filter-repo`) is a last resort |

## 10. Error messages → fix

| Message | Fix |
|---|---|
| `Permission denied (publickey)` | public key not on the host, wrong account, or agent empty (`ssh-add -l`) → [04](session-1-terminal/04-git-setup.md) |
| `ssh: connect to host … port 22: Connection timed out` | network blocks SSH → GitHub via port 443 (see 04, *Pitfalls*) or HTTPS |
| `fatal: not a git repository` | wrong folder → `pwd`, `cd` into your project |
| `nothing added to commit` | forgot `git add` |
| `error: src refspec main does not match any` | no commit yet, or branch is `master` → `git branch -m main` |
| `No such file or directory` | wrong folder (`pwd`) or typo (use `Tab`) |
| zsh: `no matches found` | wildcard matched nothing (e.g. empty folder) |
| zsh: `too many arguments` when pasting | `# comments` → `setopt interactivecomments` |
| stuck in an editor after `merge` / `revert` / `commit` | nano: `Ctrl+O`, `Enter`, `Ctrl+X` · Vim: `Esc`, `:wq`, `Enter` |
