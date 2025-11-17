# Language Crate

**Path:** `/home/user/zed/crates/language/`
**Purpose:** Language support and syntax highlighting

---

## Overview

Provides language definitions, Tree-sitter integration, and syntax highlighting.

## Core Types

```rust
pub struct Language {
    pub name: LanguageName,
    grammar: Option<Arc<Grammar>>,
    config: LanguageConfig,
}

pub struct LanguageRegistry {
    languages: Vec<Arc<Language>>,
}
```

## Documentation Files

- **[Tree-sitter](./tree-sitter.md)** - Parser integration
- **[Highlighting](./highlighting.md)** - Syntax highlighting
- **[Language Config](./language-config.md)** - Configuration files
- **README.md** - This file

## Further Reading

- [Editor](../02-editor/README.md)
- [LSP](../06-lsp/README.md)
