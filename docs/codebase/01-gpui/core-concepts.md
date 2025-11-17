# GPUI Core Concepts

**Last Updated:** 2025-11-16
**Complexity:** Intermediate to Advanced

---

## Table of Contents

1. [Introduction](#introduction)
2. [The Context Hierarchy](#the-context-hierarchy)
3. [App - Root Application Context](#app---root-application-context)
4. [Window - Window State and Drawing](#window---window-state-and-drawing)
5. [Context<T> - Entity Update Context](#contextt---entity-update-context)
6. [Async Contexts](#async-contexts)
7. [Application Lifecycle](#application-lifecycle)
8. [Global State](#global-state)
9. [Common Patterns](#common-patterns)
10. [Best Practices](#best-practices)

---

## Introduction

GPUI's context types are the foundation of the framework. Understanding when and how to use each context type is crucial for effective GPUI development.

### The Big Picture

```
Application
    │
    ├─→ App (Root Context)
    │     │
    │     ├─→ Manages all entities
    │     ├─→ Manages all windows
    │     ├─→ Holds global state
    │     └─→ Provides task executors
    │
    ├─→ Window (Window-Specific State)
    │     │
    │     ├─→ Focus management
    │     ├─→ Action dispatch
    │     ├─→ Drawing/rendering
    │     └─→ Input state
    │
    └─→ Context<T> (Entity Update Context)
          │
          ├─→ Dereferences to App
          ├─→ Entity-specific operations
          ├─→ Event emission
          └─→ Notification (trigger re-render)
```

### Context Type Quick Reference

| Context Type | When You See It | Primary Purpose |
|--------------|-----------------|-----------------|
| `App` | `cx: &App` or `cx: &mut App` | Read/create entities, access globals |
| `Window` | `window: &mut Window` | Focus, actions, drawing |
| `Context<T>` | `cx: &mut Context<T>` | Update entity `T` |
| `AsyncApp` | `cx: AsyncApp` in async | Async operations on app context |
| `AsyncWindowContext` | `cx: AsyncWindowContext` in async | Async operations with window |

---

## The Context Hierarchy

GPUI uses multiple context types to provide different capabilities at different layers:

### Layer 1: Application (App)

```rust
fn main() {
    Application::new().run(|cx: &mut App| {
        // cx is the root App context
        // Available during app initialization
    });
}
```

The `App` context provides:
- Entity creation (`cx.new()`)
- Entity access (`entity.read(cx)`, `entity.update(cx, ...)`)
- Global state (`cx.global::<T>()`, `cx.set_global()`)
- Task spawning (`cx.spawn()`, `cx.background_spawn()`)
- Window creation (`cx.open_window()`)

### Layer 2: Window

```rust
fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
    // window provides window-specific capabilities
}
```

The `Window` provides:
- Focus management (`window.focus()`, `window.is_focused()`)
- Action dispatch (`window.dispatch_action()`)
- Drawing primitives
- Input state queries
- Window properties (bounds, appearance, etc.)

### Layer 3: Context<T>

```rust
entity.update(cx, |state, cx: &mut Context<State>| {
    // cx is a Context<State>
});
```

`Context<T>` combines:
- All capabilities of `App` (via Deref)
- Entity-specific operations:
  - `cx.notify()` - trigger re-render
  - `cx.emit(event)` - emit event
  - `cx.subscribe()` - subscribe to other entities
  - `cx.observe()` - observe entity changes

### Layer 4: Async Contexts

```rust
cx.spawn(async move |this: WeakEntity<T>, cx: AsyncApp| {
    // cx is an AsyncApp - can be held across await points
});
```

Async contexts can be held across `await` points:
- `AsyncApp` - async version of `App`
- `AsyncWindowContext` - async version of `App` with window access

---

## App - Root Application Context

### Overview

`App` is the root context type. It represents the entire application state and provides access to:
- All entities
- All windows
- Global state
- Task executors
- Platform services

### File Location

**Source:** `/home/user/zed/crates/gpui/src/app.rs`

### Key Responsibilities

```rust
impl App {
    // Entity Management
    pub fn new<T>(&mut self, build: impl FnOnce(&mut Context<T>) -> T) -> Entity<T>;
    pub fn update_entity<T, R>(&mut self, entity: &Entity<T>, ...) -> R;
    pub fn read_entity<T, R>(&self, entity: &Entity<T>, ...) -> R;

    // Global State
    pub fn global<G: Global>(&self) -> &G;
    pub fn global_mut<G: Global>(&mut self) -> &mut G;
    pub fn set_global<G: Global>(&mut self, global: G);

    // Task Spawning
    pub fn spawn<R>(&self, f: impl Future<Output = R>) -> Task<R>;
    pub fn background_spawn<R>(&self, f: impl Future<Output = R>) -> Task<R>;

    // Window Management
    pub fn open_window<V>(&mut self, options: WindowOptions, build: impl FnOnce(...) -> V)
        -> Result<WindowHandle<V>>;

    // Executors
    pub fn foreground_executor(&self) -> &ForegroundExecutor;
    pub fn background_executor(&self) -> &BackgroundExecutor;

    // Quit
    pub fn quit(&mut self);
}
```

### Creating Entities

The primary way to create state in GPUI is via `cx.new()`:

```rust
// Basic entity creation
let counter = cx.new(|cx| Counter {
    count: 0,
});

// Entity with initialization logic
let editor = cx.new(|cx| {
    let mut editor = Editor::new();
    editor.set_buffer(buffer, cx);
    editor.register_actions(cx);
    editor
});
```

**Parameters:**
- `build`: A closure that receives `&mut Context<T>` and returns `T`
- Returns: `Entity<T>` - a handle to the created entity

**When to use:**
- Creating new application state
- Instantiating views
- Creating services or managers

### Accessing Entities

Reading entity state:

```rust
// Read-only access
let text = editor.read(cx).text();

// With closure
let result = editor.read_with(cx, |editor, cx| {
    editor.compute_something(cx)
});
```

Updating entity state:

```rust
// Mutable access
editor.update(cx, |editor, cx| {
    editor.insert_text("hello", cx);
    cx.notify(); // Trigger re-render
});

// With return value
let cursor_pos = editor.update(cx, |editor, cx| {
    editor.move_cursor_down(cx);
    editor.cursor_position()
});
```

**Important:** The inner `cx` in the closure must be used, not the outer `cx`:

```rust
// ❌ Bad
let outer_cx = cx;
editor.update(outer_cx, |editor, cx| {
    // Using outer_cx here would cause borrow checker errors
    let global = outer_cx.global::<Theme>(); // Error!
});

// ✅ Good
editor.update(cx, |editor, cx| {
    // Use the inner cx
    let global = cx.global::<Theme>(); // OK
});
```

### Global State

GPUI provides a global state mechanism for singleton services:

```rust
// Define a global
pub struct ThemeRegistry {
    themes: Vec<Theme>,
}

impl Global for ThemeRegistry {}

// Set global (usually during app initialization)
cx.set_global(ThemeRegistry {
    themes: vec![],
});

// Access global (read-only)
let themes = cx.global::<ThemeRegistry>();

// Access global (mutable)
cx.update_global::<ThemeRegistry, _>(|registry, cx| {
    registry.themes.push(new_theme);
});
```

**Common Globals in Zed:**
- `SettingsStore` - Application settings
- `ThemeRegistry` - Available themes
- `LanguageRegistry` - Language definitions
- `Client` - Collaboration client
- `WorkspaceStore` - Workspace state

**Best Practices:**
- Use globals for true singletons only
- Prefer entities over globals when possible
- Globals are convenient but less composable
- Use `Default` trait for default initialization

### Task Spawning

Spawn async tasks that run on the foreground thread:

```rust
// Foreground task (can access cx)
cx.spawn(async move |cx| {
    let data = fetch_data().await;

    cx.update(|cx| {
        // Use data
    }).ok();
}).detach();
```

Spawn async tasks on background thread pool:

```rust
// Background task (pure computation)
cx.background_spawn(async move {
    expensive_computation()
}).detach();
```

**Key Differences:**

| Foreground | Background |
|------------|------------|
| Runs on main thread | Runs on thread pool |
| Can access entities via cx | No cx access |
| Lower throughput | Higher throughput |
| For UI updates | For CPU-intensive work |

### Window Management

Create windows:

```rust
// Simple window
cx.open_window(WindowOptions::default(), |_, cx| {
    cx.new(|_| MyView::new())
})?;

// Window with custom options
cx.open_window(
    WindowOptions {
        bounds: WindowBounds::Maximized,
        titlebar: Some(TitlebarOptions {
            title: Some("My App".into()),
            ..Default::default()
        }),
        ..Default::default()
    },
    |_, cx| cx.new(|_| MyView::new())
)?;
```

**WindowOptions Fields:**
- `bounds`: Initial size/position
- `titlebar`: Title bar configuration
- `center`: Center on screen
- `focus`: Focus on creation
- `show`: Show immediately
- `kind`: Normal, PopUp, or Utility
- `is_movable`: User can move window
- `app_id`: Application identifier

### Quitting the Application

```rust
// Request quit (can be cancelled)
cx.quit();

// Quit modes
Application::new()
    .with_quit_mode(QuitMode::Explicit) // Explicit cx.quit() call
    .run(|cx| {
        // ...
    });
```

---

## Window - Window State and Drawing

### Overview

`Window` represents a single window's state. It's passed alongside `Context` in rendering and update methods.

### File Location

**Source:** `/home/user/zed/crates/gpui/src/window.rs`

### Key Responsibilities

```rust
impl Window {
    // Focus Management
    pub fn focus<E: FocusableView>(&mut self, view: &E);
    pub fn blur(&mut self);
    pub fn is_focused(&self, handle: &FocusHandle) -> bool;
    pub fn focused(&self) -> Option<FocusHandle>;

    // Action Dispatch
    pub fn dispatch_action(&mut self, action: Box<dyn Action>, cx: &mut App);
    pub fn dispatch_keystroke(&mut self, keystroke: &Keystroke, cx: &mut App);

    // Window Properties
    pub fn bounds(&self) -> Bounds<Pixels>;
    pub fn viewport_size(&self) -> Size<Pixels>;
    pub fn appearance(&self) -> WindowAppearance;

    // Prompts and Dialogs
    pub fn prompt(&mut self,
        level: PromptLevel,
        message: &str,
        detail: Option<&str>,
        answers: &[&str],
        cx: &mut App
    ) -> oneshot::Receiver<usize>;

    // Refresh
    pub fn refresh(&mut self);
}
```

### Focus Management

Focus is crucial for keyboard input routing:

```rust
// Create focus handle
pub struct MyView {
    focus_handle: FocusHandle,
}

impl MyView {
    fn new(cx: &mut Context<Self>) -> Self {
        Self {
            focus_handle: cx.focus_handle(),
        }
    }

    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .track_focus(&self.focus_handle)
            .on_action(cx.listener(|this, action: &MyAction, window, cx| {
                // This action only fires when focused
            }))
    }
}

// Programmatically set focus
impl MyView {
    fn focus_self(&mut self, window: &mut Window, cx: &mut Context<Self>) {
        window.focus(&self.focus_handle);
    }

    fn check_focus(&self, window: &Window) -> bool {
        window.is_focused(&self.focus_handle)
    }
}
```

**Focus Chain:**
Actions bubble up the focus chain from the focused element to the root.

```
Focused Element
    │
    ▼
Parent Element
    │
    ▼
Workspace
    │
    ▼
Root
```

### Action Dispatch

Actions are routed through the window:

```rust
// Dispatch by name
window.dispatch_action("editor::MoveDown".into(), cx);

// Dispatch with data
window.dispatch_action(Box::new(MyAction { data: 42 }), cx);

// Dispatch keystroke (for testing)
window.dispatch_keystroke(
    &Keystroke::parse("ctrl-p").unwrap(),
    cx
);
```

### Window Properties

Query window state:

```rust
fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
    let bounds = window.bounds();
    let size = window.viewport_size();
    let appearance = window.appearance();

    div()
        .w(size.width)
        .h(size.height)
        .child(format!("Window size: {}x{}", size.width, size.height))
}
```

### Prompts and Dialogs

Show native prompts:

```rust
fn confirm_delete(&mut self, window: &mut Window, cx: &mut Context<Self>) {
    let answer = window.prompt(
        PromptLevel::Warning,
        "Delete file?",
        Some("This cannot be undone."),
        &["Delete", "Cancel"],
        cx
    );

    cx.spawn_in(window, async move |this, cx| {
        let choice = answer.await.ok()?;

        if choice == 0 {
            // User clicked "Delete"
            this.update(cx, |this, cx| {
                this.perform_delete(cx);
            })?;
        }

        Some(())
    }).detach();
}
```

**PromptLevel:**
- `Info` - Informational message
- `Warning` - Warning message
- `Critical` - Critical/error message

### Refresh

Force a complete re-render:

```rust
// Bypass all caching and re-render everything
window.refresh();
```

**Use sparingly** - this is expensive. Usually `cx.notify()` is sufficient.

---

## Context<T> - Entity Update Context

### Overview

`Context<T>` is provided when updating an entity of type `T`. It combines app-level capabilities with entity-specific operations.

### File Location

**Source:** `/home/user/zed/crates/gpui/src/app/context.rs`

### Deref to App

`Context<T>` dereferences to `App`, so all `App` methods are available:

```rust
impl SomeEntity {
    fn update(&mut self, cx: &mut Context<Self>) {
        // App methods work
        let other = cx.new(|cx| OtherEntity::new());
        let theme = cx.global::<ThemeRegistry>();

        // Entity-specific methods
        cx.notify();
        cx.emit(SomeEvent);
    }
}
```

### Notification

`cx.notify()` tells GPUI that this entity has changed and needs re-rendering:

```rust
impl Counter {
    fn increment(&mut self, cx: &mut Context<Self>) {
        self.count += 1;
        cx.notify(); // Mark for re-render
    }
}
```

**When to call `cx.notify()`:**
- After changing state that affects rendering
- After changing state that observers need to know about
- When implementing `Render` trait for a view

**When NOT to call:**
- During rendering (already being rendered)
- If state didn't actually change
- If change doesn't affect UI or observers

### Event Emission

Emit events to notify subscribers:

```rust
// Define event
#[derive(Clone, Debug)]
pub enum EditorEvent {
    SelectionsChanged,
    Edited,
    BufferChanged,
}

// Declare emitter
impl EventEmitter<EditorEvent> for Editor {}

// Emit event
impl Editor {
    fn change_selection(&mut self, cx: &mut Context<Self>) {
        self.selections = new_selections;
        cx.emit(EditorEvent::SelectionsChanged);
        cx.notify();
    }
}
```

### Subscriptions

Subscribe to events from other entities:

```rust
impl MyView {
    fn new(editor: &Entity<Editor>, cx: &mut Context<Self>) -> Self {
        // Subscribe to editor events
        let subscription = cx.subscribe(editor, |this, editor, event, cx| {
            match event {
                EditorEvent::Edited => this.handle_edit(editor, cx),
                _ => {}
            }
        });

        Self {
            _subscription: subscription,
        }
    }
}
```

**Subscription Lifecycle:**
- Created via `cx.subscribe()` or `cx.observe()`
- Returns `Subscription` handle
- Automatically unsubscribes when dropped
- Store in a field (often named `_subscription` or `_subscriptions`)

### Observations

Observe any changes to an entity:

```rust
impl MyView {
    fn new(editor: &Entity<Editor>, cx: &mut Context<Self>) -> Self {
        // Called whenever editor.cx.notify() is called
        let subscription = cx.observe(editor, |this, editor, cx| {
            this.update_from_editor(editor, cx);
        });

        Self {
            _subscription: subscription,
        }
    }
}
```

**subscribe vs observe:**

| subscribe | observe |
|-----------|---------|
| Called on specific events | Called on any cx.notify() |
| Receives event parameter | No event parameter |
| More selective | Broader |

### Focus Handle

Create focus handles for focusable elements:

```rust
pub struct TextField {
    focus_handle: FocusHandle,
    text: String,
}

impl TextField {
    fn new(cx: &mut Context<Self>) -> Self {
        Self {
            focus_handle: cx.focus_handle(),
            text: String::new(),
        }
    }
}
```

### Spawning Tasks

Spawn tasks with access to the current entity:

```rust
impl MyView {
    fn fetch_data(&mut self, cx: &mut Context<Self>) {
        cx.spawn(async move |this, cx| {
            let data = fetch_from_api().await?;

            this.update(cx, |this, cx| {
                this.data = Some(data);
                cx.notify();
            })?;

            Ok(())
        }).detach();
    }
}
```

**Note:** The first parameter to the async closure is `WeakEntity<Self>`, not `Entity<Self>`. This prevents keeping the entity alive if it should be dropped.

### Listeners

Create event handlers that capture `this`:

```rust
impl Render for Button {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .on_click(cx.listener(|this: &mut Self, event, window, cx| {
                this.handle_click(event, window, cx);
            }))
    }
}
```

**Why use `cx.listener()`?**
- Captures `this` as mutable reference to the entity
- Automatically updates the entity
- Triggers `cx.notify()` after the callback

**Signature:**
```rust
cx.listener(|this: &mut T, event: &E, window: &mut Window, cx: &mut Context<T>| {
    // ...
})
```

---

## Async Contexts

### Overview

Async contexts can be held across `await` points, enabling async operations that interact with entities and windows.

### File Location

**Source:** `/home/user/zed/crates/gpui/src/app/async_context.rs`

### AsyncApp

`AsyncApp` is the async version of `App`:

```rust
cx.spawn(async move |cx: AsyncApp| {
    // Can be held across await
    let data = fetch_data().await;

    // Update app state
    cx.update(|cx: &mut App| {
        // Use data
    }).ok();
})
```

**Key Methods:**

```rust
impl AsyncApp {
    // Update app context
    pub fn update<R>(&self, f: impl FnOnce(&mut App) -> R) -> Result<R>;

    // Update entity
    pub fn update_entity<T, R>(&self, entity: &Entity<T>, f: impl FnOnce(&mut T, &mut Context<T>) -> R) -> Result<R>;

    // Read entity
    pub fn read_entity<T, R>(&self, entity: &Entity<T>, f: impl FnOnce(&T, &App) -> R) -> Result<R>;

    // Background spawn
    pub fn background_spawn<R>(&self, future: impl Future<Output = R>) -> Task<R>;
}
```

**Pattern:**

```rust
// From foreground context
let task = cx.spawn(async move |cx| {
    // Do async work
    let result = async_operation().await;

    // Update app
    cx.update(|cx| {
        // Use result
    }).ok();
});
```

### AsyncWindowContext

`AsyncWindowContext` provides window access in async contexts:

```rust
cx.spawn_in(window, async move |this, cx: AsyncWindowContext| {
    let result = fetch_data().await;

    this.update(cx, |this, cx| {
        this.process_result(result, cx);
    })?;

    Ok(())
})
```

**Created via:**
- `cx.spawn_in(window, ...)`
- Converting from window context

**Key Methods:**

```rust
impl AsyncWindowContext {
    // Update with window access
    pub fn update<R>(&self, f: impl FnOnce(&mut Window, &mut App) -> R) -> Result<R>;

    // Update entity in window
    pub fn update_entity<T, R>(&self, entity: &Entity<T>, f: impl FnOnce(&mut T, &mut Window, &mut Context<T>) -> R) -> Result<R>;
}
```

### Error Handling

Async contexts return `Result` because the app or window might have been closed:

```rust
// Handle errors explicitly
cx.update(|cx| {
    // ...
}).log_err();

// Propagate errors
cx.update(|cx| {
    // ...
})?;

// Ignore errors
cx.update(|cx| {
    // ...
}).ok();
```

---

## Application Lifecycle

### Startup Sequence

```
main()
  │
  ├─→ Application::new()
  │     │
  │     ├─→ Create platform backend
  │     ├─→ Initialize executors
  │     └─→ Setup event loop
  │
  ├─→ .run(|cx| { ... })
  │     │
  │     ├─→ User initialization code
  │     │     │
  │     │     ├─→ cx.set_global() for singletons
  │     │     ├─→ cx.open_window() for initial window(s)
  │     │     └─→ Start background services
  │     │
  │     └─→ Enter event loop
  │
  └─→ Event Loop
        │
        ├─→ Process platform events
        ├─→ Dispatch actions
        ├─→ Update entities
        ├─→ Layout and render
        └─→ Process async task completions
```

### App Initialization

```rust
fn main() {
    Application::new().run(|cx: &mut App| {
        // 1. Set up globals
        cx.set_global(SettingsStore::default());
        cx.set_global(ThemeRegistry::default());

        // 2. Initialize services
        let language_registry = Arc::new(LanguageRegistry::new(
            cx.background_executor().clone()
        ));

        // 3. Create initial window
        cx.open_window(WindowOptions::default(), |_, cx| {
            cx.new(|cx| MyApp::new(cx))
        }).ok();

        // 4. Start background tasks
        cx.spawn(async move |cx| {
            // Background initialization
        }).detach();
    });
}
```

### Shutdown Sequence

```
User Quits
  │
  ├─→ cx.quit() called
  │
  ├─→ on_app_quit handlers called
  │     │
  │     └─→ 100ms timeout for cleanup
  │
  ├─→ Close all windows
  │
  ├─→ Drop entities
  │
  └─→ Exit event loop
```

### Cleanup Handlers

```rust
cx.on_app_quit(|cx| {
    cx.spawn(async move |cx| {
        // Save state
        save_workspace_state(cx).await.ok();
    })
}).detach();
```

---

## Global State

### The Global Trait

```rust
pub trait Global: 'static {}
```

Any type implementing `Global` can be stored as global state.

### Setting Globals

```rust
// Define a global type
pub struct AppConfig {
    pub data_dir: PathBuf,
    pub log_level: String,
}

impl Global for AppConfig {}

// Set during initialization
cx.set_global(AppConfig {
    data_dir: PathBuf::from("~/.myapp"),
    log_level: "info".to_string(),
});
```

### Accessing Globals

```rust
// Read-only access
let config = cx.global::<AppConfig>();
println!("Data dir: {:?}", config.data_dir);

// Mutable access
cx.update_global::<AppConfig, _>(|config, cx| {
    config.log_level = "debug".to_string();
});
```

### Common Patterns

**Lazy Initialization:**

```rust
pub struct LazyGlobal {
    data: Option<ExpensiveData>,
}

impl Global for LazyGlobal {}

impl LazyGlobal {
    pub fn get_or_init(&mut self, cx: &mut App) -> &ExpensiveData {
        self.data.get_or_insert_with(|| {
            ExpensiveData::new(cx)
        })
    }
}
```

**Singleton Services:**

```rust
pub struct Logger {
    file: File,
}

impl Global for Logger {}

impl Logger {
    pub fn log(&mut self, message: &str) {
        writeln!(self.file, "{}", message).ok();
    }
}

// Access anywhere
cx.update_global::<Logger, _>(|logger, _cx| {
    logger.log("Something happened");
});
```

---

## Common Patterns

### Pattern 1: Entity with Subscriptions

```rust
pub struct MyView {
    editor: Entity<Editor>,
    buffer: Entity<Buffer>,
    _subscriptions: Vec<Subscription>,
}

impl MyView {
    pub fn new(
        editor: Entity<Editor>,
        buffer: Entity<Buffer>,
        cx: &mut Context<Self>,
    ) -> Self {
        let mut subscriptions = Vec::new();

        // Subscribe to editor events
        subscriptions.push(cx.subscribe(&editor, Self::handle_editor_event));

        // Observe buffer changes
        subscriptions.push(cx.observe(&buffer, |this, buffer, cx| {
            this.on_buffer_changed(buffer, cx);
        }));

        Self {
            editor,
            buffer,
            _subscriptions: subscriptions,
        }
    }

    fn handle_editor_event(
        &mut self,
        _editor: Entity<Editor>,
        event: &EditorEvent,
        cx: &mut Context<Self>,
    ) {
        match event {
            EditorEvent::Edited => self.on_edit(cx),
            _ => {}
        }
    }

    fn on_buffer_changed(&mut self, buffer: Entity<Buffer>, cx: &mut Context<Self>) {
        // React to buffer changes
        cx.notify();
    }

    fn on_edit(&mut self, cx: &mut Context<Self>) {
        // React to edits
        cx.notify();
    }
}
```

### Pattern 2: Async Data Fetching

```rust
pub struct DataView {
    data: Option<Data>,
    loading: bool,
    _fetch_task: Option<Task<()>>,
}

impl DataView {
    fn fetch_data(&mut self, cx: &mut Context<Self>) {
        self.loading = true;
        cx.notify();

        let task = cx.spawn(async move |this, cx| {
            let data = fetch_from_api().await.ok()?;

            this.update(cx, |this, cx| {
                this.data = Some(data);
                this.loading = false;
                cx.notify();
            }).ok();

            Some(())
        });

        self._fetch_task = Some(task);
    }
}
```

### Pattern 3: Focus Management

```rust
pub struct FocusableView {
    focus_handle: FocusHandle,
    focused: bool,
}

impl FocusableView {
    pub fn new(cx: &mut Context<Self>) -> Self {
        let focus_handle = cx.focus_handle();

        Self {
            focus_handle,
            focused: false,
        }
    }

    fn focus(&mut self, window: &mut Window, cx: &mut Context<Self>) {
        window.focus(&self.focus_handle);
        self.focused = true;
        cx.notify();
    }

    fn blur(&mut self, cx: &mut Context<Self>) {
        self.focused = false;
        cx.notify();
    }
}

impl Render for FocusableView {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
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
    }
}
```

---

## Best Practices

### 1. Minimize Notify Calls

```rust
// ❌ Bad: Unnecessary notifies
impl Counter {
    fn maybe_increment(&mut self, increment: bool, cx: &mut Context<Self>) {
        if increment {
            self.count += 1;
        }
        cx.notify(); // Called even when nothing changed!
    }
}

// ✅ Good: Only notify when changed
impl Counter {
    fn maybe_increment(&mut self, increment: bool, cx: &mut Context<Self>) {
        if increment {
            self.count += 1;
            cx.notify(); // Only when actually changed
        }
    }
}
```

### 2. Use Weak References in Async

```rust
// ❌ Bad: Strong reference in async
cx.spawn(async move |this: Entity<Self>, cx| {
    // Keeps entity alive even if window is closed
});

// ✅ Good: Weak reference
cx.spawn(async move |this: WeakEntity<Self>, cx| {
    // Entity can be dropped
    this.update(cx, |this, cx| {
        // ...
    }).ok(); // Handle the Result
});
```

### 3. Store Subscriptions

```rust
// ❌ Bad: Subscription dropped immediately
impl MyView {
    fn new(entity: Entity<Other>, cx: &mut Context<Self>) -> Self {
        cx.subscribe(&entity, |this, entity, event, cx| {
            // This will never be called - subscription was dropped!
        });

        Self { }
    }
}

// ✅ Good: Store subscription
impl MyView {
    fn new(entity: Entity<Other>, cx: &mut Context<Self>) -> Self {
        let subscription = cx.subscribe(&entity, |this, entity, event, cx| {
            // This will be called
        });

        Self {
            _subscription: subscription,
        }
    }
}
```

### 4. Avoid Nested Updates

```rust
// ❌ Bad: Nested update panics
entity.update(cx, |state, cx| {
    entity.update(cx, |state, cx| {
        // Panic: already borrowed!
    });
});

// ✅ Good: Sequential updates
entity.update(cx, |state, cx| {
    state.step_one();
});

entity.update(cx, |state, cx| {
    state.step_two();
});
```

### 5. Detach or Store Tasks

```rust
// ❌ Bad: Task dropped and cancelled
fn do_work(&mut self, cx: &mut Context<Self>) {
    cx.spawn(async move |cx| {
        important_work().await;
    });
    // Task is dropped here and cancelled!
}

// ✅ Good: Detach if fire-and-forget
fn do_work(&mut self, cx: &mut Context<Self>) {
    cx.spawn(async move |cx| {
        important_work().await;
    }).detach();
}

// ✅ Or: Store if you need to cancel it
fn do_work(&mut self, cx: &mut Context<Self>) {
    self.task = Some(cx.spawn(async move |cx| {
        important_work().await;
    }));
}
```

### 6. Use Inner cx in Closures

```rust
// ❌ Bad: Using outer cx
let outer = cx;
entity.update(outer, |state, cx| {
    let global = outer.global::<Theme>(); // Borrow checker error!
});

// ✅ Good: Using inner cx
entity.update(cx, |state, cx| {
    let global = cx.global::<Theme>(); // OK
});
```

---

## Summary

### Context Type Decision Tree

```
Do you need to...

├─ Create entities?
│  └─→ Use App
│
├─ Access globals?
│  └─→ Use App (or Context<T> which derefs to App)
│
├─ Update an entity?
│  └─→ Use Context<T>
│
├─ Notify about state changes?
│  └─→ Use Context<T>.notify()
│
├─ Emit events?
│  └─→ Use Context<T>.emit()
│
├─ Manage focus?
│  └─→ Use Window
│
├─ Dispatch actions?
│  └─→ Use Window
│
├─ Hold across await?
│  └─→ Use AsyncApp or AsyncWindowContext
│
└─ Render UI?
   └─→ Use Window + Context<T>
```

### Key Takeaways

1. **App** = Root context, creates entities, holds globals
2. **Window** = Window-specific state, focus, actions, drawing
3. **Context<T>** = Entity update context, notifies, emits, subscribes
4. **AsyncApp/AsyncWindowContext** = Async versions for across-await use
5. Store subscriptions in fields or they'll be dropped
6. Call `cx.notify()` when state changes that affects rendering
7. Use weak references in async closures
8. Detach or store tasks, don't drop them

---

## Further Reading

- [Entity System](./entity-system.md) - Deep dive into Entity<T>
- [Rendering](./rendering.md) - How rendering works
- [Events and Actions](./events-and-actions.md) - Action system
- [Async Tasks](./async-tasks.md) - Async programming in GPUI
- [Architecture Overview](../00-overview.md) - Overall Zed architecture
