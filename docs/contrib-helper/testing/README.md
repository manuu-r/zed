# Testing in Zed

This directory contains comprehensive guides for testing your contributions to Zed.

## Overview

Testing is a critical part of contributing to Zed. All non-trivial changes require tests, and having good test coverage makes your PR more likely to be merged.

## Available Guides

### Core Testing

1. **[Unit Testing](unit-testing.md)** - Writing effective unit tests
   - GPUI test infrastructure
   - Async testing patterns
   - Mocking and test fixtures
   - Test organization

2. **[Integration Testing](integration-testing.md)** - Testing across crates
   - Cross-crate testing
   - LSP integration tests
   - Collaboration tests
   - File system tests

3. **[UI Testing](ui-testing.md)** - Testing GPUI components
   - Visual testing
   - Interaction testing
   - Component behavior tests

### Specialized Testing

4. **[Vim Testing](vim-testing.md)** - Testing Vim mode
   - Neovim-backed tests
   - Cached test infrastructure
   - Common Vim test patterns

## Quick Reference

### Running Tests

```bash
# Run all tests (slow!)
cargo test

# Run tests in a specific crate
cargo test -p editor

# Run a specific test
cargo test -p editor test_selection_movement

# Run tests matching a pattern
cargo test -p vim test_motion

# Run with output visible
cargo test test_name -- --nocapture

# Run with logging
RUST_LOG=debug cargo test test_name -- --nocapture
```

### Using Nextest (Faster)

```bash
# Install nextest
cargo install cargo-nextest

# Run tests with nextest
cargo nextest run

# Run specific crate
cargo nextest run -p editor
```

## Test Requirements

Before submitting a PR:

- [ ] All new functionality has tests
- [ ] All tests pass locally
- [ ] Tests are deterministic (no flakiness)
- [ ] Tests have descriptive names
- [ ] Edge cases are covered
- [ ] Error cases are tested

## Test Organization

### Test Location

```
crates/
├── editor/
│   ├── src/
│   │   ├── editor.rs
│   │   ├── editor_tests.rs      # Unit tests for editor
│   │   └── movement_tests.rs    # Unit tests for movement
│   └── tests/
│       └── integration_test.rs  # Integration tests
```

### Inline vs Separate Files

**Inline tests:**
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_something() {
        // Test code
    }
}
```

**Separate test files:**
```rust
// File: editor_tests.rs
use crate::*;

#[gpui::test]
async fn test_editor_feature(cx: &mut TestApp) {
    // Test code
}
```

## Test Naming Conventions

Use descriptive names that explain what is being tested:

```rust
// ✅ Good names
#[test]
fn test_duplicate_line_with_empty_selection()
#[test]
fn test_code_action_when_language_server_unavailable()
#[test]
fn test_vim_motion_with_count()

// ❌ Bad names
#[test]
fn test1()
#[test]
fn test_feature()
#[test]
fn test_bug_fix()
```

## Common Test Patterns

### Pattern 1: Basic Unit Test

```rust
#[test]
fn test_feature() {
    let input = "test";
    let result = my_function(input);
    assert_eq!(result, expected);
}
```

### Pattern 2: GPUI Test

```rust
#[gpui::test]
async fn test_gpui_feature(cx: &mut TestApp) {
    let entity = cx.build_entity(|cx| MyType::new(cx));

    entity.update(cx, |entity, cx| {
        entity.do_something(cx);
        assert_eq!(entity.state, expected_state);
    });
}
```

### Pattern 3: Editor Test

```rust
#[gpui::test]
async fn test_editor_action(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("initial text", cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    });

    editor.update(cx, |editor, cx| {
        editor.perform_action(&Action, cx);
        assert_eq!(editor.text(cx), "expected text");
    });
}
```

### Pattern 4: Async Test

```rust
#[gpui::test]
async fn test_async_operation(cx: &mut TestApp) {
    let result = cx.spawn(async |cx| {
        // Async work
        fetch_data().await
    }).await;

    assert!(result.is_ok());
}
```

## Testing Best Practices

### 1. Test One Thing at a Time

```rust
// ✅ Good - focused test
#[test]
fn test_add_selection_below() {
    // Test only this feature
}

// ❌ Bad - testing multiple features
#[test]
fn test_selections_and_movements() {
    // Tests too many things
}
```

### 2. Use Descriptive Assertions

```rust
// ✅ Good - clear assertion
assert_eq!(
    editor.text(cx),
    "expected text",
    "Text should be duplicated after DuplicateLine action"
);

// ❌ Bad - unclear what failed
assert!(result);
```

### 3. Test Edge Cases

```rust
#[gpui::test]
async fn test_duplicate_line_at_end_of_file(cx: &mut TestApp) {
    // Edge case: last line
}

#[gpui::test]
async fn test_duplicate_line_with_empty_file(cx: &mut TestApp) {
    // Edge case: empty file
}

#[gpui::test]
async fn test_duplicate_line_with_multiple_cursors(cx: &mut TestApp) {
    // Edge case: multiple selections
}
```

### 4. Clean Up Resources

```rust
#[gpui::test]
async fn test_with_cleanup(cx: &mut TestApp) {
    // Setup
    let temp_dir = TempDir::new()?;

    // Test
    do_something(&temp_dir);

    // Cleanup happens automatically when temp_dir is dropped
}
```

## Debugging Tests

### Print Debugging

```rust
#[gpui::test]
async fn test_debug(cx: &mut TestApp) {
    eprintln!("Debug value: {:?}", value);

    editor.update(cx, |editor, cx| {
        eprintln!("Editor text: {}", editor.text(cx));
    });
}
```

### Running with Output

```bash
# See println!/eprintln! output
cargo test test_name -- --nocapture

# With logging
RUST_LOG=debug cargo test test_name -- --nocapture
```

### Isolating Failures

```bash
# Run only failing test
cargo test test_name

# Run with backtrace
RUST_BACKTRACE=1 cargo test test_name
```

## Continuous Integration

Tests run automatically on:
- Every pull request
- Every commit to main
- Nightly builds

Make sure your tests pass in CI, not just locally!

## Common Test Failures

### Flaky Tests

Avoid tests that sometimes pass, sometimes fail:
- Don't rely on timing
- Don't depend on external resources
- Avoid race conditions
- Use deterministic test data

### Platform-Specific Issues

Test on the target platform:
- macOS-specific tests
- Linux-specific tests (Wayland vs X11)
- Windows-specific tests

## Resources

- [Rust testing documentation](https://doc.rust-lang.org/book/ch11-00-testing.html)
- [GPUI test utilities](../../../crates/gpui/src/test.rs)
- [Editor tests examples](../../../crates/editor/src/editor_tests.rs)
- [Vim tests examples](../../../crates/vim/src/test/)

## Next Steps

1. **Read the guide for your test type**:
   - [Unit Testing](unit-testing.md) for most tests
   - [Integration Testing](integration-testing.md) for cross-crate tests
   - [UI Testing](ui-testing.md) for component tests
   - [Vim Testing](vim-testing.md) for Vim mode tests

2. **Write your tests** following the patterns

3. **Verify tests pass** locally and in CI

4. **Include in your PR** with clear descriptions
