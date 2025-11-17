# Template: Creating a New Panel

Template for adding new workspace panels.

## Panel: [Panel Name]

### Purpose

What problem does this panel solve?

### User Stories

- As a user, I want to [action] so that [benefit]
- As a developer, I want to [action] so that [benefit]

### Design

#### Panel Layout

```
┌─────────────────────┐
│ Toolbar             │
├─────────────────────┤
│                     │
│  Main Content       │
│                     │
└─────────────────────┘
```

#### Features

- [ ] List items
- [ ] Search/filter
- [ ] Actions (add, remove, edit)
- [ ] Keyboard shortcuts
- [ ] State persistence

### Implementation

#### Core Panel

```rust
pub struct MyPanel {
    focus_handle: FocusHandle,
    items: Vec<Item>,
    selected: Option<usize>,
}

impl Panel for MyPanel {
    // Implementation
}
```

#### Integration

- Register with workspace
- Add toggle action
- Add keybindings
- Add settings

### Testing

- Unit tests for panel logic
- Integration tests with workspace
- UI interaction tests

## Resources

- [Panel trait](../common-patterns/creating-panel.md)
- [Existing panels]
