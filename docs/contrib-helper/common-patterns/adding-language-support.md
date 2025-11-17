# Adding Language Support

This guide shows you how to add support for a new programming language in Zed through extensions.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Extension Structure](#extension-structure)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Tree-sitter Integration](#tree-sitter-integration)
6. [LSP Adapter](#lsp-adapter)
7. [Testing](#testing)
8. [Publishing](#publishing)
9. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How to create a language extension
- How to integrate tree-sitter grammars
- How to implement LSP adapters
- How to configure syntax highlighting
- How to test language features

**When to use this:**
- Adding support for a new programming language
- Improving existing language support
- Customizing language behavior

**Time to complete:** 2-4 hours

## Prerequisites

- Understanding of the target language
- Basic knowledge of tree-sitter
- Familiarity with LSP (if adding language server support)

## Extension Structure

```
my-language-extension/
├── extension.toml        # Extension manifest
├── Cargo.toml           # Rust dependencies
├── src/
│   └── lib.rs          # Extension implementation
├── grammars/
│   └── my-language/
│       └── config.toml  # Grammar configuration
└── languages/
    └── my-language/
        ├── config.toml  # Language configuration
        ├── highlights.scm  # Syntax highlighting
        ├── brackets.scm    # Bracket matching
        ├── outline.scm     # Symbol outline
        └── injections.scm  # Embedded languages
```

## Step-by-Step Implementation

### Step 1: Create Extension Scaffold

```bash
# In zed/extensions directory
mkdir my-language
cd my-language
```

Create `extension.toml`:
```toml
id = "my-language"
name = "My Language"
description = "Support for My Language"
version = "0.1.0"
schema_version = 1
authors = ["Your Name <you@example.com>"]
repository = "https://github.com/yourusername/zed-my-language"

[grammars.my-language]
repository = "https://github.com/tree-sitter/tree-sitter-mylang"
commit = "abc123..."
```

Create `Cargo.toml`:
```toml
[package]
name = "my-language"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
zed_extension_api = "0.2.0"
```

### Step 2: Implement the Extension

Create `src/lib.rs`:
```rust
use zed_extension_api::{self as zed, LanguageServerId, Result};

struct MyLanguageExtension;

impl zed::Extension for MyLanguageExtension {
    fn new() -> Self {
        Self
    }

    fn language_server_command(
        &mut self,
        language_server_id: &LanguageServerId,
        worktree: &zed::Worktree,
    ) -> Result<zed::Command> {
        // Return command to start language server
        Ok(zed::Command {
            command: "my-language-server".to_string(),
            args: vec!["--stdio".to_string()],
            env: Default::default(),
        })
    }
}

zed::register_extension!(MyLanguageExtension);
```

### Step 3: Configure the Grammar

Create `grammars/my-language/config.toml`:
```toml
repository = "https://github.com/tree-sitter/tree-sitter-mylang"
commit = "abc123..."
```

### Step 4: Configure the Language

Create `languages/my-language/config.toml`:
```toml
name = "My Language"
grammar = "my-language"
path_suffixes = ["mylang", "ml"]
line_comments = ["# "]
block_comment = ["/* ", " */"]
autoclose_before = ";:.,=}])"
brackets = [
    { start = "{", end = "}", close = true, newline = true },
    { start = "[", end = "]", close = true, newline = true },
    { start = "(", end = ")", close = true, newline = true },
]
```

### Step 5: Add Syntax Highlighting

Create `languages/my-language/highlights.scm`:
```scheme
; Keywords
[
  "fn"
  "let"
  "if"
  "else"
  "for"
  "while"
  "return"
] @keyword

; Functions
(function_declaration
  name: (identifier) @function)

(call_expression
  function: (identifier) @function)

; Types
(type_identifier) @type

; Strings
(string_literal) @string

; Numbers
(number_literal) @number

; Comments
(comment) @comment

; Operators
[
  "+"
  "-"
  "*"
  "/"
  "="
  "=="
  "!="
] @operator

; Variables
(identifier) @variable
```

### Step 6: Add Bracket Matching

Create `languages/my-language/brackets.scm`:
```scheme
(function_declaration
  "{" @open
  "}" @close)

(block
  "{" @open
  "}" @close)

(array
  "[" @open
  "]" @close)

(parameter_list
  "(" @open
  ")" @close)
```

### Step 7: Add Outline Support

Create `languages/my-language/outline.scm`:
```scheme
(function_declaration
  name: (identifier) @name) @item

(class_declaration
  name: (identifier) @name) @item

(enum_declaration
  name: (identifier) @name) @item
```

## Tree-sitter Integration

### Finding Tree-sitter Grammars

1. Check https://github.com/tree-sitter
2. Search for "tree-sitter-{language}"
3. Verify it's maintained and complete

### Grammar Configuration

```toml
# In extension.toml
[grammars.my-language]
repository = "https://github.com/tree-sitter/tree-sitter-mylang"
commit = "1a2b3c4d..."  # Pin to specific commit

# Optional: External scanner
[grammars.my-language.build]
scanner = "scanner.c"
```

### Testing Grammar Locally

```bash
# Build the extension
cargo build --release

# Link for local testing
ln -s $PWD ~/.config/zed/extensions/my-language
```

## LSP Adapter

### Full LSP Adapter Implementation

```rust
use zed_extension_api::{self as zed, LanguageServerId, Result};
use std::fs;

struct MyLanguageExtension {
    cached_binary_path: Option<String>,
}

impl zed::Extension for MyLanguageExtension {
    fn new() -> Self {
        Self {
            cached_binary_path: None,
        }
    }

    fn language_server_command(
        &mut self,
        language_server_id: &LanguageServerId,
        worktree: &zed::Worktree,
    ) -> Result<zed::Command> {
        let binary_path = self.language_server_binary_path(language_server_id, worktree)?;

        Ok(zed::Command {
            command: binary_path,
            args: vec!["--stdio".to_string()],
            env: Default::default(),
        })
    }

    fn language_server_initialization_options(
        &mut self,
        _language_server_id: &LanguageServerId,
        _worktree: &zed::Worktree,
    ) -> Result<Option<String>> {
        Ok(Some(serde_json::json!({
            "formatting": {
                "indentSize": 2,
                "insertSpaces": true
            }
        }).to_string()))
    }

    fn language_server_workspace_configuration(
        &mut self,
        _language_server_id: &LanguageServerId,
        _worktree: &zed::Worktree,
    ) -> Result<Option<String>> {
        Ok(Some(serde_json::json!({
            "mylanguage": {
                "checkOnSave": true,
                "linting": {
                    "enabled": true
                }
            }
        }).to_string()))
    }
}

impl MyLanguageExtension {
    fn language_server_binary_path(
        &mut self,
        language_server_id: &LanguageServerId,
        worktree: &zed::Worktree,
    ) -> Result<String> {
        if let Some(path) = &self.cached_binary_path {
            if fs::metadata(path).is_ok() {
                return Ok(path.clone());
            }
        }

        let binary_name = "my-language-server";
        let version = "1.0.0";

        let platform = match std::env::consts::OS {
            "macos" => "darwin",
            "linux" => "linux",
            "windows" => "windows",
            other => return Err(format!("Unsupported platform: {}", other).into()),
        };

        let arch = match std::env::consts::ARCH {
            "x86_64" => "x64",
            "aarch64" => "arm64",
            other => return Err(format!("Unsupported architecture: {}", other).into()),
        };

        let binary_path = format!(
            "{binary_name}-{platform}-{arch}",
            binary_name = binary_name,
            platform = platform,
            arch = arch
        );

        let download_url = format!(
            "https://github.com/my-language/my-language-server/releases/download/v{version}/{binary_path}",
            version = version,
            binary_path = binary_path
        );

        let binary_path = worktree.download_file(&download_url, &binary_name)?;
        self.cached_binary_path = Some(binary_path.clone());

        Ok(binary_path)
    }
}

zed::register_extension!(MyLanguageExtension);
```

### Configuring Initialization Options

```rust
fn language_server_initialization_options(
    &mut self,
    _language_server_id: &LanguageServerId,
    _worktree: &zed::Worktree,
) -> Result<Option<String>> {
    Ok(Some(serde_json::json!({
        "trace": {
            "server": "verbose"
        },
        "formatting": {
            "provider": "prettier"
        }
    }).to_string()))
}
```

## Testing

### Manual Testing

1. Build and link extension:
```bash
cargo build --release
ln -s $PWD ~/.config/zed/extensions/my-language
```

2. Restart Zed

3. Open a file with your language extension (`.mylang`)

4. Verify:
   - [ ] Syntax highlighting works
   - [ ] Bracket matching works
   - [ ] Outline panel shows symbols
   - [ ] LSP features work (if applicable)

### Test Syntax Highlighting

Create test file `test.mylang`:
```mylang
// Test various syntax elements
fn hello(name: string): string {
    let greeting = "Hello, " + name;
    return greeting;
}

class MyClass {
    private value: number = 42;

    method() {
        if (this.value > 0) {
            return true;
        }
    }
}
```

Verify highlighting for:
- Keywords (fn, let, class, if, return)
- Strings
- Numbers
- Types
- Functions
- Comments
- Operators

### Test LSP Integration

Test these LSP features:
- [ ] Autocomplete
- [ ] Go to definition
- [ ] Find references
- [ ] Hover documentation
- [ ] Diagnostics
- [ ] Formatting

## Publishing

### Prepare for Publishing

1. Update `extension.toml`:
```toml
id = "my-language"
name = "My Language"
description = "Full support for My Language with LSP"
version = "1.0.0"
schema_version = 1
authors = ["Your Name <you@example.com>"]
repository = "https://github.com/yourusername/zed-my-language"
```

2. Add README.md:
```markdown
# My Language Extension for Zed

Provides syntax highlighting and LSP support for My Language.

## Features

- Syntax highlighting
- Code completion
- Go to definition
- Diagnostics
- Formatting

## Configuration

Configure in your Zed settings:

```json
{
  "lsp": {
    "my-language-server": {
      "settings": {
        "formatting": {
          "indentSize": 2
        }
      }
    }
  }
}
```

3. Create `.gitignore`:
```
target/
Cargo.lock
*.log
```

### Submit to Extension Registry

1. Create GitHub repository
2. Push your extension
3. Submit PR to zed-industries/extensions
4. Wait for review

## Common Pitfalls

### 1. Grammar Commit Hash

❌ **Wrong:** Using `main` or branch names
✅ **Correct:** Pin to specific commit hash

### 2. Missing Tree-sitter Queries

Ensure you have all required query files:
- highlights.scm (required)
- brackets.scm (recommended)
- outline.scm (recommended)
- injections.scm (if needed)

### 3. LSP Binary Path Issues

Test on all platforms:
- macOS (Intel and Apple Silicon)
- Linux (x64 and arm64)
- Windows (x64)

## PR Checklist

- [ ] Extension builds successfully
- [ ] Grammar is pinned to specific commit
- [ ] All query files present and tested
- [ ] LSP adapter works on all platforms
- [ ] README.md with clear documentation
- [ ] Examples and test files included
- [ ] License file included
- [ ] Version number follows semver

## Resources

- [Extension API Docs](https://github.com/zed-industries/zed/tree/main/crates/extension_api)
- [Tree-sitter Documentation](https://tree-sitter.github.io/tree-sitter/)
- [LSP Specification](https://microsoft.github.io/language-server-protocol/)
- [Existing Extensions](https://github.com/zed-industries/extensions)
