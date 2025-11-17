# LSP Capabilities

**Last Updated:** 2025-11-16

---

## Overview

Server capabilities define what features are supported.

## Capability Checking

```rust
if language_server.capabilities().completion_provider.is_some() {
    // Server supports completions
}

if language_server.capabilities().hover_provider {
    // Server supports hover
}
```

## Common Capabilities

- `completionProvider` - Code completion
- `hoverProvider` - Hover information
- `definitionProvider` - Go to definition
- `referencesProvider` - Find references
- `documentFormattingProvider` - Formatting
- `codeActionProvider` - Quick fixes
- `renameProvider` - Symbol rename

## Further Reading

- [LSP README](./README.md)
- [Protocol](./protocol.md)
