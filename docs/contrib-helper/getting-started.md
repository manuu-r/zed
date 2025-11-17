# Getting Started with Zed Development

This comprehensive guide will help you set up your development environment, understand the codebase structure, and make your first contribution to Zed.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Development Environment Setup](#development-environment-setup)
3. [Building Zed from Source](#building-zed-from-source)
4. [Understanding the Codebase](#understanding-the-codebase)
5. [Running Tests](#running-tests)
6. [Your First Contribution](#your-first-contribution)
7. [Finding Good First Issues](#finding-good-first-issues)
8. [Getting Help](#getting-help)
9. [Common Setup Issues](#common-setup-issues)

## Prerequisites

Before you begin, ensure you have the following installed:

### Required Tools

1. **Rust** (latest stable version)
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

2. **Git**
   ```bash
   # macOS
   brew install git

   # Ubuntu/Debian
   sudo apt install git
   ```

3. **Platform-specific dependencies**:

   **macOS:**
   ```bash
   # Install Xcode command line tools
   xcode-select --install
   ```

   **Linux (Ubuntu/Debian):**
   ```bash
   sudo apt install \
     build-essential \
     libssl-dev \
     pkg-config \
     libfontconfig1-dev \
     libfreetype6-dev \
     libxcb-composite0-dev \
     libxkbcommon-dev \
     libwayland-dev \
     libvulkan-dev
   ```

   **Arch Linux:**
   ```bash
   sudo pacman -S \
     base-devel \
     openssl \
     pkg-config \
     fontconfig \
     freetype2 \
     xcb-util \
     libxkbcommon \
     wayland \
     vulkan-icd-loader
   ```

### Recommended Tools

- **rust-analyzer** - For IDE support
- **cargo-watch** - For auto-recompiling during development
- **cargo-nextest** - Faster test runner

```bash
rustup component add rust-analyzer
cargo install cargo-watch cargo-nextest
```

## Development Environment Setup

### 1. Fork and Clone the Repository

```bash
# Fork on GitHub first, then clone your fork
git clone https://github.com/YOUR_USERNAME/zed.git
cd zed

# Add upstream remote
git remote add upstream https://github.com/zed-industries/zed.git
```

### 2. Configure Git

```bash
# Set up your name and email
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Recommended: Set up rebase as default pull strategy
git config pull.rebase true
```

### 3. Verify Rust Installation

```bash
# Check Rust version
rustc --version

# Should be 1.70+ or newer
cargo --version
```

### 4. Install Zed's Dependencies

On first clone, cargo will download all dependencies. This may take a while:

```bash
# This will download and compile dependencies
cargo check
```

## Building Zed from Source

### Quick Build

```bash
# Debug build (faster compilation, slower runtime)
cargo build

# Release build (slower compilation, faster runtime)
cargo build --release
```

### Running Zed

```bash
# Run in debug mode
cargo run

# Run in release mode
cargo run --release

# Run with specific workspace
cargo run -- path/to/workspace

# Run with logging
RUST_LOG=debug cargo run
```

### Build Times

- **First build**: 10-30 minutes (depending on your machine)
- **Incremental builds**: 30 seconds - 5 minutes
- **Clean builds**: 5-15 minutes

**Tip:** Use `cargo build --timings` to see what takes longest to compile.

### Platform-Specific Builds

**macOS:**
```bash
# Build the app bundle
./script/bundle-mac
```

**Linux:**
```bash
# Wayland (default)
cargo build --release

# X11
cargo build --release --no-default-features --features x11
```

## Understanding the Codebase

### Crate Structure

Zed is organized into multiple crates, each with a specific responsibility:

```
crates/
├── zed/              # Main application entry point
├── gpui/             # GPU-accelerated UI framework ⭐ START HERE
├── editor/           # Core editor implementation
├── workspace/        # Window and workspace management
├── project/          # File tree and project management
├── language/         # Programming language support
├── lsp/              # Language Server Protocol client
├── vim/              # Vim mode implementation
├── ui/               # Reusable UI components
├── theme/            # Theme system
├── rpc/              # Collaboration protocol
├── collab/           # Collaboration server
└── ...               # Many more specialized crates
```

### Key Concepts

#### 1. GPUI Framework

GPUI is Zed's custom UI framework. Understanding GPUI is essential for contributing to Zed.

**Core Types:**

- `App` - Global application context
- `Context<T>` - Context for updating an entity of type T
- `Window` - Window-specific context
- `Entity<T>` - Handle to state of type T
- `WeakEntity<T>` - Weak reference to an entity

**Basic Example:**
```rust
use gpui::{App, Context, Entity, Render, Window};

struct Counter {
    count: usize,
}

impl Counter {
    fn increment(&mut self, cx: &mut Context<Self>) {
        self.count += 1;
        cx.notify(); // Tell GPUI to re-render
    }
}

impl Render for Counter {
    fn render(&mut self, _window: &mut Window, _cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .child(format!("Count: {}", self.count))
            .on_click(cx.listener(|this, _event, _window, cx| {
                this.increment(cx);
            }))
    }
}
```

**📖 Read More:**
- [GPUI README](../../crates/gpui/README.md)
- [CLAUDE.md GPUI section](../../CLAUDE.md)
- [Creating UI Components](common-patterns/creating-ui-component.md)

#### 2. Entity-Based State Management

Entities are how Zed manages state. Think of them as smart pointers to heap-allocated state with lifecycle management.

```rust
// Creating an entity
let counter: Entity<Counter> = cx.build_entity(|_cx| Counter { count: 0 });

// Reading an entity
let count = counter.read(cx).count;

// Updating an entity
counter.update(cx, |counter, cx| {
    counter.count += 1;
    cx.notify();
});
```

#### 3. Async Architecture

Zed uses async Rust extensively for non-blocking operations:

```rust
// Spawn a task on the foreground thread
cx.spawn(async move |cx| {
    // Do async work
    let result = fetch_data().await;

    // Update state
    entity.update(&cx, |this, cx| {
        this.data = result;
        cx.notify();
    })
})

// Spawn work on a background thread
cx.background_spawn(async move {
    // CPU-intensive work here
    compute_something()
})
```

### Code Organization Patterns

#### Actions

Actions are user-triggered commands. They're defined in two ways:

**Simple actions (no data):**
```rust
actions!(editor, [
    /// Saves the current file
    Save,
    /// Reverts all changes
    RevertFile
]);
```

**Actions with data:**
```rust
#[derive(Clone, PartialEq, Debug, Deserialize, JsonSchema, Action)]
#[action(namespace = editor)]
struct MoveToLine {
    line: u32,
}
```

#### Settings

Settings are user-configurable options:

```rust
#[derive(Clone, Deserialize, JsonSchema)]
pub struct MyFeatureSettings {
    /// Enable this feature
    pub enabled: bool,
    /// Maximum items to show
    pub max_items: usize,
}

// Register in init function
settings::register_setting::<MyFeatureSettings>(cx);
```

### File Naming Conventions

- Use `snake_case` for file names: `my_feature.rs`
- **Never** create `mod.rs` files - use `my_module.rs` instead
- Test files: `my_feature_tests.rs` or inline `#[cfg(test)] mod tests`
- Library roots: Use `path` in `Cargo.toml` instead of `lib.rs`

## Running Tests

### Running All Tests

```bash
# Run all tests (slow!)
cargo test

# Run tests with nextest (faster)
cargo nextest run
```

### Running Specific Tests

```bash
# Run tests in a specific crate
cargo test -p editor

# Run a specific test
cargo test -p editor test_selection_movement

# Run tests matching a pattern
cargo test -p vim test_motion

# Run with output
cargo test -p editor test_name -- --nocapture
```

### Running Tests for a Specific File

```bash
# If you modified crates/editor/src/movement.rs
cargo test -p editor movement
```

### Integration Tests

```bash
# Run workspace integration tests
cargo test -p workspace --test workspace_tests
```

### Vim Mode Tests

Vim mode has special test infrastructure that compares behavior against Neovim:

```bash
# Run vim tests
cargo test -p vim

# Run specific vim test
cargo test -p vim test_insert_mode
```

### UI/Visual Tests

```bash
# Run UI tests (requires display)
cargo test -p gpui test_render
```

### Test Debugging

```bash
# Run tests with logging
RUST_LOG=debug cargo test -p editor test_name -- --nocapture

# Run tests with backtraces
RUST_BACKTRACE=1 cargo test -p editor test_name
```

## Your First Contribution

Let's walk through making a simple contribution: adding a new editor action to convert text to title case.

### Step 1: Find the Right Place

For editor actions, look in `crates/editor/src/`:

```bash
# Examine existing case conversion actions
rg "ConvertTo" crates/editor/src/actions.rs
```

### Step 2: Define the Action

Add to `crates/editor/src/actions.rs`:

```rust
actions!(
    editor,
    [
        // ... existing actions ...
        /// Converts selected text to Title Case.
        ConvertToTitleCase,
    ]
);
```

### Step 3: Implement the Handler

In `crates/editor/src/editor.rs`, find where other case conversions are handled and add:

```rust
pub fn convert_to_title_case(&mut self, _: &ConvertToTitleCase, cx: &mut Context<Self>) {
    self.manipulate_text(cx, |text| {
        // Title case logic
        text.split_whitespace()
            .map(|word| {
                let mut chars = word.chars();
                match chars.next() {
                    None => String::new(),
                    Some(first) => first.to_uppercase().collect::<String>()
                        + &chars.as_str().to_lowercase(),
                }
            })
            .collect::<Vec<_>>()
            .join(" ")
    });
}
```

### Step 4: Register the Action Handler

Find where actions are registered (usually in `init` function or similar):

```rust
editor.on_action(cx.listener(Self::convert_to_title_case));
```

### Step 5: Add a Test

Add to `crates/editor/src/editor_tests.rs`:

```rust
#[gpui::test]
async fn test_convert_to_title_case(cx: &mut gpui::TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("hello world", cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    });

    editor.update(cx, |editor, cx| {
        editor.select_all(&SelectAll, cx);
        editor.convert_to_title_case(&ConvertToTitleCase, cx);
        assert_eq!(editor.text(cx), "Hello World");
    });
}
```

### Step 6: Run Tests

```bash
cargo test -p editor test_convert_to_title_case
```

### Step 7: Run Clippy

```bash
./script/clippy
```

### Step 8: Test Manually

```bash
cargo run
```

Then test the action in Zed using the command palette (Cmd/Ctrl+Shift+P).

### Step 9: Commit and Push

```bash
git checkout -b add-title-case-conversion
git add .
git commit -m "editor: Add ConvertToTitleCase action

Adds a new action to convert selected text to Title Case,
where the first letter of each word is capitalized and
the rest are lowercase."

git push origin add-title-case-conversion
```

### Step 10: Create Pull Request

- Go to GitHub and create a PR from your branch
- Fill in the description with:
  - What you're adding
  - Why it's useful
  - How you tested it
  - Screenshots/videos if applicable

## Finding Good First Issues

### GitHub Labels

Look for these labels:
- [`good first issue`](https://github.com/zed-industries/zed/labels/good%20first%20issue) - Perfect for newcomers
- [`help wanted`](https://github.com/zed-industries/zed/labels/help%20wanted) - Community contributions welcome
- [`documentation`](https://github.com/zed-industries/zed/labels/documentation) - Doc improvements

### Top-Voted Issues

Check [top-ranking issues](https://github.com/zed-industries/zed/issues/5393) to see what the community wants most.

### Public Roadmap

Review the [public roadmap](https://zed.dev/roadmap) for larger features you can contribute to.

### Types of Contributions

**Low Complexity:**
- Fix typos in documentation
- Add doc comments to public APIs
- Fix clippy warnings
- Add simple keyboard shortcuts

**Medium Complexity:**
- Add new editor actions
- Improve existing features
- Fix reported bugs
- Add tests

**High Complexity:**
- Implement LSP features
- Add new panels
- Implement major roadmap items
- Performance optimizations

### What to Avoid

Don't start with:
- Giant refactorings
- Rewriting large modules
- Changing fundamental architecture
- Adding features that can be extensions

## Getting Help

### Community Resources

1. **GitHub Discussions** - Ask questions
   https://github.com/zed-industries/zed/discussions

2. **Discord** - Real-time chat
   Join the Zed Discord server

3. **GitHub Issues** - Report bugs or ask about specific issues

### How to Ask Good Questions

When asking for help:

1. **Show what you've tried**
   ```
   I'm trying to add a new editor action, and I've:
   - Defined the action in actions.rs
   - Added a handler in editor.rs
   - But I'm not sure how to register it

   Here's my code: [code snippet]
   ```

2. **Provide context**
   - What are you trying to accomplish?
   - What have you already read/tried?
   - What's the error message?

3. **Include relevant code**
   - Share error messages
   - Include relevant code snippets
   - Link to your branch if possible

### Reading Existing Code

When stuck, look at similar existing features:

```bash
# Find similar implementations
rg "similar_function_name" crates/

# See how actions are registered
rg "on_action" crates/editor/src/editor.rs

# Find test examples
rg "test_" crates/editor/src/editor_tests.rs
```

## Common Setup Issues

### Issue: Compilation Errors

**Problem:** `error: could not compile ...`

**Solutions:**
```bash
# Update Rust
rustup update

# Clean and rebuild
cargo clean
cargo build

# Check for platform-specific deps
# See platform requirements above
```

### Issue: Tests Fail

**Problem:** Tests that pass in CI fail locally

**Solutions:**
```bash
# Make sure you're on latest
git pull upstream main

# Update dependencies
cargo update

# Run tests with backtrace
RUST_BACKTRACE=1 cargo test
```

### Issue: Slow Compilation

**Problem:** Builds take forever

**Solutions:**

1. **Use a faster linker:**

   Add to `~/.cargo/config.toml`:
   ```toml
   [target.x86_64-unknown-linux-gnu]
   linker = "clang"
   rustflags = ["-C", "link-arg=-fuse-ld=lld"]

   [target.x86_64-apple-darwin]
   rustflags = ["-C", "link-arg=-fuse-ld=/usr/local/opt/llvm/bin/ld64.lld"]
   ```

2. **Increase parallelism:**

   Add to `~/.cargo/config.toml`:
   ```toml
   [build]
   jobs = 8  # Adjust based on your CPU cores
   ```

3. **Use sccache:**
   ```bash
   cargo install sccache
   export RUSTC_WRAPPER=sccache
   ```

### Issue: Can't Run Zed

**Problem:** `cargo run` fails with display/graphics errors

**Linux Solutions:**
```bash
# Try Wayland
cargo run

# Or force X11
cargo run --no-default-features --features x11

# Check graphics drivers
vulkaninfo
```

**macOS Solutions:**
```bash
# Make sure Xcode tools are installed
xcode-select --install
```

### Issue: Git Hooks Failing

**Problem:** Pre-commit hooks fail

**Solution:**
```bash
# Run clippy manually
./script/clippy

# Fix issues
cargo clippy --fix

# Or skip hooks temporarily (not recommended)
git commit --no-verify
```

## Development Workflow Tips

### Fast Iteration Loop

```bash
# Terminal 1: Auto-rebuild on changes
cargo watch -x 'build -p editor'

# Terminal 2: Run tests on changes
cargo watch -x 'test -p editor'

# Terminal 3: Run Zed
cargo run
```

### Editor Setup

Use Zed itself for development!

1. Open the zed repository in Zed
2. Language server will provide:
   - Code completion
   - Go to definition
   - Inline diagnostics
3. Use Cmd/Ctrl+Shift+P for command palette
4. Use Cmd/Ctrl+P for fuzzy file search

### Debugging

**Print Debugging:**
```rust
eprintln!("Debug: value = {:?}", value);
log::debug!("Something happened: {:?}", data);
```

**Logging:**
```bash
# Run with logging
RUST_LOG=debug cargo run

# Filter to specific crate
RUST_LOG=editor=debug cargo run

# Multiple modules
RUST_LOG=editor=debug,vim=trace cargo run
```

**Using lldb/gdb:**
```bash
# Build with debug info
cargo build

# Run in debugger
rust-lldb target/debug/zed
# or
rust-gdb target/debug/zed
```

## Next Steps

Now that you're set up:

1. ✅ **Verify your setup** - Build and run Zed successfully
2. 📖 **Read GPUI docs** - [crates/gpui/README.md](../../crates/gpui/README.md)
3. 📚 **Review [CLAUDE.md](../../CLAUDE.md)** - Understand coding guidelines
4. 🔍 **Find an issue** - Pick something that interests you
5. 📖 **Read relevant guides** - Check [common patterns](common-patterns/)
6. 💻 **Start coding** - Make your first contribution!

## Checklist for Your First PR

Before submitting:

- [ ] Code builds without warnings: `cargo build`
- [ ] Tests pass: `cargo test -p <crate>`
- [ ] Clippy passes: `./script/clippy`
- [ ] Code follows [CLAUDE.md](../../CLAUDE.md) guidelines
- [ ] Tests added for new functionality
- [ ] Tested manually in running Zed
- [ ] Commit message is descriptive
- [ ] Ready for review!

Welcome to Zed development! 🎉
