# Project Search

**Last Updated:** 2025-11-16

---

## Overview

Project-wide search functionality.

## Search API

```rust
let results = project.search(SearchQuery {
    query: "function".to_string(),
    case_sensitive: false,
    whole_word: false,
    regex: false,
}, cx).await?;
```

## Implementation

Uses ripgrep for fast searching:

```rust
// Spawn ripgrep process
let output = Command::new("rg")
    .args(&["--json", query])
    .output()
    .await?;

// Parse results
for line in output.lines() {
    let match: RgMatch = serde_json::from_str(line)?;
    // ...
}
```

## Search and Replace

```rust
project.replace_all(query, replacement, cx).await?;
```

## Further Reading

- [Project README](./README.md)
- [Features Search](../11-features/search.md)
