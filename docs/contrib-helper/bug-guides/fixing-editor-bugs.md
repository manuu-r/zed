# Fixing Editor Bugs

Guide for fixing common editor-related bugs.

## Common Issues

### Selection Problems

**Issue:** Selections are in wrong position

**Debug:**
```rust
eprintln!("Selections: {:?}", editor.selections.ranges::<Point>(cx));
eprintln!("Display map: {:?}", editor.display_map.read(cx));
```

**Common causes:**
- Display coordinates vs buffer coordinates mismatch
- Not accounting for soft wraps
- Forgetting to update after buffer edits

### Cursor Positioning

**Issue:** Cursor jumps to wrong location

**Check:**
- Are you using `Point` vs `DisplayPoint` correctly?
- Did you account for multi-byte characters?
- Are fold points updated?

### Text Rendering

**Issue:** Text appears wrong

**Debug:**
```rust
eprintln!("Buffer text: {}", buffer.text());
eprintln!("Display text: {}", display_map.text());
```

## Resources

- [Editor source](../../../crates/editor/src/editor.rs)
- [Display map](../../../crates/editor/src/display_map/)
