# Integration Testing

Guide for testing features that span multiple crates or require complex setup.

## Overview

Integration tests verify that different parts of Zed work together correctly. Use these when testing:
- LSP integration
- Collaboration features
- File system operations
- Cross-crate functionality

## Quick Example

```rust
#[gpui::test]
async fn test_lsp_integration(cx: &mut TestApp) {
    let app_state = init_test(cx);

    app_state.fs.as_fake().insert_tree("/root", json!({
        "test.rs": "fn main() {}"
    })).await;

    let project = Project::test(app_state.fs.clone(), ["/root".as_ref()], cx).await;
    let buffer = project.update(cx, |project, cx| {
        project.open_local_buffer("/root/test.rs", cx)
    }).await.unwrap();

    // Test LSP features
    let completions = project.update(cx, |project, cx| {
        project.completions(&buffer, Point::new(0, 5), cx)
    }).await.unwrap();

    assert!(!completions.is_empty());
}
```

## Test Structure

Place integration tests in `crates/<name>/tests/`:

```
crates/editor/
├── src/
│   └── lib.rs
└── tests/
    ├── integration_test.rs
    └── lsp_integration_test.rs
```

## Common Patterns

### Testing with Projects

```rust
#[gpui::test]
async fn test_project_feature(cx: &mut TestApp) {
    let fs = FakeFs::new(cx.executor());
    fs.insert_tree("/root", json!({
        "src/main.rs": "fn main() {}",
        "Cargo.toml": "[package]\nname = \"test\""
    })).await;

    let project = Project::test(fs.clone(), ["/root".as_ref()], cx).await;

    // Test project operations
    let worktree = project.read(cx).worktrees().next().unwrap();
    assert_eq!(worktree.read(cx).root_name(), "root");
}
```

### Testing LSP Features

```rust
#[gpui::test]
async fn test_goto_definition(cx: &mut TestApp) {
    let (language_server, fake_server) = LanguageServer::fake(cx).await;

    fake_server
        .handle_request::<request::GotoDefinition, _, _>(|params, _| async move {
            Ok(Some(GotoDefinitionResponse::Scalar(Location {
                uri: params.text_document_position_params.text_document.uri,
                range: lsp::Range::new(
                    lsp::Position::new(5, 0),
                    lsp::Position::new(5, 10)
                ),
            })))
        })
        .next()
        .await;

    // Test goto definition
    let result = language_server
        .goto_definition(params)
        .await
        .unwrap();

    assert!(result.is_some());
}
```

### Testing Collaboration

```rust
#[gpui::test]
async fn test_collaboration_editing(cx: &mut TestApp) {
    let server = TestServer::start(cx).await;
    let client_a = server.create_client(cx, "user_a").await;
    let client_b = server.create_client(cx, "user_b").await;

    let (project_a, _) = client_a.build_local_project("/root", cx).await;
    let project_id = project_a.read_with(cx, |project, _| project.remote_id().unwrap());

    // Client B joins
    let project_b = client_b.build_remote_project(project_id, cx).await;

    // Test collaborative editing
    let buffer_a = project_a
        .update(cx, |p, cx| p.open_buffer((worktree_id, "file.rs"), cx))
        .await
        .unwrap();

    buffer_a.update(cx, |buffer, cx| {
        buffer.edit([(0..0, "Hello")], None, cx);
    });

    // Verify client B sees the edit
    cx.executor().run_until_parked();
    let buffer_b = project_b
        .update(cx, |p, cx| p.open_buffer((worktree_id, "file.rs"), cx))
        .await
        .unwrap();

    assert_eq!(buffer_b.read_with(cx, |b, _| b.text()), "Hello");
}
```

## Resources

- [Project tests](../../../crates/project/src/project_tests.rs)
- [LSP tests](../../../crates/lsp/src/lsp_tests.rs)
- [Collab tests](../../../crates/collab/src/tests/)
