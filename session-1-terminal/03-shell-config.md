# 03 — Configuring your shell

**Goals:** understand where the shell reads its settings, and add your own aliases, functions and variables in a clean, organised way.

## The config file: your shell's "startup script"

Every time a new shell starts, it reads a config file and runs the commands in it. Everything you put there is available in every new terminal.

| Shell | File | Notes |
|---|---|---|
| zsh (macOS default) | `~/.zshrc` | |
| bash (Linux) | `~/.bashrc` | |
| bash (Git Bash on Windows) | `~/.bashrc` | loaded via `~/.bash_profile`, see the box below |

The leading dot makes the file **hidden**, so you need `ls -a` to see it.

> **Windows (Git Bash):** Git Bash reads `~/.bash_profile` at startup. If your `~/.bashrc` doesn't seem to be loaded, add this line to `~/.bash_profile`:
> ```bash
> if [ -f ~/.bashrc ]; then source ~/.bashrc; fi
> ```

### Open it in an editor

```bash
code ~/.zshrc        # VS Code (if the `code` command is installed)
nano ~/.zshrc        # simple terminal editor: Ctrl+O save, Enter, Ctrl+X quit
```

(Use `~/.bashrc` instead if you're on bash.) If the file doesn't exist yet, the editor creates it.

### Changes only apply to *new* shells

After you edit the file, either open a new terminal or reload the file in the current one:

```bash
source ~/.zshrc      # or: source ~/.bashrc
```

`source` means "run this file in **my current** shell". That's also how one config file can load another.

## Aliases — short names for long commands

```bash
alias ll='ls -la'
alias ..='cd ..'
alias gs='git status'
alias sz='source ~/.zshrc'
```

Now `ll` runs `ls -la`. Rules:
- **No spaces around `=`.**
- Use single quotes around the command.
- Type `alias` on its own to list all aliases. `type ll` shows what `ll` is.

## Functions — little programs with arguments

An alias can't take arguments in the middle. A function can. `$1` is the first argument, `$2` the second, and so on.

```bash
mkcd() {
  mkdir -p "$1" && cd "$1"
}
```

`mkcd new-project` creates the folder **and** changes into it. The `&&` means "only run the second command if the first one succeeded".

## Variables and `PATH`

```bash
export EDITOR=nano                       # programs (like git) ask $EDITOR which editor to use
export PATH="$HOME/bin:$PATH"            # also look for programs in ~/bin
echo $PATH                               # show the search path
```

`PATH` is a list of directories, separated by `:`. When you type a command like `git`, the shell searches these directories in order. `which git` shows you which one it found.

## Keeping it tidy: split your config into several files

Over time your config grows. A common pattern is one main file that **sources** smaller, topic-based files:

```
~/.zshrc                      ← main file, stays short
~/.config/shell/aliases.sh    ← all aliases
~/.config/shell/functions.sh  ← all functions
```

Add this to the end of your `~/.zshrc` (or `~/.bashrc`):

```bash
for f in ~/.config/shell/*.sh; do
  [ -r "$f" ] && source "$f"
done
```

This loads every `.sh` file in that folder. Want a new topic? Create a new file, with no need to touch `.zshrc` again. Because the files use only features that bash and zsh share, you can use the **same files with both shells**, and even copy them to a new computer or keep them in a git repository ("dotfiles").

> ▶️ **Exercise checkpoint: E3 — Make the shell yours** · [open exercise](exercises-session-1.md#e3--make-the-shell-yours)
>
> Everything below this box is reference material for later.

## A real-world example: niko's zsh setup (https://github.com/cynj-at/custom-terminal-settings-hfmk)

This is the same idea, grown over time and used every day (zsh only):

```
~/.zshrc                       ← loader only, 10 lines
~/.config/zsh/
├── options.zsh                ← shell behaviour (completion, history, globbing)
├── prompt.zsh                 ← what the prompt looks like
├── aliases.zsh                ← short names
├── functions.zsh              ← little helpers
├── help.zsh                   ← `sos`: a self-made help page
├── snippets                   ← favourite commands for Ctrl+G (see fzf below)
├── local.example.zsh          ← template for the per-computer file
├── .gitignore                 ← contains "local/"
└── local/                     ← per computer / per user, NOT shared
    ├── local.zsh              ← PATH, machine-specific aliases
    ├── arch.zsh               ← things that only make sense on this OS
    └── secrets.zsh            ← tokens, chmod 600, never shared
```

The whole `~/.zshrc`:

```zsh
# ~/.zshrc — loader only. Content lives in ~/.config/zsh/
ZSH_CONF="${ZSH_CONF:-$HOME/.config/zsh}"

for f in local/secrets local/arch local/local options prompt aliases functions help; do
  [[ -r "$ZSH_CONF/$f.zsh" ]] && source "$ZSH_CONF/$f.zsh"
done
unset f
```

What's worth copying:
- **The order is explicit.** `local/*` comes first, so `PATH` is set before anything else needs it. A wildcard (`*.zsh`) would load files alphabetically, which isn't always what you want.
- **`[[ -r … ]] &&`** skips files that don't exist. A new computer without `local/secrets.zsh` still starts without errors.
- **Shared vs. local.** Everything outside `local/` contains no passwords and no paths that only exist on one computer, so it can be copied to a server, given to a colleague, or put in a git repository. `local/` is listed in `.gitignore`. More on that in session 2.
- **`${ZSH_CONF:-default}`** means "use `$ZSH_CONF` if it's set, otherwise the default". That makes testing a different config folder easy.

### `options.zsh`: a few settings that make zsh nicer

```zsh
autoload -U compinit && compinit   # Tab completion for commands, options, git branches...
setopt correct                     # "zsh: correct 'gti' to 'git' [nyae]?"
setopt no_case_glob                # *.jpg also matches IMG.JPG
setopt extended_glob               # more powerful wildcards, e.g. **/*.wav = all .wav files in all subfolders

HISTFILE=~/.zsh_history            # remember commands across sessions
HISTSIZE=10000
SAVEHIST=10000
export EDITOR="nano"
```

### Handy aliases

```bash
alias sz='source ~/.zshrc'         # reload the config after editing it
alias cz='code ~/.zshrc'           # open the config in VS Code
alias c='clear'

alias gs='git status'
alias gd='git diff'
alias gdc='git diff --cached'      # what's staged?
alias glog='git log --oneline --graph --decorate'

# Only define the alias if the tool is installed (eza = a modern ls)
if command -v eza >/dev/null; then alias ll='eza -l'; else alias ll='ls -l'; fi
```

### Handy functions

```bash
# go up N directories: up 3 = cd ../../..
up() {
  local n=${1:-1} target="" i
  for ((i = 0; i < n; i++)); do target="../$target"; done
  cd "$target"
}

# timestamped backup: backup notes.md → notes.md.20261007143000.bak
backup() { cp "$1" "$1.$(date +%Y%m%d%H%M%S).bak"; }

# find files by (part of the) name, case-insensitive
f() { find . -iname "*$1*"; }

# search text in all files below this folder, with line numbers
g() { grep -Rn "$1" .; }

# the 10 biggest things in this folder
duh() { du -sh * | sort -rh | head -n 10; }

# serve the current folder at http://localhost:8000 (e.g. to test a website)
serve() { python3 -m http.server "${1:-8000}"; }

# unpack any archive with one command
extract() {
  if [ ! -f "$1" ]; then
    echo "'$1' is not a valid file"
    return 1
  fi
  case "$1" in
    *.tar.gz|*.tgz) tar xzf "$1" ;;
    *.tar.bz2)      tar xjf "$1" ;;
    *.tar)          tar xf "$1" ;;
    *.zip)          unzip "$1" ;;
    *.gz)           gunzip "$1" ;;
    *)              echo "Don't know how to extract '$1'"; return 1 ;;
  esac
}
```

Patterns you'll see in good functions:
- **`local`** variables, so the function doesn't change variables in your shell.
- **`${1:-default}`** gives an argument a default value.
- **Check the input and `return 1`** with a helpful message when something's wrong.
- **Quote `"$1"`**, so file names with spaces work.

### Your own help page

With 30 aliases and functions you'll forget half of them. One fix is a function that prints a short overview:

```bash
sos() {
  cat <<'EOF'
Navigation:   up [N]  mkcd <dir>
Files:        backup <file>  extract <archive>  f <name>  g <text>
Disk:         duh
Dev:          serve [port]
Git:          gs  gd  gdc  glog
EOF
}
```

`cat <<'EOF' … EOF` (a "here-document") prints everything between the two `EOF` lines as-is.

### A prompt that shows the git branch

This tells you at a glance where you are and which branch you're on, which is very useful in session 2.

**zsh** (in `~/.zshrc` or a `prompt.zsh`):
```zsh
setopt prompt_subst
git_branch() {
  local branch
  branch=$(git symbolic-ref --short HEAD 2>/dev/null) || return
  printf ' (%s)' "$branch"
}
PROMPT='%F{cyan}%~%f%F{magenta}$(git_branch)%f %# '
```

**bash** (in `~/.bashrc`):
```bash
git_branch() {
  local branch
  branch=$(git symbolic-ref --short HEAD 2>/dev/null) || return
  printf ' (%s)' "$branch"
}
PS1='\[\e[36m\]\w\[\e[35m\]$(git_branch)\[\e[0m\] \$ '
```

Result: `~/git-course/s7 (new-venue) %`. In zsh, `%~` is the current folder, `%F{color}…%f` sets the colour and `%#` is `%` (or `#` for root).

### fzf: fuzzy search for history, files and your own command snippets

[fzf](https://github.com/junegunn/fzf) is a small program that turns any list into a searchable menu. You type a few letters in any order (`gitlog` finds `git log --oneline --graph`), pick with `↑`/`↓` and press `Enter`. Hooked into the shell, it replaces three things you do all the time:

| Key | Without fzf | With fzf |
|---|---|---|
| `Ctrl + R` | search history, one match at a time | fuzzy list of your **whole history** |
| `Ctrl + T` | type a long path | pick a **file or folder** below the current one and insert its path |
| `Alt + C` | `cd` step by step | pick a **subfolder** and jump into it |
| `**` + `Tab` | normal completion | fuzzy completion, e.g. `cd **<Tab>`, `code **<Tab>` |

Inside the menu: `Esc` cancels. Some search tricks: `'exact` (exact match), `^start`, `end$`, `!exclude`.

**Install:**

| System | Command |
|---|---|
| macOS | `brew install fzf` |
| Linux | `sudo apt install fzf` · `sudo dnf install fzf` · `sudo pacman -S fzf` |
| Windows | `winget install fzf`. It works best in WSL; in Git Bash the key bindings only partly work |

**Turn it on** (fzf 0.48 or newer):

```zsh
# zsh, e.g. in options.zsh: Ctrl+R history, Ctrl+T files, Alt+C cd
if command -v fzf >/dev/null; then
  source <(fzf --zsh)
fi
```

```bash
# bash, in ~/.bashrc
if command -v fzf >/dev/null; then
  eval "$(fzf --bash)"
fi
```

> **Older fzf** (`fzf --version` below 0.48, e.g. the package in Ubuntu 24.04 and older): `--zsh` / `--bash` don't exist yet. Source the files that come with the package instead, e.g. on Debian/Ubuntu `source /usr/share/doc/fzf/examples/key-bindings.zsh` (or `.bash`).

#### Ctrl+G: your own command cheat sheet

Some commands you need only every few months, and you never remember them (rsync flags, ffmpeg options…). Collect them in a text file, one per line, with a description after `#`:

```
# ~/.config/zsh/snippets: commands for Ctrl+G. One command per line, "  # description" after it.
git log --oneline --graph --all   # history of all branches as a graph
git push -u origin BRANCH   # first push of a new branch
ffmpeg -i INPUT.wav -b:a 320k OUTPUT.mp3   # wav → mp3
ffmpeg -i INPUT.mp4 -vn -c:a copy OUTPUT.m4a   # extract the audio track from a video
rsync -avn SRC/ DEST/   # dry run: what would be copied?
du -sh * | sort -rh | head   # biggest things in this folder
```

`Ctrl + G` opens the list in fzf. You search by command **or** description ("mp3", "dry run"), and the chosen command is **inserted into the prompt without the description, but not run**. You replace the `CAPITALS` placeholders and press `Enter` yourself.

**zsh** (in `functions.zsh`). Machine-specific snippets go in `local/snippets`:
```zsh
# Ctrl+G: fuzzy-pick a command from snippets (+ local/snippets) into the prompt
if command -v fzf >/dev/null; then
  _snippet-insert() {
    local sel
    sel=$(cat "$ZSH_CONF/snippets" "$ZSH_CONF/local/snippets" 2>/dev/null \
      | grep -v -e '^\s*$' -e '^\s*#' | fzf --height 40% --reverse) || { zle redisplay; return; }
    setopt localoptions extendedglob
    LBUFFER+="${sel%%[[:space:]][[:space:]]##\#*}"   # drop "  # description"
    zle redisplay
  }
  zle -N _snippet-insert
  bindkey '^G' _snippet-insert
fi
```

> **Not using the `~/.config/zsh` layout from above?** The widget reads `$ZSH_CONF/snippets`. With the `~/.config/shell/` folder from E3, replace `$ZSH_CONF` with `~/.config/shell` (and drop `"$ZSH_CONF/local/snippets"` if you don't have a `local/` folder). Otherwise Ctrl+G opens an empty menu.

- `zle` is zsh's line editor. A function registered with `zle -N` can read and change what you're typing (`LBUFFER` = the text left of the cursor), and `bindkey` puts it on a key.
- `grep -v` drops empty lines and comment lines, so the file can have headings.
- `${sel%%[[:space:]][[:space:]]##\#*}` cuts off everything from the first "two or more spaces + `#`": that's the description.

**bash** (in `~/.bashrc` / `~/.zshrc`, with the snippets in `~/.config/shell/snippets`):
```bash
_snippet_insert() {
  local sel
  sel=$(grep -v -e '^\s*$' -e '^\s*#' ~/.config/shell/snippets | fzf --height 40% --reverse) || return
  sel="${sel%%  #*}"                       # drop "  # description"
  sel="${sel%"${sel##*[![:space:]]}"}"     # drop trailing spaces
  READLINE_LINE="${READLINE_LINE:0:READLINE_POINT}$sel${READLINE_LINE:READLINE_POINT}"
  READLINE_POINT=$((READLINE_POINT + ${#sel}))
}
bind -x '"\C-g": _snippet_insert'
```

## Try it

→ Exercise **E3** in [exercises-session-1.md](exercises-session-1.md).

## Pitfalls

- **`alias ll = 'ls -la'`** (with spaces) doesn't work. No spaces around `=`.
- **macOS: `code: command not found`:** open VS Code, press `Cmd+Shift+P` and run *Shell Command: Install 'code' command in PATH*. Or use `nano`.
- **"Nothing changed":** you forgot `source ~/.zshrc` or to open a new terminal.
- **zsh: `no matches found: ~/.config/shell/*.sh`:** the folder is empty or missing. Create at least one `.sh` file in it first.
- **Edited the wrong file:** check `echo $SHELL`. zsh doesn't read `.bashrc` and bash doesn't read `.zshrc`.
- **The command isn't installed everywhere:** an alias like `ll='eza -l'` breaks on a computer without `eza`. Guard it with `command -v` (see above).
- **OS-specific commands** (`open -a Preview`, `sysctl -n hw.ncpu` on macOS, `ip addr` on Linux) belong in a per-computer file like `local/local.zsh`, not in the shared ones.
- **Don't reuse names that already exist.** zsh has a built-in `r` (repeat the last command), and `path` is a special zsh variable linked to `PATH`. Writing `local path=../..` inside a function sets your `PATH` to `../..` until the function ends, so every external command in it fails. Check first with `type <name>`.
- **Test your aliases once.** `alias x='git log --oneline head -10'` looks fine but fails (`head` isn't a git option). The right form is `git log --oneline -10`.
- **`Alt + C` does nothing on macOS:** the terminal uses Option for special characters. Terminal: *Settings → Profiles → Keyboard → Use Option as Meta key*. iTerm2: *Profiles → Keys → Left Option key: Esc+*.
- **`unknown option: --zsh`:** your fzf is older than 0.48, see the box in the fzf section.
- **Broken config, the terminal shows errors on start:** open the file in an editor and comment out (`#`) your last change. As a last resort, rename it: `mv ~/.zshrc ~/.zshrc.broken`.
