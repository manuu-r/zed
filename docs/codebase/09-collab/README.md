# Collaboration Crate

**Path:** `/home/user/zed/crates/collab/` and `/home/user/zed/crates/rpc/`
**Purpose:** Real-time collaboration

---

## Overview

Enables real-time collaborative editing with cursor sharing and CRDT-based conflict resolution.

## Core Types

```rust
pub struct CollaborationHub {
    client: Arc<Client>,
    projects: HashMap<ProjectId, SharedProject>,
}
```

## Documentation Files

- **[RPC Protocol](./rpc-protocol.md)** - Message protocol
- **[Server Architecture](./server-architecture.md)** - Server design
- **README.md** - This file

## Further Reading

- [Project](../03-project/README.md)
- [Editor](../02-editor/README.md)
