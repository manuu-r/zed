# GPUI Rendering System

**Last Updated:** 2025-11-16

---

## Overview

GPUI uses a retained-mode rendering architecture with GPU acceleration. The rendering pipeline transforms element trees into GPU primitives.

## Element Lifecycle

```
Component.render()
   │
   ▼
Element Tree Created
   │
   ├─→ request_layout() - Calculate sizes
   │
   ├─→ prepaint() - Pre-rendering setup
   │
   └─→ paint() - Draw to GPU
```

### The Element Trait

```rust
pub trait Element {
    type RequestLayoutState;
    type PrepaintState;

    fn request_layout(&mut self, window: &mut Window, cx: &mut App)
        -> (LayoutId, Self::RequestLayoutState);

    fn prepaint(&mut self, bounds: Bounds<Pixels>, state: &mut Self::RequestLayoutState,
        window: &mut Window, cx: &mut App) -> Self::PrepaintState;

    fn paint(&mut self, bounds: Bounds<Pixels>,
        layout_state: &mut Self::RequestLayoutState,
        prepaint_state: &mut Self::PrepaintState,
        window: &mut Window, cx: &mut App);
}
```

## Built-in Elements

### div() - The Foundation

```rust
div()
    .flex()
    .flex_col()
    .gap_4()
    .p_4()
    .bg(gpui::blue())
    .child("Hello")
```

**Layout Properties:**
- `flex()` - Enable flexbox
- `flex_col()` / `flex_row()` - Direction
- `gap_N()` - Gap between children
- `p_N()` - Padding
- `m_N()` - Margin
- `w()` / `h()` - Width/height
- `min_w()` / `max_w()` - Constraints

**Visual Properties:**
- `bg()` - Background color
- `border()` - Border
- `rounded()` - Border radius
- `shadow()` - Drop shadow

### Text Elements

```rust
// Simple text
div().child("Hello")

// Styled text
label("Hello").text_color(red()).text_size(px(16.0))

// With custom styling
div().child(
    StyledText::new("Hello")
        .with_highlights(...)
)
```

### Images

```rust
img(image_data)
    .w(px(100.0))
    .h(px(100.0))

svg(svg_path)
    .text_color(cx.theme().foreground)
```

## Layout Engine (Taffy)

GPUI uses Taffy for flexbox layout:

```rust
div()
    .flex()                    // display: flex
    .flex_direction(FlexDirection::Column)
    .justify_content(JustifyContent::Center)
    .align_items(AlignItems::Start)
    .gap(px(8.0))
```

**Flexbox Properties:**
- `flex()` - Enable flex layout
- `flex_grow()` - Grow factor
- `flex_shrink()` - Shrink factor
- `flex_basis()` - Base size
- `justify_content()` - Main axis alignment
- `align_items()` - Cross axis alignment
- `gap()` - Gap between children

## Painting Pipeline

### Phase 1: Request Layout

```rust
fn request_layout(&mut self, window: &mut Window, cx: &mut App)
    -> (LayoutId, Self::RequestLayoutState)
{
    // Measure text, compute constraints
    let layout_id = window.request_layout(&style, children);
    (layout_id, state)
}
```

### Phase 2: Prepaint

```rust
fn prepaint(&mut self, bounds: Bounds<Pixels>, ...) -> Self::PrepaintState {
    // Setup before painting (hitboxes, etc.)
}
```

### Phase 3: Paint

```rust
fn paint(&mut self, bounds: Bounds<Pixels>, ...) {
    window.paint_quad(bounds, background, border, ...);
    window.paint_glyph(...);
}
```

## Custom Elements

```rust
struct MyElement {
    color: Hsla,
}

impl Element for MyElement {
    type RequestLayoutState = ();
    type PrepaintState = ();

    fn request_layout(&mut self, window: &mut Window, cx: &mut App)
        -> (LayoutId, ())
    {
        let style = Style {
            size: size(px(100.0), px(100.0)),
            ..Default::default()
        };
        (window.request_layout(&style, []), ())
    }

    fn prepaint(&mut self, _: Bounds<Pixels>, _: &mut (), _: &mut Window, _: &mut App) {}

    fn paint(&mut self, bounds: Bounds<Pixels>, _: &mut (), _: &mut (),
             window: &mut Window, _: &mut App)
    {
        window.paint_quad(PaintQuad {
            bounds,
            background: self.color.into(),
            ..Default::default()
        });
    }
}
```

## Performance Optimization

### Caching

```rust
// Cache element rendering
div().cached(StyleRefinement::default())
```

### Conditional Rendering

```rust
div()
    .when(condition, |div| div.child("Shown when true"))
    .when_some(option, |div, value| div.child(value))
```

### Avoid Unnecessary Notifies

```rust
// Only notify when actually changed
if self.value != new_value {
    self.value = new_value;
    cx.notify();
}
```

## Further Reading

- [Core Concepts](./core-concepts.md)
- [Events and Actions](./events-and-actions.md)
- [Architecture Overview](../00-overview.md)
