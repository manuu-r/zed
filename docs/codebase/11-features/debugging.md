# Debugging Support

**Last Updated:** 2025-11-16

---

## Overview

Debug Adapter Protocol (DAP) support for debugging.

**Path:** `/home/user/zed/crates/dap/` and `/home/user/zed/crates/debugger_ui/`

## DAP Client

```rust
pub struct DebugAdapter {
    adapter_id: DebugAdapterId,
    capabilities: Capabilities,
}
```

## Breakpoints

```rust
pub struct Breakpoint {
    pub path: PathBuf,
    pub line: u32,
    pub condition: Option<String>,
}

// Set breakpoint
project.set_breakpoint(breakpoint, cx)?;
```

## Debug Session

```rust
// Start debugging
let session = project.start_debug(config, cx).await?;

// Step through code
session.step_over(cx).await?;
session.step_into(cx).await?;
session.step_out(cx).await?;

// Continue
session.continue_execution(cx).await?;
```

## Further Reading

- [Project](../03-project/README.md)
- [Editor](../02-editor/README.md)
