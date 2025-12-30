# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AskLater** - "Save now. Ask Claude later."

An intelligent content aggregation system using Claude Code sub-agents and MCP tools. Content shared via email is automatically processed, summarized, and stored as searchable markdown for later querying.

## AskLater System

The `/asklater` command processes new emails and routes content through adaptive AI processing:

1. **Intake Phase**: Fetch emails from Gmail, detect content type, assess complexity
2. **Processing Phase**: Spawn sub-agents for each content type (YouTube, TikTok, Article, RSS)
3. **Commit Phase**: Generate markdown, push to GitHub, update indexes

### Running AskLater

```
/asklater                    # Process all new emails
/asklater --limit 5          # Process only 5 emails
/asklater --dry-run          # Preview without processing
/asklater --depth standard   # Force specific depth
```

### State Files

```
.asklater/
├── audit.md          # Complete processing audit trail
└── processing-state.md   # Current processing state
```

### Full Documentation

- [PRD-002: AskLater](docs/PRD-002-asklater.md) - Complete system architecture

## Environment Variables

Copy `.env.example` to `.env` and configure:

- `ANTHROPIC_API_KEY` - Claude API key
- `GOOGLE_AI_API_KEY` - Gemini API for video analysis
- `GITHUB_TOKEN` - For committing processed content
- `DUMPLING_AI_KEY` - TikTok transcript extraction (optional)

## Content Structure

```
content/
├── tiktok/           # TikTok video summaries
├── youtube/          # YouTube video summaries
├── articles/         # Article summaries
├── rss/              # RSS feed summaries
└── _index.md         # Auto-generated index
```

### Markdown Format

Each content file includes:
- YAML frontmatter with metadata (source, date, topics, tags)
- TLDR summary
- Key points
- Detailed summary (depth-dependent)
- Source information

## MCP Tools Available

The Docker MCP gateway provides these tools for content processing:

- **gmail-mcp**: Email fetching (listMessages, getMessage)
- **youtube_transcript**: Video transcripts and metadata
- **markdownify**: Web page to markdown conversion
- **apify-mcp-server**: TikTok scraping
- **github**: Repository operations (push_files, create_or_update_file)
- **playwright**: Browser automation for complex pages

## Adaptive Processing Depth

| Depth | Criteria | Output |
|-------|----------|--------|
| **Minimal** | <60s video, <500 words | TLDR + 3 points |
| **Standard** | 1-10min video, 500-2000 words | Full summary + entities |
| **Comprehensive** | >10min, >2000 words, technical | Deep analysis + WebSearch |
