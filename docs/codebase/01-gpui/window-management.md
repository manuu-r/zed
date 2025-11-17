# GPUI Window Management

**Last Updated:** 2025-11-16

---

## Creating Windows

### Basic Window Creation

```rust
cx.open_window(WindowOptions::default(), |_, cx| {
    cx.new(|_| MyView::new())
})?;
```

### Window Options

```rust
cx.open_window(
    WindowOptions {
        bounds: WindowBounds::Fixed(Bounds {
            origin: point(px(100.0), px(100.0)),
            size: size(px(800.0), px(600.0)),
        }),
        titlebar: Some(TitlebarOptions {
            title: Some("My App".into()),
            appears_transparent: false,
            ..Default::default()
        }),
        center: false,
        focus: true,
        show: true,
        kind: WindowKind::Normal,
        is_movable: true,
        app_id: Some("com.example.myapp".into()),
        ..Default::default()
    },
    |_, cx| cx.new(|_| MyView::new())
)?;
```

## Window Bounds

### Fixed Size

```rust
WindowOptions {
    bounds: WindowBounds::Fixed(Bounds {
        origin: point(px(100.0), px(100.0)),
        size: size(px(800.0), px(600.0)),
    }),
    ..Default::default()
}
```

### Maximized

```rust
WindowOptions {
    bounds: WindowBounds::Maximized,
    ..Default::default()
}
```

### Fullscreen

```rust
WindowOptions {
    bounds: WindowBounds::Fullscreen,
    ..Default::default()
}
```

## Window Handles

### WindowHandle<T>

```rust
let window: WindowHandle<MyView> = cx.open_window(
    WindowOptions::default(),
    |_, cx| cx.new(|_| MyView::new())
)?;

// Update window
window.update(cx, |view, window, cx| {
    view.do_something(window, cx);
})?;

// Read from window
window.read(cx, |view, cx| {
    view.get_value()
})?;
```

### AnyWindowHandle

```rust
let any_window: AnyWindowHandle = window.into();

// Update without knowing type
any_window.update(cx, |view, window, cx| {
    // view is AnyView
})?;
```

## Window Methods

### Focus

```rust
window.update(cx, |view, window, cx| {
    window.focus(&focus_handle);
})?;
```

### Prompts

```rust
window.update(cx, |view, window, cx| {
    let response = window.prompt(
        PromptLevel::Info,
        "Save changes?",
        Some("You have unsaved changes."),
        &["Save", "Don't Save", "Cancel"],
        cx
    );

    cx.spawn_in(window, async move |this, cx| {
        let choice = response.await.ok()?;
        match choice {
            0 => { /* Save */ }
            1 => { /* Don't save */ }
            2 => { /* Cancel */ }
            _ => {}
        }
        Some(())
    }).detach();
})?;
```

### Window Properties

```rust
window.update(cx, |view, window, cx| {
    let bounds = window.bounds();
    let size = window.viewport_size();
    let appearance = window.appearance();
    let is_active = window.is_active();
})?;
```

## Multi-Window Applications

### Tracking Windows

```rust
pub struct App {
    windows: Vec<WindowHandle<EditorWindow>>,
}

impl App {
    fn new_window(&mut self, cx: &mut App) -> Result<()> {
        let window = cx.open_window(
            WindowOptions::default(),
            |_, cx| cx.new(|cx| EditorWindow::new(cx))
        )?;

        self.windows.push(window);
        Ok(())
    }

    fn close_window(&mut self, window_id: WindowId) {
        self.windows.retain(|w| w.window_id() != window_id);
    }
}
```

### Window Communication

```rust
// Update another window
other_window.update(cx, |view, window, cx| {
    view.handle_message(message, window, cx);
})?;

// Broadcast to all windows
for window in &self.windows {
    window.update(cx, |view, window, cx| {
        view.handle_event(event, window, cx);
    }).ok();
}
```

## Platform Integration

### Native Menus

```rust
cx.set_menus(vec![
    Menu {
        name: "File".into(),
        items: vec![
            MenuItem::Action {
                name: "New".into(),
                action: Box::new(New),
                os_action: None,
            },
            MenuItem::Separator,
            MenuItem::Action {
                name: "Open".into(),
                action: Box::new(Open),
                os_action: Some(OsAction::Open),
            },
        ],
    },
]);
```

### Titlebar Customization

```rust
WindowOptions {
    titlebar: Some(TitlebarOptions {
        title: Some("My App".into()),
        appears_transparent: true,
        traffic_light_position: Some(point(px(10.0), px(10.0))),
        ..Default::default()
    }),
    ..Default::default()
}
```

## Window Lifecycle

```
Create Window
  │
  ├─→ Initialize view
  │
  ├─→ First render
  │
  ├─→ Event loop
  │     │
  │     ├─→ Input events
  │     ├─→ Updates
  │     └─→ Renders
  │
  └─→ Close Window
        │
        ├─→ Drop view
        └─→ Cleanup
```

## Further Reading

- [Core Concepts](./core-concepts.md)
- [Events and Actions](./events-and-actions.md)
- [Architecture Overview](../00-overview.md)
