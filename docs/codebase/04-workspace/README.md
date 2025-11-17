# Workspace Crate

**Path:** `/home/user/zed/crates/workspace/`
**Purpose:** Window layout and pane management

---

## Overview

Manages window layout with panes, docks, tabs, and workspace items.

## Core Types

```rust
pub struct Workspace {
    panes: Vec<Entity<Pane>>,
    center: PaneGroup,
    left_dock: Entity<Dock>,
    right_dock: Entity<Dock>,
    bottom_dock: Entity<Dock>,
    project: Entity<Project>,
}
```

## Documentation Files

- **[Panes and Docks](./panes-and-docks.md)** - Layout system
- **[Items](./items.md)** - Item trait and lifecycle
- **[Persistence](./persistence.md)** - State saving/restoring
- **README.md** - This file

## Further Reading

- [Editor](../02-editor/README.md)
- [Project](../03-project/README.md)
