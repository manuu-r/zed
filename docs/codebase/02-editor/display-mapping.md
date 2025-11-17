# Editor Display Mapping

**Last Updated:** 2025-11-16

---

## Overview

Display mapping transforms buffer coordinates (actual file content) to display coordinates (what appears on screen). This handles code folding, soft wrapping, tab expansion, and inlay hints.

## Coordinate Systems

### Buffer Coordinates

Represent positions in the actual file:
- `Point`: (row, column) in buffer
- `Offset`: Byte offset from start
- `Anchor`: Stable position that tracks through edits

### Display Coordinates

Represent positions on screen:
- `DisplayPoint`: (row, column) on screen after all transformations

## The Display Map Pipeline

```
Buffer Text: "fn main() {\n    println!(\"hello\");\n}"
    │
    ▼ FoldMap (fold function body)
Buffer: [0,0]..[2,1]
Folded: [0,0]..[0,13] "fn main() {…}"
    │
    ▼ TabMap (expand tabs to spaces)
Tabs expanded to spaces based on settings
    │
    ▼ WrapMap (soft wrap long lines)
Long lines broken at word boundaries
    │
    ▼ InlayMap (insert type hints, parameter names, etc.)
Display: [0,0]..[0,30] "fn main() -> () {…}"
```

## FoldMap

**File:** `/home/user/zed/crates/editor/src/display_map/fold_map.rs`

Handles code folding:

```rust
pub struct FoldMap {
    buffer: Entity<MultiBuffer>,
    folds: SumTree<Fold>,
}

pub struct Fold {
    range: Range<Anchor>,
    placeholder: FoldPlaceholder,
}
```

**Operations:**
- `fold(ranges)`: Fold code regions
- `unfold(ranges)`: Unfold code regions
- `to_fold_point()`: Convert from buffer to fold coordinates
- `to_buffer_point()`: Convert from fold to buffer coordinates

## TabMap

**File:** `/home/user/zed/crates/editor/src/display_map/tab_map.rs`

Expands tabs to spaces:

```rust
pub struct TabMap {
    fold_snapshot: FoldSnapshot,
    tab_size: NonZeroU32,
}
```

A tab at column 4 with tab_size=4 expands to 4 spaces.
A tab at column 5 with tab_size=4 expands to 3 spaces (to reach column 8).

## WrapMap

**File:** `/home/user/zed/crates/editor/src/display_map/wrap_map.rs`

Handles soft wrapping:

```rust
pub struct WrapMap {
    tab_snapshot: TabSnapshot,
    wrap_width: Option<Pixels>,
    wraps: SumTree<Wrap>,
}
```

**Wrapping Strategy:**
1. Break at word boundaries when possible
2. Break at character boundaries if word too long
3. Respect wrap width from settings

**Example:**
```
Buffer line: "This is a very long line that needs to be wrapped"
Display (width=30):
  "This is a very long line"
  "that needs to be wrapped"
```

## InlayMap

**File:** `/home/user/zed/crates/editor/src/display_map/inlay_map.rs`

Inserts inlay hints from LSP:

```rust
pub struct InlayMap {
    wrap_snapshot: WrapSnapshot,
    inlays: SumTree<Inlay>,
}

pub struct Inlay {
    position: Anchor,
    text: String,
    kind: InlayHintKind,
}
```

**Types of Inlays:**
- Type hints: `let x: i32 = 5;`
- Parameter names: `foo(value: 42)`
- Return types: `fn bar() -> String`

## DisplayMap Entity

Coordinates all the maps:

```rust
pub struct DisplayMap {
    buffer: Entity<MultiBuffer>,
    fold_map: FoldMap,
    tab_map: TabMap,
    wrap_map: WrapMap,
    inlay_map: InlayMap,
}
```

### Coordinate Conversion

```rust
// Buffer → Display
let display_point = display_map.point_to_display_point(buffer_point);

// Display → Buffer
let buffer_point = display_map.display_point_to_point(display_point);

// With anchors (stable across edits)
let display_point = display_map.anchor_to_display_point(anchor);
```

## Snapshot Pattern

Each map provides a snapshot for thread-safe access:

```rust
let snapshot = display_map.read(cx).snapshot(cx);

// Use snapshot in background thread
cx.background_spawn(async move {
    let display_point = snapshot.point_to_display_point(buffer_point);
    // ...
})
```

## Common Operations

### Folding Code

```rust
editor.fold(&Fold, cx);
editor.unfold(&Unfold, cx);
editor.fold_at(&FoldAt { buffer_row: 10 }, cx);
```

### Wrapping Configuration

```rust
// In settings.json
{
  "soft_wrap": "editor_width",  // or "none", "preferred_line_length"
}
```

### Coordinate Translation

```rust
// In editor code
let display_point = editor
    .display_map
    .read(cx)
    .point_to_display_point(Point::new(10, 5));

// Back to buffer
let buffer_point = editor
    .display_map
    .read(cx)
    .display_point_to_point(display_point);
```

## Performance Considerations

1. **Incremental Updates**: Maps only recalculate changed regions
2. **Sum Trees**: O(log n) operations for coordinate conversion
3. **Snapshot Sharing**: Multiple snapshots share immutable data
4. **Lazy Evaluation**: Only calculate for visible regions

## Further Reading

- [Architecture](./architecture.md)
- [Editor README](./README.md)
