# LLM Wiki

A personal knowledge base maintained by an LLM, following the [Karpathy pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## Quick Start

```
raw/        # Drop sources here (articles, papers, notes)
wiki/       # LLM-generated knowledge base
├── index.md    # Master catalog
├── log.md      # Activity timeline
├── concepts/   # Ideas, patterns, techniques
├── summaries/  # Source summaries
└── synthesis/  # High-level connections
```

## Usage

1. **Add a source** → Drop a file into `raw/`
2. **Ask the agent** → "Ingest [filename] from raw/"
3. **Ask questions** → Agent searches wiki, synthesizes answers
4. **File answers** → Good answers become new wiki pages

## Schema

See [SCHEMA.md](./SCHEMA.md) for conventions and workflows.

## Contents

Currently tracking:
- 9 concept pages (Anzen, Jellyfin, N8N, OAuth, fleet management patterns)
- 5 project summaries (client-portal, Anzen Egress, PunClaw, N8N maintenance, Jellyfin)
- Cross-referenced and self-maintaining

---

Part of the [OpenClaw](https://github.com/openclaw/openclaw) hive.
