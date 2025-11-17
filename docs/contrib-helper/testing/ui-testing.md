# UI Testing

Guide for testing GPUI components and visual behavior.

## Overview

UI tests verify that components render correctly and respond to user interactions.

## Quick Example

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

    // Simulate interaction
    button.update(cx, |button, cx| {
        // Trigger click
        let element = button.render(window, cx);
        // Interaction testing
    });

    assert!(clicked.get());
}
```

## Testing Patterns

### Testing Component Rendering

```rust
#[gpui::test]
async fn test_component_renders(cx: &mut TestApp) {
    let component = cx.build_entity(|cx| MyComponent::new(cx));

    component.update(cx, |component, cx| {
        let element = component.render(&mut Window::default(), cx);
        // Verify element structure
    });
}
```

### Testing State Changes

```rust
#[gpui::test]
async fn test_interactive_component(cx: &mut TestApp) {
    let panel = cx.build_entity(|cx| CollapsiblePanel::new("Test", cx));

    // Initial state
    assert!(panel.read(cx).expanded);

    // Interact
    panel.update(cx, |panel, cx| {
        panel.toggle_expanded(cx);
    });

    // Verify state changed
    assert!(!panel.read(cx).expanded);
}
```

## Resources

- [UI component tests](../../../crates/ui/src/components/)
- [GPUI test utilities](../../../crates/gpui/src/test.rs)
