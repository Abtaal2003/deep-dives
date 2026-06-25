# The uv Project Lifecycle — A Practical Guide

A complete walkthrough of what happens when you create a Python project with [uv](https://docs.astral.sh/uv/): every command, what it touches, the files it generates and what is inside them, and how the pieces fit into one reusable mental model. The same pattern applies to a five-line script or a large application, so once the lifecycle is clear it does not change with scale.

**Sources:**
- [uv documentation](https://docs.astral.sh/uv/), the official reference
- [uv: Working on projects](https://docs.astral.sh/uv/guides/projects/), `init`, `add`, `sync`, `run`
- [uv: Managing dependencies](https://docs.astral.sh/uv/concepts/projects/dependencies/), constraints and dependency groups
- [uv: Using Python environments](https://docs.astral.sh/uv/pip/environments/), virtual environments
- [uv: Installing Python](https://docs.astral.sh/uv/guides/install-python/), managed interpreters
- [uv: Caching](https://docs.astral.sh/uv/concepts/cache/), the shared package cache
- [uv: Exporting a lockfile](https://docs.astral.sh/uv/concepts/projects/export/), generating requirements.txt
- [Python packaging: pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/), the project recipe

---

## How to read this guide

The sections follow the real order you would run things: set up, create, work, then understand the files, structure, and storage. Each section explains the command, what it changes on disk, and the reasoning behind it, then ends with a short **Notes** list of the things that bite people.

The recurring theme to watch for is a split that runs through everything. A few heavy, reusable resources live **outside** any single project (the Python interpreters and the package cache), while each project keeps only the light, specific things (its recipe and its environment). Hold that split in mind and every behavior below follows from it.

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
- [9. Anatomy of the Generated Files](#9-anatomy-of-the-generated-files)
- [10. Rebuilding from Scratch](#10-rebuilding-from-scratch)
- [11. Durable vs Disposable](#11-durable-vs-disposable)
- [12. The Older Way: requirements.txt](#12-the-older-way-requirementstxt)

**Versions and Storage**
- [13. Multiple Python Versions](#13-multiple-python-versions)
- [14. The Shared Cache](#14-the-shared-cache)

**Reference**
- [15. Common Mistakes](#15-common-mistakes)
- [16. Cheat Sheet](#16-cheat-sheet)

---

## 1. The Starting Point

Before any project exists, three things live on the machine independently of any project:

| Resource | Where it lives | Role |
| :--- | :--- | :--- |
| uv and Git | Installed system wide | The tools you drive everything with |
| Base Python interpreter | uv's own store (for example `~/.local/...` and uv's data folder) | The shared interpreter that project environments point back to |
| Package cache | A central uv folder | Previously downloaded packages, ready to re-link without re-downloading |

These persist across every project. Nothing done inside a project alters them, with one exception — adding a brand new package downloads a copy into the shared cache so future projects can reuse it instantly.

The base interpreter stays pristine. You never install project packages directly into it. Each project gets its own isolated space instead, which is what the rest of this guide builds.

**Notes:**
- The base interpreter being shared is why creating an environment is fast and cheap — nothing large is copied.
- "Installed Python" and "a virtual environment" are different things. The base interpreter is installed, but it is not itself a virtual environment.

---

## 2. The Mental Model in One Picture

Everything below is an instance of one diagram:

```text
SHARED + PERSISTENT   (outside any project, reused by every project)
  • base Python interpreter(s)
  • package cache   (downloaded packages)
        ▲
        │   each project points at the interpreter
        │   and hard-links cached packages into its .venv
        │
PER PROJECT   (inside the project folder, disposable)
  • .venv/           launcher + installed packages
  • pyproject.toml   the dependency recipe
  • uv.lock          exact pinned versions
  • your code
```

The project folder holds what is specific and disposable. The shared store holds what is heavy and reusable. A project's `.venv` does not contain a copy of Python or full copies of packages; it contains a pointer to the interpreter and hard-links to cached package files.

**Notes:**
- Deleting a project removes only the per-project block — the shared, persistent block is untouched.
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

What `uv init` does not do: it does not create the virtual environment, and it does not install anything. At this point there is no `.venv` folder yet. Section 9 opens each of these files up so you can see what is inside.

**Notes:**
- Think of `pyproject.toml` as the project's identity card. It is the durable record of what the project needs.
- If you prefer to create the folder yourself, run `uv init` with no name from inside an existing empty directory.

---

## 4. Create the Virtual Environment

```bash
uv venv
```

This creates the `.venv` folder — the project's isolated virtual environment. Inside it:

- A small launcher (`bin/python` on macOS and Linux, `Scripts\python.exe` on Windows) that points back to the shared base interpreter. It does not copy the interpreter.
- A `pyvenv.cfg` config file recording which base Python this environment uses.
- Activation scripts.
- An almost empty `site-packages` folder, which is where installed packages will live.

The environment starts empty of your packages. It is a sealed space tied to this one project, so its packages can never collide with another project's packages — that isolation is the entire reason virtual environments exist.

**Notes:**
- You often do not need to run `uv venv` explicitly. `uv add`, `uv run`, and `uv sync` all create the `.venv` automatically inside a uv project. Running it explicitly just makes the step visible.
- To build the environment on a specific Python version, add the flag: `uv venv --python 3.12`. If that version is not installed, uv downloads it on demand.

---

## 5. Add Dependencies

```bash
uv add requests
```

`uv add` does two things at once:

1. **Installs the package into `.venv`**, pulling the files from uv's cache. If the package has never been fetched, uv downloads it once into the cache first. The install uses hard-links, so the files in the project share the same bytes as the cached copy rather than duplicating them.
2. **Records the package in `pyproject.toml`** under dependencies, and pins the exact resolved version into a `uv.lock` file (created at this point if it does not exist).

That second step is the reproducibility mechanism. `pyproject.toml` records what the project depends on, and `uv.lock` records the exact versions resolved, so the environment can be rebuilt identically later.

By default uv writes a lower-bound constraint, for example `requests>=2.32.0`, rather than a hard pin. To pin a specific version instead, quote the specifier so the shell does not misread the `==`:

```bash
uv add "requests==2.28.0"
```

For development-only tools (test runners, linters, formatters), add them to a dependency group with `--dev` so they are kept separate from what your code needs at runtime:

```bash
uv add --dev pytest ruff
```

### Project commands vs the pip interface

uv exposes two layers of commands, and knowing which layer you are in explains most "why didn't that update my `pyproject.toml`?" confusion.

- **Project commands** (`uv add`, `uv remove`, `uv lock`, `uv sync`, `uv run`) operate on the project as a whole. They read and write `pyproject.toml` and `uv.lock`, and they keep the `.venv` in step. This is the layer for normal project work.
- **The pip interface** (`uv pip install`, `uv pip uninstall`, `uv pip freeze`, and so on) is a fast, drop-in replacement for `pip` that operates only on the environment. It installs into the `.venv` but does not touch `pyproject.toml` or `uv.lock`.

So `uv add requests` records `requests` in your recipe and locks it, while `uv pip install requests` just drops it into the environment, invisible to anyone who rebuilds from the recipe later. The same split applies to the `-r` form: `uv add -r requirements.txt` imports a requirements file into `pyproject.toml` (what you want when migrating an old project), while `uv pip install -r requirements.txt` only installs those packages into the environment without recording them.

| Command | Edits `pyproject.toml` + `uv.lock`? | Touches `.venv`? | Use for |
| :--- | :--- | :--- | :--- |
| `uv add requests` | Yes | Yes | a real project dependency |
| `uv pip install requests` | No | Yes | a one-off or non-project install |
| `uv add -r requirements.txt` | Yes | Yes | importing/migrating an old project |
| `uv pip install -r requirements.txt` | No | Yes | installing a requirements file without adopting it |

Use project commands (`uv add`) for anything the project genuinely depends on. Reach for the pip interface only for throwaway, one-off, or non-project installs, or to interoperate with a `requirements.txt` without adopting it.

**Notes:**
- Use `uv add` rather than `pip install` for project work. Both place files in the environment, but only `uv add` updates the recipe in `pyproject.toml` — a package installed outside the recipe is invisible to anyone rebuilding the project later.
- Editors and notebooks sometimes offer to install a missing package for you (for example `ipykernel` for Jupyter). Prefer running `uv add ipykernel` yourself, so the dependency is tracked rather than installed loosely into the `.venv`.
- Adding a package never touches the global interpreter — it only changes this project's `.venv` and recipe.

---

## 6. Select the Interpreter in the Editor

```bash
code .
```

`code .` opens the current folder in VS Code. The one essential editor step is to point the editor at this project's environment.

Open the command palette (`Ctrl+Shift+P`), run **Python: Select Interpreter**, and choose the entry whose path contains `.venv` — it is usually labeled with the project's folder name and marked "Recommended".

Once selected, the editor uses that interpreter everywhere: the Run button, the integrated terminal, and any notebook kernel. The currently selected interpreter appears in the bottom-right status bar, and clicking it reopens the same picker.

This is what makes the editor immune to "which Python is it using?" confusion. Because the interpreter is named directly per project, terminal aliases and system PATH quirks cannot interfere.

**Notes:**
- The editor often auto-detects `.venv` and may select it for you, but choosing it explicitly is the habit that keeps things reliable.
- In the interpreter list, a venv shows the project name in parentheses, for example `Python 3.14.6 ('myproject')`. An entry marked "Global" with no project name is the shared base interpreter, which you generally do not select for project work.
- The integrated terminal auto-activates the selected `.venv` when it opens, which is why a project prefix appears in the prompt.

---

## 7. Running Code

There are three ways to run code, all using the project's environment:

- **A script in the editor.** Open `main.py` and click Run. It executes with the `.venv` interpreter.
- **A notebook.** Create a `.ipynb` file, write a cell, run it, and pick the `.venv` kernel when prompted. Notebook support needs `ipykernel` in the project, so run `uv add ipykernel` in any project that uses notebooks.
- **From the terminal.** Prefix any command with `uv run`.

```bash
uv run python main.py
```

`uv run` executes inside the project's environment without manual activation, which sidesteps shell activation-policy issues entirely. It is the cleanest default habit.

**Notes:**
- On Windows PowerShell, manual activation (`.venv\Scripts\Activate.ps1`) can fail with an execution-policy error — that is a Windows default, not a broken setup. Either run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` once, or simply prefer `uv run`, which never triggers it.
- Inside an activated environment, bare `python` resolves to the project interpreter. Outside it, `python` resolves to whatever is on PATH — usually the base interpreter.

---

## 8. The Full File and Folder Layout

After the steps above, the project looks like this:

```text
myproject/
├── .git/                       # version control (hidden)
├── .venv/                      # virtual environment (DISPOSABLE, never committed)
│   ├── bin/                    # launcher + activation scripts (Scripts/ on Windows)
│   └── lib/                    # installed packages in site-packages (hard-linked from cache)
├── .gitignore                  # excludes .venv, __pycache__, etc.
├── .python-version             # "3.14"
├── pyproject.toml              # dependency recipe + config (the durable file)
├── uv.lock                     # exact pinned versions (commit this)
├── main.py                     # your code
└── README.md
```

What gets committed is everything except `.venv` and other ignored artifacts. So `pyproject.toml`, `uv.lock`, your code, the README, and `.python-version` are all committed. The `.venv` is excluded because it is large, machine specific, and fully rebuildable from the recipe.

**Notes:**
- A `.venv` contains real, working packages (so the environment actually runs), but those package files are usually hard-links to the cache rather than fresh copies, so they cost almost no extra disk space.
- The interpreter is a pointer, not a copy. Ten projects on the same Python version all reference one installed interpreter.

---

## 9. Anatomy of the Generated Files

Section 8 listed the files. This section opens the ones you actually read and edit, so the abstract "recipe" becomes concrete. The rule of thumb: you hand-edit `pyproject.toml` (and occasionally `.gitignore`), while everything else is generated and left alone.

### pyproject.toml — the project recipe

This is the heart of the project. Fresh from `uv init`, it is minimal:

```toml
[project]
name = "myproject"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.14"
dependencies = []
```

After `uv add requests` and `uv add --dev pytest ruff`, it grows into:

```toml
[project]
name = "myproject"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.14"
dependencies = [
    "requests>=2.32.0",
]

[dependency-groups]
dev = [
    "pytest>=8.3.0",
    "ruff>=0.6.0",
]
```

The tables, one at a time:

- **`[project]`** is the standard metadata table defined by Python packaging, not something uv invented. `name` and `version` identify the project, `readme` points at the README, and `description` is a one-line summary.
- **`requires-python`** is the minimum interpreter the project supports, for example `>=3.14`. uv will not resolve dependencies that require a newer Python than this allows, and it uses this bound when choosing compatible package versions.
- **`dependencies`** are the packages your code needs at runtime. This is what `uv add` writes to. By default uv adds a lower-bound constraint (`>=`) rather than a hard pin, so non-breaking updates remain allowed.
- **`[dependency-groups]`** are development-only tools (test runners, linters, type checkers), written by `uv add --dev`. Defined by [PEP 735](https://peps.python.org/pep-0735/), these are never installed when someone consumes your package, so things like `pytest` and `ruff` do not land on a user's machine. Keep runtime needs in `dependencies` and tooling here.

Two more tables appear in some projects but not in a default application:

- **`[build-system]`** is only present when the project is a library or installable package (created with `uv init --lib` or `--package`). It tells build tools how to package the project. A plain application omits it because it is not meant to be installed as a package.
- **`[tool.*]`** holds configuration for other tools, all kept in this one file. For example `[tool.ruff]` holds linter and formatter settings, and `[tool.pytest.ini_options]` holds test settings. You add these by hand as you adopt each tool.

Version specifiers you will see in `dependencies`:

| Specifier | Meaning |
| :--- | :--- |
| `requests>=2.32.0` | this version or newer (uv's default when you add) |
| `requests==2.28.0` | exactly this version (a hard pin) |
| `requests~=2.32.0` | compatible release: `>=2.32.0` and `<2.33.0` |
| `requests>=2,<3` | an explicit range |
| `jax; sys_platform == 'linux'` | an environment marker (install only when the condition holds) |

### uv.lock — the exact resolution

Where `pyproject.toml` records broad constraints, `uv.lock` records the exact versions that were resolved, for every direct and transitive dependency. A single entry looks like:

```toml
[[package]]
name = "requests"
version = "2.32.3"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "certifi" },
    { name = "charset-normalizer" },
    { name = "idna" },
    { name = "urllib3" },
]
```

It is generated and maintained by uv (on `uv add`, `uv lock`, and `uv sync`), it is cross-platform, and it carries hashes for integrity. You never hand-edit it, and you always commit it — it is what lets `uv sync` reproduce an identical environment anywhere. This division of labor is the key idea: `pyproject.toml` says what is acceptable, `uv.lock` says what was actually chosen.

### .python-version — the interpreter pin

A single line:

```text
3.14
```

uv reads it to decide which interpreter to use in this project. Changing the line (or running `uv venv --python <version>`) changes the version uv selects, and it downloads that version on demand if needed.

### .gitignore — what stays out of version control

uv writes a starter `.gitignore` that keeps generated and machine-specific files out of the repository:

```gitignore
# Python-generated files
__pycache__/
*.py[oc]
build/
dist/
wheels/
*.egg-info

# Virtual environments
.venv
```

The important line is `.venv`. The environment is excluded because it is large and fully rebuildable from `pyproject.toml` and `uv.lock`. Add your own patterns here as the project grows (for example `.env` for secrets, covered when you start integrating APIs).

### main.py and README.md — the starter content

`main.py` is a runnable hello-world placeholder you replace with real code:

```python
def main():
    print("Hello from myproject!")


if __name__ == "__main__":
    main()
```

`README.md` is an empty-ish placeholder for documentation — the front page of the project once you push it.

A quick summary of how to treat each generated file:

| File | Hand-edit? | Commit? |
| :--- | :--- | :--- |
| `pyproject.toml` | Yes, directly or via `uv add` | Yes |
| `uv.lock` | No, generated by uv | Yes |
| `.python-version` | Occasionally (to change version) | Yes |
| `.gitignore` | Yes, add your own patterns | Yes |
| `main.py` / your code | Yes | Yes |
| `README.md` | Yes | Yes |
| `.venv/` | No | No, it is ignored and rebuildable |

**Notes:**
- `pyproject.toml` is the file you own and edit. `uv.lock` is generated, so never hand-edit it; change constraints in `pyproject.toml` and let uv re-resolve.
- Commit both `pyproject.toml` and `uv.lock` together. The recipe plus the exact pins are what make rebuilds reproducible.
- Avoid hard-pinning with `==` in `pyproject.toml` if you rely on `uv sync` to pick up compatible updates. Pin in `uv.lock` (which uv manages for you), keep ranges in `pyproject.toml`.
- `[dependency-groups]` is a uv and PEP 735 concept for internal tooling. It is different from `[project.optional-dependencies]` (extras), which are features you expose to users of a published package.

---

## 10. Rebuilding from Scratch

This is the payoff of the whole structure. Anyone, including future you on a new machine, can rebuild the exact environment with one command after cloning:

```bash
git clone https://github.com/octocat/myproject.git
cd myproject
uv sync
```

`uv sync` reads `pyproject.toml` and `uv.lock`, creates a fresh `.venv`, and installs the exact recorded packages from the cache, typically in seconds. There is no guesswork about what to install, because the recipe handles it.

This is also why deleting a `.venv` is always safe. If an environment ever gets into a strange state, delete the folder and run `uv sync` (or `uv venv` then `uv sync`) to rebuild it cleanly.

### uv lock vs uv sync

Two project commands sit behind reproducibility, and they do different jobs:

- **`uv lock`** resolves your dependencies and writes `uv.lock`. It updates the *record* of exact versions but does not change your installed environment. Run it to refresh the lockfile (for example after editing constraints in `pyproject.toml`) without reinstalling anything yet.
- **`uv sync`** makes the `.venv` match `uv.lock`. It installs what is missing and removes what should not be there, so the environment exactly equals the locked record. Run it to build or update the environment.

In practice you rarely call `uv lock` on its own, because the commands that change dependencies lock implicitly: `uv add` updates the lockfile as it adds, and `uv sync` re-locks if the recipe changed. `uv lock` is the explicit, environment-free way to re-resolve, useful in CI or when you want to review a lockfile change as a separate step before installing. A one-line way to hold them apart: `uv lock` decides what should be installed, and `uv sync` makes it so.

**Updating to newer versions.** uv never upgrades on its own; it does not treat the lockfile as stale when new releases appear, so upgrades are always explicit. The upgrade flags do it:

```bash
uv lock --upgrade                    # re-resolve everything to the newest allowed versions
uv lock --upgrade-package requests   # bump just one package
uv sync                              # apply the updated lockfile to the .venv
```

The flags also work directly on `uv sync` (for example `uv sync --upgrade-package requests`) to update and install in one step. One subtlety worth knowing: `uv add requests` on a package that is already present does **not** upgrade it, because the existing lock already satisfies the constraint. Use `--upgrade-package requests` for that, or change the constraint itself (edit `pyproject.toml`, or run `uv add "requests>=3"`) when you want to allow a new major version. The rule of thumb: `uv add` changes *what is allowed*, `--upgrade` changes *what is chosen* within that.

**Notes:**
- `uv sync` rebuilds the whole environment from the recipe — `uv add` is for introducing a new dependency. Use sync when rehydrating a known project, add when changing what it depends on.
- Because rebuilds pull from the cache, they rarely re-download anything.
- Upgrades are opt-in: nothing moves to a newer version until you run `uv lock --upgrade` (or `--upgrade-package`). This is what keeps builds reproducible by default.

---

## 11. Durable vs Disposable

The single mental model to carry into every project:

| Category | What | Behavior |
| :--- | :--- | :--- |
| Durable (commit, keep) | `pyproject.toml`, `uv.lock`, your code | The recipe and the work. The source of truth. |
| Disposable (rebuildable) | `.venv` | The assembled environment. Delete freely; `uv sync` rebuilds it. |
| Shared and persistent (outside the project) | base interpreters, package cache | Untouched by project deletion. Make rebuilds fast. Removed only by explicit commands. |

Two one-directional rules make the separation safe:

- Deleting from a **project** never affects the **cache** or the installed **interpreters**.
- Clearing the cache or uninstalling an interpreter never affects an existing project's already-built `.venv` — that environment's hard-links keep its own files alive independently.

**Notes:**
- A project's whole lifecycle follows from this table: initialize, add what you need, select the interpreter, build and run, commit the durable files. The `.venv` can be discarded at any point.
- When you delete a project folder, its `.venv` goes with it — the environment lives inside the folder by default.

---

## 12. The Older Way: requirements.txt

Before `pyproject.toml` and `uv.lock` became the standard, projects listed their dependencies in a plain-text `requirements.txt`. You will still meet it constantly — in older tutorials, legacy repositories, and deployment targets that expect it. uv reads and writes the format, so you can interoperate with it without leaving the modern workflow.

### What requirements.txt is

A flat text file, one package per line, optionally with version constraints:

```text
requests>=2.32.0
fastapi==0.115.0
rich
```

Classically it is installed into an activated environment with `pip install -r requirements.txt`. There is no lockfile and no separation of direct from transitive dependencies — whatever you list (plus whatever those pull in) gets installed, and reproducibility depends entirely on how strictly you pinned. A common refinement is the pip-tools pattern: a hand-edited `requirements.in` (loose) is compiled into a pinned `requirements.txt`. The modern `pyproject.toml` plus `uv.lock` pair replaces that two-file workflow.

### The complete old-school workflow

Without uv, the same project is set up with Python's built-in `venv` module and `pip`. The full loop, end to end:

```bash
# 1. Create the virtual environment (stdlib venv, not uv)
python -m venv .venv

# 2. Activate it (required before installing)
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\Activate.ps1       # Windows PowerShell

# 3. Install packages into the activated environment
pip install requests fastapi

# 4. Record what is installed, so the project is reproducible
pip freeze > requirements.txt
```

On another machine the project is rebuilt by recreating the environment and installing from the file:

```bash
python -m venv .venv
source .venv/bin/activate         # or the Windows form above
pip install -r requirements.txt
```

When you are done, `deactivate` returns the shell to normal. A caveat worth knowing: `pip freeze` captures every installed package at its exact version, direct and transitive together, so the resulting `requirements.txt` is a crude lockfile with no record of which packages you actually asked for. This is the whole loop that uv compresses — `uv add` replaces the manual install-then-freeze step, and `uv run` or `uv sync` remove the need to activate by hand.

### How it compares to pyproject.toml + uv.lock

| Capability | requirements.txt (older) | pyproject.toml + uv.lock (modern) |
| :--- | :--- | :--- |
| Declares dependencies | Yes, as a flat list | Yes, in `[project]` |
| Separates direct from transitive | No | Yes, the lock records transitive deps |
| Records exact versions + hashes | Only if you pin or compile | Yes, automatically in `uv.lock` |
| Project metadata, dev groups, tool config | No | Yes |
| Reproducible by default | No | Yes |

### Using requirements.txt with uv

uv speaks the format fluently through its pip-compatible interface:

```bash
uv pip install -r requirements.txt        # install from a requirements file
uv pip sync requirements.txt              # install exactly, removing anything not listed
uv pip compile requirements.in -o requirements.txt   # compile inputs into a pinned file (pip-tools replacement)
uv add -r requirements.txt                # import the file into pyproject.toml (use --dev for a dev file)
```

### Adopting uv in an existing project

To take over an older project that tracks dependencies in `requirements.txt`, import them into `pyproject.toml` rather than only installing them. From inside the project folder:

```bash
uv init --bare              # add a minimal pyproject.toml, leave existing files alone
uv add -r requirements.txt  # import deps into [project].dependencies (use --dev for a dev file)
uv sync                     # resolve, write uv.lock, and build the .venv
```

The important choice here is `uv add -r` rather than `uv pip install -r`. The goal of migrating is to make `pyproject.toml` the source of truth, and only `uv add` records the dependencies there; `uv pip install -r` would merely install them into the environment and leave the recipe empty. Once this is done, `pyproject.toml` and `uv.lock` are authoritative. Retire the old `requirements.txt`, or keep regenerating it with `uv export` if a deploy target still expects it. A fuller, step-by-step walkthrough lives in the companion guide, [Anatomy of a Python Project](../python/python-project-anatomy.md).

### Producing requirements.txt from a uv project

When a deployment target or tool expects `requirements.txt` but your project uses `pyproject.toml` and `uv.lock`, export one from the lockfile:

```bash
uv export --format requirements.txt --output-file requirements.txt
```

Treat the result as a build artifact. `pyproject.toml` and `uv.lock` stay authoritative, and you regenerate `requirements.txt` whenever something downstream needs it, such as Docker images, legacy CI, or serverless deploy targets.

**Notes:**
- uv recommends against maintaining both a `uv.lock` and a hand-edited `requirements.txt` as sources of truth. Pick `pyproject.toml` plus `uv.lock` as authoritative, and export `requirements.txt` only as a generated artifact.
- `requirements.txt` has no concept of dependency groups, so runtime and dev dependencies blur together unless you split into separate files (for example `requirements.txt` plus `requirements-dev.txt`).
- For a new project, prefer the modern workflow. Reach for `requirements.txt` only to interoperate with something that requires it.

---

## 13. Multiple Python Versions

A project can run on any installed Python version, independent of other projects:

```bash
uv venv --python 3.12
```

If 3.12 is not installed, uv downloads it on demand. That interpreter is then stored once in uv's central store and shared by every future project that asks for 3.12. Different projects can run different versions side by side with no conflict — which is the standard way to handle a library that has not yet caught up to the newest Python.

Deleting a project does not remove the Python version it used. The interpreter lives outside the project, so it stays installed and ready — exactly like the cache. To inspect or remove versions:

```bash
uv python list --only-installed     # show installed interpreters
uv python install 3.12              # install a version explicitly
uv python uninstall 3.12            # remove a version deliberately
```

**Notes:**
- A listing tool may show one interpreter under several path names — a real install plus shims like `python`, `python3`, and a "latest patch" alias. Count distinct versions, not paths. Five rows can all be the same single version.
- Disk cost of an extra version is a one-time interpreter download, shared afterward. Idle RAM cost is zero, since only a running process uses memory.
- A new Python version is the one case that genuinely downloads something new — a new package version downloads once and then re-links.

---

## 14. The Shared Cache

When a package is added, uv downloads it once into a central cache and then hard-links it into the project. A hard-link is a second name for the same bytes on disk, not a second copy — so many projects "having" the same package version share one set of bytes.

This is why operations are fast and cheap: the second project to add a package skips the download entirely and links from the cache near-instantly. The cache is keyed by version, so two different versions of a package can coexist, each downloaded once.

The cache is persistent and outlives individual projects on purpose. Deleting a project removes its links but never the cached originals, so the next project that needs the same package still installs instantly.

```bash
uv cache clean       # empty the cache (rarely needed)
```

**Notes:**
- Clearing the cache only forces future re-downloads. It is not a tidy-up step after deleting a project; the project deletion was the cleanup.
- Hard-linking requires the cache and the project to be on the same drive. If they are on different drives, uv falls back to real copies for that project.
- From your code's perspective a hard-link behaves identically to a normal file — the sharing is an invisible disk-level optimization.

---

## 15. Common Mistakes

**Installing project packages into the global interpreter.** A new project's `.venv` starts empty and is sealed off from the global Python, so packages added globally are not visible inside it. Keep the base interpreter pristine and add packages per project.

**Using `pip install` instead of `uv add` for project dependencies.** Both place files in the environment, but only `uv add` records the package in `pyproject.toml`. Packages installed outside the recipe vanish when someone rebuilds the project.

**Hand-editing `uv.lock`.** It is generated. Change constraints in `pyproject.toml` and let uv re-resolve. Editing the lockfile directly invites inconsistency between the two.

**Hard-pinning everything with `==` in `pyproject.toml`.** Pinning belongs in `uv.lock`, which uv maintains. Keep ranges (`>=`) in `pyproject.toml` so `uv sync` can still pick up compatible updates.

**Putting dev tools in `dependencies`.** Test runners and linters belong in `[dependency-groups]` via `uv add --dev`, so they are not forced onto anyone who consumes the project.

**Maintaining both `uv.lock` and a hand-edited `requirements.txt`.** Keep `pyproject.toml` and `uv.lock` authoritative, and treat any `requirements.txt` as a generated artifact you export with `uv export`, not a second source of truth.

**Letting the editor install a missing package loosely.** Accepting an editor prompt to install something like `ipykernel` puts it in the `.venv` but not in the recipe. Run `uv add ipykernel` instead, so it is tracked.

**Committing the `.venv` folder.** It is large, machine specific, and fully rebuildable. The default `.gitignore` excludes it; do not override that.

**Assuming many listed Python paths mean many installs.** Tools may show one interpreter under several path names (shims and aliases). Count distinct versions, not rows.

**Clearing the cache to "tidy up" after deleting a project.** The cache is a speed-up layer, not clutter. Clearing it only forces re-downloads later.

**Expecting a global install to flow into new projects.** It will not. Isolation is one-directional and deliberate; each project declares and installs its own dependencies.

**Fighting PowerShell activation errors.** A script-execution-policy error on `Activate.ps1` is a Windows default, not a broken setup. Set the policy once for the current user, or just use `uv run`.

---

## 16. Cheat Sheet

| Task | Command |
| :--- | :--- |
| Create a new project | `uv init myproject` |
| Create the virtual environment | `uv venv` |
| Create it on a specific Python version | `uv venv --python 3.12` |
| Add a runtime dependency | `uv add requests` |
| Add a pinned version | `uv add "requests==2.28.0"` |
| Add a dev-only tool | `uv add --dev pytest ruff` |
| Add notebook support | `uv add ipykernel` |
| Remove a dependency | `uv remove requests` |
| Run a command in the environment | `uv run python main.py` |
| Rebuild the environment from the recipe | `uv sync` |
| Re-resolve and update the lockfile | `uv lock` |
| Upgrade all dependencies | `uv lock --upgrade` |
| Upgrade one dependency | `uv lock --upgrade-package <name>` |
| Install from a requirements file | `uv pip install -r requirements.txt` |
| Import a requirements file into pyproject.toml | `uv add -r requirements.txt` |
| Compile inputs into a pinned requirements file | `uv pip compile requirements.in -o requirements.txt` |
| Export the lockfile to requirements.txt | `uv export --format requirements.txt --output-file requirements.txt` |
| Open the project in the editor | `code .` |
| Select the interpreter (editor) | Command palette → Python: Select Interpreter → the `.venv` entry |
| List installed Python versions | `uv python list --only-installed` |
| Install a Python version | `uv python install 3.12` |
| Remove an unused Python version | `uv python uninstall 3.12` |
| Clear the package cache (rarely needed) | `uv cache clean` |

---

*Further reading:*
- *[uv documentation](https://docs.astral.sh/uv/), the official reference*
- *[Working on projects](https://docs.astral.sh/uv/guides/projects/), init, add, sync, run*
- *[Managing dependencies](https://docs.astral.sh/uv/concepts/projects/dependencies/), constraints and dependency groups*
- *[Exporting a lockfile](https://docs.astral.sh/uv/concepts/projects/export/), generating requirements.txt*
- *[Using Python environments](https://docs.astral.sh/uv/pip/environments/), virtual environments*
- *[Installing Python](https://docs.astral.sh/uv/guides/install-python/), managed interpreters*
- *[Caching](https://docs.astral.sh/uv/concepts/cache/), the shared package cache*
