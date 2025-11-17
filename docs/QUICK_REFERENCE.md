# Zed Quick Reference Guide

Quick lookup for common tasks, patterns, and APIs in Zed development.

---

## 🚀 Common Tasks

### Build & Run
```bash
# Build debug
cargo build

# Build release
cargo build --release

# Run Zed
cargo run

# Run specific crate tests
cargo test -p editor

# Run all tests
cargo test

# Run linter (IMPORTANT: use this, not cargo clippy)
./script/clippy

# Format code
cargo fmt
```

### Git Workflow
```bash
# Create feature branch
git checkout -b claude/your-feature-name

# Stage changes
git add .

# Commit
git commit -m "description"

# Push
git push -u origin claude/your-feature-name
```

---

## 📦 Key Types Quick Reference

### GPUI Core Types
```rust
// State management
Entity<T>              // Strong handle to state
WeakEntity<T>          // Weak handle (doesn't prevent dropping)
App                    // Root context (UI thread only!)
Context<T>             // Specialized context for Entity<T>
Window                 // Window handle
AsyncApp               // Async context
Task<R>                // Async task

// UI Building
impl Render for MyView {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div().child("Hello")
    }
}

// Common Elements
div()                  // Container element
label("text")          // Text label
button("text")         // Button
Icon::new(IconName::Check) // Icon
```

### Editor Types
```rust
Editor                 // Main editor entity
MultiBuffer            // Multiple buffers in one editor
Selection              // Text selection
DisplayPoint           // Point in display space (with wrapping)
Anchor                 // Stable position in buffer
```

### Project Types
```rust
Project                // Project entity
Worktree               // File tree
Buffer                 // Text buffer for a file
LanguageServer         // LSP server instance
```

### Workspace Types
```rust
Workspace              // Main window entity
Pane                   // Tab container
PaneGroup              // Split panes
Dock                   // Side/bottom panel area
```

---

## 🎯 Common Patterns

### Creating an Entity
```rust
let entity = cx.new(|cx| MyStruct {
    field: value,
});
```

### Updating an Entity
```rust
entity.update(cx, |entity, cx| {
    entity.do_something();
    cx.notify(); // Trigger re-render
});
```

### Reading from an Entity
```rust
let value = entity.read(cx).some_field;
```

### Spawning Async Task (Foreground)
```rust
cx.spawn(async move |this, cx| {
    let result = async_operation().await;
    this.update(&cx, |this, cx| {
        this.handle_result(result, cx);
    })?;
    Ok(())
}).detach();
```

### Spawning Async Task (Background)
```rust
cx.background_spawn(async move {
    expensive_computation()
}).detach();
```

### Defining Actions
```rust
// Simple actions (no data)
actions!(editor, [MoveUp, MoveDown, SelectAll]);

// Action with data
#[derive(Clone, PartialEq, Debug)]
struct SelectLine {
    line: u32,
}

impl_actions!(editor, [SelectLine]);
```

### Handling Actions
```rust
div()
    .on_action(cx.listener(|this: &mut MyView, action: &MyAction, window, cx| {
        this.handle_action(action, cx);
    }))
```

### Subscribing to Events
```rust
cx.subscribe(&entity, |this, entity, event, cx| {
    match event {
        Event::Changed => this.handle_change(cx),
    }
}).detach();
```

### Observing Changes
```rust
cx.observe(&entity, |this, entity, cx| {
    // Called when entity.cx.notify() is called
}).detach();
```

### Conditional Rendering
```rust
div()
    .when(condition, |div| {
        div.child("Show when true")
    })
    .when_some(option, |div, value| {
        div.child(format!("Value: {}", value))
    })
```

---

## 🎨 Styling Quick Reference

### Common Style Methods
```rust
div()
    // Size
    .w_full()              // width: 100%
    .h(px(100.))           // height: 100px
    .size_full()           // width & height: 100%

    // Flexbox
    .flex()                // display: flex
    .flex_row()            // flex-direction: row
    .flex_col()            // flex-direction: column
    .items_center()        // align-items: center
    .justify_center()      // justify-content: center
    .gap_2()               // gap: 0.5rem

    // Padding & Margin
    .p_4()                 // padding: 1rem
    .px_2()                // padding-left/right: 0.5rem
    .py_2()                // padding-top/bottom: 0.5rem
    .m_2()                 // margin: 0.5rem

    // Colors
    .bg(gpui::red())       // background
    .text_color(gpui::white()) // text color

    // Border
    .border_1()            // border: 1px
    .rounded_md()          // border-radius: medium

    // Positioning
    .relative()            // position: relative
    .absolute()            // position: absolute

    // Children
    .child("text")         // Add child
    .children(vec![...])   // Add multiple children
```

### Theme Colors
```rust
let theme = cx.theme();
let colors = &theme.colors();

colors.editor.background
colors.editor.foreground
colors.text
colors.border
colors.error
colors.warning
colors.success
```

---

## 🔧 Error Handling

### Preferred Pattern (Propagate Errors)
```rust
fn my_function() -> Result<()> {
    let value = fallible_operation()?;
    Ok(())
}
```

### Log and Continue
```rust
operation().log_err();  // Logs error, returns None
```

### Never Do This (Panics!)
```rust
❌ value.unwrap()       // DON'T
❌ value.expect("msg")  // DON'T
❌ let _ = result;      // DON'T silently ignore errors
```

---

## 🧪 Testing Patterns

### Basic Test
```rust
#[gpui::test]
async fn test_something(cx: &mut TestAppContext) {
    let entity = cx.new(|cx| MyStruct::new());

    entity.update(cx, |entity, cx| {
        entity.do_something(cx);
    });

    assert_eq!(entity.read(cx).value, expected);
}
```

### Editor Test
```rust
#[gpui::test]
async fn test_editor(cx: &mut TestAppContext) {
    init_test(cx);

    let editor = cx.new(|cx| {
        Editor::single_line(cx)
    });

    editor.update(cx, |editor, cx| {
        editor.insert("Hello", cx);
    });

    assert_eq!(editor.read(cx).text(cx), "Hello");
}
```

### Vim Test (Neovim-backed)
```rust
#[gpui::test]
async fn test_vim_motion(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("ˇHello world").await;
    cx.simulate_shared_keystrokes(["w"]).await;
    cx.assert_shared_state("Hello ˇworld").await;
}
```

---

## 📁 File Locations

### Source Code
```
crates/
├── zed/            # Main application entry point
├── gpui/           # UI framework
├── editor/         # Text editor
├── project/        # File/project management
├── workspace/      # Window management
├── language/       # Language support
├── lsp/            # LSP client
├── vim/            # Vim mode
├── ui/             # UI components
└── ...
```

### Tests
```
crates/CRATE_NAME/src/
├── lib.rs or crate_name.rs
├── module_tests.rs         # Unit tests
└── test_data/              # Test fixtures
```

### Configuration
```
assets/
├── settings/default.json   # Default settings
├── keymaps/               # Default keybindings
└── themes/                # Built-in themes
```

---

## 🔍 Finding Things

### Find Files
```bash
fd "pattern" crates/
```

### Search Code
```bash
rg "pattern" crates/
```

### Find Type Definition
```bash
rg "struct EditorName" crates/
```

### Find Trait Implementation
```bash
rg "impl Render for" crates/
```

### Find Action Definitions
```bash
rg "actions!\(|impl_actions!" crates/
```

---

## 🐛 Debugging

### Print Debugging
```rust
dbg!(value);                    // Debug print
eprintln!("Debug: {:?}", value); // Stderr print
log::info!("Info: {}", value);   // Logging
```

### Logging Levels
```rust
log::error!("Critical error");
log::warn!("Warning");
log::info!("Information");
log::debug!("Debug info");
log::trace!("Trace info");
```

### Common Issues

**"cannot borrow as mutable"**
```rust
// Problem: Multiple borrows
let value = cx.read(&entity);
cx.update(...);  // Error!

// Solution: Scope the borrow
{
    let value = cx.read(&entity);
    // Use value
} // Borrow ends
cx.update(...);  // OK!
```

**"Entity dropped"**
```rust
// Problem: WeakEntity outlived Entity
weak.update(cx, |entity, cx| { ... })?;  // May fail

// Solution: Check if still alive
if let Some(entity) = weak.upgrade() {
    entity.update(cx, |entity, cx| { ... });
}
```

---

## 📚 Key Documentation

### Essential Reads
- [Architecture Overview](./codebase/00-overview.md)
- [GPUI Core Concepts](./codebase/01-gpui/core-concepts.md)
- [Entity System](./codebase/01-gpui/entity-system.md)
- [Glossary](./src/development/glossary.md)
- [CLAUDE.md (Coding Standards)](../CLAUDE.md)

### By Task
- **Adding feature:** [Common Patterns](./contrib-helper/common-patterns/)
- **Fixing bug:** [Bug Guides](./contrib-helper/bug-guides/)
- **Writing tests:** [Testing Guides](./contrib-helper/testing/)
- **Understanding code:** [Codebase Docs](./codebase/)

---

## 🎓 Learning Resources

### Rust
- [The Rust Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [Async Book](https://rust-lang.github.io/async-book/)

### Zed
- [Getting Started](./contrib-helper/getting-started.md)
- [GPUI README](../crates/gpui/README.md)
- [Zed Discord](https://discord.gg/zed)

---

## 💬 Common Commands

### Development
```bash
# Check compilation
cargo check

# Run specific test
cargo test test_name

# Run tests for crate
cargo test -p crate_name

# Run with logs
RUST_LOG=debug cargo run

# Build documentation
cargo doc --open

# See dependencies
cargo tree -p crate_name
```

### Git
```bash
# Check status
git status

# See diff
git diff

# Stage specific file
git add path/to/file

# Commit with message
git commit -m "feat: description"

# Amend last commit
git commit --amend

# Push to remote
git push -u origin branch-name
```

---

## 🎯 Commit Message Format

```
type: brief description

Longer explanation if needed.

- Bullet points for details
- Multiple changes

Fixes #issue_number
```

**Types:**
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `refactor:` Code refactoring
- `test:` Adding tests
- `perf:` Performance improvement
- `chore:` Maintenance

---

## 🔗 Quick Links

- **[Full Documentation Index](./DOCUMENTATION_INDEX.md)**
- **[Codebase Docs](./codebase/README.md)**
- **[Contribution Guides](./contrib-helper/README.md)**
- **[Getting Started](./contrib-helper/getting-started.md)**
- **[Glossary](./src/development/glossary.md)**
- **[CONTRIBUTING.md](../CONTRIBUTING.md)**

---

**Pro Tip:** Bookmark this page for quick reference while developing!
