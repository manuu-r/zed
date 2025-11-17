# Fixing UI Bugs

Guide for debugging GPUI rendering and layout issues.

## Common Issues

### Layout Problems

**Issue:** Elements in wrong position

**Check:**
- Flexbox direction (`flex_row` vs `flex_col`)
- Justify and align settings
- Size constraints (`w_full`, `h_full`)

**Debug:**
```rust
div()
    .debug()  // Adds visual debugging
    .flex()
    .child(/* ... */)
```

### Rendering Issues

**Issue:** Elements don't appear

**Common causes:**
- Missing `cx.notify()` after state change
- Element has zero size
- Z-index issues
- Hidden by parent

**Debug:**
```rust
impl Render for MyComponent {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        eprintln!("Rendering with state: {:?}", self.state);
        div().child(/* ... */)
    }
}
```

### Theme Issues

**Problem:** Colors don't match theme

**Solution:**
```rust
// ❌ Hardcoded color
div().bg(gpui::red())

// ✅ Theme color
div().bg(cx.theme().colors().background)
```

## Resources

- [GPUI documentation](../../../crates/gpui/README.md)
- [UI components](../../../crates/ui/src/components/)
