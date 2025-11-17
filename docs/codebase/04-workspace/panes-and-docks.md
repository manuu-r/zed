# Panes and Docks

**Last Updated:** 2025-11-16

---

## Pane System

```rust
pub struct Pane {
    items: Vec<Box<dyn ItemHandle>>,
    active_item_index: usize,
}
```

Panes contain tabs (items):
- Editors
- Terminals
- Search results
- Custom items

## PaneGroup

Recursive split layout:

```rust
pub enum Member {
    Pane(Entity<Pane>),
    Axis(PaneAxis),
}

pub struct PaneAxis {
    axis: Axis, // Horizontal or Vertical
    members: Vec<Member>,
}
```

## Docks

Side panels:

```rust
pub struct Dock {
    position: DockPosition, // Left, Right, Bottom
    panels: Vec<Entity<dyn PanelHandle>>,
    visible: bool,
}
```

## Further Reading

- [Workspace README](./README.md)
- [Items](./items.md)
