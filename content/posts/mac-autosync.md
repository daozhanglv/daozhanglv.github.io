---
title: 'mac-autosync: a Mac config backup I never have to think about'
date: 2026-11-14T20:00:00+08:00
draft: false
description: 'A few hundred lines of Bash plus launchd that back up my dotfiles and Homebrew package list every night, and refuse to commit anything that looks like a secret.'
tags: ['macOS', 'Bash', 'Projects']
---
[mac-autosync](https://github.com/daozhanglv/mac-autosync) is a public template repository. You create your own private copy, list the config files you care about, run one install command, and from then on your Mac backs them up to Git every night without you doing anything.

This post explains why it exists and the decisions behind it.

## The problem

Config files such as `.zshrc` and `.gitconfig` change a little at a time, and I never remember to save them. Tools like [chezmoi](https://www.chezmoi.io) solve this well, but they expect you to run a command after every change and push the result yourself. A backup that depends on my memory is a backup I will not have on the day I need it.

So the requirement was short: one Mac, no daily action, and no chance of pushing a password to GitHub by accident.

## What happens every night

1. The files listed in `files.txt` are copied into the repository.
2. The list of installed Homebrew packages is written to `Brewfile`.
3. [gitleaks](https://github.com/gitleaks/gitleaks) scans the staged changes for passwords and tokens.
4. If the scan is clean, the changes are committed and pushed.
5. A second, optional job runs `brew update`, `brew upgrade` and `brew cleanup`.

On a new Mac, `bash install.sh --restore` copies the files back and reinstalls everything in the `Brewfile`.

## Design decisions

### Plain Bash and launchd, nothing else

The whole tool is five shell scripts, under 400 lines including comments, plus launchd, the scheduler that ships with macOS. There is no daemon and no binary to keep updated. It is written for Bash 3.2, the old version Apple still ships, so it runs on a fresh Mac before anything else is installed.

One detail took care: launchd starts jobs with a minimal `PATH`, so a script that works in the terminal can fail at night because it cannot find `brew` or `gitleaks`. The shared `lib/common.sh` adds Homebrew's locations before anything else runs.

### Secrets are blocked twice

An automatic push is only safe if it cannot leak. There are two layers:

- A fixed list of paths that are never backed up, even if you list them: SSH private keys, `*.pem`, `.aws/credentials`, `.netrc`, `.npmrc` and similar.
- A gitleaks scan of every change before it is committed. If it finds something, the commit is cancelled, nothing is pushed, and a macOS notification points to the log.

The order matters. The scan runs on staged changes before the commit, so a secret never enters Git history, not even locally.

### One direction only

Files are copied from the Mac into the repository, never the other way, except when you explicitly run `--restore`. This removes a whole class of problems: there is nothing to merge, and a nightly job can never overwrite a file I am editing. Even on restore, an existing file that differs is kept next to the new one as `<name>.bak`.

### Failures wait for the next night

Nobody is at the keyboard at 05:23, so the script cannot ask questions. A failed push is retried three times and then left alone; the commit stays local and the next run pushes it. Commits are deliberately unsigned, because signing can ask for Touch ID.

### A dry run before anything changes

`bash install.sh --check` prints what would be installed and which requirements are missing, and changes nothing. For a tool that installs background jobs, I wanted the first command a new user runs to be a harmless one.

## What it does not do

- Only single files, not folders.
- Only one Mac. There are no per-machine differences; chezmoi is the right tool for that.
- macOS only.

## Status

The scripts are a generalised version of the setup that runs daily on my own Mac. The generalised version passes shellcheck and simulated tests of backup, push retry, restore and the stop-on-secret path, but its launchd installation has not yet been run on a clean Mac. That is the next thing to verify.

The code and setup instructions are on [GitHub](https://github.com/daozhanglv/mac-autosync).
