# Obsidian Expert Agent Memory

## Project: Obsidian Librarian Plugin

**PRD Location**: `C:\Users\jimmy\Documents\AIEducation\workspace\obsidian_librarian\PRD.md`

### Core Architecture Decisions
- Plugin runtime: TypeScript 5.6+ with esbuild bundler
- AI backend: Anthropic Claude API via REST (requestUrl from Obsidian API, NOT Node fetch)
- Default model: `claude-haiku-4-5-20251001`
- Index storage: in-memory only (NoteIndex[] array), no persistence across restarts
- Settings storage: Obsidian saveData/loadData -> data.json

### Key Data Models (from PRD Section 6)
- `NoteIndex`: path, title, content, existingTags, lastModified, indexedAt
- `ClassificationRecommendation`: notePath, recommendedTags, reason, status, createdAt
- `SearchResult`: notePath, title, relevanceExplanation, matchedExcerpt, rank
- `PluginSettings`: claudeApiKey, claudeModel, excludedFolders[], maxConcurrentRequests (1-5, default 2)

### MVP Features (Phase 1)
- F001: Note scan & indexing (vault.getFiles, vault.read)
- F002: AI classification/tag recommendation
- F003: User review & apply (processFrontMatter)
- F004: Natural language search input
- F005: Search results display
- F006: Plugin settings tab
- F007: Sidebar panel (ItemView) with classify + search tabs

### Critical API Patterns
- Use `this.app.vault.requestUrl()` pattern is WRONG - use Obsidian's `requestUrl` from 'obsidian' import
- `processFrontMatter()` is the correct API for updating frontmatter (not vault.modify directly)
- ItemView requires VIEW_TYPE constant and proper leaf management

### Detailed Notes
- See `api-patterns.md` for Obsidian API code patterns
- See `prompts.md` for Claude prompt engineering strategies
- See `performance.md` for large vault optimization patterns
