# 04 — Setting up git and SSH

**Goals:** git knows who you are, and your computer can talk to GitHub / GitLab / Codeberg securely via **SSH** without asking for a password every time.

## Step 1: Tell git who you are

Every commit records an author. Set it once per computer:

```bash
git config --global user.name "Alex Example"
git config --global user.email "alex@example.org"
```

Use the **same email** as your account on the git host, so your commits are linked to your profile.

> 💡 **Privacy:** if you don't want your email to be public, GitHub, GitLab and Codeberg give you a "noreply" address in your account settings. You can use that one here.

A few useful defaults:

```bash
git config --global init.defaultBranch main     # new repos start with a branch called "main"
git config --global core.editor "nano"          # editor for commit messages (or: "code --wait")
git config --global pull.rebase false           # `git pull` merges (avoids a confusing question later)
```

Check everything:

```bash
git config --list --show-origin
```

`--global` settings live in `~/.gitconfig`. It's a plain text file, so feel free to look at it.

> ▶️ **Exercise checkpoint: E4 — Check your git configuration** · [open exercise](exercises-session-1.md#e4--check-your-git-configuration)

## Step 2: How does your computer prove who it is?

When you `push` (upload) to a server, the server must know it's really you. There are two ways:

| | **SSH** (we use this) | **HTTPS** |
|---|---|---|
| How | A key pair: private key on your computer, public key on the server | Username + token (passwords no longer work), usually stored by a helper |
| URL looks like | `git@github.com:alex/project.git` | `https://github.com/alex/project.git` |
| Works for | GitHub, GitLab, Codeberg, any server, the same way | Each host has its own login tool (`gh`, `glab`, …) |
| Set up | Once per computer | Once per computer and host |

### What's a key pair?

- **Private key** (`~/.ssh/id_ed25519`): stays on your computer. **Never share it, never upload it anywhere.**
- **Public key** (`~/.ssh/id_ed25519.pub`): you can give it to anyone. You put it on the git host.

The server checks that you hold the private key that matches the public key, without the private key ever leaving your machine. Think of it as a padlock (public) that only your key (private) can open.

## Step 3: Create an SSH key

First check whether you already have one:

```bash
ls ~/.ssh
```

If you see `id_ed25519` and `id_ed25519.pub`, skip to step 4. Otherwise:

```bash
ssh-keygen -t ed25519 -C "alex@example.org"
```

- *"Enter file in which to save the key"*: just press **Enter** (default location).
- *"Enter passphrase"*: a password that protects the key in case your laptop is stolen. **Recommended.** You won't see anything while typing, which is normal. You can also leave it empty and just press Enter.

## Step 4: Load the key into the SSH agent (so you type the passphrase only once)

If you didn't set a passphrase, you can skip this step.

**macOS**
Create or extend `~/.ssh/config` (e.g. `nano ~/.ssh/config`) with:
```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```
Then:
```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

**Windows (Git Bash)**
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
To avoid repeating this in every new terminal, add the two lines to your `~/.bashrc`.

**Linux**
Most desktops already run an agent. Just run:
```bash
ssh-add ~/.ssh/id_ed25519
```
If you get *"Could not open a connection to your authentication agent"*, run `eval "$(ssh-agent -s)"` first.

## Step 5: Put the public key on your git host

Print the **public** key (note the **`.pub`**!) and copy the whole line:

```bash
cat ~/.ssh/id_ed25519.pub
```

It looks like `ssh-ed25519 AAAAC3Nza...long...text alex@example.org`.

> Shortcuts to copy it directly to the clipboard:
> **macOS:** `pbcopy < ~/.ssh/id_ed25519.pub` · **Windows (Git Bash):** `clip < ~/.ssh/id_ed25519.pub`

Then paste it on your host's website:

| Host | Where |
|---|---|
| GitHub | Profile picture → *Settings* → *SSH and GPG keys* → *New SSH key* (<https://github.com/settings/keys>) |
| GitLab | Profile picture → *Edit profile* → *SSH Keys* → *Add new key* |
| Codeberg | Profile picture → *Settings* → *SSH / GPG Keys* → *Add key* (<https://codeberg.org/user/settings/keys>) |

Give it a title that names the computer, e.g. "Alex MacBook".

## Step 6: Test the connection

```bash
ssh -T git@github.com        # or git@gitlab.com, git@codeberg.org
```

The first time, you'll be asked *"Are you sure you want to continue connecting (yes/no)?"*. Type `yes` (the whole word). That saves the server's identity in `~/.ssh/known_hosts`.

Success looks like:

- GitHub: `Hi alex! You've successfully authenticated, but GitHub does not provide shell access.`
- GitLab: `Welcome to GitLab, @alex!`
- Codeberg: `Hi there, alex! You've successfully authenticated with the key named …`

🎉 That's it. From now on, clone with the **SSH URL** (the `git@…` one) and pushing just works.

> ▶️ **Exercise checkpoint: E5 — SSH, end to end** · [open exercise](exercises-session-1.md#e5--ssh-end-to-end)

> 🏁 **End of session 1.** Make sure `ssh -T` works. You'll need it in session 2.

## Optional: HTTPS with the GitHub CLI (`gh`)

If you prefer not to deal with keys and only use GitHub, the GitHub CLI can do the login for you:

```bash
gh auth login
```

Choose *GitHub.com* → *HTTPS* → *Login with a web browser*, and follow the instructions. `gh` then also configures git to use it for HTTPS pushes. Install `gh` from <https://cli.github.com>. *(Ask the tutor if you go this way.)*

## Pitfalls

- **`Permission denied (publickey)`:** the public key isn't on the host, the agent doesn't have the key (`ssh-add -l` lists loaded keys), or you're on the wrong account.
- **You pasted the private key:** the private key starts with `-----BEGIN OPENSSH PRIVATE KEY-----`. Delete it from the website immediately and create a new key pair.
- **Repo cloned via HTTPS but you set up SSH:** git asks for a username/password when you push. Switch the URL:
  `git remote set-url origin git@github.com:<username>/<repo>.git`
- **Typed `y` instead of `yes`** at the fingerprint question: just run the command again.
- **`ssh: connect to host … port 22: Connection timed out`:** some university or company networks block SSH's port 22. For GitHub, add this to `~/.ssh/config` and test again:
  ```
  Host github.com
    Hostname ssh.github.com
    Port 443
  ```
  GitLab offers the same via `altssh.gitlab.com`. If nothing helps, use HTTPS on that network.
