# GPUI - GPU-Accelerated UI Framework

**Path:** `/home/user/zed/crates/gpui/`
**Purpose:** Custom UI framework powering Zed's interface

---

## Overview

GPUI (GPU UI) is a high-performance, GPU-accelerated UI framework built specifically for Zed. It combines the best aspects of immediate-mode and retained-mode GUI paradigms to create a framework optimized for code editing.

### Why GPUI?

Instead of using existing UI frameworks like egui, iced, or GTK, Zed uses a custom framework for several critical reasons:

**Performance**:
- Direct GPU rendering for all UI elements
- Minimal CPU overhead for layout and rendering
- Optimized for 60+ FPS with large text documents
- Zero garbage collection pauses

**Text Editing Optimizations**:
- Custom text layout and shaping optimized for code
- Efficient glyph caching and rendering
- Support for ligatures, variable fonts, and complex scripts
- Pixel-perfect text rendering

**Collaboration**:
- Built-in support for real-time collaborative features
- Efficient state synchronization
- Minimal re-rendering on remote updates

**Native Feel**:
- Direct integration with platform windowing systems
- Native menus, dialogs, and file pickers
- Platform-specific keyboard handling

### Architecture Philosophy

GPUI follows an **immediate-mode-inspired, retained-mode** architecture:

- **Retained Mode**: The framework maintains a tree of UI elements between frames
- **Immediate Mode API**: Components describe their UI every frame using a declarative API
- **Incremental Updates**: Only changed portions of the UI tree are updated

This hybrid approach provides:
- The simplicity of immediate-mode APIs (like React or SwiftUI)
- The performance of retained-mode rendering
- Fine-grained control over what re-renders

---

## Core Concepts

GPUI is built around several key abstractions:

### 1. **App** - The Root Context

The `App` type represents the root application context. It provides:
- Access to global state
- Entity creation and management
- Task spawning and scheduling

```rust
fn my_function(cx: &mut App) {
    // Create entities
    let my_state = cx.new(|cx| MyState::new(cx));

    // Access globals
    let theme = cx.global::<ThemeRegistry>();

    // Spawn tasks
    cx.spawn(async move |cx| {
        // ...
    }).detach();
}
```

### 2. **Window** - Window State

The `Window` type represents a window's state and drawing context:
- Focus management
- Action dispatch
- Direct drawing
- Input state

```rust
fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) {
    // Check focus
    if window.is_focused(&self.focus_handle) {
        // ...
    }

    // Dispatch action
    window.dispatch_action("editor::MoveUp".into(), cx);
}
```

### 3. **Context<T>** - Entity Update Context

When updating an entity, you receive a `Context<T>`:
- Dereferences to `App` for app-level operations
- Entity-specific operations (emit events, notify)
- Subscription management

```rust
impl MyView {
    fn update(&mut self, cx: &mut Context<Self>) {
        // Update local state
        self.count += 1;

        // Trigger re-render
        cx.notify();

        // Emit event
        cx.emit(MyEvent::CountChanged);
    }
}
```

### 4. **Entity<T>** - State Handle

Entities are handles to state managed by GPUI:
- Strong reference counted
- Centrally stored
- Observable and evented

```rust
let editor: Entity<Editor> = cx.new(|cx| Editor::new(cx));

// Read state
let text = editor.read(cx).text();

// Update state
editor.update(cx, |editor, cx| {
    editor.insert("hello", cx);
    cx.notify();
});
```

### 5. **Element** - UI Building Blocks

Elements are the visual building blocks:
- Layout primitives (div, flex)
- Text rendering
- Images and icons
- Custom elements

```rust
fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
    div()
        .flex()
        .flex_col()
        .gap_2()
        .child(
            label("Hello World")
                .text_color(cx.theme().foreground)
        )
}
```

### 6. **Task<T>** - Async Operations

All async operations return `Task<T>`:
- Must be awaited, stored, or detached
- Cancelled when dropped (if not detached)
- Can run on foreground or background threads

```rust
// Foreground task (has access to cx)
let task = cx.spawn(async move |this, cx| {
    let data = fetch_data().await;
    this.update(cx, |this, cx| {
        this.apply_data(data, cx);
    })
});

// Background task (no cx access)
let task = cx.background_spawn(async move {
    expensive_computation()
});
```

---

## Documentation Files

This directory contains detailed documentation on GPUI subsystems:

### 📄 [Core Concepts](./core-concepts.md)
Deep dive into App, Window, Context, and the fundamental types.
- Application lifecycle
- Context types and their uses
- When to use each context type
- Global state management

### 📄 [Entity System](./entity-system.md)
Comprehensive guide to GPUI's entity-based state management.
- Entity lifecycle
- Entity<T> vs WeakEntity<T>
- Subscriptions and observations
- Event emission
- Common patterns and anti-patterns

### 📄 [Rendering](./rendering.md)
The rendering pipeline from elements to pixels.
- Element trait and lifecycle
- Layout engine (Taffy/Flexbox)
- Paint phase
- GPU rendering
- Performance optimization

### 📄 [Events and Actions](./events-and-actions.md)
Input handling, focus management, and the action system.
- Mouse and keyboard events
- Focus chain and focus handles
- Action definition and dispatch
- Event bubbling and capture
- Keystroke matching

### 📄 [Async and Tasks](./async-tasks.md)
Async programming model in GPUI.
- Foreground vs background tasks
- Task lifecycle management
- Async contexts (AsyncApp, AsyncWindowContext)
- Spawning tasks
- Cancellation and error handling

### 📄 [Window Management](./window-management.md)
Creating and managing windows.
- Window lifecycle
- Multi-window applications
- Window options and configuration
- Platform integration
- Modal dialogs and prompts

---

## Quick Start Example

Here's a complete minimal GPUI application:

```rust
use gpui::*;

// Define your app state
struct Counter {
    count: usize,
}

// Implement Render trait to define UI
impl Render for Counter {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .gap_4()
            .child(format!("Count: {}", self.count))
            .child(
                div()
                    .child("Increment")
                    .on_click(cx.listener(|this, _event, _window, cx| {
                        this.count += 1;
                        cx.notify();
                    }))
            )
    }
}

fn main() {
    Application::new().run(|cx: &mut App| {
        // Create a window with our counter
        cx.open_window(WindowOptions::default(), |_, cx| {
            cx.new(|_cx| Counter { count: 0 })
        }).unwrap();
    });
}
```

---

## Key Files in GPUI Crate

### Core
- **`src/gpui.rs`** - Main entry point, re-exports all public API
- **`src/app.rs`** - App context and application lifecycle
- **`src/window.rs`** - Window management and rendering
- **`src/view.rs`** - Entity<T> and view system

### Elements and Rendering
- **`src/element.rs`** - Element trait and core element types
- **`src/elements/`** - Built-in elements (div, text, image, etc.)
- **`src/scene.rs`** - Scene graph for rendering
- **`src/taffy.rs`** - Layout engine integration

### Input and Actions
- **`src/action.rs`** - Action system
- **`src/keymap.rs`** - Keymap and keystroke matching
- **`src/input.rs`** - Input event types
- **`src/key_dispatch.rs`** - Action dispatch logic

### Async and Concurrency
- **`src/executor.rs`** - Task executors
- **`src/app/async_context.rs`** - Async context types

### Platform
- **`src/platform/`** - Platform abstraction layer
  - `mac/` - macOS (Cocoa) implementation
  - `linux/` - Linux (GTK/Wayland) implementation
  - `windows/` - Windows implementation

### Text
- **`src/text_system.rs`** - Text layout and shaping
- **`src/font.rs`** - Font management

---

## Design Patterns in GPUI

### Pattern 1: Entity-Component

```rust
// State is stored in entities
let state = cx.new(|cx| MyState::new(cx));

// Components/views reference entities
struct MyComponent {
    state: Entity<MyState>,
}

impl Render for MyComponent {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        let value = self.state.read(cx).value;
        div().child(format!("Value: {}", value))
    }
}
```

### Pattern 2: Event Emission

```rust
// Declare event emitter
impl EventEmitter<MyEvent> for MyState {}

// Emit events
impl MyState {
    fn do_something(&mut self, cx: &mut Context<Self>) {
        // ... do work ...
        cx.emit(MyEvent::SomethingHappened);
        cx.notify(); // Trigger re-render
    }
}

// Subscribe to events
cx.subscribe(&state, |this, _state, event, cx| {
    match event {
        MyEvent::SomethingHappened => this.handle_event(cx),
    }
})
```

### Pattern 3: Async Updates

```rust
impl MyView {
    fn fetch_data(&mut self, cx: &mut Context<Self>) {
        let task = cx.spawn(async move |this, cx| {
            let data = fetch_from_network().await?;

            this.update(cx, |this, cx| {
                this.data = Some(data);
                cx.notify();
            })?;

            Ok(())
        });

        // Store task to prevent cancellation
        self.fetch_task = Some(task);
    }
}
```

---

## Performance Considerations

### Minimize Re-renders

```rust
// ❌ Bad: Re-renders entire tree
cx.notify();

// ✅ Good: Only notify when state actually changes
if self.value != new_value {
    self.value = new_value;
    cx.notify();
}
```

### Use Weak References

```rust
// ❌ Bad: Creates reference cycle
struct Parent {
    child: Entity<Child>,
}

struct Child {
    parent: Entity<Parent>, // Cycle!
}

// ✅ Good: Use weak reference
struct Child {
    parent: WeakEntity<Parent>,
}
```

### Batch Updates

```rust
// ❌ Bad: Multiple updates
for item in items {
    entity.update(cx, |state, cx| {
        state.add(item);
        cx.notify();
    });
}

// ✅ Good: Single update
entity.update(cx, |state, cx| {
    for item in items {
        state.add(item);
    }
    cx.notify();
});
```

---

## Common Pitfalls

### 1. Forgetting to Detach Tasks

```rust
// ❌ Bad: Task is dropped and cancelled
cx.spawn(async move |cx| {
    background_work().await;
});

// ✅ Good: Detach if you want it to run independently
cx.spawn(async move |cx| {
    background_work().await;
}).detach();
```

### 2. Holding Entity References Across Await

```rust
// ❌ Bad: Cannot hold ref across await
let state = entity.read(cx);
some_async_fn().await;
state.do_something(); // Compile error!

// ✅ Good: Use snapshot or re-read after await
let value = entity.read(cx).value.clone();
some_async_fn().await;
entity.update(cx, |state, cx| {
    state.use_value(value);
});
```

### 3. Nested Entity Updates

```rust
// ❌ Bad: Nested update causes panic
entity.update(cx, |state, cx| {
    entity.update(cx, |state, cx| { // Panic: already borrowed!
        // ...
    });
});

// ✅ Good: Separate updates
entity.update(cx, |state, cx| {
    // First update
});
entity.update(cx, |state, cx| {
    // Second update
});
```

---

## Testing with GPUI

GPUI provides a test context for unit testing:

```rust
#[gpui::test]
fn test_my_component(cx: &mut TestAppContext) {
    let window = cx.add_window(|cx| MyComponent::new(cx));

    window.update(cx, |component, cx| {
        component.do_something(cx);
        assert_eq!(component.value, 42);
    });
}
```

---

## Migration from Old API

If you see old code, here are the migrations:

| Old API | New API |
|---------|---------|
| `AppContext` | `App` |
| `ModelContext<T>` | `Context<T>` |
| `ViewContext<T>` | `Context<T>` (with `Window` parameter) |
| `WindowContext` | `Window` (+ `App` for app context) |
| `Model<T>` | `Entity<T>` |
| `View<T>` | `Entity<T>` (that implements `Render`) |

---

## Further Reading

- [Core Concepts](./core-concepts.md) - Deep dive into fundamental types
- [Entity System](./entity-system.md) - State management patterns
- [Rendering](./rendering.md) - How UI is drawn
- [Events and Actions](./events-and-actions.md) - Input handling
- [Async Tasks](./async-tasks.md) - Async programming model
- [Window Management](./window-management.md) - Working with windows

For the main architecture overview, see [Architecture Overview](../00-overview.md).
