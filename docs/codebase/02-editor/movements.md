# Editor Movements

**Last Updated:** 2025-11-16

---

## Overview

Movements define how cursors navigate through text. They're the foundation of text editing and Vim motions.

## Core Movement Functions

Located in `/home/user/zed/crates/editor/src/movement.rs`:

### Basic Movements

```rust
// Character movements
pub fn left(map: &DisplaySnapshot, point: DisplayPoint, ...) -> DisplayPoint
pub fn right(map: &DisplaySnapshot, point: DisplayPoint, ...) -> DisplayPoint

// Line movements
pub fn up(map: &DisplaySnapshot, point: DisplayPoint, ...) -> DisplayPoint
pub fn down(map: &DisplaySnapshot, point: DisplayPoint, ...) -> DisplayPoint

// Line boundaries
pub fn line_beginning(map: &DisplaySnapshot, point: DisplayPoint, ...) -> DisplayPoint
pub fn line_end(map: &DisplaySnapshot, point: DisplayPoint, ...) -> DisplayPoint
```

### Word Movements

```rust
// Next word start
pub fn next_word_start(map: &DisplaySnapshot, point: DisplayPoint) -> DisplayPoint

// Next word end
pub fn next_word_end(map: &DisplaySnapshot, point: DisplayPoint) -> DisplayPoint

// Previous word start
pub fn previous_word_start(map: &DisplaySnapshot, point: DisplayPoint) -> DisplayPoint
```

### Document Boundaries

```rust
// Start of document
pub fn start_of_document(map: &DisplaySnapshot, ...) -> DisplayPoint

// End of document
pub fn end_of_document(map: &DisplaySnapshot, ...) -> DisplayPoint
```

## Using Movements

### In Actions

```rust
impl Editor {
    fn move_down(&mut self, window: &mut Window, cx: &mut Context<Self>) {
        self.change_selections(None, window, cx, |s| {
            let line_mode = s.line_mode;
            s.move_cursors_with(|map, cursor, goal| {
                movement::down(map, cursor, goal, line_mode)
            });
        });
    }
}
```

### With Selection

```rust
// Extend selection down
editor.change_selections(None, window, cx, |s| {
    s.move_heads_with(|map, head, goal| {
        movement::down(map, head, goal, false)
    });
});
```

## Movement Modes

### Character Mode

Moves by individual characters:
```rust
movement::left(map, point, SelectionGoal::None)
movement::right(map, point, SelectionGoal::None)
```

### Word Mode

Moves by word boundaries:
```rust
movement::next_word_start(map, point)
movement::previous_word_start(map, point)
```

### Line Mode

Special mode for line-wise operations (used in Vim):
```rust
movement::down(map, point, goal, true) // line_mode = true
```

## Vim Motions

Vim mode extends movements with additional primitives:

### Motion Types

```rust
pub enum Motion {
    Left,
    Down,
    Up,
    Right,
    NextWordStart,
    NextWordEnd,
    PreviousWordStart,
    FirstNonWhitespace,
    StartOfLine,
    EndOfLine,
    Matching,           // % - matching bracket
    FindForward { char, before },
    FindBackward { char, after },
    // ... many more
}
```

### Object Motions

```rust
// Word object
pub fn word_motion(map: &DisplaySnapshot, point: DisplayPoint, ...) -> Range<DisplayPoint>

// Paragraph object
pub fn paragraph_motion(map: &DisplaySnapshot, point: DisplayPoint, ...) -> Range<DisplayPoint>

// Bracket object
pub fn surrounding_brackets(map: &DisplaySnapshot, point: DisplayPoint, ...) -> Range<DisplayPoint>
```

## Movement Helpers

### Word Detection

```rust
pub fn is_word_char(c: char) -> bool {
    c.is_alphanumeric() || c == '_'
}

pub fn find_boundary(text: &str, mut offset: usize, is_boundary: impl Fn(char) -> bool) -> usize {
    // Find next boundary
}
```

### Bracket Matching

```rust
pub fn find_matching_bracket(map: &DisplaySnapshot, point: DisplayPoint) -> Option<DisplayPoint>
```

### Indent-Based Movement

```rust
pub fn move_to_matching_indent(map: &DisplaySnapshot, point: DisplayPoint) -> DisplayPoint
```

## Selection Goal

Movements preserve the selection goal to maintain column position:

```rust
pub enum SelectionGoal {
    None,
    Column(u32),              // Remember column for vertical movement
    WrappedHorizontal,        // At wrap boundary
    HorizontalRange { start: u32, end: u32 },
}

// Example: Moving down preserves column
let goal = SelectionGoal::Column(cursor.column());
let new_cursor = movement::down(map, cursor, goal, false);
```

## Common Patterns

### Move and Select

```rust
// Move cursor
editor.change_selections(None, window, cx, |s| {
    s.move_cursors_with(|map, cursor, goal| {
        movement::next_word_start(map, cursor)
    });
});

// Extend selection
editor.change_selections(None, window, cx, |s| {
    s.move_heads_with(|map, head, goal| {
        movement::next_word_end(map, head)
    });
});
```

### Multi-Cursor Movement

```rust
// All cursors move identically
editor.change_selections(None, window, cx, |s| {
    s.move_cursors_with(|map, cursor, goal| {
        movement::down(map, cursor, goal, false)
    });
});
```

### Conditional Movement

```rust
pub fn move_if_condition(
    map: &DisplaySnapshot,
    point: DisplayPoint,
    condition: impl Fn(char) -> bool
) -> DisplayPoint {
    // Move only if condition met
}
```

## Further Reading

- [Selections](./selections.md)
- [Actions](./actions.md)
- [Vim Documentation](../08-vim/motions.md)
