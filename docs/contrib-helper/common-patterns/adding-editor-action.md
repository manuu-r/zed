# Adding an Editor Action

This guide walks you through adding a new action to Zed's editor. Actions are user-triggered commands that can be invoked via keyboard shortcuts, the command palette, or programmatically.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Types of Actions](#types-of-actions)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Complete Example](#complete-example)
6. [Adding Keybindings](#adding-keybindings)
7. [Testing](#testing)
8. [Common Pitfalls](#common-pitfalls)
9. [Advanced Patterns](#advanced-patterns)
10. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How to define actions (with and without data)
- How to implement action handlers
- How to register actions with the editor
- How to add default keybindings
- How to test actions

**When to use this:**
- Adding new editor commands
- Implementing keyboard shortcuts
- Creating menu items
- Building command palette entries

**Time to complete:** 30-60 minutes

## Prerequisites

Before starting, you should understand:
- Basic Rust syntax
- GPUI's `Context<T>` and `Entity<T>` types
- How the editor manages selections and text

**Recommended reading:**
- [Getting Started Guide](../getting-started.md)
- [CLAUDE.md](../../../CLAUDE.md) - Especially the GPUI section

## Types of Actions

### 1. Simple Actions (No Parameters)

Used for actions that don't need additional data:

```rust
actions!(
    editor,
    [
        /// Saves the current file
        Save,
        /// Reverts all changes
        RevertFile,
        /// Toggles line numbers
        ToggleLineNumbers
    ]
);
```

### 2. Actions with Parameters

Used when you need to pass data to the action:

```rust
#[derive(Clone, PartialEq, Debug, Deserialize, JsonSchema, Action)]
#[action(namespace = editor)]
pub struct MoveToLine {
    pub line: u32,
}

#[derive(Clone, PartialEq, Debug, Deserialize, JsonSchema, Action)]
#[action(namespace = editor)]
pub struct SelectNext {
    #[serde(default = "default_true")]
    pub replace_newest: bool,
}
```

### 3. Actions in Different Namespaces

Actions are organized by namespace:

```rust
// Editor namespace
actions!(editor, [Cut, Copy, Paste]);

// Vim namespace
actions!(vim, [NormalMode, InsertMode]);

// Debugger namespace
actions!(debugger, [StepOver, StepInto]);
```

## Step-by-Step Implementation

Let's implement a new action: `DuplicateLine` that duplicates the current line(s).

### Step 1: Define the Action

Actions are typically defined in `crates/editor/src/actions.rs`:

```rust
// File: crates/editor/src/actions.rs

actions!(
    editor,
    [
        // ... existing actions ...

        /// Duplicates the current line or selection.
        DuplicateLine,
    ]
);
```

**Important notes:**
- The doc comment becomes the action's description in the command palette
- Use clear, concise descriptions
- Actions are automatically exported from this module

### Step 2: Implement the Action Handler

Add the handler method to the `Editor` struct in `crates/editor/src/editor.rs`:

```rust
// File: crates/editor/src/editor.rs

impl Editor {
    // ... existing methods ...

    pub fn duplicate_line(&mut self, _: &DuplicateLine, cx: &mut Context<Self>) {
        // Get the buffer
        let mut edits = Vec::new();
        let buffer = self.buffer.read(cx);
        let snapshot = buffer.snapshot(cx);

        // For each selection, duplicate its content
        for selection in self.selections.all::<Point>(cx) {
            let start = selection.start;
            let end = selection.end;

            // Get the selected text
            let text = snapshot.text_for_range(start..end).collect::<String>();

            // Insert the duplicated text after the selection
            edits.push((end..end, text));
        }

        // Apply all edits as a single transaction
        self.transact(cx, |editor, cx| {
            editor.buffer.update(cx, |buffer, cx| {
                buffer.edit(edits, None, cx);
            });
        });
    }
}
```

**Key concepts:**
- `&mut self` - Mutable access to editor state
- `_: &DuplicateLine` - The action (unused here, so we use `_`)
- `cx: &mut Context<Self>` - Context for updating the editor
- `self.transact()` - Groups edits into a single undo/redo operation
- `cx.notify()` is called automatically by `transact`

### Step 3: Register the Action

Find the place where editor actions are registered (usually in `Editor::new` or a similar initialization function):

```rust
// File: crates/editor/src/editor.rs

impl Editor {
    pub fn new(/* ... */) -> Self {
        // ... initialization code ...

        // Register action handlers
        editor.on_action(cx.listener(Self::duplicate_line));

        // ... more registrations ...

        editor
    }
}
```

**Alternative registration patterns:**

If actions are registered elsewhere (like in `workspace` or `zed`):

```rust
// In crates/zed/src/zed.rs or similar
editor_actions::init(app);

// In crates/editor/src/actions.rs or editor.rs
pub fn init(app: &mut App) {
    // Register handlers
    app.on_action(Editor::duplicate_line);
}
```

### Step 4: Make It Public (if needed)

If the action needs to be accessible from other crates:

```rust
// File: crates/editor/src/editor.rs or actions.rs

pub use crate::actions::DuplicateLine;
```

### Step 5: Build and Test Manually

```bash
# Build the editor crate
cargo build -p editor

# Run Zed to test manually
cargo run
```

In Zed:
1. Open the command palette (Cmd/Ctrl+Shift+P)
2. Search for "Duplicate Line"
3. Execute the action

## Complete Example

Here's a complete, more sophisticated example that handles line duplication properly:

```rust
// ============================================================================
// File: crates/editor/src/actions.rs
// ============================================================================

actions!(
    editor,
    [
        // ... existing actions ...

        /// Duplicates the current line or selection below.
        DuplicateLine,

        /// Duplicates the current line or selection above.
        DuplicateLineAbove,
    ]
);

// ============================================================================
// File: crates/editor/src/editor.rs
// ============================================================================

impl Editor {
    pub fn duplicate_line(&mut self, _: &DuplicateLine, cx: &mut Context<Self>) {
        self.duplicate_line_impl(false, cx);
    }

    pub fn duplicate_line_above(&mut self, _: &DuplicateLineAbove, cx: &mut Context<Self>) {
        self.duplicate_line_impl(true, cx);
    }

    fn duplicate_line_impl(&mut self, above: bool, cx: &mut Context<Self>) {
        let display_map = self.display_map.update(cx, |map, cx| map.snapshot(cx));
        let selections = self.selections.all::<Point>(cx);

        self.transact(cx, |editor, cx| {
            editor.buffer.update(cx, |buffer, cx| {
                let snapshot = buffer.snapshot(cx);
                let mut edits = Vec::new();

                for selection in selections.iter() {
                    let mut start = selection.start;
                    let mut end = selection.end;

                    // If nothing is selected, expand to full line
                    if start == end {
                        start = Point::new(start.row, 0);
                        end = if start.row < snapshot.max_buffer_row() {
                            Point::new(start.row + 1, 0)
                        } else {
                            snapshot.max_point()
                        };
                    }

                    // Get the text to duplicate
                    let text = snapshot.text_for_range(start..end).collect::<String>();

                    // Determine insert position
                    let insert_position = if above { start } else { end };

                    // If we're duplicating above and not at start of line, add newline
                    let text_to_insert = if above && start.column > 0 {
                        format!("{}\n", text)
                    } else if !above && end.column > 0 && !text.ends_with('\n') {
                        format!("\n{}", text)
                    } else {
                        text
                    };

                    edits.push((insert_position..insert_position, text_to_insert));
                }

                // Sort edits by position (reverse for correct insertion)
                edits.sort_by(|a, b| b.0.start.cmp(&a.0.start));

                buffer.edit(edits, None, cx);
            });
        });
    }
}

// ============================================================================
// Registration
// ============================================================================

// In the init or setup function:
pub fn register_editor_actions(editor: &Entity<Editor>, cx: &mut Context<Editor>) {
    editor.on_action(cx.listener(Self::duplicate_line));
    editor.on_action(cx.listener(Self::duplicate_line_above));
}
```

## Adding Keybindings

Default keybindings are defined in `assets/keymaps/default-*.json`:

### macOS Keybindings

```json
// File: assets/keymaps/default-macos.json
{
  "context": "Editor",
  "bindings": {
    "cmd-d": "editor::DuplicateLine",
    "cmd-shift-d": "editor::DuplicateLineAbove"
  }
}
```

### Linux/Windows Keybindings

```json
// File: assets/keymaps/default-linux.json
{
  "context": "Editor",
  "bindings": {
    "ctrl-d": "editor::DuplicateLine",
    "ctrl-shift-d": "editor::DuplicateLineAbove"
  }
}
```

**Key concepts:**
- `context` - Where the binding is active (e.g., "Editor", "Workspace", "Terminal")
- Action names use the format: `namespace::ActionName`
- Use platform-appropriate modifiers

### Actions with Parameters in Keybindings

```json
{
  "context": "Editor",
  "bindings": {
    "ctrl-g": [
      "editor::MoveToLine",
      {
        "line": 1
      }
    ]
  }
}
```

## Testing

### Unit Test

```rust
// File: crates/editor/src/editor_tests.rs

#[gpui::test]
async fn test_duplicate_line(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("line 1\nline 2\nline 3", cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    });

    // Test duplicating a single line
    editor.update(cx, |editor, cx| {
        // Position cursor on line 2
        editor.change_selections(None, cx, |s| {
            s.select_ranges([Point::new(1, 0)..Point::new(1, 0)])
        });

        // Duplicate the line
        editor.duplicate_line(&DuplicateLine, cx);

        // Verify result
        assert_eq!(
            editor.text(cx),
            "line 1\nline 2\nline 2\nline 3"
        );
    });
}

#[gpui::test]
async fn test_duplicate_selection(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("hello world", cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    });

    editor.update(cx, |editor, cx| {
        // Select "hello"
        editor.change_selections(None, cx, |s| {
            s.select_ranges([Point::new(0, 0)..Point::new(0, 5)])
        });

        // Duplicate selection
        editor.duplicate_line(&DuplicateLine, cx);

        // Should have "hello" duplicated
        assert_eq!(editor.text(cx), "hellohello world");
    });
}

#[gpui::test]
async fn test_duplicate_multiple_selections(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| {
        let buffer = MultiBuffer::build_simple("a\nb\nc", cx);
        Editor::new(EditorMode::Full, buffer, None, false, cx)
    });

    editor.update(cx, |editor, cx| {
        // Select multiple lines
        editor.change_selections(None, cx, |s| {
            s.select_ranges([
                Point::new(0, 0)..Point::new(0, 0),
                Point::new(2, 0)..Point::new(2, 0),
            ])
        });

        editor.duplicate_line(&DuplicateLine, cx);

        assert_eq!(editor.text(cx), "a\na\nb\nc\nc");
    });
}
```

### Running Tests

```bash
# Run the specific test
cargo test -p editor test_duplicate_line

# Run all editor tests
cargo test -p editor

# Run with output
cargo test -p editor test_duplicate_line -- --nocapture
```

### Manual Testing Checklist

Test these scenarios manually:

- [ ] Action appears in command palette
- [ ] Keybinding works
- [ ] Works with empty selection (duplicates line)
- [ ] Works with selected text
- [ ] Works with multiple cursors
- [ ] Works at start of file
- [ ] Works at end of file
- [ ] Can be undone with Cmd/Ctrl+Z
- [ ] Can be redone with Cmd/Ctrl+Shift+Z

## Common Pitfalls

### 1. Forgetting to Call `cx.notify()`

❌ **Wrong:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    self.some_value = new_value;
    // Missing cx.notify() - UI won't update!
}
```

✅ **Correct:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    self.some_value = new_value;
    cx.notify();
}

// Or use transact which calls notify automatically
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    self.transact(cx, |editor, cx| {
        // Changes here
    }); // notify called automatically
}
```

### 2. Not Using `transact` for Editor Changes

❌ **Wrong:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    self.buffer.update(cx, |buffer, cx| {
        buffer.edit(edits, None, cx);
    });
    // This won't be undoable as a single operation
}
```

✅ **Correct:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    self.transact(cx, |editor, cx| {
        editor.buffer.update(cx, |buffer, cx| {
            buffer.edit(edits, None, cx);
        });
    });
    // Now it's a single undo/redo operation
}
```

### 3. Incorrect Action Naming

❌ **Wrong:**
```rust
// Action name doesn't match Rust naming conventions
actions!(editor, [duplicate_line]);  // Snake case
```

✅ **Correct:**
```rust
// Use PascalCase for action names
actions!(editor, [DuplicateLine]);  // Pascal case
```

### 4. Missing Namespace in Keybindings

❌ **Wrong:**
```json
{
  "bindings": {
    "cmd-d": "DuplicateLine"  // Missing namespace
  }
}
```

✅ **Correct:**
```json
{
  "bindings": {
    "cmd-d": "editor::DuplicateLine"  // Include namespace
  }
}
```

### 5. Not Handling Multiple Selections

❌ **Wrong:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    let selection = self.selections.newest::<Point>(cx);
    // Only handles one selection
}
```

✅ **Correct:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    let selections = self.selections.all::<Point>(cx);
    // Handle all selections
    for selection in selections {
        // Process each
    }
}
```

### 6. Using `unwrap()` Instead of `?`

❌ **Wrong:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    let result = something().unwrap();  // Will panic!
}
```

✅ **Correct:**
```rust
pub fn my_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    if let Some(result) = something() {
        // Handle result
    }
    // Or if method can return Result:
    let result = something()?;
}
```

## Advanced Patterns

### 1. Actions That Modify State and Return Values

```rust
pub fn my_action(&mut self, action: &MyAction, cx: &mut Context<Self>) -> Option<String> {
    if !self.can_perform_action() {
        return None;
    }

    let result = self.transact(cx, |editor, cx| {
        // Modifications
        Some("result".to_string())
    });

    result
}
```

### 2. Actions with Callbacks

```rust
pub fn my_action_with_callback<F>(&mut self, _: &MyAction, callback: F, cx: &mut Context<Self>)
where
    F: FnOnce(&mut Self, &mut Context<Self>),
{
    self.transact(cx, |editor, cx| {
        callback(editor, cx);
    });
}
```

### 3. Async Actions

```rust
pub fn my_async_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    let task = cx.spawn(async move |editor, cx| {
        let result = fetch_data().await?;

        editor.update(&cx, |editor, cx| {
            editor.apply_result(result, cx);
        })
    });

    task.detach_and_log_err(cx);
}
```

### 4. Conditional Actions (Context-Sensitive)

```rust
pub fn my_conditional_action(&mut self, _: &MyAction, cx: &mut Context<Self>) {
    // Only works in certain modes
    if !self.is_focused(cx) {
        return;
    }

    if self.read_only {
        cx.emit(EditorEvent::ShowError("Cannot edit read-only file".into()));
        return;
    }

    // Perform action
    self.transact(cx, |editor, cx| {
        // ...
    });
}
```

### 5. Actions That Dispatch Other Actions

```rust
pub fn macro_action(&mut self, _: &MacroAction, cx: &mut Context<Self>) {
    // Dispatch multiple actions
    self.save(&Save, cx);
    self.format(&Format, cx);
    self.save(&Save, cx);
}
```

## PR Checklist

Before submitting your pull request:

### Code Quality
- [ ] Action defined in `actions.rs` with doc comment
- [ ] Handler implemented following CLAUDE.md guidelines
- [ ] No use of `.unwrap()` - errors properly handled
- [ ] Uses `transact` for editor modifications
- [ ] Calls `cx.notify()` when needed (or via `transact`)
- [ ] Handles multiple selections correctly
- [ ] Action registered properly

### Testing
- [ ] Unit tests added and passing
- [ ] Tests cover edge cases (empty selection, multiple cursors, etc.)
- [ ] Manually tested in running Zed
- [ ] Works with undo/redo
- [ ] `./script/clippy` passes

### Documentation
- [ ] Doc comment explains what action does
- [ ] Complex logic has explanatory comments (why, not what)
- [ ] Public APIs documented

### Keybindings (if applicable)
- [ ] Default keybindings added for macOS
- [ ] Default keybindings added for Linux/Windows
- [ ] Keybindings don't conflict with existing shortcuts
- [ ] Context is correct

### User Experience
- [ ] Action name is clear and discoverable
- [ ] Action appears in command palette
- [ ] Behavior is intuitive
- [ ] Works consistently across platforms

## Next Steps

After implementing an editor action, you might want to:

- [Add Settings](adding-setting.md) to make the action configurable
- [Create UI Components](creating-ui-component.md) to visualize action results
- [Add Tests](../testing/unit-testing.md) for comprehensive coverage
- [Integrate LSP Features](integrating-lsp-feature.md) for language-aware actions

## Resources

- [Editor crate source](../../../crates/editor/src/)
- [Example actions](../../../crates/editor/src/actions.rs)
- [Editor tests](../../../crates/editor/src/editor_tests.rs)
- [GPUI documentation](../../../crates/gpui/README.md)
- [CLAUDE.md](../../../CLAUDE.md)
