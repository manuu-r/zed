# Architecture Decisions

Understanding the key architectural choices that shape Zed.

## Why Entity-Based State Management?

### The Problem

Traditional UI frameworks struggle with:
- Memory leaks from circular references
- Complex state synchronization
- Inefficient re-renders
- Difficult async coordination

### The Solution: Entities

```rust
pub struct Entity<T> {
    // Smart pointer to heap-allocated state
    // Automatic lifecycle management
    // Observer pattern built-in
}
```

**Benefits:**
- Clear ownership semantics
- Automatic cleanup
- Efficient updates (only observers notified)
- Safe async access

### Example

```rust
// Create entity
let editor = cx.build_entity(|cx| Editor::new(cx));

// Update triggers observers
editor.update(cx, |editor, cx| {
    editor.modify();
    cx.notify();  // Observers notified
});

// Weak references prevent leaks
struct Component {
    editor: WeakEntity<Editor>,  // Won't prevent Editor from being dropped
}
```

## Why Async Everywhere?

### The Problem

Blocking operations freeze the UI:
- File I/O
- Network requests
- Heavy computation

### The Solution: Async Rust

All potentially blocking operations are async:

```rust
// Spawn on foreground thread
cx.spawn(async move |cx| {
    let data = load_file().await;
    // Update UI
})

// Background work
cx.background_spawn(async move {
    expensive_computation()
})
```

**Benefits:**
- Responsive UI
- Efficient resource usage
- Cancellable operations

## Why GPUI?

### Why Build a Custom UI Framework?

**Requirements:**
- Native performance
- GPU acceleration
- Precise control over rendering
- Cross-platform consistency

**Existing frameworks didn't meet all requirements:**
- Web-based: Too slow, inconsistent
- Native widgets: Platform inconsistencies
- Immediate mode: Inefficient for complex UIs

### GPUI Design

```rust
impl Render for MyComponent {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .flex()
            .child(/* ... */)
    }
}
```

**Benefits:**
- Declarative UI (like React)
- GPU-accelerated rendering
- Type-safe
- Compile-time guarantees

## Why Multiple Crates?

### Modular Architecture

```
zed (binary)
  ↓
workspace (window management)
  ↓
editor (core editing)
  ↓
language (syntax, LSP)
  ↓
gpui (UI framework)
```

**Benefits:**
- Clear separation of concerns
- Faster compilation (incremental)
- Easier testing
- Code reusability

## Design Patterns

### 1. Observer Pattern

```rust
// Subscribe to entity changes
cx.observe(&entity, |this, entity, cx| {
    // React to changes
}).detach();
```

### 2. Command Pattern

```rust
// Actions are commands
actions!(editor, [Save, Cut, Copy, Paste]);

// Dispatched through event system
window.dispatch_action(&Save, cx);
```

### 3. Strategy Pattern

```rust
// Different rendering strategies
trait Renderer {
    fn render(&self, cx: &mut RenderContext);
}

// Swap implementations
component.set_renderer(CustomRenderer);
```

## Resources

- [GPUI Design Doc](../../../crates/gpui/README.md)
- [Entity System](../../../crates/gpui/src/entity.rs)
- [Async Architecture](../../../crates/gpui/src/task.rs)
