# Tree-sitter Integration

**Last Updated:** 2025-11-16

---

## Overview

Tree-sitter provides incremental parsing for syntax highlighting and code navigation.

## Grammar Loading

```rust
let language = Language::new(LanguageConfig {
    name: "Rust".into(),
    grammar: Some(tree_sitter_rust::language()),
    // ...
});
```

## Parsing

```rust
let tree = parser.parse(&text, None)?;
let root = tree.root_node();

// Query nodes
for node in root.children() {
    println!("{}: {:?}", node.kind(), node.byte_range());
}
```

## Incremental Parsing

Tree-sitter re-parses only changed regions:

```rust
let new_tree = parser.parse(&new_text, Some(&old_tree))?;
// Only re-parses edited regions
```

## Further Reading

- [Highlighting](./highlighting.md)
- [Language README](./README.md)
