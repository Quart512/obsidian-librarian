# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Obsidian Librarian** is a plugin that automatically summarizes and classifies Obsidian notes using Claude API, then enables natural language search across hundreds of notes using token-efficient 1-2 sentence summaries instead of full content.

**Key Innovation**: Solves Claude API's 200K token context limit for large vaults by indexing summaries (24K tokens for 800 notes) instead of full text (800K tokens).

**Current Status**: MVP development phase (weeks 1-6), starting with plugin infrastructure and core features (F001-F007).

## Architecture Overview

### Data Model

**NoteIndex** - Core persistent data structure:
- `path`: file path in vault (unique key)
- `title`: note title
- `summary`: AI-generated 1-2 sentence summary (F002 creates, F004 search uses)
- `contentHash`: for change detection
- `existingTags`: current frontmatter tags
- `lastModified`, `indexedAt`: timestamps

Stored via Obsidian `saveData/loadData()` in `.obsidian/plugins/obsidian-librarian/data.json` (plain text, no encryption).

**ClassificationRecommendation** - Temporary in-memory structure:
- `notePath`, `recommendedTags`, `reason`, `status` ('pending'/'accepted'/'rejected')

**SearchResult** - API response structure:
- `notePath`, `title`, `relevanceExplanation`, `summaryExcerpt`, `rank`

**PluginSettings**:
- `claudeApiKey` (stored plain text, user notified in UI)
- `claudeModel` (default: `claude-haiku-4-5-20251001`)
- `excludedFolders`, `maxConcurrentRequests` (1-3, default 1)

### UI/View Architecture

**Sidebar Panel** (ItemView) - Two tabs:
1. **Classification Tab** (F001-F003, F007):
   - Scan button → triggers F001 (indexing) + F002 (summary + tag recommendations)
   - Progress bar with current filename
   - Recommendation cards: title, summary, existing tags, recommended tags, reason
   - Accept/reject buttons per card, "Accept All" button
   - Card click → opens Review Modal

2. **Search Tab** (F004-F005, F007):
   - Natural language query input field
   - Calls Claude with NoteIndex.summary array only (NOT full content)
   - Results list: title, summary preview, relevance explanation
   - Click result → opens note in Obsidian editor

**Review Modal** (F003):
- Shows single note details: title, summary, content preview (first 200 chars)
- Editable tag chips UI
- Apply/Cancel buttons

**Settings Tab** (F006):
- API key input (masked, plain text storage notice)
- Model selection dropdown
- Excluded folders input
- Concurrent requests slider (1-3)
- API test button

### API Integration

**CRITICAL**: Use **Obsidian `requestUrl()` API**, NOT `fetch()`:
- Obsidian environment blocks fetch due to CORS restrictions
- `requestUrl()` works on desktop + mobile, handles authentication headers

Wrapper module signature:
```typescript
async callClaudeAPI(
  prompt: string,
  options?: { model?: string; maxTokens?: number; }
): Promise<{ summary: string; tags: string[]; reason: string }> // for F002
```

**Rate Limiting**: 429 responses → exponential backoff (1s → 2s → 4s), max 3 retries. UI shows "Waiting for API rate limit..." during backoff.

## Development Checklist (MVP = 5-6 weeks)

### Week 1: Foundation
- [ ] Obsidian plugin TypeScript + esbuild setup
- [ ] Plugin settings tab (F006): API key, model selection, excluded folders, concurrent requests
- [ ] Sidebar panel basic layout: classification & search tabs
- [ ] Claude API wrapper using Obsidian `requestUrl()`
- [ ] saveData/loadData for PluginSettings & NoteIndex persistence
- [ ] Change detection: lastModified comparison logic

### Week 2: Scan & Summarize
- [ ] F001: vault.getMarkdownFiles() + read + NoteIndex creation
- [ ] F002: Claude prompt for summary + tags + reason (single API call, JSON response)
- [ ] Store summary in NoteIndex.summary field
- [ ] Progress UI (progress bar, current filename, throttle updates ~100ms)
- [ ] Semaphore: limit concurrent requests (default 1, max 3)
- [ ] Rate limit retry logic with UI feedback

### Week 3: Apply Recommendations
- [ ] F003: Recommendation card list UI
- [ ] Individual accept/reject buttons
- [ ] "Accept All" with confirmation modal
- [ ] Review modal: tag edit chips, apply to frontmatter via `fileManager.processFrontMatter()`
- [ ] Update NoteIndex.existingTags in memory after apply

### Week 4: Search
- [ ] F004: Search input UI in search tab
- [ ] Gather all NoteIndex.summary values (NOT full content)
- [ ] Claude API call: pass summaries + query
- [ ] F005: Result card list UI with relevance explanation
- [ ] Click result → open note via Obsidian API
- [ ] Handle no-index warning

### Week 5: Integration Testing
- [ ] E2E: scan → review → apply
- [ ] E2E: search query → view results → open note
- [ ] API error handling (network, key invalid, rate limits)
- [ ] requestUrl() edge cases

### Week 6: Performance & Deploy
- [ ] Test with 100+ note vault
- [ ] Verify 500+ note search token usage < Claude context limit
- [ ] README + manifest.json + license
- [ ] Community plugin submission prep

## Key Implementation Patterns

### Obsidian API Usage
- `vault.getMarkdownFiles()`: iterate all .md files
- `vault.read(file)`: load note content
- `metadataCache.getFileCache()`: get frontmatter + headings without I/O
- `fileManager.processFrontMatter()`: safe YAML update in frontmatter
- `requestUrl()`: HTTP calls (use this, never fetch)
- `ItemView`: sidebar panel base class
- `Modal`: dialog UI
- `Notice`: toast notifications

### Concurrency Control
Implement Semaphore pattern to limit concurrent API calls:
```typescript
// default: 1, max: 3
for (let i = 0; i < files.length; i++) {
  await semaphore.acquire();
  callClaudeAPI(...).finally(() => semaphore.release());
  // Update progress UI (throttle to ~100ms)
}
```

### Summary-Based Search
Critical for token efficiency:
```typescript
// Collect only summaries, NOT full content
const summaryIndex = noteIndex.map(n => ({ path: n.path, summary: n.summary }));
// Send to Claude with query
const results = await claudeAPI.search(query, summaryIndex);
```

## API Cost & Performance

- **Note 100 tokens/summary** (1-2 sentences) → 800 notes = 80K summary tokens
- **Search**: ~24K input tokens (summaries only) vs 800K (full content) = 97% savings
- **Target**: <$0.001 per note processed, <$0.04 per search
- **Target speed**: 100 notes / 60 seconds (1 concurrent request)

## Critical Decisions

1. **Plain text API key storage** (no encryption): Obsidian standard, users notified in settings UI
2. **requestUrl() for HTTP**: CORS-safe, cross-platform
3. **Summary-first indexing**: Solves context limit, enables large-scale search
4. **Default concurrency = 1**: Prevents request flooding, safer rate limit handling
5. **lastModified tracking**: Minimizes re-indexing on vault restart
