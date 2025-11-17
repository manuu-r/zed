# Zed Contribution Helper Documentation

Welcome to the comprehensive guide for contributing to Zed! This documentation is designed to help you navigate the codebase, understand common patterns, and successfully contribute to the project.

## Table of Contents

- [Getting Started](#getting-started)
- [Common Patterns](#common-patterns)
- [Testing](#testing)
- [Bug Fixing](#bug-fixing)
- [Roadmap Features](#roadmap-features)
- [Advanced Topics](#advanced-topics)

## Overview

Zed is a high-performance code editor built in Rust using GPUI, a custom GPU-accelerated UI framework. Contributing to Zed requires understanding several key concepts:

- **GPUI Framework**: Zed's custom UI framework with entity-based state management
- **Async Architecture**: Extensive use of async Rust for performance
- **Language Server Protocol (LSP)**: Integration with language servers
- **Vim Mode**: A comprehensive Vim emulation layer
- **Collaboration Protocol**: Real-time collaboration features

## Getting Started

**Start here if you're new to Zed development!**

📖 [**Getting Started Guide**](getting-started.md)

This comprehensive guide covers:
- Setting up your development environment
- Building Zed from source
- Running tests
- Understanding the codebase structure
- Making your first contribution
- Where to find good first issues

## Common Patterns

Learn how to implement common features in Zed:

📁 [**Common Patterns Directory**](common-patterns/)

### Quick Links

- [Adding an Editor Action](common-patterns/adding-editor-action.md) - Add new editor commands
- [Creating UI Components](common-patterns/creating-ui-component.md) - Build reusable GPUI components
- [Adding Language Support](common-patterns/adding-language-support.md) - Support new programming languages
- [Implementing Vim Motions](common-patterns/implementing-vim-motion.md) - Extend Vim mode
- [Adding Settings](common-patterns/adding-setting.md) - Add user-configurable options
- [Integrating LSP Features](common-patterns/integrating-lsp-feature.md) - Implement LSP protocol features
- [Creating Panels](common-patterns/creating-panel.md) - Build new dock panels

## Testing

Understand how to test your contributions:

📁 [**Testing Guides**](testing/)

### Quick Links

- [Unit Testing](testing/unit-testing.md) - Write effective unit tests
- [Integration Testing](testing/integration-testing.md) - Test across crates
- [UI Testing](testing/ui-testing.md) - Test GPUI components
- [Vim Testing](testing/vim-testing.md) - Test Vim mode features

## Bug Fixing

Guides for fixing common types of bugs:

📁 [**Bug Fixing Guides**](bug-guides/)

### Quick Links

- [Debugging Crashes](bug-guides/debugging-crashes.md) - Track down panics and crashes
- [Fixing Editor Bugs](bug-guides/fixing-editor-bugs.md) - Selection, cursor, and display issues
- [Fixing LSP Bugs](bug-guides/fixing-lsp-bugs.md) - Language server integration issues
- [Fixing UI Bugs](bug-guides/fixing-ui-bugs.md) - Rendering and layout problems
- [Performance Issues](bug-guides/performance-issues.md) - Profile and optimize

## Roadmap Features

Contributing to larger features:

📁 [**Roadmap Guides**](roadmap/)

### Quick Links

- [Contributing to Roadmap Items](roadmap/contributing-to-roadmap.md) - How to tackle big features
- [Example: New Editor Feature](roadmap/example-new-editor-feature.md) - Template for editor features
- [Example: New Panel](roadmap/example-new-panel.md) - Template for panels
- [Example: Collaboration Feature](roadmap/example-collaboration-feature.md) - Template for collab features

## Advanced Topics

Deep dives into Zed's architecture:

📁 [**Advanced Topics**](advanced/)

### Quick Links

- [Architecture Decisions](advanced/architecture-decisions.md) - Why Zed is built this way
- [Performance Optimization](advanced/performance-optimization.md) - Make Zed faster
- [Platform-Specific Development](advanced/platform-specific.md) - macOS, Linux, Windows
- [Collaboration Protocol](advanced/collaboration-protocol.md) - Understanding RPC

## Contribution Workflow

### 1. Find an Issue

Start with issues labeled:
- `good first issue` - Perfect for newcomers
- `help wanted` - Community contributions welcome
- Check the [top-ranking issues](https://github.com/zed-industries/zed/issues/5393)
- Review the [public roadmap](https://zed.dev/roadmap)

### 2. Understand the Code

Before making changes:
1. Read the relevant guides in this documentation
2. Examine existing similar implementations
3. Run the existing tests to understand expected behavior
4. Use `./script/clippy` to check code quality

### 3. Make Your Changes

Follow these best practices:
- Follow the [Rust coding guidelines](../../CLAUDE.md)
- Write tests for your changes
- Run `./script/clippy` before committing
- Add doc comments for new public APIs
- Keep commits focused and well-described

### 4. Test Your Changes

```bash
# Run specific tests
cargo test -p editor test_name

# Run all tests in a crate
cargo test -p editor

# Run clippy
./script/clippy

# Run Zed locally to test manually
cargo run
```

### 5. Submit a Pull Request

- Write a clear description of what you're solving
- Include screenshots/videos for UI changes
- Reference any related issues
- Be responsive to review feedback

## PR Readiness Checklist

Before submitting your PR, verify:

- [ ] Code follows [CLAUDE.md](../../CLAUDE.md) guidelines
- [ ] No use of deprecated APIs (e.g., `Model<T>`, `View<T>`, `AppContext`)
- [ ] No `.unwrap()` calls on fallible operations
- [ ] Errors are properly propagated with `?` or handled with `.log_err()`
- [ ] Tests added for new functionality
- [ ] All tests pass locally
- [ ] `./script/clippy` passes with no warnings
- [ ] Doc comments added for new public APIs
- [ ] UI changes include screenshots/videos
- [ ] Commit messages are descriptive
- [ ] Code is well-organized and readable

## Common Mistakes to Avoid

### 1. Using Deprecated GPUI APIs

❌ **Wrong:**
```rust
fn old_api(cx: &mut AppContext) {
    let model: Model<MyType> = cx.build_model(...);
}
```

✅ **Correct:**
```rust
fn new_api(cx: &mut App) {
    let entity: Entity<MyType> = cx.build_entity(...);
}
```

### 2. Using `.unwrap()` Instead of Error Propagation

❌ **Wrong:**
```rust
let result = some_operation().unwrap();
```

✅ **Correct:**
```rust
let result = some_operation()?;
// or
let result = some_operation().log_err();
```

### 3. Silently Discarding Errors

❌ **Wrong:**
```rust
let _ = client.request(...).await?;
```

✅ **Correct:**
```rust
client.request(...).await?;
// or
client.request(...).await.log_err();
```

### 4. Creating `mod.rs` Files

❌ **Wrong:**
```
src/some_module/mod.rs
```

✅ **Correct:**
```
src/some_module.rs
```

## Getting Help

If you get stuck:

1. **Search existing issues** - Your question may already be answered
2. **Read the code** - Zed's codebase is well-structured and readable
3. **Ask in discussions** - GitHub Discussions is great for questions
4. **Open a draft PR** - Get feedback early on larger changes
5. **Join the community** - Connect with other contributors

## Code Review Philosophy

From [CONTRIBUTING.md](../../CONTRIBUTING.md), reviewers follow this guidance:

- If the fix/feature is obviously great and code is great → Merge
- If the fix/feature is great but code needs minor improvements → Send feedback or pair
- If the fix/feature needs significant work → Close with explanation

## What We Don't Merge

Please avoid submitting:

- Features that can be extensions (languages, themes)
- New file icons (our icons are hand-designed as a cohesive set)
- Giant refactorings without prior discussion
- Changes without tests (for non-trivial features)
- Purely stylistic changes
- AI-generated code without understanding

## Resources

### Official Documentation

- [Zed User Documentation](https://zed.dev/docs)
- [GPUI Framework](../../crates/gpui/README.md)
- [Vim Mode](../../crates/vim/README.md)
- [Extension Development](https://zed.dev/docs/extensions/developing-extensions)

### Codebase Documentation

- [Development Glossary](../src/development/glossary.md)
- [Codebase Overview](../codebase/) - Detailed crate documentation

### Key Files

- [CLAUDE.md](../../CLAUDE.md) - Rust coding guidelines and GPUI patterns
- [CONTRIBUTING.md](../../CONTRIBUTING.md) - Official contribution guide

## Repository Structure

```
zed/
├── crates/           # All Rust crates
│   ├── gpui/        # UI framework
│   ├── editor/      # Core editor implementation
│   ├── vim/         # Vim mode
│   ├── workspace/   # Window and project management
│   ├── language/    # Language support
│   ├── lsp/         # Language server protocol
│   ├── project/     # Project and file management
│   ├── ui/          # Reusable UI components
│   └── zed/         # Main application entry point
├── docs/            # Documentation
│   ├── src/         # User-facing docs
│   ├── codebase/    # Developer codebase docs
│   └── contrib-helper/  # This contribution guide (you are here!)
├── extensions/      # Extension examples
└── script/          # Development scripts
    ├── clippy       # Run clippy with proper config
    └── ...
```

## Next Steps

1. 📖 **[Read the Getting Started Guide](getting-started.md)** to set up your environment
2. 🔍 **Find an issue** that interests you
3. 📚 **Read the relevant pattern guide** for what you want to implement
4. 💻 **Write code** following the guidelines
5. ✅ **Test thoroughly** using the testing guides
6. 🚀 **Submit your PR** and iterate on feedback

Welcome to the Zed community, and happy coding!
