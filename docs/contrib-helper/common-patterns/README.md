# Common Patterns in Zed Development

This directory contains step-by-step guides for implementing common features in Zed. Each guide includes complete examples, common pitfalls, and best practices.

## Available Guides

### Core Features

1. **[Adding an Editor Action](adding-editor-action.md)**
   - Define new editor commands
   - Implement action handlers
   - Register keybindings
   - Add tests
   - **Start here** for most editor functionality

2. **[Creating UI Components](creating-ui-component.md)**
   - Build reusable GPUI components
   - Implement the `Render` trait
   - Handle user interactions
   - Style with theme support
   - Test UI components

3. **[Adding a Setting](adding-setting.md)**
   - Define setting types
   - Register settings with the system
   - Use settings in code
   - Generate JSON schema
   - Handle migrations

### Language Support

4. **[Adding Language Support](adding-language-support.md)**
   - Create language extensions
   - Integrate tree-sitter grammars
   - Implement LSP adapters
   - Test language features
   - Package for distribution

5. **[Integrating LSP Features](integrating-lsp-feature.md)**
   - Understand the LSP protocol
   - Implement client-side features
   - Handle server capabilities
   - Integrate with UI
   - Test LSP functionality

### Editor Extensions

6. **[Implementing Vim Motions](implementing-vim-motion.md)**
   - Understand Vim mode architecture
   - Implement new motions
   - Add operators
   - Test against Neovim
   - Common Vim patterns

### UI Structure

7. **[Creating a Panel](creating-panel.md)**
   - Implement the `Panel` trait
   - Handle panel state
   - Integrate with workspace
   - Add toolbar actions
   - Persist panel state

## How to Use These Guides

Each guide follows a similar structure:

1. **Overview** - What you'll learn and when to use this pattern
2. **Prerequisites** - What you need to know first
3. **Step-by-Step Instructions** - Detailed implementation steps
4. **Complete Example** - Full working code
5. **Common Pitfalls** - Mistakes to avoid
6. **Testing** - How to test your implementation
7. **PR Checklist** - What to verify before submitting

## Quick Reference

### I want to add...

- **A keyboard shortcut** → [Adding an Editor Action](adding-editor-action.md)
- **A new command** → [Adding an Editor Action](adding-editor-action.md)
- **A UI widget** → [Creating UI Components](creating-ui-component.md)
- **User preference** → [Adding a Setting](adding-setting.md)
- **Language syntax** → [Adding Language Support](adding-language-support.md)
- **LSP feature** → [Integrating LSP Features](integrating-lsp-feature.md)
- **Vim command** → [Implementing Vim Motions](implementing-vim-motion.md)
- **Sidebar panel** → [Creating a Panel](creating-panel.md)

### I'm working on...

- **Editor features** → Actions, Settings
- **Language features** → Language Support, LSP Integration
- **UI features** → UI Components, Panels
- **Vim mode** → Vim Motions

## General Principles

All patterns in Zed follow these principles:

### 1. Entity-Based State Management

State is managed through `Entity<T>`:

```rust
// Create an entity
let entity: Entity<MyState> = cx.build_entity(|cx| MyState::new(cx));

// Read state
let value = entity.read(cx).some_value;

// Update state
entity.update(cx, |state, cx| {
    state.some_value = new_value;
    cx.notify(); // Trigger re-render
});
```

### 2. Async-First Architecture

Use async for I/O and long-running operations:

```rust
// Spawn on foreground thread
cx.spawn(async move |cx| {
    let result = async_operation().await;
    // Update UI with result
})

// Spawn on background thread
cx.background_spawn(async move {
    expensive_computation()
})
```

### 3. Proper Error Handling

Never silently discard errors:

```rust
// ✅ Propagate errors
let result = operation()?;

// ✅ Log errors
operation().log_err();

// ❌ Never do this
let _ = operation();
```

### 4. Reactive Updates

Always call `cx.notify()` when state changes:

```rust
pub fn update_value(&mut self, value: String, cx: &mut Context<Self>) {
    self.value = value;
    cx.notify(); // Important! Triggers re-render
}
```

### 5. Testing

Always add tests for new functionality:

```rust
#[gpui::test]
async fn test_my_feature(cx: &mut TestApp) {
    // Test implementation
}
```

## Code Style

Follow the guidelines in [CLAUDE.md](../../../CLAUDE.md):

- **No `unwrap()`** - Use `?` or `.log_err()`
- **No mod.rs** - Use `module_name.rs` instead
- **Full variable names** - No abbreviations
- **Comments for "why"** - Not "what"
- **Proper error handling** - Always propagate or log

## Example: Quick Pattern Overview

Here's a minimal example of each pattern:

### Editor Action
```rust
// 1. Define action
actions!(editor, [DoSomething]);

// 2. Implement handler
pub fn do_something(&mut self, _: &DoSomething, cx: &mut Context<Self>) {
    // Implementation
    cx.notify();
}

// 3. Register
editor.on_action(cx.listener(Self::do_something));
```

### UI Component
```rust
struct MyWidget {
    label: SharedString,
}

impl Render for MyWidget {
    fn render(&mut self, _: &mut Window, _: &mut Context<Self>) -> impl IntoElement {
        div().child(self.label.clone())
    }
}
```

### Setting
```rust
#[derive(Clone, Deserialize, JsonSchema)]
pub struct MySettings {
    pub enabled: bool,
}

settings::register_setting::<MySettings>(cx);
```

### Panel
```rust
impl Panel for MyPanel {
    fn panel_name(&self) -> &'static str {
        "MyPanel"
    }

    fn position(&self, _: &Window, cx: &App) -> DockPosition {
        DockPosition::Right
    }
}
```

## Next Steps

1. **Choose a guide** based on what you want to implement
2. **Read the prerequisites** to ensure you have the background knowledge
3. **Follow the step-by-step instructions** to implement your feature
4. **Review the complete example** to see it all together
5. **Check common pitfalls** to avoid mistakes
6. **Add tests** following the testing section
7. **Use the PR checklist** before submitting

## Additional Resources

- [Getting Started Guide](../getting-started.md) - Set up your environment
- [Testing Guides](../testing/) - Learn how to test
- [CLAUDE.md](../../../CLAUDE.md) - Coding guidelines
- [GPUI README](../../../crates/gpui/README.md) - UI framework docs

Happy coding!
