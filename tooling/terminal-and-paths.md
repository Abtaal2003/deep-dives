# Terminal and Paths — A Practical Guide

A reference for the Linux/macOS command line: how filesystem paths work, the commands you'll actually use day to day, and the patterns that combine them. Examples are shown with the command on top and the (typical) output below, so you can see what to expect.

> **Windows users:** the commands here are bash/zsh — the shells used on Linux and macOS. On Windows, run them inside **WSL** (Windows Subsystem for Linux) or **Git Bash**. PowerShell uses different commands (`Get-ChildItem` instead of `ls`, etc.) and isn't covered here.

---

## How to read this guide

Examples follow this shape:

```bash
$ command --with-flags argument
expected output
```

The `$` is the **shell prompt** — you don't type it, it's just there to show what you'd type. Lines without `$` are output from the command.

---

## Table of Contents

**Foundations**
- [1. What Is a Terminal?](#1-what-is-a-terminal)
- [2. Anatomy of a Command](#2-anatomy-of-a-command)
- [3. The Filesystem as a Tree](#3-the-filesystem-as-a-tree)

**Paths**
- [4. Absolute Paths](#4-absolute-paths)
- [5. Relative Paths](#5-relative-paths)
- [6. The Special Path Symbols](#6-the-special-path-symbols)
- [7. The Home Directory and `~`](#7-the-home-directory-and-)
- [8. Path Resolution Rules](#8-path-resolution-rules)
- [9. Hidden Files (Dotfiles)](#9-hidden-files-dotfiles)
- [10. File Extensions Don't Matter (Mostly)](#10-file-extensions-dont-matter-mostly)

**Navigating**
- [11. Where Am I? (`pwd`)](#11-where-am-i-pwd)
- [12. Listing Files (`ls`)](#12-listing-files-ls)
- [13. Changing Directories (`cd`)](#13-changing-directories-cd)
- [14. Tab Completion](#14-tab-completion)

**Working with Files and Folders**
- [15. Creating Files (`touch`) and Folders (`mkdir`)](#15-creating-files-touch-and-folders-mkdir)
- [16. Copying (`cp`)](#16-copying-cp)
- [17. Moving and Renaming (`mv`)](#17-moving-and-renaming-mv)
- [18. Deleting (`rm`)](#18-deleting-rm)
- [19. Viewing File Contents (`cat`, `less`, `head`, `tail`)](#19-viewing-file-contents-cat-less-head-tail)

**Finding and Searching**
- [20. Finding Files by Name (`find`)](#20-finding-files-by-name-find)
- [21. Searching Inside Files (`grep`)](#21-searching-inside-files-grep)
- [22. Wildcards (Globs)](#22-wildcards-globs)

**Combining Commands**
- [23. Pipes (`|`)](#23-pipes-)
- [24. Redirects (`>`, `>>`, `<`)](#24-redirects---)
- [25. Chaining (`&&`, `||`, `;`)](#25-chaining---)

**System Awareness**
- [26. Processes (`ps`, `top`, `kill`)](#26-processes-ps-top-kill)
- [27. Disk Usage (`du`, `df`)](#27-disk-usage-du-df)
- [28. Environment Variables](#28-environment-variables)
- [29. Permissions (`chmod`, `chown`)](#29-permissions-chmod-chown)

**Quality of Life**
- [30. Command History](#30-command-history)
- [31. Keyboard Shortcuts](#31-keyboard-shortcuts)
- [32. Getting Help (`man`, `--help`)](#32-getting-help-man---help)
- [33. Aliases](#33-aliases)

**Reference**
- [34. Command Cheat Sheet](#34-command-cheat-sheet)
- [35. Common Mistakes](#35-common-mistakes)

---

## 1. What Is a Terminal?

A **terminal** is a text interface to your computer. You type a command, the computer runs it, and you see the result as text. That's the whole loop.

A few words that get used interchangeably (and shouldn't):

| Term | What it actually is |
| :--- | :--- |
| **Terminal** | The window/app you type into (Terminal.app, iTerm2, Windows Terminal, GNOME Terminal). |
| **Shell** | The program *inside* the terminal that interprets your commands. Common shells: **bash**, **zsh**, **fish**. |
| **Console** | An older term, sometimes synonymous with terminal. Sometimes refers to the system-level text interface before any GUI loads. |
| **Command line** | A general phrase covering all of the above — "the place where you type commands." |

For this guide, assume **bash** or **zsh** (macOS uses zsh by default since 2019; most Linux distros use bash).

---

## 2. Anatomy of a Command

Every command follows the same shape:

```
command  -flags  arguments
```

Example:

```bash
$ ls -la /home/user/projects
```

Breaking it down:
- **`ls`** — the command (list files)
- **`-la`** — flags (`-l` for long format, `-a` for all files including hidden). Combined into `-la`.
- **`/home/user/projects`** — the argument (which directory to list)

**Flag conventions:**
- Short flags: a single dash and one letter — `-l`, `-a`, `-r`. Can usually be combined: `-la` = `-l -a`.
- Long flags: two dashes and a word — `--all`, `--recursive`, `--help`. Cannot be combined.
- Some flags take values: `--output=file.txt` or `-o file.txt`.

---

## 3. The Filesystem as a Tree

Every Unix-like system (Linux, macOS, WSL) organises files as a single tree starting from `/`, called the **root**.

```
/                          ← root
├── bin/                   ← essential system commands (ls, cp, etc.)
├── etc/                   ← system configuration files
├── home/
│   ├── alice/             ← Alice's personal files
│   │   ├── documents/
│   │   ├── projects/
│   │   └── .bashrc        ← hidden config file
│   └── bob/
├── usr/
│   ├── bin/               ← user-installed commands
│   └── local/
├── tmp/                   ← temporary files (cleared on reboot)
└── var/
    └── log/               ← log files
```

On macOS the structure is similar but users live in `/Users/alice` instead of `/home/alice`.

There are no drive letters like `C:\` or `D:\` — external drives mount into the tree at places like `/mnt/usb` or `/Volumes/MyDrive`.

---

## 4. Absolute Paths

An **absolute path** starts with `/` and describes a file's full location from the root of the filesystem. It's unambiguous: it means the same thing no matter where you are.

```
/home/alice/projects/website/index.html
```

Reading left to right: from root → `home` → `alice` → `projects` → `website` → file `index.html`.

**Examples:**

```bash
$ cat /etc/hostname
my-laptop

$ ls /home/alice/projects
website  api  notes
```

Use absolute paths when:
- Writing scripts or configs that need to work no matter where they're invoked from.
- Referring to system files (`/etc/hosts`, `/var/log/syslog`).
- The file is far from your current location.

---

## 5. Relative Paths

A **relative path** does *not* start with `/`. It's interpreted relative to your **current working directory** (the folder you're "in" right now).

If you're currently inside `/home/alice/projects/`:

| Relative path | Resolves to |
| :--- | :--- |
| `website` | `/home/alice/projects/website` |
| `website/index.html` | `/home/alice/projects/website/index.html` |
| `./website` | `/home/alice/projects/website` (same — `./` means "here") |

If you `cd` to a different folder, the same relative path now means something different. That's the whole point — relative paths follow you around.

Use relative paths when:
- Working inside a project (referring to other files in the same project).
- Writing portable scripts (so the project can be moved without breaking).
- Typing fast — they're shorter.

---

## 6. The Special Path Symbols

Three symbols show up everywhere:

| Symbol | Meaning |
| :--- | :--- |
| `.` | The current directory |
| `..` | The parent directory (one level up) |
| `~` | The home directory (covered in the next section) |

**Examples** (assuming you're in `/home/alice/projects/website`):

| Path | Resolves to |
| :--- | :--- |
| `.` | `/home/alice/projects/website` |
| `./style.css` | `/home/alice/projects/website/style.css` |
| `..` | `/home/alice/projects` |
| `../api` | `/home/alice/projects/api` |
| `../../bob` | `/home/bob` |
| `../../..` | `/home` |

The `./` prefix is technically optional for files in the current directory (`./script.sh` and `script.sh` usually mean the same thing), with one important exception: **executing scripts**. To run a script in the current directory, you must write `./script.sh`. Bare `script.sh` makes the shell search `$PATH` instead — see Section 28.

---

## 7. The Home Directory and `~`

Your **home directory** is where your personal files live. On Linux it's typically `/home/yourname`; on macOS it's `/Users/yourname`.

The `~` (tilde) is shorthand for "my home directory." It expands automatically.

```bash
$ echo ~
/home/alice

$ ls ~/projects
website  api  notes

$ cd ~
$ pwd
/home/alice
```

`~/something` is almost always the cleanest way to refer to a file in your home directory — it works the same regardless of your username.

You can also use `~user` to refer to *another* user's home: `~bob` = `/home/bob`. Rarely needed.

---

## 8. Path Resolution Rules

When the shell sees a path, here's how it resolves it:

1. **Starts with `/`?** → Absolute. Go from root.
2. **Starts with `~`?** → Expand `~` to your home directory, then resolve.
3. **Starts with `.` or `..`?** → Relative to current directory.
4. **Anything else?** → Also relative to current directory.

Examples assuming current directory is `/home/alice/projects`:

| You type | Shell resolves to |
| :--- | :--- |
| `/etc/hosts` | `/etc/hosts` |
| `~/notes.txt` | `/home/alice/notes.txt` |
| `./website` | `/home/alice/projects/website` |
| `../bob` | `/home/bob` |
| `website` | `/home/alice/projects/website` |

This is why `pwd` (Section 11) matters constantly — it tells you what relative paths will actually resolve to.

---

## 9. Hidden Files (Dotfiles)

Any file or folder whose name starts with `.` is **hidden** by default. `ls` won't show them; `ls -a` will.

```bash
$ ls
documents  projects

$ ls -a
.            ..           .bashrc      .config      .ssh
documents    projects
```

These are typically configuration files. Common ones:

| Dotfile | What it is |
| :--- | :--- |
| `.bashrc` / `.zshrc` | Shell startup config |
| `.gitconfig` | Git settings |
| `.ssh/` | SSH keys and config |
| `.env` | Project environment variables |
| `.gitignore` | Files Git should ignore |
| `.vscode/` | VS Code workspace settings |

When you write `ls -a`, the first two entries are always `.` (current directory) and `..` (parent) — they're real entries in the directory listing, not just shell tricks.

---

## 10. File Extensions Don't Matter (Mostly)

On Windows, a `.txt` file *is* a text file because Windows treats the extension as authoritative. On Unix-like systems, **the extension is a convention, not a rule.** A file called `script` with no extension can still be a Python script if it starts with `#!/usr/bin/env python3`.

What actually matters:
- **Magic bytes** at the start of the file (the system inspects content to determine type).
- **Executable permission** (Section 29) — without it, the file can't be run directly.
- **Shebang line** (`#!/usr/bin/env python3`) — tells the shell which interpreter to use.

Use the `file` command to ask the system what something actually is:

```bash
$ file script
script: Python script, ASCII text executable

$ file image.jpg
image.jpg: JPEG image data, JFIF standard 1.01
```

Extensions still help humans (and editors) — keep using them. Just don't assume the system relies on them.

---

## 11. Where Am I? (`pwd`)

`pwd` = **print working directory.** Shows the absolute path of the folder you're currently in.

```bash
$ pwd
/home/alice/projects/website
```

Run this whenever you're unsure where you are. Most prompts also show the current directory, but not always (especially inside `tmux`, SSH sessions, or minimal shells).

---

## 12. Listing Files (`ls`)

`ls` lists the contents of a directory. Without arguments, it lists the current directory.

```bash
$ ls
documents  projects  notes.txt
```

**Useful flags:**

| Flag | What it does |
| :--- | :--- |
| `-l` | Long format — permissions, size, date, owner |
| `-a` | Show hidden files (dotfiles) |
| `-h` | Human-readable sizes (`5.2K` instead of `5234`). Use with `-l`. |
| `-t` | Sort by modification time (newest first) |
| `-r` | Reverse sort order |
| `-S` | Sort by size |
| `-R` | Recursive — show contents of subdirectories too |

**Common combinations:**

```bash
$ ls -la
total 24
drwxr-xr-x  4 alice alice 4096 May  3 14:22 .
drwxr-xr-x  3 root  root  4096 Apr 28 09:11 ..
-rw-r--r--  1 alice alice  220 Apr 28 09:11 .bashrc
drwxr-xr-x  2 alice alice 4096 May  3 14:22 documents
drwxr-xr-x  5 alice alice 4096 May  2 18:30 projects
-rw-r--r--  1 alice alice  142 May  3 14:22 notes.txt

$ ls -lh
-rw-r--r-- 1 alice alice 1.2K May  3 14:22 notes.txt
-rw-r--r-- 1 alice alice  45M May  2 11:00 dataset.csv

$ ls -lt          # newest files first
$ ls -laR ~/code  # everything, recursive, in ~/code
```

You can also list a specific path:

```bash
$ ls /etc
hosts  passwd  ssh  systemd  ...

$ ls ~/projects/website
index.html  style.css  script.js
```

---

## 13. Changing Directories (`cd`)

`cd` = **change directory.** Moves your current location to the given path.

```bash
$ pwd
/home/alice

$ cd projects/website
$ pwd
/home/alice/projects/website
```

**Special forms:**

| Command | Goes to |
| :--- | :--- |
| `cd` | Your home directory (same as `cd ~`) |
| `cd ~` | Your home directory |
| `cd ..` | Parent directory |
| `cd ../..` | Two levels up |
| `cd -` | The previous directory you were in (toggle) |
| `cd /` | Root |
| `cd /etc` | Absolute path |

The `cd -` toggle is genuinely useful — it lets you bounce between two directories quickly.

```bash
$ pwd
/home/alice/projects

$ cd /var/log
$ pwd
/var/log

$ cd -
/home/alice/projects     # prints where it went
$ pwd
/home/alice/projects
```

---

## 14. Tab Completion

Press **`Tab`** while typing a path or command and the shell will complete it for you. Press it twice to see all options if there's ambiguity.

```bash
$ cd ~/proj<Tab>
$ cd ~/projects/                    # auto-completed

$ cd ~/projects/<Tab><Tab>
api/  notes/  website/              # shows options

$ cd ~/projects/we<Tab>
$ cd ~/projects/website/            # only one match, completes
```

This is the single biggest productivity boost in the terminal. It also prevents typos. Use it constantly.

---

## 15. Creating Files (`touch`) and Folders (`mkdir`)

**Create an empty file:**

```bash
$ touch notes.txt
$ ls
notes.txt

$ touch a.txt b.txt c.txt   # multiple at once
```

`touch` actually updates the file's modification timestamp, but if the file doesn't exist, it creates an empty one. The "create empty file" use case is more common than the timestamp use case.

**Create a folder:**

```bash
$ mkdir new-project
$ ls
new-project
```

**Create nested folders in one shot** with `-p`:

```bash
$ mkdir -p projects/website/assets/images
```

This creates `projects`, then `website` inside it, then `assets`, then `images` — all in one command. Without `-p`, `mkdir` fails if any parent directory is missing. With `-p`, it creates the whole chain. **Use `-p` by default** — it's also idempotent (won't error if the folder already exists), which makes scripts more reliable.

---

## 16. Copying (`cp`)

`cp source destination` — copies a file.

```bash
$ cp notes.txt notes-backup.txt
$ ls
notes.txt  notes-backup.txt
```

**Copy to a different folder:**

```bash
$ cp notes.txt ~/documents/
```

(The trailing `/` makes it explicit you mean "into this folder," not "rename to this name.")

**Copy a folder** — needs the recursive flag `-r`:

```bash
$ cp -r website/ website-backup/
```

**Useful flags:**

| Flag | What it does |
| :--- | :--- |
| `-r` | Recursive (required for folders) |
| `-i` | Interactive — ask before overwriting existing files |
| `-v` | Verbose — print each file as it's copied |
| `-p` | Preserve permissions and timestamps |

**`cp` overwrites silently by default.** That's a footgun. Make `cp -i` your default if you're nervous, or use a version control system (Git) so accidents are recoverable.

---

## 17. Moving and Renaming (`mv`)

`mv` does double duty: it moves files, and renames them.

**Rename:**

```bash
$ mv notes.txt journal.txt
```

**Move to a folder:**

```bash
$ mv journal.txt ~/documents/
```

**Move and rename in one step:**

```bash
$ mv journal.txt ~/documents/old-journal.txt
```

`mv` works on folders too, no recursive flag needed:

```bash
$ mv old-project archive/old-project
```

Like `cp`, `mv` overwrites silently. Use `-i` for safety: `mv -i old new`.

---

## 18. Deleting (`rm`)

**Critical warning:** `rm` does **not** send files to a trash bin. They're gone. There is no Ctrl+Z. There is no Recycle Bin. Be deliberate.

**Delete a file:**

```bash
$ rm notes.txt
```

**Delete multiple files:**

```bash
$ rm a.txt b.txt c.txt
```

**Delete a folder** — needs `-r` (recursive):

```bash
$ rm -r old-project/
```

**Useful flags:**

| Flag | What it does |
| :--- | :--- |
| `-r` | Recursive (required for folders) |
| `-i` | Interactive — confirm each deletion |
| `-f` | Force — ignore non-existent files, never prompt |
| `-v` | Verbose |

**The infamous command:**

```bash
$ rm -rf /     # NEVER RUN THIS. Deletes everything you have permission to delete.
```

Most modern systems block this exact form, but variants like `rm -rf $UNSET_VARIABLE/` (which becomes `rm -rf /`) have wiped real production systems. Always quote variables and double-check `rm -rf` commands.

**Safer alternatives:**
- Use `trash-cli` (Linux) or `trash` (macOS via Homebrew) — actual trash semantics.
- Make `alias rm='rm -i'` in your shell config (Section 33).

---

## 19. Viewing File Contents (`cat`, `less`, `head`, `tail`)

Different commands for different sizes and use cases.

**`cat`** — dump the whole file to the terminal. Good for short files.

```bash
$ cat notes.txt
Buy groceries.
Call dentist.
Fix the bug in auth.py.
```

`cat` also concatenates multiple files (that's where the name comes from):

```bash
$ cat header.txt body.txt footer.txt > combined.txt
```

**`less`** — paged viewer. Good for long files. You can scroll, search, and quit.

```bash
$ less /var/log/syslog
```

Inside `less`:
- `Space` / `f` — page down
- `b` — page up
- `g` — go to top
- `G` — go to bottom
- `/pattern` — search forward
- `n` — next match
- `q` — quit

**`head`** — first 10 lines (or `-n N` for first N):

```bash
$ head data.csv
id,name,age
1,Alice,30
2,Bob,25
...

$ head -n 3 data.csv
id,name,age
1,Alice,30
2,Bob,25
```

**`tail`** — last 10 lines. Add `-f` to follow (live update — great for log files):

```bash
$ tail -n 5 logs.txt
$ tail -f logs.txt    # streams new lines as they're written. Ctrl+C to stop.
```

---

## 20. Finding Files by Name (`find`)

`find` searches a directory tree for files matching a pattern.

**Basic shape:** `find <where> <criteria>`

```bash
$ find . -name "*.py"
./src/main.py
./src/utils.py
./tests/test_main.py
```

This finds every file ending in `.py` under the current directory (`.`).

**Common criteria:**

| Criterion | Example |
| :--- | :--- |
| `-name "pattern"` | Match by name (case-sensitive) |
| `-iname "pattern"` | Case-insensitive name match |
| `-type f` | Files only |
| `-type d` | Directories only |
| `-size +10M` | Larger than 10 MB |
| `-mtime -7` | Modified in last 7 days |
| `-empty` | Empty files / directories |

**Combining:**

```bash
$ find . -type f -name "*.log" -size +1M
./logs/app.log
./logs/error.log
```

**Run a command on each match:**

```bash
$ find . -name "*.pyc" -delete           # delete compiled Python files
$ find . -name "*.txt" -exec wc -l {} \; # word-count each .txt file
```

The `{}` is replaced with each filename; the `\;` ends the `-exec` command.

**Quote your patterns!** Without quotes, the shell expands `*.py` itself before passing it to `find`. With quotes, `find` does the matching at every level. Always write `-name "*.py"`, not `-name *.py`.

---

## 21. Searching Inside Files (`grep`)

`grep` searches text inside files for a pattern.

**Basic shape:** `grep <pattern> <files>`

```bash
$ grep "TODO" main.py
# TODO: handle the empty case
# TODO: add tests
```

**Useful flags:**

| Flag | What it does |
| :--- | :--- |
| `-r` | Recursive — search inside all files in a directory |
| `-i` | Case-insensitive |
| `-n` | Show line numbers |
| `-l` | Show only filenames that match (not the matching lines) |
| `-v` | Invert — show lines that *don't* match |
| `-w` | Match whole words only |
| `-c` | Count matching lines |
| `-A 3` | Show 3 lines **a**fter each match |
| `-B 3` | Show 3 lines **b**efore |
| `-C 3` | Show 3 lines of **c**ontext (before and after) |

**Common patterns:**

```bash
$ grep -rn "TODO" .                    # find every TODO, with line numbers
$ grep -ri "error" logs/               # case-insensitive search in logs
$ grep -l "import pandas" *.py         # which Python files use pandas?
$ grep -v "^#" config.txt              # all lines that aren't comments
```

For modern alternatives: **`ripgrep`** (`rg`) is dramatically faster and respects `.gitignore` by default. Worth installing.

---

## 22. Wildcards (Globs)

The shell expands certain characters into matching filenames *before* the command runs. This is **globbing**.

| Glob | Matches |
| :--- | :--- |
| `*` | Any characters (including none), but not `/` |
| `?` | Exactly one character |
| `[abc]` | One of `a`, `b`, or `c` |
| `[a-z]` | Any lowercase letter |
| `{jpg,png,gif}` | One of the alternatives (brace expansion) |
| `**` | Any directories (recursive — needs `globstar` enabled in bash) |

**Examples:**

```bash
$ ls *.txt              # all .txt files in current directory
$ ls report?.pdf        # report1.pdf, reportA.pdf — but not report10.pdf
$ ls [abc]*.csv         # files starting with a, b, or c
$ rm *.tmp              # delete all .tmp files
$ cp *.{jpg,png} ~/photos/   # copy all jpg and png files
```

**Important:** the shell does the expansion, not the command. By the time `rm` runs, the shell has already replaced `*.tmp` with the actual list of matching files. This is why globs that match nothing can produce errors:

```bash
$ rm *.xyz
rm: cannot remove '*.xyz': No such file or directory
```

The glob didn't match, so the shell passed `*.xyz` as a literal argument.

---

## 23. Pipes (`|`)

A **pipe** sends the output of one command as input to another. This is the philosophy of Unix: small, composable tools.

```bash
$ ls | wc -l
17
```

`ls` lists files; `wc -l` counts lines. Together: count of files in this directory.

**More examples:**

```bash
$ ps aux | grep python              # find running Python processes
$ cat access.log | grep "404"       # error lines in a log
$ history | tail -20                # last 20 commands
$ ls -la | sort -k5 -n              # ls output sorted by size (5th column, numeric)
```

You can chain as many pipes as you want:

```bash
$ cat data.csv | grep "2024" | sort | uniq | wc -l
# show 2024 rows, sort, deduplicate, count
```

This is the core mental model: **each command does one thing, pipes glue them together.**

---

## 24. Redirects (`>`, `>>`, `<`)

Redirects send output to a file (or read input from one) instead of the terminal.

| Operator | Meaning |
| :--- | :--- |
| `>` | Send output to file (overwrites) |
| `>>` | Append output to file |
| `<` | Read input from file |
| `2>` | Send error output to file |
| `&>` | Send both regular and error output to file |

**Examples:**

```bash
$ ls > files.txt              # save listing to a file
$ echo "hello" > greeting.txt # write a single line to a file
$ echo "world" >> greeting.txt # append a line
$ cat greeting.txt
hello
world

$ ./script.sh 2> errors.log    # save errors only
$ ./script.sh &> output.log    # save everything
$ ./script.sh > output.log 2>&1 # same thing, older syntax
```

**Read input from a file:**

```bash
$ sort < unsorted.txt
$ python script.py < input.json
```

The combination of pipes and redirects is what makes shell scripting powerful. A pipe sends output to *another command*; a redirect sends it to *a file*.

---

## 25. Chaining (`&&`, `||`, `;`)

Run multiple commands in sequence:

| Operator | Meaning |
| :--- | :--- |
| `;` | Run sequentially regardless of success |
| `&&` | Run next only if previous **succeeded** |
| `\|\|` | Run next only if previous **failed** |

**Examples:**

```bash
$ cd ~/projects ; ls          # cd then ls, even if cd fails
$ cd ~/projects && ls         # ls only if cd succeeded (much safer)
$ cd ~/projects || echo "directory missing"
$ make && ./run-tests && echo "all good"
```

The `&&` chain is the right default — it stops at the first failure, which is almost always what you want.

```bash
$ git pull && npm install && npm test
# If pull fails, don't install. If install fails, don't test.
```

---

## 26. Processes (`ps`, `top`, `kill`)

A **process** is a running program. The shell, your editor, your browser — each is a process.

**`ps`** — list processes.

```bash
$ ps                  # processes in this terminal session
  PID TTY          TIME CMD
12345 pts/0    00:00:00 bash
12389 pts/0    00:00:00 ps

$ ps aux              # all processes, all users, detailed
USER       PID %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND
root         1  0.0  0.1 168424 11856 ?    Ss   May01   0:02 /sbin/init
alice    12345  0.0  0.0  21804  5604 pts/0 Ss   14:22   0:00 bash
alice    12500  2.3  4.1 824932 67200 ?    Sl   14:25   0:14 python app.py
```

**`top`** (or **`htop`** if installed) — live, updating view of CPU and memory usage. Press `q` to quit.

**`kill`** — send a signal to a process by its PID.

```bash
$ kill 12500          # polite request to terminate
$ kill -9 12500       # force kill (use only if polite kill fails)
```

**Find and kill in one go:**

```bash
$ pkill python              # kill all python processes
$ pgrep -f "myscript.py"    # find PID of process matching pattern
```

---

## 27. Disk Usage (`du`, `df`)

**`df`** = **disk free.** Shows total disk space per filesystem.

```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       450G  120G  310G  28% /
/dev/sdb1       1.8T  900G  900G  50% /mnt/data
```

**`du`** = **disk usage.** Shows how much space files/folders use.

```bash
$ du -sh ~/projects
3.2G    /home/alice/projects

$ du -sh *               # size of each item in current directory
1.2G    Downloads
850M    documents
3.2G    projects
```

**Find what's eating disk:**

```bash
$ du -sh * | sort -h | tail -10
# Top 10 largest items in current directory
```

`-h` = human-readable, `-s` = summary only (don't drill into subfolders).

---

## 28. Environment Variables

**Environment variables** are name → value pairs the shell (and child processes) can read. They're how programs get configuration without command-line arguments.

**View them:**

```bash
$ echo $HOME
/home/alice

$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/home/alice/.local/bin

$ env                # print all environment variables
```

**Set one for a single command:**

```bash
$ DEBUG=1 ./run.sh    # DEBUG is set only for this invocation
```

**Set one for the rest of the session:**

```bash
$ export API_KEY=abc123
$ echo $API_KEY
abc123
```

**Set one permanently** — add the `export` line to `~/.bashrc` or `~/.zshrc`:

```bash
# in ~/.bashrc
export EDITOR=vim
export PATH="$HOME/.local/bin:$PATH"
```

Then reload with `source ~/.bashrc` or open a new terminal.

**`PATH`** is the most important one. It's a colon-separated list of directories the shell searches when you type a command. When you type `python`, the shell looks for an executable called `python` in each directory in `PATH`, in order, and runs the first one it finds.

---

## 29. Permissions (`chmod`, `chown`)

Every file has three permission groups: **owner**, **group**, **others**. Each group has three permissions: **read** (`r`), **write** (`w`), **execute** (`x`).

```bash
$ ls -l script.sh
-rwxr-xr--  1 alice devs  420 May  3 14:22 script.sh
```

Reading the permissions string `-rwxr-xr--`:

| Position | Meaning |
| :--- | :--- |
| `-` | File type (`-` = file, `d` = directory, `l` = symlink) |
| `rwx` | Owner can read, write, execute |
| `r-x` | Group can read, execute |
| `r--` | Others can read |

**`chmod`** changes permissions. Two notations: symbolic and numeric.

**Symbolic:**

```bash
$ chmod +x script.sh         # add execute for everyone
$ chmod u+w file.txt         # add write for user (owner)
$ chmod go-r secret.txt      # remove read for group and others
```

`u` = user/owner, `g` = group, `o` = others, `a` = all. `+` adds, `-` removes, `=` sets exactly.

**Numeric** (each digit is the sum: read=4, write=2, execute=1):

```bash
$ chmod 755 script.sh        # rwxr-xr-x  (typical for scripts)
$ chmod 644 notes.txt        # rw-r--r--  (typical for files)
$ chmod 600 ~/.ssh/id_rsa    # rw-------  (private — SSH keys)
$ chmod 700 ~/.ssh           # rwx------  (private folder)
```

**`chown`** changes ownership (usually needs `sudo`):

```bash
$ sudo chown alice:devs file.txt    # owner=alice, group=devs
$ sudo chown -R alice ~/projects    # recursive
```

---

## 30. Command History

Every command you type is saved (usually in `~/.bash_history` or `~/.zsh_history`).

| Action | How |
| :--- | :--- |
| Previous command | Press `↑` |
| Next command | Press `↓` |
| Reverse search | Press `Ctrl+R`, then start typing |
| Show history | `history` |
| Re-run command 503 | `!503` |
| Re-run last command | `!!` |
| Re-run last command starting with `git` | `!git` |

**`Ctrl+R` is genuinely life-changing** — start typing any part of a previous command and it surfaces:

```
(reverse-i-search)`grep': grep -rn "TODO" src/
```

Press `Ctrl+R` again to cycle through older matches. `Enter` to run it, `→` to edit it first.

---

## 31. Keyboard Shortcuts

These work in bash, zsh, and most readline-based prompts:

| Shortcut | Action |
| :--- | :--- |
| `Ctrl+A` | Jump to beginning of line |
| `Ctrl+E` | Jump to end of line |
| `Ctrl+U` | Delete from cursor to start of line |
| `Ctrl+K` | Delete from cursor to end of line |
| `Ctrl+W` | Delete word before cursor |
| `Ctrl+L` | Clear screen (same as `clear`) |
| `Ctrl+C` | Cancel current command / kill running process |
| `Ctrl+D` | Logout / send EOF (close shell if line is empty) |
| `Ctrl+Z` | Suspend current process (resume with `fg`) |
| `Alt+B` / `Alt+F` | Move backward / forward one word |
| `Tab` | Auto-complete |

`Ctrl+L` is faster than typing `clear`. `Ctrl+U` is faster than holding backspace. Learn these — they compound over years of use.

---

## 32. Getting Help (`man`, `--help`)

**`man <command>`** — manual page. The canonical reference.

```bash
$ man ls
$ man grep
```

Inside `man`:
- `Space` / `f` — page down
- `b` — page up
- `/pattern` — search
- `q` — quit

**`<command> --help`** — short usage info.

```bash
$ ls --help
$ grep --help | head -20
```

For more accessible explanations: **`tldr <command>`** (install separately) shows practical examples instead of exhaustive docs.

```bash
$ tldr tar
# tar
# Archiving utility...
# - Create an archive from files:
#   tar cf target.tar file1 file2 file3
# ...
```

Reference sites worth bookmarking:
- [explainshell.com](https://explainshell.com) — paste any command, get an annotated breakdown
- [tldr.sh](https://tldr.sh) — practical examples for common commands

---

## 33. Aliases

An **alias** is a shortcut for a longer command. Defined in your shell config (`~/.bashrc` or `~/.zshrc`).

```bash
# in ~/.bashrc
alias ll='ls -la'
alias gs='git status'
alias gp='git push'
alias ..='cd ..'
alias ...='cd ../..'
alias rm='rm -i'           # safety net
alias grep='grep --color=auto'
```

Reload with `source ~/.bashrc` or open a new terminal.

```bash
$ ll
total 24
drwxr-xr-x  4 alice alice 4096 May  3 14:22 .
...
```

Functions are similar but more powerful (can take arguments):

```bash
# in ~/.bashrc
mkcd() {
    mkdir -p "$1" && cd "$1"
}
```

```bash
$ mkcd new-project
$ pwd
/home/alice/new-project
```

Don't go overboard. Aliases that diverge too far from standard commands make it painful to work on other machines.

---

## 34. Command Cheat Sheet

| Task | Command |
| :--- | :--- |
| Where am I | `pwd` |
| List files | `ls -la` |
| Change directory | `cd path` |
| Go home | `cd` or `cd ~` |
| Go up one level | `cd ..` |
| Go back to previous | `cd -` |
| Create empty file | `touch file` |
| Create folder | `mkdir -p path/to/folder` |
| Copy file | `cp src dst` |
| Copy folder | `cp -r src dst` |
| Move / rename | `mv src dst` |
| Delete file | `rm file` |
| Delete folder | `rm -r folder` |
| View file | `cat file` |
| Page through file | `less file` |
| First N lines | `head -n N file` |
| Last N lines | `tail -n N file` |
| Follow log | `tail -f file` |
| Find files by name | `find . -name "*.py"` |
| Search inside files | `grep -rn "pattern" .` |
| Pipe output to next command | `cmd1 \| cmd2` |
| Save output to file | `cmd > file` |
| Append output to file | `cmd >> file` |
| Run if previous succeeded | `cmd1 && cmd2` |
| List processes | `ps aux` |
| Live process view | `top` or `htop` |
| Kill process | `kill PID` |
| Disk free | `df -h` |
| Folder size | `du -sh folder` |
| Show env var | `echo $VAR` |
| Set env var | `export VAR=value` |
| Make executable | `chmod +x script.sh` |
| Reverse search history | `Ctrl+R` |
| Clear screen | `Ctrl+L` |
| Cancel command | `Ctrl+C` |
| Tab complete | `Tab` |
| Manual page | `man command` |
| Quick examples | `tldr command` |

---

## 35. Common Mistakes

**Forgetting `-r` on folders.** `cp folder dest` and `rm folder` both fail without `-r`. They want recursion to be explicit.

**Running `rm -rf` carelessly.** There is no undo. Always read the command twice. If a path involves a shell variable (`rm -rf $DIR/`), make sure the variable is set — an unset variable expands to empty, turning the command into `rm -rf /`.

**Confusing `>` with `>>`.** `>` overwrites the file; `>>` appends. Wrong choice loses data silently.

**Forgetting to quote globs and arguments.** `find . -name *.py` lets the shell expand `*.py` first, breaking the command. Always quote: `find . -name "*.py"`. Same for filenames with spaces: `rm "my file.txt"`, not `rm my file.txt` (which tries to delete two files).

**Not reading the error message.** "No such file or directory" tells you the path is wrong. "Permission denied" tells you the issue is access rights, not the command itself. The shell is usually clear about what went wrong.

**Confusing `.` and `./`.** `.` alone means "the current directory." `./script.sh` means "run the script located in the current directory." `./` is needed before script names because bare names are searched in `$PATH`.

**Using absolute paths in scripts that should be portable.** A script with `cd /home/alice/projects` only works on Alice's machine. Use relative paths or `$HOME`.

**Putting a space around `=` in variable assignment.** `VAR = value` is parsed as the command `VAR` with arguments `=` and `value`. Correct form is `VAR=value` (no spaces).

**Forgetting that `cd` inside a script doesn't affect the parent shell.** Scripts run in a subshell. `cd` in a script changes the script's directory, then exits — your terminal is unchanged. To `cd` from a script, you must `source` it: `source script.sh`.

**Editing system files without `sudo`.** `vim /etc/hosts` will open the file but silently refuse to save. Use `sudo vim /etc/hosts` upfront.

**Not knowing where you are before running destructive commands.** Always `pwd` before `rm -r *`. Always.

---

*Further reading:*
- *[The Linux Command Line by William Shotts](https://linuxcommand.org/tlcl.php) — free book, the most comprehensive intro*
- *[explainshell.com](https://explainshell.com) — paste any command for annotated explanation*
- *[tldr.sh](https://tldr.sh) — practical command examples*
- *[ss64.com](https://ss64.com) — quick command reference*
