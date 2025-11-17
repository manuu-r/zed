# GPUI Entity System

**Last Updated:** 2025-11-16
**Complexity:** Intermediate

---

## Table of Contents

1. [Overview](#overview)
2. [Entity Lifecycle](#entity-lifecycle)
3. [Entity<T> Deep Dive](#entityt-deep-dive)
4. [WeakEntity<T>](#weakentityt)
5. [Subscriptions](#subscriptions)
6. [Event Emission](#event-emission)
7. [Observations](#observations)
8. [Common Patterns](#common-patterns)
9. [Memory Management](#memory-management)
10. [Best Practices](#best-practices)

---

## Overview

GPUI's entity system is central to how state is managed in Zed. It provides:
- Centralized state storage
- Reference-counted handles
- Event emission and observation
- Automatic subscription management
- Prevention of common memory leaks

### Entity System Architecture

```
App
  │
  ├─→ EntityMap<T>
  │     │
  │     ├─→ Entity Storage (SlotMap)
  │     ├─→ Observers (Vec<Callback>)
  │     ├─→ Event Subscribers (HashMap<TypeId, Vec<Callback>>)
  │     └─→ Weak Reference Tracking
  │
  └─→ Entities
        ├─→ Entity<Editor>
        ├─→ Entity<Buffer>
        ├─→ Entity<Project>
        └─→ ...
```

### Why Entities?

**Problems Solved:**
1. **Shared Mutable State**: Entities allow multiple parts of the UI to reference the same state without `RefCell` or `Rc<RefCell<T>>`
2. **Lifecycle Management**: Entities are dropped when no strong references exist
3. **Observer Pattern**: Built-in event emission and subscription
4. **Memory Safety**: Prevents common Rust UI patterns that lead to leaks

---

## Entity Lifecycle

### Creation

Entities are created via `cx.new()`:

```rust
// Create entity
let editor = cx.new(|cx: &mut Context<Editor>| {
    Editor::new(buffer, cx)
});

// Type: Entity<Editor>
```

**What Happens:**
1. GPUI allocates storage for the entity
2. Calls your build function with `Context<T>`
3. Stores the entity in internal `SlotMap`
4. Returns `Entity<T>` handle
5. Assigns unique `EntityId`

### Reading

Access entity state immutably:

```rust
// Read with closure
let text = editor.read(cx).text();

// Read with callback
let result = editor.read_with(cx, |editor, cx| {
    editor.compute_something(cx)
});
```

**Borrow Rules:**
- Multiple readers allowed simultaneously
- No writers while reading
- Panics if violated (like `RefCell`)

### Updating

Modify entity state:

```rust
editor.update(cx, |editor, cx| {
    editor.insert_text("hello", cx);
    cx.notify();
});
```

**What Happens:**
1. GPUI locks entity for mutation
2. Calls your closure with `&mut Editor` and `&mut Context<Editor>`
3. Observers are notified if `cx.notify()` was called
4. Events are emitted if `cx.emit()` was called
5. Entity is unlocked

### Destruction

Entities are dropped when no strong references exist:

```rust
{
    let editor = cx.new(|cx| Editor::new(cx));
    // editor exists
} // editor is dropped here

// All subscriptions to editor are automatically cleaned up
```

**Cleanup:**
1. `Drop` implementation runs
2. Subscriptions are removed
3. Observers are notified (optional)
4. Storage is freed

---

## Entity<T> Deep Dive

### Structure

```rust
pub struct Entity<T> {
    entity_id: EntityId,
    entity_type: PhantomData<T>,
}
```

Despite the simple structure, `Entity<T>` is powerful:
- **Strong Reference**: Keeps entity alive
- **Type-Safe**: Can only access as type `T`
- **Clone**: Cheap to clone (just increments ref count)
- **Send**: Can be sent across threads (but not used until back on foreground)

### EntityId

Each entity has a unique ID:

```rust
let id = editor.entity_id();
// Type: EntityId

// Use for comparison
if entity1.entity_id() == entity2.entity_id() {
    println!("Same entity");
}

// Use as HashMap key
let mut map = HashMap::<EntityId, String>::new();
map.insert(editor.entity_id(), "My Editor".to_string());
```

### Methods

#### entity_id()

```rust
pub fn entity_id(&self) -> EntityId
```

Returns the unique identifier for this entity.

#### downgrade()

```rust
pub fn downgrade(&self) -> WeakEntity<T>
```

Creates a weak reference. Useful for:
- Breaking reference cycles
- Async closures
- Optional references

```rust
let weak_editor = editor.downgrade();

// Later...
if let Some(editor) = weak_editor.upgrade() {
    // Entity still exists
    editor.update(cx, |editor, cx| {
        // ...
    });
}
```

#### read()

```rust
pub fn read<'a>(&self, cx: &'a App) -> &'a T
```

Immutable access to entity state:

```rust
let text = editor.read(cx).text();
let cursor = editor.read(cx).cursor_position();
```

**Important:** The reference is tied to `cx` lifetime, not entity lifetime:

```rust
// ❌ Bad: Can't hold reference across cx borrow
let editor_ref = editor.read(cx);
do_something_else(cx); // Borrow checker error!
editor_ref.text();

// ✅ Good: Read and use immediately
let text = editor.read(cx).text().to_string();
do_something_else(cx); // OK
```

#### read_with()

```rust
pub fn read_with<R>(&self, cx: &App, f: impl FnOnce(&T, &App) -> R) -> R
```

Read with a closure that receives both the entity and context:

```rust
let diagnostics = buffer.read_with(cx, |buffer, cx| {
    let language = buffer.language();
    language.diagnostics(buffer, cx)
});
```

#### update()

```rust
pub fn update<R>(&self, cx: &mut App, f: impl FnOnce(&mut T, &mut Context<T>) -> R) -> R
```

Mutable access to entity:

```rust
let cursor_moved = editor.update(cx, |editor, cx| {
    let old_pos = editor.cursor_position();
    editor.move_down(cx);
    let new_pos = editor.cursor_position();
    old_pos != new_pos
});
```

**The `cx` Parameter:**
The closure receives `&mut Context<T>`, which provides:
- All `App` methods (via `Deref`)
- Entity-specific methods (`notify`, `emit`, `subscribe`, etc.)

#### update_in()

```rust
pub fn update_in(&self, cx: &mut AsyncWindowContext, f: impl FnOnce(&mut T, &mut Window, &mut Context<T>) -> R) -> Result<R>
```

Update with window access (in async contexts):

```rust
cx.spawn_in(window, async move |this, cx| {
    let result = fetch_data().await;

    this.update_in(cx, |this, window, cx| {
        this.data = result;
        window.focus(&this.focus_handle);
        cx.notify();
    })?;

    Ok(())
})
```

---

## WeakEntity<T>

### Overview

`WeakEntity<T>` is a weak reference that doesn't keep the entity alive.

```rust
pub struct WeakEntity<T> {
    entity_id: EntityId,
    entity_type: PhantomData<T>,
}
```

### When to Use

1. **Breaking Reference Cycles**

```rust
struct Parent {
    children: Vec<Entity<Child>>,
}

struct Child {
    parent: WeakEntity<Parent>, // Weak to break cycle
}
```

2. **Async Closures**

```rust
// Spawn with weak reference
cx.spawn(async move |this: WeakEntity<Self>, cx| {
    let data = fetch_data().await;

    // Entity might have been dropped
    this.update(cx, |this, cx| {
        this.apply_data(data, cx);
    }).ok(); // Returns Result
});
```

3. **Optional References**

```rust
struct Assistant {
    active_editor: WeakEntity<Editor>, // Might be closed
}

impl Assistant {
    fn update_from_editor(&mut self, cx: &mut Context<Self>) {
        if let Some(editor) = self.active_editor.upgrade() {
            // Editor still exists
            let text = editor.read(cx).text();
            // ...
        }
    }
}
```

### Methods

#### upgrade()

```rust
pub fn upgrade(&self) -> Option<Entity<T>>
```

Attempts to create a strong reference:

```rust
match weak_entity.upgrade() {
    Some(entity) => {
        // Entity still alive
        entity.update(cx, |state, cx| {
            // ...
        });
    }
    None => {
        // Entity was dropped
    }
}
```

#### read(), update(), etc.

`WeakEntity` has the same methods as `Entity`, but they return `Result`:

```rust
// read() returns Result
let text = weak_editor.read(cx)?.text();

// update() returns Result
weak_editor.update(cx, |editor, cx| {
    editor.insert_text("hello", cx);
})?;

// Common pattern: use ok() to ignore if entity is gone
weak_editor.update(cx, |editor, cx| {
    editor.update_something(cx);
}).ok();
```

---

## Subscriptions

### Overview

Subscriptions allow entities to listen for events from other entities.

### Creating Subscriptions

```rust
impl MyView {
    fn new(editor: Entity<Editor>, cx: &mut Context<Self>) -> Self {
        let subscription = cx.subscribe(&editor, |this, editor, event, cx| {
            this.handle_editor_event(editor, event, cx);
        });

        Self {
            editor,
            _subscription: subscription,
        }
    }

    fn handle_editor_event(
        &mut self,
        editor: Entity<Editor>,
        event: &EditorEvent,
        cx: &mut Context<Self>,
    ) {
        match event {
            EditorEvent::Edited => {
                self.on_edited(cx);
            }
            EditorEvent::SelectionsChanged => {
                self.on_selections_changed(cx);
            }
            _ => {}
        }
    }
}
```

### Subscription Lifecycle

```
Create Subscription
  │
  ├─→ Store in field (_subscription, _subscriptions)
  │
  ├─→ Callback invoked when event is emitted
  │
  └─→ Drop Subscription
        │
        └─→ Automatically unsubscribes
```

**Critical:** Subscriptions must be stored or they'll be dropped:

```rust
// ❌ Bad: Subscription dropped immediately
cx.subscribe(&editor, |this, editor, event, cx| {
    // Never called!
});

// ✅ Good: Subscription stored
let subscription = cx.subscribe(&editor, |this, editor, event, cx| {
    // Called when events are emitted
});
self._subscription = subscription;

// ✅ Also Good: Multiple subscriptions
let mut subscriptions = Vec::new();
subscriptions.push(cx.subscribe(&editor, ...));
subscriptions.push(cx.subscribe(&buffer, ...));
self._subscriptions = subscriptions;
```

### Multiple Subscriptions

```rust
pub struct ComplexView {
    _subscriptions: Vec<Subscription>,
}

impl ComplexView {
    fn new(
        editor: Entity<Editor>,
        project: Entity<Project>,
        cx: &mut Context<Self>,
    ) -> Self {
        let mut subscriptions = Vec::new();

        subscriptions.push(cx.subscribe(&editor, Self::handle_editor_event));
        subscriptions.push(cx.subscribe(&project, Self::handle_project_event));

        Self {
            _subscriptions: subscriptions,
        }
    }

    fn handle_editor_event(
        &mut self,
        _: Entity<Editor>,
        event: &EditorEvent,
        cx: &mut Context<Self>,
    ) {
        // Handle editor events
    }

    fn handle_project_event(
        &mut self,
        _: Entity<Project>,
        event: &ProjectEvent,
        cx: &mut Context<Self>,
    ) {
        // Handle project events
    }
}
```

---

## Event Emission

### Declaring Event Emitters

First, define your event enum:

```rust
#[derive(Clone, Debug, PartialEq, Eq)]
pub enum BufferEvent {
    Edited,
    Saved,
    FileHandleChanged,
    Reloaded,
}
```

Then implement `EventEmitter`:

```rust
impl EventEmitter<BufferEvent> for Buffer {}
```

### Emitting Events

Call `cx.emit()` during entity update:

```rust
impl Buffer {
    fn apply_edit(&mut self, edit: Edit, cx: &mut Context<Self>) {
        // Apply the edit
        self.text.edit(edit);

        // Emit event
        cx.emit(BufferEvent::Edited);

        // Trigger re-render
        cx.notify();
    }

    fn save(&mut self, path: PathBuf, cx: &mut Context<Self>) -> Task<Result<()>> {
        cx.spawn(async move |this, cx| {
            // Save to disk
            fs::write(&path, content).await?;

            // Emit saved event
            this.update(cx, |this, cx| {
                this.saved_at = Some(Instant::now());
                cx.emit(BufferEvent::Saved);
                cx.notify();
            })?;

            Ok(())
        })
    }
}
```

### Event Pattern

```
Entity::update()
  │
  ├─→ Modify state
  │
  ├─→ cx.emit(Event)
  │     │
  │     └─→ Queued for delivery
  │
  ├─→ cx.notify()
  │     │
  │     └─→ Mark for re-render
  │
  └─→ End of update()
        │
        └─→ Events delivered to subscribers
```

### Event Delivery

Events are delivered **after** the update completes:

```rust
editor.update(cx, |editor, cx| {
    editor.insert_text("hello", cx);
    cx.emit(EditorEvent::Edited);
    // Subscribers NOT called yet
});
// Subscribers called here
```

This prevents re-entrance issues and keeps state consistent.

---

## Observations

### Overview

Observations are simpler than subscriptions - they're called whenever the entity notifies:

```rust
let observation = cx.observe(&editor, |this, editor, cx| {
    // Called whenever editor.cx.notify() is called
    this.update_from_editor(editor, cx);
});
```

### Observe vs Subscribe

| Observe | Subscribe |
|---------|-----------|
| Called on every `cx.notify()` | Called only when events emitted |
| No event data | Receives event data |
| Simpler | More control |
| Broader | More specific |

### When to Use Observe

Use `observe` when you want to react to **any** change:

```rust
impl StatusBar {
    fn new(editor: Entity<Editor>, cx: &mut Context<Self>) -> Self {
        // Update status bar whenever editor changes
        let observation = cx.observe(&editor, |this, editor, cx| {
            let cursor = editor.read(cx).cursor_position();
            let line_count = editor.read(cx).line_count();

            this.cursor_display = format!("{}:{}", cursor.row, cursor.column);
            this.line_count_display = format!("{} lines", line_count);

            cx.notify();
        });

        Self {
            editor,
            _observation: observation,
            cursor_display: String::new(),
            line_count_display: String::new(),
        }
    }
}
```

### When to Use Subscribe

Use `subscribe` when you need to react to **specific events**:

```rust
impl DiagnosticsView {
    fn new(buffer: Entity<Buffer>, cx: &mut Context<Self>) -> Self {
        // Only update when buffer is edited or saved
        let subscription = cx.subscribe(&buffer, |this, buffer, event, cx| {
            match event {
                BufferEvent::Edited | BufferEvent::Saved => {
                    this.refresh_diagnostics(buffer, cx);
                }
                _ => {}
            }
        });

        Self {
            buffer,
            _subscription: subscription,
        }
    }
}
```

---

## Common Patterns

### Pattern 1: Parent-Child with Subscriptions

```rust
pub struct FileTree {
    project: Entity<Project>,
    expanded_entries: HashSet<ProjectEntryId>,
    _subscriptions: Vec<Subscription>,
}

impl FileTree {
    pub fn new(project: Entity<Project>, cx: &mut Context<Self>) -> Self {
        let mut subscriptions = Vec::new();

        // Subscribe to project events
        subscriptions.push(cx.subscribe(&project, |this, project, event, cx| {
            match event {
                ProjectEvent::WorktreeAdded(_) |
                ProjectEvent::WorktreeRemoved(_) => {
                    this.rebuild_tree(project, cx);
                }
                _ => {}
            }
        }));

        Self {
            project,
            expanded_entries: HashSet::default(),
            _subscriptions: subscriptions,
        }
    }

    fn rebuild_tree(&mut self, project: Entity<Project>, cx: &mut Context<Self>) {
        // Rebuild tree from project state
        cx.notify();
    }
}
```

### Pattern 2: Weak Reference in Async

```rust
pub struct AsyncWorker {
    result: Option<WorkResult>,
    _task: Option<Task<()>>,
}

impl AsyncWorker {
    pub fn start_work(&mut self, cx: &mut Context<Self>) {
        let task = cx.spawn(async move |this, cx| {
            // Use weak reference to avoid keeping entity alive
            let result = perform_async_work().await;

            // Update only if entity still exists
            this.update(cx, |this, cx| {
                this.result = Some(result);
                cx.notify();
            }).ok(); // Ignore error if entity was dropped
        });

        self._task = Some(task);
    }
}
```

### Pattern 3: Dynamic Subscriptions

```rust
pub struct MultiEditorView {
    editors: Vec<Entity<Editor>>,
    subscriptions: HashMap<EntityId, Subscription>,
}

impl MultiEditorView {
    pub fn add_editor(&mut self, editor: Entity<Editor>, cx: &mut Context<Self>) {
        let entity_id = editor.entity_id();

        // Subscribe to new editor
        let subscription = cx.subscribe(&editor, |this, editor, event, cx| {
            this.handle_editor_event(editor, event, cx);
        });

        self.editors.push(editor.clone());
        self.subscriptions.insert(entity_id, subscription);

        cx.notify();
    }

    pub fn remove_editor(&mut self, editor_id: EntityId, cx: &mut Context<Self>) {
        self.editors.retain(|e| e.entity_id() != editor_id);

        // Subscription automatically unsubscribes when dropped
        self.subscriptions.remove(&editor_id);

        cx.notify();
    }
}
```

### Pattern 4: Conditional Observation

```rust
pub struct ConditionalObserver {
    observed_entity: Option<Entity<SomeEntity>>,
    _observation: Option<Subscription>,
}

impl ConditionalObserver {
    pub fn observe_entity(&mut self, entity: Option<Entity<SomeEntity>>, cx: &mut Context<Self>) {
        // Clear old observation
        self._observation = None;

        // Set up new observation if entity provided
        if let Some(entity) = entity {
            let observation = cx.observe(&entity, |this, entity, cx| {
                this.handle_change(entity, cx);
            });

            self.observed_entity = Some(entity);
            self._observation = Some(observation);
        } else {
            self.observed_entity = None;
        }

        cx.notify();
    }
}
```

---

## Memory Management

### Reference Cycles

**Problem:**
Entities holding strong references to each other create cycles:

```rust
// ❌ Bad: Reference cycle
struct Parent {
    child: Entity<Child>,
}

struct Child {
    parent: Entity<Parent>, // Cycle!
}

// Neither will ever be dropped
```

**Solution:**
Use weak references to break cycles:

```rust
// ✅ Good: Weak reference breaks cycle
struct Parent {
    children: Vec<Entity<Child>>,
}

struct Child {
    parent: WeakEntity<Parent>, // Weak breaks cycle
}

impl Child {
    fn notify_parent(&self, cx: &mut Context<Self>) {
        if let Some(parent) = self.parent.upgrade() {
            parent.update(cx, |parent, cx| {
                parent.child_changed(cx);
            });
        }
    }
}
```

### Subscription Lifecycle

Subscriptions are automatically cleaned up:

```rust
{
    let view = cx.new(|cx| {
        let subscription = cx.subscribe(&editor, |this, editor, event, cx| {
            // ...
        });

        MyView {
            _subscription: subscription,
        }
    });

    // Subscription is active
}
// View dropped → Subscription dropped → Automatically unsubscribed
```

**Why This Works:**
1. Subscription stored in view
2. When view is dropped, its fields are dropped
3. When subscription is dropped, it unsubscribes

### Preventing Leaks

**Store Subscriptions:**

```rust
// ❌ Leak: Subscription dropped but callback still registered
cx.subscribe(&entity, |this, entity, event, cx| {
    // ...
});

// ✅ No leak: Subscription stored
self._subscription = cx.subscribe(&entity, |this, entity, event, cx| {
    // ...
});
```

**Use Weak References in Async:**

```rust
// ❌ Potential leak: Strong reference keeps entity alive
let entity = entity.clone();
cx.spawn(async move |cx| {
    loop {
        delay(Duration::from_secs(1)).await;
        entity.update(cx, |entity, cx| {
            // Entity can never be dropped while this runs!
        });
    }
}).detach();

// ✅ No leak: Weak reference allows entity to be dropped
let entity = entity.downgrade();
cx.spawn(async move |cx| {
    loop {
        delay(Duration::from_secs(1)).await;
        if entity.update(cx, |entity, cx| {
            // ...
        }).is_err() {
            break; // Entity was dropped, exit loop
        }
    }
}).detach();
```

---

## Best Practices

### 1. Always Store Subscriptions

```rust
// ❌ Bad
impl MyView {
    fn new(editor: Entity<Editor>, cx: &mut Context<Self>) -> Self {
        cx.subscribe(&editor, |this, editor, event, cx| {
            // Never called - subscription dropped!
        });

        Self { editor }
    }
}

// ✅ Good
impl MyView {
    fn new(editor: Entity<Editor>, cx: &mut Context<Self>) -> Self {
        let subscription = cx.subscribe(&editor, |this, editor, event, cx| {
            // Called when events emitted
        });

        Self {
            editor,
            _subscription: subscription,
        }
    }
}
```

### 2. Use Weak References in Cycles

```rust
// ❌ Bad
struct Parent {
    child: Entity<Child>,
}

struct Child {
    parent: Entity<Parent>, // Cycle!
}

// ✅ Good
struct Child {
    parent: WeakEntity<Parent>, // Breaks cycle
}
```

### 3. Call notify() After State Changes

```rust
// ❌ Bad
impl Counter {
    fn increment(&mut self, cx: &mut Context<Self>) {
        self.count += 1;
        // Forgot cx.notify() - UI won't update!
    }
}

// ✅ Good
impl Counter {
    fn increment(&mut self, cx: &mut Context<Self>) {
        self.count += 1;
        cx.notify(); // UI will update
    }
}
```

### 4. Emit Events for Observable Changes

```rust
// ❌ Bad
impl Buffer {
    fn edit(&mut self, edit: Edit, cx: &mut Context<Self>) {
        self.apply_edit(edit);
        cx.notify();
        // Subscribers won't know what changed!
    }
}

// ✅ Good
impl Buffer {
    fn edit(&mut self, edit: Edit, cx: &mut Context<Self>) {
        self.apply_edit(edit);
        cx.emit(BufferEvent::Edited);
        cx.notify();
    }
}
```

### 5. Handle Option in Weak Reference Updates

```rust
// ❌ Bad
weak_entity.update(cx, |entity, cx| {
    entity.do_something(cx);
}); // Panics if entity was dropped!

// ✅ Good
weak_entity.update(cx, |entity, cx| {
    entity.do_something(cx);
}).ok(); // Returns None if entity was dropped

// ✅ Or
if let Some(entity) = weak_entity.upgrade() {
    entity.update(cx, |entity, cx| {
        entity.do_something(cx);
    });
}
```

---

## Summary

### Entity System Cheat Sheet

```rust
// Create entity
let entity = cx.new(|cx| MyState::new());

// Read entity
let value = entity.read(cx).value;

// Update entity
entity.update(cx, |state, cx| {
    state.value = 42;
    cx.notify();
});

// Weak reference
let weak = entity.downgrade();
if let Some(entity) = weak.upgrade() {
    // Entity still exists
}

// Subscribe to events
let subscription = cx.subscribe(&entity, |this, entity, event, cx| {
    // Handle event
});
self._subscription = subscription;

// Observe changes
let observation = cx.observe(&entity, |this, entity, cx| {
    // Handle any change
});
self._observation = observation;

// Emit events
impl EventEmitter<MyEvent> for MyState {}

impl MyState {
    fn do_something(&mut self, cx: &mut Context<Self>) {
        cx.emit(MyEvent::SomethingHappened);
        cx.notify();
    }
}
```

### Key Takeaways

1. Entities are reference-counted state handles
2. Always store subscriptions or they'll be dropped
3. Use `WeakEntity` to break reference cycles
4. Call `cx.notify()` when state changes
5. Emit events for significant changes
6. Use `observe` for any change, `subscribe` for specific events
7. Weak references in async closures prevent leaks

---

## Further Reading

- [Core Concepts](./core-concepts.md) - Context types
- [Events and Actions](./events-and-actions.md) - Action system
- [Async Tasks](./async-tasks.md) - Async programming
- [Architecture Overview](../00-overview.md) - Overall architecture
