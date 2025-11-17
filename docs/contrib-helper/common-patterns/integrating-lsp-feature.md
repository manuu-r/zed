# Integrating LSP Features

This guide shows you how to implement Language Server Protocol (LSP) features in Zed.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [LSP Architecture](#lsp-architecture)
4. [Implementing a Feature](#implementing-a-feature)
5. [Common LSP Features](#common-lsp-features)
6. [Testing](#testing)
7. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How Zed's LSP client works
- How to implement LSP protocol features
- How to handle server capabilities
- How to integrate with the UI

**When to use this:**
- Adding new LSP features
- Improving language support
- Implementing IDE features

**Time to complete:** 2-4 hours

## Prerequisites

- Understanding of LSP protocol
- Familiarity with async Rust
- Knowledge of editor architecture

## LSP Architecture

### Key Components

```
crates/lsp/           # LSP client implementation
crates/project/       # Project-level LSP integration
crates/editor/        # Editor UI for LSP features
crates/language/      # Language definitions
```

### Request Flow

```
User Action
    ↓
Editor (UI)
    ↓
Project (coordination)
    ↓
LSP Client (protocol)
    ↓
Language Server
    ↓
Response processing
    ↓
Editor (UI update)
```

## Implementing a Feature

Let's implement "Code Lens" support.

### Step 1: Check Server Capability

```rust
// File: crates/lsp/src/lsp.rs

#[derive(Debug, Clone)]
pub struct ServerCapabilities {
    // ... existing capabilities

    /// Code lens support
    pub code_lens_provider: Option<CodeLensOptions>,
}

#[derive(Debug, Clone)]
pub struct CodeLensOptions {
    pub resolve_provider: Option<bool>,
}
```

### Step 2: Define the Request

```rust
// File: crates/lsp/src/lsp.rs

impl LanguageServer {
    /// Request code lenses for a document
    pub async fn code_lens(
        &self,
        params: CodeLensParams,
    ) -> Result<Option<Vec<CodeLens>>> {
        self.request::<request::CodeLensRequest>(params).await
    }

    /// Resolve a code lens
    pub async fn code_lens_resolve(
        &self,
        params: CodeLens,
    ) -> Result<CodeLens> {
        self.request::<request::CodeLensResolve>(params).await
    }
}
```

### Step 3: Add to Language Server Adapter

```rust
// File: crates/language/src/language.rs

impl LanguageServerAdapter for MyLanguageAdapter {
    fn initialization_options(&self) -> Option<serde_json::Value> {
        Some(json!({
            "codeLens": {
                "enabled": true
            }
        }))
    }

    fn workspace_configuration(&self) -> Option<serde_json::Value> {
        Some(json!({
            "codeLens": {
                "refreshOnChange": true
            }
        }))
    }
}
```

### Step 4: Implement in Project

```rust
// File: crates/project/src/project.rs

impl Project {
    pub fn code_lenses(
        &self,
        buffer: Entity<Buffer>,
        cx: &mut Context<Self>,
    ) -> Task<Result<Vec<CodeLens>>> {
        let buffer_snapshot = buffer.read(cx).snapshot();
        let language_server = self.language_server_for_buffer(buffer.read(cx), cx);

        cx.spawn(async move |this, cx| {
            let language_server = language_server?;

            let params = CodeLensParams {
                text_document: TextDocumentIdentifier {
                    uri: lsp::Url::from_file_path(&buffer_snapshot.file().path()).unwrap(),
                },
                work_done_progress_params: Default::default(),
                partial_result_params: Default::default(),
            };

            let response = language_server.code_lens(params).await?;

            Ok(response.unwrap_or_default())
        })
    }
}
```

### Step 5: Add UI Integration

```rust
// File: crates/editor/src/editor.rs

pub struct CodeLensDisplay {
    lenses: Vec<CodeLens>,
    // Display state
}

impl Editor {
    pub fn refresh_code_lenses(&mut self, cx: &mut Context<Self>) {
        let project = self.project.clone();
        let buffer = self.buffer.clone();

        let task = cx.spawn(async move |editor, cx| {
            let lenses = project
                .update(&cx, |project, cx| {
                    project.code_lenses(buffer, cx)
                })?
                .await?;

            editor.update(&cx, |editor, cx| {
                editor.display_code_lenses(lenses, cx);
            })
        });

        task.detach_and_log_err(cx);
    }

    fn display_code_lenses(&mut self, lenses: Vec<CodeLens>, cx: &mut Context<Self>) {
        // Update UI to show code lenses
        cx.notify();
    }
}
```

### Step 6: Handle User Interaction

```rust
impl Editor {
    pub fn execute_code_lens(&mut self, lens: &CodeLens, cx: &mut Context<Self>) {
        if let Some(command) = &lens.command {
            self.execute_lsp_command(command, cx);
        }
    }

    fn execute_lsp_command(&mut self, command: &Command, cx: &mut Context<Self>) {
        let project = self.project.clone();

        cx.spawn(async move |_, cx| {
            project.update(&cx, |project, cx| {
                project.execute_command(command, cx)
            })
        }).detach_and_log_err(cx);
    }
}
```

## Common LSP Features

### Code Actions

```rust
impl Project {
    pub fn code_actions(
        &mut self,
        buffer: &Entity<Buffer>,
        range: Range<Anchor>,
        cx: &mut Context<Self>,
    ) -> Task<Result<Vec<CodeAction>>> {
        let buffer_snapshot = buffer.read(cx).snapshot();
        let range_lsp = range_to_lsp(range, &buffer_snapshot);
        let language_server = self.language_server_for_buffer(buffer.read(cx), cx);

        cx.spawn(async move |_, _| {
            let language_server = language_server?;

            let params = CodeActionParams {
                text_document: text_document_identifier(&buffer_snapshot),
                range: range_lsp,
                context: CodeActionContext {
                    diagnostics: vec![],
                    only: None,
                    trigger_kind: Some(CodeActionTriggerKind::INVOKED),
                },
                work_done_progress_params: Default::default(),
                partial_result_params: Default::default(),
            };

            let response = language_server
                .code_action(params)
                .await?;

            Ok(response.unwrap_or_default())
        })
    }
}
```

### Hover Information

```rust
impl Project {
    pub fn hover(
        &self,
        buffer: Entity<Buffer>,
        position: Anchor,
        cx: &mut Context<Self>,
    ) -> Task<Result<Option<Hover>>> {
        let buffer_snapshot = buffer.read(cx).snapshot();
        let position_lsp = point_to_lsp(position.to_point(&buffer_snapshot));
        let language_server = self.language_server_for_buffer(buffer.read(cx), cx);

        cx.spawn(async move |_, _| {
            let language_server = language_server?;

            let params = HoverParams {
                text_document_position_params: TextDocumentPositionParams {
                    text_document: text_document_identifier(&buffer_snapshot),
                    position: position_lsp,
                },
                work_done_progress_params: Default::default(),
            };

            language_server.hover(params).await
        })
    }
}
```

### Document Symbols

```rust
impl Project {
    pub fn document_symbols(
        &self,
        buffer: Entity<Buffer>,
        cx: &mut Context<Self>,
    ) -> Task<Result<Vec<DocumentSymbol>>> {
        let buffer_snapshot = buffer.read(cx).snapshot();
        let language_server = self.language_server_for_buffer(buffer.read(cx), cx);

        cx.spawn(async move |_, _| {
            let language_server = language_server?;

            let params = DocumentSymbolParams {
                text_document: text_document_identifier(&buffer_snapshot),
                work_done_progress_params: Default::default(),
                partial_result_params: Default::default(),
            };

            let response = language_server
                .document_symbols(params)
                .await?;

            match response {
                Some(DocumentSymbolResponse::Nested(symbols)) => Ok(symbols),
                Some(DocumentSymbolResponse::Flat(_)) => {
                    // Convert flat symbols to nested
                    Ok(vec![])
                }
                None => Ok(vec![]),
            }
        })
    }
}
```

### Inlay Hints

```rust
impl Project {
    pub fn inlay_hints(
        &self,
        buffer: Entity<Buffer>,
        range: Range<Anchor>,
        cx: &mut Context<Self>,
    ) -> Task<Result<Vec<InlayHint>>> {
        let buffer_snapshot = buffer.read(cx).snapshot();
        let range_lsp = range_to_lsp(range, &buffer_snapshot);
        let language_server = self.language_server_for_buffer(buffer.read(cx), cx);

        cx.spawn(async move |_, _| {
            let language_server = language_server?;

            // Check capability
            if language_server.capabilities().inlay_hint_provider.is_none() {
                return Ok(vec![]);
            }

            let params = InlayHintParams {
                text_document: text_document_identifier(&buffer_snapshot),
                range: range_lsp,
                work_done_progress_params: Default::default(),
            };

            let response = language_server
                .inlay_hints(params)
                .await?;

            Ok(response.unwrap_or_default())
        })
    }
}
```

## Testing

### Unit Test

```rust
#[gpui::test]
async fn test_code_lens_request(cx: &mut TestApp) {
    let (language_server, fake_server) = LanguageServer::fake(cx).await;

    // Simulate server response
    fake_server
        .handle_request::<request::CodeLensRequest, _, _>(|params, _| async move {
            Ok(Some(vec![CodeLens {
                range: lsp::Range {
                    start: lsp::Position { line: 0, character: 0 },
                    end: lsp::Position { line: 0, character: 10 },
                },
                command: Some(Command {
                    title: "Run Test".to_string(),
                    command: "run.test".to_string(),
                    arguments: None,
                }),
                data: None,
            }]))
        })
        .next()
        .await;

    // Make request
    let result = language_server
        .code_lens(CodeLensParams {
            text_document: TextDocumentIdentifier {
                uri: lsp::Url::parse("file:///test.rs").unwrap(),
            },
            work_done_progress_params: Default::default(),
            partial_result_params: Default::default(),
        })
        .await
        .unwrap();

    assert_eq!(result.unwrap().len(), 1);
}
```

### Integration Test

```rust
#[gpui::test]
async fn test_code_lens_integration(cx: &mut TestApp) {
    let app_state = init_test(cx);

    app_state
        .fs
        .as_fake()
        .insert_tree(
            "/root",
            json!({
                "test.rs": "fn main() { println!(\"Hello\"); }",
            }),
        )
        .await;

    let project = Project::test(app_state.fs.clone(), ["/root".as_ref()], cx).await;

    let buffer = project
        .update(cx, |project, cx| {
            project.open_local_buffer("/root/test.rs", cx)
        })
        .await
        .unwrap();

    // Request code lenses
    let lenses = project
        .update(cx, |project, cx| {
            project.code_lenses(buffer.clone(), cx)
        })
        .await
        .unwrap();

    assert!(!lenses.is_empty());
}
```

## Common Pitfalls

### 1. Not Checking Capabilities

❌ **Wrong:**
```rust
// Always makes request
let result = language_server.inlay_hints(params).await?;
```

✅ **Correct:**
```rust
// Check capability first
if language_server.capabilities().inlay_hint_provider.is_none() {
    return Ok(vec![]);
}
let result = language_server.inlay_hints(params).await?;
```

### 2. Not Handling Errors Gracefully

❌ **Wrong:**
```rust
let result = language_server.request(params).await.unwrap();
```

✅ **Correct:**
```rust
let result = language_server
    .request(params)
    .await
    .log_err()
    .unwrap_or_default();
```

### 3. Blocking UI Thread

❌ **Wrong:**
```rust
pub fn my_feature(&mut self, cx: &mut Context<Self>) {
    let result = futures::executor::block_on(async_call());  // Blocks UI!
}
```

✅ **Correct:**
```rust
pub fn my_feature(&mut self, cx: &mut Context<Self>) {
    cx.spawn(async move |this, cx| {
        let result = async_call().await;
        this.update(&cx, |this, cx| {
            this.handle_result(result, cx);
        })
    }).detach_and_log_err(cx);
}
```

## PR Checklist

- [ ] Server capability checked before requests
- [ ] Async operations don't block UI
- [ ] Errors handled gracefully
- [ ] UI updates after async operations
- [ ] Tests added
- [ ] Works with multiple language servers
- [ ] Handles server restart
- [ ] Documentation added

## Resources

- [LSP Specification](https://microsoft.github.io/language-server-protocol/)
- [LSP crate source](../../../crates/lsp/)
- [Project LSP integration](../../../crates/project/src/lsp_store.rs)
- [Language definitions](../../../crates/language/)
