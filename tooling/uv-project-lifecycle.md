# The uv Project Lifecycle — A Practical Guide

A complete walkthrough of what happens when you create a Python project with [uv](https://docs.astral.sh/uv/): every command, what it touches, the files it generates and what is inside them, and how the pieces fit into one reusable mental model. The same pattern holds from a small project to a large one, so once the lifecycle is clear it does not change with scale. The one case it does not cover is the single *file*: uv can run a standalone script that declares its own dependencies in an inline `# /// script` block, with no project, no `pyproject.toml`, and no project environment at all. That mechanism is a separate topic and is not covered here.

**Sources:**
- [uv documentation](https://docs.astral.sh/uv/), the official reference
- [uv: Working on projects](https://docs.astral.sh/uv/guides/projects/), `init`, `add`, `sync`, `run`
- [uv: Managing dependencies](https://docs.astral.sh/uv/concepts/projects/dependencies/), constraints and dependency groups
- [uv: Using Python environments](https://docs.astral.sh/uv/pip/environments/), virtual environments
- [uv: Installing Python](https://docs.astral.sh/uv/guides/install-python/), managed interpreters
- [uv: Caching](https://docs.astral.sh/uv/concepts/cache/), the shared package cache
- [uv: Locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/), `--locked`, `--frozen`, and automatic locking
- [uv: Python versions](https://docs.astral.sh/uv/concepts/python-versions/), version discovery and pinning
- [uv: Using workspaces](https://docs.astral.sh/uv/concepts/projects/workspaces/), several packages in one repository
- [uv: Exporting a lockfile](https://docs.astral.sh/uv/concepts/projects/export/), generating requirements.txt
- [Python packaging: pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/), the project recipe
- [Python: venv](https://docs.python.org/3/library/venv.html), the standard-library module behind `.venv` internals and the older workflow in Section 17
- [VS Code: Python environments](https://code.visualstudio.com/docs/python/environments), the interpreter picker described in Section 7

---

## How to read this guide

The sections follow the real order you would run things: set up, decide which command starts you off, create a project, add what it needs, run it, then understand the files, structure, and storage that resulted. Each section explains the command, what it changes on disk, and the reasoning behind it, then ends with a short **Notes** list of the things that bite people.

One ordering choice is deliberate and worth flagging. The virtual environment does not appear until Section 8, well after you have already built and run a project. That is not an oversight — it is how uv actually behaves. You are never required to create an environment yourself, so the guide does not teach it as a step. It arrives later as something to understand and manage rather than something to remember to do.

The recurring theme to watch for is a split that runs through everything. A few heavy, reusable resources live **outside** any single project (the Python interpreters and the package cache), while each project keeps only the light, specific things (its recipe and its environment). Hold that split in mind and every behavior below follows from it.

---

## Table of Contents

**Foundations**
- [1. The Starting Point](#1-the-starting-point)
- [2. The Mental Model in One Picture](#2-the-mental-model-in-one-picture)

**Creating and Running a Project**
- [3. Choosing Your Starting Command](#3-choosing-your-starting-command)
- [4. Initialize the Project](#4-initialize-the-project)
- [5. Add Dependencies](#5-add-dependencies)
- [6. Running Code](#6-running-code)
- [7. Select the Interpreter in the Editor](#7-select-the-interpreter-in-the-editor)

**The Environment Itself**
- [8. Create and Manage the Virtual Environment](#8-create-and-manage-the-virtual-environment)

**Structure and Reproducibility**
- [9. The Full File and Folder Layout](#9-the-full-file-and-folder-layout)
- [10. Anatomy of the Generated Files](#10-anatomy-of-the-generated-files)
- [11. Cloning and Rebuilding](#11-cloning-and-rebuilding)
- [12. Durable vs Disposable](#12-durable-vs-disposable)
- [13. Multiple Components in One Repository](#13-multiple-components-in-one-repository)

**Versions and Storage**
- [14. Multiple Python Versions](#14-multiple-python-versions)
- [15. The Shared Cache](#15-the-shared-cache)
- [16. Cleaning Up and Reclaiming Disk Space](#16-cleaning-up-and-reclaiming-disk-space)

**Interoperability**
- [17. The Older Way: requirements.txt](#17-the-older-way-requirementstxt)

**Reference**
- [18. Common Mistakes](#18-common-mistakes)
- [19. Cheat Sheet](#19-cheat-sheet)

---

## 1. The Starting Point

Before any project exists, four things live on the machine independently of any project:

| Resource | Where it lives | Role |
| :--- | :--- | :--- |
| uv and Git | Installed system wide | The tools you drive everything with |
| Base Python interpreter | a `python/` subdirectory of uv's data directory, for example `~/.local/share/uv/python` | The shared interpreter that project environments point back to |
| Package cache | uv's cache directory, `~/.cache/uv` on Unix or `%LOCALAPPDATA%\uv\cache` on Windows | Previously downloaded packages, ready to re-link without re-downloading |
| Installed tools | a `tools/` subdirectory of the same data directory | Command-line tools added with `uv tool install`, each in its own environment |

Rather than memorising these, ask: `uv python dir`, `uv cache dir`, and `uv tool dir` print the current locations, and each is overridable through an environment variable.

These persist across every project. Nothing done inside a project alters them, with one exception — adding a brand new package downloads a copy into the shared cache so future projects can reuse it instantly.

The base interpreter stays pristine. You never install project packages directly into it. Each project gets its own isolated space instead, which is what the rest of this guide builds.

**Notes:**
- Version numbers throughout this guide (`3.14`, `requests>=2.32.0`) are illustrative. Substitute whatever is current; the mechanics are what stay fixed.
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

## 3. Choosing Your Starting Command

Almost every uv walkthrough opens with `uv init`, which quietly assumes you are starting from nothing. Most real work does not. You are usually adopting uv in a folder that already holds code, or picking up a repository someone else already set up. Running the wrong opening command is the most common way to end up fighting the tool, so the decision is worth settling before the first keystroke.

Four situations cover nearly everything:

| Situation | Command | What you get |
| :--- | :--- | :--- |
| A new project, nothing exists yet | `uv init myproject` | the full scaffold: `pyproject.toml`, `.python-version`, `main.py`, `README.md`, `.gitignore`, and a Git repository |
| An existing folder with code but no dependency file | `uv init --bare`, then `uv add <package>` for each one it needs | a minimal `pyproject.toml`, and nothing else touched |
| An existing project tracking dependencies in `requirements.txt` | `uv init --bare`, then `uv add -r requirements.txt` | the old list imported into `pyproject.toml` and locked (Section 17) |
| A cloned project that already uses uv | `uv sync` | the `.venv` rebuilt from the recipe that was committed (Section 11) |

The split between the first two rows is what `--bare` exists for. Plain `uv init` writes a starter application around you: a `main.py` you did not ask for, a `README.md`, a `.gitignore`, and a Git repository. In an empty directory that is convenient. In a folder that already has forty Python files, its own README, and five years of history it is noise at best and a collision at worst. `uv init --bare` writes only the minimal `pyproject.toml` and leaves everything else alone, which is exactly what adoption needs.

The fourth row is the one people get wrong in the other direction. A repository that already contains a `pyproject.toml` and a `uv.lock` needs nothing initialized — the recipe is already written, and your job is to rehydrate it rather than author it. `uv sync` reads what was committed and reconstructs the rest. uv guards this for you: `uv init` refuses to run where a project is already initialized, so the mistake fails loudly instead of quietly producing a second one. The exception is a `pyproject.toml` that carries only tool configuration and no `[project]` table, which uv treats as a folder awaiting a project and fills in rather than rejecting.

Two situations sit outside the table. If the repository holds several components that need dependency lists of their own, such as a web service alongside the library it imports, that is a workspace rather than a single project, and Section 13 covers the layout. And if all you want is to run one throwaway script with a couple of dependencies, you may not need a project at all — uv can run a standalone file that declares its own requirements inline, which is the one case this guide sets aside in the introduction.

**Notes:**
- Every row ends in the same place: a `pyproject.toml` and a `uv.lock` with an environment built from them. The first three build that state, the last one inherits it already committed.
- `uv init --bare` is also the right choice inside a monorepo, where a nested Git repository and a second `.gitignore` would be actively unhelpful.
- The `uv init` forms record a Python version without fetching it; the download waits for the first command that needs a working interpreter. `uv sync` in the last row is that command, so on a clone it does acquire the pinned interpreter (Section 11).
- `uv init --bare` writes no `.python-version`, so a project adopted that way has no interpreter pin until you add one. Follow the adoption with `uv python pin` and commit the result, for the reasons in Section 11.

---

## 4. Initialize the Project

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
| `.gitignore` | Pre-configured to exclude things that should never be committed, such as `.venv` and `__pycache__`. Created alongside the Git repository, so subject to the same condition below. |

It normally also initializes a Git repository (a hidden `.git` folder) so the project is version controlled from the first commit, and this is where the `.gitignore` comes from. The behavior is conditional rather than guaranteed: when the target directory already sits inside an existing repository, uv creates neither the nested `.git` nor the `.gitignore`, on the reasoning that the outer repository already covers it. If you are scaffolding inside a monorepo and wondering where your `.gitignore` went, that is why. Pass `--vcs none` to skip version control deliberately.

What `uv init` does not do: it does not create the virtual environment, and it does not install anything. At this point there is no `.venv` folder and no `uv.lock`. Both appear on their own at the next step — you never ask for either. Section 10 opens each of these files up so you can see what is inside.

### Which Python version uv init chooses

`uv init` has to decide which Python the project targets, and it writes that decision into two different files:

| File | What it receives | Role |
| :--- | :--- | :--- |
| `.python-version` | a bare version, for example `3.14` | which interpreter uv actually uses in this project |
| `pyproject.toml` | `requires-python = ">=3.14"` | the floor that dependency resolution must respect |

Both entries come from the same decision — whichever interpreter uv settled on — but they record it at different precision. `.python-version` gets the minor version as a bare string, and `requires-python` gets that same minor version as a lower bound. A machine running 3.12.7 therefore produces `3.12` and `>=3.12`, never `3.12.7` or `>=3.12.7`. Patch releases are deliberately left out of both: they do not change the language surface a project depends on, and pinning one would mean re-pinning after every security update.

Left to itself, uv does not simply take the newest Python on the machine. It works through a documented search order, and the selection rule changes depending on where it finds a candidate:

1. **uv-managed versions first.** Interpreters uv installed itself are preferred, and among those uv takes the *newest*.
2. **Then system Python on `PATH`**, discovered as `python`, `python3`, or `python3.x` (`python.exe` on Windows). Here the rule inverts: uv takes the *first compatible* interpreter it encounters and stops looking, rather than surveying all of them for the newest.
3. **Then a download.** If nothing suitable is already present, uv fetches a managed version.

That inversion in step 2 is the source of nearly all the surprise. On a machine with no uv-managed interpreters and a system `python3` of 3.10, `uv init` produces a 3.10 project even when 3.13 is also installed further along `PATH`. Nothing is broken — uv found something compatible and stopped. It is also why the same command on two machines can produce two different projects.

Note the one asymmetry worth remembering — an already-installed *system* Python still beats downloading a managed one, so uv reaches for the network last. The managed-versus-system ordering is itself adjustable through the `python-preference` setting, one of many uv settings that live in a `[tool.uv]` table in `pyproject.toml` or in a separate `uv.toml` file.

The reliable habit is to state the version you want:

```bash
uv init --python 3.12 myproject
```

That writes `3.12` into `.python-version` and `requires-python = ">=3.12"` into `pyproject.toml`. If 3.12 is not installed, uv does not fetch it at init time; it downloads it on the first `uv run` or `uv sync`.

To change the version later, do not hand-edit the file. Pin it, and let uv rewrite it:

```bash
uv python pin 3.12
```

The same deferral applies here as at init time. `uv python pin` records a *request*; it does not install anything. If 3.12 is absent from the machine, the pin still succeeds and the download happens on the next command that needs a working interpreter, which is normally `uv sync`. Section 14 goes through this in detail.

If you keep landing on a version you did not want across many new projects, set a machine-wide fallback once instead of correcting each project:

```bash
uv python pin --global 3.12
```

That writes a `.python-version` into your user configuration directory, which uv consults for any directory that has no pin of its own.

Keep `requires-python` in step when you do. The pin decides what runs, while `requires-python` decides what resolves — two different questions, and both need a true answer. If they contradict each other, such as a pin of `3.11` against `requires-python = ">=3.12"`, uv refuses to let the two disagree silently. Depending on your uv version you get an outright error or a warning on the pin, and either way the resolution is the same: change `requires-python` as well.

### Which version to choose

The rules above decide which version you *get*. They say nothing about which one you should *want*, and that answer is usually forced rather than chosen:

- **Match whatever will actually run the code.** If the project deploys to a container, a CI runner, or a cluster sitting on 3.11, develop on 3.11. A version that exists only on your laptop produces bugs that appear exactly once, in production. This constraint outranks everything below it.
- **Otherwise take the latest stable release**, unless a dependency stops you. Newer minors are faster and better supported, and in a project you control there is no cost to being current.
- **Expect the scientific and machine-learning stack to lag.** Compiled libraries ship a separate binary wheel per Python version, and a new minor release routinely waits months for the whole chain to catch up. Until those wheels exist, installing either falls back to a slow build from source or fails outright. If the project leans on that ecosystem, one minor version behind the newest is the low-friction default.
- **Set `requires-python` to what you support, not what you run.** For an application the floor barely matters, since nothing else consumes it. For a library it is a promise — `>=3.13` on a package that would work fine on 3.10 excludes users for no reason. Choose the lowest version you are willing to test against.
- **Never pin a patch version.** `3.12` means any `3.12.x`, which is what you want: security fixes arrive without any action from you. `3.12.7` freezes the project on one build and buys nothing.

What makes this less weighty in uv than in older workflows is that the decision is cheap to reverse. An extra interpreter is a one-time download shared by every project that asks for it, so trying the project on another version is a single command rather than a rebuild of your machine's Python setup. Section 14 covers that side of it.

**Notes:**
- Think of `pyproject.toml` as the project's identity card. It is the durable record of what the project needs.
- If you prefer to create the folder yourself, run `uv init` with no name from inside an existing empty directory.
- `uv init --bare` writes only a minimal `pyproject.toml` — no `.python-version`, no `main.py`, no README, and no version control setup. It is the right choice when adopting uv in a project that already has its own structure.
- `uv init` refuses to run where a project is already initialized, so it cannot quietly overwrite one. A `pyproject.toml` holding only `[tool.*]` configuration is the exception: uv adds the `[project]` table and leaves the rest intact.
- The default template is an application. `--lib` produces a library and `--package` a packaged application, both using a `src/` layout with a `[build-system]` table; Section 10 covers what that table is for.

---

## 5. Add Dependencies

```bash
uv add requests
```

`uv add` does two things at once:

1. **Installs the package into `.venv`**, creating that environment first if it does not exist yet, and pulling the files from uv's cache. If the package has never been fetched, uv downloads it once into the cache first. The install uses hard-links, so the files in the project share the same bytes as the cached copy rather than duplicating them.
2. **Records the package in `pyproject.toml`** under dependencies, and pins the exact resolved version into a `uv.lock` file (created at this point if it does not exist).

That second step is the reproducibility mechanism. `pyproject.toml` records what the project depends on, and `uv.lock` records the exact versions resolved, so the environment can be rebuilt identically later.

By default uv writes a lower-bound constraint for the most recent compatible version, for example `requests>=2.32.0`, rather than a hard pin. The style of that bound is adjustable with `--bounds`. To pin a specific version instead, quote the specifier so the shell does not misread the `==`:

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

Use project commands (`uv add`) for anything the project genuinely depends on. Reach for the pip interface only for throwaway, one-off, or non-project installs, or to interoperate with a `requirements.txt` without adopting it. uv's own guidance is blunter still: it recommends against modifying the project environment by hand at all, and points one-off needs at `uv run --with <package>`, which supplies a dependency for a single command without touching either the recipe or the environment.

**Notes:**
- Use `uv add` rather than `pip install` for project work. Both place files in the environment, but only `uv add` updates the recipe in `pyproject.toml` — a package installed outside the recipe is invisible to anyone rebuilding the project later.
- Editors and notebooks sometimes offer to install a missing package for you (for example `ipykernel` for Jupyter). Prefer running `uv add ipykernel` yourself, so the dependency is tracked rather than installed loosely into the `.venv`.
- Adding a package never touches the global interpreter — it only changes this project's `.venv` and recipe.

---

## 6. Running Code

There are three ways to run code, all using the project's environment:

- **A script in the editor.** Open `main.py` and click Run. It executes with the `.venv` interpreter.
- **A notebook.** Create a `.ipynb` file, write a cell, run it, and pick the `.venv` kernel when prompted. Notebook support needs `ipykernel` in the project, so run `uv add ipykernel` in any project that uses notebooks.
- **From the terminal.** Prefix any command with `uv run`.

```bash
uv run python main.py
```

`uv run` executes inside the project's environment without manual activation, which sidesteps shell activation-policy issues entirely. It is the cleanest default habit.

### The forms of uv run

`uv run` is one command with several shapes, and knowing them removes most of the reasons anyone reaches for activation:

```bash
uv run python main.py                 # run a script through the interpreter
uv run main.py                        # run a file directly, letting uv supply the interpreter
uv run python                         # an interactive REPL inside the environment
uv run python -c "import requests; print(requests.__version__)"
uv run pytest                         # run a tool installed in this project
uv run --with rich python             # add a package for this one command only
uv run --python 3.11 main.py          # run once on a different interpreter
```

The first two lines look interchangeable and usually are, but they are not the same command. `uv run python main.py` hands the file to the project's interpreter, which is what you want almost always. `uv run main.py` hands the file to *uv*, which first checks it for an inline `# /// script` metadata block. If it finds one, uv abandons the project entirely and runs the file in an isolated cached environment built from the block, so none of the project's dependencies are available. For an ordinary file with no such block the two forms agree. Adding `python` is the way to be explicit that you mean the project.

The bare `uv run python` form is the one worth committing to memory. It drops you into a REPL that sees exactly the packages the project has, which makes it the fastest way to answer "is this actually installed, and at what version?" without opening an editor or activating anything. `uv run python -c` is the same check as a one-liner, useful in a script or a CI step.

`uv run pytest` matters for a different reason. Bare `pytest` in an unactivated shell resolves through `PATH` to whatever version happens to be on the system, quietly testing your code with the wrong tool. Prefixing with `uv run` guarantees the project's own copy, the one recorded in `uv.lock`.

Two behaviors sit underneath all of these. Before running anything, `uv run` brings the environment up to date, installing whatever the lockfile requires but leaving unlisted strays in place — the *inexact* sync described in Section 11. That is what makes it self-healing: a deleted `.venv` or a dependency someone else added is repaired on the next invocation rather than reported as an error. And `--with` is the exception to the rule that everything must be declared, supplying a package for a single command through a temporary overlay that touches neither `pyproject.toml` nor the project environment. Reach for it when you want to try a library once, and for `uv add` the moment you want to keep it.

**Notes:**
- On Windows PowerShell, manual activation (`.venv\Scripts\Activate.ps1`) can fail with an execution-policy error — that is a Windows default, not a broken setup. Either run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` once, or simply prefer `uv run`, which never triggers it.
- Inside an activated environment, bare `python` resolves to the project interpreter. Outside it, `python` resolves to whatever is on PATH — usually the base interpreter.

---

## 7. Select the Interpreter in the Editor

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

## 8. Create and Manage the Virtual Environment

By this point you have initialized a project, added dependencies, and run code, and at no stage did you create a virtual environment. One exists anyway. `uv add`, `uv run`, and `uv sync` each create the `.venv` on demand if it is missing, which is why the preceding sections never mention it: **in normal project work you do not run the command below at all.**

```bash
uv venv
```

It is worth knowing regardless, for three reasons. Running it explicitly makes a step visible that is otherwise silent — the fastest way to see what the rest of the guide is describing. The pip interface (Section 5) genuinely does need an environment created first, since it has no project to infer one from. And the flags below are how you build an environment somewhere other than `.venv`, or on an interpreter other than the pinned one.

Whichever way it comes into being, `.venv` is the project's isolated virtual environment, and it always holds the same things:

- A small launcher (`bin/python` on macOS and Linux, `Scripts\python.exe` on Windows) that resolves back to the shared base interpreter. On Unix it is typically a symlink; on Windows the small executable is copied. Either way the interpreter *installation* — the standard library and the rest — is never duplicated, which is what keeps environments cheap.
- A `pyvenv.cfg` config file recording which base Python this environment uses.
- Activation scripts.
- An almost empty `site-packages` folder, which is where installed packages will live.
- An internal `.gitignore` of its own. uv writes this so the environment is excluded from Git regardless of what the project-level `.gitignore` says, which is why a `.venv` is never accidentally committed even in a project where `uv init` did not create one.

The environment starts empty of your packages. It is a sealed space tied to this one project, so its packages can never collide with another project's packages — that isolation is the entire reason virtual environments exist.

### Choosing where it lives and what it runs on

By default the environment is created at `.venv` in the project root, built on the interpreter named in `.python-version`. Both defaults can be overridden:

```bash
uv venv --python 3.12      # build on a specific interpreter
uv venv my-env             # create it under a different name
```

A non-default name is occasionally useful — keeping two environments side by side to check a library against two Python versions, for instance. The cost is that uv and your editor stop finding it automatically, so you have to point at it every time. For ordinary work, leave it as `.venv`.

If the requested version is not installed, uv downloads it. One trap worth naming: `uv venv --python 3.12` builds *this* environment on 3.12 but does not rewrite `.python-version`, so the next command that recreates the environment goes back to the pinned version. To change the project's Python durably, use `uv python pin`, covered with the rest of the version rules in Section 14.

### Activating and deactivating

Activating an environment puts its `bin` directory at the front of your `PATH`, so a bare `python` or `pytest` resolves to this project's copy rather than a system one:

```bash
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\Activate.ps1       # Windows PowerShell
```

The environment name appears as a prefix in your prompt, which is the visible confirmation that it worked. To leave it:

```bash
deactivate
```

`deactivate` restores the `PATH` you had before and does nothing else. It is not a cleanup step and it deletes nothing — the environment is still on disk, ready for the next activation.

You can also skip activation entirely. `uv run` executes a command inside the environment without altering your shell, which is why Section 6 recommends it as the default habit. Activation is still worth understanding, because every older tutorial assumes it and because a long session of many commands is more comfortable inside an activated shell than with `uv run` typed on every line.

### Deleting and rebuilding

The environment is disposable by design, so the repair procedure for almost any broken environment is to throw it away and regenerate it:

```bash
rm -rf .venv                        # macOS / Linux
Remove-Item -Recurse -Force .venv   # Windows PowerShell
uv sync                             # rebuild from pyproject.toml + uv.lock
```

Deactivate first if that environment is currently active, or open a fresh shell. Deleting an environment you are still inside leaves your `PATH` pointing at paths that no longer exist — the source of confusing "command not found" errors until you deactivate.

This costs almost nothing. The rebuild links from the shared cache rather than the network, so it usually completes in seconds, and nothing worth keeping ever lived in `.venv` in the first place.

**Notes:**
- uv's own guidance is that the project environment should not be modified by hand. Let `uv add` and `uv sync` manage its contents, and treat the folder as output rather than a place to work in.
- If uv warns that `VIRTUAL_ENV` does not match the project environment, some unrelated environment is activated in that shell. Run `deactivate` and let uv manage it.
- Shells outside the POSIX family have their own activation scripts, such as `.venv/bin/activate.fish`. Sourcing the plain `activate` in fish will not work.

---

## 9. The Full File and Folder Layout

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
- This is the layout of a single project. A repository holding two components that need separate dependency lists is a different shape, covered in Section 13.
- A `.venv` contains real, working packages (so the environment actually runs), but those package files are usually hard-links to the cache rather than fresh copies, so they cost almost no extra disk space.
- The interpreter is a pointer, not a copy. Ten projects on the same Python version all reference one installed interpreter.

---

## 10. Anatomy of the Generated Files

Section 9 listed the files. This section opens the ones you actually read and edit, so the abstract "recipe" becomes concrete. The rule of thumb: you hand-edit `pyproject.toml` (and occasionally `.gitignore`), while everything else is generated and left alone.

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
    "pytest>=9.0.0",
    "ruff>=0.15.0",
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
sdist = { url = "https://files.pythonhosted.org/.../requests-2.32.3.tar.gz", hash = "sha256:5536541773..." }
wheels = [
    { url = "https://files.pythonhosted.org/.../requests-2.32.3-py3-none-any.whl", hash = "sha256:70761cfe03..." },
]
```

Three things are recorded per package, and they map onto the three reasons the file exists. `version` is the exact release chosen. `dependencies` names what that release drags in, so the transitive graph is written down rather than rediscovered. And `sdist` and `wheels` name the downloadable artifacts with a digest for each, which is where the hash guarantee below comes from. The excerpt above is abbreviated: real URLs and digests are full length, and each artifact also carries its size.

It is generated and maintained by uv (on `uv add`, `uv lock`, and `uv sync`). You never hand-edit it, and you always commit it. This division of labor is the key idea: `pyproject.toml` says what is acceptable, `uv.lock` says what was actually chosen.

The word uv's own documentation uses for the file is *universal*, and it is worth pausing on. `uv.lock` does not record what to install on the machine that generated it; it records what would be installed across every combination of operating system, architecture, and Python version the project supports. One lockfile, resolved once, serves a Linux CI runner and a Windows laptop alike. That is why entries carry environment markers, and it is the property that distinguishes the file from a `pip freeze` dump, which describes exactly one machine and silently misleads on any other.

**Why a lockfile at all.** Since `pyproject.toml` already lists the dependencies, it is fair to ask what the second file buys. Three reasons, in ascending order of importance:

- **Constraints are not versions.** `requests>=2.32.0` describes a *set* of acceptable releases, not one release. Two people running `uv sync` six months apart both satisfy that constraint and both get different code. Reproducibility needs a single answer, and a constraint cannot provide one.
- **Transitive dependencies are recorded nowhere else.** `uv add requests` writes exactly one line into `pyproject.toml`, yet installing it pulls in `certifi`, `idna`, `urllib3`, and more. Those indirect packages are the ones most likely to break you — and the recipe has no record that they even exist. `uv.lock` names every one at an exact version.
- **A lockfile carries hashes.** The `sdist` and `wheels` digests above record what each artifact's contents must be, which is why `uv export` can emit a hash-checked `requirements.txt`, and why a corrupted or tampered download fails the install instead of quietly running.

The one-line version: `pyproject.toml` can tell you what a project wants, and only `uv.lock` can reproduce what it had.

### .python-version — the interpreter pin

A single line:

```text
3.14
```

uv reads it to decide which interpreter this project uses. Write it with `uv python pin` rather than by hand — that keeps the format correct and reports what it changed:

```bash
uv python pin 3.12
```

Commit it. It is what lets a clone land on the interpreter you actually developed against, and Section 11 covers what happens when it is missing. Section 14 is where this file sits in the wider set of rules uv uses to pick a version.

### .gitignore — what stays out of version control

uv writes a starter `.gitignore` that keeps generated and machine-specific files out of the repository. The exact contents are a template that can change between uv releases, so treat this as representative rather than a specification:

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

The important line is `.venv`. The environment is excluded because it is large and fully rebuildable from `pyproject.toml` and `uv.lock`. It is belt and braces rather than the only defense — the environment also carries an internal `.gitignore` of its own, as Section 8 noted. Add your own patterns here as the project grows (for example `.env` for secrets, covered when you start integrating APIs).

### main.py and README.md — the starter content

`main.py` is a runnable hello-world placeholder you replace with real code. Like the `.gitignore`, the exact text is a template and may differ slightly in your version:

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

## 11. Cloning and Rebuilding

This is the payoff of the whole structure. Anyone, including future you on a new machine, can rebuild the exact environment with one command after cloning:

```bash
git clone https://github.com/octocat/myproject.git
cd myproject
uv sync
```

### What uv sync does on a fresh clone

The clone arrives with `pyproject.toml`, `uv.lock`, and `.python-version`, but no `.venv`, because that was never committed. A single `uv sync` reads all three and reconstructs the rest, in this order:

1. **Reads `.python-version`** to learn which interpreter the project expects.
2. **Acquires that interpreter if the machine does not have it.** This is the step that surprises people, and it is the whole answer to "how do I get the right Python for this project?" — you do not get it yourself. uv downloads the pinned version into its shared store automatically. There is no separate install command and no matching versions by hand.
3. **Creates `.venv`** against that interpreter.
4. **Installs the exact versions recorded in `uv.lock`**, hard-linked from the cache, typically in seconds.

So the honest setup procedure for a cloned project is `uv sync` and nothing else. After that, open the editor and select the `.venv` interpreter exactly as in Section 7.

Two flags matter once you want a guarantee rather than a convenience:

```bash
uv sync --locked    # fail if uv.lock is out of date with pyproject.toml
uv sync --frozen    # install straight from uv.lock without re-resolving
```

Plain `uv sync` quietly re-locks when it finds the lockfile has drifted from the recipe — helpful on your own machine, dangerous in a pipeline. Use `--locked` in CI, where drift should break the build loudly instead of being silently repaired. `--frozen` is the weaker guarantee of the two: it skips the up-to-date check entirely and installs whatever the lockfile says, which is what you want when the lockfile is already known good and you would rather not pay for or risk the comparison. The two flags are mutually exclusive; passing both is an error, because "check the lockfile" and "do not check the lockfile" cannot both be true.

### When the clone is missing pieces

Not every repository is well kept. The cases you will actually meet:

| What the clone contains | What to run | What happens |
| :--- | :--- | :--- |
| `pyproject.toml` + `uv.lock` | `uv sync` | exact reproduction, the good case |
| `pyproject.toml`, no `uv.lock` | `uv sync` | uv resolves fresh and writes a lockfile; commit it |
| no `.python-version` | `uv sync` | uv falls back to the `requires-python` floor, so you may land on a different interpreter than the author used |
| only `requirements.txt` | see Section 17 | import it with `uv add -r` to adopt the modern layout |

The third row is the argument for committing `.python-version`. Without it, `requires-python = ">=3.10"` permits everything from 3.10 upward and your machine decides. Nothing visibly fails, which is the problem — the install succeeds, and you have quietly stopped testing what the author tested.

### Rebuilding an existing checkout

The same command repairs a project you already have: delete `.venv` and run `uv sync`, as covered in Section 8. `uv sync --reinstall` is the gentler option, forcing packages to be reinstalled without removing the environment first.

### uv lock vs uv sync

Two project commands sit behind reproducibility, and they do different jobs:

- **`uv lock`** resolves your dependencies and writes `uv.lock`. It updates the *record* of exact versions but does not change your installed environment. Run it to refresh the lockfile (for example after editing constraints in `pyproject.toml`) without reinstalling anything yet.
- **`uv sync`** makes the `.venv` match `uv.lock`. It installs what is missing and removes what should not be there, so the environment exactly equals the locked record. Run it to build or update the environment.

There is an asymmetry here worth knowing. `uv sync` syncs *exactly* by default, removing anything the lockfile does not list, while `uv run` syncs *inexactly*, installing what is required but leaving strays alone. So a package that survives repeated `uv run` invocations can vanish the moment you run `uv sync`. `--inexact` and `--exact` swap the behaviors if you need the other one.

In practice you rarely call `uv lock` on its own, because the commands that change dependencies lock implicitly: `uv add` updates the lockfile as it adds, and `uv sync` re-locks if the recipe changed. `uv lock` is the explicit, environment-free way to re-resolve, useful in CI or when you want to review a lockfile change as a separate step before installing. Its verification form is `uv lock --check`, which reports whether the lockfile is current without touching either the lockfile or an environment; it is the same assertion `--locked` makes on other commands, available on its own when there is nothing to install. A one-line way to hold them apart: `uv lock` decides what should be installed, and `uv sync` makes it so.

**Updating to newer versions.** uv never upgrades on its own; it does not treat the lockfile as stale when new releases appear, so upgrades are always explicit. The upgrade flags do it:

```bash
uv lock --upgrade                    # re-resolve everything to the newest allowed versions
uv lock --upgrade-package requests   # bump just one package
uv sync                              # apply the updated lockfile to the .venv
```

The flags also work directly on `uv sync` (for example `uv sync --upgrade-package requests`) to update and install in one step. One subtlety worth knowing: `uv add requests` on a package that is already present does **not** upgrade it, because the existing lock already satisfies the constraint. Use `--upgrade-package requests` for that, or change the constraint itself (edit `pyproject.toml`, or run `uv add "requests>=3"`) when you want to allow a new major version. The rule of thumb: `uv add` changes *what is allowed*, `--upgrade` changes *what is chosen* within that.

**Notes:**
- `uv sync` rebuilds the whole environment from the recipe — `uv add` is for introducing a new dependency. Use sync when rehydrating a known project, add when changing what it depends on.
- A clone never needs `uv venv` first. `uv sync` creates the environment as part of its job.
- Because rebuilds pull from the cache, they rarely re-download anything. The exception is a pinned Python version the machine has never seen, which downloads once and is then shared.
- Upgrades are opt-in: nothing moves to a newer version until you run `uv lock --upgrade` (or `--upgrade-package`). This is what keeps builds reproducible by default.

---

## 12. Durable vs Disposable

The single mental model to carry into every project:

| Category | What | Behavior |
| :--- | :--- | :--- |
| Durable (commit, keep) | `pyproject.toml`, `uv.lock`, your code | The recipe and the work. The source of truth. |
| Disposable (rebuildable) | `.venv` | The assembled environment. Delete freely; `uv sync` rebuilds it. |
| Shared and persistent (outside the project) | base interpreters, package cache | Untouched by project deletion. Make rebuilds fast. Removed only by explicit commands. |

Two one-directional rules make the separation safe:

- Deleting from a **project** never affects the **cache** or the installed **interpreters**.
- Clearing the **cache** never affects an existing project's already-built `.venv`. That environment's hard-links keep its own package files alive independently of the cached originals.

The interpreter is the one place this symmetry breaks, and it follows from Section 8: a `.venv` holds a *pointer* to the base installation rather than a copy of it. Uninstalling a Python version therefore does break every environment built on it — the launcher resolves to something that is no longer there. Nothing durable is lost, since `uv sync` re-downloads the version and rebuilds, but it is the one removal that costs more than disk space.

**Notes:**
- A project's whole lifecycle follows from this table: initialize, add what you need, select the interpreter, build and run, commit the durable files. The `.venv` can be discarded at any point.
- When you delete a project folder, its `.venv` goes with it — the environment lives inside the folder by default.

---

## 13. Multiple Components in One Repository

Everything so far has assumed one project: one `pyproject.toml`, one `.venv`, one dependency list. Repositories frequently hold more than that — exploratory notebooks at the top level with a web application in a subfolder, or a service alongside the library it imports. The question is whether those parts share one environment or each get their own, and uv answers it entirely by where it finds `pyproject.toml` files.

### How uv decides what the project is

uv does not treat your current directory as the project. It walks upward from the working directory, checking it and each parent for a `pyproject.toml`, and the first one it finds is the project. `.python-version` is discovered the same way, which is why Section 14's priority table says "the project, or the nearest parent directory".

The practical consequence is that a project at the repository root serves every subfolder beneath it:

```text
myrepo/
├── .venv/                  # one environment, at the root
├── pyproject.toml          # one recipe, at the root
├── uv.lock
├── analysis.ipynb
└── webapp/
    └── app.py              # no pyproject.toml of its own
```

Running `uv run uvicorn app:app` from inside `webapp/` finds the root project, uses the root `.venv`, and behaves exactly as it would from the root. Nothing extra is required, and there is no second environment to keep in step.

### When one project is the right answer

Sharing a single environment is the simpler arrangement and usually the correct one. It fits when the parts are developed together and their dependencies do not fight. Four consequences are worth knowing before you commit to it:

- **One dependency set, one resolution.** Both halves draw from the same `dependencies` list and the same lockfile, so a version conflict between them has no escape hatch. Notebook-only tooling can at least be held apart with `uv add --dev ipykernel`, keeping it out of what a consumer of the project would install.
- **Imports are not automatic.** A default application carries no `[build-system]` table, so the project is never installed into its own `.venv`. A statement like `from webapp import app` therefore depends on the working directory and `sys.path` rather than on the environment. When the parts genuinely need to import each other, create the project with `uv init --package` so it becomes installable, or move to a workspace.
- **Notebooks run from their own folder.** Most editors set a notebook's working directory to the notebook's location rather than the project root, which is why a relative path that works in a script fails in a notebook sitting one level down. Resolve paths from a known anchor instead.
- **Deployment carries everything.** A container built from one shared environment ships the notebook stack alongside the web application. If only one part is ever deployed, that is an argument for splitting.

### When a subfolder needs a project of its own

Add a `pyproject.toml` to `webapp/` and the discovery rule above changes the answer. uv stops at the nearest one, treats `webapp/` as its own project with its own `.venv` and its own lockfile, and ignores the root entirely. That switch happens silently, with no warning that the root has stopped governing, which makes it a genuine surprise the first time.

If the two really are separate concerns, declare the arrangement instead of stumbling into it. Add a workspace table to the root `pyproject.toml`:

```toml
[tool.uv.workspace]
members = ["webapp", "packages/*"]
exclude = ["packages/scratch"]
```

Each member keeps its own `pyproject.toml` with its own dependencies, while the workspace shares a single `uv.lock` and a single `.venv` at the root. `members` is required and `exclude` is optional, both take globs, every directory a glob matches must contain a `pyproject.toml`, and the root counts as a member itself.

The commands shift slightly once a workspace exists:

| Task | Command |
| :--- | :--- |
| Lock the entire workspace at once | `uv lock` |
| Sync the root member | `uv sync` |
| Sync every member together | `uv sync --all-packages` |
| Run a command inside one member | `uv run --package webapp uvicorn app:app` |
| Add a dependency to one member | `cd webapp` then `uv add fastapi` |

For one member to depend on another, name it as a dependency and point uv at the workspace rather than at a package index:

```toml
[project]
dependencies = ["shared-lib"]

[tool.uv.sources]
shared-lib = { workspace = true }
```

Dependencies between members are installed as editable, so an edit inside the library is visible to the application immediately with no reinstall step.

### Choosing between the three layouts

| Layout | Use when |
| :--- | :--- |
| One project, plain subfolders | the parts share a dependency set and are always developed and deployed together |
| A workspace | the parts need distinct dependency lists, import one another, or deploy separately |
| Separate projects | the parts are unrelated, or their requirements genuinely conflict |

The third row is the one uv's own documentation is firm about. Workspaces are not meant for members with conflicting requirements, and not for members that each want an environment of their own. A workspace resolves to a single `requires-python` for everything, taken as the intersection of all members' values, so a member needing an older Python than the rest cannot be accommodated at all. When that is the situation, separate projects joined by path dependencies give you the isolation a workspace deliberately refuses:

```toml
[tool.uv.sources]
shared-lib = { path = "../shared-lib", editable = true }
```

The trade is that `uv run --package` is no longer available, so each command has to be run from the relevant directory.

**Notes:**
- `uv init` inside an existing project adds the new directory as a workspace member automatically, creating the `[tool.uv.workspace]` table if there is not one yet. Convenient, and occasionally surprising.
- Point the editor at the repository root rather than a subfolder. Opened at `webapp/`, the interpreter picker will not offer a `.venv` that lives one level up.
- Plain `uv sync` at a workspace root syncs the root member only, and because it is exact by default it will uninstall other members' packages. Use `--all-packages` when you want everything present at once.
- A root that exists only to organize the members can drop its `[project]` table entirely and keep just `[tool.uv.workspace]`. Such a virtual root has no dependencies of its own and is never published.
- Python cannot enforce dependency isolation, so uv cannot guarantee that one member only imports what it declared. A member may import a package another member pulled in and appear to work until it is built alone.

---

## 14. Multiple Python Versions

Projects run on different Python versions side by side with no conflict — the ordinary way to keep one project on 3.12 because a library has not caught up, while everything else moves to 3.14. Each interpreter is stored once in uv's central store and shared by every project that asks for it. This section collects the rules for controlling which one a project gets.

### Which version a command actually uses

Several sources can have an opinion. uv resolves them in this order:

| Priority | Source | Scope |
| :--- | :--- | :--- |
| 1 | `--python` on the command | this single invocation |
| 2 | `.python-version` in the project, or the nearest parent directory | the project, and travels with the repository |
| 3 | `.python-version` in your user configuration directory | every project on this machine without a pin of its own |
| 4 | `requires-python` in `pyproject.toml` | a floor rather than a choice: the first compatible version wins |
| 5 | uv's discovery order | whatever the machine offers, by the rules in Section 4 |

uv looks for `.python-version` in the working directory and each of its parents, stopping at the boundary of the project (or, in a repository holding several related packages, the workspace containing them, as Section 13 describes), then falling back to the user configuration directory. Rather than reasoning through the table, you can simply ask:

```bash
uv python find
```

The distinction between the top two rows is the one that catches people out. `uv run --python 3.11 main.py` runs once on 3.11 and changes nothing on disk, which makes it the right tool for a quick compatibility check against another version. `uv python pin 3.11` changes the project for everyone who clones it.

### Seeing what is available

Before pinning, installing, or removing anything, the useful first move is to look at what the machine already has and what uv could fetch:

```bash
uv python list
```

The output is two columns. The left names an exact build; the right is either the path where that build lives or a marker saying it is available to download:

```text
cpython-3.14.0-linux-x86_64-gnu   <download available>
cpython-3.13.6-linux-x86_64-gnu   <download available>
cpython-3.12.11-linux-x86_64-gnu  /home/you/.local/share/uv/python/cpython-3.12.11-linux-x86_64-gnu/bin/python3.12
cpython-3.10.12-linux-x86_64-gnu  /usr/bin/python3.10
cpython-3.10.12-linux-x86_64-gnu  /usr/bin/python3 -> python3.10
pypy-3.11.13-linux-x86_64-gnu     <download available>
```

Read the left column as implementation, version, operating system, architecture, and the C library it was built against — `gnu` or `musl` on Linux, and `none` where the field does not apply. `cpython` is the standard implementation; `pypy` and `graalpy` are alternatives uv can also install. Special builds appear as a suffix on the version instead, so a free-threaded 3.14 shows up as `cpython-3.14.0+freethreaded-...`. The right column is what makes the listing useful: a path means the interpreter is on the machine right now, and `<download available>` means it is one command away.

That path column is also where the trap named in the notes below becomes visible. The last two `3.10.12` rows are one interpreter listed twice, once under its real name and once under a `python3` shim pointing at it. Rows are not installations. Count distinct versions in the left column, and read the arrows in the right one.

Four commands cover the questions you will actually ask here:

| Command | Answers |
| :--- | :--- |
| `uv python list` | what is here, plus everything I could download |
| `uv python list --only-installed` | what is already on this machine, nothing else |
| `uv python list --all-versions` | every patch release, not just the newest of each minor |
| `uv python find` | which single interpreter *this directory* would actually use |

The default view hides old patch releases and builds for other operating systems, which is why the list is readable at all. `--only-installed` is the one to reach for when auditing, and it is the form used in Section 16 for deciding what is safe to uninstall. `uv python find` answers a different question from all three: not what exists, but what the resolution rules above have settled on here and now.

### Installing and upgrading versions

```bash
uv python install 3.12              # install a version explicitly
```

Explicit installation is rarely necessary, since any command that needs a missing version downloads it. It is worth doing when you want the download to happen now rather than in the middle of something else — before boarding a flight, say.

This is also the place to be precise about what `uv python pin` does, because the two commands are easy to conflate. `uv python pin 3.12` writes `3.12` into `.python-version` and checks the request against `requires-python`, and that is the whole of it. It does not download an interpreter, it does not rebuild `.venv`, and it does not touch `pyproject.toml`. If 3.12 is not installed, the pin still succeeds and nothing else happens until the next command that needs a real interpreter, at which point uv downloads it. The clean sequence for switching a project's Python is therefore:

```bash
uv python pin 3.12    # record the decision
uv sync               # acquire the interpreter if needed, rebuild .venv against it
```

If `requires-python` still says `>=3.14`, uv will not accept the pin quietly — it errors, or warns and then fails at the next `uv sync`, so a version change usually means editing that bound too. This is the one legitimate reason to hand-edit `pyproject.toml` during a version change, and the edit is why the `uv sync` above is doing real work rather than repeating itself.

Patch versions have a subtlety of their own. A pin of `3.12` is a request for any `3.12.x`, and uv will not fetch a newer patch on its own. When you do upgrade, existing environments follow without being rebuilt:

```bash
uv python upgrade 3.12    # one minor version
uv python upgrade         # every uv-managed version
```

Treat this one as provisional. uv's documentation still marks patch upgrading as a *preview* feature, meaning the behavior is experimental and may change, so verify it against the docs before building a routine around it. As currently implemented, uv keeps each managed interpreter behind a minor-version directory that points at the current patch, so an environment built on `3.12` starts using `3.12.11` transparently. The exception is an environment created against an explicit patch, such as `uv venv --python 3.10.8`, which stays exactly where it was put. Minor versions never upgrade this way — moving from 3.12 to 3.13 can change dependency resolution, so uv makes you ask for it.

Deleting a project does not remove the version it used. The interpreter lives outside the project, so it stays installed and ready — exactly like the cache. Section 16 covers removing versions you no longer want.

**Notes:**
- Other tools list interpreters too, and they share the shim problem shown above: `python`, `python3`, and a "latest patch" alias can all be one install. Count distinct versions wherever you are reading.
- Disk cost of an extra version is a one-time interpreter download, shared afterward. Idle RAM cost is zero, since only a running process uses memory.
- A new Python version is the one case that genuinely downloads something new. A new package version downloads once and then re-links.
- `uv python pin --global` sets a fallback for every project on the machine that has no pin of its own — the fix for repeatedly landing on a version you did not want.

---

## 15. The Shared Cache

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

## 16. Cleaning Up and Reclaiming Disk Space

Sections 14 and 15 established that interpreters and cached packages deliberately outlive the projects that pulled them in. That is the right default while you are working. Across a few years of projects it also means uv accumulates storage in three places you never explicitly chose, so knowing how to audit and reclaim them is the closing stage of the lifecycle.

### Where the space actually goes

```bash
uv cache dir      # the shared package cache
uv python dir     # installed interpreters
uv tool dir       # tools installed with uv tool install
```

The cache is normally by far the largest of the three. On Unix it defaults to `$XDG_CACHE_HOME/uv` or `~/.cache/uv`, and on Windows to `%LOCALAPPDATA%\uv\cache`. Point a disk usage tool at those paths and get real numbers before deleting anything.

The third path is the one that surprises people. `uv tool install` and `uvx` create environments of their own outside any project, so anything on the machine that shells out to `uvx` has been quietly filling that directory. Those have nothing to do with the projects you remember creating.

### Retiring a single project

Deleting the project folder is the entire cleanup:

```bash
rm -rf myproject
```

The `.venv` lives inside the folder, so it goes with it, and nothing else on the machine is affected. That is exactly the guarantee Section 12 described.

What this does **not** do is shrink the cache. The packages that project used stay cached at full size, and neither `uv cache clean` nor `uv cache prune` takes a project as an argument, so there is no command that means "remove everything only this deleted project needed". Reclamation is a cache-level operation, not a per-project one. The productive way to read that is as a deliberate speed-up: those entries are already paid for, and the next project wanting the same packages installs them instantly.

### Pruning the cache

```bash
uv cache prune
```

`uv cache prune` removes unused cache entries, including entries left behind by earlier versions of uv, and it is documented as safe to run periodically. That last clause is the important one. Prune is routine maintenance rather than surgery — nothing you currently depend on is at risk, so there is no reason to hesitate before running it.

Where it pays off is a machine that has lived through many uv releases. uv versions its internal cache buckets, and a bucket written by an incompatible older version is pure dead weight that prune knows how to identify. On a well-used laptop this can free a substantial amount of space in seconds.

### Clearing the cache

```bash
uv cache clean            # remove every entry
uv cache clean requests   # remove entries for one package
```

`uv cache clean` empties the cache completely. Reach for it when you suspect cache corruption, or when you need the space immediately and accept that the next install of everything re-downloads. The single-package form is the more useful one day to day, for forcing a fresh fetch of one misbehaving dependency.

Neither command harms an existing `.venv`. Those environments hold hard-links to the package files, so the bytes survive as long as an environment still references them, and the projects keep working after the cache is gone.

### Removing interpreters and tools

```bash
uv python list --only-installed
uv python uninstall 3.11

uv tool list
uv tool uninstall ruff
```

Interpreters are the one category worth being conservative about. Each is a modest one-time download and each is shared by every project requesting that version, so there is little to reclaim. Removing one also reaches into projects you are not thinking about: any existing `.venv` built on that version points at the installation you just deleted and stops working until `uv sync` rebuilds it. Nothing durable is lost, but the cost is other projects' environments rather than a wasted minute. List first, and remove only versions that nothing pins.

### A full decommission

Putting it together, retiring a batch of finished work looks like this:

```bash
rm -rf old-project-1 old-project-2   # delete the projects and their .venv folders
uv cache prune                       # reclaim stale cache entries
uv tool list                         # audit tools no longer used
uv python list --only-installed      # audit interpreters nothing pins
```

**Notes:**
- Never delete files inside the cache directory by hand. uv's documentation is explicit that modifying the cache directly is unsafe — the metadata it maintains will no longer match what is on disk. Use `uv cache prune` or `uv cache clean` instead.
- `uv cache prune --ci` is a different tool for a different job. It discards pre-built wheels while keeping wheels built from source, which is the right tradeoff at the end of a CI run and the wrong one on a laptop.
- If a cache command reports that it is waiting on other uv processes, that is the cache lock behaving correctly. Let it finish, or close the editor terminal holding an environment open.
- Deactivate before deleting an environment you are currently inside, or the shell keeps a stale `PATH` until you do.

---

## 17. The Older Way: requirements.txt

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
uv add -r requirements.txt  # import deps, write uv.lock, and build the .venv
```

Two commands, not three. A trailing `uv sync` is a common addition here and it does nothing: `uv add` resolves, writes `uv.lock`, and installs into `.venv` as part of running, exactly as the table in Section 5 records. Running sync afterwards re-checks an environment that already matches the lockfile and reports no work. It is harmless, but it is not a step, and treating it as one obscures when `uv sync` is genuinely required — after a hand-edit to `pyproject.toml`, on a fresh clone, or after deleting the environment, all cases where something changed outside a uv command.

The important choice here is `uv add -r` rather than `uv pip install -r`. The goal of migrating is to make `pyproject.toml` the source of truth, and only `uv add` records the dependencies there; `uv pip install -r` would merely install them into the environment and leave the recipe empty. Once this is done, `pyproject.toml` and `uv.lock` are authoritative. Retire the old `requirements.txt`, or keep regenerating it with `uv export` if a deploy target still expects it. A fuller, step-by-step walkthrough lives in the companion guide, [Anatomy of a Python Project](../python/python-project-anatomy.md).

### Producing requirements.txt from a uv project

When a deployment target or tool expects `requirements.txt` but your project uses `pyproject.toml` and `uv.lock`, export one from the lockfile:

```bash
uv export --format requirements.txt --output-file requirements.txt
```

Treat the result as a build artifact. `pyproject.toml` and `uv.lock` stay authoritative, and you regenerate `requirements.txt` whenever something downstream needs it, such as Docker images, legacy CI, or serverless deploy targets.

### The standardized successor: pylock.toml

`requirements.txt` never had a specification. [PEP 751](https://peps.python.org/pep-0751/) gives Python a standardized resolution format, `pylock.toml`, intended to replace it. Being tool-agnostic, a `pylock.toml` written by one tool can in principle be installed by another, which is precisely what `requirements.txt` could never promise.

uv keeps `uv.lock` for project work, because some of what it records cannot be expressed in the standard format, but it treats `pylock.toml` as a first-class export and install target:

```bash
uv export --format pylock.toml                    # from a uv.lock
uv pip compile requirements.in -o pylock.toml     # from loose requirements
uv pip sync pylock.toml                           # install from one
```

If you are exporting for a deploy target today, `requirements.txt` remains the safe default on compatibility grounds. `pylock.toml` is where the ecosystem is heading.

**Notes:**
- uv recommends against maintaining both a `uv.lock` and a hand-edited `requirements.txt` as sources of truth. Pick `pyproject.toml` plus `uv.lock` as authoritative, and export `requirements.txt` only as a generated artifact.
- `requirements.txt` has no concept of dependency groups, so runtime and dev dependencies blur together unless you split into separate files (for example `requirements.txt` plus `requirements-dev.txt`).
- For a new project, prefer the modern workflow. Reach for `requirements.txt` only to interoperate with something that requires it.

---

## 18. Common Mistakes

**Installing project packages into the global interpreter.** A new project's `.venv` starts empty and is sealed off from the global Python, so packages added globally are not visible inside it. Keep the base interpreter pristine and add packages per project.

**Using `pip install` instead of `uv add` for project dependencies.** Both place files in the environment, but only `uv add` records the package in `pyproject.toml`. Packages installed outside the recipe vanish when someone rebuilds the project.

**Hand-editing `uv.lock`.** It is generated. Change constraints in `pyproject.toml` and let uv re-resolve. Editing the lockfile directly invites inconsistency between the two.

**Hard-pinning everything with `==` in `pyproject.toml`.** Pinning belongs in `uv.lock`, which uv maintains. Keep ranges (`>=`) in `pyproject.toml` so `uv sync` can still pick up compatible updates.

**Putting dev tools in `dependencies`.** Test runners and linters belong in `[dependency-groups]` via `uv add --dev`, so they are not forced onto anyone who consumes the project.

**Maintaining both `uv.lock` and a hand-edited `requirements.txt`.** Keep `pyproject.toml` and `uv.lock` authoritative, and treat any `requirements.txt` as a generated artifact you export with `uv export`, not a second source of truth.

**Letting the editor install a missing package loosely.** Accepting an editor prompt to install something like `ipykernel` puts it in the `.venv` but not in the recipe. Run `uv add ipykernel` instead, so it is tracked.

**Committing the `.venv` folder.** It is large, machine specific, and fully rebuildable. The default `.gitignore` excludes it; do not override that.

**Assuming many listed Python paths mean many installs.** `uv python list` prints one row per path that reaches an interpreter, so a single install appears several times behind shims and aliases. Count distinct versions in the left column, not rows.

**Assuming `uv init` picked the newest Python.** Newest wins only among uv-managed interpreters. Falling back to system Python, uv takes the first compatible one on `PATH`, so a system `python3` of 3.10 beats a 3.13 sitting further down the path. Pass `uv init --python 3.12` whenever the version matters.

**Expecting `uv python pin` to install the interpreter.** It writes `.python-version` and validates the request, nothing more. The download happens on the next command that needs a real interpreter, so follow a pin with `uv sync` if you want the environment on the new version now.

**Running `uv sync` immediately after `uv add`.** `uv add` already resolves, locks, and installs. The trailing sync is a no-op that makes the pair look like two required steps. Save `uv sync` for when something changed outside a uv command.

**Running `uv init` in a project you just cloned.** A repository with a `pyproject.toml` needs `uv sync`, not initialization. uv refuses the init, but the instinct to reach for it is the sign of a wrong mental model: the recipe already exists and you are rebuilding from it.

**Dropping a `pyproject.toml` into a subfolder without declaring a workspace.** uv stops at the nearest one, so that folder silently becomes its own project with its own `.venv` and lockfile, and the root stops governing it. Declare `[tool.uv.workspace]` at the root when the split is intentional.

**Changing the Python version by any route other than `uv python pin`.** Hand-editing `.python-version` risks the format, and `uv venv --python 3.12` builds one environment on 3.12 while leaving the pin untouched, so the next rebuild reverts. Pin it, and keep `requires-python` consistent.

**Not committing `.python-version`.** Without it a clone falls back to the `requires-python` floor and every machine picks its own interpreter. Nothing fails visibly, which is precisely why it is worth avoiding.

**Treating `deactivate` as cleanup.** It restores your `PATH` and does nothing else. The environment is still on disk; deleting the folder is what reclaims the space.

**Reaching for `uv cache clean` when you mean `uv cache prune`.** `clean` empties the cache and forces every future install to re-download. `prune` removes only unused entries and is safe to run periodically. For routine disk maintenance prune is the command; clean is for suspected corruption or a hard reset.

**Expecting a deleted project to shrink the cache.** It will not. The cache is shared and uv has no per-project reclamation, so deleting the folder frees the project's own space and nothing more. Run `uv cache prune` separately as cache maintenance.

**Expecting a global install to flow into new projects.** It will not. Isolation is one-directional and deliberate; each project declares and installs its own dependencies.

**Fighting PowerShell activation errors.** A script-execution-policy error on `Activate.ps1` is a Windows default, not a broken setup. Set the policy once for the current user, or just use `uv run`.

---

## 19. Cheat Sheet

**Creating and cloning**

| Task | Command |
| :--- | :--- |
| Create a new project | `uv init myproject` |
| Create one on a specific Python version | `uv init --python 3.12 myproject` |
| Add uv to an existing project | `uv init --bare` |
| Adopt an existing `requirements.txt` project | `uv init --bare` then `uv add -r requirements.txt` |
| Set up a freshly cloned project | `uv sync` |
| Sync, failing if the lockfile drifted | `uv sync --locked` |
| Sync straight from the lockfile | `uv sync --frozen` |

**Python versions**

| Task | Command |
| :--- | :--- |
| Pin the project's Python version | `uv python pin 3.12` |
| Set a machine-wide fallback version | `uv python pin --global 3.12` |
| Show which interpreter would be used | `uv python find` |
| Run once on a different version | `uv run --python 3.11 main.py` |
| List installed versions and available downloads | `uv python list` |
| List installed Python versions only | `uv python list --only-installed` |
| List every patch release | `uv python list --all-versions` |
| Install a Python version | `uv python install 3.12` |
| Upgrade to the latest patch | `uv python upgrade 3.12` |
| Remove an unused Python version | `uv python uninstall 3.12` |

**The virtual environment**

| Task | Command |
| :--- | :--- |
| Create the virtual environment | `uv venv` |
| Create it on a specific Python version | `uv venv --python 3.12` |
| Activate it (macOS / Linux) | `source .venv/bin/activate` |
| Activate it (Windows PowerShell) | `.venv\Scripts\Activate.ps1` |
| Leave it | `deactivate` |
| Delete it | `rm -rf .venv` |
| Rebuild it after deleting | `uv sync` |
| Force reinstall without deleting | `uv sync --reinstall` |

**Dependencies and running**

| Task | Command |
| :--- | :--- |
| Add a runtime dependency | `uv add requests` |
| Add a pinned version | `uv add "requests==2.28.0"` |
| Add a dev-only tool | `uv add --dev pytest ruff` |
| Add notebook support | `uv add ipykernel` |
| Remove a dependency | `uv remove requests` |
| Run a script in the environment | `uv run python main.py` |
| Open a REPL in the environment | `uv run python` |
| Check something with a one-liner | `uv run python -c "import requests"` |
| Run a tool installed in the project | `uv run pytest` |
| Run with a package added for this call only | `uv run --with rich python` |
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

**Workspaces**

| Task | Command |
| :--- | :--- |
| Declare a workspace | add `[tool.uv.workspace]` with `members` to the root `pyproject.toml` |
| Lock the whole workspace | `uv lock` |
| Sync every member at once | `uv sync --all-packages` |
| Run a command in one member | `uv run --package webapp pytest` |
| Add a dependency to one member | `cd webapp` then `uv add fastapi` |
| Depend on a sibling member | `shared-lib = { workspace = true }` under `[tool.uv.sources]` |

**Cleanup and storage**

| Task | Command |
| :--- | :--- |
| Show the cache directory | `uv cache dir` |
| Show the interpreter directory | `uv python dir` |
| Show the tool directory | `uv tool dir` |
| Reclaim unused cache entries | `uv cache prune` |
| Clear the cache for one package | `uv cache clean requests` |
| Clear the entire cache (hard reset) | `uv cache clean` |
| List installed tools | `uv tool list` |
| Remove an installed tool | `uv tool uninstall ruff` |
| Retire a project | `rm -rf myproject` |

---

*Further reading:*
- *[uv documentation](https://docs.astral.sh/uv/), the official reference*
- *[Working on projects](https://docs.astral.sh/uv/guides/projects/), init, add, sync, run*
- *[Managing dependencies](https://docs.astral.sh/uv/concepts/projects/dependencies/), constraints and dependency groups*
- *[Exporting a lockfile](https://docs.astral.sh/uv/concepts/projects/export/), generating requirements.txt*
- *[Using Python environments](https://docs.astral.sh/uv/pip/environments/), virtual environments*
- *[Installing Python](https://docs.astral.sh/uv/guides/install-python/), managed interpreters*
- *[Locking and syncing](https://docs.astral.sh/uv/concepts/projects/sync/), --locked, --frozen, and automatic locking*
- *[Python versions](https://docs.astral.sh/uv/concepts/python-versions/), version discovery and pinning*
- *[Using workspaces](https://docs.astral.sh/uv/concepts/projects/workspaces/), several packages in one repository*
- *[Caching](https://docs.astral.sh/uv/concepts/cache/), the shared package cache*
