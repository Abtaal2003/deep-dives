# Python, The Complete Language Guide

A thorough, interview-ready tour of the Python language itself: how it runs, every built-in data type and the methods that come with it, control flow, functions, object-oriented programming, errors, modules, and the deeper mechanics (identity, mutability, the GIL) that interviewers probe. Every concept is paired with a runnable example and its real output, so nothing is left abstract. The focus is the language, not algorithms: data structures and algorithms get their own companion guide.

The examples target **Python 3.14** (the current stable series, 3.14.6 as of mid-2026), but everything here works on 3.10+ unless a version is called out. Where a feature is newer, that is flagged inline.

**Sources:**
- [The Python Language Reference](https://docs.python.org/3/reference/) — the authoritative specification
- [The Python Standard Library](https://docs.python.org/3/library/) — built-in types, functions, and modules
- [The Python Tutorial](https://docs.python.org/3/tutorial/) — the official introduction
- [Built-in Types](https://docs.python.org/3/library/stdtypes.html) — the per-type method tables
- [PEP 8](https://peps.python.org/pep-0008/) — the style guide
- [What's New in Python 3.14](https://docs.python.org/3/whatsnew/3.14.html) — the latest release notes
- [W3Schools Python](https://www.w3schools.com/python/) and [GeeksforGeeks Python](https://www.geeksforgeeks.org/python-programming-language/) — worked examples and references

---

## How to read this guide

The sections move from the ground up: first how Python runs and how its objects work, then each data type in turn, then the constructs that combine them (control flow, functions, classes), then the parts that make programs robust (errors, files, modules, types), and finally a set of interview deep-dives into the mechanics behind common questions. The last three sections are pure reference: Common Mistakes, a rapid-fire interview Q&A, and a Cheat Sheet.

Two mental models recur and are worth holding from the start. First, **everything in Python is an object**, including numbers, functions, and classes themselves, so the same rules about identity and references apply everywhere. Second, **a variable is a name bound to an object, not a box holding a value**, which is the single idea behind most of the "why did my list change?" surprises later in the guide.

Each section ends with a short **Notes** list of the things that trip people up or that interviewers like to ask about.

---

## Table of Contents

**Foundations**
- [1. What Python Is and How It Runs](#1-what-python-is-and-how-it-runs)
- [2. The REPL, Scripts, and print](#2-the-repl-scripts-and-print)
- [3. pip and the Package Ecosystem](#3-pip-and-the-package-ecosystem)
- [4. Variables, Objects, and References](#4-variables-objects-and-references)

**The Built-in Data Types**
- [5. Numbers: int, float, complex, bool](#5-numbers-int-float-complex-bool)
- [6. Strings](#6-strings)
- [7. Lists](#7-lists)
- [8. Tuples](#8-tuples)
- [9. Sets and frozenset](#9-sets-and-frozenset)
- [10. Dictionaries](#10-dictionaries)
- [11. None and Truthiness](#11-none-and-truthiness)
- [12. Type Conversion and Inspection](#12-type-conversion-and-inspection)

**Operators and Control Flow**
- [13. Operators](#13-operators)
- [14. Conditionals and Pattern Matching](#14-conditionals-and-pattern-matching)
- [15. Loops](#15-loops)
- [16. Comprehensions](#16-comprehensions)

**Functions**
- [17. Defining and Calling Functions](#17-defining-and-calling-functions)
- [18. Parameters and Arguments](#18-parameters-and-arguments)
- [19. Scope: LEGB, global, nonlocal](#19-scope-legb-global-nonlocal)
- [20. Lambdas and Functional Tools](#20-lambdas-and-functional-tools)
- [21. Closures and Decorators](#21-closures-and-decorators)
- [22. Iterators and Generators](#22-iterators-and-generators)

**Object-Oriented Programming**
- [23. Classes and Instances](#23-classes-and-instances)
- [24. Methods: Instance, Class, Static](#24-methods-instance-class-static)
- [25. Inheritance and the MRO](#25-inheritance-and-the-mro)
- [26. Encapsulation and Properties](#26-encapsulation-and-properties)
- [27. Dunder Methods](#27-dunder-methods)
- [28. Dataclasses](#28-dataclasses)

**Robust Programs**
- [29. Errors and Exceptions](#29-errors-and-exceptions)
- [30. Files and Context Managers](#30-files-and-context-managers)
- [31. Modules, Packages, and the Standard Library](#31-modules-packages-and-the-standard-library)
- [32. Type Hints](#32-type-hints)

**Interview Deep-Dives**
- [33. Mutability, Identity, and Copying](#33-mutability-identity-and-copying)
- [34. How Python Executes: CPython, Bytecode, the GIL](#34-how-python-executes-cpython-bytecode-the-gil)
- [35. Memory, Iteration, and Performance Habits](#35-memory-iteration-and-performance-habits)

**Reference**
- [36. Common Mistakes](#36-common-mistakes)
- [37. Interview Quick-Fire](#37-interview-quick-fire)
- [38. Cheat Sheet](#38-cheat-sheet)

---

## 1. What Python Is and How It Runs

Python is a high-level, dynamically typed, interpreted, garbage-collected language. Each of those words is a real interview talking point, so it helps to unpack them.

**High-level** means you work with concepts like lists and dictionaries rather than memory addresses and registers. **Dynamically typed** means a variable has no fixed type; the *object* a name points to carries the type, and you can rebind the same name to a different type later. **Interpreted** means you do not compile to a standalone machine-code binary ahead of time — instead the reference implementation (CPython) compiles your source to bytecode and a virtual machine executes that bytecode. **Garbage-collected** means you never free memory by hand; Python reclaims objects automatically (primarily by reference counting, with a cycle collector as backup).

The usual implementation is **CPython**, written in C, and "Python" colloquially means CPython unless stated otherwise. Other implementations exist (PyPy with a JIT for speed, and others), but they all implement the same language.

The path from source to running program looks like this:

```text
your_code.py  ──compile──▶  bytecode (.pyc, cached in __pycache__)  ──run──▶  Python Virtual Machine
```

You do not manage any of this by hand. Running `python your_code.py` does the compile-then-execute step in one go. The `.pyc` files in `__pycache__` are just a cache so unchanged modules skip recompilation next time.

Python is strongly typed as well as dynamically typed, and conflating the two is a classic interview slip. Dynamic typing is about *when* types are checked (at runtime, on the object). Strong typing is about Python refusing nonsensical implicit conversions: `"3" + 5` raises an error rather than guessing — where a weakly typed language might return `"35"` or `8`.

```python
x = 10
print(type(x))     # <class 'int'>
x = "now a string"
print(type(x))     # <class 'str'>   the name was rebound to a new object

"3" + 5            # TypeError: can only concatenate str (not "int") to str
```

**Notes:**
- "Compiled vs interpreted" is not binary. Python compiles to bytecode, then interprets the bytecode. Saying simply "Python is interpreted" is fine in casual use, but knowing the bytecode step is what interviewers want.
- Dynamic typing is about the timing of type association; strong typing is about refusing silent, unsafe coercions. Python is both dynamically and strongly typed.
- `__pycache__` and `.pyc` files are a performance cache, not something you edit or commit. The default `.gitignore` excludes them.

---

## 2. The REPL, Scripts, and print

There are two ways to run Python, and you will use both daily.

The **REPL** (Read-Eval-Print Loop) is the interactive prompt you get by typing `python` with no file. You type an expression, it evaluates and prints the result immediately — which makes it ideal for trying things out. In the REPL, an expression's value is echoed automatically, so `2 + 2` shows `4` with no `print`.

A **script** is a `.py` file you run with `python script.py`. In a script, nothing is printed unless you call `print`, because a script evaluates expressions but only *displays* what you explicitly output.

```python
# In the REPL:
>>> 2 + 2
4                      # echoed automatically
>>> "hello".upper()
'HELLO'

# In a script (script.py), you must print to see anything:
2 + 2                  # computed, then discarded, nothing shown
print(2 + 2)           # 4
```

`print` is the everyday output function. Its useful keyword arguments are `sep` (the separator between items, default a space) and `end` (what to append, default a newline):

```python
print("a", "b", "c")              # a b c
print("a", "b", "c", sep="-")     # a-b-c
print("no newline", end=" >> ")
print("same line")                # no newline >> same line
```

The input counterpart is `input`, which always returns a string — so you convert it when you need a number:

```python
name = input("Name: ")            # always a str
age = int(input("Age: "))         # convert explicitly
```

**Comments** begin with `#` and run to the end of the line. Python has no dedicated block-comment syntax, so you stack `#` lines (editors toggle them in bulk). A `#` inside a string is just text, not a comment.

```python
x = 10            # an inline comment, ignored at runtime
# a full-line comment
# stacked lines stand in
# for a block comment
print("# this is not a comment")   # # this is not a comment
```

**Notes:**
- Comments use `#` to the end of the line; there is no block-comment syntax, so stack `#` lines. A `#` inside a string literal is ordinary text.
- The REPL echoes expression results; a script does not. This is why beginners think a script "did nothing" when it computed plenty but printed nothing.
- `input` returns a string even when the user types digits. Forgetting to convert is a frequent off-by-one-type bug.
- In the REPL, the special variable `_` holds the last result, handy for quick chaining.

---

## 3. pip and the Package Ecosystem

The standard library ships with Python, but most real work uses third-party packages. **pip** is Python's package installer: it downloads packages from **PyPI** (the Python Package Index, the public registry at pypi.org) and installs them into your environment so you can `import` them.

The mental model: the standard library is what comes in the box; PyPI is the warehouse of everything else; pip is the tool that fetches from the warehouse.

```bash
pip install requests           # install the latest compatible version
pip install "requests==2.32.3" # install an exact version
pip install requests numpy     # install several at once
pip uninstall requests         # remove a package
pip list                       # show everything installed
pip show requests              # details about one package
pip install --upgrade requests # upgrade to the newest version
```

Once installed, a package is importable like any standard-library module:

```python
import requests
resp = requests.get("https://example.com")
print(resp.status_code)        # 200
```

Two pieces of context complete the picture. **`requirements.txt`** is a plain list of packages (often pinned to exact versions) that records what a project needs, so a teammate can recreate your setup with one command:

```bash
pip freeze > requirements.txt        # write current packages to the file
pip install -r requirements.txt      # install everything listed in it
```

**Virtual environments** keep each project's packages isolated — so project A's `requests==2.28` cannot collide with project B's `requests==2.32`. You almost never install into the global interpreter; you make a per-project environment first. The classic standard-library way:

```bash
python -m venv .venv                 # create an isolated environment
source .venv/bin/activate            # activate it (Windows: .venv\Scripts\activate)
pip install requests                 # now installs only into this project
```

Modern tooling such as `uv` compresses all of this (environment creation, installs, and a precise lockfile) into a faster single tool — but it sits on the same pip and PyPI foundations described here. For the full project-and-dependency story, see the companion guides on project anatomy and the uv lifecycle.

**Notes:**
- pip installs from PyPI into an environment; it does not run your code or manage Python versions. Those are separate concerns.
- Install into a virtual environment, not the global interpreter. A polluted global Python is the cause of countless "works on my machine" problems.
- `pip` and `pip3` may differ if multiple Pythons are installed. Running `python -m pip ...` guarantees you use the pip belonging to the exact `python` you mean.
- `requirements.txt` records dependencies; `pip freeze` writes that file from the current environment; `pip install -r` reads it back. The two directions are opposites and are easy to confuse.

---

## 4. Variables, Objects, and References

This is the most important conceptual section in the guide, because nearly every later "gotcha" follows from it.

A Python variable is **a name bound to an object**, not a container that holds a value. Assignment never copies the object — it points another name at the same object. Picture a sticky label you stick onto a thing, not a box you pour a value into.

```python
a = [1, 2, 3]
b = a              # b is a SECOND label on the SAME list, not a copy
b.append(4)
print(a)           # [1, 2, 3, 4]   a sees the change, because a and b are one object
print(a is b)      # True           same object in memory
```

Every object has three properties: an **identity** (its address, returned by `id()` and compared with `is`), a **type** (returned by `type()`), and a **value**. The `is` operator compares identity (are these the same object?), while `==` compares value (do these objects look equal?). Mixing them up is one of the most common interview traps.

```python
x = [1, 2, 3]
y = [1, 2, 3]
print(x == y)      # True   same contents
print(x is y)      # False  two distinct objects
print(id(x) == id(y))  # False
```

The other half of the model is **mutability**. Some objects can be changed in place (mutable); others can never change once created (immutable):

| Category | Types |
| :--- | :--- |
| Immutable | `int`, `float`, `complex`, `bool`, `str`, `tuple`, `frozenset`, `bytes`, `range` |
| Mutable | `list`, `dict`, `set`, `bytearray`, most custom class instances |

With immutable objects, "changing" actually rebinds the name to a new object, so other names pointing at the old object are unaffected:

```python
a = 5
b = a
a = a + 1          # makes a NEW int 6 and rebinds a; b still points at 5
print(a, b)        # 6 5
```

With mutable objects, an in-place change is visible through every name bound to that object, which is the behavior the list example above showed. Section 33 returns to this with copying and function-argument behavior — the places it matters most.

**Notes:**
- `=` binds a name; it never copies. Two names can refer to one object, and a mutable object changed through one name is changed for all of them.
- `is` compares identity (same object), `==` compares value (equal contents). Use `==` for comparisons and reserve `is` for `None` (`if x is None`) and other singletons.
- Immutable does not mean "constant variable". You can always rebind the name; you just cannot mutate the object the name points to.
- CPython caches small integers (roughly -5 to 256) and interns some short strings, so `is` may surprise you by returning `True` for small numbers. Never rely on that; it is an implementation detail, not a language guarantee.

---

## 5. Numbers: int, float, complex, bool

Python has three numeric types plus a boolean type that is technically a number.

**`int`** is a whole number of *unbounded* size: Python integers never overflow, growing as large as memory allows. **`float`** is a 64-bit double-precision decimal, the same IEEE 754 format used everywhere, which means it carries the usual floating-point rounding quirks. **`complex`** has a real and imaginary part written with `j`. **`bool`** is a subclass of `int`, so `True` equals `1` and `False` equals `0` — and they can be used in arithmetic.

```python
big = 2 ** 100
print(big)              # 1267650600228229401496703205376  (no overflow)

print(0.1 + 0.2)        # 0.30000000000000004  (float rounding, not a Python bug)
print(0.1 + 0.2 == 0.3) # False

z = 2 + 3j
print(z.real, z.imag)   # 2.0 3.0

print(True + True)      # 2     bool is a subclass of int
print(isinstance(True, int))  # True
```

The arithmetic operators include two kinds of division — which interviewers love to separate. `/` always returns a float (true division), while `//` is floor division returning the value rounded toward negative infinity. `%` is modulo, and `**` is exponentiation.

```python
print(7 / 2)      # 3.5    true division, always float
print(7 // 2)     # 3      floor division
print(-7 // 2)    # -4     floors toward negative infinity, not toward zero
print(7 % 3)      # 1      remainder
print(-7 % 3)     # 2      sign follows the divisor in Python
print(divmod(7, 3))   # (2, 1)   quotient and remainder together
print(2 ** 10)    # 1024
```

Useful number-related built-ins and the rounding rule that surprises people:

```python
print(abs(-5))         # 5
print(round(3.14159, 2))   # 3.14
print(round(0.5), round(1.5), round(2.5))   # 0 2 2   banker's rounding (round half to even)
print(round(2.675, 2)) # 2.67   not 2.68, because 2.675 is not exactly representable
print(pow(2, 10))      # 1024
print(min(3, 1, 2), max(3, 1, 2))   # 1 3
print(int(3.9), float(3))           # 3 3.0   int() truncates toward zero
```

For exact decimal arithmetic (money, for example) use the `decimal` module; for exact fractions use `fractions`. Both avoid the binary float rounding shown above.

```python
from decimal import Decimal
print(Decimal("0.1") + Decimal("0.2"))   # 0.3   exact
```

**Notes:**
- Python integers are unbounded, so there is no `INT_MAX`. This is a real difference from C, Java, or Go and a common interview point.
- `0.1 + 0.2 != 0.3` is IEEE 754 behavior, not a Python flaw. Compare floats with a tolerance (`math.isclose`) rather than `==`, or use `Decimal` for exact decimal math.
- `round` uses banker's rounding (round half to even), so `round(0.5)` is `0`, not `1`. This trips up nearly everyone.
- `bool` is a subclass of `int`. `True == 1` and `sum([True, True, False])` is `2`, which is occasionally useful for counting.
- `//` floors toward negative infinity, and `%` takes the sign of the divisor. `-7 // 2` is `-4` and `-7 % 3` is `2`.

---

## 6. Strings

A string (`str`) is an immutable sequence of Unicode characters. Immutable is the key word: every method that "changes" a string actually returns a brand-new string and leaves the original untouched.

Strings can be written with single, double, or triple quotes. Triple quotes span multiple lines and are also how docstrings are written. Prefix letters change how the literal is interpreted: `f` enables f-strings, `r` makes a raw string (backslashes are literal), and `b` makes bytes.

```python
single = 'hello'
double = "hello"
multi = """line one
line two"""
raw = r"C:\new\path"        # backslashes stay literal: C:\new\path
print(raw)                  # C:\new\path
```

**f-strings** are the modern way to build strings: put an `f` before the quote and embed expressions in braces. They also support a rich format mini-language after a colon.

```python
name, score = "Ada", 0.875
print(f"{name} scored {score:.1%}")   # Ada scored 87.5%
print(f"{42:05d}")    # 00042   pad to width 5 with zeros
print(f"{255:#x}")    # 0xff    hex with prefix
print(f"{3.14159:.2f}")   # 3.14
print(f"{name=}")     # name='Ada'   the = shows the expression and its value (debug)
```

Since Python 3.14, **t-strings** (template strings, prefix `t`) produce a `Template` object instead of an immediate string — letting you process interpolated values before rendering (useful for safe HTML or SQL). They share f-string syntax but are not themselves a finished `str`.

Indexing and slicing work because a string is a sequence. Slicing uses `[start:stop:step]`, where `stop` is exclusive and negative indices count from the end.

```python
s = "Python"
print(s[0], s[-1])    # P n
print(s[0:3])         # Pyt   indices 0,1,2 (stop is exclusive)
print(s[::-1])        # nohtyP   reverse with a step of -1
print(s[2:])          # thon
print(len(s))         # 6
```

The method set is large; these are the ones that come up constantly:

| Method | Purpose | Example result |
| :--- | :--- | :--- |
| `upper()` / `lower()` | change case | `"Hi".upper()` to `"HI"` |
| `strip()` / `lstrip()` / `rstrip()` | trim whitespace (or given chars) | `"  hi  ".strip()` to `"hi"` |
| `split(sep)` | split into a list | `"a,b".split(",")` to `["a","b"]` |
| `join(iterable)` | join a list into one string | `"-".join(["a","b"])` to `"a-b"` |
| `replace(old, new)` | replace substrings | `"aaa".replace("a","b")` to `"bbb"` |
| `find(sub)` / `index(sub)` | locate a substring | `find` returns -1 if absent, `index` raises |
| `startswith()` / `endswith()` | prefix/suffix test | `"hi.py".endswith(".py")` to `True` |
| `count(sub)` | count occurrences | `"banana".count("a")` to `3` |
| `title()` / `capitalize()` | title or sentence case | `"a b".title()` to `"A B"` |
| `zfill(n)` | left-pad with zeros | `"7".zfill(3)` to `"007"` |
| `isdigit()` / `isalpha()` / `isspace()` | character-class tests | return `True`/`False` |

```python
print("  Hello World  ".strip())        # Hello World
print("a,b,c".split(","))               # ['a', 'b', 'c']
print("-".join(["2026", "06", "24"]))   # 2026-06-24
print("banana".count("a"))              # 3
print("file.py".endswith(".py"))        # True
print("hello".replace("l", "L"))        # heLLo
```

Because strings are immutable, building a long string by repeated `+=` in a loop is wasteful — each step creates a new string. The idiom is to collect parts in a list and `join` once.

```python
parts = []
for i in range(3):
    parts.append(str(i))
print("".join(parts))     # 012   one allocation instead of many
```

**Notes:**
- Strings are immutable. `s.upper()` returns a new string; it does not modify `s`. Forgetting this ("I called `.strip()` but the string still has spaces") is extremely common: you must assign the result back.
- Prefer f-strings over older `%` formatting and `str.format()`. They are faster and far more readable.
- `find` returns `-1` when the substring is absent; `index` raises `ValueError`. Choose based on whether "not found" is normal or exceptional.
- Build large strings with `"".join(list_of_parts)`, not repeated `+=`, to avoid quadratic copying.
- `split()` with no argument splits on any run of whitespace and drops empties, which differs from `split(" ")`.

---

## 7. Lists

A list is a mutable, ordered sequence that can hold items of any type, including a mix. It is the everyday workhorse container — the structure most algorithm questions start from.

```python
nums = [3, 1, 4, 1, 5]
mixed = [1, "two", 3.0, [4, 5]]    # any types, even nested lists
empty = []
from_range = list(range(5))        # [0, 1, 2, 3, 4]
```

Indexing and slicing match strings, but because lists are mutable you can also assign through a slice:

```python
nums = [0, 1, 2, 3, 4, 5]
print(nums[1:4])      # [1, 2, 3]
print(nums[::-1])     # [5, 4, 3, 2, 1, 0]
nums[1:3] = [10, 20, 30]   # slice assignment can change length
print(nums)           # [0, 10, 20, 30, 3, 4, 5]
```

The methods split into those that **mutate in place** (and usually return `None`) and those that **return a new value**. Confusing the two is a top-five beginner bug:

| Method | Effect | Returns |
| :--- | :--- | :--- |
| `append(x)` | add one item to the end | `None` |
| `extend(iterable)` | add each item of an iterable | `None` |
| `insert(i, x)` | insert before index `i` | `None` |
| `remove(x)` | delete first matching value | `None` |
| `pop(i)` | remove and return item at `i` (last by default) | the item |
| `sort()` | sort in place | `None` |
| `reverse()` | reverse in place | `None` |
| `clear()` | remove all items | `None` |
| `index(x)` | first index of a value | the index |
| `count(x)` | how many times `x` appears | the count |
| `copy()` | shallow copy | a new list |

```python
nums = [3, 1, 4, 1, 5]
nums.append(9)        # [3, 1, 4, 1, 5, 9]
nums.sort()           # in place: [1, 1, 3, 4, 5, 9]
print(nums)           # [1, 1, 3, 4, 5, 9]
print(nums.pop())     # 9   removes and returns the last item
print(nums.count(1))  # 2

result = [3, 1, 2].sort()   # classic trap
print(result)               # None   sort() mutates and returns None
```

`sort()` mutates in place and returns `None`; `sorted()` leaves the original alone and returns a new sorted list. Both accept `key` (a function deciding what to sort by) and `reverse`.

```python
words = ["banana", "kiwi", "apple"]
print(sorted(words, key=len))        # ['kiwi', 'apple', 'banana']
print(sorted([3, 1, 2], reverse=True))   # [3, 2, 1]
print(sorted(["B", "a", "C"], key=str.lower))   # ['a', 'B', 'C']
```

One sharp edge: list multiplication with nested lists shares the inner object. `[[0] * 3] * 2` makes two references to the *same* inner list — so writing to one row writes to both.

```python
grid = [[0] * 3] * 2
grid[0][0] = 9
print(grid)           # [[9, 0, 0], [9, 0, 0]]   not what you wanted
# correct: build independent rows
grid = [[0] * 3 for _ in range(2)]
grid[0][0] = 9
print(grid)           # [[9, 0, 0], [0, 0, 0]]
```

**Notes:**
- Mutating methods (`append`, `sort`, `reverse`, `extend`) return `None`. `x = my_list.sort()` leaves `x` as `None`. Use `sorted()` when you want a returned value.
- `append` adds one element; `extend` adds each element of an iterable. `[1,2].append([3,4])` gives `[1,2,[3,4]]`, while `extend` gives `[1,2,3,4]`.
- `[[0]*3]*2` aliases the inner list. Build grids with a comprehension so each row is independent.
- Slicing returns a new (shallow) list, so `b = a[:]` is a quick shallow copy. Nested objects are still shared (see Section 33).

---

## 8. Tuples

A tuple is an immutable, ordered sequence. Think of it as a frozen list: same indexing and slicing, but no methods that change it, and only two methods total (`count` and `index`).

```python
point = (3, 4)
single = (42,)        # a trailing comma is REQUIRED for a one-element tuple
not_tuple = (42)      # this is just the int 42 in parentheses
print(type(single), type(not_tuple))   # <class 'tuple'> <class 'int'>
empty = ()
```

The comma, not the parentheses, is what makes a tuple — so you can often omit the brackets. This is how multiple return values and swapping work:

```python
def min_max(nums):
    return min(nums), max(nums)        # returns a tuple

lo, hi = min_max([3, 1, 4, 1, 5])      # tuple unpacking
print(lo, hi)        # 1 5

a, b = 1, 2
a, b = b, a          # swap with no temp variable
print(a, b)          # 2 1
```

Unpacking extends to a starred name that absorbs the middle or ends:

```python
first, *rest = [1, 2, 3, 4]
print(first, rest)         # 1 [2, 3, 4]
first, *middle, last = [1, 2, 3, 4, 5]
print(first, middle, last) # 1 [2, 3, 4] 5
```

Why choose a tuple over a list? Three reasons interviewers want to hear. Tuples signal **intent** (this group should not change, like a coordinate). They are **hashable** when their contents are, so they can be dictionary keys or set members — which lists cannot. And they are marginally **lighter and faster** to construct.

```python
locations = {(0, 0): "origin", (1, 2): "point A"}   # tuple keys, fine
# {[0, 0]: "origin"}   # TypeError: unhashable type: 'list'
```

**Notes:**
- A one-element tuple needs a trailing comma: `(42,)`. Without it you have a parenthesized value, not a tuple.
- Tuples are immutable but can contain mutable objects. A tuple holding a list is itself unhashable, because hashability requires all contents be hashable.
- Tuple unpacking powers multiple return values, swapping, and `for key, value in d.items()`. It is everywhere in idiomatic Python.
- Prefer a tuple when the collection is a fixed record (a coordinate, an RGB color); prefer a list when it is a growing/changing collection.

---

## 9. Sets and frozenset

A set is a mutable, unordered collection of **unique, hashable** items. Its defining strengths are instant membership testing and removing duplicates. Because it is backed by a hash table, `x in my_set` is on average O(1) — versus O(n) for a list.

```python
s = {3, 1, 4, 1, 5, 9, 2, 6}
print(s)              # {1, 2, 3, 4, 5, 6, 9}   duplicates dropped, order not guaranteed
print(set([1, 1, 2, 2, 3]))   # {1, 2, 3}   deduplicate a list
empty = set()         # NOTE: {} is an empty DICT, not a set
print(4 in s)         # True   fast membership test
```

Sets shine for the mathematical set operations, available as operators or named methods:

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
print(a & b)          # {3, 4}          intersection (in both)
print(a | b)          # {1, 2, 3, 4, 5, 6}   union (in either)
print(a - b)          # {1, 2}          difference (in a, not b)
print(a ^ b)          # {1, 2, 5, 6}    symmetric difference (in exactly one)
print({1, 2} <= a)    # True            subset test
```

Common mutating methods are `add`, `remove` (raises if absent), `discard` (silent if absent), and `pop` (removes an arbitrary element):

```python
s = {1, 2, 3}
s.add(4)              # {1, 2, 3, 4}
s.discard(99)         # no error even though 99 is absent
s.remove(2)           # {1, 3, 4}
print(s)              # {1, 3, 4}
```

A **frozenset** is the immutable version. Because it cannot change, it is hashable and can be a dictionary key or an element of another set — which a normal set cannot.

```python
fs = frozenset([1, 2, 3])
groups = {fs: "primary"}      # frozenset as a dict key: allowed
# {{1, 2}: "x"}               # TypeError: a normal set is unhashable
```

**Notes:**
- `{}` is an empty dictionary, not an empty set. Use `set()` for an empty set. This is a frequent slip.
- Set elements must be hashable, so you can store numbers, strings, and tuples but not lists or dicts.
- Sets are unordered. Do not rely on any iteration order, and do not index a set (`s[0]` is an error).
- The classic interview use is deduplication and fast membership. Converting a list to a set to remove duplicates loses order; if order matters, use `dict.fromkeys(seq)` instead.

---

## 10. Dictionaries

A dictionary maps **keys to values**, with keys that must be unique and hashable. It is Python's most important data structure after the list: object attributes, JSON data, counters, and caches are all dictionaries underneath. Since Python 3.7, dictionaries **preserve insertion order** as a language guarantee.

```python
person = {"name": "Ada", "age": 36, "field": "math"}
print(person["name"])      # Ada
person["age"] = 37         # update
person["city"] = "London"  # add a new key
print(person)              # {'name': 'Ada', 'age': 37, 'field': 'math', 'city': 'London'}
```

Accessing a missing key with `[]` raises `KeyError`. The safer `get` returns `None` (or a default you supply) instead — which is the idiomatic way to read keys that may be absent:

```python
print(person.get("salary"))           # None   no error
print(person.get("salary", 0))        # 0      supply a default
# person["salary"]                    # KeyError
```

The core methods, with the three view objects (`keys`, `values`, `items`) that you iterate over:

| Method | Purpose |
| :--- | :--- |
| `get(k, default)` | read a key safely with a fallback |
| `keys()` / `values()` / `items()` | views of keys, values, or key-value pairs |
| `setdefault(k, default)` | get `k`, inserting `default` if missing |
| `update(other)` | merge another dict in (overwriting on conflict) |
| `pop(k)` | remove and return a key's value |
| `popitem()` | remove and return the last inserted pair |
| `del d[k]` | delete a key (statement, not a method) |

```python
person = {"name": "Ada", "age": 37}
for key, value in person.items():
    print(key, "=>", value)
# name => Ada
# age => 37

print(list(person.keys()))      # ['name', 'age']
print(list(person.values()))    # ['Ada', 37]
```

`setdefault` and the counting pattern come up often. To tally items you either use `setdefault`/`get` or reach for `collections.Counter` — which does it in one line:

```python
text = "banana"
counts = {}
for ch in text:
    counts[ch] = counts.get(ch, 0) + 1
print(counts)         # {'b': 1, 'a': 3, 'n': 2}

from collections import Counter
print(Counter("banana"))   # Counter({'a': 3, 'n': 2, 'b': 1})
```

Dictionaries merge with `update` or, since Python 3.9, the `|` operator. Building dicts from pairs uses `dict(zip(keys, values))` or a dict comprehension:

```python
defaults = {"color": "black", "size": "M"}
chosen = {"size": "L"}
print(defaults | chosen)      # {'color': 'black', 'size': 'L'}   right side wins

keys = ["a", "b", "c"]
vals = [1, 2, 3]
print(dict(zip(keys, vals)))  # {'a': 1, 'b': 2, 'c': 3}
print({k: k * 10 for k in range(3)})   # {0: 0, 1: 10, 2: 20}
```

**Notes:**
- Keys must be hashable, so strings, numbers, and tuples work but lists and dicts cannot be keys.
- `[]` raises `KeyError` on a missing key; `get` returns a default. Use `get` whenever absence is normal rather than exceptional.
- Insertion order is preserved (3.7+). For counting and grouping, prefer `collections.Counter` and `collections.defaultdict` over hand-rolled `get` logic.
- `dict.keys()`, `.values()`, and `.items()` are live *views*, not lists. Wrap in `list(...)` if you need a snapshot or indexing.

---

## 11. None and Truthiness

`None` is Python's single null object — the absence of a value. Functions that do not explicitly `return` something return `None`. It is a singleton, so the correct test is identity (`is None`), never `== None`.

```python
def do_nothing():
    pass
print(do_nothing())        # None

x = None
print(x is None)           # True   the idiomatic null check
```

**Truthiness** is the rule for how non-boolean values behave in a boolean context (an `if`, a `while`, `and`/`or`). Every object is either "truthy" or "falsy". The falsy values are few and worth memorizing; everything else is truthy.

| Falsy values | |
| :--- | :--- |
| `None` | the null object |
| `False` | the boolean |
| `0`, `0.0`, `0j` | any numeric zero |
| `""` | the empty string |
| `[]`, `()`, `{}`, `set()` | any empty container |

```python
print(bool(0), bool(""), bool([]), bool(None))   # False False False False
print(bool(42), bool("0"), bool([0]), bool(" ")) # True True True True

items = []
if not items:              # idiomatic "is this empty?"
    print("empty")         # empty
```

`and` and `or` do not return booleans; they return one of their operands (short-circuit evaluation). `and` returns the first falsy operand or the last one; `or` returns the first truthy operand or the last one. This enables a common default-value idiom.

```python
print(0 or "fallback")     # fallback   first truthy (or last) value
print("a" and "b")         # b          last value when all truthy
print(None or [])          # []

name = ""
display = name or "Anonymous"
print(display)             # Anonymous
```

Be careful: the `or` default trick treats *all* falsy values as "missing" — so a legitimate `0` or `""` would be replaced. When you specifically mean "if this is None", test with `is None` instead.

**Notes:**
- Test for `None` with `is None`, not `== None`. `None` is a singleton and identity is both correct and faster.
- "Empty container is falsy" lets you write `if my_list:` instead of `if len(my_list) > 0:`. The short form is the Pythonic idiom.
- `and`/`or` return an operand, not `True`/`False`. `5 and 0` is `0`; `[] or "x"` is `"x"`.
- The `x or default` idiom replaces any falsy `x`, including `0` and `""`. If those are valid values, guard with `x if x is not None else default`.

---

## 12. Type Conversion and Inspection

Python never silently coerces unrelated types (it is strongly typed), so you convert explicitly with the type constructors. These are functions named after each type: `int`, `float`, `str`, `bool`, `list`, `tuple`, `set`, `dict`.

```python
print(int("42"))          # 42     string to int
print(int("ff", 16))      # 255    parse as base 16
print(int(3.9))           # 3      float to int truncates toward zero
print(float("3.14"))      # 3.14
print(str(99))            # '99'
print(list("abc"))        # ['a', 'b', 'c']   string to list of chars
print(tuple([1, 2, 3]))   # (1, 2, 3)
print(set([1, 1, 2]))     # {1, 2}
print(bool(0), bool(1))   # False True
```

Conversions that do not make sense raise `ValueError` rather than guessing, which is the strong-typing guarantee in action:

```python
# int("hello")            # ValueError: invalid literal for int() with base 10: 'hello'
# int("3.14")             # ValueError   int() will not parse a decimal string
print(int(float("3.14"))) # 3   convert in two steps instead
```

To inspect types at runtime, use `type` and `isinstance`. The important distinction: `type(x) is C` checks the exact class, while `isinstance(x, C)` also returns `True` for subclasses — which is almost always what you want.

```python
print(type(42))                  # <class 'int'>
print(type(42) is int)           # True
print(isinstance(42, int))       # True
print(isinstance(True, int))     # True    bool is a subclass of int
print(isinstance(42, (int, float)))   # True   accepts a tuple of types
```

`isinstance` is preferred in real code because it respects inheritance — which is how polymorphism is meant to work. Reserve exact `type(...) is ...` checks for the rare case where a subclass genuinely must be rejected.

**Notes:**
- Conversion is explicit. There is no automatic `"3" + 5`; you convert one side first.
- `int("3.14")` fails. Parse a decimal string with `int(float("3.14"))`, accepting the truncation.
- Prefer `isinstance(x, T)` over `type(x) == T`. `isinstance` accepts subclasses and a tuple of types, and it is the standard idiom.
- The constructors double as converters and as empty-container factories: `list()` is `[]`, `dict()` is `{}`, `set()` is an empty set.

---

## 13. Operators

Operators group into a few families. Most are obvious — the ones worth dwelling on are identity vs equality, the membership operators, and the walrus.

**Arithmetic:** `+  -  *  /  //  %  **`. Recall from Section 5 that `/` is float division and `//` is floor division.

**Comparison:** `==  !=  <  <=  >  >=`. These return booleans and, usefully, can be **chained**: `a < b < c` reads as `a < b and b < c` with `b` evaluated once.

```python
print(1 < 2 < 3)      # True    chained, equivalent to (1 < 2) and (2 < 3)
print(5 < 3 < 10)     # False
x = 5
print(0 <= x <= 10)   # True    range check in one expression
```

**Logical:** `and`, `or`, `not`. As Section 11 showed, `and`/`or` short-circuit and return an operand, not necessarily a boolean.

**Identity vs equality:** `is` / `is not` test whether two names point at the *same object*; `==` / `!=` test whether two objects have *equal value*. Use `==` for comparisons and `is` only for `None` and other singletons.

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)      # True   equal contents
print(a is b)      # False  different objects
print(a is a)      # True
```

**Membership:** `in` / `not in` test containment, working across strings, lists, sets, dicts (keys), and any iterable.

```python
print("a" in "cat")            # True
print(3 in [1, 2, 3])          # True
print("name" in {"name": "x"}) # True   checks dict KEYS
```

**Assignment shortcuts:** `+=`, `-=`, `*=`, and so on apply an operation in place. For mutable objects like lists, `+=` mutates the existing object, while `=` with `+` would rebind — a subtle difference Section 33 revisits.

**The walrus operator `:=`** (Python 3.8+) assigns *and* returns a value inside an expression, which avoids computing something twice or saves a line in a loop or comprehension:

```python
data = [1, 2, 3, 4, 5]
if (n := len(data)) > 3:
    print(f"{n} items")        # 5 items   n is assigned and tested at once

# filter while transforming, computing double only once per item:
print([d for x in data if (d := x * 2) > 4])   # [6, 8, 10]
```

Operators have a **precedence** order (`**` binds tightest, then unary minus, then `* /`, then `+ -`, then comparisons, then `not`, `and`, `or`). When in doubt, add parentheses for the reader rather than memorizing the full table.

**Notes:**
- `is` is not a faster `==`. It compares identity, not value, and using it for general equality (`if x is 5`) is a bug that only accidentally works for cached small integers.
- Chained comparisons (`a < b < c`) are real Python and evaluate the middle term once. They are clear and idiomatic for range checks.
- The walrus `:=` needs parentheses in most positions and is for assigning within an expression. Overusing it hurts readability; reach for it when it genuinely removes duplication.
- `in` on a dict checks keys, not values. Use `value in d.values()` to search values.

---

## 14. Conditionals and Pattern Matching

The `if` / `elif` / `else` chain is the basic branch. Python uses indentation, not braces, to mark the block, and the colon is required.

```python
score = 72
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"
print(grade)          # C
```

For a single value-producing branch, the **conditional (ternary) expression** reads as `value_if_true if condition else value_if_false`:

```python
n = 7
parity = "even" if n % 2 == 0 else "odd"
print(parity)         # odd
print(max_val := (10 if 10 > 5 else 5))   # 10
```

Python has no `switch` statement. Since Python 3.10, **structural pattern matching** with `match` / `case` fills that role and does much more — it can match literal values, destructure sequences and mappings, bind names, and apply guards. The `_` case is the catch-all default.

```python
def describe(command):
    match command.split():
        case ["go", direction]:
            return f"moving {direction}"
        case ["drop", *items]:
            return f"dropping {items}"
        case ["quit"]:
            return "bye"
        case _:
            return "unknown"

print(describe("go north"))    # moving north
print(describe("drop sword shield"))   # dropping ['sword', 'shield']
print(describe("dance"))       # unknown
```

Patterns can match the shape of data, including class instances and dictionaries, with optional `if` guards:

```python
def http_status(code):
    match code:
        case 200 | 201 | 204:
            return "success"
        case n if 400 <= n < 500:
            return "client error"
        case n if 500 <= n < 600:
            return "server error"
        case _:
            return "other"

print(http_status(404))   # client error
print(http_status(201))   # success
```

For simple value dispatch, a dictionary mapping is often cleaner than either a long `if` chain or `match`:

```python
handlers = {"add": lambda a, b: a + b, "sub": lambda a, b: a - b}
print(handlers["add"](2, 3))   # 5
```

**Notes:**
- Indentation defines blocks. Mixing tabs and spaces is an error; pick spaces (PEP 8 says four) and stay consistent.
- The ternary order is `A if cond else B`, with the condition in the middle. This reads oddly at first but is the established form.
- `match`/`case` is structural pattern matching, not a plain switch. Its power is destructuring (binding parts of a value), and `case _` is the default. It needs Python 3.10+.
- A `|` in a `case` pattern means "or" between patterns; a bare name in a `case` *binds* (it does not compare to an existing variable), a subtlety that surprises people.

---

## 15. Loops

Python has two loops: `for` iterates over the items of any iterable, and `while` repeats as long as a condition holds. The `for` loop is by far the more common — because it works directly on the thing you want to traverse rather than on an index.

```python
for fruit in ["apple", "banana", "cherry"]:
    print(fruit)
# apple
# banana
# cherry

for ch in "hi":
    print(ch)          # h then i

for key, value in {"a": 1, "b": 2}.items():
    print(key, value)  # a 1 then b 2
```

`range(start, stop, step)` generates a sequence of integers lazily and is the standard way to loop a fixed number of times. `stop` is exclusive.

```python
for i in range(3):
    print(i)           # 0 1 2
print(list(range(2, 10, 2)))   # [2, 4, 6, 8]
print(list(range(5, 0, -1)))   # [5, 4, 3, 2, 1]
```

Two built-ins make `for` loops cleaner and are interview favorites. `enumerate` yields `(index, item)` pairs so you avoid manual counters, and `zip` walks several iterables in parallel.

```python
for i, fruit in enumerate(["a", "b", "c"], start=1):
    print(i, fruit)    # 1 a / 2 b / 3 c

names = ["Ada", "Alan"]
ages = [36, 41]
for name, age in zip(names, ages):
    print(name, age)   # Ada 36 / Alan 41
```

`break` exits the loop early, `continue` skips to the next iteration, and Python adds a rarely taught `else` clause on loops that runs **only if the loop finished without breaking** — perfect for search-and-report logic.

```python
for n in [4, 6, 8, 9]:
    if n % 2 != 0:
        print(f"found odd: {n}")
        break
else:
    print("all even")          # would print if no break happened
# found odd: 9
```

A `while` loop suits the case where you do not know the count in advance, such as reading until a sentinel or retrying until success:

```python
count = 3
while count > 0:
    print(count)       # 3 2 1
    count -= 1
print("liftoff")
```

**Notes:**
- Prefer `for item in collection` over `for i in range(len(collection))`. If you need the index too, use `enumerate`, not a manual counter.
- `range` is lazy: it does not build the full list in memory, so `range(10**9)` is cheap. Wrap in `list()` only when you truly need the materialized list.
- `zip` stops at the shortest iterable. Use `itertools.zip_longest` if you need to pad to the longest instead.
- The loop `else` runs when the loop completes without `break`. Read it as "no break" rather than "otherwise"; that mental relabeling makes it click.
- Modifying a list while iterating over it causes skipped or repeated items. Iterate over a copy (`for x in items[:]`) or build a new list instead.

---

## 16. Comprehensions

A comprehension builds a collection from an iterable in a single, declarative expression. It is one of Python's signature features and a near-certain interview topic. The list form reads as `[expression for item in iterable if condition]`.

```python
squares = [x**2 for x in range(5)]
print(squares)         # [0, 1, 4, 9, 16]

evens = [x for x in range(10) if x % 2 == 0]
print(evens)           # [0, 2, 4, 6, 8]

# equivalent loop, for comparison:
evens = []
for x in range(10):
    if x % 2 == 0:
        evens.append(x)
```

The same syntax produces **dict** and **set** comprehensions by changing the brackets, and a parenthesized version produces a lazy **generator**:

```python
print({x: x**2 for x in range(4)})    # {0: 0, 1: 1, 2: 4, 3: 9}   dict
print({x % 3 for x in range(7)})      # {0, 1, 2}                  set (deduped)
gen = (x**2 for x in range(4))        # generator: lazy, not a list
print(sum(gen))                       # 14   consumed on demand
```

A conditional *expression* can appear in the output part (transforming each item), which is different from the trailing `if` that *filters*:

```python
print([x if x > 0 else 0 for x in [-1, 2, -3, 4]])   # [0, 2, 0, 4]   transform
print([x for x in [-1, 2, -3, 4] if x > 0])          # [2, 4]         filter
```

Comprehensions can nest, mirroring nested loops. The order of the `for` clauses matches the order you would write the loops top to bottom:

```python
pairs = [(x, y) for x in [1, 2] for y in ["a", "b"]]
print(pairs)           # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]

matrix = [[1, 2], [3, 4]]
flat = [n for row in matrix for n in row]
print(flat)            # [1, 2, 3, 4]
```

Comprehensions are concise and usually faster than the equivalent explicit loop — but readability has a limit. If a comprehension grows two or more conditions and a nested loop, a plain `for` loop is clearer. Prefer the generator form `(...)` over `[...]` when you only iterate once and do not need to keep the whole result, because it avoids building the full list in memory.

```python
# memory-friendly: never materializes a billion-item list
total = sum(x for x in range(10**6) if x % 2 == 0)
```

**Notes:**
- The trailing `if` filters items; an `if/else` in the expression part transforms them. They sit in different places and do different jobs.
- A generator expression `(x for x in ...)` is lazy and single-use. After you iterate it once it is exhausted, which surprises people who try to reuse it.
- Nested comprehension `for` clauses read in the same order as nested loops. When in doubt, write the loops first, then collapse them.
- Do not sacrifice clarity for a one-liner. A deeply nested or multi-condition comprehension is harder to read than the loop it replaces.

---

## 17. Defining and Calling Functions

A function packages reusable logic behind a name. You define it with `def`, give it parameters, and optionally `return` a value. A function with no explicit `return` returns `None`.

```python
def greet(name):
    return f"Hello, {name}!"

print(greet("Ada"))        # Hello, Ada!

def log(message):
    print(message)         # side effect, no return
result = log("hi")         # hi
print(result)              # None
```

The first statement in a function can be a **docstring**, a string literal describing what the function does. Tools and `help()` read it, and it is the standard way to document behavior.

```python
def area(radius):
    """Return the area of a circle with the given radius."""
    return 3.14159 * radius ** 2

print(area.__doc__)        # Return the area of a circle with the given radius.
```

Classes and methods take docstrings the same way: the first string literal in the class or method body becomes its documentation. The built-in `help()` function reads all of these and prints a formatted summary, which is the fastest way to explore an unfamiliar object in the REPL.

```python
class Circle:
    """A circle defined by its radius."""
    def area(self):
        """Return the circle's area."""
        return 3.14159 * self.radius ** 2

print(Circle.__doc__)        # A circle defined by its radius.
print(Circle.area.__doc__)   # Return the circle's area.
help(Circle)                 # prints the class, its docstring, and its methods
```

Functions are **first-class objects** — you can assign them to variables, store them in lists or dicts, pass them as arguments, and return them from other functions. This is the foundation for the functional tools and decorators later in this block.

```python
def shout(text): return text.upper()
def whisper(text): return text.lower()

f = shout                  # assign a function to a name
print(f("hi"))             # HI

for fn in [shout, whisper]:
    print(fn("Hello"))     # HELLO then hello
```

`return` immediately ends the function — code after it does not run. Returning multiple values is really returning one tuple, which the caller can unpack (Section 8):

```python
def stats(nums):
    return min(nums), max(nums), sum(nums) / len(nums)

low, high, avg = stats([2, 4, 6])
print(low, high, avg)      # 2 6 4.0
```

**Notes:**
- A function with no `return` (or a bare `return`) yields `None`. Printing inside a function is a side effect, not a return value, and the two are easy to conflate.
- Functions are objects. You can pass and store them, which is what makes `sorted(data, key=fn)`, callbacks, and decorators possible.
- Write a one-line docstring for any non-trivial function. It is the cheapest documentation you will ever produce and `help()` surfaces it.
- Classes and methods take docstrings too (the first string literal in the body). `help(obj)` reads docstrings and prints a summary, the quickest way to inspect an unfamiliar object.
- "Returning multiple values" is returning a tuple. The parentheses are optional, and the caller unpacks it.

---

## 18. Parameters and Arguments

Parameters are the names in the definition; arguments are the values you pass. Python's calling convention is rich, and the ordering rules are a frequent interview question.

**Positional and keyword arguments.** You can pass by position or by name. Keyword arguments make calls self-documenting and order-independent:

```python
def rectangle(width, height):
    return width * height

print(rectangle(3, 4))             # 12   positional
print(rectangle(height=4, width=3))# 12   keyword, order does not matter
```

**Default values** make a parameter optional. Defaults are evaluated once, at definition time — which leads directly to the most infamous Python gotcha (mutable default arguments) covered below.

```python
def power(base, exponent=2):
    return base ** exponent

print(power(5))        # 25   exponent defaults to 2
print(power(5, 3))     # 125
```

**`*args` and `**kwargs`** collect extra positional and keyword arguments into a tuple and a dict respectively, letting a function accept any number of arguments:

```python
def total(*args):
    return sum(args)
print(total(1, 2, 3, 4))   # 10   args is the tuple (1, 2, 3, 4)

def describe(**kwargs):
    return kwargs
print(describe(name="Ada", age=36))   # {'name': 'Ada', 'age': 36}

def flexible(*args, **kwargs):
    return args, kwargs
print(flexible(1, 2, x=3))            # ((1, 2), {'x': 3})
```

The mirror image is **unpacking** at the call site: `*` spreads an iterable into positional arguments and `**` spreads a dict into keyword arguments.

```python
def add(a, b, c): return a + b + c
nums = [1, 2, 3]
print(add(*nums))                     # 6    spread the list
opts = {"a": 1, "b": 2, "c": 3}
print(add(**opts))                    # 6    spread the dict
```

The full parameter order is: positional-only, then normal, then `*args`, then keyword-only, then `**kwargs`. A bare `*` forces every parameter after it to be keyword-only, and a `/` forces every parameter before it to be positional-only:

```python
def f(pos_only, /, normal, *, kw_only):
    return pos_only, normal, kw_only
print(f(1, 2, kw_only=3))             # (1, 2, 3)
# f(1, 2, 3)        # TypeError: kw_only must be passed by name
```

**The mutable default trap.** Because a default is created once and shared across all calls — a mutable default (like `[]`) accumulates state between calls:

```python
def append_to(item, target=[]):       # BUG: one list shared by every call
    target.append(item)
    return target

print(append_to(1))    # [1]
print(append_to(2))    # [1, 2]   the same list persisted, not what you wanted

def append_to(item, target=None):      # FIX: sentinel + fresh list each call
    if target is None:
        target = []
    target.append(item)
    return target

print(append_to(1))    # [1]
print(append_to(2))    # [2]   fresh list every call
```

**Notes:**
- Default arguments are evaluated once at definition time. Never use a mutable default like `[]` or `{}`; use `None` as a sentinel and create the object inside the body.
- `*args` is a tuple of extra positionals; `**kwargs` is a dict of extra keywords. At the call site, `*` and `**` do the reverse, spreading a collection into arguments.
- Keyword arguments make calls readable and order-independent. For functions with several flags, prefer keyword-only parameters (after a bare `*`) so callers must name them.
- Argument order in a definition is fixed: positional-only `/`, then normal, then `*args`, then keyword-only after `*`, then `**kwargs`.

---

## 19. Scope: LEGB, global, nonlocal

When you use a name, Python searches four scopes in a fixed order, summarized by the acronym **LEGB**: **L**ocal (inside the current function), **E**nclosing (any outer function wrapping this one), **G**lobal (the module's top level), and **B**uilt-in (names like `len` and `print`). The first match wins.

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)       # local      L is found first
    inner()
    print(x)           # enclosing  E here
outer()
print(x)               # global     G here
```

Assigning to a name inside a function makes it **local by default** — even if a global of the same name exists. Reading is fine, but assigning shadows the outer name unless you say otherwise. This produces a classic confusing error:

```python
count = 0
def increment():
    count = count + 1      # UnboundLocalError: count is local here but used before assignment
# the assignment marks count as local, so the read on the right has no value yet
```

The `global` keyword lets a function rebind a module-level name, and `nonlocal` lets a nested function rebind a name in the nearest enclosing function:

```python
count = 0
def increment():
    global count           # now count refers to the module-level name
    count += 1
increment(); increment()
print(count)               # 2

def make_counter():
    n = 0
    def step():
        nonlocal n         # rebind n in the enclosing make_counter scope
        n += 1
        return n
    return step
c = make_counter()
print(c(), c(), c())       # 1 2 3
```

In everyday code you rarely need `global` — reaching for it usually signals that a value should be a return value or an object attribute instead. `nonlocal` is more legitimately useful, mostly inside closures and decorators.

**Notes:**
- Name lookup follows LEGB: Local, Enclosing, Global, Built-in, first match wins. Knowing this order explains nearly every "why is this variable that value?" question.
- Assigning to a name anywhere in a function makes it local for the whole function. Reading it before that assignment raises `UnboundLocalError`, even when a global of the same name exists.
- `global` rebinds a module-level name; `nonlocal` rebinds the nearest enclosing function's name. You only need them to *reassign*, not to read or to mutate a mutable object in place.
- Avoid `global` for state. Prefer returning values or storing state on an object; heavy `global` use makes code hard to test and reason about.

---

## 20. Lambdas and Functional Tools

A **lambda** is a small anonymous function written inline as `lambda args: expression`. Its body is a single expression whose value is returned automatically. Lambdas exist for the throwaway case where naming a function with `def` would be overkill — most often as a `key` argument.

```python
square = lambda x: x ** 2
print(square(5))           # 25

# the real use: an inline key for sorting
pairs = [("apple", 3), ("banana", 1), ("cherry", 2)]
print(sorted(pairs, key=lambda p: p[1]))   # [('banana', 1), ('cherry', 2), ('apple', 3)]
```

Three classic **functional built-ins** pair naturally with lambdas. `map` applies a function to every item, `filter` keeps the items where a predicate is true, and `functools.reduce` folds a sequence down to a single value. `map` and `filter` return lazy iterators — so wrap them in `list()` to see the contents.

```python
nums = [1, 2, 3, 4, 5]
print(list(map(lambda x: x * 2, nums)))         # [2, 4, 6, 8, 10]
print(list(filter(lambda x: x % 2 == 0, nums))) # [2, 4]

from functools import reduce
print(reduce(lambda acc, x: acc + x, nums))     # 15   sum by folding
print(reduce(lambda acc, x: acc * x, nums, 1))  # 120  product, with an initial value
```

In modern Python, a comprehension is usually preferred over `map`/`filter` with a lambda, because it is more readable and just as fast:

```python
print([x * 2 for x in nums])            # equivalent to the map above
print([x for x in nums if x % 2 == 0])  # equivalent to the filter above
```

Other genuinely useful functional helpers live in `functools` and `operator`. `functools.partial` pre-fills some arguments of a function, and the `operator` module provides function versions of operators that are faster and clearer than lambdas as keys:

```python
from functools import partial
from operator import itemgetter

add = lambda a, b: a + b
add_ten = partial(add, 10)
print(add_ten(5))                       # 15

people = [{"name": "Ada", "age": 36}, {"name": "Alan", "age": 41}]
print(sorted(people, key=itemgetter("age")))   # sorted by the age key
```

**Notes:**
- A lambda is limited to one expression with no statements, so no assignments, loops, or `return`. If you need more, use a named `def`.
- `map` and `filter` return lazy iterators in Python 3, not lists. They are single-use and must be wrapped in `list()` to materialize.
- Prefer comprehensions to `map`/`filter` plus lambda for readability. Reserve `map`/`filter` for when you already have a named function to apply.
- For sort keys, `operator.itemgetter` and `operator.attrgetter` are faster and clearer than equivalent lambdas, and they read well.

---

## 21. Closures and Decorators

A **closure** is a nested function that remembers values from the scope where it was defined, even after that outer scope has returned — the inner function "closes over" those variables. This is what lets a function carry private state without a class.

```python
def make_multiplier(factor):
    def multiply(n):
        return n * factor      # factor is remembered from the enclosing scope
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
print(double(5), triple(5))    # 10 15   each closure kept its own factor
```

A **decorator** is a function that takes a function and returns a new function wrapping it, used to add behavior (logging, timing, caching, access control) without editing the original. The `@decorator` syntax above a `def` is sugar for `func = decorator(func)`.

```python
def shout(fn):
    def wrapper(*args, **kwargs):
        return fn(*args, **kwargs).upper()
    return wrapper

@shout                         # greet = shout(greet)
def greet(name):
    return f"hi {name}"

print(greet("bob"))            # HI BOB
```

A practical decorator usually accepts `*args, **kwargs` so it works on any function, and uses `functools.wraps` to preserve the original function's name and docstring (otherwise they are replaced by the wrapper's):

```python
import functools, time

def timed(fn):
    @functools.wraps(fn)       # keep fn's __name__ and __doc__
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = fn(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{fn.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timed
def slow_sum(n):
    return sum(range(n))

slow_sum(1_000_000)            # slow_sum took 0.01...s
```

The standard library ships ready-made decorators you will use directly. `functools.lru_cache` memoizes results, turning an exponential recursive function into a fast one — `functools.cache` (3.9+) is the unbounded version:

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

print(fib(50))                 # 12586269025   instant, thanks to caching
```

Decorators can also take arguments, which requires one more layer of nesting (a function returning a decorator returning a wrapper), but the `lru_cache` example above shows the common case where you just call a parameterized decorator factory from the library.

**Notes:**
- A closure remembers the *variables*, not their values at definition time. In a loop that creates closures, every closure may share the final loop value; capture it explicitly with a default argument (`lambda x=x: ...`) if you need the per-iteration value.
- `@decorator` over a `def` is exactly `func = decorator(func)`. There is no magic; it is a function transformation applied at definition time.
- Always forward `*args, **kwargs` in the wrapper and apply `functools.wraps`, or the decorated function loses its signature, name, and docstring.
- `functools.lru_cache` / `cache` is the one-line fix for slow recursive functions and repeated pure calls. Mentioning it in an interview signals fluency.

---

## 22. Iterators and Generators

This section explains the machinery behind every `for` loop and is one of the highest-value interview topics, because it connects iteration, laziness, and memory.

An **iterable** is anything you can loop over (a list, string, dict, file). An **iterator** is the object that actually produces items one at a time. Calling `iter()` on an iterable gives an iterator, and `next()` pulls the next item until it raises `StopIteration`. A `for` loop does exactly this under the hood.

```python
it = iter([10, 20, 30])
print(next(it))        # 10
print(next(it))        # 20
print(next(it))        # 30
# next(it)             # StopIteration
```

A **generator** is the easy way to create an iterator — write a normal function but use `yield` instead of `return`. Each `yield` produces a value and *pauses* the function, resuming where it left off on the next request. Nothing runs until you iterate, and only one item exists in memory at a time.

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i        # produce i, then pause here
        i += 1

for num in count_up_to(3):
    print(num)         # 1 2 3

gen = count_up_to(3)
print(next(gen))       # 1   manual stepping works too
print(list(count_up_to(5)))   # [1, 2, 3, 4, 5]
```

The point of generators is **laziness and memory**. A list of a billion numbers needs gigabytes; a generator producing them needs almost nothing, because it computes each value on demand and forgets it. This makes generators ideal for large or infinite streams and for pipelines.

```python
def naturals():        # an infinite generator, impossible as a list
    n = 1
    while True:
        yield n
        n += 1

g = naturals()
print(next(g), next(g), next(g))   # 1 2 3   take as many as you want

# memory-light pipeline: never builds a big intermediate list
total = sum(x * x for x in range(1_000_000))
```

A generator can also receive a return value into `StopIteration` and delegate to another generator with `yield from`, which flattens nested iteration:

```python
def chain(*iterables):
    for it in iterables:
        yield from it      # yield every item from each iterable in turn

print(list(chain([1, 2], [3, 4], [5])))   # [1, 2, 3, 4, 5]
```

A generator is **single-use** — once exhausted, iterating again yields nothing. If you need to traverse twice, either recreate the generator or materialize it into a list. This trips people up when they iterate a generator to compute a length and then try to loop it again.

```python
g = (x for x in range(3))
print(list(g))         # [0, 1, 2]
print(list(g))         # []   already exhausted
```

**Notes:**
- A list holds all items at once; an iterator/generator produces them one at a time. The trade is memory for re-iterability: generators are tiny but single-pass.
- `yield` turns a function into a generator. Calling it does not run the body; it returns a generator object that runs lazily as you iterate.
- Generators are exhausted after one full pass. To reuse, rebuild the generator or store the items in a list.
- Prefer a generator expression `(...)` over a list comprehension `[...]` when feeding `sum`, `any`, `all`, `max`, or any consumer that reads once, to avoid building a throwaway list.
- `yield from sub` delegates to another iterable and is the clean way to compose or flatten generators.

---

## 23. Classes and Instances

A class is a blueprint for creating objects that bundle **data** (attributes) and **behavior** (methods) together. An instance is one object built from that blueprint. Object-oriented programming is the bulk of many interviews, so the vocabulary here matters.

The `__init__` method is the **initializer** (often loosely called the constructor) — it runs automatically when you create an instance and sets up its attributes. The first parameter of every instance method is `self`, the instance being operated on. You never pass `self` explicitly — Python supplies it.

```python
class Dog:
    def __init__(self, name, age):
        self.name = name       # instance attribute, unique per dog
        self.age = age

    def bark(self):
        return f"{self.name} says woof"

rex = Dog("Rex", 3)            # __init__ runs with self=rex
print(rex.name)                # Rex
print(rex.bark())              # Rex says woof
print(type(rex))               # <class '__main__.Dog'>
```

There are two kinds of attributes, and the distinction is a favorite question. **Instance attributes** (set on `self`) are unique to each object. **Class attributes** are defined in the class body and shared by every instance, useful for constants and shared counters.

```python
class Dog:
    species = "Canis familiaris"   # class attribute, shared by all dogs
    count = 0

    def __init__(self, name):
        self.name = name           # instance attribute, per dog
        Dog.count += 1             # mutate the shared counter

a = Dog("Rex")
b = Dog("Fido")
print(a.species, b.species)        # Canis familiaris Canis familiaris   shared
print(Dog.count)                   # 2   two dogs created
```

A subtle trap: assigning to `instance.attr` always creates or updates an *instance* attribute, even if a class attribute of the same name exists. It shadows rather than mutates the class attribute. This is why a shared mutable class attribute (like a list) is dangerous: mutating it in place affects all instances, but reassigning through one instance only shadows it.

**Notes:**
- `self` is the instance, passed automatically. Forgetting `self` in a method signature, or trying to pass it manually, are both common early errors.
- `__init__` initializes an already-created object; it is not literally the constructor (that role belongs to `__new__`, which you rarely override). Calling it "the constructor" is acceptable shorthand.
- Class attributes are shared; instance attributes are per object. Use class attributes for constants and shared state, and be cautious with mutable class attributes.
- Reading `instance.attr` falls back to the class attribute if no instance attribute exists, but assigning `instance.attr = ...` always creates an instance attribute that shadows the class one.

---

## 24. Methods: Instance, Class, Static

A class can hold three kinds of methods, distinguished by what (if anything) they receive automatically and what they operate on.

An **instance method** (the default) takes `self` and works on a specific object's data. A **class method**, marked `@classmethod`, takes `cls` (the class itself) instead of an instance and is typically used as an alternative constructor. A **static method**, marked `@staticmethod`, takes neither — it is a plain function grouped inside the class for organization.

```python
class Pizza:
    def __init__(self, toppings):
        self.toppings = toppings

    def summary(self):                     # instance method: needs an object
        return f"Pizza with {self.toppings}"

    @classmethod
    def margherita(cls):                   # class method: alternative constructor
        return cls(["cheese", "tomato"])

    @staticmethod
    def is_vegetarian(toppings):           # static method: utility, no self/cls
        return "pepperoni" not in toppings

p = Pizza.margherita()                     # build via the class method
print(p.summary())                         # Pizza with ['cheese', 'tomato']
print(Pizza.is_vegetarian(["cheese"]))     # True
```

The class method is the idiomatic way to offer **multiple ways to construct** an object. Because it receives `cls`, it also works correctly with subclasses (it builds an instance of whatever class it was called on), which a hard-coded constructor call would not.

```python
class Date:
    def __init__(self, year, month, day):
        self.year, self.month, self.day = year, month, day

    @classmethod
    def from_string(cls, text):            # "2026-06-24" -> Date(2026, 6, 24)
        y, m, d = map(int, text.split("-"))
        return cls(y, m, d)

d = Date.from_string("2026-06-24")
print(d.year, d.month, d.day)              # 2026 6 24
```

Choosing between them: if the method uses the object's data, it is an instance method. If it builds or relates to the class as a whole, it is a class method. If it does neither and just happens to belong to the class conceptually, it is a static method.

**Notes:**
- Instance methods take `self`, class methods take `cls`, static methods take neither. The decorator changes what Python passes in automatically.
- Class methods are the standard alternative-constructor pattern (`from_string`, `from_dict`). Using `cls(...)` rather than the class name makes them subclass-friendly.
- A static method is just a namespaced function. If it never touches `self` or `cls`, marking it `@staticmethod` documents that and avoids an unused parameter.
- You can call a class method on an instance too (`p.margherita()`), but calling it on the class is clearer about intent.

---

## 25. Inheritance and the MRO

Inheritance lets a class reuse and extend another. The new class (subclass/child) gets the attributes and methods of the existing one (superclass/parent) and can add to or override them. This models an "is-a" relationship — a `Dog` is an `Animal`.

```python
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        return f"{self.name} makes a sound"

class Dog(Animal):                 # Dog inherits from Animal
    def __init__(self, name, breed):
        super().__init__(name)     # call the parent initializer
        self.breed = breed
    def speak(self):               # override the parent method
        return f"{self.name} barks"

d = Dog("Rex", "Labrador")
print(d.speak())                   # Rex barks       overridden
print(d.name, d.breed)             # Rex Labrador
print(isinstance(d, Animal))       # True            a Dog is an Animal
```

In single inheritance you can read `super()` as "call the parent's version of this method", used most often in `__init__` to let the parent set up its part before the child adds more. Note that `self` inside the parent's `__init__` is the same object the child is initializing — both initializers write into one attribute namespace, so if parent and child assign the same attribute, whichever assignment runs last wins (with the parent-first `super().__init__()` pattern, that is the child's). Calling `super().__init__(...)` rather than `Animal.__init__(self, ...)` is preferred because it cooperates correctly with multiple inheritance; what `super()` really means is subtler, and the multiple-inheritance discussion below shows it.

**Polymorphism** is the payoff — code written against the parent interface works on any subclass. The same `speak()` call dispatches to the right override depending on the actual object.

```python
class Cat(Animal):
    def speak(self):
        return f"{self.name} meows"

for animal in [Dog("Rex", "Lab"), Cat("Tom")]:
    print(animal.speak())          # Rex barks / Tom meows   same call, different behavior
```

With **multiple inheritance**, a class has several parents, and Python needs a deterministic order to look up methods. That order is the **Method Resolution Order (MRO)**, computed by the C3 linearization algorithm. You can inspect it on any class, and it explains exactly which method wins in a diamond hierarchy — the classic case where two parents share a grandparent.

```python
class A:
    def who(self): return "A"
class B(A):
    def who(self): return "B"
class C(A):
    def who(self): return "C"
class D(B, C):
    pass

print(D().who())                   # B    follows the MRO
print([cls.__name__ for cls in D.__mro__])
# ['D', 'B', 'C', 'A', 'object']
```

The MRO reads left to right across parents, never places a class before its own subclass, and ends at `object`, the root of all Python classes. `Class.__mro__` is the practical tool: whenever you are unsure which method a call will hit, print it and read left to right. The first class in that list that defines the method wins.

**What `super()` really means.** The simple model, "call my parent's version", is accurate in single inheritance but it is not what `super()` actually does. `super()` means *the next class after me in the MRO of the object being handled*, and that is decided at runtime by the object's class, not when the code is written. The consequence is surprising the first time you see it:

```python
class A:
    def __init__(self):
        print("A")

class B(A):
    def __init__(self):
        print("B")
        super().__init__()

class C(A):
    def __init__(self):
        print("C")
        super().__init__()

class D(B, C):
    def __init__(self):
        print("D")
        super().__init__()

D()                    # prints D B C A   one chain through the MRO
B()                    # prints B A       same line in B, different target
```

Inside `B.__init__`, the call `super().__init__()` reached **C**, a sibling that `B` knows nothing about. When the object is a `D`, the MRO is `[D, B, C, A, object]`, so "next after B" is `C`. When the object is a plain `B`, the MRO is `[B, A, object]`, and the very same line calls `A`. One line of code, two different targets, resolved per object.

**Why it is designed this way.** In a diamond, if `B` and `C` each hardcoded `A.__init__(self)`, constructing a `D` would run `A.__init__` twice. With `super()`, the calls form a single chain that walks the MRO once, so every class initializes exactly once, in a consistent order. This pattern is called **cooperative multiple inheritance**, and it comes with a rule: if any class in a hierarchy uses `super()`, all of them should. Cooperative `__init__` methods conventionally accept `**kwargs` and pass them along, so arguments can flow through classes that do not recognize them:

```python
class Named:
    def __init__(self, name, **kwargs):
        self.name = name
        super().__init__(**kwargs)     # hand off to whoever is next

class Aged:
    def __init__(self, age, **kwargs):
        self.age = age
        super().__init__(**kwargs)

class Person(Named, Aged):
    pass

p = Person(name="Ada", age=36)     # each __init__ runs exactly once
print(p.name, p.age)               # Ada 36
```

Finally, not every hierarchy is even legal. If the bases are ordered so that C3 cannot honor both of its promises (left to right, and subclass before superclass), Python refuses to create the class at all:

```python
class X: pass
class Y(X): pass
class Z(X, Y): pass
# TypeError: Cannot create a consistent method resolution order (MRO) for bases X, Y
```

`Z` asks for `X` before `Y`, but `Y` is a subclass of `X` and must come first. The two demands contradict, so C3 fails loudly at class definition time instead of guessing.

**Notes:**
- `super().__init__(...)` calls the next initializer in the MRO and is preferred over naming the parent class directly, because it works under multiple inheritance without double-running anything.
- Polymorphism means the same method call behaves correctly across subclasses. This is what lets you write general code (`for a in animals: a.speak()`).
- The MRO (`Class.__mro__`) is the deterministic lookup order for methods, computed by C3 linearization. In a diamond like `D(B, C)`, `B` is consulted before `C`, and `object` is always last. Print it whenever method resolution is unclear.
- `super()` does not mean "my parent" — it means the next class in the runtime object's MRO. In a diamond, `super()` inside `B` can dispatch to `B`'s sibling `C`.
- Cooperative hierarchies use `super()` everywhere and pass `**kwargs` through the chain, so each class runs exactly once. Mixing `super()` with hardcoded parent calls breaks the chain.
- An impossible base ordering raises `TypeError: Cannot create a consistent method resolution order` the moment the class is defined.
- Favor composition over deep inheritance when a relationship is "has-a" rather than "is-a". Tall inheritance trees become hard to follow and to change.

---

## 26. Encapsulation and Properties

Encapsulation is the practice of controlling access to an object's internal data. Python has no truly private attributes; instead it relies on **naming conventions** and offers **properties** for controlled access. This honesty (everything is technically reachable) is itself an interview talking point.

A single leading underscore (`_name`) is a convention meaning "internal, please do not touch from outside". A double leading underscore (`__name`) triggers **name mangling** — Python rewrites it to `_ClassName__name`, which discourages accidental access and avoids clashes in subclasses, but is not real security.

If you are coming from Java or C++, the rough mapping is this: no prefix is public, a single underscore is protected by convention, and a double underscore is private via name mangling — but unlike those languages, none of these are enforced by the interpreter.

```python
class Account:
    def __init__(self, balance):
        self._balance = balance        # "internal" by convention
        self.__pin = "1234"            # name-mangled to _Account__pin

a = Account(100)
print(a._balance)                      # 100   reachable, just discouraged
# print(a.__pin)                       # AttributeError
print(a._Account__pin)                 # 1234  mangled name, still reachable
```

A **property** lets a method masquerade as an attribute, so you can run validation or computation on access without changing the public interface. You read `obj.area` like a plain attribute, but a method runs behind it. The `@property` decorator marks the getter, and an optional `@name.setter` controls assignment.

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):                  # getter: obj.radius runs this
        return self._radius

    @radius.setter
    def radius(self, value):           # setter: obj.radius = x runs this
        if value < 0:
            raise ValueError("radius cannot be negative")
        self._radius = value

    @property
    def area(self):                    # computed, read-only attribute
        return 3.14159 * self._radius ** 2

c = Circle(2)
print(round(c.area, 2))                # 12.57   looks like an attribute, runs a method
c.radius = 5                           # goes through the setter (validated)
print(round(c.area, 2))                # 78.54
# c.radius = -1                        # ValueError: radius cannot be negative
```

Properties are the Pythonic alternative to Java-style `get_x()` / `set_x()` methods. You start with a plain public attribute and only introduce a property later if you need validation or computation, without breaking any calling code — which is the whole appeal.

**Notes:**
- Python has no enforced privacy. `_x` is "internal by convention" and `__x` is name-mangled (not hidden). Both are reachable; the underscores communicate intent to humans.
- A property turns attribute access into method calls transparently, so you can add validation or make a value computed without changing how callers use it.
- Do not write Java-style getters and setters by default. Expose a plain attribute, and upgrade to a property only when you need the extra control.
- Name mangling (`__x` to `_Class__x`) exists to prevent subclass attribute collisions, not to provide access control. Do not rely on it for security.

---

## 27. Dunder Methods

Dunder ("double underscore") methods, also called magic or special methods, let your objects plug into Python's built-in syntax and functions. Defining `__len__` makes `len(obj)` work; defining `__eq__` makes `==` work; defining `__add__` makes `+` work. This is how operator overloading and Pythonic objects are built.

The most common ones, grouped by what they enable:

| Dunder | Enables | Triggered by |
| :--- | :--- | :--- |
| `__init__` | initialization | `Obj(...)` |
| `__repr__` | unambiguous debug string | `repr(obj)`, the REPL echo |
| `__str__` | readable display string | `str(obj)`, `print(obj)` |
| `__eq__` | equality | `obj == other` |
| `__lt__`, `__gt__`, etc. | ordering | `<`, `>`, sorting |
| `__hash__` | use as dict key / set member | `hash(obj)` |
| `__len__` | length | `len(obj)` |
| `__getitem__` | indexing and iteration | `obj[i]`, `for x in obj` |
| `__contains__` | membership | `x in obj` |
| `__add__`, `__mul__`, etc. | arithmetic operators | `obj + other` |
| `__call__` | call the instance like a function | `obj(...)` |
| `__enter__` / `__exit__` | context-manager `with` | `with obj:` |

`__repr__` versus `__str__` is a frequent question. `__str__` is the friendly, readable form for end users (`print`) — `__repr__` is the unambiguous, developer-facing form (the REPL and containers use it), ideally something you could paste back to recreate the object. If you define only one, make it `__repr__` — because it is the fallback for both.

```python
class Money:
    def __init__(self, amount):
        self.amount = amount
    def __str__(self):
        return f"${self.amount:.2f}"        # for humans
    def __repr__(self):
        return f"Money({self.amount})"      # for developers

m = Money(5)
print(str(m))      # $5.00
print(repr(m))     # Money(5)
print([m])         # [Money(5)]   containers use __repr__ on their elements
```

Operator overloading lets your type behave like a built-in. A vector class with `__add__` and `__eq__` reads naturally:

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)
    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

print(Vector(1, 2) + Vector(3, 4))     # Vector(4, 6)
print(Vector(1, 2) == Vector(1, 2))    # True
```

If you define `__eq__`, you usually must also define `__hash__` (or the class becomes unhashable and cannot go in a set or be a dict key), because Python's rule is that equal objects must have equal hashes. Implementing `__getitem__` and `__len__` is enough to make a custom object indexable and iterable, since Python falls back to integer-indexing when no `__iter__` exists.

**Notes:**
- `__str__` is for users (readable), `__repr__` is for developers (unambiguous). Define `__repr__` at minimum; it is the fallback when `__str__` is missing and is what containers display.
- Defining `__eq__` without `__hash__` makes instances unhashable. If the object should be usable in sets or as dict keys, define both, keeping "equal implies equal hash".
- Operator overloading via `__add__`, `__mul__`, and friends makes custom types feel native, but only overload operators where the meaning is obvious to a reader.
- `__getitem__` plus `__len__` gives indexing and iteration for free. For full control over iteration, implement `__iter__` returning an iterator instead.

---

## 28. Dataclasses

Writing `__init__`, `__repr__`, and `__eq__` by hand for a class that is mostly a bundle of fields is tedious and error-prone — the `@dataclass` decorator (Python 3.7+, in the standard library) generates all of that from type-annotated field declarations, removing the boilerplate.

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

p = Point(1, 2)
print(p)               # Point(x=1, y=2)   __repr__ generated
print(p == Point(1, 2))  # True            __eq__ generated
print(p.x, p.y)        # 1 2
```

Fields can have defaults, and mutable defaults must use `field(default_factory=...)` to avoid the shared-default trap from Section 18. Passing `order=True` generates the comparison methods so instances sort, and `frozen=True` makes instances immutable (and hashable).

```python
from dataclasses import dataclass, field

@dataclass(order=True)
class Item:
    name: str
    quantity: int = 0
    tags: list = field(default_factory=list)   # fresh list per instance

a = Item("widget", 5)
b = Item("widget", 9)
print(a)               # Item(name='widget', quantity=5, tags=[])
print(a < b)           # True   ordered by fields in declaration order

@dataclass(frozen=True)
class Coord:
    lat: float
    lon: float

c = Coord(51.5, -0.1)
# c.lat = 0            # FrozenInstanceError: cannot assign to a frozen field
print({c})             # {Coord(lat=51.5, lon=-0.1)}   hashable because frozen
```

Dataclasses are the modern default for "plain data holder" classes — configuration objects, records, value types, and the results returned from functions. When you need richer validation or serialization, libraries like `pydantic` extend the same annotation-driven idea, but the standard-library dataclass covers most cases.

**Notes:**
- `@dataclass` generates `__init__`, `__repr__`, and `__eq__` from annotated fields. It is the antidote to repetitive boilerplate classes.
- Mutable field defaults require `field(default_factory=list)`, never `tags: list = []`, for the same reason mutable default arguments are dangerous.
- `frozen=True` gives an immutable, hashable instance suitable for set members and dict keys; `order=True` gives sortable instances compared field by field.
- Reach for a dataclass whenever a class is mostly data. Keep regular classes for objects whose identity and behavior dominate over their fields.

---

## 29. Errors and Exceptions

When something goes wrong, Python raises an **exception** — an object describing the error that propagates up the call stack until something handles it or the program crashes. Handling exceptions is how you write robust code, and the `try` / `except` mechanics are interview staples.

The basic form catches a specific exception type and responds to it instead of crashing:

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    result = None
    print("cannot divide by zero")
print(result)              # cannot divide by zero \n None
```

The full structure has four clauses, each with a distinct job. `try` holds the risky code, `except` handles a matching error, `else` runs only if no exception occurred, and `finally` always runs (success or failure) — which is where cleanup belongs.

```python
def read_number(text):
    try:
        value = int(text)          # might raise ValueError
    except ValueError:
        print("not a number")
        return None
    else:
        print("parsed cleanly")    # runs only if no exception
        return value
    finally:
        print("done")              # always runs

read_number("42")
# parsed cleanly
# done
read_number("abc")
# not a number
# done
```

Catch specific exceptions, not a bare `except:`. You can catch several types in one tuple, and capturing the exception object with `as` gives you its message and details:

```python
try:
    data = {"a": 1}
    print(data["b"])
except (KeyError, IndexError) as e:
    print(f"lookup failed: {type(e).__name__}: {e}")
# lookup failed: KeyError: 'b'
```

You **raise** exceptions yourself to signal invalid states, and you can define your own exception classes by subclassing `Exception` for domain-specific errors. The `raise ... from ...` form preserves the original cause when re-raising, which keeps tracebacks honest.

```python
class InsufficientFundsError(Exception):
    pass

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(f"need {amount}, have {balance}")
    return balance - amount

try:
    withdraw(100, 150)
except InsufficientFundsError as e:
    print(e)               # need 150, have 100
```

The exception hierarchy matters: catching a broad parent like `Exception` also catches its subclasses, so order your `except` clauses from specific to general. Never silence errors with a bare `except: pass`, which hides bugs including ones you did not anticipate (a mistyped name raises `NameError`, which you would swallow silently).

```python
# Python 3.14 allows omitting the parentheses around multiple exception types:
# except ValueError, TypeError as e:   (PEP 758)
```

**Notes:**
- Catch specific exception types, ordered specific-to-general. A bare `except:` (or `except Exception`) hides real bugs and makes debugging miserable.
- `finally` always runs, even if the `try` returns or re-raises, so it is the right place for cleanup. The `else` clause runs only when no exception was raised.
- Define custom exceptions by subclassing `Exception` for clear, catchable domain errors. Subclass `Exception`, not the lower-level `BaseException`, which also covers `KeyboardInterrupt` and `SystemExit`.
- "Ask forgiveness, not permission" (try/except) is often more Pythonic than checking conditions first, especially for race-prone operations like file or dict access.

---

## 30. Files and Context Managers

Reading and writing files uses the built-in `open`, which returns a file object. The right way to use it is the `with` statement, a **context manager** that guarantees the file is closed even if an error occurs midway. Forgetting to close files leaks resources — `with` removes that risk entirely.

```python
# write a file
with open("notes.txt", "w") as f:
    f.write("first line\n")
    f.write("second line\n")
# the file is automatically closed here, even on an exception

# read it all back
with open("notes.txt", "r") as f:
    content = f.read()
print(content)
# first line
# second line
```

The `open` mode controls the operation — `"r"` read (default), `"w"` write (truncates the file), `"a"` append, `"x"` create-only (fails if it exists), and a `"b"` suffix for binary. Always pass `encoding="utf-8"` for text to avoid platform-dependent defaults.

| Mode | Meaning |
| :--- | :--- |
| `"r"` | read, error if the file is missing (default) |
| `"w"` | write, creating or truncating the file |
| `"a"` | append to the end, creating if missing |
| `"x"` | create a new file, error if it already exists |
| `"r+"` | read and write |
| `"rb"` / `"wb"` | binary read / write |

For large files, iterate line by line rather than reading everything into memory. A file object is itself an iterator that yields one line at a time, so a `for` loop streams it:

```python
with open("notes.txt", encoding="utf-8") as f:
    for line in f:                 # streams one line at a time, memory-light
        print(line.rstrip())       # rstrip drops the trailing newline
```

A **context manager** is any object implementing `__enter__` and `__exit__`. `with` calls `__enter__` on entry and `__exit__` on exit (even on error), which is the pattern behind files, locks, and database connections. You can write your own most easily with the `contextlib.contextmanager` decorator on a generator:

```python
from contextlib import contextmanager

@contextmanager
def timer(label):
    import time
    start = time.perf_counter()
    yield                          # the with-block runs here
    print(f"{label}: {time.perf_counter() - start:.4f}s")

with timer("work"):
    sum(range(1_000_000))
# work: 0.0...s
```

For filesystem paths, prefer `pathlib.Path` over string manipulation. It is cross-platform and offers a clean object interface for joining, reading, and inspecting paths:

```python
from pathlib import Path
p = Path("data") / "notes.txt"     # OS-correct path joining with /
print(p.suffix)                    # .txt
# p.write_text("hi", encoding="utf-8")   # one-line write
# text = p.read_text(encoding="utf-8")   # one-line read
```

**Notes:**
- Always open files with `with`, so they close automatically even on an exception. Manual `open`/`close` pairs leak file handles when an error skips the `close`.
- Mode `"w"` truncates the file to empty before writing. Use `"a"` to append, and `"x"` to refuse to overwrite an existing file.
- Pass `encoding="utf-8"` explicitly for text files. The default encoding is platform-dependent and a frequent source of cross-machine bugs (3.15 will make UTF-8 the default).
- Iterate a file object directly to stream large files line by line instead of `read()`-ing the whole thing into memory.
- Use `pathlib.Path` for path work; it replaces fragile `os.path.join` string juggling with a clear, portable API.

---

## 31. Modules, Packages, and the Standard Library

A **module** is a single `.py` file of reusable code — a **package** is a folder of modules. The `import` statement loads them so you can use their names. This is covered in depth in the companion project-anatomy guide, so here the focus is the standard library, the large set of modules that ship with Python and that interviewers expect you to reach for.

The import forms and what name you end up using:

```python
import math                     # use as math.sqrt
print(math.sqrt(16))            # 4.0

from math import sqrt, pi       # use sqrt directly
print(sqrt(25), pi)             # 5.0 3.141592653589793

import datetime as dt           # alias for brevity
print(dt.date.today().year)     # 2026
```

The "batteries included" standard library is a genuine Python selling point. The modules that come up most in interviews and daily work:

| Module | What it gives you |
| :--- | :--- |
| `math` | numeric functions: `sqrt`, `floor`, `gcd`, `pi`, `inf` |
| `random` | random numbers, `choice`, `shuffle`, `sample` |
| `datetime` | dates, times, and durations |
| `collections` | `Counter`, `defaultdict`, `deque`, `namedtuple`, `OrderedDict` |
| `itertools` | iterator building blocks: `chain`, `combinations`, `groupby`, `count` |
| `functools` | `reduce`, `lru_cache`, `partial`, `wraps` |
| `os` / `sys` | operating system and interpreter interaction |
| `pathlib` | object-oriented filesystem paths |
| `json` | parse and serialize JSON |
| `re` | regular expressions |

A few `collections` types are worth knowing by name because they turn several lines of code into one — `Counter` tallies, `defaultdict` removes "key may be missing" boilerplate, and `deque` is a fast double-ended queue (O(1) appends and pops at both ends, unlike a list which is O(n) at the front):

```python
from collections import Counter, defaultdict, deque

print(Counter("mississippi"))          # Counter({'i': 4, 's': 4, 'p': 2, 'm': 1})

groups = defaultdict(list)             # missing keys default to an empty list
for word in ["apple", "ant", "bee"]:
    groups[word[0]].append(word)
print(dict(groups))                    # {'a': ['apple', 'ant'], 'b': ['bee']}

q = deque([1, 2, 3])
q.appendleft(0)                        # O(1) at the front
q.append(4)
print(q)                               # deque([0, 1, 2, 3, 4])
```

`itertools` provides lazy combinatorial and iteration tools that are common in interview solutions:

```python
import itertools
print(list(itertools.combinations([1, 2, 3], 2)))   # [(1, 2), (1, 3), (2, 3)]
print(list(itertools.permutations([1, 2], 2)))      # [(1, 2), (2, 1)]
print(list(itertools.accumulate([1, 2, 3, 4])))     # [1, 3, 6, 10]
```

`json` converts between Python objects and JSON text, which is most API and config work:

```python
import json
data = {"name": "Ada", "skills": ["math", "code"]}
text = json.dumps(data)                # Python -> JSON string
print(text)                            # {"name": "Ada", "skills": ["math", "code"]}
back = json.loads(text)                # JSON string -> Python
print(back["name"])                    # Ada
```

**Notes:**
- "Batteries included" is real: before adding a third-party dependency, check whether the standard library already solves it. `collections`, `itertools`, and `functools` cover an enormous amount.
- `from module import name` imports a name directly; `import module` keeps the namespace visible (`module.name`). Avoid `from module import *`, which hides where names came from.
- `collections.deque` is the right structure when you need fast pops/appends at the front; a list is O(n) there. Knowing this distinction is a common interview signal.
- `defaultdict` and `Counter` replace hand-written "if key not in dict" patterns and are cleaner and faster. Reach for them in grouping and counting problems.

---

## 32. Type Hints

Type hints (PEP 484, Python 3.5+) annotate the expected types of variables, parameters, and return values. They are **optional and not enforced at runtime** — Python ignores them when executing. Their value is for human readers, editors (autocomplete and inline errors), and static checkers like `mypy` or `pyright` that catch type bugs before the program runs.

```python
def greet(name: str, times: int = 1) -> str:
    return f"hi {name} " * times

age: int = 36
names: list[str] = ["Ada", "Alan"]
scores: dict[str, float] = {"Ada": 0.9}
print(greet("Ada", 2))     # hi Ada hi Ada
```

The `->` in a signature introduces the **return type annotation**: `def greet(...) -> str:` declares that `greet` returns a `str`, just as `name: str` declares the parameter's type. The arrow is only readable syntax that Python itself doesn't enforce, but frameworks such as FastAPI and libraries like pydantic read these annotations at runtime and build real behavior (request parsing, validation, docs) from them.

Since Python 3.9 you write the built-in generics directly (`list[int]`, `dict[str, int]`, `tuple[int, ...]`) with no imports. The `typing` module supplies the rest. `Optional[X]` means "X or None" and `X | Y` (3.10+) is the modern union syntax. `Any` opts out of checking for a value.

```python
from typing import Optional

def find_user(uid: int) -> Optional[str]:   # returns str or None
    users = {1: "Ada"}
    return users.get(uid)

def parse(value: int | str) -> int:         # accepts int OR str (3.10+ syntax)
    return int(value)

print(find_user(1))        # Ada
print(find_user(99))       # None
```

A type alias names a complex type for reuse, and since 3.12 the `type` statement makes that explicit. Custom classes are valid type hints too, so you annotate with your own types just as readily:

```python
type Matrix = list[list[float]]            # 3.12+ type alias statement

def transpose(m: Matrix) -> Matrix:
    return [list(row) for row in zip(*m)]

print(transpose([[1, 2], [3, 4]]))         # [[1, 3], [2, 4]]
```

Type hints shine in larger codebases and at team boundaries — a well-typed function signature documents intent precisely and lets tools catch a whole class of bugs (passing a `str` where an `int` is expected) without writing a single test. They cost a little verbosity, which is why small scripts often skip them, but for anything shared or long-lived they pay off.

**Notes:**
- Type hints are not enforced at runtime. Python does not check them; a static analyzer (`mypy`, `pyright`) or your editor does. Passing the wrong type still runs unless a checker flags it first.
- Use built-in generics (`list[int]`, `dict[str, int]`) on 3.9+ rather than the old capitalized `typing.List`. Use `X | Y` for unions on 3.10+.
- `Optional[X]` is shorthand for `X | None`. Annotate functions that may return nothing this way so callers know to handle `None`.
- Hints are most valuable in shared libraries, APIs, and large projects. For a five-line throwaway script they are optional; for code others depend on, they are close to essential.

---

## 33. Mutability, Identity, and Copying

This section gathers the reference-and-mutability ideas introduced in Section 4 into the three places they cause real bugs and real interview questions: function arguments, copying, and augmented assignment. Everything here follows from one fact: assignment binds a name to an object and never copies.

**Function arguments are passed by assignment.** When you call `f(x)`, the parameter inside `f` is bound to the *same object* `x` points to. So mutating that object inside the function is visible to the caller, but *rebinding* the parameter to a new object is not. People call this "pass by object reference", and distinguishing the two cases is the whole question.

```python
def mutate(lst):
    lst.append(99)         # mutates the shared object: visible outside
def rebind(lst):
    lst = [0]              # rebinds the local name only: invisible outside

x = [1, 2]
mutate(x)
print(x)                   # [1, 2, 99]   the caller's list changed

y = [1, 2]
rebind(y)
print(y)                   # [1, 2]        the caller's list is untouched
```

The same single rule explains why immutable arguments *look* like pass by value. Rebinding an `int` (or any immutable) inside a function only moves the local name to a new object, so the caller is untouched:

```python
def add_ten(n):
    n = n + 10             # rebinds the local name to a new int
    print(n)               # 15

num = 5
add_ten(num)
print(num)                 # 5    the caller's int is unchanged
```

There is no second mechanism at work: the list example mutates a shared object and the int example rebinds a local name, but both follow the one rule that the parameter is bound to the same object the caller passed. Some texts call this model **call by sharing**.

This is also why mutable default arguments are dangerous (Section 18) and why returning a new value is often safer than mutating an argument in place.

**Copying: shallow vs deep.** Assignment makes an alias, not a copy — to get an independent object you copy explicitly. A **shallow copy** duplicates the outer container but shares the nested objects inside it — a **deep copy** recursively duplicates everything. For a flat list of immutables the distinction does not matter, but for nested structures it is the difference between safety and a surprise.

```python
import copy

original = [[1, 2], [3, 4]]
shallow = copy.copy(original)       # also: original[:] or list(original)
deep = copy.deepcopy(original)

original[0][0] = 99
print(shallow)     # [[99, 2], [3, 4]]   inner list is shared, so it changed too
print(deep)        # [[1, 2], [3, 4]]    fully independent
```

Common shallow-copy idioms are `list(x)`, `x[:]`, `dict(x)`, and `x.copy()`. Reach for `copy.deepcopy` only when you have genuinely nested mutable data, because it is slower.

**Augmented assignment is not always in-place.** `a += b` calls the in-place operation when the object supports it (lists do, via `__iadd__`), mutating the existing object. But `a = a + b` always builds a new object and rebinds. For a shared list these differ visibly:

```python
a = [1, 2]; b = a
a += [3]               # in-place extend: same object, so b sees it
print(b)               # [1, 2, 3]

a = [1, 2]; b = a
a = a + [3]            # new list, rebind a only
print(b)               # [1, 2]          b still points at the original
```

**Identity caching is an implementation detail.** CPython reuses small integer objects (about -5 to 256) and interns some strings, so `is` can return `True` for equal small values. This is not a guarantee and varies by context (the same literal on one line may be folded to one object, while values computed separately are not). The lesson is simply: never use `is` to compare values. Use `==` for equality and reserve `is` for `None` and other singletons.

```python
# Do not rely on either of these results; they are implementation-dependent:
print(256 is 256)      # may be True (cached)
# Always compare values with ==:
print(1000 == 1000)    # True, the correct way to compare
```

**Notes:**
- Arguments are passed by assignment (object reference). Mutating the argument's object affects the caller; rebinding the parameter does not. This single rule explains most "did my function change my data?" questions.
- A shallow copy shares nested objects; a deep copy duplicates them recursively. Use `copy.deepcopy` only for nested mutable structures, since it is slower and occasionally hits reference cycles.
- `a += b` may mutate in place (lists), while `a = a + b` always rebinds. For shared mutable objects the two have different visible effects.
- `is` compares identity and must not be used for value equality. Small-integer caching and string interning are CPython optimizations, not language guarantees.

---

## 34. How Python Executes: CPython, Bytecode, the GIL

Interviewers often probe whether you understand what happens beneath the syntax. The key facts are the bytecode compile step, reference-counting memory management, and the Global Interpreter Lock.

**Source to bytecode to VM.** CPython compiles your source into **bytecode** (a compact, platform-independent instruction set) and then a virtual machine executes those instructions. You can see the bytecode with the `dis` module, which demystifies "what does this line actually do":

```python
import dis
def add(a, b):
    return a + b
dis.dis(add)
# shows LOAD_FAST a, LOAD_FAST b, BINARY_OP +, RETURN_VALUE
```

The `.pyc` files in `__pycache__` are cached bytecode so unchanged modules skip recompilation. They are an optimization, not something you manage.

**Memory management is automatic.** CPython primarily uses **reference counting**: every object tracks how many names point at it, and when that count hits zero the memory is freed immediately. A separate **cyclic garbage collector** handles reference cycles (objects that point at each other) that reference counting alone cannot reclaim. You never `free` anything by hand.

```python
import sys
x = [1, 2, 3]
y = x
print(sys.getrefcount(x))   # a small number: refs from x, y, and the call itself
```

**The GIL.** The **Global Interpreter Lock** is a mutex in CPython that allows only one thread to execute Python bytecode at a time. This is the single most-asked Python internals question. Its consequence — threads do not give you parallel speedup for **CPU-bound** Python code, because only one runs at once. They *do* help **I/O-bound** code (network, disk, waiting), because a thread waiting on I/O releases the GIL so another can run.

The practical decision tree most interviews want:

| Workload | Best tool | Why |
| :--- | :--- | :--- |
| I/O-bound (network, files, DB) | `threading` or `asyncio` | waiting threads release the GIL; concurrency hides latency |
| CPU-bound (number crunching) | `multiprocessing` | separate processes sidestep the GIL with true parallelism |
| Many concurrent connections | `asyncio` | single-threaded cooperative concurrency, very scalable for I/O |

```python
# CPU-bound work parallelizes across processes, not threads:
from multiprocessing import Pool
def square(n): return n * n
# with Pool(4) as p:
#     print(p.map(square, range(10)))   # uses multiple cores
```

A major recent change: **Python 3.13 introduced an experimental free-threaded build** that can run without the GIL, and **Python 3.14 made free-threaded Python officially supported** (PEP 779). It is a separate build, not yet the default — but it signals that the long-standing GIL limitation is finally being lifted. For the foreseeable future, though, the standard interpreter still has the GIL, so the table above remains the right answer in an interview.

**Notes:**
- Python compiles to bytecode, then a VM runs it. "Interpreted" is shorthand; the bytecode step is the detail interviewers reward you for knowing. The `dis` module lets you inspect it.
- Memory is managed by reference counting plus a cycle collector. Objects are freed when their reference count reaches zero; you never free memory manually.
- The GIL allows one thread to run Python bytecode at a time. Use `multiprocessing` for CPU-bound parallelism and `threading`/`asyncio` for I/O-bound concurrency. This is the canonical answer.
- Free-threaded (no-GIL) CPython is officially supported as of 3.14 but is still an opt-in build, not the default. Mentioning it shows you are current.

---

## 35. Memory, Iteration, and Performance Habits

A handful of habits separate code that scales from code that does not, and they make good interview talking points because each has a clear "why".

**Prefer lazy iteration over building large lists.** Generators and `range` produce items on demand, so they use near-constant memory regardless of size. Building a full list first wastes memory you may never need.

```python
# wasteful: materializes a million-item list just to sum it
total = sum([x * x for x in range(1_000_000)])
# better: a generator expression streams values, building no list
total = sum(x * x for x in range(1_000_000))
```

**Know the cost of common operations.** Choosing the right container is often the entire optimization. Membership testing and the front of a sequence are the classic traps:

| Operation | List | Set / Dict | Note |
| :--- | :--- | :--- | :--- |
| `x in collection` | O(n) | O(1) | use a set/dict for repeated lookups |
| index access `a[i]` | O(1) | n/a | lists are arrays under the hood |
| append at end | O(1) amortized | n/a | lists grow cheaply at the tail |
| insert/pop at front | O(n) | n/a | use `collections.deque` for O(1) |
| key lookup | n/a | O(1) | dicts are hash tables |

```python
# O(n) per check, O(n*m) overall, slow for big data:
big = list(range(1_000_000))
found = 999_999 in big            # scans the whole list

# O(1) per check after a one-time O(n) conversion:
big_set = set(big)
found = 999_999 in big_set        # instant
```

**Use the right tool for joins and counts.** Build strings with `"".join(parts)`, count with `Counter`, group with `defaultdict`, and cache pure functions with `lru_cache`. Each replaces a slower hand-written loop and reads better.

**Measure before optimizing.** Use `timeit` for micro-benchmarks and a profiler (`cProfile`) for whole programs, rather than guessing. Premature optimization on the wrong line is wasted effort — the slow part is frequently not where you expect.

```python
import timeit
t = timeit.timeit("'-'.join(str(n) for n in range(100))", number=10000)
print(round(t, 4), "seconds for 10k runs")
```

**Write clear code first.** Python's primary strength is readability and developer speed. Reach for the clever optimization only when a measurement shows you need it — for most code, the clear version is both fast enough and the version your reviewers (and interviewers) prefer.

**Notes:**
- Generators and `range` are lazy and memory-light. Prefer a generator expression to a list comprehension whenever the result is consumed once by `sum`, `any`, `all`, `max`, or a loop.
- Use a set or dict for repeated membership tests; `in` is O(1) there versus O(n) on a list. Converting a list to a set once pays for itself across many lookups.
- `collections.deque` gives O(1) operations at both ends; a list is O(n) at the front. Pick the structure that matches your access pattern.
- Profile before optimizing. `timeit` for snippets, `cProfile` for programs. Optimizing the wrong line is the most common wasted effort, and "I would measure first" is a strong interview answer.

---

## 36. Common Mistakes

**Using `is` to compare values.** `is` checks identity, not equality. `if x is 5` only works by accident through small-integer caching. Use `==` for values and reserve `is` for `None` and singletons.

**Mutable default arguments.** `def f(x, acc=[])` shares one list across all calls, accumulating state. Use `acc=None` and create the list inside the function.

**Expecting `sort()` to return a list.** `list.sort()` mutates in place and returns `None`. Use `sorted(...)` when you want a returned value.

**Aliasing instead of copying.** `b = a` makes a second name for the same object, not a copy. Mutating through `b` changes what `a` sees. Copy explicitly with `list(a)`, `a[:]`, `a.copy()`, or `copy.deepcopy` for nested data.

**`[[0]*3]*2` shares the inner list.** List multiplication repeats references. Build grids with `[[0]*3 for _ in range(2)]` so each row is independent.

**Confusing `{}` with an empty set.** `{}` is an empty dict. Use `set()` for an empty set.

**Forgetting `input()` returns a string.** `input()` always yields `str`. Wrap in `int(...)` or `float(...)` when you need a number.

**Modifying a list while iterating it.** Removing items mid-loop skips elements. Iterate over a copy (`for x in items[:]`) or build a new filtered list.

**Bare `except:`.** Catching everything hides real bugs, including typos that raise `NameError`. Catch specific exceptions, ordered specific-to-general.

**Comparing floats with `==`.** `0.1 + 0.2 != 0.3` due to IEEE 754. Use `math.isclose` or `Decimal` for exact decimals.

**Reusing an exhausted generator.** A generator is single-pass. After one full iteration it yields nothing; rebuild it or store the items in a list.

**Building strings with `+=` in a loop.** Each step copies the whole string (O(n squared) overall). Collect parts in a list and `"".join` once.

**`mutable += ` surprises.** `a += [x]` mutates a shared list in place, while `a = a + [x]` rebinds. The two differ when another name shares the object.

**Defining `__eq__` without `__hash__`.** This silently makes instances unhashable, so they cannot go in a set or be dict keys. Define both, keeping equal objects equal-hashed.

**Shadowing built-ins.** Naming a variable `list`, `dict`, `str`, `sum`, or `id` overwrites the built-in for the rest of the scope. Pick another name.

---

## 37. Interview Quick-Fire

Short, high-frequency questions with the answers an interviewer expects.

**Is Python interpreted or compiled?** Both, in a sense: CPython compiles source to bytecode, then a virtual machine interprets the bytecode. Casually "interpreted", precisely "compiled to bytecode, then interpreted".

**Dynamically vs strongly typed?** Python is both. Dynamic: types attach to objects at runtime and names can be rebound to any type. Strong: it refuses unsafe implicit coercions like `"3" + 5`.

**`is` vs `==`?** `is` compares identity (same object); `==` compares value (equal contents). Use `==` for comparisons, `is` for `None`.

**List vs tuple?** Lists are mutable and for changing collections; tuples are immutable, hashable (so usable as keys), and signal a fixed record.

**Set vs list?** Sets are unordered, hold unique hashable items, and offer O(1) membership; lists are ordered, allow duplicates, and are O(n) for `in`.

**Shallow vs deep copy?** Shallow duplicates the outer container but shares nested objects; deep duplicates everything recursively.

**What is the GIL?** A lock allowing one thread to execute Python bytecode at a time. Threads help I/O-bound work; use `multiprocessing` for CPU-bound parallelism. Free-threaded builds are officially supported from 3.14 but not yet default.

**`*args` and `**kwargs`?** Collect extra positional arguments into a tuple and extra keyword arguments into a dict, letting a function accept any number of arguments.

**Decorator?** A function that takes a function and returns a wrapped one to add behavior. `@d` over a `def` is `func = d(func)`.

**Generator vs list?** A generator yields items lazily one at a time (constant memory, single-pass); a list holds all items at once (re-iterable, more memory).

**`@staticmethod` vs `@classmethod`?** A class method takes `cls` and is typically an alternative constructor; a static method takes neither and is just a namespaced utility.

**How does Python manage memory?** Reference counting frees objects when their count hits zero, plus a cycle collector for reference cycles.

**Mutable vs immutable types?** Immutable: `int`, `float`, `str`, `tuple`, `frozenset`, `bytes`. Mutable: `list`, `dict`, `set`, and most class instances.

**What does a dict guarantee about order?** Insertion order is preserved since Python 3.7.

**`__str__` vs `__repr__`?** `__str__` is the readable user form (`print`); `__repr__` is the unambiguous developer form (REPL, containers). Define `__repr__` at minimum.

**How do you make code faster?** Choose the right data structure (set/dict for lookups, deque for ends), iterate lazily, cache pure functions, and profile before optimizing.

---

## 38. Cheat Sheet

**Built-in data types**

| Type | Mutable | Ordered | Example |
| :--- | :--- | :--- | :--- |
| `int`, `float`, `complex` | no | n/a | `42`, `3.14`, `2+3j` |
| `bool` | no | n/a | `True`, `False` |
| `str` | no | yes | `"hello"` |
| `list` | yes | yes | `[1, 2, 3]` |
| `tuple` | no | yes | `(1, 2, 3)` |
| `set` | yes | no | `{1, 2, 3}` |
| `frozenset` | no | no | `frozenset({1, 2})` |
| `dict` | yes | yes (insertion) | `{"a": 1}` |
| `NoneType` | no | n/a | `None` |

**Sequence operations (string, list, tuple)**

| Task | Code |
| :--- | :--- |
| length | `len(seq)` |
| index | `seq[0]`, `seq[-1]` |
| slice | `seq[1:4]`, `seq[::-1]`, `seq[::2]` |
| membership | `x in seq` |
| concatenate | `seq1 + seq2` |
| repeat | `seq * 3` |

**Control flow**

| Construct | Form |
| :--- | :--- |
| branch | `if cond: ... elif cond: ... else: ...` |
| ternary | `a if cond else b` |
| pattern match | `match x: case pattern: ...` |
| for loop | `for item in iterable:` |
| while loop | `while cond:` |
| comprehension | `[expr for x in it if cond]` |

**Functions**

| Feature | Form |
| :--- | :--- |
| define | `def f(a, b=1, *args, **kwargs):` |
| lambda | `lambda x: x * 2` |
| keyword-only | `def f(a, *, b):` |
| positional-only | `def f(a, /, b):` |
| unpack call | `f(*list)`, `f(**dict)` |
| decorator | `@decorator` above `def` |

**Classes**

| Feature | Form |
| :--- | :--- |
| define | `class C:` |
| init | `def __init__(self, ...):` |
| inherit | `class D(C):` with `super().__init__()` |
| class method | `@classmethod def m(cls):` |
| static method | `@staticmethod def m():` |
| property | `@property def x(self):` |
| dataclass | `@dataclass` over field annotations |

**Common built-in functions**

| Function | Purpose |
| :--- | :--- |
| `len`, `sum`, `min`, `max` | size and aggregates |
| `sorted`, `reversed` | new sorted / reversed iterable |
| `enumerate`, `zip` | index pairs / parallel iteration |
| `map`, `filter` | transform / select (lazy) |
| `any`, `all` | boolean reductions |
| `range` | lazy integer sequence |
| `type`, `isinstance` | runtime type inspection |
| `abs`, `round`, `pow`, `divmod` | numeric helpers |
| `open` | file access (use with `with`) |
| `print`, `input` | console I/O |

**Error handling**

```python
try:
    risky()
except SpecificError as e:
    handle(e)
else:
    ran_without_error()
finally:
    always_cleanup()
```

**Pip and environments**

| Task | Command |
| :--- | :--- |
| install a package | `pip install requests` |
| install exact version | `pip install "requests==2.32.3"` |
| list installed | `pip list` |
| freeze to file | `pip freeze > requirements.txt` |
| install from file | `pip install -r requirements.txt` |
| create venv | `python -m venv .venv` |
| activate (Unix) | `source .venv/bin/activate` |

---

*Further reading:*
- *[The Python Tutorial](https://docs.python.org/3/tutorial/) — the official introduction*
- *[The Python Language Reference](https://docs.python.org/3/reference/) — the formal specification*
- *[The Python Standard Library](https://docs.python.org/3/library/) — built-in types, functions, and modules*
- *[Built-in Types](https://docs.python.org/3/library/stdtypes.html) — the per-type method tables*
- *[PEP 8](https://peps.python.org/pep-0008/) — the style guide*
- *[What's New in Python 3.14](https://docs.python.org/3/whatsnew/3.14.html) — the latest release notes*
