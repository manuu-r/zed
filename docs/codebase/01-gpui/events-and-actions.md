# GPUI Events and Actions

**Last Updated:** 2025-11-16

---

## Actions System

Actions are user commands that can be triggered by keyboard shortcuts, menu items, or programmatically.

### Defining Actions

```rust
// Simple actions (no data)
actions!(editor, [MoveUp, MoveDown, MoveLeft, MoveRight]);

// Actions with data
#[derive(Clone, PartialEq, Deserialize)]
pub struct MoveTo {
    pub line: u32,
    pub column: u32,
}

impl_actions!(editor, [MoveTo]);
```

### Registering Action Handlers

```rust
impl Editor {
    fn register_actions(cx: &mut Context<Self>) {
        cx.on_action(|editor: &mut Editor, action: &MoveUp, window, cx| {
            editor.move_up(window, cx);
        });

        cx.on_action(|editor: &mut Editor, action: &MoveTo, window, cx| {
            editor.move_to(action.line, action.column, window, cx);
        });
    }
}
```

### Dispatching Actions

```rust
// From window
window.dispatch_action("editor::MoveUp".into(), cx);

// With data
window.dispatch_action(Box::new(MoveTo { line: 10, column: 5 }), cx);

// From focus handle
focus_handle.dispatch_action(&MoveUp, window, cx);
```

## Event System

### Mouse Events

```rust
div()
    .on_click(|event, window, cx| {
        println!("Clicked at {:?}", event.position);
    })
    .on_mouse_down(MouseButton::Left, |event, window, cx| {
        // Handle mouse down
    })
    .on_mouse_up(MouseButton::Left, |event, window, cx| {
        // Handle mouse up
    })
    .on_mouse_move(|event, window, cx| {
        // Handle mouse move
    })
```

### Keyboard Events

```rust
div()
    .track_focus(&focus_handle)
    .on_key_down(|event, window, cx| {
        if event.keystroke.key == "Enter" {
            // Handle enter
        }
    })
    .on_action(cx.listener(|this, action: &MyAction, window, cx| {
        this.handle_action(action, window, cx);
    }))
```

### Focus Events

```rust
div()
    .track_focus(&self.focus_handle)
    .on_focus_in(cx.listener(|this, _event, _window, cx| {
        this.focused = true;
        cx.notify();
    }))
    .on_focus_out(cx.listener(|this, _event, _window, cx| {
        this.focused = false;
        cx.notify();
    }))
```

## Focus Management

### Creating Focus Handles

```rust
pub struct TextField {
    focus_handle: FocusHandle,
}

impl TextField {
    fn new(cx: &mut Context<Self>) -> Self {
        Self {
            focus_handle: cx.focus_handle(),
        }
    }

    fn focus(&self, window: &mut Window) {
        window.focus(&self.focus_handle);
    }

    fn is_focused(&self, window: &Window) -> bool {
        window.is_focused(&self.focus_handle)
    }
}
```

### Focus Chain

Actions bubble up the focus chain:

```
TextField (focused)
    ↓
Container
    ↓
Panel
    ↓
Workspace
    ↓
Root
```

### Tracking Focus

```rust
impl Render for TextField {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .track_focus(&self.focus_handle)  // Required for keyboard events
            .on_key_down(cx.listener(|this, event, window, cx| {
                this.handle_key(event, window, cx);
            }))
    }
}
```

## Dispatch Phases

### Capture Phase

Events are dispatched from root to focused element:

```rust
div()
    .on_click_in(DispatchPhase::Capture, |event, window, cx| {
        // Called before child handlers
        event.stop_propagation(); // Prevent bubbling
    })
```

### Bubble Phase (Default)

Events bubble from focused element to root:

```rust
div()
    .on_click(|event, window, cx| {
        // Called after child handlers (default)
    })
```

## Keystroke Matching

### Keymap

```rust
cx.bind_keys([
    KeyBinding::new("ctrl-p", editor::actions::MoveUp, Some("Editor")),
    KeyBinding::new("cmd-s", workspace::actions::Save, Some("Workspace")),
]);
```

### Context Predicates

```rust
KeyBinding::new("enter", Submit, Some("PromptEditor && !menu_open"))
```

## Common Patterns

### Listener Pattern

```rust
div()
    .on_click(cx.listener(|this: &mut Self, event, window, cx| {
        this.handle_click(event, window, cx);
    }))
```

### Conditional Event Handling

```rust
div()
    .when(self.enabled, |div| {
        div.on_click(cx.listener(|this, event, window, cx| {
            this.handle_click(event, window, cx);
        }))
    })
```

### Preventing Propagation

```rust
div()
    .on_click(|event, window, cx| {
        event.stop_propagation();  // Don't bubble to parent
        // Handle click
    })
```

## Further Reading

- [Core Concepts](./core-concepts.md)
- [Window Management](./window-management.md)
