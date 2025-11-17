# Editor LSP Integration

**Last Updated:** 2025-11-16

---

## Overview

The editor integrates deeply with Language Server Protocol (LSP) to provide IDE features.

## LSP Features in Editor

### Completions

Triggered on typing or explicitly:

```rust
impl Editor {
    fn show_completions(&mut self, cx: &mut Context<Self>) {
        let project = self.project.clone();
        let buffer = self.buffer.clone();
        let position = self.selections.newest_anchor().head();

        cx.spawn(async move |editor, cx| {
            let completions = project
                .completions(&buffer, position, cx)
                .await?;

            editor.update(cx, |editor, cx| {
                editor.show_completions_menu(completions, cx);
            })?;

            Ok(())
        }).detach();
    }
}
```

### Hover Information

Show type and documentation on hover:

```rust
fn show_hover(&mut self, point: DisplayPoint, cx: &mut Context<Self>) {
    let buffer_point = self.display_map
        .read(cx)
        .display_point_to_point(point);

    let task = self.project.hover(&self.buffer, buffer_point, cx);

    cx.spawn(async move |editor, cx| {
        let hover = task.await?;

        editor.update(cx, |editor, cx| {
            editor.show_hover_popover(hover, cx);
        })?;

        Ok(())
    }).detach();
}
```

### Diagnostics

Display errors and warnings:

```rust
pub struct Editor {
    diagnostics: HashMap<LanguageServerId, Vec<Diagnostic>>,
}

impl Editor {
    fn update_diagnostics(&mut self, cx: &mut Context<Self>) {
        // Subscribe to project diagnostics
        self.project.diagnostics(&self.buffer, cx)
    }

    fn render_diagnostics(&self, window: &mut Window, cx: &Context<Self>) {
        // Render squiggles, gutter icons
    }
}
```

### Code Actions

Quick fixes and refactorings:

```rust
fn show_code_actions(&mut self, cx: &mut Context<Self>) {
    let range = self.selections.newest_anchor().range();

    cx.spawn(async move |editor, cx| {
        let actions = project
            .code_actions(&buffer, range, cx)
            .await?;

        editor.update(cx, |editor, cx| {
            editor.show_code_actions_menu(actions, cx);
        })?;

        Ok(())
    }).detach();
}
```

### Go to Definition

```rust
fn go_to_definition(&mut self, cx: &mut Context<Self>) {
    let position = self.selections.newest_anchor().head();

    cx.spawn(async move |editor, cx| {
        let locations = project
            .definition(&buffer, position, cx)
            .await?;

        editor.update(cx, |editor, cx| {
            editor.navigate_to(locations.first()?, cx);
        })?;

        Ok(())
    }).detach();
}
```

### Find References

```rust
fn find_all_references(&mut self, cx: &mut Context<Self>) {
    let position = self.selections.newest_anchor().head();

    cx.spawn(async move |editor, cx| {
        let references = project
            .references(&buffer, position, cx)
            .await?;

        editor.update(cx, |editor, cx| {
            editor.show_references_panel(references, cx);
        })?;

        Ok(())
    }).detach();
}
```

### Rename Symbol

```rust
fn rename(&mut self, new_name: String, cx: &mut Context<Self>) {
    let position = self.selections.newest_anchor().head();

    cx.spawn(async move |editor, cx| {
        let edit = project
            .rename(&buffer, position, new_name, cx)
            .await?;

        editor.update(cx, |editor, cx| {
            editor.apply_workspace_edit(edit, cx);
        })?;

        Ok(())
    }).detach();
}
```

## Inlay Hints

Display inline type information:

```rust
pub struct Editor {
    inlay_hint_cache: InlayHintCache,
}

impl Editor {
    fn refresh_inlay_hints(&mut self, cx: &mut Context<Self>) {
        let visible_range = self.visible_range(cx);

        cx.spawn(async move |editor, cx| {
            let hints = project
                .inlay_hints(&buffer, visible_range, cx)
                .await?;

            editor.update(cx, |editor, cx| {
                editor.display_map.update(cx, |map, cx| {
                    map.splice_inlays(hints, cx);
                });
                cx.notify();
            })?;

            Ok(())
        }).detach();
    }
}
```

## Signature Help

Show function signatures while typing:

```rust
fn show_signature_help(&mut self, cx: &mut Context<Self>) {
    let position = self.selections.newest_anchor().head();

    cx.spawn(async move |editor, cx| {
        let signature = project
            .signature_help(&buffer, position, cx)
            .await?;

        editor.update(cx, |editor, cx| {
            editor.show_signature_help_popover(signature, cx);
        })?;

        Ok(())
    }).detach();
}
```

## Formatting

Format on save or on demand:

```rust
fn format(&mut self, cx: &mut Context<Self>) {
    cx.spawn(async move |editor, cx| {
        let edits = project
            .format(&buffer, cx)
            .await?;

        editor.update(cx, |editor, cx| {
            editor.apply_edits(edits, cx);
        })?;

        Ok(())
    }).detach();
}
```

## LSP Event Handling

```rust
impl Editor {
    fn on_buffer_event(
        &mut self,
        _buffer: Entity<Buffer>,
        event: &BufferEvent,
        cx: &mut Context<Self>,
    ) {
        match event {
            BufferEvent::Edited => {
                // Request new diagnostics
                self.refresh_diagnostics(cx);

                // Update inlay hints
                self.refresh_inlay_hints(cx);
            }
            BufferEvent::Saved => {
                // Format on save
                if self.settings.format_on_save {
                    self.format(cx);
                }
            }
            _ => {}
        }
    }
}
```

## Further Reading

- [Project LSP Store](../03-project/lsp-store.md)
- [LSP Documentation](../06-lsp/README.md)
- [Language Documentation](../05-language/README.md)
