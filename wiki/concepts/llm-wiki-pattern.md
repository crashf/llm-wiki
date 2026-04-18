---
title: LLM Wiki Pattern
type: concept
tags: [knowledge-management, llms, automation, personal-knowledge-base]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

A pattern where LLMs maintain a persistent, compounding wiki of markdown files that synthesizes your sources. Unlike RAG (retrieve at query time), knowledge is compiled once and kept current — cross-references exist, contradictions are flagged, and synthesis is already done.

## Key Points

- **Three layers:** Raw sources (immutable) → Wiki (LLM-owned markdown) → Schema (config for the LLM)
- **Compounding artifact:** The wiki gets richer with every source and question — knowledge persists across sessions
- **LLM does bookkeeping:** Summarizing, cross-referencing, filing, updating — the tedious work humans abandon
- **Human does curation:** Sourcing, exploring, asking questions, thinking about meaning
- **Special files:** index.md (catalog) and log.md (timeline) help navigation
- **Vaneva Bush's Memex:** Related vision from 1945 — personal curated knowledge with associative trails

## How It Works

1. **Ingest:** Drop source → LLM reads → creates/updates wiki pages → updates index → logs entry
2. **Query:** LLM searches wiki → synthesizes answer → can file answer back as new wiki page
3. **Lint:** Health-check for contradictions, orphans, stale claims, missing links

## Tools Mentioned

- **Obsidian** — IDE for browsing wiki (graph view, plugins)
- **qmd** — local search engine for markdown (BM25 + vector + LLM reranking)
- **Marp** — markdown slide decks
- **Dataview** — query pages via YAML frontmatter

## Connections

- Related to [[rag-vs-memory]] — contrasts with retrieval-at-query-time approach
- Compare to [[note-taking-systems]] — traditional vs LLM-maintained wikis

## Details

Most people's LLM + document experience is RAG: upload files, retrieve chunks at query time, generate answer. The LLM rediscovers knowledge from scratch every time. Nothing accumulates.

LLM Wiki is different. The LLM incrementally builds structured markdown files between you and the raw sources. When you add a source, the LLM integrates it — updates entity pages, revises topic summaries, notes contradictions, strengthens or challenges the synthesis.

The key insight: **good answers should be filed back into the wiki as new pages.** Your explorations compound in the knowledge base just like ingested sources do.

> "The human's job is to curate sources, direct the analysis, ask good questions, and think about what it all means. The LLM's job is everything else." — adapted from Karpathy

## Source

- [Karpathy's LLM Wiki Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
