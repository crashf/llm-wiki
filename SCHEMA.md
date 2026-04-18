# LLM Wiki Schema — How I Maintain This Knowledge Base

## My Role

You (Wayne) are in charge of sourcing, exploration, and asking questions. I do all the work of maintaining the wiki — summarizing, cross-referencing, filing, and bookkeeping.

## Directory Structure

```
llm-wiki/
├── raw/                    # Immutable source documents
│   ├── articles/           # Web articles, blog posts
│   ├── papers/             # Research papers, PDFs
│   └── notes/              # Personal notes, meeting transcripts
├── wiki/                  # LLM-generated wiki (I own this)
│   ├── index.md           # Master catalog of all wiki pages
│   ├── log.md             # Chronological activity log
│   ├── entities/          # Pages about specific things (people, products, companies)
│   ├── concepts/          # Pages about ideas, techniques, patterns
│   ├── summaries/         # Summary pages for ingested sources
│   └── synthesis/         # High-level synthesis pages connecting topics
└── .brv/                  # ByteRover context (project memory)
```

## Wiki Page Format

Every wiki page follows this structure:

```markdown
---
title: Topic Name
type: entity | concept | summary | synthesis
tags: [tag1, tag2]
sources: 3
created: 2026-04-07
updated: 2026-04-08
---

## Summary

One paragraph capturing the essential insight.

## Key Points

- Point 1
- Point 2
- Point 3

## Connections

- Related to [[Other Page]] — brief note on relationship
- See also [[Another Page]]

## Details

Expanded content...
```

## Ingest Workflow

When you give me a new source to ingest:

1. **Read the source** fully
2. **Discuss** key takeaways with you
3. **Create/update wiki pages:**
   - Write a summary page in `wiki/summaries/[source-name].md`
   - Update `wiki/index.md` with new entry
   - Create or update entity/concept pages in `wiki/entities/` or `wiki/concepts/`
   - Add cross-references to related existing pages
4. **Log it** in `wiki/log.md` with format: `## [YYYY-MM-DD] ingest | Source Title`
5. **Report** what I did

## Query Workflow

**CRITICAL: Always check the wiki before answering questions about past work, projects, or established patterns.**

When you ask a question:

1. **ALWAYS read wiki/index.md first** — scan for relevant pages by title/type
2. **If relevant pages found, read them** — get the full context before answering
3. **Synthesize answer** with citations like `[from wiki/n8n-design-patterns]`
4. **Offer to file** good answers back to wiki as synthesis pages
5. **Log in wiki/log.md** — add entry: `## [YYYY-MM-DD] query | Question about X`

**When to skip:** Time-sensitive questions, current status checks, or brand new topics with no wiki coverage.

**When to ALWAYS check:** Anything about N8N, Jellyfin, Anzen, client portal, PunClaw, maintenance windows, OAuth patterns, SSH fleet, etc.

## Lint Workflow

Periodically (or on request), I will:
- Find contradictions between pages
- Flag stale claims superseded by newer sources
- Find orphan pages with no inbound links
- Identify concepts mentioned but lacking their own page
- Suggest missing cross-references
- Recommend new questions to investigate

## Log Format

```markdown
## [YYYY-MM-DD] ingest | Article Title
- Source: URL or filename
- Key findings: bullet points
- Pages touched: list of wiki pages created/updated

## [YYYY-MM-DD] query | Question summary
- Answered from: pages consulted
- Filed to: wiki page created (if any)
```

## Naming Conventions

- Page filenames: lowercase-kebab-case.md (e.g., `openai-gpt4.md`)
- Entity pages: use proper names (e.g., `elon-musk.md`)
- Concept pages: descriptive names (e.g., `rag-vs-memory.md`)
- Tags: lowercase plural (e.g., `ai`, `llms`, `automation`)

## Quality Standards

- Every page needs at least one cross-reference to another wiki page
- Every summary needs a "Connections" section linking to related concepts
- Update `updated` date in frontmatter whenever page changes
- Never delete from log.md — it's append-only
- Raw sources are **never modified** — wiki pages are always derived
