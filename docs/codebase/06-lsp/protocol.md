# LSP Protocol

**Last Updated:** 2025-11-16

---

## Overview

LSP uses JSON-RPC over stdio or sockets.

## Request/Response

```rust
// Send request
let response = language_server.request::<lsp::Completion>(
    lsp::CompletionParams {
        text_document: text_document_identifier,
        position: lsp_position,
        context: None,
    }
).await?;
```

## Notifications

```rust
// Send notification (no response)
language_server.notify::<lsp::DidChangeTextDocument>(
    lsp::DidChangeTextDocumentParams {
        text_document: versioned_identifier,
        content_changes: vec![change],
    }
)?;
```

## Message Handling

Server-to-client notifications:

```rust
language_server.on_notification::<lsp::PublishDiagnostics, _>(
    |params, cx| {
        // Handle diagnostics
    }
);
```

## Further Reading

- [LSP README](./README.md)
- [Capabilities](./capabilities.md)
