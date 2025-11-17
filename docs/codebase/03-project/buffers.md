# Project Buffers

**Last Updated:** 2025-11-16

---

## Overview

Buffers are in-memory representations of file contents.

## Buffer vs MultiBuffer

- **Buffer**: Single file
- **MultiBuffer**: Multiple excerpts (used by editor)

```rust
pub struct Buffer {
    text: Rope,
    version: clock::Global,
    operations: SumTree<Operation>,
}

pub struct MultiBuffer {
    buffers: HashMap<BufferId, Entity<Buffer>>,
    excerpts: SumTree<Excerpt>,
}
```

## Opening Buffers

```rust
let buffer = project.open_buffer(path, cx).await?;
```

## Buffer Lifecycle

```
Create Buffer
  │
  ├─→ Load from disk
  │
  ├─→ Apply edits
  │
  ├─→ Save to disk
  │
  └─→ Close (drop when no references)
```

## CRDT for Collaboration

Buffers use CRDT (Conflict-free Replicated Data Type) for collaborative editing:

```rust
buffer.edit([(range, "text")], cx);
// Generates CRDT operation
// Sent to collaborators
```

## Further Reading

- [Project README](./README.md)
- [Editor](../02-editor/README.md)
