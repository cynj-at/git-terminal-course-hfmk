# Exercises — Session 2: real-life git scenarios

Every exercise tells a small story. Most of them start with a **setup block**: paste it into the terminal and it builds the situation (including the mistake) in its own folder under `~/git-course/`. Paste it again any time to start over.

> **Before you start:**
> - zsh users: run `setopt interactivecomments` so you can paste commands with `# comments`.
> - If git opens an editor (e.g. for a merge or revert message): in **nano** save with `Ctrl+O`, `Enter`, quit with `Ctrl+X`. In **Vim** type `Esc`, then `:wq`, `Enter`.
> - **`git status` is always allowed.** Use it after every step.

| # | Scenario | Commands you'll learn |
|---|---|---|
| S1 | My first repository | `init`, `add`, `commit`, `.gitignore`, `remote`, `push` |
| S2 | The daily routine | `diff`, partial staging, `log`, `clone`, `pull` |
| S3 | "I messed up a file / staged the wrong thing" | `restore`, `restore --staged` |
| S4 | "Typo in my commit and I forgot a file" | `commit --amend` |
| S5 | "A bug went out to everyone" | `revert` |
| S6 | "My last commits are a mess", "I deleted too much!" | `reset --soft/--mixed/--hard`, `reflog` |
| S7 | Working on a branch, merge conflict | `switch -c`, `merge`, conflict resolution |
| S8 ⭐ | "I need just *that one* commit" | `cherry-pick`, `restore --source`, detached HEAD |
| S9 ⭐ | "My shell config in git, with my password in it" | `.gitignore`, `rm --cached`, `commit --amend` |

⭐ = optional extra exercise.

---

## S1 — My first repository

*You're putting together the program for a concert and want to track it with git, and back it up online.*

**Tasks:**
1. Create the folder `~/git-course/concert-program`, go into it and turn it into a git repository.
2. Create a `README.md` with a heading and one sentence about the project.
3. Create a fake audio file: `echo "fake" > rehearsal.wav`. Audio exports should **not** be committed. Create a `.gitignore` that ignores all `.wav` files.
4. Check `git status`. Which files does git see? Is `rehearsal.wav` listed?
5. Stage and commit `README.md` and `.gitignore` with a good message.
6. Look at your history with `git log --oneline`.
7. On your git host, create a **new, empty** repository called `concert-program`. **Don't** tick "Add a README" (your repo already has a history).
8. Connect your local repo to it and push. The host shows you the commands. Use the **SSH** URL.
9. Refresh the web page. Your README is online. 🎉

> **No account / SSH not working yet?** Use a "fake server" on your own disk instead and take this path in step 8:
> ```bash
> git init --bare -b main ~/git-course/remote.git
> git remote add origin ~/git-course/remote.git
> git push -u origin main
> ```

<details>
<summary>Solution</summary>

```bash
mkdir -p ~/git-course/concert-program
cd ~/git-course/concert-program
git init

printf "# Concert program\n\nProgram and notes for the spring concert.\n" > README.md
echo "fake" > rehearsal.wav
echo "*.wav" > .gitignore

git status                     # shows README.md and .gitignore, NOT rehearsal.wav
git add README.md .gitignore
git commit -m "Add README and ignore audio exports"
git log --oneline

git remote add origin git@<your-host>:<username>/concert-program.git
git push -u origin main
```

If `git push` complains about `master` vs `main`: your branch is called `master` because `init.defaultBranch` wasn't set. Rename it with `git branch -m main`.
</details>

---

## S2 — The daily routine

*Next day. You continue working on the program, and later on a second computer.*

Continue in `~/git-course/concert-program` from S1.

**Tasks:**
1. Add a line to `README.md` (any text), **and** create a new file `program.md` with three pieces.
2. Run `git status` and `git diff`. Why doesn't `git diff` show `program.md`?
3. These are two separate changes, so make **two commits**: first stage and commit only `program.md`, then `README.md`. Check `git diff --staged` before each commit.
4. `git log --oneline` should now show 3 commits. Push them.
5. **Simulate a second computer:** clone your repository into another folder:
   ```bash
   cd ~/git-course
   git clone <the-same-url-as-origin> laptop2
   cd laptop2
   ```
   (If you used the fake server: `git clone ~/git-course/remote.git laptop2`.)
6. In `laptop2`, add a fourth piece to `program.md`, commit and push.
7. Go back to `~/git-course/concert-program`. Is the fourth piece there? Fetch it with `git pull`.

<details>
<summary>Solution</summary>

```bash
cd ~/git-course/concert-program
echo "Rehearsals are on Tuesdays." >> README.md
printf "1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n3. Ravel - Bolero\n" > program.md

git status          # README.md: modified; program.md: untracked
git diff            # only README.md. git diff doesn't show untracked (brand new) files

git add program.md
git diff --staged
git commit -m "Add first draft of the program"
git add README.md
git commit -m "Mention rehearsal day in README"
git log --oneline
git push

cd ~/git-course
git clone git@<your-host>:<username>/concert-program.git laptop2
cd laptop2
echo "4. Debussy - Clair de Lune" >> program.md
git commit -am "Add Debussy to the program"
git push

cd ~/git-course/concert-program
cat program.md      # still 3 pieces
git pull
cat program.md      # now 4 pieces
```

`git commit -am` = stage all *already tracked* modified files and commit. It does **not** pick up new files.
</details>

---

## S3 — "I messed up a file / staged the wrong thing"

*You accidentally wiped the content of `program.md` (not committed yet). On top of that, you staged your private notes, which should never end up in the repository.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/s3
mkdir -p ~/git-course/s3 && cd ~/git-course/s3
git init -q -b main
printf "# Concert program\n\n1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n3. Ravel - Bolero\n" > program.md
git add program.md
git commit -q -m "Add concert program"
echo "OOPS - deleted everything by accident" > program.md
echo "my private thoughts about the conductor" > private-notes.txt
git add private-notes.txt
git status
```

**Tasks:**
1. Read `git status` carefully. It even tells you the commands you need.
2. Get the committed version of `program.md` back.
3. Take `private-notes.txt` out of the staging area, but **keep the file** on disk.
4. Make sure you can never stage it accidentally again.
5. Finally, `git status` should only show `.gitignore` as new.

<details>
<summary>Solution</summary>

```bash
git restore program.md                  # 2: back to the last committed version
cat program.md
git restore --staged private-notes.txt  # 3: unstage, the file stays
ls                                      #    private-notes.txt is still there
echo "private-notes.txt" >> .gitignore  # 4
git status                              # 5: only .gitignore is untracked
```

⚠️ `git restore <file>` throws away uncommitted changes in that file. There's no undo for that.
</details>

---

## S4 — "Typo in my commit, and I forgot a file"

*You just committed, then noticed a typo in the message ("conert") **and** that `biography.md` should have been part of the same commit. You haven't pushed yet.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/s4
mkdir -p ~/git-course/s4 && cd ~/git-course/s4
git init -q -b main
echo "# Readme" > README.md
git add README.md
git commit -q -m "Add README"
printf "1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n" > program.md
echo "Alex Example studied cello in Freiburg." > biography.md
git add program.md
git commit -q -m "Add conert program"
git log --oneline
git status
```

**Tasks:**
1. Add `biography.md` to the **last** commit (don't create a new commit).
2. Fix the message to *"Add concert program and biography"*.
3. Check with `git log --oneline` (still 2 commits) and `git show --stat HEAD` (both files are in it).
4. Compare the commit's hash before and after. What do you notice?

<details>
<summary>Solution</summary>

```bash
git add biography.md
git commit --amend -m "Add concert program and biography"
git log --oneline
git show --stat HEAD
```

4: the hash **changed**. `--amend` doesn't edit the old commit, it *replaces* it with a new one. That's why you shouldn't amend commits you've already pushed: other people would still have the old one.

Only forgot the file but the message was fine? `git commit --amend --no-edit`.
</details>

---

## S5 — "A bug went out to everyone"

*Yesterday you committed "Speed up tempo", but you typed `tempo=600` instead of `tempo=66`. Since then you've made another commit, and everything has already been **pushed**, so your ensemble colleagues have it too. You can't rewrite history. You need to undo it publicly.*

**Setup:** (the setup also creates a "server" on your disk at `~/git-course/s5-remote.git`)
```bash
cd ~
rm -rf ~/git-course/s5 ~/git-course/s5-remote.git
mkdir -p ~/git-course
git init -q --bare -b main ~/git-course/s5-remote.git
git clone -q ~/git-course/s5-remote.git ~/git-course/s5
cd ~/git-course/s5
printf "tempo=60\nkey=D minor\n" > settings.txt
git add settings.txt
git commit -q -m "Add settings"
printf "tempo=600\nkey=D minor\n" > settings.txt
git commit -q -am "Speed up tempo"
echo "Program notes for the concert." > notes.md
git add notes.md
git commit -q -m "Add program notes"
git push -q -u origin main
git log --oneline
```

**Tasks:**
1. Find the bad commit. `git log --oneline` and `git show <hash>` help.
2. Undo **only** that commit, without removing "Add program notes" and without rewriting history.
3. Check: `settings.txt` says `tempo=60` again, `notes.md` still exists, and `git log --oneline` shows **4** commits.
4. Push the fix.
5. Bonus: now do the actual change properly: set `tempo=66`, commit, push.

<details>
<summary>Solution</summary>

```bash
git log --oneline
git show HEAD~1                # that's "Speed up tempo"
git revert HEAD~1              # or: git revert <hash>. An editor opens with a message, save and close
cat settings.txt               # tempo=60
ls                             # notes.md is still there
git log --oneline              # 4 commits: the newest is 'Revert "Speed up tempo"'
git push
```

`git revert --no-edit <hash>` skips the editor.
</details>

---

## S6 — "My last commits are a mess" and "I deleted too much!"

*You're composing. The last two commits are messy work-in-progress commits ("WIP"). You haven't pushed them.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/s6
mkdir -p ~/git-course/s6 && cd ~/git-course/s6
git init -q -b main
echo "Movement 1: Allegro" > piece.md
git add piece.md
git commit -q -m "Add movement 1"
echo "Movement 2: Adagio" >> piece.md
git commit -q -am "Add movement 2"
echo "Movement 3: first idea" >> piece.md
git commit -q -am "WIP"
echo "Movement 3: second idea" >> piece.md
git commit -q -am "WIP again"
git log --oneline
```

**Part A — tidy up (`--soft`):** combine the two WIP commits into **one** commit called *"Add movement 3 draft"*.
Hint: move the branch back 2 commits but keep the changes staged.

**Part B — throw away (`--hard`):** you listen to it again. Movement 3 is terrible. Remove the draft commit **and its changes** completely. `piece.md` should only contain movements 1 and 2.

**Part C — the rescue (`reflog`):** the next morning you change your mind. Movement 3 was actually great. Get the draft commit back.

**Part D — bonus (`--mixed`):** run `git reset HEAD~1` (no option = `--mixed`). What does `git status` show now? How is that different from `--soft`?

<details>
<summary>Solution</summary>

```bash
# Part A
git reset --soft HEAD~2
git status                      # piece.md is staged: the changes of both WIP commits
git commit -m "Add movement 3 draft"
git log --oneline               # 3 commits

# Part B
git reset --hard HEAD~1
cat piece.md                    # only movements 1 and 2
git log --oneline               # 2 commits

# Part C
git reflog                      # find the line "commit: Add movement 3 draft"
git reset --hard HEAD@{1}       # or: git reset --hard <hash from the reflog>
cat piece.md                    # movement 3 is back
git log --oneline               # 3 commits

# Part D
git reset HEAD~1
git status                      # piece.md: modified but NOT staged ("Changes not staged for commit")
```

- `--soft`: commit gone, changes **staged**
- `--mixed` (default): commit gone, changes **in the file, unstaged**
- `--hard`: commit gone, changes **gone** (but the commit is still in the reflog)
</details>

---

## S7 — Working on a branch, and a merge conflict

*You're making the poster for the spring concert. You try out changes on branches so `main` always stays "printable".*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/s7
mkdir -p ~/git-course/s7 && cd ~/git-course/s7
git init -q -b main
printf "Title: Spring Concert\nDate: 2026-05-01\nVenue: Small Hall\n" > poster.md
git add poster.md
git commit -q -m "Add poster"
```

**Part A — a branch without trouble**
1. Create and switch to a branch `add-program`.
2. Add the line `Program: Bach, Chopin, Ravel` at the end of `poster.md` and commit.
3. Switch back to `main`. Look at `poster.md`: the program line is gone. (Don't worry, it's safe on the branch.)
4. Merge `add-program` into `main`. Now the line is on `main` too.
5. Delete the branch `add-program`. It's merged, so you don't need it anymore.

**Part B — the conflict**
1. Create and switch to a branch `new-venue`. Change the venue to `Great Hall` and commit.
2. Switch back to `main`. Meanwhile the organisers decided something else: on `main`, change the venue to `Chamber Music Hall` and commit.
3. Look at the history: `git log --oneline --graph --all`. The two lines have forked.
4. Merge `new-venue` into `main`. 💥 **CONFLICT.**
5. Read `git status`, open `poster.md` in your editor and look at the markers `<<<<<<<`, `=======`, `>>>>>>>`.
6. Decide on `Great Hall`. Remove **all** markers so the file looks the way it should, then save.
7. Finish the merge: `git add poster.md`, then `git commit` (accept the suggested message).
8. Look at the graph again.

> Want to cancel a merge in progress instead? `git merge --abort` brings you back to the state before the merge.

<details>
<summary>Solution</summary>

```bash
# Part A
git switch -c add-program
echo "Program: Bach, Chopin, Ravel" >> poster.md
git commit -am "Add program to poster"
git switch main
cat poster.md                   # no program line
git merge add-program           # "Fast-forward": main simply moves forward
git branch -d add-program

# Part B
git switch -c new-venue
# edit poster.md: Venue: Great Hall
git commit -am "Move concert to Great Hall"
git switch main
# edit poster.md: Venue: Chamber Music Hall
git commit -am "Move concert to Chamber Music Hall"
git log --oneline --graph --all
git merge new-venue             # CONFLICT (content): Merge conflict in poster.md
git status                      # "both modified: poster.md"
```

`poster.md` during the conflict:
```
Title: Spring Concert
Date: 2026-05-01
<<<<<<< HEAD
Venue: Chamber Music Hall
=======
Venue: Great Hall
>>>>>>> new-venue
Program: Bach, Chopin, Ravel
```
- Between `<<<<<<< HEAD` and `=======`: the version on your current branch (`main`)
- Between `=======` and `>>>>>>> new-venue`: the version from the branch you're merging in

After editing:
```
Title: Spring Concert
Date: 2026-05-01
Venue: Great Hall
Program: Bach, Chopin, Ravel
```

```bash
git add poster.md
git commit                      # editor opens with "Merge branch 'new-venue'": save and close
git log --oneline --graph --all
```

Many editors (e.g. VS Code) show buttons like *Accept Current / Accept Incoming / Accept Both* for conflicts. They do the same thing for you.
</details>

---

## S8 ⭐ — "I need just *that one* commit"

*On the branch `rework` you've been experimenting with the score. In between, you also fixed a mistake in the credits, and that fix is needed on `main` **now** because the program booklet goes to print. The experiments aren't ready yet.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/s8
mkdir -p ~/git-course/s8 && cd ~/git-course/s8
git init -q -b main
printf "Bar 1: C E G\nBar 2: F A C\nBar 3: G B D\n" > score.txt
echo "Composer: Anonymous" > credits.txt
git add score.txt credits.txt
git commit -q -m "Initial score"
git switch -q -c rework
printf "Bar 1: C E G\nBar 2: F A C\nBar 3: G B D\nBar 4: C E G C\n" > score.txt
git commit -q -am "Add bar 4"
echo "Composer: Alex Example" > credits.txt
git commit -q -am "Fix composer name in credits"
printf "Bar 1: C Eb G\nBar 2: F Ab C\nBar 3: G B D\nBar 4: C Eb G C\n" > score.txt
git commit -q -am "Experiment: minor version"
git switch -q main
git log --oneline --graph --all
```

**Part A — cherry-pick**
1. You're on `main`. Find the hash of *"Fix composer name in credits"*.
2. Copy **only that commit** to `main`.
3. Check: `credits.txt` on `main` is fixed, `score.txt` on `main` still has only 3 bars.

**Part B — one file from the past**
1. Switch to `rework`. The minor-key experiment didn't work, but you want to keep the commit in the history.
2. Bring back `score.txt` **as it was in "Add bar 4"**, without touching anything else, and commit that.

**Part C — time travel (detached HEAD)**
1. Look at the project as it was in *"Initial score"*: `git switch --detach <hash>`.
2. Read `git status`. What does git say about `HEAD`?
3. Look around (`cat score.txt`), then go back to `rework`.

<details>
<summary>Solution</summary>

```bash
# Part A
git log --oneline rework        # look up the hash of "Fix composer name in credits"
git cherry-pick <hash>
cat credits.txt                 # Composer: Alex Example
cat score.txt                   # 3 bars, the experiments did NOT come along
git log --oneline --graph --all

# Part B
git switch rework
git log --oneline               # look up the hash of "Add bar 4"
git restore --source=<hash> score.txt
cat score.txt                   # major version with bar 4
git commit -am "Go back to major version of the score"

# Part C
git log --oneline               # hash of "Initial score"
git switch --detach <hash>
git status                      # "HEAD detached at <hash>"
cat score.txt                   # 3 bars, the original
git switch rework
```

The cherry-picked commit on `main` gets a **new hash**. It's a copy with the same changes, not the same commit. In older tutorials you'll see `git checkout <hash>` for Part C. It does the same thing.
</details>

---

## S9 ⭐ — "My shell config in git, oops, with my password in it"

*You want your shell config from session 1 in a git repository, so you can use it on every computer. Like in the [real-world example](../session-1-terminal/03-shell-config.md#a-real-world-example-nikos-zsh-setup), personal things live in `local/`: here an API token and a computer-specific alias. In a hurry you ran `git add .` and committed. Now the token is in the commit. Luckily you **haven't pushed yet**.*

**Setup:** (a practice copy, so your real config stays untouched)
```bash
cd ~
rm -rf ~/git-course/s9
mkdir -p ~/git-course/s9/shell/local && cd ~/git-course/s9/shell
printf "alias ll='ls -la'\nalias gs='git status'\n" > aliases.sh
printf 'mkcd() { mkdir -p "$1" && cd "$1"; }\n' > functions.sh
echo "export WEATHER_API_TOKEN=abc123-very-secret" > local/secrets.sh
echo "alias music='cd ~/Music'" > local/local.sh
git init -q -b main
git add .
git commit -q -m "Add shell config"
git show --stat HEAD
```

**Tasks:**
1. Confirm the problem: `git show HEAD` contains the token.
2. Remove the `local/` folder **from the repository**, but keep the files on your disk.
3. Make sure `local/` can never be added again.
4. Fix the existing commit so the token isn't in the history at all (it hasn't been pushed, so you may rewrite it).
5. Check: `git log -p | grep TOKEN` prints nothing, `ls local` still shows both files, and `git status` is clean.
6. Push to a new **private** repository on your host.
7. Do it for real: the same steps with your actual `~/.config/shell` from E3.

<details>
<summary>Solution</summary>

```bash
git show HEAD | grep TOKEN          # 1: there it is
git rm -r --cached local            # 2: --cached = remove from git only, keep the files
echo "local/" > .gitignore          # 3
git add .gitignore
git commit --amend -m "Add shell config"   # 4: replace the commit, now without local/
git log -p | grep TOKEN             # 5: no output
ls local                            #    local.sh  secrets.sh
git status                          #    nothing to commit, working tree clean
git remote add origin git@<your-host>:<username>/shell-config.git
git push -u origin main             # 6
```

**What if it had already been pushed?** Then treat the token as **public**: revoke it with the service and create a new one. Removing it from the history afterwards is possible (e.g. with `git filter-repo`), but it rewrites all commits, needs a force-push, and anyone who cloned the repo still has the old copy. Prevention (`.gitignore` *before* the first `git add .`) is much easier than the cure.
</details>

---

## Clean up

When you're done with everything:
```bash
rm -rf ~/git-course
```
(The repository on your git host remains. Delete it on the website if you like.)
