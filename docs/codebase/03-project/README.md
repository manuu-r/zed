# Project Crate

**Path:** `/home/user/zed/crates/project/`
**Purpose:** Project and file management

---

## Overview

The project crate manages workspaces, files, buffers, language servers, and project-wide operations.

### Key Responsibilities

- **Worktree Management**: File watching and directory structure
- **Buffer Management**: Open buffers and their lifecycle
- **LSP Management**: Language server lifecycle and communication
- **Project Search**: Search across all files
- **Git Integration**: Repository status and operations
- **Diagnostics**: Aggregate diagnostics from all language servers
- **Tasks**: Build and run tasks

## Core Types

```rust
pub struct Project {
    buffer_store: Entity<BufferStore>,
    lsp_store: Entity<LspStore>,
    worktree_store: Entity<WorktreeStore>,
    git_store: Entity<GitStore>,
    // ...
}
```

## Documentation Files

- **[Worktrees](./worktrees.md)** - File watching and directory structure
- **[Buffers](./buffers.md)** - Buffer management and MultiBuffer
- **[LSP Store](./lsp-store.md)** - Language server management
- **[Search](./search.md)** - Project-wide search
- **README.md** - This file

## Quick Examples

### Creating a Project

```rust
let project = Project::local(client, node, user_store, languages, fs, cx);
```

### Opening a Buffer

```rust
let buffer = project.open_buffer(path, cx).await?;
```

### Searching

```rust
let results = project.search("query", cx).await?;
```

## Further Reading

- [Editor](../02-editor/README.md)
- [Workspace](../04-workspace/README.md)
- [LSP](../06-lsp/README.md)
