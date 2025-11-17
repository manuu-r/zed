# Language Configuration

**Last Updated:** 2025-11-16

---

## Overview

Language configuration files define language behavior.

## Config Structure

```json
{
  "name": "Rust",
  "grammar": "rust",
  "path_suffixes": ["rs"],
  "brackets": [
    { "start": "{", "end": "}", "close": true },
    { "start": "[", "end": "]", "close": true }
  ],
  "autoclose_before": ";:.,=}])>",
  "line_comments": ["// "],
  "block_comment": ["/*", "*/"]
}
```

## Extension Loading

Languages can be provided by extensions:

```rust
extension.language_config(language_id)?
```

## Further Reading

- [Language README](./README.md)
- [Extensions](../extension-system.md)
