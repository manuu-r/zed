# Vim Mode Testing

Guide for testing Vim mode features using Neovim-backed tests.

## Overview

Vim tests use a special `NeovimBackedTestContext` that compares Zed's behavior against actual Neovim, ensuring compatibility.

## Quick Example

```rust
#[gpui::test]
async fn test_vim_motion(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("hello ˇworld").await;
    cx.simulate_shared_keystrokes("w").await;
    cx.shared_state().await.assert_eq("hello worldˇ");
}
```

## Neovim-Backed Tests

### Basic Motion Test

```rust
#[gpui::test]
async fn test_word_motion(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("one two threeˇ").await;
    cx.simulate_shared_keystrokes("b").await;
    cx.shared_state().await.assert_eq("one two ˇthree");

    cx.simulate_shared_keystrokes("b").await;
    cx.shared_state().await.assert_eq("one ˇtwo three");
}
```

### Operator Test

```rust
#[gpui::test]
async fn test_delete_word(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("hello ˇworld test").await;
    cx.simulate_shared_keystrokes("d w").await;
    cx.shared_state().await.assert_eq("hello ˇtest");
}
```

### Visual Mode Test

```rust
#[gpui::test]
async fn test_visual_selection(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("ˇhello world").await;
    cx.simulate_shared_keystrokes("v w").await;
    cx.shared_state().await.assert_eq("«hello ˇ»world");
}
```

## Test Markers

- `ˇ` - Cursor position
- `«text»` - Visual selection
- `«textˇ»` - Visual selection with cursor at end

## Cached Tests

For frequently used Vim states:

```rust
#[gpui::test]
async fn test_with_cached_state(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    // Uses cached Neovim state for common scenarios
    cx.set_shared_state_with_cache("ˇtest").await;
}
```

## Resources

- [Vim tests](../../../crates/vim/src/test/)
- [Neovim test context](../../../crates/vim/src/test/neovim_backed_test_context.rs)
