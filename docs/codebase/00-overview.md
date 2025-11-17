# Zed Architecture Overview

**Last Updated:** 2025-11-16
**Audience:** Contributors, maintainers, and developers interested in understanding Zed's internal architecture

---

## Table of Contents

1. [Introduction](#introduction)
2. [High-Level Architecture](#high-level-architecture)
3. [Core Crates and Dependencies](#core-crates-and-dependencies)
4. [Application Bootstrapping](#application-bootstrapping)
5. [Event Loop and Threading Model](#event-loop-and-threading-model)
6. [Memory Management](#memory-management)
7. [Data Flow](#data-flow)
8. [Key Design Patterns](#key-design-patterns)
9. [Performance Considerations](#performance-considerations)
10. [Extension Points](#extension-points)

---

## Introduction

Zed is a high-performance, collaborative code editor built from the ground up in Rust. The architecture is designed around several core principles:

- **Performance First**: Native Rust implementation, careful memory management, and efficient rendering
- **Collaboration Built-in**: Real-time collaborative editing is a first-class feature, not an afterthought
- **Extensibility**: Plugin system using WebAssembly for sandboxed extensions
- **Modern UI**: Custom GPU-accelerated UI framework (GPUI) for maximum performance and flexibility
- **Language Server Protocol**: First-class LSP integration for language features

### Key Architectural Decisions

1. **Custom UI Framework**: Instead of using existing UI frameworks, Zed uses GPUI, a custom immediate-mode-inspired retained-mode UI framework designed specifically for high-performance text editing
2. **Single Foreground Thread**: All UI rendering and entity updates happen on one thread, simplifying concurrency
3. **Entity-Component System**: State management using an entity system similar to ECS patterns in game engines
4. **Async-First**: Extensive use of async/await for I/O operations, LSP communication, and collaboration
5. **Tree-sitter**: Incremental parsing for syntax highlighting and structural navigation

---

## High-Level Architecture

Zed's architecture can be visualized as a layered system:

```
┌─────────────────────────────────────────────────────────────────┐
│                         Application Layer                        │
│  (zed crate - main entry point, workspace management, menus)   │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────────┐
│                      Feature Crates Layer                        │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────────┐  │
│  │ editor   │workspace │ project  │  vim     │   terminal   │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │file_finder│collab_ui│diagnostics│ theme   │ settings_ui  │  │
│  └──────────┴──────────┴──────────┴──────────┴──────────────┘  │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────────┐
│                     Core Services Layer                          │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────────┐  │
│  │ language │   lsp    │  git     │  rpc     │ multi_buffer │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │  text    │  fuzzy   │  fs      │  db      │   settings   │  │
│  └──────────┴──────────┴──────────┴──────────┴──────────────┘  │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────────┐
│                      Foundation Layer                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                       GPUI Framework                       │  │
│  │  (UI primitives, event handling, windowing, rendering)   │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────┬──────────┬──────────┬──────────────────────────┐  │
│  │collections│  util   │  rope    │    platform-specific     │  │
│  └──────────┴──────────┴──────────┴──────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

**Foundation Layer**:
- GPUI: Core UI framework, window management, event dispatch, rendering pipeline
- Collections: Optimized data structures (SumTree, TreeMap, etc.)
- Util: Common utilities, async helpers, string manipulation
- Rope: Efficient text buffer representation using ropes
- Platform: OS-specific implementations (macOS, Linux, Windows)

**Core Services Layer**:
- Language: Language definitions, syntax highlighting, Tree-sitter integration
- LSP: Language Server Protocol client implementation
- Git: Git integration, diff display, blame information
- RPC: Network protocol for collaboration
- Multi Buffer: Managing multiple buffer excerpts in a single editor
- Text: Core text manipulation primitives
- FS: File system abstraction and watching
- DB: SQLite-based persistence
- Settings: Configuration management

**Feature Crates Layer**:
- Editor: The main editor component with all editing capabilities
- Workspace: Window layout, panes, docks, tabs
- Project: Project management, worktrees, language servers
- Vim: Vim mode implementation
- Terminal: Integrated terminal
- File Finder: Fuzzy file finder
- Collab UI: Collaboration UI components
- Diagnostics: Displaying compiler/linter errors
- Theme: Theme system
- Settings UI: Settings editor

**Application Layer**:
- Zed: Main application entry point, initialization, menu handling
- Platform integration: Native menus, window management, system integration

---

## Core Crates and Dependencies

### Critical Path Crates

These crates form the backbone of Zed:

#### 1. **gpui** (`/home/user/zed/crates/gpui/`)

The GPU-accelerated UI framework powering Zed.

**Key Responsibilities**:
- Window creation and management
- Event loop and dispatch
- Layout engine (using Taffy - a Rust flexbox implementation)
- Rendering pipeline (2D vector graphics, text rendering)
- Entity system for state management
- Action system for commands
- Keyboard and mouse input handling
- Focus management
- Async task scheduling

**Key Types**:
- `App`: Root application context
- `Window`: Window state and drawing context
- `Context<T>`: Entity update context
- `Entity<T>`: Handle to state
- `Element`: UI element trait
- `Task<T>`: Async task handle
- `Action`: User commands

**Dependencies**:
- `taffy`: Flexbox layout
- `cosmic-text`: Text shaping and layout
- `blade`: Graphics abstraction (supports Metal, Vulkan, DX12)
- `smol`: Async runtime

#### 2. **editor** (`/home/user/zed/crates/editor/`)

The core text editing component.

**Key Responsibilities**:
- Text editing operations (insert, delete, undo/redo)
- Multi-cursor support
- Selection handling
- Display mapping (buffer coordinates → screen coordinates)
- Syntax highlighting
- Code folding
- Inlay hints
- Completions and code actions
- Hover information
- Find and replace
- Git diff display
- Collaborative cursors

**Key Types**:
- `Editor`: Main editor entity
- `DisplayMap`: Maps between buffer and display coordinates
- `MultiBufferSnapshot`: Snapshot of buffer(s) being edited
- `Selection`: Represents a cursor/selection
- `Anchor`: Stable text position that tracks through edits

**Sub-modules**:
- `display_map/`: Coordinate mapping with folds, wraps, inlays
- `element/`: Editor rendering
- `movement/`: Cursor movement logic
- `actions/`: All editor actions

#### 3. **project** (`/home/user/zed/crates/project/`)

Project and workspace file management.

**Key Responsibilities**:
- Worktree management (file watching)
- Buffer lifecycle management
- Language server lifecycle
- Project-wide search
- Diagnostics aggregation
- Git status tracking
- Prettier integration
- Task management
- Debugging support (DAP)

**Key Types**:
- `Project`: Main project entity
- `Worktree`: File tree with watching
- `BufferStore`: Manages all open buffers
- `LspStore`: Manages language servers
- `GitStore`: Git repository information

#### 4. **workspace** (`/home/user/zed/crates/workspace/`)

Window layout and pane management.

**Key Responsibilities**:
- Pane layout (splits, tabs)
- Dock management (left, right, bottom)
- Item lifecycle (editors, terminals, etc.)
- Command palette integration
- Status bar
- Notifications
- Modal layers
- Workspace persistence
- Drag and drop

**Key Types**:
- `Workspace`: Main workspace entity
- `Pane`: Tab container
- `PaneGroup`: Split pane layout
- `Dock`: Side panels
- `Item`: Trait for pane items (editors, terminals, etc.)

#### 5. **language** (`/home/user/zed/crates/language/`)

Language support infrastructure.

**Key Responsibilities**:
- Language definitions
- Tree-sitter parser management
- Syntax highlighting queries
- Outline/symbols extraction
- Bracket matching
- Auto-indent logic
- Code context extraction
- Toolchain management

**Key Types**:
- `Language`: Language definition
- `LanguageRegistry`: Central registry
- `Buffer`: Text buffer with language information
- `Syntax`: Tree-sitter integration

#### 6. **lsp** (`/home/user/zed/crates/lsp/`)

Language Server Protocol client.

**Key Responsibilities**:
- LSP message protocol
- Request/response handling
- Server lifecycle management
- Capability negotiation
- Workspace symbols
- Diagnostics

**Key Types**:
- `LanguageServer`: LSP server connection
- `LanguageServerId`: Unique server identifier
- Various LSP types following the spec

#### 7. **text** (`/home/user/zed/crates/text/`)

Core text manipulation primitives.

**Key Responsibilities**:
- Rope data structure for efficient text storage
- Anchor tracking through edits
- Operation composition
- CRDT for collaborative editing

**Key Types**:
- `Rope`: Efficient string representation
- `Anchor`: Stable position in text
- `Point`: (row, column) position
- `Offset`: Byte offset
- `Edit`: Text edit operation

#### 8. **rpc** (`/home/user/zed/crates/rpc/`)

Network protocol for collaboration.

**Key Responsibilities**:
- Message serialization/deserialization
- Connection management
- Request/response correlation
- Streaming support

**Key Types**:
- `TypedEnvelope<T>`: Typed message envelope
- `proto`: Protocol buffer definitions

### Dependency Graph

```
zed
 ├── workspace
 │    ├── project
 │    │    ├── language
 │    │    ├── lsp
 │    │    ├── fs
 │    │    ├── git
 │    │    └── rpc
 │    ├── editor
 │    │    ├── language
 │    │    ├── text
 │    │    └── multi_buffer
 │    ├── terminal
 │    └── gpui
 ├── collab_ui
 │    └── workspace
 ├── vim
 │    └── editor
 └── gpui
      ├── collections
      ├── util
      └── platform (macOS, Linux, Windows)
```

---

## Application Bootstrapping

The application initialization follows a carefully orchestrated sequence:

### 1. Entry Point (`/home/user/zed/crates/zed/src/main.rs`)

```rust
pub fn main() {
    // 1. Record startup time
    STARTUP_TIME.get_or_init(|| Instant::now());

    // 2. Parse command-line arguments
    let args = Args::parse();

    // 3. Handle special modes (askpass, crash handler, etc.)

    // 4. Initialize crash reporting

    // 5. Setup paths and directories

    // 6. Start the GPUI application
    Application::new()
        .with_quit_mode(QuitMode::Explicit)
        .run(move |cx| {
            // Application initialization
        })
}
```

### 2. Initialization Sequence

```
main()
  │
  ├─→ Parse CLI args
  │
  ├─→ Initialize crash handler
  │
  ├─→ Create necessary directories
  │
  ├─→ Application::new().run() ─→ GPUI takes over
  │                                 │
  │                                 ├─→ Create event loop
  │                                 ├─→ Initialize platform (Cocoa/GTK/Win32)
  │                                 ├─→ Setup rendering backend (Metal/Vulkan/DX12)
  │                                 └─→ Call user initialization closure
  │
  ├─→ Initialize globals:
  │     ├─→ ReleaseChannel
  │     ├─→ SettingsStore
  │     ├─→ KeymapStore
  │     ├─→ ThemeRegistry
  │     ├─→ LanguageRegistry
  │     ├─→ Client
  │     └─→ WorkspaceStore
  │
  ├─→ Setup file watchers for config
  │
  ├─→ Initialize language servers
  │
  ├─→ Restore or create workspace
  │     ├─→ Load persisted workspace
  │     ├─→ Create window
  │     ├─→ Initialize Workspace entity
  │     └─→ Open files/restore tabs
  │
  └─→ Enter event loop
```

### 3. Global State Initialization

Many components in Zed use the `Global` trait for singleton state:

```rust
// In initialization code:
cx.set_global(SettingsStore::new());
cx.set_global(ThemeRegistry::new());
cx.set_global(LanguageRegistry::new());

// Later accessed via:
let settings = cx.global::<SettingsStore>();
```

**Key Globals**:
- `SettingsStore`: User settings and config
- `ThemeRegistry`: Available themes
- `LanguageRegistry`: Language definitions
- `Client`: Collaboration client
- `NodeRuntime`: Node.js runtime for extensions
- `WorkspaceStore`: Workspace restoration state

### 4. Workspace Creation

A workspace is created for each window:

```rust
let workspace = cx.new(|window, cx| {
    Workspace::new(
        workspace_id,
        project,
        app_state,
        window,
        cx
    )
});

// Open window with workspace
cx.open_window(
    build_window_options(),
    |_, cx| cx.new(|_| workspace)
);
```

---

## Event Loop and Threading Model

### Single Foreground Thread Model

Zed uses a **single-threaded foreground model** for all UI and entity updates. This greatly simplifies reasoning about state:

```
┌─────────────────────────────────────────────────────────────┐
│                     Foreground Thread                        │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              GPUI Event Loop                           │ │
│  │                                                        │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  1. Process platform events (keyboard, mouse)    │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  2. Dispatch actions                             │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  3. Update entities                              │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  4. Layout UI                                    │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  5. Paint (render to GPU)                        │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │  6. Process async task completions               │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   Background Thread Pool                     │
│  ┌────────────┬────────────┬────────────┬────────────────┐  │
│  │File I/O    │LSP Comms   │Git Ops     │Syntax Parsing │  │
│  │Tasks       │Tasks       │Tasks       │Tasks          │  │
│  └────────────┴────────────┴────────────┴────────────────┘  │
└─────────────────────────────────────────────────────────────┘
          │                    │                    │
          └────────────────────┴────────────────────┘
                               │
                  Results sent back to foreground
```

### Async Task Model

Zed uses an async task model for all I/O operations:

```rust
// Spawn on foreground thread (can access cx)
cx.spawn(async move |this, cx| {
    // Can await and interact with entities
    let result = some_async_operation().await;
    this.update(cx, |this, cx| {
        this.apply_result(result, cx);
    })
})

// Spawn on background thread (no cx access)
cx.background_spawn(async move {
    // Pure computation, file I/O, network, etc.
    let result = expensive_computation().await;
    result
})
```

**Task Lifecycle**:
1. Task created via `cx.spawn()` or `cx.background_spawn()`
2. Returns `Task<T>` handle
3. Task must be either:
   - Awaited (`.await`)
   - Detached (`.detach()`, `.detach_and_log_err()`)
   - Stored (in a struct field)
4. If dropped without detaching, the task is cancelled

### Executor Model

GPUI uses `smol` as its async runtime:

```rust
// Foreground executor (single-threaded)
let executor = cx.foreground_executor();
executor.spawn(async { ... });

// Background executor (thread pool)
let executor = cx.background_executor();
executor.spawn(async { ... });
```

**Thread Pools**:
- **Foreground**: 1 thread (the main UI thread)
- **Background**: Configurable thread pool (defaults to number of CPU cores)
- **Platform**: Platform-specific threads (e.g., file watching)

---

## Memory Management

### Entity System

Zed uses an entity system for managing application state:

```rust
// Create an entity
let editor = cx.new(|cx| Editor::new(buffer, cx));
// Type: Entity<Editor>

// Read entity state
editor.read(cx).selection();

// Update entity state
editor.update(cx, |editor, cx| {
    editor.change_selections(selections, cx);
});
```

**Entity Lifecycle**:
1. Created via `cx.new()`
2. Stored as `Entity<T>` (strong reference)
3. Can be downgraded to `WeakEntity<T>`
4. Dropped when no strong references exist

**Benefits**:
- Centralized entity storage
- Automatic subscription management
- Prevents most reference cycles
- Mutable access without RefCell

### Memory Patterns

**Reference Counting**:
```rust
// Arc for shared ownership
let buffer: Arc<Buffer> = ...;

// Weak references to prevent cycles
let weak_editor: WeakEntity<Editor> = editor.downgrade();
```

**Snapshots**:
Many data structures provide cheap snapshots:

```rust
// Buffer snapshot - immutable view of buffer at point in time
let snapshot = buffer.snapshot();

// Can be sent to background threads safely
cx.background_spawn(async move {
    process_text(snapshot.text());
})
```

**Copy-on-Write**:
```rust
// Shared string - either &'static str or Arc<str>
let text: SharedString = "hello".into();

// ArcCow - Arc with copy-on-write
let data: ArcCow<[u8]> = ...;
```

### Avoiding Leaks

**Subscription Patterns**:
```rust
struct MyView {
    // Subscriptions automatically unsubscribe when dropped
    _subscriptions: Vec<Subscription>,
}

impl MyView {
    fn new(editor: &Entity<Editor>, cx: &mut Context<Self>) -> Self {
        let mut subscriptions = Vec::new();

        // Subscribe to entity events
        subscriptions.push(
            cx.subscribe(editor, |this, editor, event, cx| {
                this.handle_editor_event(event, cx);
            })
        );

        Self { _subscriptions: subscriptions }
    }
}
```

**Weak References in Async**:
```rust
cx.spawn(async move |weak_this, cx| {
    let result = fetch_data().await;

    // Update only if entity still exists
    weak_this.update(cx, |this, cx| {
        this.handle_result(result, cx);
    }).ok(); // ok() to ignore error if entity was dropped
})
```

---

## Data Flow

### User Input Flow

```
User Input (keyboard/mouse)
         │
         ▼
Platform Event (NSEvent, XEvent, etc.)
         │
         ▼
GPUI Platform Layer
         │
         ▼
Window Input Handling
         │
         ├─→ Keyboard Input ──→ Key Dispatch ──→ Action
         │                                         │
         └─→ Mouse Input ─────→ Element Handlers ─┘
                                                   │
                                                   ▼
                                    Action Dispatch to Focus Chain
                                                   │
         ┌─────────────────────────────────────────┤
         │                                         │
         ▼                                         ▼
    Editor Handles                          Workspace Handles
         │                                         │
         ├─→ Update State                          ├─→ Update State
         ├─→ Call cx.notify()                      ├─→ Call cx.notify()
         └─→ Maybe cx.spawn() async                └─→ Maybe dispatch to others
                     │                                        │
                     └────────────────────────────────────────┘
                                      │
                                      ▼
                            Frame Rendering Triggered
                                      │
                                      ├─→ Layout
                                      ├─→ Paint
                                      └─→ Present to GPU
```

### LSP Communication Flow

```
Editor
  │ (request completion)
  ▼
Project
  │ (get appropriate language server)
  ▼
LspStore
  │
  ▼
LanguageServer
  │ (send LSP request)
  ▼
Background Task (JSON-RPC over stdio/socket)
  │
  │ ◄───────────────────────────────────────────┐
  │                                             │
  ▼                                             │
LSP Server                                      │
  │                                             │
  │ (response)                                  │
  └─────────────────────────────────────────────┘
  │
  ▼
Background Task receives response
  │
  ▼
Send to foreground via channel
  │
  ▼
Project handles response
  │
  ▼
Editor.update() to show completions
  │
  ▼
cx.notify() triggers re-render
```

### Collaborative Editing Flow

```
Local Edit
  │
  ▼
Editor.handle_input()
  │
  ├─→ Update local buffer
  │
  └─→ Project.buffer_store.send_operation()
        │
        ▼
      Client.send(proto::Operation)
        │
        ▼
      Network ──→ Collab Server
                      │
                      └──→ Broadcast to other clients
                              │
                              ▼
                         Remote Client
                              │
                              ▼
                         Client.receive(proto::Operation)
                              │
                              ▼
                         BufferStore.apply_remote_operation()
                              │
                              ▼
                         Buffer.apply_ops()
                              │
                              ▼
                         Editor.notify() ─→ UI updates
```

---

## Key Design Patterns

### 1. Entity-Component Pattern

Entities store state, components interact with entities:

```rust
// Entity
struct MyState {
    data: String,
}

// Create entity
let entity = cx.new(|cx| MyState {
    data: "initial".to_string()
});

// Component accesses entity
struct MyComponent {
    entity: Entity<MyState>,
}

impl MyComponent {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        let data = self.entity.read(cx).data.clone();
        div().child(data)
    }
}
```

### 2. Observer Pattern

Entities notify observers when they change:

```rust
// Entity emits events
impl EventEmitter<EditorEvent> for Editor {}

// Observer subscribes
cx.observe(&editor, |this, editor, cx| {
    // Called whenever editor.cx.notify() is called
    this.handle_editor_changed(editor, cx);
})
```

### 3. Command Pattern (Actions)

All user commands are modeled as actions:

```rust
// Define action
#[derive(Clone, PartialEq, Deserialize)]
struct MoveDown;

impl_actions!(editor, [MoveDown]);

// Register handler
fn register_actions(editor: &mut Editor, cx: &mut Context<Editor>) {
    cx.on_action(|editor: &mut Editor, _: &MoveDown, window, cx| {
        editor.move_down(window, cx);
    });
}

// Dispatch action
window.dispatch_action("editor::MoveDown".into(), cx);
```

### 4. Builder Pattern

Extensive use for UI construction:

```rust
div()
    .flex()
    .flex_col()
    .gap_2()
    .p_4()
    .bg(cx.theme().background)
    .child(
        label("Hello").text_color(cx.theme().foreground)
    )
```

### 5. Snapshot Pattern

Immutable snapshots for safe threading:

```rust
// Create snapshot
let snapshot = buffer.read(cx).snapshot();

// Use in background thread
cx.background_spawn(async move {
    let text = snapshot.text();
    expensive_analysis(text)
})
```

### 6. State Machine Pattern

Used extensively in Vim mode:

```rust
enum VimMode {
    Normal,
    Insert,
    Visual { line_mode: bool },
    // ...
}

impl Vim {
    fn transition(&mut self, new_mode: VimMode, cx: &mut Context<Self>) {
        self.mode = new_mode;
        self.update_editor_for_mode(cx);
        cx.notify();
    }
}
```

---

## Performance Considerations

### 1. Rendering Performance

**Frame Budget**: 16.67ms for 60 FPS

**Optimizations**:
- **Retained Mode**: Only re-render changed elements
- **Layout Caching**: Taffy caches layout computations
- **GPU Acceleration**: All rendering via GPU
- **Texture Atlasing**: Glyphs cached in texture atlas
- **Incremental Layout**: Only relayout dirty subtrees

### 2. Text Performance

**Rope Data Structure**:
```
Traditional String: O(n) insert/delete
Rope: O(log n) insert/delete, O(log n) indexing
```

**Optimizations**:
- **Piece Table**: Efficient undo/redo
- **Lazy Syntax Highlighting**: Parse only visible regions
- **Incremental Parsing**: Tree-sitter re-parses only changed regions
- **Anchor Tracking**: O(log n) position updates through edits

### 3. File System Performance

**Optimizations**:
- **Background Watching**: File watching on separate threads
- **Debouncing**: Coalesce rapid file changes
- **Incremental Git Status**: Only update changed files
- **Snapshot Isolation**: Background threads work on snapshots

### 4. LSP Performance

**Optimizations**:
- **Request Batching**: Batch similar requests
- **Cancellation**: Cancel stale requests
- **Throttling**: Limit request rate
- **Caching**: Cache semantic tokens, symbols, etc.

### 5. Memory Performance

**Strategies**:
- **Weak References**: Prevent cycles
- **Snapshot Sharing**: Multiple snapshots share immutable data
- **Lazy Loading**: Load buffers/trees on demand
- **Buffer Pooling**: Reuse allocated buffers

---

## Extension Points

### 1. Language Extensions

Add new languages via WebAssembly:

```rust
// Extension API
impl Extension {
    fn language_config(&mut self, language_id: String) -> LanguageConfig {
        // Return language configuration
    }
}
```

**Capabilities**:
- Grammar (Tree-sitter)
- Syntax highlighting queries
- Bracket pairs
- Auto-indent rules
- LSP server configuration

### 2. Theme Extensions

Custom themes via JSON:

```json
{
  "name": "My Theme",
  "appearance": "dark",
  "style": {
    "background": "#1e1e1e",
    "foreground": "#d4d4d4",
    ...
  }
}
```

### 3. Keymap Extensions

Custom keybindings:

```json
[
  {
    "bindings": {
      "ctrl-shift-p": "command_palette::Toggle"
    }
  }
]
```

### 4. Slash Commands (Assistant)

Add custom slash commands for AI assistant:

```rust
impl SlashCommand for MyCommand {
    fn run(&self, query: &str, cx: &mut App) -> Task<Result<String>> {
        // Execute command
    }
}
```

---

## Conclusion

Zed's architecture is built around:

1. **GPUI**: Custom UI framework for maximum performance
2. **Entity System**: Structured state management
3. **Single Thread UI**: Simplified concurrency model
4. **Async Everywhere**: All I/O is async
5. **Extensibility**: WebAssembly for safe extensions

The architecture prioritizes:
- **Performance**: Native code, GPU acceleration, efficient data structures
- **Collaboration**: Built-in from the ground up
- **Maintainability**: Clear separation of concerns, strong typing
- **Extensibility**: Plugin system for languages, themes, and features

For detailed information on specific subsystems, see the other documentation files in this directory.

---

## Further Reading

- [GPUI Documentation](./01-gpui/README.md)
- [Editor Architecture](./02-editor/README.md)
- [Project System](./03-project/README.md)
- [Workspace Management](./04-workspace/README.md)
- [Language Support](./05-language/README.md)
- [LSP Integration](./06-lsp/README.md)
