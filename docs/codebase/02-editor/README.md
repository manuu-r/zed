# Editor Crate

**Path:** `/home/user/zed/crates/editor/`
**Purpose:** Core text editing component

---

## Overview

The editor crate contains Zed's primary text editing functionality. It's a sophisticated component that handles everything from basic text input to advanced IDE features like code completion, diagnostics, and collaborative editing.

### Key Responsibilities

- **Text Editing**: Insert, delete, undo/redo
- **Multi-cursor**: Multiple simultaneous cursors and selections
- **Display Mapping**: Transform buffer coordinates to screen coordinates (wrapping, folding, inlays)
- **LSP Integration**: Completions, hover, diagnostics, code actions
- **Syntax Highlighting**: Tree-sitter-based highlighting
- **Collaborative Editing**: Real-time cursor sharing and conflict-free edits
- **Git Integration**: Diff hunks, blame information
- **Search**: Find and replace within editor

### File Structure

```
editor/
├── src/
│   ├── editor.rs              - Main Editor entity
│   ├── element.rs             - EditorElement (rendering)
│   ├── display_map/           - Coordinate transformation
│   │   ├── mod.rs
│   │   ├── fold_map.rs
│   │   ├── tab_map.rs
│   │   ├── wrap_map.rs
│   │   └── inlay_map.rs
│   ├── movement.rs            - Cursor movement logic
│   ├── actions.rs             - All editor actions
│   ├── selections_collection.rs - Selection management
│   ├── items.rs               - Editor as workspace item
│   └── ...
└── Cargo.toml
```

## Core Types

### Editor

The main entity type:

```rust
pub struct Editor {
    buffer: Entity<MultiBuffer>,
    display_map: Entity<DisplayMap>,
    selections: SelectionsCollection,
    scroll_manager: ScrollManager,
    // ... many more fields
}
```

### DisplayMap

Handles coordinate transformations:

```rust
pub struct DisplayMap {
    buffer: Entity<MultiBuffer>,
    fold_map: FoldMap,
    tab_map: TabMap,
    wrap_map: WrapMap,
    inlay_map: InlayMap,
}
```

## Documentation Files

### 📄 [Architecture](./architecture.md)
- Editor structure and components
- How pieces fit together
- Rendering pipeline
- Event flow

### 📄 [Selections](./selections.md)
- Selection system
- Multi-cursor support
- Selection transformations
- Collaborative selections

### 📄 [Movements](./movements.md)
- Cursor movement primitives
- Vim motions
- Word/line/paragraph navigation
- Selection expansion

### 📄 [Display Mapping](./display-mapping.md)
- Buffer vs Display coordinates
- Folding system
- Soft wrapping
- Tab expansion
- Inlay hints

### 📄 [LSP Integration](./lsp-integration.md)
- How editor uses LSP
- Completions
- Hover information
- Code actions
- Diagnostics display

### 📄 [Actions](./actions.md)
- All editor actions
- Keybindings
- Action dispatch
- Custom actions

## Quick Examples

### Creating an Editor

```rust
let buffer = cx.new(|cx| Buffer::new(0, "Hello, world!", cx));
let multi_buffer = cx.new(|cx| MultiBuffer::singleton(buffer, cx));

let editor = cx.new(|cx| {
    Editor::for_buffer(multi_buffer, None, cx)
});
```

### Editing Text

```rust
editor.update(cx, |editor, cx| {
    editor.insert("text to insert", cx);
    editor.delete(&Delete, cx);
    editor.undo(&Undo, cx);
});
```

### Multi-cursor

```rust
editor.update(cx, |editor, cx| {
    editor.add_selection_below(&AddSelectionBelow, cx);
    editor.insert("same text at all cursors", cx);
});
```

### Getting Text

```rust
let text = editor.read(cx).text(cx);
```

## Key Concepts

### Buffer vs Display Coordinates

- **Buffer**: Actual position in the text file (Point, Offset, Anchor)
- **Display**: Position on screen after wrapping, folding, etc. (DisplayPoint)

### Anchors

Stable positions that track through edits:

```rust
let anchor = editor.read(cx).buffer().read(cx).anchor_at(offset);
// Anchor remains valid after edits
```

### Selections

Represent cursor positions and selected ranges:

```rust
pub struct Selection {
    pub id: SelectionId,
    pub start: Anchor,
    pub end: Anchor,
    pub reversed: bool,
    pub goal: SelectionGoal,
}
```

## Common Workflows

### Implementing a New Action

1. Define action in `actions.rs`:
```rust
#[derive(Clone, PartialEq, Deserialize)]
pub struct MyAction;
impl_actions!(editor, [MyAction]);
```

2. Register handler:
```rust
cx.on_action(|editor: &mut Editor, action: &MyAction, window, cx| {
    editor.handle_my_action(action, window, cx);
});
```

3. Bind key:
```json
{
  "bindings": {
    "ctrl-shift-x": "editor::MyAction"
  }
}
```

### Adding a New Movement

1. Implement in `movement.rs`:
```rust
pub fn my_movement(map: &DisplaySnapshot, ...) -> DisplayPoint {
    // Calculate new position
}
```

2. Use in action handler:
```rust
editor.change_selections(None, window, cx, |s| {
    s.move_cursors_with(|map, cursor, goal| {
        movement::my_movement(map, cursor, goal)
    });
});
```

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                      Editor Entity                       │
│  ┌───────────────────────────────────────────────────┐  │
│  │              SelectionsCollection                  │  │
│  │  (manages all cursors and selections)             │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │            DisplayMap Entity                       │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  MultiBuffer (one or more excerpts)         │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  FoldMap (code folding)                     │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  TabMap (tab expansion)                     │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  WrapMap (soft wrapping)                    │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │  InlayMap (inlay hints)                     │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │             EditorElement                          │  │
│  │  (renders editor to screen)                       │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Performance Characteristics

- **Text Operations**: O(log n) for insert/delete via rope
- **Display Mapping**: O(log n) for coordinate transformations
- **Syntax Highlighting**: Incremental, only visible regions
- **Rendering**: Only visible lines are laid out and painted
- **Selection Updates**: Batch operations for efficiency

## Testing

```rust
#[gpui::test]
fn test_editor_insert(cx: &mut TestAppContext) {
    let buffer = cx.new(|cx| Buffer::new(0, "abc", cx));
    let editor = cx.new(|cx| Editor::for_buffer(buffer, None, cx));

    editor.update(cx, |editor, cx| {
        editor.insert("x", cx);
        assert_eq!(editor.text(cx), "xabc");
    });
}
```

## Further Reading

- [Architecture](./architecture.md) - Detailed architecture
- [Display Mapping](./display-mapping.md) - Coordinate systems
- [LSP Integration](./lsp-integration.md) - Language server features
- [Project Documentation](../03-project/README.md) - Buffer management
