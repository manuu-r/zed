# Project Worktrees

**Last Updated:** 2025-11-16

---

## Overview

Worktrees represent directories in the project with file watching.

## Worktree Structure

```rust
pub struct Worktree {
    entries: SumTree<Entry>,
    repository: Option<RepositoryWorkDirectory>,
}

pub struct Entry {
    path: Arc<Path>,
    is_file: bool,
    mtime: SystemTime,
    // ...
}
```

## File Watching

Worktrees watch for file system changes:

```rust
project.add_worktree(path, cx).await?;
// Automatically watches for changes
```

## Git Integration

Each worktree tracks git status:

```rust
let status = worktree.git_status(&path);
```

## Further Reading

- [Project README](./README.md)
- [Git Store](./git-store.md)
