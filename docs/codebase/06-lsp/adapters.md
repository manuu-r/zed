# LSP Adapters

**Last Updated:** 2025-11-16

---

## Overview

Language-specific adapters customize LSP behavior.

## Adapter Trait

```rust
pub trait LspAdapter {
    fn name(&self) -> String;
    
    fn server_args(&self) -> Vec<String>;
    
    fn initialization_options(&self) -> Option<serde_json::Value>;
    
    fn workspace_configuration(&self) -> Option<serde_json::Value>;
}
```

## Example: Rust Analyzer

```rust
impl LspAdapter for RustLspAdapter {
    fn name(&self) -> String {
        "rust-analyzer".to_string()
    }

    fn server_args(&self) -> Vec<String> {
        vec![]
    }

    fn initialization_options(&self) -> Option<serde_json::Value> {
        Some(json!({
            "checkOnSave": {
                "command": "clippy"
            }
        }))
    }
}
```

## Further Reading

- [LSP README](./README.md)
- [Language](../05-language/README.md)
