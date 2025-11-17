# Collaboration Protocol

Understanding Zed's real-time collaboration architecture.

## Overview

Zed's collaboration uses a client-server architecture with RPC for real-time communication.

## Architecture

```
Client A ←→ Collab Server ←→ Client B
    ↓            ↓             ↓
  Local      Authority      Local
  State        State        State
```

### Components

1. **Collab Server** - Central authority for:
   - User authentication
   - Project coordination
   - Message routing
   - State synchronization

2. **RPC Client** - Each Zed instance:
   - Connects to server
   - Sends/receives messages
   - Maintains local state

## RPC Protocol

### Message Types

```rust
// Request-response pattern
pub enum Request {
    JoinProject { project_id: u64 },
    OpenBuffer { buffer_id: u64 },
    // ...
}

pub enum Response {
    JoinProjectResponse { /* ... */ },
    OpenBufferResponse { /* ... */ },
    // ...
}

// One-way notifications
pub enum Notification {
    BufferEdited { buffer_id: u64, edits: Vec<Edit> },
    ProjectUpdated { /* ... */ },
    // ...
}
```

### Sending Requests

```rust
impl CollabClient {
    pub async fn join_project(&self, project_id: u64) -> Result<ProjectState> {
        let response = self.client.request(proto::JoinProject {
            project_id,
        }).await?;

        Ok(response.into())
    }
}
```

### Handling Messages

```rust
impl CollabServer {
    fn handle_join_project(
        &mut self,
        request: proto::JoinProject,
        user_id: UserId,
    ) -> Result<proto::JoinProjectResponse> {
        // Validate access
        self.check_access(user_id, request.project_id)?;

        // Get project state
        let project = self.get_project(request.project_id)?;

        // Notify other collaborators
        self.broadcast(proto::UserJoined {
            user_id,
            project_id: request.project_id,
        });

        Ok(proto::JoinProjectResponse {
            // Project state
        })
    }
}
```

## State Synchronization

### Operational Transformation

Zed uses OT for concurrent edits:

```rust
fn apply_edit(
    &mut self,
    edit: Edit,
    lamport_timestamp: u64,
) -> Result<()> {
    // Transform against concurrent edits
    let transformed = self.transform_edit(edit, lamport_timestamp);

    // Apply to local buffer
    self.buffer.apply_edit(transformed)?;

    // Broadcast to peers
    self.broadcast_edit(transformed);

    Ok(())
}
```

### Lamport Timestamps

For ordering concurrent operations:

```rust
struct Edit {
    range: Range<Anchor>,
    text: String,
    lamport_timestamp: u64,
}
```

## Testing Collaboration

### Multi-Client Tests

```rust
#[gpui::test]
async fn test_collaborative_editing(cx: &mut TestApp) {
    let server = TestServer::start(cx).await;

    let client_a = server.create_client(cx, "user_a").await;
    let client_b = server.create_client(cx, "user_b").await;

    // Client A creates project
    let (project_a, _) = client_a.build_local_project("/root", cx).await;

    // Client B joins
    let project_id = project_a.read_with(cx, |p, _| p.remote_id().unwrap());
    let project_b = client_b.build_remote_project(project_id, cx).await;

    // Test concurrent edits
    // ...
}
```

## Security

### Authentication

```rust
impl CollabServer {
    fn authenticate(&self, token: &str) -> Result<UserId> {
        // Verify JWT token
        let claims = verify_token(token)?;
        Ok(claims.user_id)
    }
}
```

### Authorization

```rust
impl CollabServer {
    fn check_access(&self, user_id: UserId, project_id: u64) -> Result<()> {
        let project = self.get_project(project_id)?;

        if !project.has_access(user_id) {
            return Err(anyhow!("Access denied"));
        }

        Ok(())
    }
}
```

## Resources

- [RPC Protocol](../../../crates/rpc/proto/)
- [Collab Server](../../../crates/collab/src/)
- [Collab Client](../../../crates/client/src/)
