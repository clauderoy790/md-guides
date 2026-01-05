# Dotfiles Management with a Bare Git Repository

> Track your dotfiles with git without cluttering your home directory.

---

## Table of Contents

1. [Overview](#overview)
2. [Initial Setup (New Machine)](#initial-setup-new-machine)
3. [Restoring on a New Machine](#restoring-on-a-new-machine)
4. [Daily Workflow](#daily-workflow)
5. [Troubleshooting](#troubleshooting)
6. [Quick Reference](#quick-reference)

---

## Overview

This method uses a **bare git repository** to track dotfiles directly in your home directory without needing symlinks or a separate dotfiles folder.

### How It Works

- A bare repo lives at `~/.cfg/` (no working tree inside)
- An alias (`config`) sets the work tree to `$HOME`
- Only explicitly added files are tracked
- Untracked files are hidden from status output

### The Alias

Add this to your shell config (`~/.config/fish/config.fish` for fish, `~/.bashrc` for bash, `~/.zshrc` for zsh):

**Fish:**
```fish
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
```

**Bash/Zsh:**
```bash
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
```

---

## Initial Setup (New Machine)

Use this when setting up dotfiles tracking for the first time (no existing repo).

### 1. Create the Bare Repository

```bash
git init --bare $HOME/.cfg
```

### 2. Add the Alias

Add the alias to your shell config (see [The Alias](#the-alias) above), then reload:

**Fish:**
```fish
source ~/.config/fish/config.fish
```

**Bash/Zsh:**
```bash
source ~/.bashrc  # or ~/.zshrc
```

### 3. Hide Untracked Files

This prevents `config status` from showing every file in your home directory:

```bash
config config --local status.showUntrackedFiles no
```

### 4. Add Your Dotfiles

```bash
config add ~/.vimrc
config add ~/.config/fish/config.fish
# Add any other dotfiles you want to track
```

### 5. Commit and Push

```bash
config commit -m "initial dotfiles"

# Create a repo on GitHub first, then:
config remote add origin git@github.com:USERNAME/dotfiles.git
config push -u origin master
```

---

## Restoring on a New Machine

Use this when you already have a dotfiles repo and want to restore it on a new machine.

### Method 1: Clean Machine (Recommended)

If you don't have existing dotfiles you want to keep:

```bash
# 1. Clone as bare repo
git clone --bare git@github.com:USERNAME/dotfiles.git $HOME/.cfg

# 2. Add the alias to your current shell session temporarily
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'

# 3. Checkout the files
config checkout

# 4. Hide untracked files
config config --local status.showUntrackedFiles no

# 5. Add the alias permanently to your shell config
```

### Method 2: Keep Local Files (Overwrite Remote)

If you want to keep your local dotfiles and push them as the new version:

```bash
# 1. Create bare repo
git init --bare $HOME/.cfg

# 2. Add the alias (see above)

# 3. Add remote
config remote add origin git@github.com:USERNAME/dotfiles.git

# 4. Fetch history (doesn't touch local files)
config fetch origin

# 5. Hide untracked files
config config --local status.showUntrackedFiles no

# 6. Add your local files
config add ~/.vimrc
config add ~/.config/fish/config.fish

# 7. Commit
config commit -m "update config"

# 8. Force push (overwrites remote with your local files)
config push --set-upstream origin master --force
```

### Method 3: Backup and Replace

If checkout fails due to existing files:

```bash
# 1. Clone as bare repo
git clone --bare git@github.com:USERNAME/dotfiles.git $HOME/.cfg

# 2. Add alias temporarily
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'

# 3. Backup existing files that would be overwritten
mkdir -p ~/.config-backup
config checkout 2>&1 | grep -E "^\s+" | awk '{print $1}' | xargs -I{} mv {} ~/.config-backup/{}

# 4. Now checkout should work
config checkout

# 5. Hide untracked files
config config --local status.showUntrackedFiles no
```

---

## Daily Workflow

Once set up, use `config` like regular `git`:

### Check Status

```bash
config status
```

### Add Changes

```bash
# Add specific files
config add ~/.vimrc

# Add multiple files
config add ~/.vimrc ~/.config/fish/config.fish
```

### Commit

```bash
config commit -m "update vim config"
```

### Push

```bash
config push
```

### Pull Updates

```bash
config pull
```

### View History

```bash
config log
config log --oneline
```

### View Diff

```bash
# Unstaged changes
config diff

# Staged changes
config diff --cached
```

### List Tracked Files

```bash
config ls-files
```

---

## Troubleshooting

### "Not a git repository" Error

The `~/.cfg` directory doesn't exist. You need to either:
- Create it fresh: `git init --bare $HOME/.cfg`
- Clone from remote: `git clone --bare git@github.com:USERNAME/dotfiles.git $HOME/.cfg`

### Authentication Failed (HTTPS)

Switch to SSH:

```bash
config remote set-url origin git@github.com:USERNAME/dotfiles.git
```

### Checkout Conflicts with Existing Files

Either:
- Back up and remove the conflicting files first
- Use Method 2 above to keep local files and force push

### Alias Not Working

Make sure:
1. The alias is in your shell config file
2. You've reloaded the config: `source ~/.config/fish/config.fish`
3. You're using the correct shell (fish vs bash vs zsh)

### Status Shows Too Many Files

Run:

```bash
config config --local status.showUntrackedFiles no
```

---

## Quick Reference

### Setup Commands

```bash
git init --bare $HOME/.cfg                    # Create bare repo
config config --local status.showUntrackedFiles no  # Hide untracked
config remote add origin git@github.com:USER/dotfiles.git  # Add remote
```

### Daily Commands

```bash
config status                    # Check status
config add ~/.vimrc              # Stage file
config commit -m "message"       # Commit
config push                      # Push to remote
config pull                      # Pull from remote
config ls-files                  # List tracked files
config diff                      # View changes
```

### Recovery Commands

```bash
config log                       # View history
config checkout <commit> -- <file>  # Restore file from commit
config reset --hard HEAD         # Discard all local changes
```

### The Alias

**Fish** (`~/.config/fish/config.fish`):
```fish
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
```

**Bash** (`~/.bashrc`) / **Zsh** (`~/.zshrc`):
```bash
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
```

---

## References

- [Atlassian Dotfiles Tutorial](https://www.atlassian.com/git/tutorials/dotfiles)
- [Storing Dotfiles in a Git Repository](https://mjones44.medium.com/storing-dotfiles-in-a-git-repository-53f765c0005d)

---

*Keep your dotfiles safe across machines without symlinks or complex tools.*
