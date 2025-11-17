# Template: Adding Collaboration Features

Template for features involving real-time collaboration.

## Feature: [Collaboration Feature Name]

### Overview

Description of the collaborative feature.

### Protocol Changes

#### New Messages

```rust
pub enum Message {
    NewFeatureRequest(NewFeatureRequest),
    NewFeatureResponse(NewFeatureResponse),
}
```

#### RPC Methods

- `request_feature()` - Client requests feature
- `broadcast_update()` - Server broadcasts to all clients

### Client Implementation

```rust
impl CollaborationFeature {
    pub async fn perform_action(&self, cx: &mut AsyncApp) -> Result<()> {
        // Send request to server
        let response = self.client.request(Request {
            // ...
        }).await?;

        // Handle response
        self.apply_response(response, cx)?;

        Ok(())
    }
}
```

### Server Implementation

```rust
impl CollabServer {
    fn handle_feature_request(
        &mut self,
        request: Request,
        cx: &mut Context,
    ) -> Result<Response> {
        // Validate request
        // Update server state
        // Broadcast to other clients
        Ok(response)
    }
}
```

### Testing

- Single client tests
- Multi-client tests
- Conflict resolution tests
- Network interruption tests

### Security Considerations

- Authentication
- Authorization
- Data validation
- Rate limiting

## Resources

- [RPC protocol](../../../crates/rpc/)
- [Collab server](../../../crates/collab/)
