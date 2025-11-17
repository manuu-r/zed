# LSP Crate

**Path:** `/home/user/zed/crates/lsp/`
**Purpose:** Language Server Protocol client implementation

---

## Overview

Implements LSP client for IDE features like completions, diagnostics, and refactoring.

## Core Types

```rust
pub struct LanguageServer {
    server_id: LanguageServerId,
    name: String,
    capabilities: ServerCapabilities,
    // ...
}
```

## Documentation Files

- **[Protocol](./protocol.md)** - LSP message protocol
- **[Capabilities](./capabilities.md)** - Server capabilities
- **[Adapters](./adapters.md)** - Language-specific adapters
- **README.md** - This file

## Further Reading

- [Editor LSP Integration](../02-editor/lsp-integration.md)
- [Project LSP Store](../03-project/lsp-store.md)
