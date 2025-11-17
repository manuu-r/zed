# Vim Modes

**Last Updated:** 2025-11-16

---

## Mode System

```rust
pub enum VimMode {
    Normal,      // Command mode
    Insert,      // Insert text
    Visual { line: bool },  // Visual selection
    VisualBlock, // Block selection
}
```

## Mode Transitions

```
Normal
  │
  ├─→ i → Insert
  ├─→ v → Visual
  ├─→ V → Visual Line
  ├─→ ctrl-v → Visual Block
  
Insert
  │
  └─→ Esc → Normal

Visual
  │
  └─→ Esc → Normal
```

## Implementation

```rust
impl Vim {
    fn switch_mode(&mut self, mode: VimMode, cx: &mut Context<Self>) {
        self.mode = mode;
        self.update_editor_for_mode(cx);
        cx.notify();
    }
}
```

## Further Reading

- [Vim README](./README.md)
- [Operators](./operators.md)
