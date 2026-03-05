# Obsidian API Patterns for Librarian Plugin

## Critical: requestUrl vs fetch
- ALWAYS use `requestUrl` from 'obsidian' package for external HTTP calls
- Native fetch() works in Obsidian desktop but fails in some mobile contexts
- requestUrl handles CORS and certificate issues automatically

## processFrontMatter Pattern
```typescript
await this.app.fileManager.processFrontMatter(file, (fm) => {
    fm.tags = mergedTags;
});
```
This is atomic and safe - never use vault.read() + vault.modify() for frontmatter updates.

## ItemView Lifecycle
- onOpen(): create DOM, register event handlers
- onClose(): clean up, remove event handlers
- VIEW_TYPE must be unique string constant

## Concurrency Control Pattern
Use a semaphore pattern for maxConcurrentRequests:
```typescript
class Semaphore {
    private queue: Array<() => void> = [];
    constructor(private limit: number) {}
    async acquire() { ... }
    release() { ... }
}
```
