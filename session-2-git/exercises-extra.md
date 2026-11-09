# Extra exercises: merging, merge conflicts and special situations

More practice after the session-2 exercises. Each scenario is **self-contained**: paste its setup block and it builds the situation in its own folder under `~/git-course/`, including a fake "server" and a "colleague" where the story needs them. Paste the setup again any time to start over.

> **Before you start:**
> - You know `add`, `commit` and `switch`. Start with **X0** if merging is new to you. X1–X4 assume you've seen a conflict once ([S7](exercises-session-2.md#s7--working-on-a-branch-and-a-merge-conflict)).
> - zsh users: run `setopt interactivecomments` so you can paste commands with `# comments`.
> - If git opens an editor: in **nano** save with `Ctrl+O`, `Enter`, quit with `Ctrl+X`. In **Vim** type `Esc`, then `:wq`, `Enter`.
> - **`git status` is always allowed**, and during a conflict it's the most useful command there is.

| # | Scenario | You'll learn |
|---|---|---|
| X0 | Merging without conflicts | fast-forward, merge commit, `--no-ff`, `branch --merged`, `branch -d` |
| X1 | Two conflicts in one file, and a panic button | several conflict blocks, combining both sides, `merge --abort` |
| X2 | "Just take mine" / "just take theirs", including an audio file | `checkout --ours` / `--theirs`, binary conflicts |
| X3 | My push is rejected: a colleague was faster | `push` rejected, `pull` with conflict, push again |
| X4 | One side deleted the file, the other changed it | modify/delete conflict, `git rm` during a merge |
| X5 | Urgent fix, but I'm in the middle of something | `stash`, `stash -u`, `stash pop` |
| X6 | Oops, I committed on `main` instead of a branch | move commits to a new branch |
| X7 | `.gitignore` doesn't ignore my file | `rm --cached` for already tracked files |
| X8 | My experiment in "detached HEAD" is gone | find lost commits with `reflog`, `branch <name> <hash>` |
| X9 | I need just that one fix, but it conflicts | `cherry-pick` with conflict, `--abort`, `--continue` |

### Conflict vocabulary (for all exercises)

```
<<<<<<< HEAD
the version on the branch you're ON ("ours")
=======
the version from the branch you're MERGING IN ("theirs")
>>>>>>> other-branch
```

Resolving = edit the file until it looks the way it should, **delete all three marker lines**, then `git add <file>` and `git commit`.

---

## X0 — Merging without conflicts

*You're putting the concert program together. Each part is prepared on its own branch and then merged into `main`. Most merges just work, but they don't all look the same in the history.*

**Merging in one sentence:** `git merge <other>` brings the commits of `<other>` **into the branch you're on**. So first `git switch` to the branch that should *receive* the changes (usually `main`).

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x0
mkdir -p ~/git-course/x0 && cd ~/git-course/x0
git init -q -b main
printf "Program\n" > program.md
echo "Spring Concert" > poster.md
git add program.md poster.md
git commit -q -m "Start program and poster"
git switch -q -c intro
echo "1. Intro: Bach - Prelude" >> program.md
git commit -q -am "Add intro piece"
echo "   (played by the whole ensemble)" >> program.md
git commit -q -am "Note who plays the intro"
git switch -q main
git log --oneline --graph --all
```

**Part A — fast-forward**
1. You're on `main`. Look at the graph: `intro` is two commits ahead, and `main` has nothing new.
2. Merge `intro` into `main`. Which word does git print?
3. Look at the graph again. Is there a merge commit?

**Part B — a real merge (merge commit)**
1. Create a branch `finale` and add the line `2. Finale: Ravel - Bolero` to the end of `program.md`. Commit.
2. Switch back to `main`. Meanwhile the poster changes: add the line `Date: 2026-05-01` to `poster.md` and commit.
3. Look at the graph: the history has forked.
4. Merge `finale` into `main`. An editor opens with a message. Accept it.
5. Look at the graph and at `git show --stat HEAD`. How many parents does the new commit have? Do both files have their changes?

**Part C — `--no-ff`: keep the branch visible**
1. Create a branch `encore`, add `3. Encore: Brahms - Hungarian Dance 5` to `program.md`, commit.
2. Switch to `main` and merge `encore` with `--no-ff` and the message `Merge encore`. A fast-forward would be possible, so what's different?

**Part D — clean up branches**
1. Create a branch `experiment` with one commit (any change), then go back to `main`. Don't merge it.
2. Which branches are already merged into `main`? Which aren't? (`git branch --merged`, `git branch --no-merged`)
3. Delete all merged branches.
4. Try to delete `experiment` the same way. What does git say?

**Bonus — push the result** (with a fake server, like in S1):
```bash
git init -q --bare -b main ~/git-course/x0-server.git
git remote add origin ~/git-course/x0-server.git
git push -u origin main
```

<details>
<summary>Solution</summary>

```bash
# Part A
git log --oneline --graph --all
git merge intro                 # "Fast-forward"
git log --oneline --graph --all # a straight line, no merge commit: main simply moved forward to intro

# Part B
git switch -c finale
echo "2. Finale: Ravel - Bolero" >> program.md
git commit -am "Add finale"
git switch main
echo "Date: 2026-05-01" >> poster.md
git commit -am "Add date to poster"
git log --oneline --graph --all # forked: one commit on each side
git merge finale                # editor: "Merge branch 'finale'". Save and close
git log --oneline --graph --all # the two lines come together in a merge commit
git show --stat HEAD            # "Merge: abc1234 def5678" = two parents
cat program.md poster.md        # both changes are there

# Part C
git switch -c encore
echo "3. Encore: Brahms - Hungarian Dance 5" >> program.md
git commit -am "Add encore"
git switch main
git merge --no-ff -m "Merge encore" encore
git log --oneline --graph --all # a merge commit, although a fast-forward was possible

# Part D
git switch -c experiment
echo "4. Maybe: something by Debussy?" >> program.md
git commit -am "Experiment: Debussy"
git switch main
git branch --merged             # encore, finale, intro, * main
git branch --no-merged          # experiment
git branch -d intro finale encore
git branch -d experiment        # error: the branch 'experiment' is not fully merged
```

**The three kinds of merge:**

| | When | Result in the history |
|---|---|---|
| **Fast-forward** | `main` has no new commits since the branch started | `main` just moves forward. Straight line, no merge commit |
| **Merge commit** (three-way merge) | both sides have new commits | a new commit with **two parents** joins the lines |
| **`--no-ff`** | you *want* a merge commit even if a fast-forward would work | the branch stays visible as a "bubble" in the history. Many teams prefer this for features |

- `git branch -d` only deletes branches that are merged, so it's safe. `git branch -D experiment` deletes it anyway, and its commits are only reachable through the `reflog` after that. Use it only when you're sure.
- If both sides change the **same lines**, the merge stops with a conflict → X1–X4.
</details>

---

## X1 — Two conflicts in one file, and a panic button

*The concert poster again. The organiser (branch `organiser`) moved the date and raised the ticket price. At the same time the ensemble changed date, time, price and dress code on `main`. The organiser also updated the program file, which nobody else touched.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x1
mkdir -p ~/git-course/x1 && cd ~/git-course/x1
git init -q -b main
printf "Title: Spring Concert\nDate: 2026-05-01\nTime: 19:00\nVenue: Small Hall\nAddress: Main Street 1\nProgram: see program.md\nConductor: A. Example\nSoloist: B. Example\nDoors open: 18:30\nTickets: 10 EUR\nDress code: black\n" > poster.md
printf "1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n" > program.md
git add poster.md program.md
git commit -q -m "Add poster and program"
git switch -q -c organiser
printf "Title: Spring Concert\nDate: 2026-05-08\nTime: 19:00\nVenue: Small Hall\nAddress: Main Street 1\nProgram: see program.md\nConductor: A. Example\nSoloist: B. Example\nDoors open: 18:30\nTickets: 15 EUR\nDress code: black\n" > poster.md
printf "1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n3. Ravel - Bolero\n" > program.md
git commit -q -am "Organiser: new date, new price, add Ravel"
git switch -q main
printf "Title: Spring Concert\nDate: 2026-05-02\nTime: 20:00\nVenue: Small Hall\nAddress: Main Street 1\nProgram: see program.md\nConductor: A. Example\nSoloist: B. Example\nDoors open: 18:30\nTickets: 12 EUR\nDress code: concert black\n" > poster.md
git commit -q -am "Ensemble: date, time, price, dress code"
git log --oneline --graph --all
```

**Part A — the panic button**
1. Merge `organiser` into `main`. Read the output: which file has a conflict, which one merged automatically?
2. Open `poster.md` and look at it. Too much? Cancel the whole merge.
3. Check with `git status` and `cat poster.md` that everything is exactly as before the merge.

**Part B — resolve it properly**
1. Merge again.
2. `poster.md` has **two** conflict blocks. Resolve them with these decisions:
   - Date: the organiser's (`2026-05-08`), time: the ensemble's (`20:00`)
   - Tickets: the organiser's (`15 EUR`), dress code: the ensemble's (`concert black`)
3. Check: `grep -n "<<<<<<<\|>>>>>>>\|=======" poster.md` prints nothing.
4. Finish the merge. Did `program.md` get the Ravel line?

<details>
<summary>Solution</summary>

```bash
# Part A
git merge organiser             # CONFLICT (content): Merge conflict in poster.md
                                # program.md: "Auto-merging" with no conflict
git status                      # "both modified: poster.md", program.md already staged
git merge --abort
git status                      # nothing to commit, working tree clean
cat poster.md                   # the main version again

# Part B
git merge organiser
```

`poster.md` during the conflict (two separate blocks; the unchanged lines between them stay as they are):
```
Title: Spring Concert
<<<<<<< HEAD
Date: 2026-05-02
Time: 20:00
=======
Date: 2026-05-08
Time: 19:00
>>>>>>> organiser
Venue: Small Hall
Address: Main Street 1
Program: see program.md
Conductor: A. Example
Soloist: B. Example
Doors open: 18:30
<<<<<<< HEAD
Tickets: 12 EUR
Dress code: concert black
=======
Tickets: 15 EUR
Dress code: black
>>>>>>> organiser
```

After editing. Note that each block takes one line from **each** side:
```
Title: Spring Concert
Date: 2026-05-08
Time: 20:00
Venue: Small Hall
Address: Main Street 1
Program: see program.md
Conductor: A. Example
Soloist: B. Example
Doors open: 18:30
Tickets: 15 EUR
Dress code: concert black
```

```bash
grep -n "<<<<<<<\|>>>>>>>\|=======" poster.md    # no output
git add poster.md
git commit                      # accept the message "Merge branch 'organiser'"
cat program.md                  # yes, Ravel is there
git log --oneline --graph --all
```

A resolution doesn't have to be "one side or the other". You can combine both, or write something completely new.
</details>

---

## X2 — "Just take mine" / "just take theirs", including an audio file

*Two people re-recorded the intro (`intro.wav`) and changed the set list. For the recording, the new take from branch `take2` is clearly better. For the set list, your version on `main` is the right one. And git can't show you what changed inside a `.wav` file.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x2
mkdir -p ~/git-course/x2 && cd ~/git-course/x2
git init -q -b main
printf "RIFF\000\001WAVE take1" > intro.wav
printf "1. Intro\n2. Theme\n3. Finale\n" > setlist.md
git add intro.wav setlist.md
git commit -q -m "Add first take and set list"
git switch -q -c take2
printf "RIFF\000\002WAVE take2 - much better" > intro.wav
printf "1. Intro (new take)\n2. Theme\n3. Finale\n" > setlist.md
git commit -q -am "Re-record intro"
git switch -q main
printf "RIFF\000\003WAVE take1 - remastered" > intro.wav
printf "1. Intro\n2. Theme and variations\n3. Finale\n" > setlist.md
git commit -q -am "Remaster intro, extend theme"
```

**Tasks:**
1. Merge `take2` into `main`. How does git describe the conflict in `intro.wav`? Is there anything to open in an editor?
2. Take **their** version (`take2`) of `intro.wav` completely.
3. Take **your** version (`main`) of `setlist.md` completely, without editing it by hand.
4. Finish the merge and check both files.

<details>
<summary>Solution</summary>

```bash
git merge take2
# warning: Cannot merge binary files: intro.wav (HEAD vs. take2)
# CONFLICT (content): Merge conflict in intro.wav
# CONFLICT (content): Merge conflict in setlist.md

git checkout --theirs intro.wav       # 2: the version from take2
git checkout --ours setlist.md        # 3: the version from main
git add intro.wav setlist.md
git commit                            # accept the merge message
cat intro.wav                         # ...take2 - much better
cat setlist.md                        # Theme and variations
```

- **ours** = the branch you're on (`main`), **theirs** = the branch you're merging in (`take2`).
- Newer git also has `git restore --ours <file>` / `git restore --theirs <file>`. It does the same thing.
- Binary files (audio, images, PDF, Word) never get conflict markers. You can only pick one side. That's one reason big binary files are better kept out of git, or managed with **Git LFS**.
- ⚠️ During a **rebase**, `ours` and `theirs` are swapped. Always double-check with `cat` before you `add`.
</details>

---

## X3 — My push is rejected: a colleague was faster

*You and a colleague both work on the rehearsal plan. You changed Wednesday locally and committed. Meanwhile your colleague changed the same line **and pushed first**.*

**Setup:** (creates a fake server `server.git` and two clones: `me` and `colleague`)
```bash
cd ~
rm -rf ~/git-course/x3
mkdir -p ~/git-course/x3
git init -q --bare -b main ~/git-course/x3/server.git
git clone -q ~/git-course/x3/server.git ~/git-course/x3/me
cd ~/git-course/x3/me
printf "Rehearsal plan\nMon: strings\nWed: winds\nFri: tutti\n" > plan.md
git add plan.md
git commit -q -m "Add rehearsal plan"
git push -q -u origin main
git clone -q ~/git-course/x3/server.git ~/git-course/x3/colleague
cd ~/git-course/x3/colleague
printf "Rehearsal plan\nMon: strings\nWed: brass\nFri: tutti\n" > plan.md
git commit -q -am "Wed is brass, not winds"
git push -q
cd ~/git-course/x3/me
printf "Rehearsal plan\nMon: strings\nWed: winds and percussion\nFri: tutti\n" > plan.md
git commit -q -am "Add percussion on Wed"
git log --oneline
```

**Tasks:**
1. You're in `~/git-course/x3/me`. Push. What does git say, and why?
2. Get your colleague's changes. 💥 Conflict.
3. Agree on `Wed: brass and percussion`. Resolve, commit, push.
4. Switch to the colleague's folder (`cd ~/git-course/x3/colleague`) and get the result there. Do both now have the same `plan.md`?

<details>
<summary>Solution</summary>

```bash
git push
# ! [rejected]  main -> main (fetch first)
# hint: Updates were rejected because the remote contains work that you do not have locally.
# → the server has a commit (the colleague's) that you don't have yet. git won't overwrite it.

git pull                         # CONFLICT (content): Merge conflict in plan.md
```

If `git pull` instead complains *"Need to specify how to reconcile divergent branches"*, run `git config --global pull.rebase false` once (see [04](../session-1-terminal/04-git-setup.md#step-1-tell-git-who-you-are)) and pull again.

```
Rehearsal plan
Mon: strings
<<<<<<< HEAD
Wed: winds and percussion
=======
Wed: brass
>>>>>>> 1a2b3c4...
Fri: tutti
```
(When pulling, the lower part is marked with the commit hash from the server.)

```bash
# edit plan.md → "Wed: brass and percussion", delete the markers
git add plan.md
git commit                       # "Merge branch 'main' of ..."
git push                         # works now

cd ~/git-course/x3/colleague
git pull
cat plan.md                      # Wed: brass and percussion
```

**Rule of thumb:** `git pull` before you start working, and before you push. Then conflicts are rare and small.

> ⚠️ **Never** "solve" a rejected push with `git push --force`. That throws away your colleague's commit on the server.
</details>

---

## X4 — One side deleted the file, the other changed it

*Branch `cleanup` deletes the outdated `old-program.md`, because everything now lives in `program.md`. But on `main` someone just added an encore to exactly that old file.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x4
mkdir -p ~/git-course/x4 && cd ~/git-course/x4
git init -q -b main
printf "1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n" > old-program.md
printf "1. Bach - Cello Suite 1\n2. Chopin - Ballade 1\n3. Ravel - Bolero\n" > program.md
git add old-program.md program.md
git commit -q -m "Add programs"
git switch -q -c cleanup
git rm -q old-program.md
git commit -q -m "Remove outdated program"
git switch -q main
echo "4. Encore: Brahms - Hungarian Dance 5" >> old-program.md
git commit -q -am "Add encore"
```

**Tasks:**
1. Merge `cleanup` into `main`. What kind of conflict is this? What does `git status` call it?
2. Decision: the old file **should** be deleted, but the encore must not get lost. Move the encore line into `program.md`.
3. Finish the merge so that `old-program.md` is gone and `program.md` contains 4 pieces.

<details>
<summary>Solution</summary>

```bash
git merge cleanup
# CONFLICT (modify/delete): old-program.md deleted in cleanup and modified in HEAD.
git status                      # "deleted by them: old-program.md"

tail -n 1 old-program.md >> program.md   # 2: rescue the encore line
cat program.md                           #    4 pieces

git rm old-program.md           # 3: resolve the conflict as "deleted"
git add program.md
git commit
ls                              # only program.md
```

If you had decided to **keep** the file instead: `git add old-program.md`, then `git commit`.

There are no conflict markers in a modify/delete conflict. The question is simply *"keep or delete?"*, and you answer it with `git add` or `git rm`.
</details>

---

## X5 — Urgent fix, but I'm in the middle of something

*You're on branch `arrangement`, halfway through bar 4, and you have a new file with ideas. Nothing is ready to commit. Then the conductor calls: there's a typo on `main` ("Titel"), and it must be fixed **now**.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x5
mkdir -p ~/git-course/x5 && cd ~/git-course/x5
git init -q -b main
printf "Titel: Sonata\nBar 1: C E G\nBar 2: F A C\n" > score.md
git add score.md
git commit -q -m "Add score"
git switch -q -c arrangement
printf "Titel: Sonata\nBar 1: C E G\nBar 2: F A C\nBar 3: G B D\n" > score.md
git commit -q -am "Add bar 3"
printf "Titel: Sonata\nBar 1: C E G\nBar 2: F A C\nBar 3: G B D\nBar 4: C E (work in progress)\n" > score.md
echo "idea: add a cello part" > ideas.md
git status
```

**Tasks:**
1. Try `git switch main`. What happens and why?
2. Put your unfinished work aside with `git stash`. Check `git status`. Is `ideas.md` still there?
3. Run `git stash pop` to get everything back. Then stash again so that **new (untracked) files are included** too.
4. Switch to `main`, fix the typo (`Titel` → `Title`), commit.
5. Go back to `arrangement` and get your unfinished work back. Is bar 4 there? Is `ideas.md` there?

<details>
<summary>Solution</summary>

```bash
git switch main
# error: Your local changes to the following files would be overwritten by checkout: score.md
# → main has a different score.md, and switching would destroy your unsaved bar 4

git stash                       # 2
git status                      # ideas.md is still listed as untracked: stash ignores new files by default
git stash pop                   # 3: everything back
git stash -u                    #    -u = also stash untracked files
git status                      #    clean
git stash list                  #    stash@{0}: WIP on arrangement: ...

git switch main                 # 4
# edit score.md: Titel → Title  (or: sed -i.bak 's/Titel/Title/' score.md && rm score.md.bak)
git commit -am "Fix typo in title"

git switch arrangement          # 5
git stash pop
cat score.md                    # bar 4 is back
ls                              # ideas.md is back
```

- The stash is a **stack**: `git stash list` shows it, `git stash pop` takes the newest entry and removes it, `git stash apply` takes it but keeps it in the list.
- The stash is local and easy to forget. For anything longer than a quick interruption, a *work-in-progress commit* on your branch is often the better choice.
- Notice that the typo is still there on `arrangement`: the fix lives on `main`. To get it, `git merge main` while on `arrangement`.
</details>

---

## X6 — Oops, I committed on `main` instead of a branch

*You meant to work on movement 2 on a new branch. You forgot to create it, and now there are two draft commits directly on `main`. You haven't pushed yet.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x6
mkdir -p ~/git-course/x6 && cd ~/git-course/x6
git init -q -b main
echo "Movement 1: Allegro" > piece.md
git add piece.md
git commit -q -m "Add movement 1"
echo "Movement 2: draft A" >> piece.md
git commit -q -am "Draft movement 2"
echo "Movement 2: draft B" >> piece.md
git commit -q -am "Second draft of movement 2"
git log --oneline
```

**Tasks:**
1. Move the two draft commits to a new branch `movement-2`.
2. `main` should end at *"Add movement 1"* again.
3. Continue working on `movement-2`.
4. Check with `git log --oneline --graph --all`.

<details>
<summary>Solution</summary>

```bash
git branch movement-2           # 1: new branch label on the current commit (the drafts are now "on" it)
git reset --hard HEAD~2         # 2: move main back 2 commits. movement-2 still points to the drafts
git switch movement-2           # 3
cat piece.md                    #    drafts A and B are here
git log --oneline --graph --all # 4: main on "Add movement 1", movement-2 two commits ahead
```

Why this works: a branch is just a label. `git branch movement-2` puts a second label on the newest commit. Then `reset` moves only the `main` label back, and the commits stay reachable through `movement-2`.

⚠️ Only do this if the commits are **not pushed** yet. If they are, leave `main` alone and talk to your team.
</details>

---

## X7 — `.gitignore` doesn't ignore my file

*Your notation program writes a `render.log` on every export. It got committed by accident. You added `*.log` to `.gitignore` afterwards, but `git status` still shows the log file every time it changes.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x7
mkdir -p ~/git-course/x7 && cd ~/git-course/x7
git init -q -b main
echo "Rehearsal notes" > notes.md
echo "render 1: ok" > render.log
git add notes.md render.log
git commit -q -m "Add notes"
echo "*.log" > .gitignore
git add .gitignore
git commit -q -m "Ignore log files"
echo "render 2: ok" >> render.log
git status
```

**Tasks:**
1. Why does `git status` still show `render.log` even though `.gitignore` contains `*.log`?
2. Make git stop tracking `render.log`, but **keep the file** on your disk.
3. Commit that.
4. Test: append another line to `render.log`. `git status` should now stay clean.

<details>
<summary>Solution</summary>

1: `.gitignore` only applies to files git **doesn't track yet**. `render.log` was committed before the rule existed, so git keeps tracking it.

```bash
git rm --cached render.log      # 2: remove from git, keep on disk
ls                              #    render.log is still there
git status                      #    "deleted: render.log" (staged). That's correct, it's deleted from the *repository*
git commit -m "Stop tracking render.log"
echo "render 3: ok" >> render.log
git status                      # 4: nothing to commit, working tree clean
```

For a whole folder: `git rm -r --cached build/`.

⚠️ The file is still in the **old** commits. That's fine for a log file. For passwords, see [S9](exercises-session-2.md#s9---my-shell-config-in-git-oops-with-my-password-in-it).
</details>

---

## X8 — My experiment in "detached HEAD" is gone

*You went back to an older version of your piece to try a different ending, and committed it there. Then you switched back to `main`, and now the experiment is nowhere to be found: not in `git log`, not in `git branch`.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x8
mkdir -p ~/git-course/x8 && cd ~/git-course/x8
git init -q -b main
printf "Intro\n" > piece.md
git add piece.md
git commit -q -m "Add intro"
printf "Intro\nMiddle\n" > piece.md
git commit -q -am "Add middle"
printf "Intro\nMiddle\nEnding: loud\n" > piece.md
git commit -q -am "Add loud ending"
git switch -q --detach HEAD~1
printf "Intro\nMiddle\nEnding: quiet\n" > piece.md
git commit -q -am "Try quiet ending"
git switch -q main
git log --oneline --all
```

**Tasks:**
1. Convince yourself it's "gone": `git log --oneline --all`, `git branch`.
2. Find the commit *"Try quiet ending"* anyway.
3. Give it a branch called `quiet-ending`, so it can't get lost again.
4. Check with `git log --oneline --graph --all` and look at the file on that branch.

<details>
<summary>Solution</summary>

```bash
git log --oneline --all         # 1: 3 commits, no "quiet ending"
git branch                      #    only main

git reflog                      # 2: look for "commit: Try quiet ending" and note its hash
git branch quiet-ending <hash>  # 3: put a branch label on that commit

git log --oneline --graph --all # 4: quiet-ending forks off after "Add middle"
git switch quiet-ending
cat piece.md                    # Ending: quiet
```

- A commit without a branch isn't deleted right away, it's just invisible. The `reflog` remembers it (for about 90 days by default).
- To avoid the problem: when git says *"You are in 'detached HEAD' state"* and you want to keep your work, create a branch **before** you switch away: `git switch -c quiet-ending`.
- Git even warns you when you leave: *"Warning: you are leaving 1 commit behind, not connected to any of your branches"*, and suggests the exact `git branch <name> <hash>` command.
</details>

---

## X9 — I need just that one fix, but it conflicts

*On branch `rework` you're experimenting with a string arrangement. Along the way you also fixed a wrong note in bar 2 (`A` → `Ab`). That fix is needed on `main` **now**, but not the experiment. Meanwhile someone added a dynamic marking to bar 2 on `main`, so the same line has changed on both sides.*

**Setup:**
```bash
cd ~
rm -rf ~/git-course/x9
mkdir -p ~/git-course/x9 && cd ~/git-course/x9
git init -q -b main
printf "Title: Etude\nBar 1: C E G\nBar 2: F A C\nBar 3: G B D\nBar 4: C E G\n" > score.txt
git add score.txt
git commit -q -m "Add etude"
git switch -q -c rework
echo "Strings: tremolo everywhere" > arrangement.md
git add arrangement.md
git commit -q -m "Experiment: string arrangement"
printf "Title: Etude\nBar 1: C E G\nBar 2: F Ab C\nBar 3: G B D\nBar 4: C E G\n" > score.txt
git commit -q -am "Fix wrong note in bar 2"
git switch -q main
printf "Title: Etude\nBar 1: C E G\nBar 2: F A C (p)\nBar 3: G B D\nBar 4: C E G\n" > score.txt
git commit -q -am "Add dynamics to bar 2"
git log --oneline --graph --all
```

**Tasks:**
1. You're on `main`. Find the hash of *"Fix wrong note in bar 2"*.
2. Cherry-pick **only that commit** onto `main`. 💥 Conflict.
3. Read `git status`. What is git in the middle of? Cancel the cherry-pick and check that `main` is unchanged.
4. Cherry-pick again and resolve it: bar 2 needs **both** changes: `Bar 2: F Ab C (p)`.
5. Finish the cherry-pick (it's not `git commit` this time, see what `git status` suggests).
6. Check: `score.txt` on `main` has the fix and the dynamic marking, **no** `arrangement.md` came along, and `git log --oneline --graph --all` shows the fix as a new commit on `main`.

<details>
<summary>Solution</summary>

```bash
git log --oneline rework        # 1: note the hash of "Fix wrong note in bar 2"
git cherry-pick <hash>          # 2: CONFLICT (content): Merge conflict in score.txt
git status                      # 3: "You are currently cherry-picking commit <hash>." / "both modified: score.txt"
git cherry-pick --abort
git status                      #    clean
cat score.txt                   #    Bar 2: F A C (p), as before

git cherry-pick <hash>          # 4
```

`score.txt` during the conflict:
```
Title: Etude
Bar 1: C E G
<<<<<<< HEAD
Bar 2: F A C (p)
=======
Bar 2: F Ab C
>>>>>>> 1a2b3c4 (Fix wrong note in bar 2)
Bar 3: G B D
Bar 4: C E G
```
(During a cherry-pick, the lower part is labelled with the hash and message of the commit you're copying.)

After editing:
```
Title: Etude
Bar 1: C E G
Bar 2: F Ab C (p)
Bar 3: G B D
Bar 4: C E G
```

```bash
git add score.txt
git cherry-pick --continue      # 5: an editor opens with the original message. Save and close
cat score.txt                   # 6: Bar 2: F Ab C (p)
ls                              #    no arrangement.md: the experiment stayed on rework
git log --oneline --graph --all #    "Fix wrong note in bar 2" now exists twice: on rework and on main
```

- A cherry-pick **copies** the changes of one commit. The copy on `main` gets a new hash, and the original stays on `rework`.
- Conflicts work the same as with `merge`: edit, `git add`, then **`git cherry-pick --continue`** instead of `git commit`. Panic button: **`git cherry-pick --abort`**.
- Several commits at once: `git cherry-pick <hash1> <hash2>`. On a conflict, git stops at that commit, and `--continue` goes on with the rest.
- Tip: `git cherry-pick -x <hash>` adds *"(cherry picked from commit …)"* to the message, so you can see later where the copy came from.
- When `rework` is merged into `main` later, git usually notices the fix is already there. If both sides changed bar 2 again in the meantime, you may get the same conflict once more.
</details>

---

## Clean up

```bash
rm -rf ~/git-course/x0 ~/git-course/x0-server.git ~/git-course/x1 ~/git-course/x2 ~/git-course/x3 ~/git-course/x4 ~/git-course/x5 ~/git-course/x6 ~/git-course/x7 ~/git-course/x8 ~/git-course/x9
```
