# Advanced Topics

Deep dives into Zed's architecture, optimization techniques, and platform-specific development.

## Available Guides

1. **[Architecture Decisions](architecture-decisions.md)** - Why Zed is built the way it is
2. **[Performance Optimization](performance-optimization.md)** - Make Zed faster
3. **[Platform-Specific Development](platform-specific.md)** - macOS, Linux, Windows
4. **[Collaboration Protocol](collaboration-protocol.md)** - Understanding RPC and real-time collaboration

## Who Should Read These

These guides are for contributors who:
- Want to understand Zed's architecture deeply
- Are working on performance-critical code
- Need to implement platform-specific features
- Are contributing to collaboration features

## Prerequisites

Before diving into advanced topics:
- Complete the [Getting Started Guide](../getting-started.md)
- Understand GPUI fundamentals
- Have made at least one successful contribution

## Key Architectural Principles

### 1. Entity-Based State Management

Zed uses `Entity<T>` for managing component state, providing:
- Automatic memory management
- Efficient updates
- Observer pattern support

### 2. Async-First Architecture

All I/O and long-running operations are async:
- Non-blocking UI
- Efficient resource usage
- Responsive user experience

### 3. GPU-Accelerated Rendering

GPUI uses the GPU for rendering:
- Smooth animations
- High performance
- Consistent frame rates

### 4. Language Server Protocol

Deep LSP integration provides:
- Rich language features
- Extensible language support
- Consistent IDE experience

## Resources

- [GPUI Architecture](../../../crates/gpui/README.md)
- [Project Architecture](../../../crates/project/)
- [Collaboration Architecture](../../../crates/collab/)

## Getting Help

For advanced questions:
- GitHub Discussions
- Discord #development channel
- Direct questions in PRs
