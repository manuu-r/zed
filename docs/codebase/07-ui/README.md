# UI Crate

**Path:** `/home/user/zed/crates/ui/`
**Purpose:** Reusable UI components

---

## Overview

Provides common UI components built on GPUI.

## Common Components

```rust
// Button
Button::new("click-me", "Click Me")
    .on_click(|event, window, cx| { ... })

// Label
Label::new("Hello World")
    .text_color(cx.theme().foreground)

// Icon
Icon::new(IconName::Check)
    .color(Color::Success)

// List
List::new(items, |item| {
    ListItem::new(item.id).child(item.name)
})
```

## Documentation Files

- **[Components](./components.md)** - Available components
- **[Styling](./styling.md)** - Styling system
- **README.md** - This file

## Further Reading

- [GPUI](../01-gpui/README.md)
- [Rendering](../01-gpui/rendering.md)
