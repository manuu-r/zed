# UI Styling

**Last Updated:** 2025-11-16

---

## Theme Integration

```rust
div()
    .bg(cx.theme().background)
    .text_color(cx.theme().foreground)
    .border_color(cx.theme().border)
```

## Colors

```rust
pub struct ThemeColors {
    pub background: Hsla,
    pub foreground: Hsla,
    pub border: Hsla,
    pub success: Hsla,
    pub warning: Hsla,
    pub error: Hsla,
}
```

## Component Variants

```rust
Button::new("btn", "Click")
    .style(ButtonStyle::Filled)   // or Outlined, Ghost
    .size(ButtonSize::Large)       // or Medium, Small
```

## Further Reading

- [UI README](./README.md)
- [Theme System](../10-settings-theme/themes.md)
