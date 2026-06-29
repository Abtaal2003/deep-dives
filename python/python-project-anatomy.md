# Anatomy of a Python Project — A Complete Guide

A thorough tour of what actually makes up a Python project: every file, every folder, and the reasoning behind each one. The guide builds a project the classic way first — with the standard-library `venv`, `pip`, and `requirements.txt` — because that workflow is everywhere and it teaches the underlying mechanics. A generic project is the spine, a small web app is the running sub-example, and the modern `uv` tool is introduced at the very end as the thing that compresses all of it.

**Sources:**
- [Python Packaging User Guide](https://packaging.python.org/) — the authoritative reference
- [Packaging a project tutorial](https://packaging.python.org/en/latest/tutorials/packaging-projects/) — structure and metadata
- [Writing pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) — the modern config file
- [venv — Creation of virtual environments](https://docs.python.org/3/library/venv.html) — standard-library environments
- [pip user guide](https://pip.pypa.io/en/stable/user_guide/) — installing and requirements files
- [python-dotenv](https://github.com/theskumar/python-dotenv) — loading `.env` files
- [The uv Project Lifecycle](../tooling/uv-project-lifecycle.md) — companion guide for the modern uv workflow

---

## How to read this guide

The guide assembles a project from nothing, introducing each file at the moment it first matters and explaining it in full before moving on. The order is deliberate: structure first, then how code is organized into modules and packages, then environments and dependencies, then configuration and secrets, then the supporting and tooling files, then a complete web-app example, and finally the modern `uv` method.

Two example projects run throughout. The **generic project** (`myproject`) stands in for any script or library. The **web app** (`myapp`) is a small Flask/FastAPI-style service used wherever a file only makes sense in a web context, such as `app.py`, `templates/`, `static/`, and `.env`.

Each section ends with a short **Notes** list of the things that trip people up. Where a file is generated rather than written by hand, that is called out, because the rule that separates "commit this" from "ignore this" is the same rule throughout.

---

## Table of Contents

**Foundations**
- [1. What a Python Project Is](#1-what-a-python-project-is)
- [2. Modules, Packages, and Imports](#2-modules-packages-and-imports)

**Structure**
- [3. Flat Layout vs src Layout](#3-flat-layout-vs-src-layout)
- [4. Source Code and Entry Points](#4-source-code-and-entry-points)
- [5. Tests](#5-tests)

**Environments and Dependencies (the classic method)**
- [6. Virtual Environments](#6-virtual-environments)
- [7. requirements.txt](#7-requirementstxt)
- [8. Runtime vs Dev Dependencies](#8-runtime-vs-dev-dependencies)

**Configuration and Secrets**
- [9. .env and Environment Variables](#9-env-and-environment-variables)
- [10. pyproject.toml](#10-pyprojecttoml)
- [11. Legacy Packaging Files](#11-legacy-packaging-files)

**Supporting and Tooling Files**
- [12. .gitignore and Generated Artifacts](#12-gitignore-and-generated-artifacts)
- [13. Tooling Configuration](#13-tooling-configuration)
- [14. README, LICENSE, and Docs](#14-readme-license-and-docs)

**A Web App, End to End**
- [15. A Web App, End to End](#15-a-web-app-end-to-end)

**The Modern Method**
- [16. The uv Method](#16-the-uv-method)

**Reference**
- [17. Common Mistakes](#17-common-mistakes)
- [18. File Reference Cheat Sheet](#18-file-reference-cheat-sheet)

---

## 1. What a Python Project Is

At its simplest, a Python project is a folder containing Python source files plus the supporting files that make the code runnable, installable, testable, and shareable. There is no single mandatory file that makes a folder "a project" — a single `script.py` is already a project of one file. What turns a loose script into a real project is the set of conventions layered on top: a place for the code, a declared list of dependencies, an isolated environment to install them into, and metadata describing the whole thing.

Two shapes of project exist, and the distinction matters because it changes which files appear:

- An **application** is meant to be run. It has an entry point you execute (`python main.py`, or a web server command), and it is usually not installed as a library. Web apps, command-line tools, and data pipelines are applications.
- A **library** (or package) is meant to be imported by other code and usually published so others can `pip install` it. It needs packaging metadata and a build configuration that an application can skip.

Most of this guide applies to both. Where a file belongs to only one shape, that is stated.

**Notes:**
- "Project," "package," and "module" are different things. A project is the whole folder; a package and a module are units of code inside it, defined in the next section.
- You can start with a flat script and grow into a structured project incrementally. None of the structure here has to exist on day one.

---

## 2. Modules, Packages, and Imports

Before looking at folders, it helps to know the three units Python organizes code into, because the whole directory layout exists to serve them.

- A **module** is a single `.py` file. `utils.py` is a module; `import utils` runs it once and gives you access to its names.
- A **package** is a folder of modules that Python treats as one importable unit. Historically a folder became a package by containing an `__init__.py` file. `import mypackage.utils` reaches a module inside a package.
- The **import system** is how Python finds these. When you write `import something`, Python searches a list of locations (`sys.path`), which includes the directory of the script being run and the `site-packages` folders of the active environment. Understanding this is what makes import errors stop being mysterious: a module is "not found" almost always because the folder it lives in is not on that search path.

### Writing a module and importing it

A module is just a `.py` file with names (functions, classes, variables) defined in it. Create one called `mathutils.py`:

```python
# mathutils.py
PI = 3.14159

def add(a, b):
    return a + b

def circle_area(radius):
    return PI * radius * radius
```

Now a second file in the same folder can import and use it. There are a few import forms, and the difference is what name you end up using to reach things:

```python
# main.py

# Form 1: import the whole module, reach names through it
import mathutils
print(mathutils.add(2, 3))          # 5
print(mathutils.PI)                 # 3.14159

# Form 2: import specific names directly into this file
from mathutils import add, circle_area
print(add(2, 3))                    # 5, no "mathutils." prefix needed

# Form 3: import the module under a shorter alias
import mathutils as mu
print(mu.circle_area(10))           # 314.159

# Form 4: import a name from a module inside a package
from mypackage.mathutils import add  # when mathutils lives in a package folder
```

Run it with `python main.py`. When Python hits the first `import mathutils`, it finds `mathutils.py` (the script's own folder is on `sys.path`), executes that file once, and makes its names available. Importing the same module again later in the same run does not re-execute it; Python caches it after the first import.

The choice between forms is mostly about readability. `import mathutils` keeps the origin of every name visible (`mathutils.add`), which is clearer in large files. `from mathutils import add` is more concise when you use a name often. Both reach the same function.

### The wildcard import, and why to avoid it

There is one more form, the wildcard or star import, which behaves differently enough to treat on its own:

```python
from mathutils import *
print(add(2, 3))        # works, no prefix
print(circle_area(10))  # works
print(PI)               # works
```

It pulls **every** public name from the module into your file at once. That convenience is exactly what makes it risky:

- **It is implicit.** The explicit forms show precisely what arrived and from where (`from mathutils import add` brings in `add`, nothing else). With `*`, a reader cannot tell from the import line what names are now defined.
- **It silently clobbers.** If two modules both define `add`, doing `from a import *` then `from b import *` overwrites the first `add` with no warning. With explicit imports the collision is visible.
- **It pollutes the namespace.** Everything public lands in your file, which can shadow your own variables or Python built-ins unexpectedly.

A module can control what `*` exports by defining `__all__`, a list of names:

```python
# mathutils.py
__all__ = ["add"]        # only `add` is exported by `from mathutils import *`
```

With that in place, `from mathutils import *` brings in only `add`. Without `__all__`, the star grabs every name that does not start with an underscore.

The practical rule: avoid `from module import *` in real code. It is the one import form that is generally discouraged, because the reader can no longer trace where a name came from. It is fine in throwaway scripts or an interactive session, and a package occasionally exposes a deliberately curated public API through it via `__all__`, but ordinary modules should use explicit imports.

### `__init__.py`

A file named `__init__.py` marks its folder as a package and runs when the package is first imported. It can be completely empty (its mere presence is the signal) or it can expose a curated public interface by importing names up from submodules, so users write `from mypackage import thing` instead of `from mypackage.internal.module import thing`.

```python
# mypackage/__init__.py
from mypackage.core import run        # re-export for a clean public API
__version__ = "0.1.0"
```

Modern Python also supports "namespace packages" (folders without `__init__.py`), but for ordinary projects an explicit `__init__.py` is clearer and avoids subtle import surprises, so prefer including it.

**Notes:**
- An import failure is a path problem far more often than a typo. The question to ask is "is this module's folder on `sys.path`?", which is exactly what the src-layout discussion in Section 3 is about.
- `__init__.py` can hold real code, but keep it light. Heavy work on import makes every consumer pay for it.
- A package is a folder; a module is a file. Keeping that vocabulary straight makes error messages readable.

---

## 3. Flat Layout vs src Layout

A project's top level can organize its source code in one of two conventional ways, and the choice is one of the first real decisions you make.

### Flat layout

The importable package sits directly at the project root:

```text
myproject/
├── myproject/          # the package (note: same name as the project)
│   ├── __init__.py
│   └── core.py
├── tests/
├── requirements.txt
└── README.md
```

This is simple and common in tutorials. Its weakness shows up at packaging time: because the package folder is right next to where you run commands, your tests and scripts can accidentally import the local source folder instead of the installed version, which can hide bugs that only appear once the project is installed elsewhere.

### src layout

The importable package is moved one level down into a `src/` directory:

```text
myproject/
├── src/
│   └── myproject/
│       ├── __init__.py
│       └── core.py
├── tests/
├── pyproject.toml
└── README.md
```

The `src/` folder is not itself a package and has no `__init__.py`. Its only job is to push the real package off the project root. This forces you to install the project (even in editable mode) before importing it, which means your tests run against the same installed package a user would get. That extra honesty is why the `src` layout is the recommendation for anything you intend to package or distribute.

**Notes:**
- For a small application you run in place, flat layout is fine. For a library you will package, prefer `src`.
- The `src` folder never gets an `__init__.py`. It is a container, not a package.
- The trade-off is one extra step: with `src` you typically run `pip install -e .` once so the package is importable. That editable install is covered in Section 10.

---

## 4. Source Code and Entry Points

Inside the package live your modules, and somewhere there is an **entry point**: the place execution begins.

### `main.py` and the `__main__` guard

For an application, a conventional entry file is `main.py` (or `app.py` for web apps, covered in Section 15). The recognizable pattern at the bottom is the main guard:

```python
def main():
    print("Hello from myproject!")


if __name__ == "__main__":
    main()
```

`if __name__ == "__main__":` means "only run this when the file is executed directly, not when it is imported." When you run `python main.py`, Python sets that module's `__name__` to the string `"__main__"`, so the block fires. When another file does `import main`, the block is skipped. This is what lets a file be both runnable and importable without side effects.

### `__main__.py`

A package folder can contain a `__main__.py`, which makes the whole package runnable with `python -m mypackage`. This is the mechanism behind commands like `python -m venv` and `python -m pytest`: those tools are packages with a `__main__.py`.

```text
mypackage/
├── __init__.py
├── __main__.py     # runs on: python -m mypackage
└── core.py
```

### Modules and sub-packages

Everything else is ordinary modules (`core.py`, `utils.py`, `models.py`) and, as the project grows, sub-packages (folders with their own `__init__.py`). Group by responsibility, not by type, so related code sits together.

**Notes:**
- The main guard is not optional boilerplate. Without it, importing your entry file would execute it, which breaks tests and tooling.
- `python -m package` runs `__main__.py`; `python file.py` runs that file directly. The two resolve imports slightly differently, which occasionally explains why one works and the other does not.
- Keep the entry point thin. It should wire things together and call into real modules, not contain the logic itself.

---

## 5. Tests

Tests live in a top-level `tests/` directory, kept separate from the source so they are not shipped with the package and so the separation of "code" from "checks on the code" is visible.

```text
myproject/
├── src/myproject/
│   └── core.py
└── tests/
    ├── __init__.py          # optional, depends on layout
    ├── test_core.py
    └── conftest.py          # shared fixtures (pytest)
```

The dominant test runner is **pytest**. It discovers any file named `test_*.py` (or `*_test.py`) and runs any function named `test_*` inside it. A test is just a function with an `assert`:

```python
# tests/test_core.py
from myproject.core import add

def test_add():
    assert add(2, 3) == 5
```

`conftest.py` is a special pytest file holding shared setup ("fixtures") and configuration that applies to the tests beside it, discovered automatically without being imported.

**Notes:**
- Name test files `test_*.py` or pytest will not discover them by default.
- With the `src` layout, tests import the installed package (`from myproject.core import add`), which is the point: they exercise what a user would actually get.
- `conftest.py` is never imported by hand. pytest finds it. Putting shared fixtures there keeps individual test files clean.

---

## 6. Virtual Environments

A **virtual environment** is an isolated, per-project copy of the Python environment, so each project gets its own set of installed packages instead of dumping everything into one shared system Python. Without isolation, two projects that need different versions of the same library cannot coexist. With it, each project's dependencies are sealed off.

The standard-library way to create one is the `venv` module:

```bash
python -m venv .venv
```

This creates a `.venv/` folder containing a launcher that points back at your base Python, plus an empty `site-packages` where this project's packages will go. The folder name `.venv` is a near-universal convention.

### Activation

Before installing or running, you **activate** the environment, which puts its `bin`/`Scripts` directory first on your shell's PATH for that session, so `python` and `pip` resolve to the environment's copies:

```bash
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\Activate.ps1       # Windows PowerShell
.venv\Scripts\activate.bat       # Windows cmd
```

Your prompt gains a `(.venv)` prefix while active. When finished, `deactivate` returns the shell to normal. Activation is per-session: open a new terminal and you activate again.

### Do you activate every time?

Activation lasts only for the current shell session, so the answer depends on where you are running from:

- **A plain terminal (PowerShell, Command Prompt, or bash).** Yes, you activate once per session. Open a new terminal window and you re-activate, because the previous activation applied only to the window you closed. Use the form for your shell:

```bash
source .venv/bin/activate        # macOS / Linux (bash, zsh)
.venv\Scripts\Activate.ps1       # Windows PowerShell
.venv\Scripts\activate.bat       # Windows Command Prompt (cmd)
```

- **VS Code.** You do not activate manually. Once you select the project's interpreter (command palette, "Python: Select Interpreter", and choose the `.venv` one), two things happen automatically: the Run button uses that environment directly, and every new integrated terminal you open **auto-activates** the `.venv` (you will see the `(.venv)` prefix appear on its own). You select the interpreter once per project, not once per session.

- **With uv, you can skip activation entirely.** `uv run <command>` (for example `uv run python main.py`) executes inside the project's environment without activating anything, in any shell. This is why uv-based workflows rarely activate by hand at all.

So "activate every time" is really "activate once per terminal session" for manual shells, "select the interpreter once" for VS Code, and "never, just use `uv run`" with uv.

### What lives inside

```text
.venv/
├── bin/ (or Scripts/ on Windows)   # python, pip, activate scripts
├── lib/.../site-packages/          # installed packages
└── pyvenv.cfg                      # records which base Python this points to
```

The environment is disposable. It is never committed to version control (it is large and machine-specific) and it is rebuilt from your dependency list whenever needed.

**Notes:**
- Activation only affects the current shell session. A new terminal starts deactivated.
- Never commit `.venv/`. It is rebuildable and belongs in `.gitignore` (Section 12).
- On Windows PowerShell, activation can fail with a script-execution-policy error. Running `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` once fixes it permanently.
- Some projects name the folder `venv` instead of `.venv`. Both are fine; `.venv` hides it from directory listings and is the more common modern choice.

---

## 7. requirements.txt

With an environment active, you install dependencies and record them. The classic record is **`requirements.txt`**, a flat text file listing one package per line, optionally with version constraints:

```text
flask>=3.0
requests==2.32.3
python-dotenv
```

You install from it with:

```bash
pip install -r requirements.txt
```

### Producing it

The crude-but-common way to generate the file is to install what you need and then freeze the environment:

```bash
pip install flask requests python-dotenv
pip freeze > requirements.txt
```

`pip freeze` lists every installed package at its exact version, which makes the result reproducible but blunt: it records direct and transitive dependencies together, with no indication of which ones you actually asked for. A cleaner discipline (the pip-tools pattern) keeps a hand-written `requirements.in` of just your direct dependencies and compiles it into a fully pinned `requirements.txt`.

### Version specifiers

| Specifier | Meaning |
| :--- | :--- |
| `flask` | any version (latest available) |
| `flask>=3.0` | this version or newer |
| `flask==3.0.3` | exactly this version (a hard pin) |
| `flask~=3.0.3` | compatible release: `>=3.0.3` and `<3.1.0` |
| `flask>=3.0,<4.0` | an explicit range |

### Updating dependencies

Changing a dependency's version in the classic method is a manual, two-part job: change what is installed, then re-record it. There are two common paths.

Edit the file, then reinstall:

```bash
# 1. change the pin in requirements.txt by hand, e.g. flask==3.0.3 -> flask==3.1.0
# 2. reinstall so the environment matches the file
pip install -r requirements.txt --upgrade
```

Or upgrade in the environment, then re-record:

```bash
pip install -U flask            # upgrade the package in the environment
pip freeze > requirements.txt   # re-capture exact versions back into the file
```

The honest caveat: plain `pip` upgrades only the package you name, not the rest of the tree, so that package's own dependencies can stay on older versions unless you upgrade them too, and `pip freeze` then records whatever happens to be installed. This is exactly the messiness that the optional `requirements.in` plus `pip-compile` discipline (and uv) exist to remove: with those, you bump the loose input and recompile a fully consistent pinned file in one step.

**Notes:**
- `pip install -r requirements.txt` reads the file; `pip freeze > requirements.txt` writes it. Do not confuse the two directions.
- A `requirements.txt` from `pip freeze` mixes your real dependencies with everything they pulled in. For clarity, maintain a short `requirements.in` of direct deps and compile from it.
- Unpinned files (`flask` with no version) install whatever is latest at install time, so two people can get different versions. Pin for reproducibility.
- `requirements.txt` is the dependency mechanism of the classic workflow. The modern alternative (`pyproject.toml` plus a lockfile) is covered in Section 16.

---

## 8. Runtime vs Dev Dependencies

Not every package your project uses at development time should be installed for an end user. A test runner, a linter, and a formatter are needed by contributors but useless to someone who just wants to run your application. The classic convention splits them across two files:

```text
requirements.txt        # runtime: what the app needs to run
requirements-dev.txt    # development: tooling for contributors
```

A common pattern is for the dev file to include the runtime file, so installing dev gives you everything:

```text
# requirements-dev.txt
-r requirements.txt
pytest>=8.0
ruff>=0.6
mypy>=1.10
```

A contributor runs `pip install -r requirements-dev.txt` and gets both sets; a deployment installs only `requirements.txt`.

**Notes:**
- The `-r requirements.txt` line inside the dev file pulls in the runtime deps, so you maintain the runtime list in one place.
- Keep test and lint tools out of `requirements.txt`. Shipping `pytest` to production is harmless but wasteful and signals a blurred boundary.
- The modern equivalent of this split is dependency groups in `pyproject.toml` (Section 10), which formalize the same idea.

---

## 9. .env and Environment Variables

Applications need configuration that should not be hard-coded: database URLs, API keys, secret keys, feature flags. Two rules govern these: configuration that changes between machines should live outside the code, and secrets must never be committed to version control. The standard solution is **environment variables**, and the conventional place to define them for local development is a **`.env`** file.

### What a .env file is

A `.env` file is a plain text file of `KEY=value` lines, holding configuration and secrets, never packages:

```text
SECRET_KEY=dev-only-not-a-real-secret
DATABASE_URL=postgresql://localhost:5432/myapp
DEBUG=true
OPENAI_API_KEY=sk-...
```

It is loaded into the process environment at startup. Many frameworks read it automatically; otherwise the `python-dotenv` library does it explicitly:

```python
from dotenv import load_dotenv
import os

load_dotenv()                       # reads .env into the environment
secret = os.environ["SECRET_KEY"]   # your code reads from the environment, not the file
```

The key idea is the indirection: your code reads from the **environment** (`os.environ`), and the `.env` file is just a convenient way to populate that environment locally. In production, the same variables are set by the host or orchestration platform instead of a file, so the code does not change.

### .env must be gitignored, and .env.example is committed

Because `.env` holds secrets, it is always listed in `.gitignore` and never committed. To still document which variables a project needs, you commit a sanitized template, conventionally `.env.example`, with the keys present but the values blanked or faked:

```text
# .env.example  (committed)
SECRET_KEY=
DATABASE_URL=
DEBUG=false
OPENAI_API_KEY=
```

A new contributor copies `.env.example` to `.env` and fills in real values.

### config.py: centralizing configuration

Reading `os.environ[...]` directly all over the codebase scatters configuration and makes it hard to see what a project actually needs. A common pattern is a dedicated `config.py` module that loads the environment once and exposes the values as a single, importable object:

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    SECRET_KEY = os.environ["SECRET_KEY"]                       # required: error if missing
    DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///dev.db")  # optional: default provided
    DEBUG = os.environ.get("DEBUG", "false").lower() == "true"  # parse string to bool
```

The rest of the app then imports the config instead of touching the environment directly:

```python
from config import Config
print(Config.DATABASE_URL)
```

This gives you one place that defines every setting, sensible defaults for optional values, and a single point where a missing required variable fails loudly and early. Note the two access styles: `os.environ["KEY"]` raises an error if the variable is absent (use it for things the app cannot run without), while `os.environ.get("KEY", default)` returns a fallback (use it for optional settings).

Web frameworks have first-class support for this pattern. Flask, for example, loads a config object with `app.config.from_object(Config)`, and larger projects often define several classes (`DevelopmentConfig`, `ProductionConfig`) selected by an environment variable. The `.env` file still supplies the raw values; `config.py` is the structured layer your code reads from.

**Notes:**
- `.env` holds configuration and secrets, never packages. It has nothing to do with dependency management; it is unrelated to `requirements.txt`.
- Never commit `.env`. Committing a secret means rotating it, since it lives forever in Git history. Commit `.env.example` instead.
- Code should read from `os.environ` (or a `config.py` that wraps it), not from the file directly. The file is just a loader for the environment.
- `config.py` is committed; it should contain no secret values, only the code that reads them from the environment. The secrets stay in `.env`.
- A leaked API key in a public repository is one of the most common and costly beginner mistakes. The `.gitignore` entry for `.env` is what prevents it.

---

## 10. pyproject.toml

`pyproject.toml` is the modern, standardized configuration file for Python projects, defined by Python packaging standards (PEP 518, PEP 621, and others). It serves three roles in one file: project metadata, build configuration, and tool settings. Even projects that still use `requirements.txt` for dependencies commonly use `pyproject.toml` for the latter two.

### Project metadata: `[project]`

```toml
[project]
name = "myproject"
version = "0.1.0"
description = "A short summary of the project"
readme = "README.md"
requires-python = ">=3.11"
dependencies = [
    "flask>=3.0",
    "requests>=2.32",
]
```

The `[project]` table names and describes the project, sets the minimum Python version, and can declare dependencies directly. When a project lists dependencies here, this table is the source of truth and tools install from it, an alternative to `requirements.txt`.

### Extras and dependency groups

Two mechanisms separate optional or development dependencies from the core list:

```toml
[project.optional-dependencies]
postgres = ["psycopg2-binary>=2.9"]    # extras: features a user can opt into

[dependency-groups]
dev = ["pytest>=8.0", "ruff>=0.6"]     # dev tooling: never shipped to users
```

`[project.optional-dependencies]` are **extras**, installed on demand with `pip install "myproject[postgres]"`, used for features you expose to users. `[dependency-groups]` (PEP 735) are internal development dependencies that are never published, the modern equivalent of `requirements-dev.txt`.

### Build configuration: `[build-system]`

For a library you intend to package and publish, a build-system table tells packaging tools how to build it:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

A plain application that is never installed as a package can omit this. Its presence is what allows `pip install .` (or an editable `pip install -e .`) to build and install the project.

### Tool configuration: `[tool.*]`

Most tools read their settings from a `[tool.<name>]` table here, keeping all configuration in one file instead of scattering `.flake8`, `mypy.ini`, and `pytest.ini` around the root:

```toml
[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

### Editable installs

For `src`-layout projects, you make your package importable during development with an editable install, which links the installed package back to your source so edits take effect immediately:

```bash
pip install -e .          # editable install of the current project
```

**Notes:**
- `pyproject.toml` is not uv-specific. It is the Python-wide standard, used by pip, build tools, and most linters and test runners.
- A project can declare dependencies in `[project]` (modern) or in `requirements.txt` (classic), but maintaining both as sources of truth invites drift. Pick one.
- `[project.optional-dependencies]` are user-facing extras; `[dependency-groups]` are internal dev tooling. They look similar but serve different audiences.
- The `[build-system]` table is only needed for installable packages. Pure applications can skip it.

---

## 11. Legacy Packaging Files

Before `pyproject.toml` unified everything, packaging metadata and build instructions lived in separate files. You will still encounter these in older projects, so they are worth recognizing even though new projects should not start with them.

| File | What it was | Status today |
| :--- | :--- | :--- |
| `setup.py` | An executable script defining package metadata and build steps via a `setup()` call | Superseded by `pyproject.toml`; still works, but discouraged for new projects |
| `setup.cfg` | A static, declarative companion to `setup.py` for metadata | Largely replaced by `[project]` in `pyproject.toml` |
| `MANIFEST.in` | Lists non-code files (data, templates) to include when building a distribution | Still used occasionally; many build backends now handle this another way |

A project might contain a minimal `setup.py` purely for backward compatibility (for example to allow an editable install on very old tooling), but the metadata it once held now belongs in `pyproject.toml`.

**Notes:**
- If a project has both `setup.py` and `pyproject.toml`, the `pyproject.toml` is usually authoritative and the `setup.py` is a thin shim or legacy leftover.
- You do not need to write these for a new project. Recognize them in old code; do not reach for them.

---

## 12. .gitignore and Generated Artifacts

A Python project generates a lot of files that should never be committed: caches, build outputs, the virtual environment, and secrets. `.gitignore` lists patterns Git should ignore, keeping the repository to source and configuration only.

A representative Python `.gitignore`:

```gitignore
# Byte-compiled / cache
__pycache__/
*.py[cod]
.pytest_cache/
.mypy_cache/
.ruff_cache/

# Virtual environments
.venv/
venv/

# Build / distribution artifacts
build/
dist/
*.egg-info/

# Secrets and local config
.env

# Coverage
.coverage
htmlcov/
```

What each group is and why it is ignored:

- **`__pycache__/` and `*.pyc`** — Python compiles modules to bytecode on import and caches it here. Regenerated automatically, so never committed.
- **`.venv/`** — the virtual environment, large and machine-specific, rebuilt from your dependency list.
- **`build/`, `dist/`, `*.egg-info/`** — outputs produced when packaging a project for distribution.
- **`.env`** — secrets and local config, covered in Section 9.
- **Tool caches and coverage** — `.pytest_cache/`, `.mypy_cache/`, `.ruff_cache/`, `.coverage`, all regenerated on demand.

**Notes:**
- The guiding principle: commit what a human writes (source, config, docs); ignore what a tool generates (caches, builds, environments) and what is secret (`.env`).
- If a generated folder is already tracked because it was committed before being ignored, adding it to `.gitignore` does not remove it. You have to `git rm -r --cached` it once.
- Ignoring `.env` is the single most important line for safety. It prevents leaking secrets.

---

## 13. Tooling Configuration

Mature projects use automated tools to keep code consistent and correct. Each has a configuration home, increasingly inside `pyproject.toml` rather than its own file.

| Tool | Purpose | Config location |
| :--- | :--- | :--- |
| Ruff | Linting and formatting (fast, widely adopted) | `[tool.ruff]` in `pyproject.toml` |
| Black | Opinionated code formatter | `[tool.black]` in `pyproject.toml` |
| mypy | Static type checking | `[tool.mypy]` in `pyproject.toml`, or `mypy.ini` |
| pytest | Test runner | `[tool.pytest.ini_options]`, or `pytest.ini` |
| isort | Import sorting (often subsumed by Ruff) | `[tool.isort]` in `pyproject.toml` |

### pre-commit

A widely used safety net is **pre-commit**, which runs checks automatically every time you commit. It is configured in its own file, `.pre-commit-config.yaml`, listing the hooks to run:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
      - id: ruff-format
```

After `pre-commit install`, these run on staged files at each commit, so formatting and lint problems are caught before they enter history.

### Continuous integration

CI configuration lives under `.github/workflows/` (for GitHub Actions) as YAML files that run your tests and checks on every push. This is a deep topic of its own; here it is enough to know the folder exists and holds the automation that runs your test and lint commands in the cloud.

**Notes:**
- Consolidating tool config into `pyproject.toml` keeps the project root tidy. Reach for separate files (`mypy.ini`, `pytest.ini`) only when a tool does not yet support `pyproject.toml` or you have a specific reason.
- `pre-commit` config is committed; the hooks it installs into `.git/` are not (they are local machinery).
- CI YAML under `.github/workflows/` runs the same commands you run locally. If it passes locally with the same versions, it should pass in CI.

---

## 14. README, LICENSE, and Docs

These files describe and govern the project rather than run it, but they are part of a complete project.

- **`README.md`** — the front page. It explains what the project is, how to install and run it, and how to contribute. It is the first thing a visitor reads on a code host, rendered automatically. A good README covers purpose, setup steps, and a usage example.
- **`LICENSE`** — the legal terms under which others may use the code. Without one, default copyright applies and others have no clear right to use it. Common open-source choices include MIT, Apache-2.0, and GPL.
- **`CHANGELOG.md`** — a human-readable history of notable changes per version. Optional but valuable for anything others depend on.
- **`docs/`** — a folder for fuller documentation when the README outgrows itself, often built with a tool like Sphinx or MkDocs into a documentation site.

**Notes:**
- The README is the highest-leverage file for a public project. Many readers decide whether to use a project from the README alone.
- Choosing no license is a choice: it means "all rights reserved" and discourages reuse. Pick one deliberately.
- Keep the README's setup instructions in sync with reality. Stale install steps are a common source of first-run failures.

---

## 15. A Web App, End to End

To see the web-specific files in context, here is the layout of a small Flask-style application, `myapp`. A FastAPI app differs mainly in the framework calls; the file roles are the same.

```text
myapp/
├── app.py                  # the application entry point
├── templates/              # HTML templates (server-rendered pages)
│   └── index.html
├── static/                 # CSS, JavaScript, images served as-is
│   └── style.css
├── .env                    # secrets and config (gitignored)
├── .env.example            # committed template of required variables
├── requirements.txt        # flask, python-dotenv, ...
├── .gitignore
└── README.md
```

### `app.py`

`app.py` is the conventional entry point for a web app, the equivalent of `main.py` for a script. It creates the application object and defines routes:

```python
import os
from flask import Flask, render_template
from dotenv import load_dotenv

load_dotenv()

app = Flask(__name__)
app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]

@app.route("/")
def index():
    return render_template("index.html")

if __name__ == "__main__":
    app.run(debug=True)
```

### How the app is served: localhost, ports, and the cloud

Running `python app.py` does not just execute and exit like a script. The `app.run()` call starts a long-running **web server** that waits for incoming requests until you stop it. Understanding where that server can be reached is the missing piece between "I ran it" and "I can see it in a browser."

When you run it locally, the development server binds to **`localhost`** on a **port**:

```text
 * Running on http://127.0.0.1:5000
```

- **`localhost`** (and its numeric form `127.0.0.1`) means "this same machine." A server bound to `localhost` is reachable only from your own computer, not from anywhere else on the network. It is the private, development-only address.
- A **port** is the numbered channel the server listens on, the `:5000` part. One machine can run many servers at once, each on a different port (Flask defaults to `5000`, FastAPI/Uvicorn to `8000`). The full address to open in a browser is the protocol, host, and port together: `http://localhost:5000`.

**How a request reaches your code.** When you open `http://localhost:5000/` in a browser, the browser sends a request for the path `/` to the server. The server matches that path against your routes and calls the matching function. This is exactly what `@app.route("/")` declares: "when a request arrives for `/`, run `index()` and return what it produces." Add `@app.route("/about")` and a request to `http://localhost:5000/about` calls that function instead. The route is the mapping from a URL path to a Python function.

**Local versus the cloud.** The difference between running locally and running in the cloud is mostly *which address the server binds to*:

- **Locally**, it binds to `127.0.0.1` (localhost), so only you can reach it. Perfect for development, invisible to the internet.
- **In the cloud**, the server must accept requests from anyone, so it binds to `0.0.0.0` (meaning "all network interfaces, not just this machine") on a port the hosting platform assigns, often provided through an environment variable like `PORT`. The platform then maps that to a public domain (`https://myapp.example.com`) so the world can reach it.

This is also where the deployment entry points below come in: in the cloud you do not run the development server at all. A production server (Gunicorn or Uvicorn) binds the public host and port and serves your app, which is why production needs the `wsgi.py`/`asgi.py` entry point rather than `app.run()`.

```python
# binding for the cloud: read the platform's port, listen on all interfaces
import os
port = int(os.environ.get("PORT", 8000))
app.run(host="0.0.0.0", port=port)   # development server shown for illustration; use Gunicorn/Uvicorn in production
```

### `templates/` and `static/`

- **`templates/`** holds server-rendered HTML files. Frameworks look here by convention when you call something like `render_template("index.html")`. Templates usually contain placeholders the server fills in per request.
- **`static/`** holds files served unchanged: stylesheets, client-side JavaScript, images, fonts. The framework exposes them at a URL like `/static/style.css`.

### `.env` in a web app

The web app is where `.env` earns its keep. A real service needs a `SECRET_KEY` for signing sessions, a `DATABASE_URL`, and often third-party API keys, exactly the secrets that must stay out of source control. The app loads them at startup (as shown above) and reads from `os.environ`, while `.env` itself is gitignored and `.env.example` documents the required keys.

### Deployment entry points

In production you do not use the framework's built-in development server. Instead a production server (Gunicorn, Uvicorn) imports your app object, conventionally exposed through a small `wsgi.py` (for traditional/Flask apps) or `asgi.py` (for async/FastAPI apps):

```python
# wsgi.py
from app import app    # the production server imports this object
```

A `Procfile` (one line naming the start command) appears in projects deployed to platforms that read it.

**Notes:**
- `templates/` and `static/` are framework conventions. The framework finds them by name, so the folder names matter.
- The development server (`app.run(debug=True)` or `flask run`) is for local use only. Never run it in production; use a real server importing the app via `wsgi.py`/`asgi.py`.
- The web app makes the earlier files concrete: `app.py` is the entry point, `.env` holds the secrets, `requirements.txt` lists `flask` and `python-dotenv`, and `.gitignore` keeps `.env` and `__pycache__/` out.

---

## 16. The uv Method

Everything so far is the classic, manual workflow: create an environment with `venv`, install with `pip`, and track dependencies in `requirements.txt`. The modern tool **uv** compresses that entire workflow into one fast program, and it changes which files are authoritative. This section covers the uv method the same way the rest of the guide covers the classic one: file by file, what each is and how it differs from the classic equivalent. The full command lifecycle (every flag, the cache, multiple Python versions) lives in the companion guide, [The uv Project Lifecycle](../tooling/uv-project-lifecycle.md); here the focus is the anatomy.

### What uv replaces

The whole classic workflow maps onto a handful of uv commands:

| Classic step | uv equivalent |
| :--- | :--- |
| `python -m venv .venv` | `uv venv` (or automatic on first command) |
| activate, then `pip install flask` | `uv add flask` |
| `pip install -r requirements.txt` | `uv sync` |
| `pip freeze > requirements.txt` | maintained automatically in `uv.lock` |
| `requirements.txt` as the dependency record | `pyproject.toml` + `uv.lock` |
| `requirements-dev.txt` for tooling | `[dependency-groups]` via `uv add --dev` |
| activate before running | `uv run <command>` |

The key shift is that the **dependency record moves from `requirements.txt` to `pyproject.toml` plus `uv.lock`**, and the environment becomes something you rarely touch by hand.

### The uv files, one at a time

A uv project contains the same source code, tests, `.env`, and supporting files as any other Python project. What changes is the environment-and-dependency layer, described by the files below.

**pyproject.toml — the single source of truth.**
uv makes `pyproject.toml` (introduced in Section 10) the authoritative dependency record. Your runtime dependencies live in `[project].dependencies`, your tooling lives in `[dependency-groups]`, and `uv add` edits these for you rather than you maintaining a `requirements.txt` by hand:

```toml
[project]
name = "myapp"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "flask>=3.0",
    "python-dotenv>=1.0",
]

[dependency-groups]
dev = [
    "pytest>=8.0",
    "ruff>=0.6",
]
```

This is the same file the classic method uses for tool config and packaging metadata. uv simply also treats it as the dependency list, replacing `requirements.txt` and `requirements-dev.txt` at once.

**uv.lock — the automatic, exact lockfile.**
`uv.lock` is what the classic workflow never had cleanly: a precise, generated record of the exact version of every direct and transitive dependency, with hashes, resolved to work across platforms. It is the rigorous version of `pip freeze > requirements.txt`, except uv maintains it automatically on every `uv add`, `uv lock`, and `uv sync`. You never hand-edit it, and you always commit it. The division of labor is the same idea as the classic pip-tools split (`requirements.in` versus a compiled `requirements.txt`), but automatic: `pyproject.toml` states what is acceptable, `uv.lock` records what was chosen.

**.venv — still there, rarely touched.**
The virtual environment is unchanged in nature from Section 6: same `.venv` folder, same isolation, still disposable, still gitignored. What changes is that you rarely create or activate it by hand. `uv run` executes commands inside it without activation, and `uv sync` rebuilds it from `pyproject.toml` and `uv.lock`. The manual `python -m venv` plus `activate` dance from the classic method is gone.

**requirements.txt — now an optional export.**
In a uv project, `requirements.txt` is no longer the source of truth. It becomes an artifact you generate only when something downstream (a Docker image, a legacy CI step, a serverless platform) specifically requires that format:

```bash
uv export --format requirements.txt --output-file requirements.txt
```

You regenerate it from the lockfile whenever needed and do not hand-edit it. `pyproject.toml` and `uv.lock` stay authoritative.

### Adopting uv in an existing requirements.txt project

A common real situation is inheriting an older project that uses `requirements.txt` and wanting to move it to uv. The migration is short, because uv can read the old file directly. From inside the existing project folder:

```bash
# 1. Create a pyproject.toml without overwriting your existing files.
#    --bare produces a minimal pyproject.toml and nothing else.
uv init --bare

# 2. Import the existing dependencies into pyproject.toml.
#    uv reads the file and adds each entry under [project].dependencies.
uv add -r requirements.txt

# 3. (If you keep dev tooling separately) import those into a dev group.
uv add --dev -r requirements-dev.txt

# 4. uv resolves everything and writes uv.lock automatically.
#    Build the environment to confirm it all installs.
uv sync
```

After this, `pyproject.toml` and `uv.lock` are the authoritative record. You can then retire the old `requirements.txt` from version control, or keep regenerating it with `uv export` if a deployment target still expects it. Verify the app runs (`uv run python app.py` or your usual entry point), commit `pyproject.toml` and `uv.lock`, and the project is now uv-managed.

| Migration step | Command | Result |
| :--- | :--- | :--- |
| Create the recipe file | `uv init --bare` | minimal `pyproject.toml`, existing files untouched |
| Import dependencies | `uv add -r requirements.txt` | deps written into `[project].dependencies` |
| Import dev tools | `uv add --dev -r requirements-dev.txt` | tools written into `[dependency-groups]` |
| Lock and install | `uv sync` | `uv.lock` generated, `.venv` built |

### Why learn the classic method first

uv's conveniences make sense precisely because you now know what they replace. `uv add` is "install and record" in one step because you have done those as two. `uv sync` is "recreate the environment from the record" because you have done that with `pip install -r`. `uv.lock` is a precise, automatic `requirements.txt`. The mental model is identical; uv just removes the manual steps.

**Notes:**
- uv does not change what a Python project *is*. The structure, modules, packages, tests, `.env`, and supporting files are all the same. uv only modernizes the environment-and-dependency layer.
- `pyproject.toml` is shared ground: it predates uv and is a Python-wide standard. uv adds `uv.lock` and a faster workflow on top of it.
- Migrating an existing project is mostly `uv init --bare` then `uv add -r requirements.txt`. The old file is read, not discarded, so nothing is lost in the move.
- For a new project today, the uv workflow is the recommended default. The classic method remains essential to understand because you will meet it constantly in existing code.
- For the complete uv command reference and the full lifecycle, see the companion guide, [The uv Project Lifecycle](../tooling/uv-project-lifecycle.md).

---

## 17. Common Mistakes

**Committing `.env`.** Secrets in Git history live forever and must be rotated once leaked. Always gitignore `.env` and commit `.env.example` instead.

**Committing the virtual environment.** `.venv/` is large, machine-specific, and rebuildable. It belongs in `.gitignore`, not the repository.

**Reading secrets from the file instead of the environment.** Code should read `os.environ`, with `.env` merely loaded into that environment at startup. This keeps the same code working in production, where variables come from the host, not a file.

**Putting dev tools in `requirements.txt`.** Test runners and linters belong in `requirements-dev.txt` (classic) or `[dependency-groups]` (modern), not in the runtime list shipped to users.

**Unpinned dependencies in a project meant to be reproducible.** A bare `flask` installs whatever is latest, so two installs can differ. Pin versions, or use a lockfile.

**Forgetting the `__main__` guard.** Without `if __name__ == "__main__":`, importing your entry file executes it, breaking tests and tooling.

**Confusing `pip install -r` with `pip freeze >`.** One reads the requirements file to install; the other writes the file from the environment. They point in opposite directions.

**Expecting `src`-layout code to import without installing.** With a `src` layout you must install the project (often `pip install -e .`) before the package is importable. That is the layout working as intended.

**Maintaining dependencies in two places.** Declaring deps in both `requirements.txt` and `pyproject.toml` invites drift. Choose one source of truth.

**Running the development server in production.** A framework's built-in server is for local development. Production uses a real server (Gunicorn/Uvicorn) importing the app via `wsgi.py`/`asgi.py`.

---

## 18. File Reference Cheat Sheet

| File / folder | What it is | Commit? |
| :--- | :--- | :--- |
| `src/` or package folder | Your source code | Yes |
| `__init__.py` | Marks a folder as a package | Yes |
| `__main__.py` | Makes a package runnable with `python -m` | Yes |
| `main.py` / `app.py` | Application entry point | Yes |
| `tests/` | Test suite (pytest) | Yes |
| `conftest.py` | Shared pytest fixtures | Yes |
| `.venv/` | Virtual environment | No (gitignored) |
| `requirements.txt` | Runtime dependencies (classic) | Yes |
| `requirements-dev.txt` | Development dependencies (classic) | Yes |
| `requirements.in` | Hand-written direct deps for pip-tools | Yes |
| `.env` | Secrets and local config | No (gitignored) |
| `.env.example` | Template of required variables | Yes |
| `pyproject.toml` | Metadata, build, tool config, modern deps | Yes |
| `uv.lock` | Exact pinned versions (uv) | Yes |
| `setup.py` / `setup.cfg` | Legacy packaging metadata | Yes (if present) |
| `.gitignore` | Patterns Git should ignore | Yes |
| `.pre-commit-config.yaml` | Pre-commit hook configuration | Yes |
| `.github/workflows/` | CI pipelines (GitHub Actions) | Yes |
| `templates/` | Server-rendered HTML (web app) | Yes |
| `static/` | CSS, JS, images (web app) | Yes |
| `wsgi.py` / `asgi.py` | Production server entry point | Yes |
| `README.md` | Project front page | Yes |
| `LICENSE` | Usage terms | Yes |
| `__pycache__/`, `*.pyc` | Bytecode cache | No (gitignored) |
| `build/`, `dist/`, `*.egg-info/` | Packaging output | No (gitignored) |

### The two workflows side by side

| Task | Classic (pip + venv) | Modern (uv) |
| :--- | :--- | :--- |
| Create environment | `python -m venv .venv` | `uv venv` |
| Activate | `source .venv/bin/activate` | not needed (`uv run`) |
| Install a package | `pip install flask` | `uv add flask` |
| Record dependencies | `pip freeze > requirements.txt` | automatic in `uv.lock` |
| Reinstall elsewhere | `pip install -r requirements.txt` | `uv sync` |
| Run code | `python main.py` | `uv run python main.py` |

---

*Further reading:*
- *[Python Packaging User Guide](https://packaging.python.org/) — the authoritative reference*
- *[Writing pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) — the modern config file*
- *[venv documentation](https://docs.python.org/3/library/venv.html) — standard-library virtual environments*
- *[pip user guide](https://pip.pypa.io/en/stable/user_guide/) — installing and requirements files*
- *[python-dotenv](https://github.com/theskumar/python-dotenv) — loading .env files*
- *[The uv Project Lifecycle](../tooling/uv-project-lifecycle.md) — companion guide for the modern uv workflow*
