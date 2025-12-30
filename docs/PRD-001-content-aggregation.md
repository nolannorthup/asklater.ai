# PRD-001: Content Aggregation System

**Status:** Approved
**Created:** 2025-12-29
**Author:** Nolan / Claude Code
**Target Release:** 2025-01
**Related Issues:** N/A

---

## 1. Overview

### Problem Statement

Content is consumed from multiple sources (TikTok, YouTube, articles, RSS feeds, Google Alerts) but there's no centralized system to capture, process, and organize this information for future reference and AI-assisted discussion.

Currently:
- Interesting content is lost or forgotten after viewing
- No way to search or reference past content
- Content from different sources is siloed
- No AI-enhanced summaries or context augmentation

### Proposed Solution

Build a **hybrid MCP + n8n** content aggregation system that monitors a Gmail inbox for shared content, automatically detects content type, routes to appropriate AI processors, and stores AI-friendly markdown files in a Git repository for future Claude Code conversations.

**Architecture Options:**

1. **Pure MCP (Recommended)**: Claude Code orchestrates everything via MCP tools - no n8n required
2. **Hybrid**: n8n handles triggers/scheduling, Claude Code handles processing via MCP
3. **Pure n8n**: Original approach with n8n workflows (fallback option)

### Success Metrics

- [ ] Content shared via email is automatically processed within 5 minutes
- [ ] Markdown files are queryable and discussable with Claude Code
- [ ] System handles TikTok, YouTube, articles, and RSS feeds
- [ ] Web search augmentation provides additional context
- [ ] All content is indexed and searchable

---

## 2. User Stories

### Primary User Story

**As a** knowledge worker
**I want to** share interesting content to an email address and have it automatically processed and stored
**So that** I can easily reference, search, and discuss the content later with AI assistance

**Acceptance Criteria:**
- [ ] Sharing a TikTok video via email results in a processed markdown file
- [ ] Sharing an article URL results in a summarized markdown file
- [ ] All markdown files have consistent frontmatter for indexing
- [ ] Claude Code can read and discuss any captured content

### Secondary User Stories

**As a** researcher
**I want to** have web search augmentation added to captured content
**So that** I get additional context without manual research

**As a** content curator
**I want to** RSS feeds automatically captured and summarized
**So that** I don't miss important updates from my subscribed sources

---

## 3. Requirements

### Functional Requirements

1. **Gmail Monitoring**: Poll configured Gmail account for new emails every 5 minutes
2. **Content Type Detection**: Automatically detect TikTok, YouTube, article, or RSS content
3. **AI Processing**: Route text content to Claude, video content to Gemini
4. **Markdown Generation**: Create structured markdown with frontmatter
5. **Web Search Augmentation**: Enhance content with additional context
6. **Git Storage**: Commit processed files to repository
7. **Index Generation**: Maintain searchable index of all content

### Non-Functional Requirements

- **Performance**: Content processed within 5 minutes of email receipt
- **Reliability**: Failed processing should not block other content
- **Maintainability**: Modular workflows for easy updates
- **Cost**: Use cost-effective AI models (Claude Sonnet, Gemini Pro)

---

## 4. Technical Approach

### Architecture Option 1: Pure MCP (Recommended)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Claude Code Orchestration                     │
│                                                                  │
│  User Command: "process new emails" or scheduled via cron       │
│                                                                  │
│  ┌──────────────┐    ┌─────────────────┐    ┌───────────────┐  │
│  │  gmail-mcp   │───▶│  Content Type   │───▶│  Processors   │  │
│  │ listMessages │    │  Detection      │    │               │  │
│  │ findMessage  │    │  (Claude Code)  │    │  YouTube:     │  │
│  └──────────────┘    └─────────────────┘    │  └─youtube_   │  │
│                                              │    transcript │  │
│                                              │               │  │
│                                              │  TikTok:      │  │
│                                              │  └─apify-mcp  │  │
│                                              │               │  │
│                                              │  Articles:    │  │
│                                              │  └─browser/   │  │
│                                              │    WebFetch   │  │
│                                              └───────────────┘  │
│                                                      │          │
│                              ┌───────────────────────┘          │
│                              ▼                                   │
│                    ┌─────────────────┐                          │
│                    │  AI Processing  │                          │
│                    │  (Claude Code)  │                          │
│                    │  + WebSearch    │                          │
│                    └────────┬────────┘                          │
│                             │                                    │
│                             ▼                                    │
│                    ┌─────────────────┐                          │
│                    │   github MCP    │                          │
│                    │  push_files     │                          │
│                    └─────────────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

### MCP Tools Available

| Capability | MCP Server | Tools | Status |
|------------|------------|-------|--------|
| Email Monitoring | `gmail-mcp` | listMessages, findMessage | ✅ Working |
| YouTube Transcripts | `youtube_transcript` | get_transcript, get_video_info | ✅ Working |
| TikTok Scraping | `apify-mcp-server` | call-actor (TikTok scrapers) | ✅ Configured |
| Article Extraction | `browser` (Playwright) | navigate, snapshot | ✅ Built-in |
| Web Search | Claude Code | WebSearch | ✅ Built-in |
| GitHub Commits | `github` | push_files, create_or_update_file | ✅ Working |
| AI Processing | Claude Code | Native capability | ✅ Built-in |

### Architecture Option 2: n8n-Based (Original)

```
                    +------------------+
                    |  Gmail Trigger   |
                    | (5 min polling)  |
                    +--------+---------+
                             |
                             v
                    +------------------+
                    | Content Router   |
                    | (Switch Node)    |
                    +--------+---------+
                             |
         +-------------------+-------------------+
         |           |           |               |
         v           v           v               v
    +--------+  +--------+  +---------+    +--------+
    | TikTok |  |YouTube |  | Article |    |  RSS   |
    |Processor| |Processor| |Processor|    |Processor|
    +----+---+  +----+---+  +----+----+    +----+---+
         |           |           |              |
         v           v           v              v
    +--------+  +--------+  +---------+    +--------+
    | Gemini |  | Gemini |  | Claude  |    | Claude |
    +----+---+  +----+---+  +----+----+    +----+---+
         |           |           |              |
         +-------------------+-------------------+
                             |
                             v
                    +------------------+
                    | File Commit      |
                    | Handler          |
                    +------------------+
                             |
                             v
                    +------------------+
                    | Git Repository   |
                    | (content/)       |
                    +------------------+
```

### Components Affected

- [ ] **n8n Docker Container** - Add volume mount for repository
- [ ] **Repository Structure** - Add content/, workflows/, docs/ folders
- [ ] **.env** - Add API keys for Anthropic, Google AI, GitHub
- [ ] **CLAUDE.md** - Add content system documentation

### Technology Stack

#### Pure MCP Approach (Recommended)

| Component | Technology | Notes |
|-----------|------------|-------|
| Orchestration | Claude Code | Direct execution or cron-scheduled |
| Email Access | gmail-mcp | IMAP via app password |
| YouTube | youtube_transcript MCP | Free, no API key needed |
| TikTok | apify-mcp-server | ~$1/1000 videos via Apify actors |
| Articles | Playwright browser MCP | Built-in to Docker MCP toolkit |
| AI Processing | Claude Code (native) | No external API calls needed |
| Web Augmentation | WebSearch (built-in) | Included with Claude Code |
| Git Storage | github MCP | Direct commits via API |

#### n8n Approach (Alternative)

- **Workflow Engine**: n8n (Docker)
- **Text AI**: Anthropic Claude (claude-sonnet-4)
- **Video AI**: Google Gemini (gemini-2.5-pro)
- **Content Extraction**: HTTP Request + HTML Extract nodes
- **TikTok Transcripts**: Dumpling AI (optional)
- **Storage**: Git repository with markdown files

### Architecture Comparison

| Factor | Pure MCP | n8n | Hybrid |
|--------|----------|-----|--------|
| **Setup Complexity** | Low | Medium | Medium |
| **Scheduling** | Manual/Cron | Built-in | n8n triggers |
| **AI Processing** | Claude (native) | External API calls | Claude (native) |
| **Cost** | Lower (no API calls) | Higher (API usage) | Medium |
| **Debugging** | Claude conversation | n8n UI | Both |
| **Flexibility** | High | Medium | High |
| **Maintenance** | Low | Medium | Medium |

### Data Model

#### Markdown Frontmatter Schema

```yaml
---
id: string                    # Unique identifier (content-YYYYMMDD-HHMMSS-hash)
title: string                 # Extracted or generated title
source_type: enum             # tiktok | youtube | article | rss
source_url: string            # Original URL
source_name: string           # Creator/publication name
captured_at: datetime         # When email was received
processed_at: datetime        # When AI processing completed
ai_processor: string          # Model used (gemini-2.5-pro | claude-sonnet)
content_type: enum            # video | article | feed_item
topics: string[]              # AI-extracted topics
tags: string[]                # AI-generated tags
entities:
  people: string[]            # Named people
  companies: string[]         # Named companies
  technologies: string[]      # Named technologies
quality_score: number         # AI-assessed relevance (1-10)
word_count: number            # Summary word count
media_duration: string        # For video (HH:MM:SS)
email_subject: string         # Original email subject
email_from: string            # Sender email
email_received: datetime      # Email timestamp
---
```

#### Markdown Body Structure

```markdown
## TLDR
[1-2 sentence summary]

## Key Points
- Point 1
- Point 2
- Point 3

## Summary
[3-5 paragraph detailed summary]

## Notable Quotes
> "Quote from content"
> -- Attribution

## Context (Web Search Augmented)
[Additional context from web search]

## Topics for Exploration
- [ ] Follow-up question 1
- [ ] Related topic to investigate

## Raw Content
<details>
<summary>Original transcript/text</summary>
[Full original content]
</details>

## Processing Notes
- Processed by: [model]
- Confidence: [High/Medium/Low]
```

### Folder Structure

```
asklater.ai/
├── .env                          # API keys (gitignored)
├── .gitignore
├── CLAUDE.md                     # Claude Code guidance
├── content/                      # Processed content
│   ├── tiktok/
│   │   └── 2025/12/
│   │       └── 20251229-143052-title.md
│   ├── youtube/
│   │   └── 2025/12/
│   ├── articles/
│   │   └── 2025/12/
│   ├── rss/
│   │   └── 2025/12/
│   ├── _index.md                 # Auto-generated index
│   ├── _topics.md                # Topic-based index
│   └── _recent.md                # Recent items
├── workflows/                    # n8n workflow exports
│   ├── 01-content-intake.json
│   ├── 02-tiktok-processor.json
│   ├── 03-youtube-processor.json
│   ├── 04-article-processor.json
│   ├── 05-rss-processor.json
│   └── 06-file-commit-handler.json
└── docs/
    └── PRD-001-content-aggregation.md
```

---

## 5. Dependencies

### MCP Approach Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| Docker MCP Toolkit | ✅ Installed | Gateway running |
| gmail-mcp | ✅ Configured | App password set |
| youtube_transcript | ✅ Working | No API key needed |
| apify-mcp-server | ✅ Configured | Token set |
| github MCP | ✅ Working | PAT configured |
| Playwright browser | ✅ Built-in | Part of MCP toolkit |

### n8n Approach Dependencies (Alternative)

- [ ] **n8n** - Workflow automation (already installed via Docker)
- [ ] **Anthropic API** - Claude for text summarization
- [ ] **Google AI API** - Gemini for video analysis
- [ ] **Google OAuth2** - Gmail access
- [ ] **Dumpling AI** (optional) - TikTok transcript extraction

### Internal Dependencies

- [x] Gmail account configured with app password
- [x] Git repository initialized
- [x] Docker MCP toolkit configured

---

## 6. Implementation Plan

### Pure MCP Implementation (Recommended)

#### Phase 1: MCP Setup ✅ COMPLETE
- [x] Install Docker MCP Toolkit
- [x] Configure gmail-mcp with app password
- [x] Configure github MCP with personal access token
- [x] Configure apify-mcp-server with API token
- [x] Test youtube_transcript MCP

#### Phase 2: Create Processing Skill
- [ ] Create `/process-emails` Claude Code skill
- [ ] Implement content type detection (regex patterns)
- [ ] Create markdown template generator
- [ ] Test with sample emails

#### Phase 3: Content Processors
- [ ] YouTube: get_transcript → Claude analysis → markdown
- [ ] TikTok: Apify actor → Claude analysis → markdown
- [ ] Articles: Browser fetch → Claude analysis → markdown
- [ ] RSS: Direct fetch → Claude analysis → markdown

#### Phase 4: GitHub Integration
- [ ] Implement push_files for batch commits
- [ ] Create folder structure dynamically
- [ ] Generate _index.md on each commit

#### Phase 5: Automation
- [ ] Create Windows Task Scheduler job (or cron)
- [ ] Or: n8n webhook trigger → Claude Code skill

### n8n Implementation (Alternative)

#### Phase 1: Infrastructure
- [ ] Configure Google OAuth2 credentials in n8n
- [ ] Configure Anthropic credentials
- [ ] Configure GitHub header auth

#### Phase 2: Core Intake Workflow
- [ ] Create Gmail Trigger node
- [ ] Build content type detection logic (Switch node)
- [ ] Add URL extraction (Code node)
- [ ] Implement error handling

#### Phase 3: Content Processors
- [ ] Article Processor (HTTP Request → HTML Extract → Claude)
- [ ] TikTok Processor (URL Extract → Transcript API → Gemini)
- [ ] YouTube Processor (YouTube node → Gemini)
- [ ] RSS Processor (RSS Trigger → HTTP Request → Claude)

#### Phase 4: File Handler
- [ ] Markdown template generation
- [ ] Frontmatter population
- [ ] Git add/commit/push automation
- [ ] Index file regeneration

### Phase 5: Testing & Polish
- [ ] End-to-end testing with sample content
- [ ] Error notification workflow
- [ ] Documentation updates

---

## 7. Out of Scope

- **Real-time processing** - 5 minute polling is acceptable
- **Mobile app** - Web interface only (n8n dashboard)
- **Multi-user support** - Single user system
- **Content editing UI** - Edit markdown directly in IDE
- **Automatic social posting** - Consumption only, not publishing
- **Video file storage** - Transcripts and summaries only

---

## 8. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| TikTok URL extraction fails | Med | Med | Multiple regex patterns; fallback to manual |
| Gmail OAuth expires | High | Low | Refresh token handling; alert on failure |
| API rate limits hit | Med | Low | Batch processing; backoff logic |
| Large content exceeds token limits | Med | Med | Chunking; summarize in stages |
| Git conflicts | Low | Low | Sequential commits; atomic operations |

---

## 9. Testing Strategy

### Integration Tests

- [ ] Email → TikTok detection → Gemini → Markdown → Git
- [ ] Email → Article detection → Claude → Markdown → Git
- [ ] RSS Trigger → Claude → Markdown → Git

### Manual Testing

- [ ] Share TikTok to email, verify processing
- [ ] Share YouTube to email, verify processing
- [ ] Share article URL to email, verify processing
- [ ] Verify markdown renders correctly
- [ ] Verify Claude Code can read and discuss content

---

## 10. Documentation

- [ ] **CLAUDE.md updates** - Content system commands and structure
- [ ] **Workflow exports** - JSON files in workflows/ folder
- [ ] **This PRD** - Feature documentation

---

## 11. Rollout Plan

### Pre-Launch Checklist

- [ ] All API credentials configured
- [ ] Docker volume mounted correctly
- [ ] Test email processed successfully
- [ ] Git commits working

### Launch Strategy

- [ ] Start with article processing (simplest)
- [ ] Add TikTok processing
- [ ] Add YouTube processing
- [ ] Add RSS feeds

---

## 12. Open Questions

1. **TikTok transcript extraction method?**
   - **Decision:** Use Apify MCP actors (e.g., `clockworks/tiktok-scraper`)
   - **Rationale:** ~$1/1000 videos, integrated with MCP toolkit, no separate API

2. **RSS feed configuration?**
   - **Decision:** Add feeds incrementally via n8n UI
   - **Rationale:** Flexibility to add/remove feeds easily

3. **Index regeneration frequency?**
   - **Decision:** Regenerate on each new content addition
   - **Rationale:** Always up-to-date; low overhead

---

## 13. Approvals

| Stakeholder | Role | Status | Date |
|-------------|------|--------|------|
| Nolan | Owner | Approved | 2025-12-29 |

---

## 14. Change Log

| Date | Author | Changes |
|------|--------|---------|
| 2025-12-29 | Claude Code | Initial draft |
| 2025-12-30 | Claude Code | Added MCP-powered architecture as recommended approach; updated dependencies and implementation plan |
