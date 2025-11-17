# Creating UI Components

This guide shows you how to create reusable UI components in Zed using GPUI, the GPU-accelerated UI framework.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Component Types](#component-types)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Complete Examples](#complete-examples)
6. [Styling and Theming](#styling-and-theming)
7. [Handling Interactions](#handling-interactions)
8. [Testing UI Components](#testing-ui-components)
9. [Common Pitfalls](#common-pitfalls)
10. [Advanced Patterns](#advanced-patterns)
11. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How to implement the `Render` and `RenderOnce` traits
- How to style components with Tailwind-like APIs
- How to handle user interactions
- How to integrate with Zed's theme system
- How to make components reactive

**When to use this:**
- Building reusable UI widgets
- Creating custom controls
- Implementing new UI features
- Designing complex layouts

**Time to complete:** 1-2 hours

## Prerequisites

Before starting, you should understand:
- GPUI's `Entity<T>`, `Context<T>`, and `Window` types
- The `Render` trait and element system
- Basic flexbox layout concepts

**Recommended reading:**
- [Getting Started Guide](../getting-started.md)
- [GPUI README](../../../crates/gpui/README.md)
- [CLAUDE.md](../../../CLAUDE.md)

## Component Types

### 1. Stateful Components (Entity-based)

Use `Entity<T>` for components with mutable state:

```rust
struct Counter {
    count: usize,
}

impl Render for Counter {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .child(format!("Count: {}", self.count))
            .on_click(cx.listener(|this, _event, _window, cx| {
                this.count += 1;
                cx.notify();
            }))
    }
}
```

### 2. Stateless Components (Value-based)

Use `RenderOnce` for one-time rendering without state:

```rust
#[derive(IntoElement)]
struct Label {
    text: SharedString,
}

impl RenderOnce for Label {
    fn render(self, _: &mut Window, _: &mut App) -> impl IntoElement {
        div().child(self.text)
    }
}
```

### 3. Wrapper Components

Components that enhance or modify child elements:

```rust
struct Card {
    child: AnyElement,
}

impl RenderOnce for Card {
    fn render(self, window: &mut Window, cx: &mut App) -> impl IntoElement {
        div()
            .elevation_2(cx)
            .rounded_md()
            .p_4()
            .child(self.child)
    }
}
```

## Step-by-Step Implementation

Let's build a `ProgressBar` component with configurable colors and animations.

### Step 1: Define the Component Structure

```rust
// File: crates/ui/src/components/progress_bar.rs

use gpui::*;
use ui::prelude::*;

pub struct ProgressBar {
    progress: f32,  // 0.0 to 1.0
    label: Option<SharedString>,
    color: Color,
}
```

### Step 2: Add Constructor and Builder Methods

```rust
impl ProgressBar {
    pub fn new(progress: f32) -> Self {
        Self {
            progress: progress.clamp(0.0, 1.0),
            label: None,
            color: Color::Accent,
        }
    }

    pub fn label(mut self, label: impl Into<SharedString>) -> Self {
        self.label = Some(label.into());
        self
    }

    pub fn color(mut self, color: Color) -> Self {
        self.color = color;
        self
    }
}
```

### Step 3: Implement `RenderOnce`

```rust
impl RenderOnce for ProgressBar {
    fn render(self, window: &mut Window, cx: &mut App) -> impl IntoElement {
        let progress_percent = (self.progress * 100.0) as u32;

        v_flex()
            .gap_2()
            .w_full()
            .when_some(self.label.clone(), |this, label| {
                this.child(
                    div().child(
                        Label::new(label)
                            .size(LabelSize::Small)
                            .color(Color::Muted)
                    )
                )
            })
            .child(
                div()
                    .h_2()
                    .w_full()
                    .rounded_full()
                    .bg(cx.theme().colors().element_background)
                    .child(
                        div()
                            .h_full()
                            .rounded_full()
                            .bg(self.color.color(cx))
                            .w(relative(self.progress))
                            .transition_all(cx)
                    )
            )
            .when(self.label.is_some(), |this| {
                this.child(
                    div()
                        .text_ui_sm(cx)
                        .text_color(cx.theme().colors().text_muted)
                        .child(format!("{}%", progress_percent))
                )
            })
    }
}
```

### Step 4: Add the `IntoElement` Derive

```rust
#[derive(IntoElement)]
pub struct ProgressBar {
    progress: f32,
    label: Option<SharedString>,
    color: Color,
}
```

### Step 5: Export from Module

```rust
// File: crates/ui/src/components.rs

mod progress_bar;
pub use progress_bar::*;
```

### Step 6: Use the Component

```rust
// In your application code
fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
    v_flex()
        .gap_4()
        .child(
            ProgressBar::new(0.75)
                .label("Loading...")
                .color(Color::Accent)
        )
        .child(
            ProgressBar::new(self.download_progress)
                .label("Downloading")
                .color(Color::Success)
        )
}
```

## Complete Examples

### Example 1: Button Component

```rust
use gpui::*;
use ui::prelude::*;

#[derive(IntoElement)]
pub struct Button {
    label: SharedString,
    on_click: Option<Box<dyn Fn(&ClickEvent, &mut Window, &mut App) + 'static>>,
    style: ButtonStyle,
    disabled: bool,
}

#[derive(Default, Clone, Copy)]
pub enum ButtonStyle {
    #[default]
    Primary,
    Secondary,
    Danger,
}

impl Button {
    pub fn new(label: impl Into<SharedString>) -> Self {
        Self {
            label: label.into(),
            on_click: None,
            style: ButtonStyle::Primary,
            disabled: false,
        }
    }

    pub fn on_click(
        mut self,
        handler: impl Fn(&ClickEvent, &mut Window, &mut App) + 'static,
    ) -> Self {
        self.on_click = Some(Box::new(handler));
        self
    }

    pub fn style(mut self, style: ButtonStyle) -> Self {
        self.style = style;
        self
    }

    pub fn disabled(mut self, disabled: bool) -> Self {
        self.disabled = disabled;
        self
    }
}

impl RenderOnce for Button {
    fn render(self, _window: &mut Window, cx: &mut App) -> impl IntoElement {
        let colors = cx.theme().colors();

        let (bg, hover_bg, text_color) = match self.style {
            ButtonStyle::Primary => (
                colors.element_background,
                colors.element_hover,
                colors.text,
            ),
            ButtonStyle::Secondary => (
                colors.ghost_element_background,
                colors.ghost_element_hover,
                colors.text_muted,
            ),
            ButtonStyle::Danger => (
                colors.error_background,
                colors.error,
                colors.error_foreground,
            ),
        };

        div()
            .px_4()
            .py_2()
            .rounded_md()
            .bg(bg)
            .text_color(text_color)
            .when(!self.disabled, |this| {
                this.hover(|style| style.bg(hover_bg))
                    .active(|style| style.opacity(0.8))
                    .cursor_pointer()
            })
            .when(self.disabled, |this| {
                this.opacity(0.5).cursor_not_allowed()
            })
            .child(self.label)
            .when_some(self.on_click, |this, handler| {
                this.on_click(move |event, window, cx| {
                    if !self.disabled {
                        handler(event, window, cx);
                    }
                })
            })
    }
}
```

Usage:
```rust
Button::new("Save")
    .style(ButtonStyle::Primary)
    .on_click(|_, window, cx| {
        // Handle click
    })

Button::new("Delete")
    .style(ButtonStyle::Danger)
    .disabled(true)
    .on_click(|_, window, cx| {
        // Handle delete
    })
```

### Example 2: Stateful Component (Collapsible Panel)

```rust
use gpui::*;
use ui::prelude::*;

pub struct CollapsiblePanel {
    title: SharedString,
    expanded: bool,
    content: AnyElement,
}

impl CollapsiblePanel {
    pub fn new(
        title: impl Into<SharedString>,
        content: impl IntoElement,
        cx: &mut App,
    ) -> Entity<Self> {
        cx.build_entity(|_cx| Self {
            title: title.into(),
            expanded: true,
            content: content.into_any_element(),
        })
    }

    fn toggle_expanded(&mut self, cx: &mut Context<Self>) {
        self.expanded = !self.expanded;
        cx.notify();
    }
}

impl Render for CollapsiblePanel {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        v_flex()
            .w_full()
            .rounded_md()
            .border_1()
            .border_color(cx.theme().colors().border)
            .child(
                // Header
                h_flex()
                    .justify_between()
                    .p_3()
                    .cursor_pointer()
                    .hover(|style| {
                        style.bg(cx.theme().colors().element_hover)
                    })
                    .on_click(cx.listener(|this, _event, _window, cx| {
                        this.toggle_expanded(cx);
                    }))
                    .child(
                        Label::new(self.title.clone())
                            .size(LabelSize::Default)
                    )
                    .child(
                        Icon::new(if self.expanded {
                            IconName::ChevronDown
                        } else {
                            IconName::ChevronRight
                        })
                    )
            )
            .when(self.expanded, |this| {
                this.child(
                    div()
                        .p_3()
                        .border_t_1()
                        .border_color(cx.theme().colors().border)
                        .child(self.content.clone())
                )
            })
    }
}
```

Usage:
```rust
CollapsiblePanel::new(
    "Settings",
    v_flex()
        .gap_2()
        .child("Setting 1")
        .child("Setting 2"),
    cx
)
```

## Styling and Theming

### Tailwind-like Style Methods

GPUI provides Tailwind-inspired styling methods:

```rust
div()
    // Layout
    .flex()
    .flex_col()
    .items_center()
    .justify_between()
    .gap_4()

    // Sizing
    .w_full()
    .h(px(200.0))
    .min_w_64()
    .max_w_96()

    // Spacing
    .p_4()           // padding all sides
    .px_6()          // padding horizontal
    .py_2()          // padding vertical
    .m_2()           // margin
    .gap_3()         // gap between children

    // Colors
    .bg(gpui::red())
    .text_color(gpui::white())
    .border_color(gpui::blue())

    // Borders
    .border_1()
    .border_t_2()
    .rounded_md()
    .rounded_full()

    // Effects
    .shadow_lg()
    .opacity(0.8)

    // Interactions
    .hover(|style| style.bg(gpui::gray()))
    .active(|style| style.scale(0.95))
```

### Using Theme Colors

Always use theme colors for consistency:

```rust
impl RenderOnce for MyComponent {
    fn render(self, _: &mut Window, cx: &mut App) -> impl IntoElement {
        let colors = cx.theme().colors();

        div()
            .bg(colors.background)
            .text_color(colors.text)
            .border_color(colors.border)
            .child(
                div()
                    .bg(colors.element_background)
                    .hover(|style| style.bg(colors.element_hover))
            )
    }
}
```

### Common Theme Colors

```rust
let colors = cx.theme().colors();

// Backgrounds
colors.background           // Main background
colors.element_background   // UI element background
colors.panel_background     // Panel background

// Text
colors.text                 // Primary text
colors.text_muted           // Secondary text
colors.text_placeholder     // Placeholder text

// Interactive
colors.element_hover        // Hover state
colors.element_active       // Active state
colors.element_selected     // Selected state

// Semantic
colors.error                // Error color
colors.warning              // Warning color
colors.success              // Success color
colors.info                 // Info color

// Borders
colors.border               // Standard border
colors.border_variant       // Subtle border
```

### Responsive Sizing

```rust
// Use px for explicit pixel values
div().w(px(200.0)).h(px(100.0))

// Use relative for percentages
div().w(relative(0.5))  // 50% width

// Use rems for scalable sizing
div().w(rems(20.0))

// Predefined sizes
div()
    .w_full()      // 100%
    .w_1_2()       // 50%
    .w_64()        // 16rem
```

## Handling Interactions

### Click Events

```rust
div()
    .on_click(|event: &ClickEvent, window: &mut Window, cx: &mut App| {
        // Handle click
    })
```

### With Entity Context

```rust
impl Render for MyComponent {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .on_click(cx.listener(|this, event, window, cx| {
                // `this` is &mut Self
                this.handle_click(cx);
            }))
    }
}
```

### Mouse Events

```rust
div()
    .on_mouse_down(|event, window, cx| { /* ... */ })
    .on_mouse_up(|event, window, cx| { /* ... */ })
    .on_mouse_move(|event, window, cx| { /* ... */ })
    .on_hover(|hovering, window, cx| { /* ... */ })
```

### Keyboard Events

```rust
div()
    .on_key_down(|event: &KeyDownEvent, window, cx| {
        if event.keystroke.key == "Enter" {
            // Handle enter
        }
    })
    .on_key_up(|event, window, cx| { /* ... */ })
```

### Focus Management

```rust
struct MyComponent {
    focus_handle: FocusHandle,
}

impl MyComponent {
    fn new(cx: &mut Context<Self>) -> Self {
        Self {
            focus_handle: cx.focus_handle(),
        }
    }
}

impl Render for MyComponent {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .track_focus(&self.focus_handle)
            .on_action(cx.listener(Self::handle_action))
            .when(self.focus_handle.is_focused(cx), |this| {
                this.border_2().border_color(cx.theme().colors().border_focused)
            })
    }
}
```

## Testing UI Components

### Visual Test

```rust
#[gpui::test]
async fn test_progress_bar_rendering(cx: &mut TestApp) {
    let progress_bar = cx.build_entity(|cx| {
        ProgressBar::new(0.5)
    });

    progress_bar.update(cx, |bar, cx| {
        let element = bar.render(&mut Window::default(), cx);
        // Verify structure
        assert!(element.children().len() > 0);
    });
}
```

### Interaction Test

```rust
#[gpui::test]
async fn test_button_click(cx: &mut TestApp) {
    let clicked = Rc::new(Cell::new(false));
    let clicked_clone = clicked.clone();

    let button = cx.build_entity(|cx| {
        Button::new("Click me")
            .on_click(move |_, _, _| {
                clicked_clone.set(true);
            })
    });

    // Simulate click
    button.update(cx, |button, cx| {
        // Trigger click event
        cx.dispatch_action(Click.boxed_clone());
    });

    assert!(clicked.get());
}
```

### State Update Test

```rust
#[gpui::test]
async fn test_collapsible_panel_toggle(cx: &mut TestApp) {
    let panel = CollapsiblePanel::new("Test", div(), cx);

    // Initially expanded
    assert!(panel.read(cx).expanded);

    // Toggle
    panel.update(cx, |panel, cx| {
        panel.toggle_expanded(cx);
    });

    // Now collapsed
    assert!(!panel.read(cx).expanded);
}
```

## Common Pitfalls

### 1. Forgetting `IntoElement` Derive

❌ **Wrong:**
```rust
pub struct MyComponent {
    text: String,
}

// Missing #[derive(IntoElement)]
impl RenderOnce for MyComponent { /* ... */ }
```

✅ **Correct:**
```rust
#[derive(IntoElement)]
pub struct MyComponent {
    text: String,
}

impl RenderOnce for MyComponent { /* ... */ }
```

### 2. Not Using Theme Colors

❌ **Wrong:**
```rust
div().bg(gpui::rgb(0x000000))  // Hardcoded color
```

✅ **Correct:**
```rust
div().bg(cx.theme().colors().background)
```

### 3. Cloning Elements Incorrectly

❌ **Wrong:**
```rust
struct MyComponent {
    child: impl IntoElement,  // Cannot be stored directly
}
```

✅ **Correct:**
```rust
struct MyComponent {
    child: AnyElement,  // Use AnyElement for storage
}

// Convert to AnyElement when storing:
child: element.into_any_element()
```

### 4. Forgetting to Call `cx.notify()`

❌ **Wrong:**
```rust
fn update_value(&mut self, value: String, cx: &mut Context<Self>) {
    self.value = value;
    // Missing cx.notify() - UI won't update!
}
```

✅ **Correct:**
```rust
fn update_value(&mut self, value: String, cx: &mut Context<Self>) {
    self.value = value;
    cx.notify();
}
```

### 5. Incorrect Builder Pattern

❌ **Wrong:**
```rust
impl MyComponent {
    pub fn with_label(&mut self, label: String) {
        self.label = label;  // Mutates, doesn't return Self
    }
}
```

✅ **Correct:**
```rust
impl MyComponent {
    pub fn with_label(mut self, label: String) -> Self {
        self.label = label;
        self  // Return Self for chaining
    }
}
```

## Advanced Patterns

### 1. Component with Children

```rust
#[derive(IntoElement)]
struct Container {
    children: SmallVec<[AnyElement; 2]>,
}

impl Container {
    pub fn new() -> Self {
        Self {
            children: SmallVec::new(),
        }
    }

    pub fn child(mut self, child: impl IntoElement) -> Self {
        self.children.push(child.into_any_element());
        self
    }

    pub fn children(
        mut self,
        children: impl IntoIterator<Item = impl IntoElement>,
    ) -> Self {
        self.children.extend(
            children.into_iter().map(|c| c.into_any_element())
        );
        self
    }
}

impl RenderOnce for Container {
    fn render(self, _: &mut Window, cx: &mut App) -> impl IntoElement {
        div()
            .flex()
            .flex_col()
            .gap_2()
            .children(self.children)
    }
}
```

### 2. Generic Components

```rust
#[derive(IntoElement)]
struct List<T: Clone> {
    items: Vec<T>,
    render_item: Arc<dyn Fn(T, &mut App) -> AnyElement>,
}

impl<T: Clone> List<T> {
    pub fn new(
        items: Vec<T>,
        render_item: impl Fn(T, &mut App) -> AnyElement + 'static,
    ) -> Self {
        Self {
            items,
            render_item: Arc::new(render_item),
        }
    }
}

impl<T: Clone> RenderOnce for List<T> {
    fn render(self, _: &mut Window, cx: &mut App) -> impl IntoElement {
        v_flex()
            .gap_1()
            .children(
                self.items.into_iter().map(|item| {
                    (self.render_item)(item, cx)
                })
            )
    }
}
```

### 3. Conditional Rendering

```rust
impl RenderOnce for MyComponent {
    fn render(self, _: &mut Window, cx: &mut App) -> impl IntoElement {
        div()
            // Simple condition
            .when(self.is_active, |this| {
                this.bg(cx.theme().colors().element_active)
            })
            // Condition with else
            .map(|this| {
                if self.is_loading {
                    this.child("Loading...")
                } else {
                    this.child(self.content.clone())
                }
            })
            // Optional value
            .when_some(self.label.clone(), |this, label| {
                this.child(Label::new(label))
            })
    }
}
```

### 4. Animation Support

```rust
impl RenderOnce for AnimatedComponent {
    fn render(self, _: &mut Window, cx: &mut App) -> impl IntoElement {
        div()
            .transition_all(cx)  // Enable transitions
            .w(if self.expanded {
                px(200.0)
            } else {
                px(50.0)
            })
            .opacity(if self.visible { 1.0 } else { 0.0 })
    }
}
```

## PR Checklist

Before submitting:

### Code Quality
- [ ] Component follows builder pattern for configuration
- [ ] Uses theme colors, not hardcoded values
- [ ] Properly implements `Render` or `RenderOnce`
- [ ] Has `#[derive(IntoElement)]` if using `RenderOnce`
- [ ] Calls `cx.notify()` when state changes
- [ ] No use of `.unwrap()`

### Functionality
- [ ] Component is reusable
- [ ] Handles all interaction states (hover, active, disabled)
- [ ] Works with theme changes
- [ ] Accessible (focus states, keyboard navigation)
- [ ] Responsive to window size changes

### Testing
- [ ] Unit tests added
- [ ] Visual tests for rendering
- [ ] Interaction tests for user events
- [ ] Edge cases covered

### Documentation
- [ ] Doc comments explain component purpose
- [ ] Usage examples provided
- [ ] Builder methods documented
- [ ] Public API complete

### Style
- [ ] Consistent with existing UI components
- [ ] Uses standard spacing/sizing
- [ ] Follows design system
- [ ] Works in both light and dark themes

## Next Steps

- [Adding Settings](adding-setting.md) to make components configurable
- [Creating Panels](creating-panel.md) for larger UI features
- [Testing UI Components](../testing/ui-testing.md) for comprehensive coverage
- [UI Component Library](../../../crates/ui/src/components/) for examples

## Resources

- [UI crate source](../../../crates/ui/src/)
- [GPUI README](../../../crates/gpui/README.md)
- [Existing components](../../../crates/ui/src/components/)
- [Theme system](../../../crates/theme/)
