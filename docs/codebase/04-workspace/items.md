# Workspace Items

**Last Updated:** 2025-11-16

---

## Item Trait

```rust
pub trait Item {
    fn tab_content(&self, window: &Window, cx: &App) -> AnyElement;
    fn serialized_item_kind() -> Option<&'static str>;
    fn deserialize(
        project: Entity<Project>,
        workspace: WeakEntity<Workspace>,
        data: ItemData,
        cx: &mut App,
    ) -> Task<Result<Entity<Self>>>;
}
```

## Common Items

- **Editor**: Text editor
- **Terminal**: Integrated terminal
- **ProjectSearch**: Search results
- **ImageViewer**: Image display

## Item Lifecycle

```
Create Item
  │
  ├─→ Add to pane
  │
  ├─→ Activate (show in tab)
  │
  ├─→ User interaction
  │
  └─→ Close (remove from pane)
```

## Further Reading

- [Workspace README](./README.md)
- [Editor](../02-editor/README.md)
