# LSP Store

**Last Updated:** 2025-11-16

---

## Overview

LspStore manages language servers for the project.

## Structure

```rust
pub struct LspStore {
    language_servers: HashMap<LanguageServerId, LanguageServerState>,
    buffers: HashMap<BufferId, BufferState>,
}
```

## Language Server Lifecycle

```
Project opens buffer
  │
  ▼
Determine language
  │
  ▼
Find/start language server
  │
  ▼
Register buffer with server
  │
  ▼
Send didOpen notification
  │
  ▼
Receive diagnostics/capabilities
```

## Request Handling

```rust
// Completions
let completions = lsp_store.completions(buffer, position, cx).await?;

// Hover
let hover = lsp_store.hover(buffer, position, cx).await?;

// Definition
let locations = lsp_store.definition(buffer, position, cx).await?;
```

## Further Reading

- [LSP Documentation](../06-lsp/README.md)
- [Editor LSP Integration](../02-editor/lsp-integration.md)
