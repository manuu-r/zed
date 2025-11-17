# Vim Motions

**Last Updated:** 2025-11-16

---

## Motion Types

```rust
pub enum Motion {
    Left,
    Down,
    Up,
    Right,
    NextWordStart,
    NextWordEnd,
    PreviousWordStart,
    FirstNonWhitespace,
    StartOfLine,
    EndOfLine,
    Matching,  // %
    FindForward { char, before },
    FindBackward { char, after },
}
```

## Motion Execution

```rust
fn execute_motion(motion: Motion, editor: &mut Editor, cx: &mut Context<Vim>) {
    editor.change_selections(None, window, cx, |s| {
        s.move_cursors_with(|map, cursor, goal| {
            motion.move_cursor(map, cursor, goal)
        });
    });
}
```

## Text Objects

```rust
// Word: w, W
// Sentence: s
// Paragraph: p
// Brackets: (, ), {, }, [, ]
```

## Further Reading

- [Editor Movements](../02-editor/movements.md)
- [Vim README](./README.md)
