# Zed Codebase Documentation

Welcome to the comprehensive Zed codebase documentation! This guide will help you understand the architecture, implementation details, and contribution patterns for the Zed code editor.

## 📚 Documentation Structure

This documentation is organized into modules that mirror the crate structure of Zed:

### Core Foundation

1. **[00 - Architecture Overview](./00-overview.md)**
   High-level architecture, design patterns, and how everything fits together

2. **[01 - GPUI Framework](./01-gpui/README.md)**
   The GPU-accelerated UI framework that powers Zed
   - Entity system and state management
   - Rendering pipeline
   - Events and actions
   - Async task execution
   - Window management

### Text Editing Core

3. **[02 - Editor](./02-editor/README.md)**
   The core text editing engine
   - Editor architecture
   - Selection and cursor management
   - Text movements and operations
   - Display mapping (wrapping, folding)
   - LSP integration for intellisense

4. **[03 - Project](./03-project/README.md)**
   File and project management
   - Worktree file watching
   - Buffer management
   - LSP store and language servers
   - Project-wide search
   - Git integration

### Window and UI

5. **[04 - Workspace](./04-workspace/README.md)**
   Window and pane management
   - Workspace layout
   - Pane and dock system
   - Tab management
   - Status bar and toolbars
   - Modal dialogs and notifications

6. **[07 - UI Components](./07-ui/README.md)**
   Reusable UI components
   - Button, Label, Input
   - Icons and theming
   - Lists and trees
   - Context menus and popovers

### Language Support

7. **[05 - Language](./05-language/README.md)**
   Programming language support
   - Language definitions
   - Tree-sitter syntax highlighting
   - Symbol extraction and outline
   - Indentation and formatting

8. **[06 - LSP Client](./06-lsp/README.md)**
   Language Server Protocol implementation
   - LSP process management
   - Request/response handling
   - Capabilities and features
   - Error handling

### Features and Extensions

9. **[08 - Vim Mode](./08-vim/README.md)**
   Vim keybindings and modal editing
   - Mode system (Normal, Insert, Visual)
   - Motions and operators
   - Text objects
   - Registers and macros

10. **[09 - Collaboration](./09-collab/README.md)**
    Real-time collaboration features
    - RPC protocol
    - Collaboration server
    - Shared editing
    - Channels and calls

11. **[10 - Settings & Themes](./10-settings-theme/README.md)**
    Configuration and theming
    - Settings system
    - Theme registry
    - Icon themes
    - User preferences

12. **[11 - Additional Features](./11-features/README.md)**
    Other important features
    - Terminal integration
    - Search and replace
    - Debugger support
    - Git UI

## 🎯 Quick Navigation by Task

### I want to understand...

- **How rendering works** → [GPUI Rendering](./01-gpui/rendering.md)
- **How text editing works** → [Editor Architecture](./02-editor/architecture.md)
- **How files are managed** → [Project Worktrees](./03-project/worktrees.md)
- **How LSP integration works** → [LSP Store](./03-project/lsp-store.md)
- **How windows are organized** → [Workspace Layout](./04-workspace/README.md)
- **How syntax highlighting works** → [Language Highlighting](./05-language/README.md)
- **How Vim mode works** → [Vim Implementation](./08-vim/README.md)

### I want to implement...

- **A new UI component** → [UI Components](./07-ui/README.md)
- **A new editor action** → [Editor Actions](./02-editor/actions.md)
- **Language support** → [Language Definitions](./05-language/README.md)
- **An LSP feature** → [LSP Integration](./06-lsp/README.md)
- **A Vim motion** → [Vim Motions](./08-vim/motions.md)
- **A theme** → [Theme System](./10-settings-theme/themes.md)

## 🛠️ Contribution Guides

For practical guides on contributing to Zed, see the **[Contribution Helper Documentation](../contrib-helper/README.md)**:

- [Getting Started with Contributions](../contrib-helper/getting-started.md)
- [Common Contribution Patterns](../contrib-helper/common-patterns/)
- [Roadmap Implementation Guides](../contrib-helper/roadmap/)
- [Bug Fixing Guides](../contrib-helper/bug-guides/)

## 📖 Reading Guide

### For Complete Beginners

1. Start with [Architecture Overview](./00-overview.md)
2. Read [GPUI Core Concepts](./01-gpui/core-concepts.md)
3. Understand [Entity System](./01-gpui/entity-system.md)
4. Explore [Editor Architecture](./02-editor/architecture.md)
5. Try a [simple contribution](../contrib-helper/getting-started.md)

### For Intermediate Developers

1. Review [Architecture Overview](./00-overview.md)
2. Deep dive into your area of interest (Editor, LSP, UI, etc.)
3. Study [Common Patterns](../contrib-helper/common-patterns/)
4. Pick a [roadmap item](../contrib-helper/roadmap/) to implement

### For Advanced Contributors

1. Skim the architecture for context
2. Focus on specific subsystems you're working on
3. Reference API documentation as needed
4. Contribute to documentation improvements

## 🔍 Key Concepts Across the Codebase

### Entity-Based State Management
Almost all state in Zed is managed through `Entity<T>`, which is a handle to state managed by GPUI. Understanding this pattern is crucial.

**See:** [GPUI Entity System](./01-gpui/entity-system.md)

### Event-Driven Architecture
Components communicate through events emitted via `EventEmitter<E>` trait. Changes trigger notifications and re-renders.

**See:** [GPUI Events](./01-gpui/events-and-actions.md)

### Async by Default
Most operations are asynchronous using `async/await` and GPUI's `Task<R>` type.

**See:** [GPUI Async Tasks](./01-gpui/async-tasks.md)

### Trait-Based Extensibility
Zed uses traits extensively for extensibility: `Render`, `Item`, `ProjectItem`, `SearchableItem`, etc.

**See:** Each module's trait documentation

### Type Safety
Rust's type system is leveraged heavily for correctness. Errors are handled via `Result<T, E>` rather than exceptions.

**See:** [Rust patterns in CLAUDE.md](../../CLAUDE.md)

## 📝 Documentation Conventions

### Code Examples
All code examples are complete and runnable (unless marked as pseudocode).

### File References
File paths are shown relative to `/home/user/zed/` repository root.

Example: `crates/gpui/src/app.rs:100` refers to line 100

### Terminology
See the [Glossary](../development/glossary.md) for Zed-specific terminology.

### Diagrams
```
[Component A] --uses--> [Component B]
[Component A] --emits--> Event
```

## 🤝 Contributing to Documentation

This documentation is a living resource. If you find:

- **Errors or outdated information** - Please submit a correction
- **Missing topics** - Open an issue or PR with additions
- **Unclear explanations** - Suggest improvements
- **New patterns** - Document them for others

## 📚 External Resources

- [The Rust Book](https://doc.rust-lang.org/book/) - Learn Rust
- [Zed User Docs](https://zed.dev/docs) - Using Zed
- [GPUI README](../../crates/gpui/README.md) - GPUI overview
- [Contributing Guide](../../CONTRIBUTING.md) - Contribution guidelines
- [Code of Conduct](https://zed.dev/code-of-conduct) - Community standards

## 🗺️ Repository Layout

```
zed/
├── crates/              # 200+ Rust crates
│   ├── zed/            # Main application
│   ├── gpui/           # UI framework
│   ├── editor/         # Text editing
│   ├── project/        # File management
│   ├── workspace/      # Window management
│   ├── language/       # Language support
│   ├── lsp/            # LSP client
│   ├── vim/            # Vim mode
│   ├── ui/             # UI components
│   ├── collab/         # Collaboration server
│   └── ...             # Many more
├── docs/               # Documentation (you are here!)
├── extensions/         # Extension examples
├── script/             # Build and development scripts
└── assets/             # Themes, icons, etc.
```

## 💡 Tips for Reading Code

1. **Start from the entry point** - `crates/zed/src/main.rs`
2. **Follow the types** - Rust's type system guides you
3. **Use rust-analyzer** - Jump to definitions easily
4. **Read tests** - Tests show how to use APIs
5. **Check CLAUDE.md** - Project-specific Rust guidelines

## 🎓 Learning Path

### Week 1: Foundations
- [ ] Read Architecture Overview
- [ ] Understand GPUI basics
- [ ] Study Entity system
- [ ] Run and build Zed locally

### Week 2: Core Systems
- [ ] Explore Editor implementation
- [ ] Understand Project management
- [ ] Study Workspace layout
- [ ] Read Language support

### Week 3: Advanced Topics
- [ ] LSP integration deep dive
- [ ] Vim mode implementation
- [ ] UI component patterns
- [ ] Collaboration architecture

### Week 4: Contribution
- [ ] Pick a good first issue
- [ ] Implement a small feature
- [ ] Write tests
- [ ] Submit your first PR!

---

**Ready to dive in?** Start with the [Architecture Overview](./00-overview.md) or jump to a specific topic above!

**Want to contribute?** Check out the [Contribution Helper](../contrib-helper/README.md)!

**Questions?** See the [Glossary](../development/glossary.md) or ask in [Zed Discord](https://discord.gg/zed)
