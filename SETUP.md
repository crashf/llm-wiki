# LLM Wiki — Setup & Usage Guide

## Project Location

```
/root/.openclaw/workspace/projects/llm-wiki/
```

## Structure

```
llm-wiki/
├── raw/                    # Drop sources here to ingest
│   ├── articles/
│   ├── papers/
│   └── notes/
├── wiki/                  # The LLM-maintained wiki
│   ├── index.md          # Master catalog
│   ├── log.md            # Activity timeline
│   ├── entities/         # Pages about specific things
│   ├── concepts/          # Pages about ideas/patterns
│   ├── summaries/        # Summary pages for sources
│   └── synthesis/        # High-level synthesis
├── .brv/                 # ByteRover context
├── SCHEMA.md             # My configuration (for AI agent)
├── SETUP.md              # This file
└── original-pattern.md   # Karpathy's original document
```

## How To Use

### 1. Add a Source to Ingest

Drop any document into `raw/`:
- Copy/paste article text
- Download a PDF
- Save a webpage as markdown
- Paste meeting notes

Then tell me: **"Ingest [filename] from raw/"**

I'll:
- Read the source
- Discuss key takeaways with you
- Create/Update wiki pages
- Update the index
- Log the activity

### 2. Ask a Question

Just ask! I'll search the wiki and synthesize an answer.

If the answer is valuable, I'll offer to save it back to the wiki as a new page.

### 3. Lint the Wiki

Tell me: **"Run lint on the wiki"**

I'll check for:
- Contradictions between pages
- Stale claims superseded by new sources
- Orphan pages with no links
- Missing cross-references
- Concepts without their own page

### 4. Browse the Wiki

Open the wiki folder in any markdown editor (Obsidian recommended for graph view), or just ask me to show you a page.

## Workflow Examples

### Ingest an Article

```
You: "Can you ingest this article about Kubernetes? [paste text]"
Me: Reads it, discusses key points, creates:
  - wiki/summaries/kubernetes-overview.md
  - wiki/concepts/kubernetes.md (or updates existing)
  - Updates wiki/index.md
  - Logs in wiki/log.md
```

### Ask and File

```
You: "What's our understanding of how to scale n8n workflows?"
Me: Searches wiki, synthesizes answer about scaling patterns,
    explains it to you.
Me: "Want me to save this as a synthesis page?"
You: "Yes"
Me: Creates wiki/synthesis/n8n-scaling-patterns.md
```

### Research Sprint

```
You: "I want to go deep on AI agents. Ingest these 5 articles."
Me: Ingest each one, create comprehensive AI agents concept page,
    update index, note connections to existing pages.
Me: "Done. Your wiki now has a structured AI agents section.
    I noticed it connects to your existing llm-wiki-pattern page
    — want me to explore that connection?"
```

## Tips

- **Be selective with sources** — quality over quantity. A few well-digested sources beat dozens of skimmed ones.
- **Review my work** — I maintain the wiki but you know what's important. Guide me on emphasis.
- **Let answers compound** — if you asked a good question, file the answer. Future you will thank present you.
- **Check the graph** — in Obsidian, the graph view shows you the shape of your knowledge.

## Integration with This Agent

This workspace agent (me, Adrianna) is configured with:
- `SCHEMA.md` — tells me how to maintain the wiki
- ByteRover context in `.brv/` — for project memory
- Direct file access to the wiki folder

Just talk to me about anything you want to learn or track, and I'll integrate it into the wiki.

## Next Steps

1. Start ingesting relevant sources
2. Build out concept pages in areas you care about
3. Use it as our shared knowledge base for projects
