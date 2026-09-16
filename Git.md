# Git Command Cheatsheet

A comprehensive reference for Git commands — what they do, how to use them, and examples.

---

## Table of Contents

1. [Configuration](#1-configuration)
2. [Getting Started](#2-getting-started)
3. [Staging & Committing](#3-staging--committing)
4. [Branching](#4-branching)
5. [Merging & Rebasing](#5-merging--rebasing)
6. [Remote Repositories](#6-remote-repositories)
7. [Inspecting History & Status](#7-inspecting-history--status)
8. [Undoing Changes](#8-undoing-changes)
9. [Stashing](#9-stashing)
10. [Tagging](#10-tagging)
11. [Diffing](#11-diffing)
12. [Cleaning](#12-cleaning)
13. [Cherry-picking](#13-cherry-picking)
14. [Submodules](#14-submodules)
15. [Advanced / Useful Tools](#15-advanced--useful-tools)

---

## 1. Configuration

### `git config`
Sets configuration values (user identity, editor, aliases, etc.) at system, global, or local (repo) level.

```bash
# Set your identity (required before committing)
git config --global user.name "Gaurav Sumeet"
git config --global user.email "gauravsumeet@gmail.com"

# Set default branch name for new repos
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "code --wait"

# View all config settings
git config --list

# View a specific setting
git config user.name
```
- `--global`: applies to all repos for the current user (stored in `~/.gitconfig`).
- `--local` (default): applies only to the current repo (stored in `.git/config`).
- `--system`: applies to all users on the machine.

---

## 2. Getting Started

### `git init`
Initializes a new, empty Git repository in the current directory (creates a `.git` folder).

```bash
git init
git init my-project      # creates a new folder "my-project" and initializes it there
```

### `git clone`
Copies an existing remote repository to your local machine, including full history.

```bash
git clone https://github.com/user/repo.git
git clone https://github.com/user/repo.git my-folder   # clone into a custom folder name
git clone --branch develop https://github.com/user/repo.git   # clone a specific branch
git clone --depth 1 https://github.com/user/repo.git   # shallow clone (latest commit only, faster)
```

---

## 3. Staging & Committing

### `git status`
Shows the state of the working directory and staging area — which files are modified, staged, or untracked.

```bash
git status
git status -s     # short format (e.g., "M file.txt", "?? newfile.txt")
```

### `git add`
Moves changes from the working directory into the staging area (index), preparing them for a commit.

```bash
git add file.txt          # stage a single file
git add file1.txt file2.txt   # stage multiple files
git add .                 # stage all changes in current directory and subdirectories
git add -A                # stage all changes in the entire repo (including deletions)
git add -p                # interactively choose hunks of changes to stage
```

### `git commit`
Saves the staged changes as a new snapshot (commit) in the repository history.

```bash
git commit -m "Add login feature"
git commit -am "Fix typo in README"   # stage all tracked, modified files AND commit in one step
git commit --amend -m "Updated message"   # edit the most recent commit's message
git commit --amend --no-edit   # add staged changes to the last commit without changing its message
```
> `-a` only stages files that are already tracked — it will NOT add new untracked files.

---

## 4. Branching

### `git branch`
Lists, creates, or deletes branches.

```bash
git branch                  # list local branches
git branch -a                # list all branches (local + remote)
git branch feature-login     # create a new branch (doesn't switch to it)
git branch -d feature-login  # delete a branch (safe — only if merged)
git branch -D feature-login  # force delete a branch (even if not merged)
git branch -m old-name new-name   # rename a branch
```

### `git switch`
Switches between branches (modern replacement for `git checkout` for branch switching).

```bash
git switch develop           # switch to existing branch "develop"
git switch -c feature-login  # create AND switch to a new branch
git switch -                 # switch back to the previously checked-out branch
```

### `git checkout`
Older, multi-purpose command for switching branches or restoring files. (Still widely used, though `switch`/`restore` are the newer, safer split of its responsibilities.)

```bash
git checkout develop         # switch to branch "develop"
git checkout -b feature-login   # create and switch to a new branch
git checkout main -- file.txt   # restore file.txt from the "main" branch into working dir
```

---

## 5. Merging & Rebasing

### `git merge`
Combines changes from one branch into the current branch, creating a merge commit if histories have diverged.

```bash
git switch main
git merge feature-login       # merges "feature-login" into "main"
git merge --no-ff feature-login   # force a merge commit even if fast-forward is possible
git merge --abort              # abort a merge in progress (e.g., due to conflicts)
```

### `git rebase`
Re-applies commits from your current branch on top of another branch, creating a linear history (rewrites commit hashes).

```bash
git switch feature-login
git rebase main                # replay feature-login's commits on top of main

git rebase -i HEAD~3            # interactive rebase: edit/squash/reorder the last 3 commits
git rebase --continue           # continue after resolving a conflict during rebase
git rebase --abort              # cancel the rebase and return to original state
```
> **Rule of thumb:** Never rebase commits that have already been pushed and shared with others — it rewrites history and can break collaborators' repos.

---

## 6. Remote Repositories

### `git remote`
Manages connections ("remotes") to other repositories (e.g., GitHub, GitLab).

```bash
git remote -v                       # list remotes with their URLs
git remote add origin https://github.com/user/repo.git   # add a new remote named "origin"
git remote remove origin            # remove a remote
git remote set-url origin https://github.com/user/new-repo.git   # change a remote's URL
```

### `git fetch`
Downloads commits, branches, and tags from a remote — but does NOT merge them into your working files.

```bash
git fetch origin
git fetch --all      # fetch from all configured remotes
```

### `git pull`
Fetches from a remote AND merges (or rebases) the changes into your current branch. Equivalent to `git fetch` + `git merge`.

```bash
git pull origin main
git pull --rebase origin main   # fetch and rebase instead of merge (keeps history linear)
```

### `git push`
Uploads your local commits to a remote repository.

```bash
git push origin main
git push -u origin feature-login   # push and set upstream tracking (so future "git push" alone works)
git push --force                   # overwrite remote history with your local branch (use with caution!)
git push --force-with-lease        # safer force-push — fails if remote has commits you don't have locally
git push --tags                    # push all local tags to the remote
git push origin --delete feature-login   # delete a remote branch
```

---

## 7. Inspecting History & Status

### `git log`
Shows the commit history.

```bash
git log
git log --oneline              # condensed, one line per commit
git log --oneline --graph --all   # visual graph of all branches
git log -n 5                   # show only the last 5 commits
git log --author="Gaurav"      # filter commits by author
git log --since="2 weeks ago"  # filter commits by date
git log -- file.txt            # show commits that touched a specific file
```

### `git show`
Displays details (metadata + diff) of a specific commit.

```bash
git show HEAD              # show the latest commit
git show a1b2c3d            # show a specific commit by hash
```

### `git blame`
Shows who last modified each line of a file, and in which commit.

```bash
git blame file.txt
git blame -L 10,20 file.txt   # only show lines 10–20
```

### `git reflog`
Shows a log of where `HEAD` has pointed over time (local only) — a lifesaver for recovering "lost" commits.

```bash
git reflog
git reset --hard HEAD@{2}   # recover to a previous state shown in reflog
```

---

## 8. Undoing Changes

### `git restore`
Restores working directory files to a previous state (modern replacement for parts of `checkout`).

```bash
git restore file.txt              # discard uncommitted changes to file.txt
git restore --staged file.txt     # unstage file.txt (keep the changes in working dir)
```

### `git reset`
Moves the current branch pointer (`HEAD`) to a different commit, optionally changing staged/working files.

```bash
git reset --soft HEAD~1     # undo last commit, keep changes staged
git reset --mixed HEAD~1    # undo last commit, keep changes unstaged (default mode)
git reset --hard HEAD~1     # undo last commit AND discard all changes (destructive!)
git reset a1b2c3d            # reset current branch to a specific commit
```

### `git revert`
Creates a NEW commit that undoes the changes of a previous commit — safe for shared/published history.

```bash
git revert HEAD              # revert the most recent commit
git revert a1b2c3d            # revert a specific commit by hash
git revert --no-commit HEAD~3..HEAD   # revert multiple commits without auto-committing
```
> Use `revert` (not `reset`) on commits that have already been pushed/shared.

---

## 9. Stashing

### `git stash`
Temporarily shelves (saves) uncommitted changes so you can work on something else, then reapply them later.

```bash
git stash                      # stash tracked changes
git stash -u                   # also stash untracked files
git stash list                 # list all stashes
git stash apply                # reapply the latest stash (keeps it in the stash list)
git stash pop                  # reapply the latest stash AND remove it from the list
git stash apply stash@{2}      # apply a specific stash
git stash drop stash@{0}       # delete a specific stash
git stash clear                # delete all stashes
git stash save "WIP: login form"   # stash with a custom message
```

---

## 10. Tagging

### `git tag`
Marks specific commits as important points in history (commonly used for releases like `v1.0.0`).

```bash
git tag                          # list all tags
git tag v1.0.0                   # create a lightweight tag on the current commit
git tag -a v1.0.0 -m "Release 1.0.0"   # create an annotated tag (recommended — stores author, date, message)
git tag -a v1.0.0 a1b2c3d -m "Tag old commit"   # tag a specific past commit
git push origin v1.0.0           # push a single tag to remote
git push origin --tags           # push all tags to remote
git tag -d v1.0.0                # delete a local tag
git push origin --delete v1.0.0  # delete a remote tag
```

---

## 11. Diffing

### `git diff`
Shows differences between commits, branches, staged changes, or the working directory.

```bash
git diff                     # unstaged changes vs. last commit
git diff --staged            # staged changes vs. last commit
git diff main develop        # compare two branches
git diff a1b2c3d..e4f5g6h     # compare two commits
git diff -- file.txt         # diff for a specific file only
```

---

## 12. Cleaning

### `git clean`
Removes untracked files from the working directory (irreversible — use with caution).

```bash
git clean -n     # dry run — show what WOULD be deleted, without deleting
git clean -f      # force-delete untracked files
git clean -fd     # also delete untracked directories
git clean -fx     # also delete files ignored by .gitignore
```

---

## 13. Cherry-picking

### `git cherry-pick`
Applies a specific commit from one branch onto another, without merging the whole branch.

```bash
git cherry-pick a1b2c3d          # apply a single commit onto the current branch
git cherry-pick a1b2c3d e4f5g6h  # apply multiple commits
git cherry-pick --continue        # continue after resolving conflicts
git cherry-pick --abort           # cancel the cherry-pick
```

---

## 14. Submodules

### `git submodule`
Allows including another Git repository as a subdirectory (dependency) inside your repository.

```bash
git submodule add https://github.com/user/lib.git libs/lib   # add a submodule
git submodule init                # initialize submodules after cloning a repo
git submodule update               # fetch submodule commits per parent repo's reference
git submodule update --init --recursive   # one-shot init + update (common after cloning)
git submodule update --remote      # pull latest changes from the submodule's remote
```

---

## 15. Advanced / Useful Tools

### `git bisect`
Uses binary search across commit history to find the exact commit that introduced a bug.

```bash
git bisect start
git bisect bad                # mark current commit as broken
git bisect good a1b2c3d        # mark a known-good commit
# Git checks out a commit in between — test it, then mark it:
git bisect good   # or
git bisect bad
# ...repeat until Git identifies the culprit commit
git bisect reset               # end the bisect session, return to original branch
```

### `git rm`
Removes files from both the working directory and the staging area (and stages the deletion).

```bash
git rm file.txt                # delete file and stage the removal
git rm --cached file.txt       # remove from Git tracking but KEEP the local file (e.g., to fix .gitignore mistakes)
git rm -r folder/              # recursively remove a directory
```

### `git mv`
Renames or moves a file and stages the change in one step.

```bash
git mv oldname.txt newname.txt
```

### `git archive`
Exports a snapshot of the repository (or a specific branch/tag) as a zip/tar file, without the `.git` history.

```bash
git archive --format=zip HEAD -o project.zip
git archive --format=tar.gz main -o main-branch.tar.gz
```

### `git worktree`
Lets you check out multiple branches into separate directories simultaneously, sharing the same `.git` history.

```bash
git worktree add ../hotfix-dir hotfix-branch   # create a new worktree for "hotfix-branch"
git worktree list                               # list all active worktrees
git worktree remove ../hotfix-dir               # remove a worktree
```

### `.gitignore`
Not a command, but a file listing patterns of files/folders Git should never track.

```gitignore
node_modules/
*.log
.env
dist/
.DS_Store
```

---

## Common Git Workflow Example

```bash
# 1. Clone the repo
git clone https://github.com/user/repo.git
cd repo

# 2. Create a feature branch
git switch -c feature-login

# 3. Make changes, then stage and commit
git add .
git commit -m "Add login form"

# 4. Keep branch up to date with main
git fetch origin
git rebase origin/main

# 5. Push the branch to remote
git push -u origin feature-login

# 6. After review, merge into main
git switch main
git pull origin main
git merge feature-login
git push origin main

# 7. Clean up
git branch -d feature-login
git push origin --delete feature-login
```

---

## Quick Reference Table

| Command | Purpose |
|---|---|
| `git init` | Create a new repository |
| `git clone <url>` | Copy a remote repository locally |
| `git status` | Show working directory state |
| `git add <file>` | Stage changes |
| `git commit -m "msg"` | Save staged changes |
| `git branch` | List/create/delete branches |
| `git switch <branch>` | Switch branches |
| `git merge <branch>` | Combine branch histories |
| `git rebase <branch>` | Replay commits on another base |
| `git pull` | Fetch + merge from remote |
| `git push` | Upload commits to remote |
| `git log` | View commit history |
| `git diff` | View uncommitted differences |
| `git stash` | Temporarily save changes |
| `git tag` | Mark a specific commit |
| `git reset` | Move HEAD / undo commits |
| `git revert` | Safely undo a commit with a new commit |
| `git cherry-pick` | Apply a specific commit elsewhere |
| `git bisect` | Binary-search for a bad commit |
