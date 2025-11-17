# Zed Documentation Index

## 📚 Complete Documentation Guide for Contributors

This is your central hub for all Zed documentation. Whether you're learning the codebase or ready to contribute, start here!

---

## 🎯 Quick Start Paths

### "I'm brand new to Zed development"
1. **[Beginner's Guide](./codebase/README.md)** - Start here!
2. **[Architecture Overview](./codebase/00-overview.md)** - Understand how Zed works
3. **[Getting Started](./contrib-helper/getting-started.md)** - Set up your environment
4. **[Your First Contribution](./contrib-helper/getting-started.md#your-first-contribution)** - Make your first PR

### "I want to understand a specific subsystem"
- **GPUI (UI Framework)** → [GPUI Documentation](./codebase/01-gpui/README.md)
- **Text Editing** → [Editor Documentation](./codebase/02-editor/README.md)
- **File Management** → [Project Documentation](./codebase/03-project/README.md)
- **Window Layout** → [Workspace Documentation](./codebase/04-workspace/README.md)
- **Language Support** → [Language Documentation](./codebase/05-language/README.md)
- **LSP Integration** → [LSP Documentation](./codebase/06-lsp/README.md)
- **UI Components** → [UI Documentation](./codebase/07-ui/README.md)
- **Vim Mode** → [Vim Documentation](./codebase/08-vim/README.md)
- **Collaboration** → [Collab Documentation](./codebase/09-collab/README.md)
- **Settings & Themes** → [Settings Documentation](./codebase/10-settings-theme/README.md)
- **Features** → [Features Documentation](./codebase/11-features/README.md)

### "I want to implement something specific"
- **Add keyboard shortcut** → [Adding Editor Action](./contrib-helper/common-patterns/adding-editor-action.md)
- **Create UI component** → [Creating UI Component](./contrib-helper/common-patterns/creating-ui-component.md)
- **Add language support** → [Adding Language Support](./contrib-helper/common-patterns/adding-language-support.md)
- **Implement Vim motion** → [Implementing Vim Motion](./contrib-helper/common-patterns/implementing-vim-motion.md)
- **Add user setting** → [Adding Setting](./contrib-helper/common-patterns/adding-setting.md)
- **Integrate LSP feature** → [Integrating LSP Feature](./contrib-helper/common-patterns/integrating-lsp-feature.md)
- **Create panel** → [Creating Panel](./contrib-helper/common-patterns/creating-panel.md)

### "I'm fixing a bug"
- **Crash or panic** → [Debugging Crashes](./contrib-helper/bug-guides/debugging-crashes.md)
- **Editor issue** → [Fixing Editor Bugs](./contrib-helper/bug-guides/fixing-editor-bugs.md)
- **LSP problem** → [Fixing LSP Bugs](./contrib-helper/bug-guides/fixing-lsp-bugs.md)
- **UI rendering** → [Fixing UI Bugs](./contrib-helper/bug-guides/fixing-ui-bugs.md)
- **Performance** → [Performance Issues](./contrib-helper/bug-guides/performance-issues.md)

### "I need to write tests"
- **Unit tests** → [Unit Testing](./contrib-helper/testing/unit-testing.md)
- **Integration tests** → [Integration Testing](./contrib-helper/testing/integration-testing.md)
- **UI tests** → [UI Testing](./contrib-helper/testing/ui-testing.md)
- **Vim tests** → [Vim Testing](./contrib-helper/testing/vim-testing.md)

---

## 📖 Complete Documentation Map

### I. Codebase Architecture Documentation
**Location:** `/home/user/zed/docs/codebase/`

Comprehensive technical documentation explaining how Zed is built:

#### 00. Foundation
- [Architecture Overview](./codebase/00-overview.md) - System design, patterns, data flow

#### 01. GPUI - UI Framework
- [GPUI Overview](./codebase/01-gpui/README.md)
- [Core Concepts](./codebase/01-gpui/core-concepts.md) - App, Window, Context, Entity
- [Entity System](./codebase/01-gpui/entity-system.md) - State management deep dive
- [Rendering](./codebase/01-gpui/rendering.md) - Rendering pipeline, elements, layout
- [Events & Actions](./codebase/01-gpui/events-and-actions.md) - Input handling, keyboard
- [Async Tasks](./codebase/01-gpui/async-tasks.md) - Task execution, spawning
- [Window Management](./codebase/01-gpui/window-management.md) - Window lifecycle

#### 02. Editor - Text Editing
- [Editor Overview](./codebase/02-editor/README.md)
- [Architecture](./codebase/02-editor/architecture.md) - Editor design, components
- [Selections](./codebase/02-editor/selections.md) - Selection system, multi-cursor
- [Movements](./codebase/02-editor/movements.md) - Text navigation, motions
- [Display Mapping](./codebase/02-editor/display-mapping.md) - Coordinates, wrapping, folding
- [LSP Integration](./codebase/02-editor/lsp-integration.md) - IntelliSense features
- [Actions](./codebase/02-editor/actions.md) - Editor commands

#### 03. Project - File Management
- [Project Overview](./codebase/03-project/README.md)
- [Worktrees](./codebase/03-project/worktrees.md) - File watching, directory trees
- [Buffers](./codebase/03-project/buffers.md) - Buffer management, MultiBuffer
- [LSP Store](./codebase/03-project/lsp-store.md) - Language server coordination
- [Search](./codebase/03-project/search.md) - Project-wide search

#### 04. Workspace - Window Management
- [Workspace Overview](./codebase/04-workspace/README.md)
- [Panes & Docks](./codebase/04-workspace/panes-and-docks.md) - Layout system
- [Items](./codebase/04-workspace/items.md) - Tab system, item lifecycle
- [Persistence](./codebase/04-workspace/persistence.md) - Session restoration

#### 05. Language - Language Support
- [Language Overview](./codebase/05-language/README.md)
- [Tree-sitter](./codebase/05-language/tree-sitter.md) - Syntax parsing
- [Highlighting](./codebase/05-language/highlighting.md) - Syntax coloring
- [Language Config](./codebase/05-language/language-config.md) - Language definitions

#### 06. LSP - Language Server Protocol
- [LSP Overview](./codebase/06-lsp/README.md)
- [Protocol](./codebase/06-lsp/protocol.md) - LSP implementation
- [Capabilities](./codebase/06-lsp/capabilities.md) - Feature support
- [Adapters](./codebase/06-lsp/adapters.md) - Language-specific adapters

#### 07. UI - Components
- [UI Overview](./codebase/07-ui/README.md)
- [Components](./codebase/07-ui/components.md) - Button, Label, Input, etc.
- [Styling](./codebase/07-ui/styling.md) - Theming, styles

#### 08. Vim - Vim Mode
- [Vim Overview](./codebase/08-vim/README.md)
- [Modes](./codebase/08-vim/modes.md) - Normal, Insert, Visual
- [Motions](./codebase/08-vim/motions.md) - Movement commands
- [Operators](./codebase/08-vim/operators.md) - d, c, y, etc.

#### 09. Collaboration
- [Collab Overview](./codebase/09-collab/README.md)
- [RPC Protocol](./codebase/09-collab/rpc-protocol.md) - Message protocol
- [Server Architecture](./codebase/09-collab/server-architecture.md) - Backend design

#### 10. Settings & Themes
- [Settings Overview](./codebase/10-settings-theme/README.md)
- [Settings System](./codebase/10-settings-theme/settings-system.md) - Configuration
- [Themes](./codebase/10-settings-theme/themes.md) - Theme system

#### 11. Features
- [Features Overview](./codebase/11-features/README.md)
- [Terminal](./codebase/11-features/terminal.md) - Terminal integration
- [Search](./codebase/11-features/search.md) - Find and replace
- [Debugging](./codebase/11-features/debugging.md) - Debugger support

### II. Contribution Helper Documentation
**Location:** `/home/user/zed/docs/contrib-helper/`

Practical guides for contributing to Zed:

#### Getting Started
- [Contribution Helper Index](./contrib-helper/README.md)
- [Getting Started Guide](./contrib-helper/getting-started.md) - Setup, build, first PR

#### Common Patterns (How-To Guides)
- [Patterns Overview](./contrib-helper/common-patterns/README.md)
- [Adding Editor Action](./contrib-helper/common-patterns/adding-editor-action.md)
- [Creating UI Component](./contrib-helper/common-patterns/creating-ui-component.md)
- [Adding Language Support](./contrib-helper/common-patterns/adding-language-support.md)
- [Implementing Vim Motion](./contrib-helper/common-patterns/implementing-vim-motion.md)
- [Adding Setting](./contrib-helper/common-patterns/adding-setting.md)
- [Integrating LSP Feature](./contrib-helper/common-patterns/integrating-lsp-feature.md)
- [Creating Panel](./contrib-helper/common-patterns/creating-panel.md)

#### Testing Guides
- [Testing Overview](./contrib-helper/testing/README.md)
- [Unit Testing](./contrib-helper/testing/unit-testing.md)
- [Integration Testing](./contrib-helper/testing/integration-testing.md)
- [UI Testing](./contrib-helper/testing/ui-testing.md)
- [Vim Testing](./contrib-helper/testing/vim-testing.md)

#### Bug Fixing Guides
- [Bug Guides Overview](./contrib-helper/bug-guides/README.md)
- [Debugging Crashes](./contrib-helper/bug-guides/debugging-crashes.md)
- [Fixing Editor Bugs](./contrib-helper/bug-guides/fixing-editor-bugs.md)
- [Fixing LSP Bugs](./contrib-helper/bug-guides/fixing-lsp-bugs.md)
- [Fixing UI Bugs](./contrib-helper/bug-guides/fixing-ui-bugs.md)
- [Performance Issues](./contrib-helper/bug-guides/performance-issues.md)

#### Roadmap & Features
- [Roadmap Overview](./contrib-helper/roadmap/README.md)
- [Contributing to Roadmap](./contrib-helper/roadmap/contributing-to-roadmap.md)
- [Example: Editor Feature](./contrib-helper/roadmap/example-new-editor-feature.md)
- [Example: Panel](./contrib-helper/roadmap/example-new-panel.md)
- [Example: Collaboration](./contrib-helper/roadmap/example-collaboration-feature.md)

#### Advanced Topics
- [Advanced Overview](./contrib-helper/advanced/README.md)
- [Architecture Decisions](./contrib-helper/advanced/architecture-decisions.md)
- [Performance Optimization](./contrib-helper/advanced/performance-optimization.md)
- [Platform Specific](./contrib-helper/advanced/platform-specific.md)
- [Collaboration Protocol](./contrib-helper/advanced/collaboration-protocol.md)

### III. Existing Zed Documentation
**Location:** `/home/user/zed/docs/src/`

Original Zed documentation:

#### Development
- [Glossary](./src/development/glossary.md) - **Essential terminology**
- [Building on macOS](./src/development/macos.md)
- [Building on Linux](./src/development/linux.md)
- [Building on Windows](./src/development/windows.md)
- [Local Collaboration](./src/development/local-collaboration.md)
- [Debuggers](./src/development/debuggers.md)
- [Debugging Crashes](./src/development/debugging-crashes.md)

---

## 🔍 Search by Topic

### Architecture & Design
- [Architecture Overview](./codebase/00-overview.md)
- [Architecture Decisions](./contrib-helper/advanced/architecture-decisions.md)
- [GPUI Core Concepts](./codebase/01-gpui/core-concepts.md)
- [Entity System](./codebase/01-gpui/entity-system.md)

### State Management
- [Entity System](./codebase/01-gpui/entity-system.md)
- [GPUI Core Concepts](./codebase/01-gpui/core-concepts.md)
- [Editor Architecture](./codebase/02-editor/architecture.md)

### UI & Rendering
- [GPUI Rendering](./codebase/01-gpui/rendering.md)
- [UI Components](./codebase/07-ui/components.md)
- [Creating UI Component](./contrib-helper/common-patterns/creating-ui-component.md)
- [Fixing UI Bugs](./contrib-helper/bug-guides/fixing-ui-bugs.md)

### Text Editing
- [Editor Architecture](./codebase/02-editor/architecture.md)
- [Selections](./codebase/02-editor/selections.md)
- [Movements](./codebase/02-editor/movements.md)
- [Display Mapping](./codebase/02-editor/display-mapping.md)

### Language Support
- [Language Overview](./codebase/05-language/README.md)
- [LSP Overview](./codebase/06-lsp/README.md)
- [Adding Language Support](./contrib-helper/common-patterns/adding-language-support.md)
- [Integrating LSP Feature](./contrib-helper/common-patterns/integrating-lsp-feature.md)

### Vim Mode
- [Vim Overview](./codebase/08-vim/README.md)
- [Vim Modes](./codebase/08-vim/modes.md)
- [Implementing Vim Motion](./contrib-helper/common-patterns/implementing-vim-motion.md)
- [Vim Testing](./contrib-helper/testing/vim-testing.md)

### Testing
- [Unit Testing](./contrib-helper/testing/unit-testing.md)
- [Integration Testing](./contrib-helper/testing/integration-testing.md)
- [UI Testing](./contrib-helper/testing/ui-testing.md)
- [Vim Testing](./contrib-helper/testing/vim-testing.md)

### Performance
- [Performance Issues](./contrib-helper/bug-guides/performance-issues.md)
- [Performance Optimization](./contrib-helper/advanced/performance-optimization.md)

### Platform-Specific
- [Platform Specific](./contrib-helper/advanced/platform-specific.md)
- [Building on macOS](./src/development/macos.md)
- [Building on Linux](./src/development/linux.md)
- [Building on Windows](./src/development/windows.md)

---

## 📊 Documentation Statistics

### Codebase Documentation
- **Total Files:** 48 files
- **Estimated Lines:** ~12,000+ lines
- **Coverage:** All major subsystems

### Contribution Helper
- **Total Files:** 30 files
- **Estimated Lines:** ~8,000+ lines
- **Guides:** 30+ practical tutorials

### Total Documentation
- **78+ comprehensive documentation files**
- **20,000+ lines of detailed content**
- **Complete coverage of Zed architecture**

---

## 🎓 Recommended Learning Paths

### Path 1: Complete Beginner (4 weeks)
**Week 1: Foundations**
- [ ] Read [Architecture Overview](./codebase/00-overview.md)
- [ ] Study [GPUI Core Concepts](./codebase/01-gpui/core-concepts.md)
- [ ] Understand [Entity System](./codebase/01-gpui/entity-system.md)
- [ ] Follow [Getting Started](./contrib-helper/getting-started.md)

**Week 2: Core Systems**
- [ ] Read [Editor Architecture](./codebase/02-editor/architecture.md)
- [ ] Study [Project Overview](./codebase/03-project/README.md)
- [ ] Explore [Workspace Layout](./codebase/04-workspace/README.md)
- [ ] Review [UI Components](./codebase/07-ui/README.md)

**Week 3: Specialized Topics**
- [ ] Choose area of interest (Vim, LSP, etc.)
- [ ] Deep dive into that subsystem
- [ ] Read related contribution patterns
- [ ] Study testing guides

**Week 4: First Contribution**
- [ ] Find a good first issue
- [ ] Follow relevant pattern guide
- [ ] Write tests
- [ ] Submit PR!

### Path 2: Specific Feature (1-2 weeks)
**Choose your area:**
- Editor features → [Editor docs](./codebase/02-editor/) + [Adding Action](./contrib-helper/common-patterns/adding-editor-action.md)
- UI components → [UI docs](./codebase/07-ui/) + [Creating Component](./contrib-helper/common-patterns/creating-ui-component.md)
- Language support → [Language docs](./codebase/05-language/) + [Adding Language](./contrib-helper/common-patterns/adding-language-support.md)
- Vim mode → [Vim docs](./codebase/08-vim/) + [Vim Motion](./contrib-helper/common-patterns/implementing-vim-motion.md)

### Path 3: Bug Fix (2-3 days)
**Identify bug type:**
- Crash → [Debugging Crashes](./contrib-helper/bug-guides/debugging-crashes.md)
- Editor → [Fixing Editor Bugs](./contrib-helper/bug-guides/fixing-editor-bugs.md)
- LSP → [Fixing LSP Bugs](./contrib-helper/bug-guides/fixing-lsp-bugs.md)
- UI → [Fixing UI Bugs](./contrib-helper/bug-guides/fixing-ui-bugs.md)
- Performance → [Performance Issues](./contrib-helper/bug-guides/performance-issues.md)

---

## 🔗 External Resources

### Official Zed Resources
- [Zed Website](https://zed.dev) - Official website
- [Zed User Docs](https://zed.dev/docs) - User documentation
- [Zed Blog](https://zed.dev/blog) - Technical blog posts
- [Zed Discord](https://discord.gg/zed) - Community chat
- [Zed GitHub](https://github.com/zed-industries/zed) - Source code

### Rust Learning
- [The Rust Book](https://doc.rust-lang.org/book/) - Learn Rust
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/) - Hands-on learning
- [Rustlings](https://github.com/rust-lang/rustlings) - Interactive exercises
- [Async Book](https://rust-lang.github.io/async-book/) - Async programming

### GPUI
- [GPUI README](../../crates/gpui/README.md) - Framework overview
- [GPUI Examples](../../crates/gpui/examples/) - Sample applications

### Project Documentation
- [CONTRIBUTING.md](../../CONTRIBUTING.md) - Contribution guidelines
- [CLAUDE.md](../../CLAUDE.md) - Coding standards
- [Glossary](./src/development/glossary.md) - Terminology

---

## 🤝 Contributing to Documentation

Found an issue? Want to improve the docs?

1. **For small fixes:** Submit a PR directly
2. **For larger changes:** Open an issue first
3. **For questions:** Ask in Discord #documentation

**Documentation guidelines:**
- Keep it practical and actionable
- Include code examples
- Cross-reference related docs
- Update both technical docs and guides
- Follow markdown best practices

---

## 💡 Tips for Using This Documentation

### For Reading
- Start with overview, then dive deep
- Follow cross-references
- Try code examples locally
- Use search (Ctrl+F) liberally

### For Contributing
- Check pattern guides first
- Read testing guides before writing tests
- Reference bug guides when stuck
- Consult advanced topics for optimization

### For Maintaining
- Keep examples up to date
- Add new patterns as they emerge
- Link to real PRs as examples
- Update when APIs change

---

## 📞 Getting Help

**Stuck?** Here's where to get help:

1. **Search this documentation** - Use the index above
2. **Check the Glossary** - [Development Glossary](./src/development/glossary.md)
3. **Read CONTRIBUTING.md** - [Contributing Guide](../../CONTRIBUTING.md)
4. **Ask in Discord** - [Zed Discord](https://discord.gg/zed)
5. **Open an issue** - [GitHub Issues](https://github.com/zed-industries/zed/issues)

---

**Last Updated:** 2025-11-16

**Documentation Version:** 1.0

**Covers Zed:** Latest development version
