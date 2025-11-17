# Editor Actions

**Last Updated:** 2025-11-16

---

## Overview

Editor actions are user commands triggered by keyboard shortcuts, menu items, or programmatically.

**File:** `/home/user/zed/crates/editor/src/actions.rs`

## Defining Actions

```rust
// Simple actions (no parameters)
actions!(
    editor,
    [
        MoveUp,
        MoveDown,
        MoveLeft,
        MoveRight,
        Newline,
        Backspace,
        Delete,
        Tab,
        // ... many more
    ]
);

// Actions with parameters
#[derive(Clone, PartialEq, Deserialize)]
pub struct MoveTo {
    pub line: u32,
    pub column: u32,
}

impl_actions!(editor, [MoveTo]);
```

## Common Editor Actions

### Movement Actions

```rust
MoveUp
MoveDown
MoveLeft
MoveRight
MoveToBeginningOfLine
MoveToEndOfLine
MoveToBeginning
MoveToEnd
MovePageUp
MovePageDown
```

### Selection Actions

```rust
SelectUp
SelectDown
SelectLeft
SelectRight
SelectAll
SelectLine
SelectWord
ExpandSelection
ShrinkSelection
AddSelectionAbove
AddSelectionBelow
```

### Editing Actions

```rust
Newline
Backspace
Delete
DeleteLine
DuplicateLine
Indent
Outdent
ToggleComments
JoinLines
```

### Clipboard Actions

```rust
Copy
Cut
Paste
Undo
Redo
```

### Search Actions

```rust
FindNext
FindPrevious
SelectNext
SelectPrevious
```

### Code Actions

```rust
Format
GoToDefinition
GoToTypeDefinition
GoToImplementation
FindAllReferences
Rename
ShowCompletions
ShowCodeActions
ToggleCodeActions
```

## Registering Action Handlers

```rust
impl Editor {
    pub fn register_actions(cx: &mut Context<Self>) {
        // Simple action
        cx.on_action(|editor: &mut Editor, _: &MoveUp, window, cx| {
            editor.move_up(window, cx);
        });

        // Action with parameters
        cx.on_action(|editor: &mut Editor, action: &MoveTo, window, cx| {
            editor.move_to(action.line, action.column, window, cx);
        });

        // Action requiring async work
        cx.on_action(|editor: &mut Editor, _: &Format, window, cx| {
            editor.format(window, cx);
        });
    }
}
```

## Action Implementation Patterns

### Simple Movement

```rust
fn move_up(&mut self, window: &mut Window, cx: &mut Context<Self>) {
    self.change_selections(None, window, cx, |s| {
        s.move_cursors_with(|map, cursor, goal| {
            movement::up(map, cursor, goal)
        });
    });
}
```

### Editing with Undo

```rust
fn insert(&mut self, text: &str, cx: &mut Context<Self>) {
    self.transact(cx, |editor, cx| {
        editor.buffer.update(cx, |buffer, cx| {
            let edits = editor.selections.all::<usize>(cx)
                .iter()
                .map(|selection| {
                    (selection.start..selection.end, text)
                });

            buffer.edit(edits, None, cx);
        });

        editor.change_selections(None, cx, |s| {
            s.move_cursors_with(|map, cursor, _| {
                // Move cursor after inserted text
            });
        });
    });
}
```

### Async Action

```rust
fn format(&mut self, window: &mut Window, cx: &mut Context<Self>) {
    let buffer = self.buffer.clone();
    let project = self.project.clone();

    cx.spawn(async move |editor, cx| {
        let edits = project.format(&buffer, cx).await?;

        editor.update(cx, |editor, cx| {
            editor.apply_edits(edits, cx);
        })?;

        Ok(())
    }).detach_and_log_err(cx);
}
```

## Keybindings

Actions are bound to keys in keymaps:

```json
{
  "context": "Editor",
  "bindings": {
    "up": "editor::MoveUp",
    "down": "editor::MoveDown",
    "cmd-/": "editor::ToggleComments",
    "cmd-shift-f": "editor::Format",
    "cmd-left_bracket": "editor::Outdent",
    "cmd-right_bracket": "editor::Indent"
  }
}
```

### Context Predicates

```json
{
  "context": "Editor && mode == 'full'",
  "bindings": {
    "ctrl-p": "editor::MoveUp"
  }
}
```

## Dispatching Actions

### From Code

```rust
// Dispatch by action instance
window.dispatch_action(&MoveUp, cx);

// Dispatch by name
window.dispatch_action("editor::MoveUp".into(), cx);

// With parameters
window.dispatch_action(Box::new(MoveTo { line: 10, column: 0 }), cx);
```

### From Focus Handle

```rust
focus_handle.dispatch_action(&MoveUp, window, cx);
```

## Custom Actions

To add a new action:

1. **Define the action:**
```rust
#[derive(Clone, PartialEq, Deserialize)]
pub struct MyCustomAction {
    pub param: String,
}

impl_actions!(editor, [MyCustomAction]);
```

2. **Register handler:**
```rust
cx.on_action(|editor: &mut Editor, action: &MyCustomAction, window, cx| {
    editor.handle_my_custom_action(action, window, cx);
});
```

3. **Add to keymap:**
```json
{
  "bindings": {
    "ctrl-shift-x": ["editor::MyCustomAction", { "param": "value" }]
  }
}
```

## Further Reading

- [GPUI Events and Actions](../01-gpui/events-and-actions.md)
- [Editor Architecture](./architecture.md)
- [Movements](./movements.md)
