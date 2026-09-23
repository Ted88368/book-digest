# AGENTS.md — Book Digest & Knowledge Library Guide

Welcome to **book-digest**, a local AI-powered reading and knowledge repository built on top of [vitalysim/the-knowledge-guy](https://github.com/vitalysim/the-knowledge-guy).

---

## 📁 Repository Layout

```plaintext
book-digest/
├── raw/                  # Place raw .epub or .pdf books here
├── notes/                # General notes, summaries, and PKM markdown files
├── artifacts/            # Generated HTML visual cards, interactive courses, and synthesis reports
├── .claude/skills/       # Installed skills (and symlinked to skills/)
│   ├── book-to-skill/    # Ingest pipeline (PDF/EPUB → 2-Tier Skill)
│   ├── the-knowledge-guy/# Router & interactive tutor across all bookshelf skills
│   └── <book-skill>/     # Generated two-tier skill for each ingested book
├── scripts/              # Helper scripts and benchmarks
├── README.md             # Human documentation
└── AGENTS.md             # This instructions file for AI Agents
```

---

## 🛠️ Workflows for AI Agents

### 1. Ingesting a Book (`/book-to-skill`)
When a user places a book in `raw/` (e.g. `raw/deep_work.pdf`) and requests to digest it:
1. Run the **`book-to-skill`** pipeline:
   - Command: `/book-to-skill raw/deep_work.pdf`
   - Stage 0 executes `.claude/skills/book-to-skill/.venv/bin/python .claude/skills/book-to-skill/scripts/extract.py`.
   - Generates two tiers:
     - **Tier 1 (`SKILL.md`)**: Always-loaded ~3k token concept map, load-bearing frameworks, chapter index.
     - **Tier 2 (`chapters/*.md`)**: On-demand detailed chapter toolkits.

### 2. Querying & Tutoring (`/the-knowledge-guy`)
Once books are ingested as skills under `.claude/skills/`:
- **Cross-Book Query**: `/the-knowledge-guy what do my books say about <topic>?`
- **Interactive Tutor**: `/the-knowledge-guy walk me through <topic>`
- **Interactive Web Course**: `/the-knowledge-guy course <book-slug>` (Renders interactive HTML into `artifacts/`)
- **Nutshell Summary**: `/the-knowledge-guy nutshell <book-slug>`
- **Concept Comparison**: `/the-knowledge-guy compare <topic>`

---

## ⚙️ Execution Environment

- **Python Virtualenv**: `.claude/skills/book-to-skill/.venv/bin/python`
  Contains `pymupdf`, `ebooklib`, `beautifulsoup4`, `pypdf`.
- **Plumbing in Python, Intelligence in AI**: Python handles document extraction and slicing; AI handles semantic synthesis and framework mapping.
