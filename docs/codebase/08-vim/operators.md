# Vim Operators

**Last Updated:** 2025-11-16

---

## Operator Types

```rust
pub enum Operator {
    Delete,    // d
    Change,    // c
    Yank,      // y
    Indent,    // >
    Outdent,   // <
}
```

## Operator + Motion

```
d w  → Delete word
c $  → Change to end of line
y y  → Yank line
> >  → Indent line
```

## Implementation

```rust
fn execute_operator(operator: Operator, motion: Motion, editor: &mut Editor, cx: &mut Context<Vim>) {
    let range = motion.calculate_range(editor, cx);
    
    match operator {
        Operator::Delete => editor.delete_range(range, cx),
        Operator::Change => {
            editor.delete_range(range, cx);
            self.switch_mode(VimMode::Insert, cx);
        }
        Operator::Yank => editor.copy_range(range, cx),
        // ...
    }
}
```

## Further Reading

- [Vim README](./README.md)
- [Modes](./modes.md)
