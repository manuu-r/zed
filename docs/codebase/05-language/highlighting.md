# Syntax Highlighting

**Last Updated:** 2025-11-16

---

## Overview

Syntax highlighting uses Tree-sitter queries to identify token types.

## Highlight Queries

```scheme
; highlights.scm
(function_item name: (identifier) @function)
(type_identifier) @type
(string_literal) @string
(comment) @comment
```

## Highlight Map

```rust
pub struct HighlightMap {
    highlights: Vec<(CaptureId, HighlightId)>,
}

// Map capture names to highlight IDs
let highlight_id = highlight_map.get("function");
```

## Rendering

```rust
for chunk in syntax_highlights {
    paint_text(chunk.text, theme.color(chunk.highlight_id));
}
```

## Further Reading

- [Tree-sitter](./tree-sitter.md)
- [Language README](./README.md)
