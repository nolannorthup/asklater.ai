# PRD-002: AskLater

**"Save now. Ask Claude later."**

**Status:** Draft
**Created:** 2025-12-29
**Author:** Nolan / Claude Code
**Target Release:** 2025-01
**Related Issues:** Supersedes PRD-001 (n8n approach)

---

## 1. Overview

### Problem Statement

Content is consumed from multiple sources (TikTok, YouTube, articles, RSS feeds) but there's no centralized system to capture, process, and organize this information. The previous approach (PRD-001) relied on n8n workflows with fixed processing paths, which lacked:

- **Adaptive processing** - All content treated the same regardless of complexity
- **Intelligent routing** - No assessment of content before processing
- **Unified orchestration** - External workflow engine required
- **Native AI integration** - External API calls for AI processing

### Proposed Solution

**AskLater** is a personal knowledge capture system that uses Claude Code sub-agents and Docker MCP tools to intelligently process content you find interesting - so you can ask Claude about it later.

The system:

1. **Captures content** you share via email
2. **Assesses complexity** to determine processing depth
3. **Processes with specialized agents** (TikTok, YouTube, Article, RSS)
4. **Stores as AI-queryable markdown** for future Claude conversations
5. **Maintains audit trail** of all processing decisions

### Success Metrics

- [ ] Content shared via email is automatically processed within 5 minutes
- [ ] Processing depth adapts to content complexity (simple articles vs deep technical content)
- [ ] Sub-agents successfully use MCP tools for extraction and commits
- [ ] All content is indexed and searchable in Git repository
- [ ] Claude Code can read and discuss any captured content

---

## 2. User Stories

### Primary User Story

**As a** knowledge worker
**I want to** share interesting content to an email address and have it intelligently processed
**So that** I can ask Claude about it later with full context

**Acceptance Criteria:**
- [ ] Simple content gets quick TLDR processing (minimal depth)
- [ ] Complex technical content gets comprehensive analysis (full depth)
- [ ] Processing adapts without manual intervention
- [ ] All content stored as markdown with consistent frontmatter

### Secondary User Stories

**As a** researcher
**I want** complex academic or technical content to receive deeper analysis
**So that** I capture nuanced insights and related topics for exploration

**As a** casual user
**I want** entertainment content (memes, short videos) processed quickly
**So that** I don't wait for unnecessary deep analysis

---

## 3. Requirements

### Functional Requirements

1. **Email Intake Agent**: Monitor Gmail for new content, classify content type
2. **Content Assessment**: Evaluate complexity, determine processing depth
3. **Adaptive Processing**: Route to appropriate depth level (minimal/standard/comprehensive)
4. **Content-Specific Agents**: TikTok, YouTube, Article, RSS processors
5. **MCP Tool Integration**: Use Docker MCP toolkit for all external operations
6. **GitHub Commit Agent**: Push processed content to repository
7. **Audit Logging**: Track all processing decisions and outcomes

### Non-Functional Requirements

- **Performance**: Content processed within 5 minutes of email receipt
- **Adaptability**: Processing depth scales with content complexity
- **Reliability**: Failed processing logged but doesn't block other content
- **Cost**: No external API calls - all AI processing via Claude Code native

---

## 4. Technical Approach

### AskLater Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              AskLater                                    │
│                     "Save now. Ask Claude later."                        │
│                                                                          │
│  Trigger: "/asklater" command or scheduled task                          │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                    🔵 INTAKE PHASE                                  │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐    │ │
│  │  │ gmail-mcp   │───▶│ Content     │───▶│ Complexity          │    │ │
│  │  │ listMessages│    │ Detection   │    │ Assessment          │    │ │
│  │  └─────────────┘    │             │    │                     │    │ │
│  │                     │ • TikTok    │    │ • Minimal (quick)   │    │ │
│  │                     │ • YouTube   │    │ • Standard (normal) │    │ │
│  │                     │ • Article   │    │ • Comprehensive     │    │ │
│  │                     │ • RSS       │    │   (deep analysis)   │    │ │
│  │                     └─────────────┘    └─────────────────────┘    │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                    │                                     │
│                                    ▼                                     │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                    🟢 PROCESSING PHASE                              │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │                                                                     │ │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │ │
│  │  │ TikTok Agent    │  │ YouTube Agent   │  │ Article Agent   │    │ │
│  │  │                 │  │                 │  │                 │    │ │
│  │  │ • apify-mcp     │  │ • youtube_      │  │ • markdownify   │    │ │
│  │  │ • markdownify   │  │   transcript    │  │   webpage-to-md │    │ │
│  │  │ • Claude analyze│  │ • Claude analyze│  │ • Claude analyze│    │ │
│  │  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘    │ │
│  │           │                    │                    │              │ │
│  │           └────────────────────┼────────────────────┘              │ │
│  │                                ▼                                    │ │
│  │                    ┌─────────────────┐                             │ │
│  │                    │ Markdown Gen    │                             │ │
│  │                    │ (depth-aware)   │                             │ │
│  │                    └────────┬────────┘                             │ │
│  └─────────────────────────────┼──────────────────────────────────────┘ │
│                                ▼                                         │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                    🟡 COMMIT PHASE                                  │ │
│  ├────────────────────────────────────────────────────────────────────┤ │
│  │  ┌─────────────────┐    ┌─────────────────┐                        │ │
│  │  │ github MCP      │───▶│ Update Index    │                        │ │
│  │  │ push_files      │    │ _index.md       │                        │ │
│  │  └─────────────────┘    └─────────────────┘                        │ │
│  └────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### Sub-Agent Architecture

| Agent | Role | MCP Tools | Task Tool Config |
|-------|------|-----------|------------------|
| **Intake Agent** | Fetch emails, classify content, assess complexity | `gmail-mcp` | `subagent_type: "general-purpose"` |
| **TikTok Agent** | Extract TikTok content and transcripts | `apify-mcp`, `markdownify` | `subagent_type: "general-purpose"` |
| **YouTube Agent** | Get transcripts, analyze video content | `youtube_transcript`, `markdownify` | `subagent_type: "general-purpose"` |
| **Article Agent** | Scrape and extract article content | `markdownify.webpage-to-markdown` | `subagent_type: "general-purpose"` |
| **RSS Agent** | Parse RSS feeds, extract entries | `markdownify`, direct fetch | `subagent_type: "general-purpose"` |
| **Commit Agent** | Push to GitHub, update indexes | `github.push_files` | `subagent_type: "general-purpose"` |

### Adaptive Processing Depth

| Depth Level | When Applied | Processing |
|-------------|--------------|------------|
| **Minimal** | Entertainment, memes, short clips | TLDR + 3 key points |
| **Standard** | News articles, tutorials, how-tos | Full summary + topics + entities |
| **Comprehensive** | Technical papers, long-form, complex topics | Deep analysis + web augmentation + exploration topics |

#### Complexity Assessment Criteria

```
MINIMAL depth if:
- Video < 60 seconds
- Article < 500 words
- Entertainment/meme content
- No technical terminology

STANDARD depth if:
- Video 1-10 minutes
- Article 500-2000 words
- News/tutorial content
- Some technical content

COMPREHENSIVE depth if:
- Video > 10 minutes
- Article > 2000 words
- Academic/research content
- Heavy technical terminology
- Multiple complex topics
```

### MCP Tool Mapping

| Capability | MCP Server | Tool(s) | Status |
|------------|------------|---------|--------|
| Email Access | `gmail-mcp` | `listMessages`, `getMessage` | ✅ Working |
| YouTube Transcripts | `youtube_transcript` | `get_transcript`, `get_video_info` | ✅ Working |
| TikTok Scraping | `apify-mcp-server` | `call-actor` (TikTok scrapers) | ✅ Configured |
| Web Scraping | `markdownify` | `webpage-to-markdown` | ✅ Working |
| PDF/Doc Conversion | `markdownify` | `pdf-to-markdown`, `docx-to-markdown` | ✅ Available |
| Web Search | Claude Code | `WebSearch` (built-in) | ✅ Built-in |
| GitHub Commits | `github` | `push_files`, `create_or_update_file` | ✅ Working |
| Browser Automation | `playwright` | Full browser control | ✅ Available |

### Data Model

#### Markdown Frontmatter Schema

```yaml
---
id: string                    # Unique identifier
title: string                 # Extracted or generated title
source_type: enum             # tiktok | youtube | article | rss
source_url: string            # Original URL
captured_at: datetime         # When email was received
processed_at: datetime        # When AI processing completed

# AskLater Metadata
processing_depth: enum        # minimal | standard | comprehensive
complexity_score: number      # 1-10 assessed complexity
processing_agent: string      # Which sub-agent processed this

# Content Analysis
topics: string[]              # AI-extracted topics
tags: string[]                # AI-generated tags
entities:
  people: string[]
  companies: string[]
  technologies: string[]

# Audit Trail
intake_timestamp: datetime    # When intake agent received
assessment_timestamp: datetime # When complexity assessed
completion_timestamp: datetime # When processing finished
processing_notes: string[]    # Any issues or decisions logged
---
```

#### Markdown Body Structure (Depth-Aware)

**Minimal Depth:**
```markdown
## TLDR
[1-2 sentence summary]

## Key Points
- Point 1
- Point 2
- Point 3

## Source
- URL: [link]
- Processed: [timestamp]
```

**Standard Depth:**
```markdown
## TLDR
[1-2 sentence summary]

## Key Points
- Point 1
- Point 2
- Point 3
- Point 4
- Point 5

## Summary
[3-5 paragraph detailed summary]

## Topics
[Comma-separated topics]

## Entities
- People: [names]
- Companies: [names]
- Technologies: [names]

## Source
- URL: [link]
- Processed: [timestamp]
```

**Comprehensive Depth:**
```markdown
## TLDR
[1-2 sentence summary]

## Key Points
- Point 1 with detail
- Point 2 with detail
- Point 3 with detail
- Point 4 with detail
- Point 5 with detail

## Summary
[5-10 paragraph detailed analysis]

## Notable Quotes
> "Quote from content"

## Context (Web Search Augmented)
[Additional context from WebSearch]

## Topics for Exploration
- [ ] Follow-up question 1
- [ ] Related topic to investigate
- [ ] Deeper dive opportunity

## Related Content
[Links to related processed content]

## Raw Content
<details>
<summary>Original transcript/text</summary>
[Full original content]
</details>

## Processing Notes
- Depth: Comprehensive
- Complexity Score: X/10
- Processing Agent: [agent name]
- Processed: [timestamp]
```

### Folder Structure

```
asklater.ai/
├── .env                          # API keys (gitignored)
├── .mcp.json                     # Docker MCP configuration
├── .asklater/                    # AskLater processing logs
│   ├── audit.md                  # Complete audit trail
│   └── state.md                  # Current processing state
├── CLAUDE.md                     # Claude Code guidance
├── content/                      # Processed content
│   ├── tiktok/2025/12/
│   ├── youtube/2025/12/
│   ├── articles/2025/12/
│   ├── rss/2025/12/
│   ├── _index.md                 # Auto-generated index
│   ├── _topics.md                # Topic-based index
│   └── _recent.md                # Recent items
└── docs/
    ├── PRD-001-content-aggregation.md  # Original approach (deprecated)
    └── PRD-002-asklater.md             # This PRD
```

---

## 5. Dependencies

### MCP Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| Docker MCP Toolkit | ✅ Installed | Gateway running |
| gmail-mcp | ✅ Configured | App password set |
| youtube_transcript | ✅ Working | No API key needed |
| apify-mcp-server | ✅ Configured | Token set |
| markdownify | ✅ Working | Paths configured |
| github MCP | ✅ Working | PAT configured |
| playwright | ✅ Available | Built-in to toolkit |

### Internal Dependencies

- [x] Gmail account configured with app password
- [x] Git repository initialized
- [x] Docker MCP toolkit configured
- [x] Claude Code with Task tool access

---

## 6. Implementation Plan

### Phase 1: Core Skill Creation ✅
- [x] Create `/asklater` Claude Code skill
- [x] Implement Intake Agent with gmail-mcp
- [x] Build content type detection (TikTok, YouTube, Article, RSS patterns)
- [x] Create complexity assessment logic

### Phase 2: Content Processor Agents
- [ ] YouTube Agent: youtube_transcript → Claude analysis → markdown
- [ ] Article Agent: markdownify → Claude analysis → markdown
- [ ] TikTok Agent: apify-mcp → Claude analysis → markdown
- [ ] RSS Agent: direct fetch → Claude analysis → markdown

### Phase 3: Adaptive Depth Implementation
- [ ] Implement minimal depth processing
- [ ] Implement standard depth processing
- [ ] Implement comprehensive depth with WebSearch augmentation
- [ ] Create depth selection logic based on complexity assessment

### Phase 4: GitHub Integration
- [ ] Implement Commit Agent with github.push_files
- [ ] Create folder structure dynamically
- [ ] Generate _index.md on each commit
- [ ] Implement _topics.md and _recent.md updates

### Phase 5: Audit & State Tracking
- [ ] Create .asklater/audit.md logging
- [ ] Implement state.md tracking
- [ ] Add error handling and recovery
- [ ] Create processing statistics

### Phase 6: Automation
- [ ] Test manual `/asklater` command
- [ ] Create Windows Task Scheduler job (optional)
- [ ] Document usage in CLAUDE.md

---

## 7. Out of Scope

- ❌ **n8n integration** - Pure Claude Code + MCP approach
- ❌ **Real-time processing** - Batch processing on command
- ❌ **Mobile app** - CLI/chat interface only
- ❌ **Multi-user support** - Single user system
- ❌ **Video file storage** - Transcripts and summaries only
- ❌ **Social posting** - Consumption only, not publishing

---

## 8. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| MCP tool failures | Med | Low | Graceful degradation, retry logic |
| Complexity assessment errors | Low | Med | Default to standard depth |
| Large content exceeds context | Med | Med | Chunking, summarize in stages |
| TikTok scraping blocked | Med | Med | Multiple Apify actors as fallback |
| GitHub rate limits | Low | Low | Batch commits, backoff logic |

---

## 9. Testing Strategy

### Integration Tests
- [ ] Email → Intake Agent → Content Detection → Correct Processor Agent
- [ ] YouTube URL → YouTube Agent → Markdown → GitHub Commit
- [ ] Article URL → Article Agent → Markdown → GitHub Commit
- [ ] Complexity assessment → Correct depth level applied

### Manual Testing
- [ ] Share TikTok to email, verify processing with correct depth
- [ ] Share long technical article, verify comprehensive processing
- [ ] Share short meme video, verify minimal processing
- [ ] Verify Claude Code can read and discuss content

---

## 10. Documentation

- [ ] **CLAUDE.md updates** - Add /asklater command documentation
- [ ] **This PRD** - Complete feature documentation
- [ ] **.asklater/README.md** - Explain audit trail format
- [ ] **Skill documentation** - How to use and extend processors

---

## 11. Rollout Plan

### Pre-Launch Checklist
- [ ] All MCP tools tested and working
- [ ] Skill created and functional
- [ ] Test email processed successfully
- [ ] Git commits working

### Launch Strategy
- [ ] Start with Article processing (simplest)
- [ ] Add YouTube processing
- [ ] Add TikTok processing
- [ ] Add RSS feeds

---

## 12. Open Questions

1. **Scheduling approach?**
   - **Decision:** Manual command initially, Task Scheduler later
   - **Rationale:** Simpler to debug, add automation once stable

2. **Parallel vs sequential processing?**
   - **Decision:** Parallel sub-agents for multiple content items
   - **Rationale:** Task tool supports concurrent agents

3. **State persistence between runs?**
   - **Decision:** Use state.md and Git history
   - **Rationale:** Simple, auditable, no database needed

---

## 13. Approvals

| Stakeholder | Role | Status | Date |
|-------------|------|--------|------|
| Nolan | Owner | Pending | - |

---

## 14. Change Log

| Date | Author | Changes |
|------|--------|---------|
| 2025-12-29 | Claude Code | Initial draft - AI-DLC adapted architecture |
| 2025-12-30 | Claude Code | Renamed to AskLater, updated branding and folder structure |
