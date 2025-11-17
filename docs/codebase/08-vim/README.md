# Vim Crate

**Path:** `/home/user/zed/crates/vim/`
**Purpose:** Vim mode implementation

---

## Overview

Implements Vim keybindings and modal editing.

## Core Types

```rust
pub struct Vim {
    mode: VimMode,
    state: VimState,
    editor: WeakEntity<Editor>,
}

pub enum VimMode {
    Normal,
    Insert,
    Visual { line: bool },
    VisualBlock,
}
```

## Documentation Files

- **[Modes](./modes.md)** - Vim modes
- **[Motions](./motions.md)** - Motion implementation
- **[Operators](./operators.md)** - Operator implementation
- **README.md** - This file

## Further Reading

- [Editor](../02-editor/README.md)
- [Movements](../02-editor/movements.md)
