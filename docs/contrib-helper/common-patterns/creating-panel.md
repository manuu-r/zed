# Creating a Panel

This guide shows you how to create a new dockable panel in Zed's workspace.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Panel Architecture](#panel-architecture)
4. [Step-by-Step Implementation](#step-by-step-implementation)
5. [Complete Example](#complete-example)
6. [Panel State Persistence](#panel-state-persistence)
7. [Testing](#testing)
8. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How to implement the `Panel` trait
- How to create panel UI
- How to integrate with workspace
- How to persist panel state
- How to add toolbar actions

**When to use this:**
- Adding new sidebar panels
- Creating tool windows
- Building informational displays

**Time to complete:** 2-4 hours

## Prerequisites

- Understanding of GPUI's `Render` trait
- Familiarity with workspace structure
- Knowledge of state management

## Panel Architecture

### Panel Trait

```rust
pub trait Panel: Focusable + EventEmitter<PanelEvent> + Render {
    /// Unique identifier for this panel
    fn panel_name(&self) -> &'static str;

    /// Where the panel should be docked
    fn position(&self, window: &Window, cx: &App) -> DockPosition;

    /// Whether panel can be closed
    fn can_close(&self, cx: &App) -> bool {
        true
    }

    /// Icon to show in panel tab
    fn icon(&self, cx: &App) -> Option<IconName> {
        None
    }

    /// Label for panel tab
    fn icon_label(&self, cx: &App) -> Option<String> {
        None
    }

    /// Toggle panel visibility
    fn toggle_action(&self) -> Box<dyn Action>;
}
```

### Dock Positions

```rust
pub enum DockPosition {
    Left,
    Right,
    Bottom,
}
```

## Step-by-Step Implementation

Let's create a "Bookmarks Panel" that shows file bookmarks.

### Step 1: Define Panel Structure

```rust
// File: crates/bookmarks_panel/src/bookmarks_panel.rs

use gpui::*;
use ui::prelude::*;
use workspace::{Panel, PanelEvent};

pub struct BookmarksPanel {
    focus_handle: FocusHandle,
    bookmarks: Vec<Bookmark>,
    selected_index: Option<usize>,
    width: Option<Pixels>,
}

#[derive(Clone, Debug)]
pub struct Bookmark {
    pub file_path: PathBuf,
    pub line: u32,
    pub label: String,
}
```

### Step 2: Implement Actions

```rust
actions!(bookmarks_panel, [
    /// Toggle bookmarks panel
    ToggleFocus,
    /// Add bookmark at cursor
    AddBookmark,
    /// Remove selected bookmark
    RemoveBookmark,
    /// Jump to selected bookmark
    OpenBookmark,
]);
```

### Step 3: Implement Panel Trait

```rust
impl Panel for BookmarksPanel {
    fn panel_name(&self) -> &'static str {
        "BookmarksPanel"
    }

    fn position(&self, _window: &Window, _cx: &App) -> DockPosition {
        DockPosition::Right
    }

    fn can_close(&self, _cx: &App) -> bool {
        true
    }

    fn icon(&self, _cx: &App) -> Option<IconName> {
        Some(IconName::Bookmark)
    }

    fn icon_label(&self, _cx: &App) -> Option<String> {
        Some(format!("{} bookmarks", self.bookmarks.len()))
    }

    fn toggle_action(&self) -> Box<dyn Action> {
        Box::new(ToggleFocus)
    }
}
```

### Step 4: Implement Focusable

```rust
impl Focusable for BookmarksPanel {
    fn focus_handle(&self, _cx: &App) -> FocusHandle {
        self.focus_handle.clone()
    }
}
```

### Step 5: Implement EventEmitter

```rust
impl EventEmitter<PanelEvent> for BookmarksPanel {}
```

### Step 6: Implement Constructor

```rust
impl BookmarksPanel {
    pub fn new(cx: &mut Context<Self>) -> Self {
        Self {
            focus_handle: cx.focus_handle(),
            bookmarks: Vec::new(),
            selected_index: None,
            width: None,
        }
    }

    pub fn load(
        workspace: WeakEntity<Workspace>,
        cx: &mut Context<Self>,
    ) -> Task<Result<Entity<Self>>> {
        cx.spawn(async move |_, cx| {
            // Load saved state
            let panel = cx.build_entity(|cx| Self::new(cx))?;

            // Restore bookmarks from disk
            panel.update(&cx, |panel, cx| {
                panel.load_bookmarks(cx);
            })?;

            Ok(panel)
        })
    }

    fn load_bookmarks(&mut self, cx: &mut Context<Self>) {
        // Load from persisted state
        if let Some(bookmarks) = self.read_bookmarks_from_disk() {
            self.bookmarks = bookmarks;
            cx.notify();
        }
    }
}
```

### Step 7: Implement Render

```rust
impl Render for BookmarksPanel {
    fn render(&mut self, _window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        v_flex()
            .track_focus(&self.focus_handle)
            .size_full()
            .child(self.render_toolbar(cx))
            .child(self.render_bookmarks_list(cx))
    }
}

impl BookmarksPanel {
    fn render_toolbar(&self, cx: &mut Context<Self>) -> impl IntoElement {
        h_flex()
            .h(px(32.0))
            .px_2()
            .gap_1()
            .bg(cx.theme().colors().title_bar_background)
            .border_b_1()
            .border_color(cx.theme().colors().border)
            .child(
                IconButton::new("add_bookmark", IconName::Plus)
                    .tooltip("Add Bookmark")
                    .on_click(cx.listener(|this, _, _window, cx| {
                        this.add_bookmark(cx);
                    }))
            )
            .child(
                IconButton::new("remove_bookmark", IconName::Trash)
                    .tooltip("Remove Bookmark")
                    .disabled(self.selected_index.is_none())
                    .on_click(cx.listener(|this, _, _window, cx| {
                        this.remove_selected_bookmark(cx);
                    }))
            )
    }

    fn render_bookmarks_list(&self, cx: &mut Context<Self>) -> impl IntoElement {
        if self.bookmarks.is_empty() {
            return v_flex()
                .size_full()
                .items_center()
                .justify_center()
                .child(
                    Label::new("No bookmarks")
                        .color(Color::Muted)
                )
                .into_any_element();
        }

        list(self.bookmarks.clone())
            .size_full()
            .with_sizing_behavior(ListSizingBehavior::Infer)
            .track_scroll(self.scroll_handle.clone())
            .children(
                self.bookmarks
                    .iter()
                    .enumerate()
                    .map(|(index, bookmark)| {
                        self.render_bookmark(index, bookmark, cx)
                    })
            )
            .into_any_element()
    }

    fn render_bookmark(
        &self,
        index: usize,
        bookmark: &Bookmark,
        cx: &mut Context<Self>,
    ) -> impl IntoElement {
        let selected = self.selected_index == Some(index);

        h_flex()
            .w_full()
            .px_2()
            .py_1()
            .gap_2()
            .when(selected, |this| {
                this.bg(cx.theme().colors().element_selected)
            })
            .hover(|style| {
                style.bg(cx.theme().colors().element_hover)
            })
            .cursor_pointer()
            .on_click(cx.listener(move |this, _, _window, cx| {
                this.select_bookmark(index, cx);
            }))
            .child(
                Icon::new(IconName::Bookmark)
                    .color(Color::Accent)
                    .size(IconSize::Small)
            )
            .child(
                v_flex()
                    .flex_1()
                    .gap_1()
                    .child(
                        Label::new(bookmark.label.clone())
                            .size(LabelSize::Small)
                    )
                    .child(
                        Label::new(format!(
                            "{}:{}",
                            bookmark.file_path.display(),
                            bookmark.line
                        ))
                        .size(LabelSize::XSmall)
                        .color(Color::Muted)
                    )
            )
    }
}
```

### Step 8: Implement Panel Actions

```rust
impl BookmarksPanel {
    fn add_bookmark(&mut self, cx: &mut Context<Self>) {
        // Get current editor position
        // Create bookmark
        let bookmark = Bookmark {
            file_path: current_file(),
            line: current_line(),
            label: format!("Bookmark {}", self.bookmarks.len() + 1),
        };

        self.bookmarks.push(bookmark);
        self.save_bookmarks(cx);
        cx.notify();
    }

    fn remove_selected_bookmark(&mut self, cx: &mut Context<Self>) {
        if let Some(index) = self.selected_index {
            self.bookmarks.remove(index);
            self.selected_index = None;
            self.save_bookmarks(cx);
            cx.notify();
        }
    }

    fn select_bookmark(&mut self, index: usize, cx: &mut Context<Self>) {
        self.selected_index = Some(index);
        cx.notify();
    }

    fn open_selected_bookmark(&mut self, cx: &mut Context<Self>) {
        if let Some(index) = self.selected_index {
            if let Some(bookmark) = self.bookmarks.get(index) {
                // Open file at bookmark location
                cx.emit(PanelEvent::ActivateEditor {
                    path: bookmark.file_path.clone(),
                    position: Some(Point::new(bookmark.line, 0)),
                });
            }
        }
    }
}
```

### Step 9: Register Panel with Workspace

```rust
// File: crates/zed/src/zed.rs or similar init file

pub fn init_bookmarks_panel(cx: &mut App) {
    workspace::register_panel::<BookmarksPanel>(cx, |workspace, cx| {
        BookmarksPanel::load(workspace.downgrade(), cx)
    });
}
```

## Complete Example

See the complete minimal panel example:

```rust
// Minimal panel implementation
use gpui::*;
use ui::prelude::*;
use workspace::{Panel, PanelEvent};

pub struct SimplePanel {
    focus_handle: FocusHandle,
    content: String,
}

actions!(simple_panel, [ToggleFocus]);

impl SimplePanel {
    pub fn new(cx: &mut Context<Self>) -> Self {
        Self {
            focus_handle: cx.focus_handle(),
            content: "Hello, Panel!".to_string(),
        }
    }
}

impl Panel for SimplePanel {
    fn panel_name(&self) -> &'static str {
        "SimplePanel"
    }

    fn position(&self, _: &Window, _: &App) -> DockPosition {
        DockPosition::Right
    }

    fn toggle_action(&self) -> Box<dyn Action> {
        Box::new(ToggleFocus)
    }
}

impl Focusable for SimplePanel {
    fn focus_handle(&self, _: &App) -> FocusHandle {
        self.focus_handle.clone()
    }
}

impl EventEmitter<PanelEvent> for SimplePanel {}

impl Render for SimplePanel {
    fn render(&mut self, _: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .track_focus(&self.focus_handle)
            .size_full()
            .p_4()
            .child(self.content.clone())
    }
}
```

## Panel State Persistence

### Saving State

```rust
impl BookmarksPanel {
    fn save_bookmarks(&self, cx: &mut Context<Self>) {
        let state = BookmarksPanelState {
            bookmarks: self.bookmarks.clone(),
            width: self.width,
        };

        cx.background_spawn(async move {
            if let Some(path) = bookmarks_state_file() {
                let json = serde_json::to_string_pretty(&state)?;
                std::fs::write(path, json)?;
            }
            Ok(())
        }).detach();
    }

    fn read_bookmarks_from_disk(&self) -> Option<Vec<Bookmark>> {
        let path = bookmarks_state_file()?;
        let json = std::fs::read_to_string(path).ok()?;
        let state: BookmarksPanelState = serde_json::from_str(&json).ok()?;
        Some(state.bookmarks)
    }
}

#[derive(Serialize, Deserialize)]
struct BookmarksPanelState {
    bookmarks: Vec<Bookmark>,
    width: Option<Pixels>,
}
```

### Loading State

```rust
impl BookmarksPanel {
    pub fn load(
        workspace: WeakEntity<Workspace>,
        cx: &mut Context<Self>,
    ) -> Task<Result<Entity<Self>>> {
        cx.spawn(async move |_, cx| {
            let panel = cx.build_entity(|cx| {
                let mut panel = Self::new(cx);

                // Load persisted state
                if let Some(state) = Self::load_state() {
                    panel.bookmarks = state.bookmarks;
                    panel.width = state.width;
                }

                panel
            })?;

            Ok(panel)
        })
    }
}
```

## Testing

### Unit Test

```rust
#[gpui::test]
async fn test_bookmarks_panel(cx: &mut TestApp) {
    let panel = cx.build_entity(|cx| BookmarksPanel::new(cx));

    // Test adding bookmark
    panel.update(cx, |panel, cx| {
        panel.add_bookmark(cx);
        assert_eq!(panel.bookmarks.len(), 1);
    });

    // Test removing bookmark
    panel.update(cx, |panel, cx| {
        panel.selected_index = Some(0);
        panel.remove_selected_bookmark(cx);
        assert_eq!(panel.bookmarks.len(), 0);
    });
}
```

### Integration Test

```rust
#[gpui::test]
async fn test_panel_in_workspace(cx: &mut TestApp) {
    let app_state = init_test(cx);

    let workspace = cx.build_entity(|cx| {
        Workspace::test_new(app_state.project.clone(), cx)
    });

    // Open panel
    workspace.update(cx, |workspace, cx| {
        workspace.toggle_panel::<BookmarksPanel>(cx);
    });

    // Verify panel is visible
    workspace.update(cx, |workspace, cx| {
        assert!(workspace.panel::<BookmarksPanel>(cx).is_some());
    });
}
```

## Common Pitfalls

### 1. Not Implementing All Required Traits

Must implement: `Panel`, `Focusable`, `EventEmitter<PanelEvent>`, `Render`

### 2. Forgetting to Register Panel

```rust
// Must call in init
workspace::register_panel::<MyPanel>(cx, constructor);
```

### 3. Not Handling Focus Correctly

```rust
// Always track focus
div().track_focus(&self.focus_handle)
```

### 4. Not Emitting Events

```rust
// Emit events for panel state changes
cx.emit(PanelEvent::Close);
cx.emit(PanelEvent::ZoomIn);
```

## PR Checklist

- [ ] Implements `Panel` trait
- [ ] Implements `Focusable` trait
- [ ] Implements `EventEmitter<PanelEvent>`
- [ ] Implements `Render` trait
- [ ] Toggle action defined
- [ ] Panel registered with workspace
- [ ] State persistence implemented (if needed)
- [ ] Tests added
- [ ] Keyboard navigation works
- [ ] Icon and label provided
- [ ] Works in all dock positions
- [ ] Handles window resize

## Resources

- [Panel trait](../../../crates/workspace/src/dock.rs)
- [Existing panels](../../../crates/)
- [Workspace integration](../../../crates/workspace/)
