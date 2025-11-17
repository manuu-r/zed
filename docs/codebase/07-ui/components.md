# UI Components

**Last Updated:** 2025-11-16

---

## Button

```rust
Button::new("my-button", "Click")
    .style(ButtonStyle::Filled)
    .on_click(cx.listener(|this, _event, window, cx| {
        this.handle_click(window, cx);
    }))
```

## Label

```rust
Label::new("Text")
    .size(LabelSize::Large)
    .color(Color::Default)
```

## Icon

```rust
Icon::new(IconName::Check)
    .size(IconSize::Small)
    .color(Color::Success)
```

## List

```rust
List::new(items, |item, cx| {
    ListItem::new(item.id)
        .child(item.name)
        .selected(item.selected)
})
```

## Input

```rust
TextField::new("my-input")
    .placeholder("Enter text...")
    .on_change(cx.listener(|this, value, cx| {
        this.handle_input(value, cx);
    }))
```

## Further Reading

- [UI README](./README.md)
- [Styling](./styling.md)
