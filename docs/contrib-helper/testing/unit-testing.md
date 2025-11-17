# Unit Testing in Zed

Comprehensive guide to writing effective unit tests for Zed.

## Quick Start

```rust
#[gpui::test]
async fn test_my_feature(cx: &mut TestApp) {
    let entity = cx.build_entity(|cx| MyType::new(cx));

    entity.update(cx, |entity, cx| {
        entity.do_something(cx);
        assert_eq!(entity.result(), expected);
    });
}
```

## GPUI Test Infrastructure

### Test Attribute

Use `#[gpui::test]` for tests that need GPUI:

```rust
#[gpui::test]
async fn test_with_gpui(cx: &mut TestApp) {
    // Test code
}

// Regular Rust test for pure functions
#[test]
fn test_pure_function() {
    // No GPUI needed
}
```

### TestApp Context

`TestApp` is the test version of the application context:

```rust
#[gpui::test]
async fn test_example(cx: &mut TestApp) {
    // Build entities
    let entity = cx.build_entity(|cx| MyType::new(cx));

    // Update application state
    cx.update(|cx| {
        // Modify global state
    });

    // Update entity
    entity.update(cx, |entity, cx| {
        entity.modify(cx);
    });

    // Read entity
    let value = entity.read(cx).value;

    // Spawn async work
    let result = cx.spawn(async |cx| {
        // Async operations
    }).await;
}
```

## Testing Patterns

### Pattern 1: Testing Pure Functions

```rust
#[test]
fn test_text_manipulation() {
    let input = "hello world";
    let result = to_title_case(input);
    assert_eq!(result, "Hello World");
}

#[test]
fn test_range_calculation() {
    let range = calculate_range(Point::new(0, 0), Point::new(5, 10));
    assert_eq!(range.start, Point::new(0, 0));
    assert_eq!(range.end, Point::new(5, 10));
}
```

### Pattern 2: Testing Editor Actions

```rust
#[gpui::test]
async fn test_duplicate_line(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("line 1\nline 2", cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    });

    editor.update(cx, |editor, cx| {
        // Position cursor on line 1
        editor.change_selections(None, cx, |s| {
            s.select_ranges([Point::new(0, 0)..Point::new(0, 0)])
        });

        // Execute action
        editor.duplicate_line(&DuplicateLine, cx);

        // Verify result
        assert_eq!(editor.text(cx), "line 1\nline 1\nline 2");
    });
}
```

### Pattern 3: Testing State Changes

```rust
#[gpui::test]
async fn test_state_update(cx: &mut TestApp) {
    let component = cx.build_entity(|cx| MyComponent::new(cx));

    // Initial state
    assert_eq!(component.read(cx).state, State::Initial);

    // Trigger state change
    component.update(cx, |component, cx| {
        component.transition_to_active(cx);
    });

    // Verify new state
    assert_eq!(component.read(cx).state, State::Active);

    // Observer should be notified
    cx.run_until_parked();
    assert!(component.read(cx).observer_called);
}
```

### Pattern 4: Testing Async Operations

```rust
#[gpui::test]
async fn test_async_load(cx: &mut TestApp) {
    let loader = cx.build_entity(|cx| DataLoader::new(cx));

    // Start async load
    let task = loader.update(cx, |loader, cx| {
        loader.load_data(cx)
    });

    // Wait for completion
    let result = task.await;

    // Verify result
    assert!(result.is_ok());
    assert_eq!(loader.read(cx).data.len(), 10);
}
```

### Pattern 5: Testing Error Handling

```rust
#[gpui::test]
async fn test_error_handling(cx: &mut TestApp) {
    let component = cx.build_entity(|cx| MyComponent::new(cx));

    component.update(cx, |component, cx| {
        // Operation that should fail
        let result = component.invalid_operation(cx);

        // Verify error
        assert!(result.is_err());
        assert_eq!(
            result.unwrap_err().to_string(),
            "Invalid operation"
        );
    });
}
```

## Mocking and Test Fixtures

### Mocking File System

```rust
#[gpui::test]
async fn test_with_fake_fs(cx: &mut TestApp) {
    let fs = FakeFs::new(cx.executor());

    // Create fake file structure
    fs.insert_tree(
        "/test",
        json!({
            "file.rs": "fn main() {}",
            "folder": {
                "nested.rs": "// nested file"
            }
        })
    ).await;

    // Test operations on fake fs
    let content = fs.load("/test/file.rs").await.unwrap();
    assert_eq!(content, "fn main() {}");
}
```

### Creating Test Buffers

```rust
#[gpui::test]
async fn test_buffer_operations(cx: &mut TestApp) {
    let buffer = cx.build_entity(|cx| {
        Buffer::local("initial content", cx)
    });

    buffer.update(cx, |buffer, cx| {
        buffer.edit([(0..0, "prefix ")], None, cx);
        assert_eq!(buffer.text(), "prefix initial content");
    });
}
```

### Creating Test Editors

```rust
#[gpui::test]
async fn test_editor_setup(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("test content", cx);
        let mut editor = Editor::new(
            EditorMode::Full,
            buffer,
            None,
            false,
            cx
        );

        // Configure editor for test
        editor.set_soft_wrap_mode(SoftWrap::None, cx);

        editor
    });

    // Use editor in test
    editor.update(cx, |editor, cx| {
        // Test operations
    });
}
```

## Assertions and Expectations

### Basic Assertions

```rust
// Equality
assert_eq!(actual, expected);
assert_ne!(actual, not_expected);

// Boolean
assert!(condition);
assert!(!condition);

// With custom message
assert_eq!(
    actual,
    expected,
    "Values should match after operation"
);
```

### Testing Editor State

```rust
#[gpui::test]
async fn test_editor_state(cx: &mut TestApp) {
    editor.update(cx, |editor, cx| {
        // Test text content
        assert_eq!(editor.text(cx), "expected");

        // Test selections
        let selections = editor.selections.ranges::<Point>(cx);
        assert_eq!(selections.len(), 1);
        assert_eq!(selections[0], Point::new(0, 0)..Point::new(0, 5));

        // Test cursor position
        let cursor = editor.selections.newest_anchor().head();
        assert_eq!(cursor.to_point(&buffer.read(cx)), Point::new(1, 10));
    });
}
```

### Testing Events

```rust
#[gpui::test]
async fn test_events(cx: &mut TestApp) {
    let emitter = cx.build_entity(|cx| EventEmitter::new(cx));
    let received_events = Arc::new(Mutex::new(Vec::new()));

    // Subscribe to events
    let events_clone = received_events.clone();
    cx.subscribe(&emitter, move |event, _cx| {
        events_clone.lock().push(event.clone());
    }).detach();

    // Trigger event
    emitter.update(cx, |emitter, cx| {
        cx.emit(MyEvent::Happened);
    });

    // Verify event received
    cx.run_until_parked();
    assert_eq!(received_events.lock().len(), 1);
}
```

## Testing Tips

### Test Organization

Group related tests:

```rust
#[cfg(test)]
mod selection_tests {
    use super::*;

    #[gpui::test]
    async fn test_add_selection_above(cx: &mut TestApp) {
        // Test
    }

    #[gpui::test]
    async fn test_add_selection_below(cx: &mut TestApp) {
        // Test
    }
}

#[cfg(test)]
mod movement_tests {
    use super::*;

    #[gpui::test]
    async fn test_move_left(cx: &mut TestApp) {
        // Test
    }

    #[gpui::test]
    async fn test_move_right(cx: &mut TestApp) {
        // Test
    }
}
```

### Helper Functions

Extract common setup:

```rust
fn build_test_editor(text: &str, cx: &mut TestApp) -> Entity<Editor> {
    cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple(text, cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    })
}

#[gpui::test]
async fn test_with_helper(cx: &mut TestApp) {
    let editor = build_test_editor("test", cx);

    editor.update(cx, |editor, cx| {
        // Test
    });
}
```

### Parameterized Tests

Test multiple cases:

```rust
#[gpui::test]
async fn test_case_conversion(cx: &mut TestApp) {
    let cases = vec![
        ("hello world", "Hello World"),
        ("HELLO WORLD", "Hello World"),
        ("hello_world", "Hello_world"),
    ];

    for (input, expected) in cases {
        let editor = build_test_editor(input, cx);

        editor.update(cx, |editor, cx| {
            editor.convert_to_title_case(&ConvertToTitleCase, cx);
            assert_eq!(editor.text(cx), expected);
        });
    }
}
```

## Common Issues

### Issue: Test Times Out

```rust
// ❌ Forgetting to await
#[gpui::test]
async fn test_timeout(cx: &mut TestApp) {
    entity.update(cx, |entity, cx| {
        cx.spawn(async {
            // This never completes
        });
    });
}

// ✅ Properly awaiting
#[gpui::test]
async fn test_correct(cx: &mut TestApp) {
    let task = entity.update(cx, |entity, cx| {
        cx.spawn(async {
            // Work
        })
    });

    task.await;
}
```

### Issue: Entity Already Borrowed

```rust
// ❌ Multiple borrows
entity.update(cx, |entity, cx| {
    let other = entity.other.read(cx);  // Borrows entity
    entity.modify();  // Error: already borrowed
});

// ✅ Release borrow first
entity.update(cx, |entity, cx| {
    let value = entity.other.read(cx).value.clone();
    // Borrow released
    entity.modify_with_value(value);
});
```

## Resources

- [Example tests](../../../crates/editor/src/editor_tests.rs)
- [Test utilities](../../../crates/gpui/src/test.rs)
- [Rust testing guide](https://doc.rust-lang.org/book/ch11-00-testing.html)
