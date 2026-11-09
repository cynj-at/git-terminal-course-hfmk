# 01 — Terminal basics

**Goals:** know what a terminal, a shell and a prompt are, and which shell you're using.

## Terminal, shell, prompt — what's the difference?

| Term | What it is | Analogy |
|---|---|---|
| **Terminal** (emulator) | The window that shows text and takes your keyboard input | The phone |
| **Shell** | The program inside that interprets your commands and runs other programs | The person you talk to on the phone |
| **Prompt** | The text the shell prints when it's waiting for input, e.g. `alex@laptop ~ %` | "Hello?" |

So you *open a terminal*, a *shell* starts inside it, and the shell shows a *prompt*.

## Why use a terminal at all?

- **Precise:** one command says exactly what happens. That's easy to document, repeat and share.
- **Automatable:** whatever you can type, you can put in a script.
- **Some tools exist only there:** git, servers, build tools, many scientific and audio tools.
- **Remote work:** on a server there's often nothing *but* a terminal.

## Common shells

| Shell | Where you'll meet it |
|---|---|
| **bash** | Default on most Linux systems and in **Git Bash** on Windows |
| **zsh** | Default on **macOS** since 2019; popular on Linux (often with "Oh My Zsh") |
| **fish** | Friendly, colourful shell. Its syntax differs from bash/zsh |
| **PowerShell** / **cmd** | Windows' own shells. Different commands. |

bash and zsh are ~95 % identical for everyday use. Everything in this course works in both.

## Which shell am I using?

```bash
echo $SHELL     # your login shell, e.g. /bin/zsh or /usr/bin/bash
echo $0         # the shell running right now, e.g. -zsh or bash
```

## Change shell

To make zsh (or bash) your default shell:

```bash
chsh -s $(which zsh)     # asks for your password; applies to new terminals / after logging in again
```

- No `sudo` needed: you're changing your own user's shell.
- **macOS** already uses zsh. **Windows (Git Bash)** has no `chsh`; Git Bash is always bash.
- **Linux:** install zsh first if it's missing (e.g. `sudo apt install zsh`). `chsh` only accepts shells listed in `/etc/shells`.

## Anatomy of a command

```
ls  -l  -a  Documents
│   └──┬──┘ └───┬───┘
│   options   argument
command
```

- **Command:** the program to run (`ls` = list).
- **Options/flags:** change behaviour. Usually start with `-` (short: `-l`) or `--` (long: `--all`). Short options can be combined: `-la`.
- **Arguments:** what the command works on (a file, a folder, text…).

Spaces separate the parts. That's why file names **with spaces** need quotes: `ls "My Documents"`.

## Getting help

```bash
man ls          # the manual (scroll with arrow keys, quit with q)
ls --help       # short help (Linux / Git Bash)
```

> **macOS:** `--help` often doesn't work for built-in tools, so use `man`.
> **Windows (Git Bash):** `man` isn't available, so use `--help`.

> ⏱ **Together, 2 min:** everyone runs the four *Try it* steps, then straight on to [02](02-navigation-files.md).

## Try it

1. Open your terminal.
2. Find out which shell you're using.
3. Run `whoami`, `date` and `echo Hello terminal`.
4. Open `man ls` (or `ls --help`) and find out what `-a` does. Quit with `q`.

## Pitfalls

- **Nothing happens / the prompt is gone:** a program is still running or waiting for input. Press `Ctrl + C` to cancel.
- **Stuck in a screen you can't leave:** try `q` (manual pages, `less`), then `Ctrl + C`. In *Vim*, type `Esc`, then `:q!`, Enter.
- **Upper/lower case matters:** `Documents` and `documents` are two different names (except on macOS/Windows file systems, which usually ignore case. Don't rely on that).
