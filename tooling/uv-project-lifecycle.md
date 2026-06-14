# The uv Project Lifecycle

A complete walkthrough of what happens when you create a Python project with [uv](https://docs.astral.sh/uv/): every command, what it touches, the environments and files involved, and how the pieces fit into one reusable mental model. The same pattern applies to a five-line script or a large application, so once the lifecycle is clear it does not change with scale.

**Sources:**
- [uv documentation](https://docs.astral.sh/uv/) — official reference
- [uv: Working on projects](https://docs.astral.sh/uv/guides/projects/) — `init`, `add`, `sync`, `run`
- [uv: Using Python environments](https://docs.astral.sh/uv/pip/environments/) — virtual environments
- [uv: Installing Python](https://docs.astral.sh/uv/guides/install-python/) — managed interpreters
- [uv: Caching](https://docs.astral.sh/uv/concepts/cache/) — the shared package cache
- [Python packaging: pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) — the project recipe

---

## How to read this guide

The sections follow the real order you would run things: set up, create, work, then understand structure and storage. Each section explains the command, what it changes on disk, and the reasoning behind it, then ends with a short **Notes** list of the things that bite people.

The recurring theme to watch for is a split that runs through everything: a few heavy, reusable resources live **outside** any single project (the Python interpreters and the package cache), while each project keeps only the light, specific things (its recipe and its environment). Hold that split in mind and every behavior below follows from it.

---

## Table of Contents

**Foundations**
- [1. The Starting Point](#1-the-starting-point)
- [2. The Mental Model in One Picture](#2-the-mental-model-in-one-picture)

**Creating a Project**
- [3. Initialize the Project](#3-initialize-the-project)
- [4. Create the Virtual Environment](#4-create-the-virtual-environment)
- [5. Add Dependencies](#5-add-dependencies)

**Working in the Project**
- [6. Select the Interpreter in the Editor](#6-select-the-interpreter-in-the-editor)
- [7. Running Code](#7-running-code)

**Structure and Reproducibility**
- [8. The Full File and Folder Layout](#8-the-full-file-and-folder-layout)
- [9. Rebuilding from Scratch](#9-rebuilding-from-scratch)
- [10. Durable vs Disposable](#10-durable-vs-disposable)

**Versions and Storage**
- [11. Multiple Python Versions](#11-multiple-python-versions)
- [12. The Shared Cache](#12-the-shared-cache)

**Reference**
- [13. Common Mistakes](#13-common-mistakes)
- [14. Cheat Sheet](#14-cheat-sheet)

---

## 1. The Starting Point

Before any project exists, three things live on the machine independently of any project:

| Resource | Where it lives | Role |
| :--- | :--- | :--- |
| uv and Git | Installed system wide | The tools you drive everything with |
| Base Python interpreter | uv's own store (for example `~/.local/...` and uv's data folder) | The shared interpreter that project environments point back to |
| Package cache | A central uv folder | Previously downloaded packages, ready to re-link without re-downloading |

These persist across every project. Nothing done inside a project alters them, with one exception: adding a brand new package downloads a copy into the shared cache so future projects can reuse it instantly.

The base interpreter stays pristine. You never install project packages directly into it. Each project gets its own isolated space instead, which is what the rest of this guide builds.

**Notes:**
- The base interpreter being shared is why creating an environment is fast and cheap: nothing large is copied.
- "Installed Python" and "a virtual environment" are different things. The base interpreter is installed but is not itself a virtual environment.

---

## 2. The Mental Model in One Picture

Everything below is an instance of one diagram:

```text
   SHARED, PERSISTENT (outside projects)        PER PROJECT (inside the folder)
   ┌───────────────────────────────┐            ┌──────────────────────────────┐
   │  base Python interpreter(s)    │ ◄───points─┤  .venv  (launcher + packages)│
   │  package cache (downloads)     │ ◄───links──┤  pyproject.toml  (the recipe)│
   └───────────────────────────────┘            │  uv.lock  (exact versions)   │
                                                 │  your code                   │
                                                 └──────────────────────────────┘
```

The project folder holds what is specific and disposable. The shared store holds what is heavy and reusable. A project's `.venv` does not contain a copy of Python or full copies of packages; it contains a pointer to the interpreter and hard-links to cached package files.

**Notes:**
- Deleting a project removes only the right-hand box. The left-hand box is untouched.
- This is why "delete freely and rebuild later" is safe: the durable recipe plus the shared store reconstruct the environment on demand.

---

## 3. Initialize the Project

```bash
uv init myproject
cd myproject
```

`uv init` scaffolds a new project and creates a small set of starter files:

| File | Purpose |
| :--- | :--- |
| `pyproject.toml` | The project's identity and dependency recipe: name, required Python version, and the dependency list. The single most important file. |
| `.python-version` | One line naming the Python version for this project (for example `3.14`). uv reads it to select the right interpreter here. |
| `main.py` | A trivial starter script. |
| `README.md` | A placeholder for documentation. |
| `.gitignore` | Pre-configured to exclude things that should never be committed, such as `.venv` and `__pycache__`. |

It also initializes a Git repository (a hidden `.git` folder), so the project is version controlled from the first commit.

What `uv init` does not do: it does not create the virtual environment, and it does not install anything. At this point there is no `.venv` folder yet.

**Notes:**
- Think of `pyproject.toml` as the project's identity card. It is the durable record of what the project needs.
- If you prefer to create the folder yourself, run `uv init` with no name from inside an existing empty directory.

---

## 4. Create the Virtual Environment

```bash
uv venv
```

This creates the `.venv` folder, the project's isolated virtual environment. Inside it:

- A small launcher (`Scripts/python.exe` on Windows, `bin/python` on macOS and Linux) that points back to the shared base interpreter. It does not copy the interpreter.
- A `pyvenv.cfg` config file recording which base Python this environment uses.
- Activation scripts.
- An almost empty `site-packages` folder, which is where installed packages will live.

The environment starts empty of your packages. It is a sealed space tied to this one project, so its packages can never collide with another project's packages. That isolation is the entire reason virtual environments exist.

**Notes:**
- You often do not need to run `uv venv` explicitly. `uv add`, `uv run`, and `uv sync` all create the `.venv` automatically inside a uv project. Running it explicitly just makes the step visible.
- To build the environment on a specific Python version, add the flag: `uv venv --python 3.12`. If that version is not installed, uv downloads it on demand.

---

## 5. Add Dependencies

```bash
uv add requests
```

`uv add` does two things at once:

1. **Installs the package into `.venv/site-packages`**, pulling the files from uv's cache. If the package has never been fetched, uv downloads it once into the cache first. The install uses hard-links, so the files in the project share the same bytes as the cached copy rather than duplicating them.
2. **Records the package in `pyproject.toml`** under dependencies, and pins the exact resolved version into a `uv.lock` file (created at this point if it does not exist).

That second step is the reproducibility mechanism. `pyproject.toml` records what the project depends on, and `uv.lock` records the exact versions resolved, so the environment can be rebuilt identically later.

Repeat `uv add` for each package the project needs. To pin a specific version, quote the specifier so the shell does not misread the `==`:

```bash
uv add "requests==2.28.0"
```

**Notes:**
- Use `uv add` rather than `pip install` for project work. Both place files in the environment, but only `uv add` updates the recipe in `pyproject.toml`. A package installed outside the recipe is invisible to anyone rebuilding the project later.
- Editors and notebooks sometimes offer to install a missing package for you (for example `ipykernel` for Jupyter). Prefer running `uv add ipykernel` yourself so the dependency is tracked, not installed loosely into the `.venv`.
- Adding a package never touches the global interpreter. It only changes this project's `.venv` and recipe.

---

## 6. Select the Interpreter in the Editor

```bash
code .
```

`code .` opens the current folder in VS Code. The one essential editor step is to point the editor at this project's environment.

Open the command palette (`Ctrl+Shift+P`), run **Python: Select Interpreter**, and choose the entry whose path contains `.venv` (it is usually labeled with the project's folder name and marked "Recommended").

Once selected, the editor uses that interpreter everywhere: the Run button, the integrated terminal, and any notebook kernel. The currently selected interpreter appears in the bottom-right status bar, and clicking it reopens the same picker.

This is what makes the editor immune to "which Python is it using?" confusion. Because the interpreter is named directly per project, terminal aliases and system PATH quirks cannot interfere.

**Notes:**
- The editor often auto-detects `.venv` and may select it for you, but choosing it explicitly is the habit that keeps things reliable.
- In the interpreter list, a venv shows the project name in parentheses, for example `Python 3.14.6 ('myproject')`. An entry marked "Global" with no project name is the shared base interpreter, which you generally do not select for project work.
- The integrated terminal auto-activates the selected `.venv` when it opens, which is why a project prefix appears in the prompt.

---

## 7. Running Code

There are three ways to run code, all using the project's environment:

- **A script in the editor:** open `main.py` and click Run. It executes with the `.venv` interpreter.
- **A notebook:** create a `.ipynb` file, write a cell, run it, and pick the `.venv` kernel when prompted. Notebook support needs `ipykernel` in the project, so run `uv add ipykernel` in any project that uses notebooks.
- **From the terminal:** prefix any command with `uv run`.

```bash
uv run python main.py
```

`uv run` executes inside the project's environment without manual activation, which sidesteps shell activation-policy issues entirely. It is the cleanest default habit.

**Notes:**
- On Windows PowerShell, manual activation (`.venv\Scripts\Activate.ps1`) can fail with an execution-policy error. Either run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` once, or simply prefer `uv run`, which never triggers it.
- Inside an activated environment, bare `python` resolves to the project interpreter. Outside it, `python` resolves to whatever is on PATH, which is usually the base interpreter.

---

## 8. The Full File and Folder Layout

After the steps above, the project looks like this:

```text
myproject/
├── .git/                       # version control (hidden)
├── .venv/                      # virtual environment (DISPOSABLE, never committed)
│   ├── Scripts/                # launcher + activation scripts
│   └── Lib/site-packages/      # installed packages (hard-linked from cache)
├── .gitignore                  # excludes .venv, __pycache__, etc.
├── .python-version             # "3.14"
├── pyproject.toml              # dependency recipe + config (the durable file)
├── uv.lock                     # exact pinned versions (commit this)
├── main.py                     # your code
└── README.md
```

What gets committed is everything except `.venv` and other ignored artifacts. So `pyproject.toml`, `uv.lock`, your code, the README, and `.python-version` are all committed. The `.venv` is excluded because it is large, machine specific, and fully rebuildable from the recipe.

**Notes:**
- A `.venv` contains real, working packages (so the environment actually runs), but those package files are usually hard-links to the cache, not fresh copies, so they cost almost no extra disk space.
- The interpreter is a pointer, not a copy. Ten projects on the same Python version all reference one installed interpreter.

---

## 9. Rebuilding from Scratch

This is the payoff of the whole structure. Anyone, including future you on a new machine, can rebuild the exact environment with one command after cloning:

```bash
git clone https://github.com/octocat/myproject.git
cd myproject
uv sync
```

`uv sync` reads `pyproject.toml` and `uv.lock`, creates a fresh `.venv`, and installs the exact recorded packages from the cache, typically in seconds. There is no guesswork about what to install, because the recipe handles it.

This is also why deleting a `.venv` is always safe. If an environment ever gets into a strange state, delete the folder and run `uv sync` (or `uv venv` then `uv sync`) to rebuild it cleanly.

**Notes:**
- `uv sync` rebuilds the whole environment from the recipe. `uv add` is for introducing a new dependency. Use sync when rehydrating a known project, add when changing what it depends on.
- Because rebuilds pull from the cache, they rarely re-download anything.

---

## 10. Durable vs Disposable

The single mental model to carry into every project:

| Category | What | Behavior |
| :--- | :--- | :--- |
| Durable (commit, keep) | `pyproject.toml`, `uv.lock`, your code | The recipe and the work. The source of truth. |
| Disposable (rebuildable) | `.venv` | The assembled environment. Delete freely; `uv sync` rebuilds it. |
| Shared and persistent (outside the project) | base interpreters, package cache | Untouched by project deletion. Make rebuilds fast. Removed only by explicit commands. |

Two one-directional rules make the separation safe:

- Deleting from a **project** never affects the **cache** or the installed **interpreters**.
- Clearing the cache or uninstalling an interpreter never affects an existing project's already-built `.venv`, because that environment's hard-links keep its own files alive independently.

**Notes:**
- A project's whole lifecycle follows from this table: initialize, add what you need, select the interpreter, build and run, commit the durable files. The `.venv` can be discarded at any point.
- When you delete a project folder, its `.venv` goes with it, because the environment lives inside the folder by default.

---

## 11. Multiple Python Versions

A project can run on any installed Python version, independent of other projects:

```bash
uv venv --python 3.12
```

If 3.12 is not installed, uv downloads it on demand. That interpreter is then stored once in uv's central store and shared by every future project that asks for 3.12. Different projects can run different versions side by side with no conflict, which is the standard way to handle a library that has not yet caught up to the newest Python.

Deleting a project does not remove the Python version it used. The interpreter lives outside the project, so it stays installed and ready, exactly like the cache. To inspect or remove versions:

```bash
uv python list --only-installed     # show installed interpreters
uv python install 3.12              # install a version explicitly
uv python uninstall 3.12            # remove a version deliberately
```

**Notes:**
- A listing tool may show one interpreter under several path names (a real install plus shims like `python`, `python3`, and a "latest patch" alias). Count distinct versions, not paths. Five rows can all be the same single version.
- Disk cost of an extra version is a one-time interpreter download, shared afterward. Idle RAM cost is zero, since only a running process uses memory.
- A new Python version is the one case that genuinely downloads something new; a new package version downloads once and then re-links.

---

## 12. The Shared Cache

When a package is added, uv downloads it once into a central cache and then hard-links it into the project. A hard-link is a second name for the same bytes on disk, not a second copy, so many projects "having" the same package version share one set of bytes.

This is why operations are fast and cheap: the second project to add a package skips the download entirely and links from the cache near-instantly. The cache is keyed by version, so two different versions of a package can coexist, each downloaded once.

The cache is persistent and outlives individual projects on purpose. Deleting a project removes its links but never the cached originals, so the next project that needs the same package still installs instantly.

```bash
uv cache clean       # empty the cache (rarely needed)
```

**Notes:**
- Clearing the cache only forces future re-downloads. It is not a tidy-up step after deleting a project; the project deletion was the cleanup.
- Hard-linking requires the cache and the project to be on the same drive. If they are on different drives, uv falls back to real copies for that project.
- From your code's perspective a hard-link behaves identically to a normal file; the sharing is an invisible disk-level optimization.

---

## 13. Common Mistakes

**Installing project packages into the global interpreter.** A new project's `.venv` starts empty and is sealed off from the global Python, so packages added globally are not visible inside it. Keep the base interpreter pristine and add packages per project.

**Using `pip install` instead of `uv add` for project dependencies.** Both place files in the environment, but only `uv add` records the package in `pyproject.toml`. Packages installed outside the recipe vanish when someone rebuilds the project.

**Letting the editor install a missing package loosely.** Accepting an editor prompt to install something like `ipykernel` puts it in the `.venv` but not in the recipe. Run `uv add ipykernel` instead so it is tracked.

**Committing the `.venv` folder.** It is large, machine specific, and fully rebuildable. The default `.gitignore` excludes it; do not override that.

**Assuming many listed Python paths mean many installs.** Tools may show one interpreter under several path names (shims and aliases). Count distinct versions, not rows.

**Clearing the cache to "tidy up" after deleting a project.** The cache is a speed-up layer, not clutter. Clearing it only forces re-downloads later.

**Expecting a global install to flow into new projects.** It will not. Isolation is one-directional and deliberate; each project declares and installs its own dependencies.

**Fighting PowerShell activation errors.** A script-execution-policy error on `Activate.ps1` is a Windows default, not a broken setup. Set the policy once for the current user, or just use `uv run`.

---

## 14. Cheat Sheet

| Task | Command |
| :--- | :--- |
| Create a new project | `uv init myproject` |
| Create the virtual environment | `uv venv` |
| Create it on a specific Python version | `uv venv --python 3.12` |
| Add a dependency | `uv add requests` |
| Add a pinned version | `uv add "requests==2.28.0"` |
| Add notebook support | `uv add ipykernel` |
| Run a command in the environment | `uv run python main.py` |
| Rebuild the environment from the recipe | `uv sync` |
| Open the project in the editor | `code .` |
| Select the interpreter (editor) | Command palette → Python: Select Interpreter → the `.venv` entry |
| List installed Python versions | `uv python list --only-installed` |
| Install a Python version | `uv python install 3.12` |
| Remove an unused Python version | `uv python uninstall 3.12` |
| Clear the package cache (rarely needed) | `uv cache clean` |

---

*This guide draws on the official uv documentation:*
- *[uv documentation](https://docs.astral.sh/uv/)*
- *[Working on projects](https://docs.astral.sh/uv/guides/projects/)*
- *[Using Python environments](https://docs.astral.sh/uv/pip/environments/)*
- *[Installing Python](https://docs.astral.sh/uv/guides/install-python/)*
- *[Caching](https://docs.astral.sh/uv/concepts/cache/)*
