# Editor Selections

**Last Updated:** 2025-11-16

---

## Overview

Selections represent cursor positions and selected text ranges in the editor.

## Selection Structure

```rust
pub struct Selection<T> {
    pub id: SelectionId,
    pub start: T,        // Start of selection (anchor)
    pub end: T,          // End of selection (head/cursor)
    pub reversed: bool,  // Whether cursor is at start
    pub goal: SelectionGoal,
}
```

### Selection Types

- `Selection<Anchor>`: Stable across edits
- `Selection<Point>`: Buffer coordinates
- `Selection<DisplayPoint>`: Screen coordinates

## Multi-Cursor Support

The editor supports multiple simultaneous cursors:

```rust
pub struct SelectionsCollection {
    selections: Vec<Selection<Anchor>>,
    pending_anchor_selections: Vec<Selection<Anchor>>,
}
```

### Adding Selections

```rust
// Add selection above
editor.add_selection_above(&AddSelectionAbove, cx);

// Add selection below
editor.add_selection_below(&AddSelectionBelow, cx);

// Add selection at mouse
editor.add_selection_at_point(point, cx);
```

### Modifying Selections

```rust
editor.change_selections(None, window, cx, |s| {
    s.move_cursors_with(|map, cursor, goal| {
        // Return new cursor position
        movement::down(map, cursor, goal)
    });
});
```

## Selection Operations

### Moving Cursors

```rust
editor.change_selections(None, window, cx, |s| {
    s.move_cursors_with(|map, cursor, goal| {
        movement::right(map, cursor, goal)
    });
});
```

### Extending Selections

```rust
editor.change_selections(None, window, cx, |s| {
    s.move_heads_with(|map, head, goal| {
        movement::down(map, head, goal)
    });
});
```

### Collapsing Selections

```rust
editor.change_selections(None, window, cx, |s| {
    s.move_with(|map, selection| {
        selection.collapse_to(selection.end, goal)
    });
});
```

## Selection Goal

```rust
pub enum SelectionGoal {
    None,
    Column(u32),          // Vertical movement remembers column
    WrappedHorizontal,    // Horizontal at wrap boundary
    HorizontalRange { start: u32, end: u32 },
}
```

The goal helps maintain cursor position during vertical movement across lines of different lengths.

## Collaborative Selections

In collaborative editing, each participant's selections are tracked:

```rust
pub struct CollaboratorSelections {
    participant_id: ParticipantIndex,
    selections: Vec<Selection<Anchor>>,
    cursor_shape: CursorShape,
}
```

These are rendered as colored cursors and selection ranges.

## Selection Transformations

```rust
// Select all
editor.select_all(&SelectAll, cx);

// Select line
editor.select_line(&SelectLine, cx);

// Select word
editor.select_word(&SelectWord, cx);

// Expand selection
editor.select_larger_syntax_node(&SelectLargerSyntaxNode, cx);
```

## Common Patterns

### Single Cursor Operation

```rust
editor.change_selections(None, window, cx, |s| {
    let selection = s.newest_anchor().clone();
    s.select_anchors(vec![selection]);
});
```

### Multi-Cursor Insert

```rust
editor.insert("text", cx);
// Inserts at all cursor positions
```

### Selection Iteration

```rust
let selections = editor.selections.all::<Point>(cx);
for selection in selections {
    let range = selection.range();
    // Process each selection
}
```

## Further Reading

- [Movements](./movements.md)
- [Actions](./actions.md)
- [Architecture](./architecture.md)
