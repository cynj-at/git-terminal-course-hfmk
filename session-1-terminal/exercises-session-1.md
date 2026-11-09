# Exercises — Session 1

> **How to use the setup blocks:** copy the whole grey block, paste it into your terminal and press Enter. It creates the starting situation for the exercise. You can always paste it again to start over.
>
> Pasting in Git Bash: `Shift + Insert` or right-click → *Paste*. Ctrl+V doesn't work there.
>
> **zsh users (macOS):** the solutions contain `# comments`. By default zsh doesn't accept comments typed at the prompt. Run `setopt interactivecomments` once, or better, add that line to your `~/.zshrc`.

---

## E1 — Scavenger hunt (navigation)

A friend sent you a messy project folder. Find your way around it.

**Setup:**
```bash
cd ~
rm -rf ~/hunt
mkdir -p ~/hunt/concert/scores/old ~/hunt/concert/audio ~/hunt/admin
echo "Brahms Op. 25 - parts" > ~/hunt/concert/scores/brahms.txt
echo "draft from last year" > ~/hunt/concert/scores/old/brahms-OLD.txt
echo "fake wav" > ~/hunt/concert/audio/rehearsal-01.wav
echo "fake wav" > ~/hunt/concert/audio/rehearsal-02.wav
echo "the treasure: 42" > ~/hunt/admin/.secret
echo "room booking: Tue 10:00" > ~/hunt/admin/booking.txt
cd ~/hunt
```

**Tasks:**
1. Where are you? Print the full path.
2. Without changing directory, list the contents of `concert/audio`.
3. Go into `concert/scores/old`. From there, go to `admin` with **one** `cd` and a **relative** path.
4. There's a hidden file in `admin`. Find it and print its content.
5. Create a folder `~/hunt/concert/archive` and **move** `brahms-OLD.txt` into it.
6. Rename `rehearsal-01.wav` to `2024-10-01-rehearsal.wav`.
7. Copy both `.wav` files into `archive` with a single command (wildcard).
8. Bonus: `find ~/hunt -name "*.txt"` — what does it do?

<details>
<summary>Solution</summary>

```bash
pwd                                       # 1
ls concert/audio                          # 2
cd concert/scores/old                     # 3
cd ../../../admin
ls -a                                     # 4
cat .secret
mkdir ~/hunt/concert/archive              # 5
mv ~/hunt/concert/scores/old/brahms-OLD.txt ~/hunt/concert/archive/
cd ~/hunt/concert/audio                   # 6
mv rehearsal-01.wav 2024-10-01-rehearsal.wav
cp *.wav ../archive/                      # 7
find ~/hunt -name "*.txt"                 # 8: lists all .txt files below ~/hunt, in all subfolders
```
</details>

---

## E2 — Pipes and redirection

**Setup:**
```bash
mkdir -p ~/pipes && cd ~/pipes
printf "Bach - Cello Suite 1\nBeethoven - Sonata 8\nBach - Partita 2\nChopin - Ballade 1\nBeethoven - Sonata 14\nBach - Goldberg Variations\nRavel - Bolero\n" > repertoire.txt
```

**Tasks:**
1. Show the file's content.
2. Show only the first 3 lines (`head -n 3`).
3. Show only the lines that contain "Bach" (`grep`).
4. **Count** how many pieces by Bach are in the list (combine `grep` and `wc -l` with a pipe).
5. Save all Beethoven lines into a new file `beethoven.txt`.
6. Append the line `Debussy - Clair de Lune` to `repertoire.txt` **without** overwriting it.
7. Bonus: show the list sorted alphabetically (`sort`).

<details>
<summary>Solution</summary>

```bash
cat repertoire.txt                              # 1
head -n 3 repertoire.txt                        # 2
grep Bach repertoire.txt                        # 3
grep Bach repertoire.txt | wc -l                # 4  → 3
grep Beethoven repertoire.txt > beethoven.txt   # 5
echo "Debussy - Clair de Lune" >> repertoire.txt  # 6
sort repertoire.txt                             # 7
```
</details>

---

## E3 — Make the shell yours

Build a small, tidy shell config with separate files.

**Tasks:**
1. Find out your shell (`echo $SHELL`) and therefore your config file (`~/.zshrc` or `~/.bashrc`).
2. Create the folder `~/.config/shell`.
3. Create `~/.config/shell/aliases.sh` with these aliases:
   - `ll` → `ls -la`
   - `..` → `cd ..`
   - `gs` → `git status`
4. Create `~/.config/shell/functions.sh` with a function `mkcd` that creates a folder and changes into it.
5. Add the loop that sources all files in `~/.config/shell/` to the **end** of your config file (see [03-shell-config.md](03-shell-config.md)).
6. Reload the config and test: `ll`, `mkcd ~/test-mkcd`, `pwd`, `..`, `type gs`.
7. Bonus: add a function `backup` that copies a file to the same name with `.bak` appended (`backup notes.txt` → `notes.txt.bak`).
8. Bonus: pick two or three helpers you like from the [real-world example](03-shell-config.md#a-real-world-example-nikos-zsh-setup) (`up`, `f`, `extract`, the git-branch prompt…) and add them to your files.
9. Bonus: write a `sos` function that prints an overview of *your* aliases and functions.

<details>
<summary>Solution</summary>

```bash
mkdir -p ~/.config/shell
```

`~/.config/shell/aliases.sh`:
```bash
alias ll='ls -la'
alias ..='cd ..'
alias gs='git status'
```

`~/.config/shell/functions.sh`:
```bash
mkcd() {
  mkdir -p "$1" && cd "$1"
}

backup() {
  cp "$1" "$1.bak"
}

sos() {
  cat <<'EOF'
Aliases:    ll  ..  gs
Functions:  mkcd <dir>  backup <file>
EOF
}
```

End of `~/.zshrc` / `~/.bashrc`:
```bash
for f in ~/.config/shell/*.sh; do
  [ -r "$f" ] && source "$f"
done
```

Reload and test:
```bash
source ~/.zshrc        # or ~/.bashrc
ll
mkcd ~/test-mkcd
pwd                    # → .../test-mkcd
..
type gs                # → "gs is an alias for git status" (bash: "gs is aliased to `git status'")
```
</details>

---

## E4 — Check your git configuration

1. Set your name, email, default branch, editor and `pull.rebase` as described in [04-git-setup.md](04-git-setup.md#step-1-tell-git-who-you-are).
2. Run `git config --list --show-origin`. Which file do your settings come from?
3. Print **only** your email: `git config user.email`.
4. Look at the file directly: `cat ~/.gitconfig`.

<details>
<summary>Solution</summary>

2: the lines start with `file:/home/<you>/.gitconfig` (macOS: `/Users/<you>/.gitconfig`). That's your global config. There may be other lines from system-wide config files.
</details>

---

## E5 — SSH, end to end

1. Create an SSH key (or check that you have one).
2. Add the public key to your git host.
3. Test the connection with `ssh -T git@<your-host>`.
4. On the website, create a **new repository** called `ssh-test` and tick *"Add a README"* (or *"Initialize repository"*).
5. Copy its **SSH** URL (the one starting with `git@`) and clone it:
   ```bash
   cd ~
   git clone git@<your-host>:<username>/ssh-test.git
   cd ssh-test
   ls
   ```

<details>
<summary>Solution / troubleshooting</summary>

See [04-git-setup.md](04-git-setup.md), steps 3–6 and *Pitfalls*. The most common problems:

- `Permission denied (publickey)` → the key isn't on the host, or isn't loaded (`ssh-add -l`).
- Cloned with an `https://` URL by mistake → delete the folder and clone again with the `git@` URL.
</details>
