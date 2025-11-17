# Collaboration Server

**Last Updated:** 2025-11-16

---

## Server Structure

```rust
pub struct Server {
    connections: HashMap<ConnectionId, Connection>,
    projects: HashMap<ProjectId, SharedProject>,
}
```

## Connection Handling

```
Client Connect
  │
  ├─→ Authenticate
  │
  ├─→ Join Project
  │
  ├─→ Sync State
  │
  └─→ Stream Updates
```

## State Synchronization

New clients receive full state:

```rust
// Send initial state
for buffer in project.buffers() {
    send_buffer_state(buffer, connection);
}

for participant in project.participants() {
    send_participant_state(participant, connection);
}
```

## Further Reading

- [Collab README](./README.md)
- [RPC Protocol](./rpc-protocol.md)
