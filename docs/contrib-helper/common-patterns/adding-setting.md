# Adding a Setting

This guide shows you how to add a new user-configurable setting to Zed.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Step-by-Step Implementation](#step-by-step-implementation)
4. [Complete Example](#complete-example)
5. [Schema Generation](#schema-generation)
6. [Using Settings in Code](#using-settings-in-code)
7. [Testing](#testing)
8. [Migration](#migration)
9. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How to define setting types
- How to register settings
- How to use settings in code
- How to generate JSON schema
- How to handle setting migrations

**When to use this:**
- Adding user-configurable options
- Making features customizable
- Adding preferences

**Time to complete:** 1-2 hours

## Prerequisites

- Understanding of Rust structs and serde
- Familiarity with JSON Schema
- Knowledge of where your feature is implemented

## Step-by-Step Implementation

Let's add a setting to control auto-save behavior.

### Step 1: Define the Setting Struct

```rust
// File: crates/editor/src/editor_settings.rs

use schemars::JsonSchema;
use serde::{Deserialize, Serialize};
use settings::Setting;

/// Settings for auto-save behavior
#[derive(Clone, Debug, Serialize, Deserialize, JsonSchema, PartialEq)]
#[serde(deny_unknown_fields)]
pub struct AutoSaveSettings {
    /// Enable auto-save
    ///
    /// When enabled, buffers will be automatically saved after a delay.
    ///
    /// Default: false
    #[serde(default)]
    pub enabled: bool,

    /// Delay in milliseconds before auto-saving
    ///
    /// Default: 1000
    #[serde(default = "default_auto_save_delay")]
    pub delay_ms: u64,

    /// Only auto-save when focus is lost
    ///
    /// Default: false
    #[serde(default)]
    pub on_focus_lost_only: bool,
}

fn default_auto_save_delay() -> u64 {
    1000
}

impl Default for AutoSaveSettings {
    fn default() -> Self {
        Self {
            enabled: false,
            delay_ms: 1000,
            on_focus_lost_only: false,
        }
    }
}
```

### Step 2: Register the Setting

```rust
// File: crates/editor/src/editor.rs or settings initialization

use settings::Settings;

pub fn init(cx: &mut App) {
    // Register the setting
    settings::register_setting::<AutoSaveSettings>(cx);
}
```

### Step 3: Access Settings in Code

```rust
// File: wherever you need to use the setting

impl Editor {
    pub fn maybe_auto_save(&mut self, cx: &mut Context<Self>) {
        let settings = AutoSaveSettings::get_global(cx);

        if !settings.enabled {
            return;
        }

        if settings.on_focus_lost_only && self.is_focused(cx) {
            return;
        }

        // Schedule auto-save
        let delay = Duration::from_millis(settings.delay_ms);
        cx.spawn(async move |editor, cx| {
            cx.background_executor().timer(delay).await;
            editor.update(&cx, |editor, cx| {
                editor.save(&Save, cx);
            })
        }).detach();
    }
}
```

### Step 4: Add to Settings File Schema

Users configure settings in `settings.json`. The setting will be available:

```json
{
  "auto_save": {
    "enabled": true,
    "delay_ms": 2000,
    "on_focus_lost_only": false
  }
}
```

### Step 5: Add Default Keymap (if needed)

If your setting affects keybindings:

```json
// File: assets/keymaps/default-*.json
{
  "context": "Editor",
  "bindings": {
    "cmd-s": "editor::Save"
  }
}
```

## Complete Example

Here's a complete example with nested settings:

```rust
// File: crates/terminal_view/src/terminal_settings.rs

use gpui::Pixels;
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};
use settings::{Setting, SettingsSources};

#[derive(Clone, Debug, Serialize, Deserialize, JsonSchema, PartialEq)]
pub struct TerminalSettings {
    /// Font family for terminal
    ///
    /// Default: Uses editor font
    pub font_family: Option<String>,

    /// Font size for terminal
    ///
    /// Default: 14
    #[serde(default = "default_font_size")]
    pub font_size: f32,

    /// Line height multiplier
    ///
    /// Default: 1.2
    #[serde(default = "default_line_height")]
    pub line_height: f32,

    /// Shell configuration
    #[serde(default)]
    pub shell: ShellSettings,

    /// Cursor style
    #[serde(default)]
    pub cursor_style: CursorStyle,

    /// Blinking cursor
    ///
    /// Default: true
    #[serde(default = "default_true")]
    pub blinking_cursor: bool,

    /// Working directory
    pub working_directory: Option<String>,
}

#[derive(Clone, Debug, Serialize, Deserialize, JsonSchema, PartialEq)]
#[serde(rename_all = "snake_case")]
pub enum CursorStyle {
    Block,
    Underline,
    Bar,
}

#[derive(Clone, Debug, Serialize, Deserialize, JsonSchema, PartialEq)]
pub struct ShellSettings {
    /// Program to run
    ///
    /// Default: System default shell
    pub program: Option<String>,

    /// Arguments to pass
    ///
    /// Default: []
    #[serde(default)]
    pub args: Vec<String>,

    /// Environment variables
    ///
    /// Default: {}
    #[serde(default)]
    pub env: std::collections::HashMap<String, String>,
}

// Default implementations
fn default_font_size() -> f32 {
    14.0
}

fn default_line_height() -> f32 {
    1.2
}

fn default_true() -> bool {
    true
}

impl Default for TerminalSettings {
    fn default() -> Self {
        Self {
            font_family: None,
            font_size: 14.0,
            line_height: 1.2,
            shell: ShellSettings::default(),
            cursor_style: CursorStyle::Block,
            blinking_cursor: true,
            working_directory: None,
        }
    }
}

impl Default for CursorStyle {
    fn default() -> Self {
        Self::Block
    }
}

impl Default for ShellSettings {
    fn default() -> Self {
        Self {
            program: None,
            args: Vec::new(),
            env: std::collections::HashMap::new(),
        }
    }
}

// Implement Setting trait
impl Setting for TerminalSettings {
    const KEY: Option<&'static str> = Some("terminal");

    type FileContent = Self;

    fn load(
        sources: SettingsSources<Self::FileContent>,
        _: &mut App,
    ) -> anyhow::Result<Self> {
        sources.json_merge()
    }
}
```

Usage in `settings.json`:

```json
{
  "terminal": {
    "font_family": "JetBrains Mono",
    "font_size": 13,
    "line_height": 1.3,
    "shell": {
      "program": "/bin/zsh",
      "args": ["-l"],
      "env": {
        "TERM": "xterm-256color"
      }
    },
    "cursor_style": "block",
    "blinking_cursor": true,
    "working_directory": "~"
  }
}
```

## Schema Generation

### Automatic Schema Generation

Zed automatically generates JSON schemas from your setting types:

```rust
use schemars::JsonSchema;

#[derive(JsonSchema)]
pub struct MySettings {
    /// This doc comment becomes schema description
    pub my_field: bool,
}
```

### Custom Schema Attributes

```rust
#[derive(JsonSchema)]
pub struct MySettings {
    /// Field with default
    #[serde(default = "default_value")]
    pub field: i32,

    /// Field that must match pattern
    #[schemars(regex(pattern = r"^\w+$"))]
    pub name: String,

    /// Field with specific range
    #[schemars(range(min = 0, max = 100))]
    pub percentage: u8,

    /// Field with examples
    #[schemars(example = "example_value")]
    pub example_field: String,
}
```

### Running Schema Generation

```bash
# Generate schema for all settings
cargo run --bin schema_generator
```

## Using Settings in Code

### Global Settings

```rust
// Get global settings
let settings = MySettings::get_global(cx);

// Use the setting
if settings.enabled {
    // Do something
}
```

### Per-File Settings

Some settings can vary by file:

```rust
// Get settings for specific file
let settings = MySettings::get(file_path, cx);
```

### Observing Setting Changes

```rust
impl MyComponent {
    fn new(cx: &mut Context<Self>) -> Self {
        // React to setting changes
        cx.observe_global::<SettingsStore>(|this, cx| {
            this.settings_changed(cx);
        }).detach();

        Self { /* ... */ }
    }

    fn settings_changed(&mut self, cx: &mut Context<Self>) {
        let settings = MySettings::get_global(cx);
        // Update based on new settings
        cx.notify();
    }
}
```

### Workspace-Specific Settings

```rust
// Settings can vary by workspace
#[derive(JsonSchema)]
pub struct WorkspaceSettings {
    /// Setting that varies by project
    pub project_specific: bool,
}

impl Setting for WorkspaceSettings {
    const KEY: Option<&'static str> = Some("workspace");

    type FileContent = Self;

    fn load(
        sources: SettingsSources<Self::FileContent>,
        _: &mut App,
    ) -> anyhow::Result<Self> {
        // Merge settings from:
        // 1. Default settings
        // 2. User global settings
        // 3. Project-local settings
        sources.json_merge()
    }
}
```

## Testing

### Unit Test

```rust
#[gpui::test]
async fn test_auto_save_settings(cx: &mut TestApp) {
    // Update settings
    cx.update(|cx| {
        settings::update_settings_file::<AutoSaveSettings>(
            cx,
            |settings| {
                settings.enabled = true;
                settings.delay_ms = 500;
            },
        );
    });

    // Verify settings applied
    let settings = AutoSaveSettings::get_global(cx);
    assert!(settings.enabled);
    assert_eq!(settings.delay_ms, 500);
}
```

### Integration Test

```rust
#[gpui::test]
async fn test_setting_affects_behavior(cx: &mut TestApp) {
    let editor = cx.build_entity(|cx| Editor::new(/* ... */));

    // Verify default behavior
    editor.update(cx, |editor, cx| {
        assert!(!editor.is_auto_save_enabled());
    });

    // Enable setting
    cx.update(|cx| {
        settings::update_settings_file::<AutoSaveSettings>(
            cx,
            |settings| settings.enabled = true,
        );
    });

    // Verify behavior changed
    editor.update(cx, |editor, cx| {
        assert!(editor.is_auto_save_enabled());
    });
}
```

### Test Default Values

```rust
#[test]
fn test_default_settings() {
    let settings = AutoSaveSettings::default();
    assert!(!settings.enabled);
    assert_eq!(settings.delay_ms, 1000);
    assert!(!settings.on_focus_lost_only);
}
```

## Migration

### Adding New Fields

When adding fields to existing settings, always provide defaults:

```rust
#[derive(JsonSchema)]
pub struct MySettings {
    // Existing field
    pub old_field: bool,

    // New field with default
    #[serde(default = "default_new_field")]
    pub new_field: String,
}

fn default_new_field() -> String {
    "default value".to_string()
}
```

### Deprecating Fields

```rust
#[derive(JsonSchema)]
pub struct MySettings {
    /// DEPRECATED: Use new_field instead
    #[serde(default)]
    #[schemars(deprecated)]
    pub old_field: Option<bool>,

    /// New recommended field
    #[serde(default)]
    pub new_field: bool,
}
```

### Renaming Fields

```rust
#[derive(JsonSchema)]
pub struct MySettings {
    /// Renamed from old_name
    #[serde(alias = "old_name")]
    pub new_name: bool,
}
```

### Data Migration

```rust
impl Setting for MySettings {
    fn load(
        sources: SettingsSources<Self::FileContent>,
        cx: &mut App,
    ) -> anyhow::Result<Self> {
        let mut settings = sources.json_merge()?;

        // Migrate old format
        if settings.old_field.is_some() && !settings.new_field {
            settings.new_field = settings.old_field.unwrap_or(false);
        }

        Ok(settings)
    }
}
```

## Common Pitfalls

### 1. Not Providing Defaults

❌ **Wrong:**
```rust
#[derive(JsonSchema)]
pub struct MySettings {
    pub field: String,  // No default, will fail if not in config
}
```

✅ **Correct:**
```rust
#[derive(JsonSchema)]
pub struct MySettings {
    #[serde(default = "default_field")]
    pub field: String,
}

fn default_field() -> String {
    "default".to_string()
}
```

### 2. Missing `deny_unknown_fields`

```rust
// Catches typos in user config
#[derive(JsonSchema)]
#[serde(deny_unknown_fields)]
pub struct MySettings {
    // ...
}
```

### 3. Poor Documentation

❌ **Wrong:**
```rust
pub enabled: bool,  // No doc comment
```

✅ **Correct:**
```rust
/// Enable the feature
///
/// When enabled, this will do X, Y, and Z.
///
/// Default: false
pub enabled: bool,
```

### 4. Forgetting to Register

```rust
// Must call in init function
settings::register_setting::<MySettings>(cx);
```

## PR Checklist

- [ ] Setting struct defined with JsonSchema derive
- [ ] All fields have defaults
- [ ] Doc comments on all fields explain purpose
- [ ] Default values documented in comments
- [ ] `deny_unknown_fields` attribute added
- [ ] Setting registered in init
- [ ] Tests added
- [ ] Schema generated successfully
- [ ] Migration handled (if modifying existing setting)
- [ ] Documentation updated

## Resources

- [Settings crate](../../../crates/settings/)
- [Editor settings example](../../../crates/editor/src/editor_settings.rs)
- [Project settings example](../../../crates/project/src/project_settings.rs)
- [JSON Schema docs](https://json-schema.org/)
