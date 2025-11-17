# RPC Protocol

**Last Updated:** 2025-11-16

---

## Message Types

```rust
pub enum Message {
    JoinProject { project_id: u64 },
    UpdateBuffer { buffer_id: u64, operations: Vec<Operation> },
    UpdateSelections { selections: Vec<Selection> },
    // ...
}
```

## CRDT Operations

Buffers use CRDT for conflict-free merging:

```rust
pub struct Operation {
    pub replica_id: ReplicaId,
    pub version: clock::Global,
    pub edit: Edit,
}
```

## Message Flow

```
Client A                Server                Client B
   │                      │                      │
   ├─ Edit Buffer ──────→ │                      │
   │                      ├─ Broadcast ────────→ │
   │                      │                      ├─ Apply Edit
   │                      │                      │
   │                      │ ←──── Edit Buffer ───┤
   │ ←─── Broadcast ──────┤                      │
   ├─ Apply Edit          │                      │
```

## Further Reading

- [Collab README](./README.md)
- [Buffers](../03-project/buffers.md)
