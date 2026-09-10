# minibrainstarterkit
Mini project repository brain generator
# Brain Lite — Project Wiki Powered by Claude Code

A lightweight, self-hostable project knowledge base that turns your existing Office and PDF
documents into a searchable wiki that Claude can query to answer questions and generate
deliverables — with no ongoing token cost for indexing.

---

## How it works

```
Project Documents/        ← you drop your .pptx .docx .xlsx .pdf files here
        │
        ▼
distill_and_index.exe     ← extracts text from every file (zero LLM, pure Python)
        │
        ▼
Claude/Memory/twins/      ← one markdown "twin" per source file
Claude/Memory/index.md    ← searchable table: one row per file
        │
        ▼
Claude Code               ← reads the index and twins to answer questions,
                             find prior examples, and draft new deliverables
```

Distillation is done entirely by the exe — no API calls, no tokens consumed. Claude only
reads the pre-built twins at query time. Re-run the exe whenever files are added or changed;
it skips files that are already current.

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **Claude Code** | The Anthropic CLI / desktop app, opened in this project folder |
| **Windows** | The exe is built for Windows x64; Mac/Linux users can run `distill_and_index.py` directly (requires Python 3.10+) |

No Python installation required for Windows users — the exe bundles everything.

---

## Setup

### 1. Copy the kit

Copy this folder to your project location (SharePoint, OneDrive, local drive — anywhere
Claude Code can open it).

### 2. Add your documents

Drop your source files into `Project Documents/`. Any subfolder structure is fine:

```
Project Documents/
  Planning/
    Sprint Plan.pptx
    Resource Model.xlsx
  Discovery/
    Current State Assessment.pdf
  Delivery/
    ...
```

### 3. Open in Claude Code

Open the **root folder** (the one containing `CLAUDE.md` and `Project Documents/`) in
Claude Code. On session start, Claude automatically runs the exe to index everything —
no manual steps needed.

That's it. The brain is live.

---

## Folder structure

```
ProjectBrain/
  Project Documents/          ← your source files (any subfolder structure)
  Claude/
    distill_and_index.exe     ← the indexing engine
    Memory/
      index.md                ← auto-generated searchable index
      twins/                  ← auto-generated markdown twins (one per source file)
      .last-distill-report.txt ← run summary from the last exe invocation
  CLAUDE.md                   ← Claude's instruction set (must stay at root)
  .claude/
    settings.json             ← denies web access; pre-approves the exe
```

> **Do not move `CLAUDE.md` or `.claude/` into a subfolder.** Claude Code looks for them
> in the folder it is opened from. Moving them breaks auto-read.

---

## Updating the brain

Whenever you add, change, or delete source files:

```
Claude/distill_and_index.exe
```

Or ask Claude to do it — the boot sequence runs it automatically at session start anyway.

Run with `--dry-run` to preview what would change without writing anything:

```
Claude/distill_and_index.exe --dry-run
```

Run with `--force` to also refresh twins that were hand-edited (the fresh extract is
appended; existing hand-written content is never removed):

```
Claude/distill_and_index.exe --force
```

---

## What Claude can do with the brain

**Answer questions about your project**
> *"What integration approach did we use for the data migration?"*
> *"Which documents cover the test strategy?"*

**Find prior examples**
> *"Find the best sprint plan we have and summarise the sprint structure."*

**Generate new deliverables**
> *"Draft a change impact assessment for [CLIENT] using our best prior examples."*
> Claude finds the most relevant twins, uses them as scaffolding, flags assumptions with
> `[VALIDATE]`, and prepends a review disclaimer before presenting any output.

---

## Supported file formats

| Format | Notes |
|---|---|
| `.pptx` | Slide text, titles, tables, speaker notes |
| `.docx` | Paragraphs, headings, tables |
| `.xlsx` | All sheets, up to 500 non-empty rows per sheet |
| `.pdf` | Text-based PDFs; scanned/image PDFs produce a stub |
| `.vsdx` | Visio (optional; not included in the default exe build) |

---

## Confidentiality

`settings.json` blocks `WebFetch` and `WebSearch` for any Claude Code session opened in
this folder. Claude cannot send content to external URLs or search the web. Suitable for
confidential project material.

---

## For kit distributors — building the exe

The exe must be built once by whoever distributes the kit. Recipients do not need Python.

```bash
pip install pyinstaller python-pptx python-docx openpyxl pypdf
pyinstaller --onefile distill_and_index.py
```

Copy `dist/distill_and_index.exe` into the `Claude/` folder, then zip the kit for distribution.

> **Corporate AV note:** PyInstaller executables are occasionally flagged as suspicious by
> Windows Defender or endpoint AV (false positive). Test on a representative machine before
> distributing. If AV is a persistent blocker, users can run `distill_and_index.py` directly
> with Python 3.10+ installed instead.

---

## Customising for your project

Open `distill_and_index.py` and edit the config block near the top:

```python
# Keyword → tag mapping for auto-tagging. Add terms relevant to your project.
KEYWORD_TAGS: dict[str, str] = {
    "integration":  "integration",
    "governance":   "governance",
    "migration":    "migration",
    # add your own...
}
```

Tags appear in `Claude/Memory/index.md` and in each twin's frontmatter, making keyword
searches more reliable. Then rebuild the exe.

---

## How twins are structured

Each twin is a markdown file with:

- **Frontmatter** — source path, distillation date, mtime stamp, auto-generated summary and tags
- **AUTO-EXTRACT block** — faithful mechanical extraction of the source file's text
- **Human layer** (optional) — any notes, annotations, or cross-references added outside the
  AUTO-EXTRACT block are preserved across re-runs

The exe never overwrites content outside the AUTO-EXTRACT block.
