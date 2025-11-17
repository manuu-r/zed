# Workspace Persistence

**Last Updated:** 2025-11-16

---

## Overview

Workspaces save and restore their state between sessions.

## Serialization

```rust
pub struct SerializedWorkspace {
    pub id: WorkspaceId,
    pub location: SerializedWorkspaceLocation,
    pub center_group: SerializedPaneGroup,
    pub bounds: SerializedWindowBounds,
}
```

## Database

Uses SQLite to store workspace state:

```rust
// Save workspace
workspace_db.save_workspace(serialized_workspace);

// Restore workspace
let workspace = workspace_db.workspace(id)?;
```

## Item Serialization

Items implement serialization:

```rust
impl SerializableItem for Editor {
    fn serialize(&self, cx: &App) -> Option<ItemData> {
        Some(ItemData {
            kind: "Editor".to_string(),
            data: serde_json::to_value(self.buffer.path())?,
        })
    }
}
```

## Further Reading

- [Workspace README](./README.md)
