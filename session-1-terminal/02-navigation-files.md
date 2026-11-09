# 02 — Navigating and working with files

**Goals:** move around the file system, create/copy/move/delete files and folders, and work fast with tab completion and history.

## The file system is a tree

```
/                       ← root ("the very top")
├── home/  (macOS: Users/)
│   └── alex/           ← your home directory, short: ~
│       ├── Documents/
│       │   └── notes.txt
│       └── Music/
└── usr/
    └── bin/            ← many programs live here
```

> **Windows (Git Bash):** your `C:` drive is `/c/`, and your home directory is `/c/Users/<username>`, also reachable as `~`.

## Where am I? What's here?

```bash
pwd             # print working directory: where am I?
ls              # list: what's in here?
ls -l           # long format: size, date, permissions
ls -a           # also show hidden files (names starting with a dot)
ls -la ~        # both, for your home directory
```

## Moving around: `cd`

```bash
cd Documents    # go into Documents (relative to where you are)
cd ..           # one level up
cd ../Music     # up, then into Music
cd ~            # home directory (plain `cd` does the same)
cd /            # root of the file system
cd -            # back to the previous directory
```

### Absolute vs. relative paths

| Path | Starts at | Example |
|---|---|---|
| **absolute** | `/` (or `~`) | `/home/alex/Documents/notes.txt`, `~/Documents/notes.txt` |
| **relative** | where you are now | `Documents/notes.txt`, `../Music` |

Special names: `.` = this directory, `..` = the parent directory, `~` = your home directory.

## Creating, copying, moving, deleting

```bash
mkdir projects                  # make a directory
mkdir -p projects/a/b/c         # including all missing parent directories
touch notes.txt                 # create an empty file (or update its timestamp)
echo "Hello" > hello.txt        # write text into a file (overwrites)
echo "again" >> hello.txt       # append text
cat hello.txt                   # print the whole file
less long-file.txt              # page through a long file (q to quit)

cp hello.txt copy.txt           # copy a file
cp -r projects projects-backup  # copy a directory (-r = recursive)
mv copy.txt renamed.txt         # rename (= move to a new name)
mv renamed.txt projects/        # move into a directory

rm renamed.txt                  # delete a file
rm -r projects-backup           # delete a directory and everything in it
rmdir empty-dir                 # delete an empty directory
rm -i                           # interactive delete (safety net for big mistakes)
```

> ⚠️ **`rm` has no trash can.** What you delete is gone. Be extra careful with `rm -r` and `*`.

> ▶️ **Exercise checkpoint: E1 — Scavenger hunt** · [open exercise](exercises-session-1.md#e1--scavenger-hunt-navigation)

## Wildcards

```bash
ls *.txt          # all files ending in .txt
cp *.wav backup/  # copy all .wav files
ls track-0?.mp3   # ? = exactly one character: track-01.mp3, track-02.mp3 ...
```

## Pipes and redirection

Programs can be combined: the output of one becomes the input of the next.

```bash
ls -l | less                 # | pipe: send the output to less
history | grep git           # search your history for "git"
cat notes.txt | wc -l        # count lines
ls > filelist.txt            # > write output to a file (overwrite)
ls >> filelist.txt           # >> append
```

> ▶️ **Exercise checkpoint: E2 — Pipes and redirection** · [open exercise](exercises-session-1.md#e2--pipes-and-redirection)

## Working fast — the keys you'll use all the time

| Key | Effect |
|---|---|
| `Tab` | **Auto-complete** file names and commands. Press twice to show options |
| `↑` / `↓` | Previous / next command from history |
| `Ctrl + R` | Search your history (type part of a command, Enter to run) |
| `Ctrl + C` | Cancel the running command / discard the current line |
| `Ctrl + L` | Clear the screen (or `clear`) |
| `Ctrl + A` / `Ctrl + E` | Jump to the beginning / end of the line |

> 💡 **Use Tab constantly.** It saves typing, and it prevents typos: if Tab doesn't complete, the file probably doesn't exist where you think it does.

## Try it

1. Go to your home directory and show *all* files, including hidden ones.
2. Create `~/terminal-test/a/b`, go into `b` and check where you are with `pwd`.
3. Go back to `~/terminal-test` with a **relative** path.
4. Create a file `hello.txt` with some text, copy it to `a/`, then rename the copy.
5. Delete `~/terminal-test` completely.

## Pitfalls

- **"No such file or directory":** you're in a different directory than you think (`pwd`!), or there's a typo (Tab!).
- **Spaces in names:** `cd My Documents` fails. Use `cd "My Documents"` or `cd My\ Documents` (Tab does this for you).
- **`>` overwrites without asking.** `>>` appends.
