# Settings System

**Last Updated:** 2025-11-16

---

## Settings Store

```rust
pub struct SettingsStore {
    settings: HashMap<TypeId, Box<dyn Any>>,
}

impl Global for SettingsStore {}
```

## Defining Settings

```rust
#[derive(Clone, Deserialize)]
pub struct EditorSettings {
    pub tab_size: usize,
    pub soft_wrap: SoftWrap,
    pub show_line_numbers: bool,
}

impl Settings for EditorSettings {
    const KEY: Option<&'static str> = Some("editor");
}
```

## Accessing Settings

```rust
let settings = EditorSettings::get_global(cx);
let tab_size = settings.tab_size;
```

## Observing Changes

```rust
cx.observe_global::<SettingsStore>(|this, cx| {
    this.on_settings_changed(cx);
});
```

## Further Reading

- [Settings README](./README.md)
- [Themes](./themes.md)
