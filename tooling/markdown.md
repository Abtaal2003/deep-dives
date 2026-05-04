# Markdown Guide

**Sources:**
- [markdowntutorial.com](https://www.markdowntutorial.com/) — interactive 10-minute tutorial (do this first on Saturday 1)
- [markdownguide.org](https://www.markdownguide.org/) — full reference (basic + extended syntax)
- [Definitive Guide to Markdown — Medium](https://medium.com/@tuenguyends/definitive-guide-to-markdown-b3e1d59b72b) — practical, data-science-focused context

---

## Table of Contents

**Foundations**
- [0. What Is Markdown?](#0-what-is-markdown)
- [0.1 Where Markdown Is Used](#01-where-markdown-is-used)
- [0.2 Markdown Flavors](#02-markdown-flavors)
- [0.3 How to Write and Preview Markdown](#03-how-to-write-and-preview-markdown)
- [0.4 Why Markdown (the full case)](#04-why-markdown-the-full-case)

**Core Syntax**
- [1. Headings](#1-headings)
- [2. Paragraphs and Line Breaks](#2-paragraphs-and-line-breaks)
- [3. Emphasis](#3-emphasis)
- [4. Lists](#4-lists)
- [5. Code](#5-code)
- [6. Links](#6-links)
- [7. Images](#7-images)
- [8. Blockquotes](#8-blockquotes)
- [9. Horizontal Rules](#9-horizontal-rules)

**Extended Syntax**
- [10. Tables](#10-tables-extended-syntax)
- [11. Strikethrough](#11-strikethrough-extended-syntax)
- [12. Footnotes](#12-footnotes-extended-syntax)
- [13. Heading IDs](#13-heading-ids-extended-syntax)
- [14. Escaping Characters](#14-escaping-characters)
- [15. HTML Inside Markdown](#15-html-inside-markdown)
- [16. LaTeX / Math (Jupyter-specific)](#16-latex-math-jupyter-specific)
- [17. GitHub-Specific Features](#17-github-specific-features)

**Plan-Specific Reference**
- [18. The Saturday Log Template](#18-the-saturday-log-template-in-markdown)
- [19. The Deep-Dive Template](#19-the-deep-dive-template-in-markdown)
- [20. Quick Reference Cheat Sheet](#20-quick-reference-cheat-sheet)
- [21. Common Mistakes to Avoid](#21-common-mistakes-to-avoid)
- [22. Where Markdown Is Used in This Plan](#22-where-markdown-is-used-in-this-plan-file-by-file)

---

## 0. What Is Markdown?

Markdown is a **lightweight markup language** created by John Gruber in 2004. A markup language is one where you annotate plain text with symbols to convey formatting — the symbols tell a renderer how the text should look when displayed. Markdown's design goal was to be readable *as-is* in raw form, so the symbols feel natural rather than cluttered.

When you write `**bold**`, that is valid readable text even before rendering. When a renderer (GitHub, VS Code, Jupyter, Obsidian) processes it, the asterisks disappear and the word appears **bold**. Under the hood, Markdown converts to HTML — `**bold**` becomes `<strong>bold</strong>`.

### The two-layer model

```
You write:    Plain text + Markdown symbols  →  .md file
Renderer:     .md file + parser               →  HTML / PDF / display
```

The `.md` file is just a text file. Any text editor can open it. The rendering only happens in applications that know how to parse Markdown.

### What problem does it solve?

Before Markdown, the two options for formatted writing were:

1. **Rich text editors** (Word, Google Docs) — formatting is invisible, stored in a binary format, hard to version-control with Git, and not portable across systems.
2. **Raw HTML** — valid but tedious to write for everyday documentation.

Markdown sits in the middle: human-readable plain text that renders beautifully and works perfectly with Git. For a developer or data scientist, this matters because your notes, READMEs, and documentation live in the same repos as your code, versioned and reviewable like everything else.

---

## 0.1 Where Markdown Is Used

Understanding *where* Markdown renders helps you know which features are available in any given context.

### GitHub
Every `.md` file in a GitHub repo is rendered automatically. The most important one is `README.md` in the root of a repo — GitHub displays it as the repo's front page. GitHub uses **GitHub Flavored Markdown (GFM)**, a superset of standard Markdown that adds task lists, tables, strikethrough, and auto-linked references to issues and PRs.

**In your plan:** Every project repo you build — `fastapi-service/`, `ml-from-scratch/`, `production-ml-system/`, `capstone/` — needs a professional `README.md`. This is the first thing a recruiter or hiring manager sees when they click your GitHub link. It is also where the CI green badge lives.

### Jupyter Notebooks
Jupyter has two cell types: **code cells** and **Markdown cells**. In a Markdown cell, you write Markdown and the notebook renders it inline — headings, lists, images, and LaTeX math all render between your code blocks. This is where you will write your ML/DL notes during Phases 2–3: theory explanation in Markdown, implementation in code, in the same file.

**In your plan:** Phases 2–3 are where Jupyter becomes your primary workspace. Your Andrew Ng ML Spec implementations, ML-from-scratch builds, and NLP Spec projects will all live in notebooks where Markdown and code coexist.

### VS Code
VS Code renders `.md` files in a split preview pane (open with `Ctrl+Shift+V` or `Cmd+Shift+V`). You write on the left, see the rendered output on the right. This is where you will write your Saturday logs and deep-dive notes.

### Obsidian
If you choose Obsidian as your notes editor (the plan lists it as an option), it renders Markdown live as you type and adds `[[wiki-style links]]` between notes. Your deep-dives can link to each other — for example, a note on gradient descent can link to your note on backpropagation.

### GitHub Pages (Portfolio site)
In Phase 4, your portfolio site is built on GitHub Pages. GitHub Pages can serve Markdown files directly via Jekyll, converting them to HTML pages automatically. Your project descriptions, blog posts, and portfolio index can all be written in Markdown.

### Other places you will encounter it
- `CLAUDE.md` (the context file Claude Code reads per repo — written in Markdown)
- `SKILL.md` and sub-agent config files in `.claude/` (also Markdown)
- FastAPI and Python docstrings (some tools render Markdown in API docs)
- PyPI package READMEs (rendered on the package page)
- Pull request descriptions and GitHub Issues (GFM rendered)
- LangChain, HuggingFace, and most ML library documentation (written in Markdown, rendered on their sites)

---

## 0.2 Markdown Flavors

Standard Markdown (the original 2004 spec) is the baseline, but most platforms use an extended version. The important ones for this plan:

| Flavor | Where used | Key additions |
| :--- | :--- | :--- |
| **Standard Markdown** | Baseline everywhere | All core syntax |
| **GitHub Flavored Markdown (GFM)** | GitHub repos, Issues, PRs | Task lists, tables, strikethrough, auto-links, @mentions, #references |
| **CommonMark** | Many modern parsers | Stricter, more consistent version of standard Markdown |
| **Jupyter Markdown** | Jupyter Notebooks | All GFM features + LaTeX math via MathJax |
| **Obsidian Markdown** | Obsidian app | GFM + `[[wikilinks]]`, callouts, graph view |

**Practical rule:** Write to GFM and you will be fine on GitHub, Jupyter, and VS Code — which covers 95% of your use cases in this plan. The extended syntax features (tables, task lists, fenced code blocks with language tags, strikethrough) are all part of GFM.

---

## 0.3 How to Write and Preview Markdown

### Option 1: VS Code (recommended for this plan)
- Open any `.md` file.
- Press `Ctrl+Shift+V` (Windows/Linux) or `Cmd+Shift+V` (Mac) to open the preview pane.
- Or press `Ctrl+K V` to open preview side-by-side.
- Install the **Markdown All in One** extension for shortcuts, auto-formatting, and a table of contents generator.

### Option 2: Obsidian
- Opens `.md` files natively and renders Markdown live as you type.
- Free for personal use.
- Use if you want cross-linking between your deep-dive notes via `[[wikilinks]]`.

### Option 3: GitHub web UI
- Edit `.md` files directly on GitHub.
- The editor has a "Preview" tab you can toggle.
- Maximum simplicity — no local setup needed.
- Slowest for writing large amounts of content.

### Jupyter Markdown cells
- Create a new cell, change its type to Markdown (dropdown in toolbar or press `M` in command mode).
- Press `Shift+Enter` to render the cell.
- Double-click a rendered cell to edit it again.

**Pick one editor and stick with it.** The plan says this explicitly. Consistency matters more than the tool. If you are already comfortable in VS Code from work, use VS Code.

---

## 0.4 Why Markdown (the full case)

**Portability.** A `.md` file is plain text. It opens in any editor on any OS without any special software. A `.docx` file requires Word or a compatible application. In five years, your Markdown notes will be just as readable as today.

**Git-friendly.** Because Markdown is plain text, Git can diff it line by line. If you update a README or a note, you can see exactly what changed in a pull request. Binary formats like `.docx` show as a blob — no meaningful diff.

**Focus on content.** Rich text editors constantly tempt you to adjust fonts, colors, and spacing. Markdown has none of those controls. You write, the renderer decides how it looks. This is actually a feature — it removes a category of distraction entirely.

**Developer standard.** GitHub READMEs, API documentation, technical blog posts, open source contribution guides, and most ML library docs are all written in Markdown. Learning it is not optional for a developer workflow — it is table stakes.

**Jupyter integration.** For data science and ML work, mixing executable code with formatted explanations in the same notebook is a superpower. When your data changes, you re-run the notebook and the results update automatically — no copying and pasting into Word. This is the workflow you will use heavily in Phases 2–3.

**Conversion.** Markdown can be converted to HTML, PDF, Word, slides, and more via tools like Pandoc. Your notes are not locked into a proprietary format.

---

## 1. Headings

```markdown
# H1 — Document title (one per file)
## H2 — Major section
### H3 — Subsection
#### H4 — Rarely needed
##### H5
###### H6
```

**Best practices:**
- Always put a blank line before and after a heading.
- Always put a space between `#` and the heading text. `#Heading` does not render correctly in all parsers.
- In practice, never go beyond `### H3`. Deeply nested headings make docs hard to follow. Keep structure flat.
- One `# H1` per file — that is your document title.

**Alternate syntax** (H1 and H2 only — avoid for consistency):
```markdown
Heading Level 1
===============

Heading Level 2
---------------
```

---

## 2. Paragraphs and Line Breaks

```markdown
This is paragraph one.

This is paragraph two. A blank line between them creates two separate paragraphs.
```

For a line break *within* a paragraph (without starting a new paragraph), end the line with **two trailing spaces** or use `<br>`:

```markdown
This is line one.  
This is line two (same paragraph, new line).

This is line one.<br>
This is line two.
```

**Do not** indent paragraphs with tabs or spaces — this can trigger unintended code block formatting.

---

## 3. Emphasis

```markdown
**bold text**          → bold (preferred)
__bold text__          → bold (avoid with underscores mid-word)

*italic text*          → italic (preferred)
_italic text_          → italic

***bold and italic***  → both
```

**Best practice:** Use `*asterisks*` not `_underscores_` when emphasising words mid-sentence. Many parsers handle underscores inconsistently inside words.

---

## 4. Lists

### Unordered
```markdown
- Item one
- Item two
- Item three
```

You can also use `*` or `+` as the bullet character, but **pick one and stick with it** across the whole file. Do not mix.

### Ordered
```markdown
1. First item
2. Second item
3. Third item
```

Tip: You can write `1.` for every item and the renderer auto-numbers them. This means inserting a new item mid-list does not require renumbering everything.

```markdown
1. First item
1. Second item
1. Third item
```

Both render identically. Always use `.` as the delimiter, not `)`.

### Nested lists
Indent with **4 spaces** (or 1 tab) per level:

```markdown
- Item one
    - Sub-item 1.1
    - Sub-item 1.2
- Item two
    - Sub-item 2.1
```

Mixed ordered/unordered nesting works:
```markdown
1. Step one
    - Note A
    - Note B
2. Step two
```

Keep nesting to **two levels maximum**. Deeper nesting is a sign the structure needs rethinking.

### Adding elements inside lists
Indent the element 4 spaces to keep it inside the list:

```markdown
- Item one

    This paragraph stays inside the list item.

- Item two

    > A blockquote inside a list item.

- Item three
```

### Task lists (GitHub Flavored Markdown)
Extremely useful for Saturday logs and project checklists:

```markdown
- [x] Complete Hour 1
- [x] Complete Hour 2
- [ ] Push commits to GitHub
- [ ] Write saturday-XX.md
```

Renders as interactive checkboxes on GitHub.

---

## 5. Code

### Inline code
Wrap in single backticks. Use for variable names, commands, function names, file paths:

```markdown
Run `pytest` before pushing. The function is `calculate_loss()`. The file is `config.py`.
```

If the code itself contains backticks, use double backticks:
```markdown
``Use `code` in your Markdown.``
```

### Fenced code blocks
Wrap in triple backticks. Always specify the language for syntax highlighting:

````markdown
```python
def train(model, data):
    loss = model.forward(data)
    return loss
```
````

````markdown
```bash
uv pip install fastapi
pytest tests/ -v
```
````

````markdown
```sql
SELECT user_id, COUNT(*) AS purchases
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```
````

````markdown
```json
{
  "model": "claude-sonnet-4-5",
  "max_tokens": 1000
}
```
````

Common language identifiers: `python`, `bash`, `sql`, `javascript`, `typescript`, `json`, `yaml`, `markdown`, `html`, `css`, `dockerfile`, `text`.

### Indented code blocks (older style — avoid)
Four spaces of indentation also creates a code block, but the fenced style above is preferred everywhere because it supports syntax highlighting.

---

## 6. Links

### Inline links
```markdown
[Link text](https://www.example.com)
[Link text](https://www.example.com "Optional hover tooltip")
```

### Auto-links
```markdown
<https://www.markdownguide.org>
<yourname@example.com>
```

### Reference-style links
Useful for long URLs that would clutter the reading flow. Define the reference anywhere in the file (often at the bottom):

```markdown
Check out the [Markdown Guide][1] for more details.

[1]: https://www.markdownguide.org "The Markdown Guide"
```

### Formatting links
```markdown
**[Bold link](https://example.com)**
*[Italic link](https://example.com)*
[`code link`](https://example.com)
```

### Anchor links (linking to headings)
GitHub auto-generates anchor IDs from headings. Link to a section within the same file:

```markdown
See the [Code section](#5-code) for examples.
```

Rules for the anchor ID:
- All lowercase
- Spaces become hyphens
- Special characters are removed
- Numbers and letters are kept

So `## 5. Code` becomes `#5-code`.

**Best practice:** URL-encode spaces as `%20` if your URL contains them: `[My Page](https://example.com/my%20page)`.

---

## 7. Images

```markdown
![Alt text](path/to/image.png)
![Alt text](https://example.com/image.png "Optional title")
```

Alt text is important — it describes the image for accessibility and shows when the image fails to load. Always write it.

### Linked image (click image to open link)
```markdown
[![Alt text](image.png)](https://example.com)
```

### Images in GitHub READMEs
For a project repo, store images in a `/docs/assets/` or `/assets/` folder and reference relatively:

```markdown
![Architecture diagram](assets/architecture.png)
```

---

## 8. Blockquotes

```markdown
> This is a blockquote.

> Multi-paragraph blockquote.
>
> Second paragraph — add `>` on the blank line too.

> Nested blockquote.
>
>> This is nested one level deeper.
```

Blockquotes can contain other Markdown elements:

```markdown
> ### Title inside a blockquote
>
> - List item
> - Another item
>
> **Bold text** inside a blockquote.
```

**Best practice:** Put blank lines before and after blockquotes for compatibility.

---

## 9. Horizontal Rules

Three or more dashes, asterisks, or underscores on their own line:

```markdown
---

***

___
```

All render identically. Use `---` by convention. Always put blank lines before and after, otherwise `---` directly under text renders it as an H2 heading.

---

## 10. Tables (Extended Syntax)

```markdown
| Column A | Column B | Column C |
| -------- | -------- | -------- |
| Cell     | Cell     | Cell     |
| Cell     | Cell     | Cell     |
```

### Column alignment
```markdown
| Left         |   Center    |          Right |
| :----------- |    :---:    |    ----------: |
| left text    | center text |     right text |
```

- `:---` = left align
- `:---:` = center align
- `---:` = right align

### Formatting inside tables
You can use inline code, bold, italic, and links inside cells. You cannot use headings, lists, code blocks, blockquotes, or images.

To display a pipe character `|` inside a cell, use `&#124;`.

**Tip:** Build tables visually at [tablesgenerator.com/markdown_tables](https://www.tablesgenerator.com/markdown_tables) instead of typing them by hand.

---

## 11. Strikethrough (Extended Syntax)

```markdown
~~This text is struck through.~~
```

Renders as: ~~This text is struck through.~~

---

## 12. Footnotes (Extended Syntax)

```markdown
This fact needs a citation.[^1] This one too.[^note]

[^1]: Source: Markdown Guide, 2026.
[^note]: This is a longer footnote with more context.
```

Footnotes render as superscript numbers in the text and collect at the bottom of the document. You do not need to place them at the end of the file — put them near the relevant paragraph for maintainability.

---

## 13. Heading IDs (Extended Syntax)

```markdown
### My Section {#custom-id}
```

Then link to it:
```markdown
[Jump to My Section](#custom-id)
```

GitHub generates heading IDs automatically from heading text (see Section 6 for the rules), so custom IDs are mainly useful when you need a stable ID that won't break if the heading text changes.

---

## 14. Escaping Characters

To display a Markdown symbol literally, prefix it with `\`:

```markdown
\*  \`  \_  \#  \[  \]  \(  \)  \{  \}  \+  \-  \.  \!  \|
```

Example — without escaping, asterisks trigger italic. With escaping, they appear literally:
```markdown
*This is italic*

\*This is not italic — the asterisks are visible\*
```

Rendered output of the second line: `*This is not italic — the asterisks are visible*` (literal asterisks, no formatting).

---

## 15. HTML Inside Markdown

Most Markdown renderers allow raw HTML. Useful for things Markdown cannot do natively:

```markdown
<br> — forced line break

<mark>Highlighted text</mark>

<sup>Superscript</sup>

<sub>Subscript</sub>

<details>
<summary>Click to expand</summary>
Hidden content here.
</details>
```

The `<details>` / `<summary>` collapsible block is especially useful in GitHub READMEs for keeping long content clean.

**Note:** You cannot use Markdown syntax inside block-level HTML tags. `<p>**bold**</p>` will not render the bold.

---

## 16. LaTeX / Math (Jupyter-specific)

In Jupyter Notebooks (not standard GitHub Markdown), you can render LaTeX math. This will be relevant during Phases 2–3 when you take notes on ML/DL math.

Inline math (wrap in single `$`):
```markdown
The loss function is $L = -\sum_{i} y_i \log(\hat{y}_i)$.
```

Block math (wrap in `$$`):
```markdown
$$
\hat{y} = \sigma(W^T x + b)
$$
```

```markdown
$$
\nabla_\theta J(\theta) = \frac{1}{m} X^T (X\theta - y)
$$
```

This renders in Jupyter but **not** by default in GitHub READMEs unless the repo has math rendering enabled. GitHub supports the same `$...$` inline and `$$...$$` block syntax natively in Markdown files — it just needs to be enabled in your repo settings or it renders automatically on github.com for `.md` files.

For GitHub math rendering (fenced block alternative, always works on GitHub):
````markdown
```math
E = mc^2
```
````

---

## 17. GitHub-Specific Features

GitHub uses **GitHub Flavored Markdown (GFM)**, which is a superset of standard Markdown.

### Task lists
```markdown
- [x] Done
- [ ] Not done
```

### Mentions and references
```markdown
@username          — mention a user
#123               — reference an issue or PR
owner/repo#123     — cross-repo reference
SHA                — reference a commit
```

### Automatic URL linking
GitHub automatically turns raw URLs into clickable links. No brackets needed.

### Syntax highlighting
GitHub supports syntax highlighting for all major languages in fenced code blocks (see Section 5).

### Collapsible sections
```markdown
<details>
<summary>Show more</summary>

Content here. Markdown works inside this block.

</details>
```

### Badges
Commonly used in READMEs to show build status, coverage, and Python version. Format:

```markdown
![CI](https://github.com/username/repo/actions/workflows/ci.yml/badge.svg)
![Coverage](https://codecov.io/gh/username/repo/branch/main/graph/badge.svg)
```

This is required for every project in this plan: "Every project ships with CI green badge."

---

## 18. The Saturday Log Template in Markdown

From the plan, every Saturday log uses this template. Written here in fully formatted Markdown so you can see the structure clearly:

```markdown
# Saturday X — YYYY-MM-DD

## What I covered
- Hour 1: [topic]
- Hour 2: [topic]
- Hour 3: [topic]
- Hour 4: [topic]

## What I learned (write from memory, no source access)

3–5 sentences max. Most important things from today.

## What was hard

1–2 things you struggled with.

## What I want to revisit next Saturday

Specific items. Becomes your retrieval prompt for Saturday X+1.

## Artifacts pushed

- [Link to repo + folder]
```

---

## 19. The Deep-Dive Template in Markdown

```markdown
# [Topic Name]

**Phase / Week:** Phase X, Saturday Y  
**Tags:** #python #fundamentals

## What is it?

One sentence.

## What problem does it solve?

One sentence.

## How does it work?

3–5 sentences at mechanism level. If visual, sketch by hand and add a photo below.

## When to use vs alternatives?

One paragraph. Compare to at least one alternative.

## Gotcha

One sentence on what trips people up.

## Source

[Link]
```

---

## 20. Quick Reference Cheat Sheet

| Element | Syntax |
| :--- | :--- |
| H1 | `# Heading` |
| H2 | `## Heading` |
| H3 | `### Heading` |
| Bold | `**text**` |
| Italic | `*text*` |
| Bold + Italic | `***text***` |
| Strikethrough | `~~text~~` |
| Inline code | `` `code` `` |
| Code block | ` ```python ... ``` ` |
| Unordered list | `- item` |
| Ordered list | `1. item` |
| Task list | `- [x] done` / `- [ ] todo` |
| Link | `[text](url)` |
| Image | `![alt](path)` |
| Blockquote | `> quote` |
| Horizontal rule | `---` |
| Table | `\| col \| col \|` + `\| --- \| --- \|` |
| Footnote | `[^1]` + `[^1]: text` |
| Escape character | `\*` |
| HTML line break | `<br>` |
| Collapsible | `<details><summary>...</summary>...</details>` |

---

## 21. Common Mistakes to Avoid

**Missing blank lines around block elements.** Headings, blockquotes, horizontal rules, and code blocks all need blank lines before and after them for reliable cross-parser rendering.

**Mixing bullet characters.** Pick `-` for unordered lists and stick with it throughout the file.

**Forgetting the space after `#`.** `#Heading` does not render in all parsers. Always write `# Heading`.

**Using `_underscores_` mid-word.** Use `*asterisks*` for all emphasis. Underscores inside words behave unpredictably across parsers.

**Indenting paragraphs with spaces.** Do not manually indent paragraph text. Only indent when creating nested list elements.

**Not specifying the language on code blocks.** Always write ` ```python `, ` ```bash `, etc. Unstyled code blocks miss syntax highlighting and look unprofessional.

**Forgetting alt text on images.** Always write descriptive alt text: `![Architecture diagram showing FastAPI + PostgreSQL](assets/arch.png)`.

---

## 22. Where Markdown Is Used in This Plan — File-by-File

| File / Location | Phase | Notes |
| :--- | :---: | :--- |
| `learning-notes/saturdays/saturday-XX.md` | All | Weekly log using the template in Section 18. One per Saturday, committed same day. |
| `deep-dives/**/*.md` | All | 5-field deep-dive per concept using the template in Section 19. |
| `projects/*/README.md` | All | Professional project README — description, setup instructions, CI badge, live URL. This is what recruiters see. |
| `projects/python-fundamentals/README.md` | 1 | First README you write. Keep it simple: what the scripts do, how to run them. |
| `projects/fastapi-service/README.md` | 1 | Must include CI green badge, live Render URL, and the Anthropic Claude integration description. |
| `projects/fastapi-service/CLAUDE.md` | 1 | Markdown file Claude Code reads for repo context. Written in plain Markdown paragraphs. |
| `projects/fastapi-service/.claude/skills/*.md` | 1 | `SKILL.md` files for custom Claude Code skills. Each is a Markdown doc with YAML frontmatter. |
| `projects/ml-from-scratch/README.md` | 2 | Documents each from-scratch implementation with comparison to sklearn baseline. |
| `projects/production-ml-system/README.md` | 3 | CV centrepiece. Most detailed README you write — architecture, MLflow tracking, eval gate, monitoring, demo. |
| Jupyter Notebook cells (`.ipynb`) | 2–3 | Markdown cells for theory notes, section headings, and math formulas (LaTeX). See Section 16. |
| `learning-notes/behavioral-stories.md` | 3–4 | STAR method stories in Markdown. One `##` heading per story. |
| `projects/capstone/README.md` | 4 | Final capstone README. Linked from your portfolio site. |
| Portfolio site (`GitHub Pages`) | 4 | Pages rendered from Markdown via Jekyll. Your project index and about page. |
| Pull request descriptions | All | Written in GFM on GitHub. Describe what changed and why. Markdown renders in the PR view. |

---

*This guide consolidates content from:*
- *[markdowntutorial.com](https://www.markdowntutorial.com/) — interactive tutorial*
- *[markdownguide.org](https://www.markdownguide.org/) — basic and extended syntax reference*
- *[Definitive Guide to Markdown — Tue Nguyen, Medium](https://medium.com/@tuenguyends/definitive-guide-to-markdown-b3e1d59b72b) — practical data science context*
