# LLM Wiki — Personal Knowledge Base Pattern

**Source:** https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Core Idea

Instead of RAG (retrieval at query time), the LLM incrementally builds a **persistent wiki** — structured markdown files that synthesize your sources. Knowledge is compiled once and kept current, not re-derived on every query.

**Key difference:** The wiki is a compounding artifact. Cross-references exist. Contradictions are flagged. Synthesis is already done.

## Three Layers

1. **Raw Sources** (`/raw/`) — immutable source documents (articles, papers, PDFs). Your source of truth.
2. **Wiki** (`/wiki/`) — LLM-generated markdown (summaries, entity pages, concept pages, synthesis). LLM owns this layer entirely.
3. **Schema** — A config file (CLAUDE.md / AGENTS.md) telling the LLM how to structure, ingest, and maintain the wiki.

## Workflows

- **Ingest:** Drop source → LLM reads → creates/updates wiki pages → updates index → logs entry
- **Query:** Ask question → LLM searches wiki → synthesizes answer → answer can be filed back as new wiki page
- **Lint:** Health-check the wiki — find contradictions, orphans, stale claims, missing links

## Special Files

- **index.md** — catalog of every wiki page with one-line summaries (replaces RAG at small scale)
- **log.md** — append-only timeline of ingests, queries, lint passes

## Tools Mentioned

- **Obsidian** — IDE for browsing the wiki (graph view, plugins)
- **qmd** — local search engine for markdown (BM25 + vector + LLM reranking)
- **Obsidian Web Clipper** — browser extension to save articles as markdown
- **Marp** — markdown slide decks
- **Dataview** — query pages via YAML frontmatter

## For Our Implementation

See `SETUP.md` for full setup guide and `SCHEMA.md` for the LLM configuration.
