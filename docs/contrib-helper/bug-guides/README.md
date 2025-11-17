# Bug Fixing Guides

Comprehensive guides for debugging and fixing common types of bugs in Zed.

## Available Guides

1. **[Debugging Crashes](debugging-crashes.md)** - Track down panics and crashes
2. **[Fixing Editor Bugs](fixing-editor-bugs.md)** - Selection, cursor, and display issues
3. **[Fixing LSP Bugs](fixing-lsp-bugs.md)** - Language server integration problems
4. **[Fixing UI Bugs](fixing-ui-bugs.md)** - Rendering and layout issues
5. **[Performance Issues](performance-issues.md)** - Profiling and optimization

## General Debugging Tips

### Enable Logging

```bash
# Run with debug logging
RUST_LOG=debug cargo run

# Filter to specific crate
RUST_LOG=editor=debug cargo run

# Multiple modules with different levels
RUST_LOG=editor=debug,vim=trace cargo run
```

### Using Debug Prints

```rust
// Use eprintln! for debugging
eprintln!("Debug: value = {:?}", value);

// Or use log macros
log::debug!("Processing: {:?}", data);
log::info!("Operation completed");
log::warn!("Unexpected state: {:?}", state);
```

### Running with Debugger

```bash
# Build with debug info
cargo build

# Run in lldb (macOS/Linux)
rust-lldb target/debug/zed

# Run in gdb (Linux)
rust-gdb target/debug/zed
```

## Quick Reference

### Common Error Patterns

**Panic on unwrap:**
```rust
// ❌ Can panic
let value = option.unwrap();

// ✅ Handle gracefully
let value = option?;
// or
let value = option.log_err()?;
```

**Entity borrow conflicts:**
```rust
// ❌ Multiple borrows
entity.update(cx, |e, cx| {
    let other = e.other.read(cx);  // Borrows
    e.modify();  // Error: already borrowed
});

// ✅ Release borrow
entity.update(cx, |e, cx| {
    let value = e.other.read(cx).value.clone();
    e.modify();
});
```

## Resources

- [Rust error handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
- [GPUI debugging tips](../../../crates/gpui/README.md)
