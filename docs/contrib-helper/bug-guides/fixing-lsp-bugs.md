# Fixing LSP Bugs

Guide for debugging Language Server Protocol issues.

## Common Issues

### Server Won't Start

**Debug steps:**
1. Check server binary exists
2. Verify server permissions
3. Check initialization options
4. Review server logs

```bash
# Enable LSP logging
RUST_LOG=lsp=debug cargo run
```

### Requests Timeout

**Common causes:**
- Server is busy
- Wrong document URI
- Server crashed
- Network issues (for remote servers)

**Debug:**
```rust
log::debug!("Sending LSP request: {:?}", params);
let result = language_server.request(params).await.log_err();
log::debug!("LSP response: {:?}", result);
```

### Features Don't Work

**Check:**
- Server capabilities
- Client capabilities sent during initialization
- Document synchronization state

## Resources

- [LSP client](../../../crates/lsp/)
- [LSP specification](https://microsoft.github.io/language-server-protocol/)
