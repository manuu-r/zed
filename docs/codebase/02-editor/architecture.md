# Editor Architecture

**Last Updated:** 2025-11-16

---

## Overview

The Editor is Zed's central component, implementing a sophisticated text editing engine with IDE features.

## Core Components

### Editor Entity

```rust
pub struct Editor {
    // State
    buffer: Entity<MultiBuffer>,
    display_map: Entity<DisplayMap>,
    selections: SelectionsCollection,

    // UI
    scroll_manager: ScrollManager,
    hover_state: HoverState,
    context_menu: Option<ContextMenu>,

    // Features
    completions_menu: Option<CompletionsMenu>,
    code_actions_menu: Option<CodeActionsMenu>,

    // Collaboration
    remote_id: Option<ViewId>,
    collaboration_hub: Option<CollaborationHub>,

    // Subscriptions
    _subscriptions: Vec<Subscription>,
}
```

## Component Interactions

```
User Input
    │
    ▼
Editor::handle_input()
    │
    ├─→ Update Selections
    │
    ├─→ Modify Buffer
    │     │
    │     └─→ MultiBuffer.edit()
    │           │
    │           └─→ Buffer.edit() (CRDT operation)
    │
    ├─→ Update DisplayMap
    │     │
    │     └─→ Recalculate wrapping/folding
    │
    ├─→ Request LSP features
    │     │
    │     └─→ Completions, diagnostics, etc.
    │
    └─→ cx.notify() to trigger re-render
          │
          └─→ EditorElement.paint()
```

## Display Map Pipeline

The display map transforms buffer coordinates to screen coordinates through several layers:

```
Buffer (actual file content)
    │
    ▼
FoldMap (apply code folding)
    │
    ▼
TabMap (expand tabs to spaces)
    │
    ▼
WrapMap (soft wrap long lines)
    │
    ▼
InlayMap (insert inlay hints)
    │
    ▼
DisplayMap (final screen coordinates)
```

Each map:
- Maintains its own coordinate space
- Converts between input and output coordinates
- Updates incrementally on edits

## Rendering Pipeline

```
Editor.render()
    │
    ▼
Create EditorElement
    │
    ├─→ request_layout()
    │     │
    │     ├─→ Layout visible lines
    │     ├─→ Shape text (cosmic-text)
    │     └─→ Calculate line heights
    │
    ├─→ prepaint()
    │     │
    │     ├─→ Compute scrollbar
    │     ├─→ Setup hitboxes
    │     └─→ Prepare decorations
    │
    └─→ paint()
          │
          ├─→ Paint background
          ├─→ Paint line numbers
          ├─→ Paint text with syntax highlighting
          ├─→ Paint cursors and selections
          ├─→ Paint diagnostics
          └─→ Paint scrollbar
```

## Selection Management

```rust
pub struct SelectionsCollection {
    selections: Vec<Selection>,
    buffer: Entity<MultiBuffer>,
}
```

**Operations:**
- Add/remove selections
- Move cursors
- Transform selections (uppercase, lowercase, etc.)
- Resolve selections after edits

## LSP Integration Points

The editor integrates with LSP at multiple points:

```
Editor
  │
  ├─→ On edit: Request diagnostics
  │
  ├─→ On trigger char: Request completions
  │
  ├─→ On hover: Request hover information
  │
  ├─→ On save: Request formatting
  │
  └─→ On request: Code actions, refactorings
```

## Subscription Architecture

The editor subscribes to several entities:

```rust
impl Editor {
    fn new(...) -> Self {
        let mut subscriptions = Vec::new();

        // Subscribe to buffer changes
        subscriptions.push(cx.subscribe(&buffer, Self::on_buffer_event));

        // Subscribe to project events
        subscriptions.push(cx.subscribe(&project, Self::on_project_event));

        // Subscribe to settings changes
        subscriptions.push(cx.observe_global::<SettingsStore>(Self::on_settings_changed));

        Self {
            buffer,
            _subscriptions: subscriptions,
            // ...
        }
    }
}
```

## Threading Model

- **Foreground**: All editor state, UI updates
- **Background**: Syntax parsing, file I/O, LSP communication
- **Synchronization**: Snapshots for thread-safe access

## Performance Optimizations

1. **Incremental Syntax Highlighting**: Only parse visible regions
2. **Lazy Line Layout**: Only layout visible lines
3. **Cached Glyphs**: Glyph atlas for fast text rendering
4. **Snapshot Isolation**: Background threads work on snapshots
5. **Batch Selection Updates**: Update all selections together

## Further Reading

- [Display Mapping](./display-mapping.md)
- [Selections](./selections.md)
- [LSP Integration](./lsp-integration.md)
