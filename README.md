# Job Search Engine

**Status:** 🚧 In Development

An AI-powered, multi-source job search engine that aggregates listings from multiple job boards and matches them against user profiles.

## Features (Planned)

- 📄 **Resume Parsing** — PDF, DOCX, LinkedIn profile extraction
- 🔍 **Multi-Source Search** — Indeed, JSearch, RSS feeds, company career pages
- 🎯 **Smart Matching** — AI-powered relevance scoring
- 📬 **Multiple Delivery Options** — Telegram, email, CSV export
- ⚙️ **Extensible Architecture** — Plugin system for sources and outputs

## Quick Start

*Coming soon*

## Documentation

- [Project Planning (PRJ-001)](docs/PRJ-001-planning.md)

## Architecture

```
Profile Engine → Search Engine → Scoring Engine → Delivery Engine
      ↓               ↓              ↓               ↓
  Resume/LinkedIn   APIs/RSS      Match Score    Telegram/Email
```

## Development

Built with:
- Node.js
- OpenClaw (AI agent framework)
- Various job board APIs

## License

MIT

---

*Part of the [KJS-Openclaw](https://github.com/KJS-Openclaw) organization*
