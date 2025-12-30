# /asklater - Save now. Ask Claude later.

Process new emails sent to `asklater@upnorthdigital.ai` and route content through adaptive AI processing pipeline.

## Usage

```
/asklater [options]
```

**Options:**
- `--limit N` - Process only N most recent emails (default: 10)
- `--dry-run` - Show what would be processed without executing
- `--depth minimal|standard|comprehensive` - Force specific processing depth
- `--all` - Check all recent emails, not just those to the asklater alias

## Email Alias

By default, this command searches for emails sent to `to:asklater@upnorthdigital.ai`.
Simply forward or send any content you want to save to this address from any device.

## Safeguards

### Sender Whitelist
Only process emails from these authorized senders:
- `nolan.northup@sierra-cedar.com`
- `nolannorthup@gmail.com`
- `admin@upnorthdigital.ai`
- `nolannorthup@upnorthdigital.ai`

Emails from other senders will be skipped with a warning in the audit log.

### Duplicate Prevention
- Check `content/` for existing files with the same source URL
- Skip already-processed content to avoid duplicates

### Processing Limits
- Default limit: 10 emails per run
- Skip videos longer than 60 minutes
- Skip articles longer than 10,000 words

## How It Works

This command implements the AskLater Content Processing System (PRD-002):

### Phase 1: Intake (Assessment)
1. Fetch emails to asklater alias via Gmail OAuth MCP (`to:asklater@upnorthdigital.ai`)
2. Detect content type (TikTok, YouTube, Article, RSS)
3. Assess complexity to determine processing depth

### Phase 2: Processing (Sub-Agents)
4. Spawn appropriate processor agent based on content type
5. Extract content using MCP tools (youtube_transcript, markdownify, apify)
6. Generate markdown with depth-appropriate detail

### Phase 3: Commit (Operations)
7. Push markdown files to GitHub via `github` MCP
8. Update index files (_index.md, _recent.md)
9. Log processing to .asklater/audit.md

## Content Type Detection

| Pattern | Type | Processor |
|---------|------|-----------|
| `tiktok.com`, `vm.tiktok.com` | TikTok | TikTok Agent |
| `youtube.com`, `youtu.be` | YouTube | YouTube Agent |
| `google.com/alerts`, RSS keywords | RSS | RSS Agent |
| Any other URL | Article | Article Agent |

## Adaptive Depth

| Depth | Criteria | Output |
|-------|----------|--------|
| **Minimal** | <60s video, <500 words, entertainment | TLDR + 3 points |
| **Standard** | 1-10min video, 500-2000 words | Full summary + entities |
| **Comprehensive** | >10min video, >2000 words, technical | Deep analysis + WebSearch |

## MCP Tools Used

- `gmail-oauth`: search_emails, read_email (OAuth-based, full body access)
- `youtube_transcript`: get_transcript, get_video_info
- `markdownify`: webpage-to-markdown
- `apify-mcp-server`: call-actor (TikTok scrapers)
- `github`: push_files, create_or_update_file

## Example Output

```
AskLater processing 3 new emails...

[1/3] TikTok: "AI coding tips"
      Depth: Minimal (45s video)
      -> content/tiktok/2025/12/20251229-182301-ai-coding-tips.md

[2/3] YouTube: "Building LLM Agents"
      Depth: Comprehensive (45min, technical)
      -> content/youtube/2025/12/20251229-182315-building-llm-agents.md

[3/3] Article: "The State of AI 2025"
      Depth: Standard (1500 words)
      -> content/articles/2025/12/20251229-182330-state-of-ai-2025.md

Committed 3 files to GitHub.
Updated _index.md and _recent.md.
```

## Execution

When this command is invoked, Claude Code will:

1. **Search emails** using `to:asklater@upnorthdigital.ai newer_than:7d`
2. **For each email with URLs:**
   - Read full email body via Gmail OAuth MCP
   - Extract URLs from email content
   - Detect content type from URL patterns
   - Assess complexity based on content characteristics
   - Spawn Task agent for that content type
   - Process and generate markdown
3. **Batch commit** all processed files to GitHub
4. **Update indexes** and audit log

## Implementation Notes

This skill uses the Task tool to spawn sub-agents for parallel processing:

```
Task(subagent_type="general-purpose", prompt="Process YouTube: [url]...")
Task(subagent_type="general-purpose", prompt="Process Article: [url]...")
```

Each sub-agent has access to MCP tools and returns processed markdown.
