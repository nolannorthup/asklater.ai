# /asklater - Save now. Ask Claude later.

Process new emails and route content through adaptive AI processing pipeline with security controls.

## Usage

```
/asklater [options]
```

**Options:**
- `--limit N` - Process only N most recent emails (default from config)
- `--dry-run` - Show what would be processed without executing
- `--execute` - Actually process and commit (required if dry_run_default is true)
- `--depth minimal|standard|comprehensive` - Force specific processing depth
- `--all` - Check all recent emails, not just those to the asklater alias

## Configuration

All settings are in `.asklater/config.yaml`. Copy from `config.example.yaml` to get started.

```yaml
email:
  alias: asklater@yourdomain.com
  whitelist:
    - your-email@example.com
  check_interval_days: 7

limits:
  emails_per_run: 10
  max_video_minutes: 60
  max_article_words: 10000

security:
  require_https: true
  block_attachments: true
```

See `config.example.yaml` for full documentation of all options.

## Security Controls

### Sender Whitelist
Only process emails from addresses listed in `config.yaml → email.whitelist`.
Emails from unknown senders are logged and skipped.

### URL Validation
- **HTTPS Required**: HTTP links are blocked (configurable)
- **Domain Blocklist**: URL shorteners blocked by default (bit.ly, t.co, etc.)
- **File Type Blocklist**: Executable files skipped (.exe, .zip, .sh, etc.)
- **Query String Limit**: Overly long query strings rejected

### Rate Limiting
- **Per-run limit**: Configurable max emails per `/asklater` invocation
- **Daily limit**: Max emails processed per day
- **Cooldown**: Minimum time between runs

### Duplicate Prevention
Processed URLs tracked in `.asklater/processed-urls.txt` to prevent reprocessing.

### Attachment Blocking
Email attachments are never processed (security risk).

## How It Works

### Phase 1: Intake (Assessment)
1. Load config from `.asklater/config.yaml`
2. Fetch emails via Gmail OAuth MCP using configured alias
3. Validate sender against whitelist
4. Validate URLs against security rules
5. Check for duplicates in `processed-urls.txt`
6. Detect content type and assess complexity

### Phase 2: Processing (Sub-Agents)
7. Spawn appropriate processor agent based on content type
8. Extract content using MCP tools
9. Generate markdown with depth-appropriate detail

### Phase 3: Commit (Operations)
10. Push markdown files to GitHub
11. Update `processed-urls.txt` with new entries
12. Update index files (_index.md)
13. Log all activity to `.asklater/audit.md`

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

Depth can be overridden per-domain in `config.yaml → depth_overrides`.

## MCP Tools Used

- `gmail-oauth`: search_emails, read_email (OAuth-based, full body access)
- `youtube_transcript`: get_transcript, get_video_info
- `markdownify`: webpage-to-markdown
- `apify-mcp-server`: call-actor (TikTok scrapers)
- `github`: push_files, create_or_update_file

## Example Output

```
Loading config from .asklater/config.yaml...

Security checks:
  ✓ Sender whitelist: 4 addresses
  ✓ Blocked domains: 8 patterns
  ✓ HTTPS required: enabled
  ✓ Attachments: blocked

Searching for emails to asklater@upnorthdigital.ai (last 7 days)...
Found 3 emails to process.

[1/3] From: nolannorthup@gmail.com
      URL: https://youtube.com/watch?v=abc123
      Type: YouTube
      ✓ Sender whitelisted
      ✓ URL validated
      ✓ Not a duplicate
      Depth: Comprehensive (45min, technical)
      -> content/youtube/2025/12/building-llm-agents.md

[2/3] From: admin@upnorthdigital.ai
      URL: http://example.com/article
      ✗ SKIPPED: HTTP not allowed (HTTPS required)
      Logged to audit.md

[3/3] From: spam@unknown.com
      ✗ SKIPPED: Sender not in whitelist
      Logged to audit.md

Processed: 1 | Skipped: 2
Committed 1 file to GitHub.
Updated processed-urls.txt and _index.md.
```

## State Files

```
.asklater/
├── config.yaml         # Your private configuration
├── config.example.yaml # Template (committed to git)
├── processed-urls.txt  # Duplicate tracking
├── audit.md            # Processing log
└── processing-state.md # Current run state
```

## Implementation Notes

This skill uses the Task tool to spawn sub-agents for parallel processing:

```
Task(subagent_type="general-purpose", prompt="Process YouTube: [url]...")
Task(subagent_type="general-purpose", prompt="Process Article: [url]...")
```

Each sub-agent has access to MCP tools and returns processed markdown.
