# Implementing Vim Motions

This guide shows you how to add new motions, operators, and commands to Zed's Vim mode.

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Vim Mode Architecture](#vim-mode-architecture)
4. [Adding a Motion](#adding-a-motion)
5. [Adding an Operator](#adding-an-operator)
6. [Adding a Command](#adding-a-command)
7. [Testing Against Neovim](#testing-against-neovim)
8. [Common Patterns](#common-patterns)
9. [PR Checklist](#pr-checklist)

## Overview

**What you'll learn:**
- How Zed's Vim mode is structured
- How to implement motions (movement commands)
- How to implement operators (commands that act on text objects)
- How to test Vim behavior against Neovim

**When to use this:**
- Adding missing Vim commands
- Fixing Vim mode bugs
- Extending Vim functionality

**Time to complete:** 1-3 hours

## Prerequisites

- Familiarity with Vim/Neovim
- Understanding of Vim's modal editing
- Knowledge of editor selections and movements

## Vim Mode Architecture

### Key Components

```
crates/vim/src/
├── vim.rs           # Main Vim state
├── normal.rs        # Normal mode handlers
├── insert.rs        # Insert mode handlers
├── visual.rs        # Visual mode handlers
├── motion.rs        # Motion implementations
├── object.rs        # Text object implementations
├── command.rs       # Ex commands
└── test/            # Neovim-backed tests
```

### State Management

```rust
pub struct Vim {
    // Current mode
    mode: Mode,

    // Operator state (when pending)
    operator: Option<Operator>,

    // Count (e.g., "5" in "5j")
    count: Option<usize>,

    // Register (e.g., "a in "ay)
    register: Option<String>,

    // ... more state
}
```

## Adding a Motion

Let's add support for `ge` (move backward to end of word).

### Step 1: Define the Action

```rust
// File: crates/vim/src/vim.rs or normal.rs

actions!(vim, [
    /// Move backward to end of word
    MoveToEndOfPreviousWord,
]);
```

### Step 2: Implement the Motion Logic

```rust
// File: crates/vim/src/motion.rs

impl Motion {
    pub fn to_end_of_previous_word(
        map: &DisplaySnapshot,
        mut point: DisplayPoint,
        times: usize,
    ) -> DisplayPoint {
        for _ in 0..times {
            // Move to previous word boundary
            point = movement::find_preceding_boundary(
                map,
                point,
                movement::FindRange::SingleLine,
                |left, right| {
                    // Word boundary logic
                    left.kind() != right.kind()
                },
            );

            // Move to end of that word
            if point.column() > 0 {
                point = movement::saturating_left(map, point);
            }
        }

        point
    }
}
```

### Step 3: Add to Motion Enum

```rust
// File: crates/vim/src/motion.rs

#[derive(Clone, Copy, Debug, PartialEq)]
pub enum Motion {
    // ... existing motions

    /// Move backward to end of word (ge)
    EndOfPreviousWord {
        ignore_punctuation: bool,
    },
}

impl Motion {
    pub fn move_point(
        &self,
        map: &DisplaySnapshot,
        point: DisplayPoint,
        goal: SelectionGoal,
        times: Option<usize>,
    ) -> (DisplayPoint, SelectionGoal) {
        let times = times.unwrap_or(1);

        match self {
            // ... existing matches

            Motion::EndOfPreviousWord { ignore_punctuation } => {
                let point = Self::to_end_of_previous_word(
                    map,
                    point,
                    times,
                );
                (point, SelectionGoal::None)
            }
        }
    }
}
```

### Step 4: Register the Keybinding

```rust
// File: crates/vim/src/normal.rs

pub fn normal_mode_bindings() -> Vec<Binding> {
    vec![
        // ... existing bindings

        Binding::new("g e", MoveToEndOfPreviousWord),
    ]
}
```

### Step 5: Connect Action to Motion

```rust
// File: crates/vim/src/normal.rs

impl Vim {
    pub fn move_to_end_of_previous_word(
        &mut self,
        _action: &MoveToEndOfPreviousWord,
        cx: &mut Context<Self>,
    ) {
        self.motion(
            Motion::EndOfPreviousWord {
                ignore_punctuation: false,
            },
            cx,
        )
    }
}

// Register the action handler
editor.on_action(cx.listener(Vim::move_to_end_of_previous_word));
```

## Adding an Operator

Let's add the `gu` operator (convert to lowercase).

### Step 1: Define the Operator

```rust
// File: crates/vim/src/state.rs

#[derive(Clone, Copy, Debug, PartialEq)]
pub enum Operator {
    // ... existing operators
    Change,
    Delete,
    Yank,

    /// Convert to lowercase
    Lowercase,
    /// Convert to uppercase
    Uppercase,
}
```

### Step 2: Define the Action

```rust
// File: crates/vim/src/normal.rs

actions!(vim, [
    /// Convert to lowercase
    PushLowercase,
    /// Convert to uppercase
    PushUppercase,
]);
```

### Step 3: Implement the Operator Handler

```rust
// File: crates/vim/src/normal.rs

impl Vim {
    pub fn push_lowercase(&mut self, _: &PushLowercase, cx: &mut Context<Self>) {
        self.push_operator(Operator::Lowercase, cx);
    }

    pub fn push_uppercase(&mut self, _: &PushUppercase, cx: &mut Context<Self>) {
        self.push_operator(Operator::Uppercase, cx);
    }
}
```

### Step 4: Implement the Transformation

```rust
// File: crates/vim/src/vim.rs

impl Vim {
    fn apply_operator(
        &mut self,
        operator: Operator,
        editor: &mut Editor,
        range: Range<Point>,
        cx: &mut Context<Self>,
    ) {
        match operator {
            // ... existing operators

            Operator::Lowercase => {
                editor.transact(cx, |editor, cx| {
                    editor.manipulate_text_in_range(range, cx, |text| {
                        text.to_lowercase()
                    });
                });
            }

            Operator::Uppercase => {
                editor.transact(cx, |editor, cx| {
                    editor.manipulate_text_in_range(range, cx, |text| {
                        text.to_uppercase()
                    });
                });
            }
        }
    }
}
```

### Step 5: Register Keybindings

```rust
// File: crates/vim/src/normal.rs

pub fn normal_mode_bindings() -> Vec<Binding> {
    vec![
        // ... existing bindings

        Binding::new("g u", PushLowercase),
        Binding::new("g U", PushUppercase),
    ]
}
```

## Adding a Command

Let's add `:sort` command.

### Step 1: Define the Command

```rust
// File: crates/vim/src/command.rs

#[derive(Debug, Clone, PartialEq)]
pub enum Command {
    // ... existing commands
    Write,
    Quit,

    /// Sort lines in range
    Sort {
        reverse: bool,
        ignore_case: bool,
    },
}
```

### Step 2: Parse the Command

```rust
// File: crates/vim/src/command.rs

impl Command {
    pub fn parse(input: &str) -> Option<Command> {
        let input = input.trim();

        // ... existing parsing logic

        if input.starts_with("sort") {
            let args = &input[4..].trim();
            let reverse = args.contains('!');
            let ignore_case = args.contains('i');

            return Some(Command::Sort {
                reverse,
                ignore_case,
            });
        }

        None
    }
}
```

### Step 3: Implement the Command

```rust
// File: crates/vim/src/command.rs

impl Command {
    pub fn execute(
        &self,
        vim: &mut Vim,
        editor: &mut Editor,
        cx: &mut Context<Vim>,
    ) {
        match self {
            // ... existing commands

            Command::Sort { reverse, ignore_case } => {
                Self::execute_sort(*reverse, *ignore_case, editor, cx);
            }
        }
    }

    fn execute_sort(
        reverse: bool,
        ignore_case: bool,
        editor: &mut Editor,
        cx: &mut Context<Vim>,
    ) {
        editor.transact(cx, |editor, cx| {
            let selections = editor.selections.all::<Point>(cx);

            for selection in selections {
                let start_row = selection.start.row;
                let end_row = selection.end.row;

                // Get lines in range
                let snapshot = editor.buffer.read(cx).snapshot(cx);
                let mut lines: Vec<String> = (start_row..=end_row)
                    .map(|row| {
                        snapshot
                            .text_for_range(
                                Point::new(row, 0)..Point::new(row + 1, 0)
                            )
                            .collect()
                    })
                    .collect();

                // Sort lines
                lines.sort_by(|a, b| {
                    let a = if ignore_case { a.to_lowercase() } else { a.clone() };
                    let b = if ignore_case { b.to_lowercase() } else { b.clone() };

                    if reverse {
                        b.cmp(&a)
                    } else {
                        a.cmp(&b)
                    }
                });

                // Replace text
                let sorted_text = lines.join("");
                let range = Point::new(start_row, 0)..Point::new(end_row + 1, 0);

                editor.buffer.update(cx, |buffer, cx| {
                    buffer.edit([(range, sorted_text)], None, cx);
                });
            }
        });
    }
}
```

## Testing Against Neovim

Zed's Vim mode uses Neovim-backed tests to ensure compatibility.

### Writing a Test

```rust
// File: crates/vim/src/test/vim_tests.rs

#[gpui::test]
async fn test_move_to_end_of_previous_word(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("hello world testˇ").await;
    cx.simulate_shared_keystrokes("g e").await;
    cx.shared_state().await.assert_eq("hello world ˇtest");

    cx.simulate_shared_keystrokes("g e").await;
    cx.shared_state().await.assert_eq("hello ˇworld test");
}

#[gpui::test]
async fn test_lowercase_operator(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state("HELLO ˇWORLD").await;
    cx.simulate_shared_keystrokes("g u i w").await;
    cx.shared_state().await.assert_eq("HELLO ˇworld");
}

#[gpui::test]
async fn test_sort_command(cx: &mut TestAppContext) {
    let mut cx = NeovimBackedTestContext::new(cx).await;

    cx.set_shared_state(indoc! {"
        ˇzebra
        apple
        banana
    "}).await;

    cx.simulate_shared_keystrokes("V j j").await;  // Select all lines
    cx.simulate_shared_keystrokes(": sort").await;
    cx.simulate_shared_keystrokes("enter").await;

    cx.shared_state().await.assert_eq(indoc! {"
        ˇapple
        banana
        zebra
    "});
}
```

### Running Vim Tests

```bash
# Run all vim tests
cargo test -p vim

# Run specific test
cargo test -p vim test_move_to_end_of_previous_word

# Run with Neovim output
RUST_LOG=debug cargo test -p vim test_name -- --nocapture
```

## Common Patterns

### Pattern 1: Motion with Count

```rust
pub fn my_motion(&mut self, action: &MyMotion, cx: &mut Context<Self>) {
    let count = self.take_count(cx).unwrap_or(1);

    self.motion(
        Motion::MyMotion { count },
        cx,
    );
}
```

### Pattern 2: Operator with Motion

```rust
// User types "dw" (delete word)
// 1. "d" sets operator to Delete
// 2. "w" provides the motion
// 3. Vim applies Delete operator to the word

pub fn delete(&mut self, _: &Delete, cx: &mut Context<Self>) {
    self.push_operator(Operator::Delete, cx);
}
```

### Pattern 3: Visual Mode Operation

```rust
pub fn visual_operation(&mut self, cx: &mut Context<Self>) {
    if let Mode::Visual { .. } = self.mode {
        let selections = /* get visual selections */;

        // Apply operation to selections
        self.apply_operator_to_selections(selections, cx);

        // Return to normal mode
        self.switch_mode(Mode::Normal, cx);
    }
}
```

### Pattern 4: Text Objects

```rust
// "iw" = inner word, "aw" = around word
impl Object {
    pub fn range(
        &self,
        map: &DisplaySnapshot,
        relative_to: DisplayPoint,
        around: bool,
    ) -> Option<Range<DisplayPoint>> {
        match self {
            Object::Word { .. } => {
                if around {
                    // Include surrounding whitespace
                } else {
                    // Just the word
                }
            }
        }
    }
}
```

### Pattern 5: Dot Repeat

```rust
// Make action repeatable with "."
pub fn my_action(&mut self, action: &MyAction, cx: &mut Context<Self>) {
    // Record for dot repeat
    self.record_current_action(cx);

    // Perform action
    self.perform_action(cx);
}
```

## Common Pitfalls

### 1. Not Handling Counts

❌ **Wrong:**
```rust
pub fn my_motion(&mut self, _: &MyMotion, cx: &mut Context<Self>) {
    // Always moves by 1, ignores count
    self.motion(Motion::MyMotion, cx);
}
```

✅ **Correct:**
```rust
pub fn my_motion(&mut self, _: &MyMotion, cx: &mut Context<Self>) {
    let count = self.take_count(cx).unwrap_or(1);
    self.motion(Motion::MyMotion { count }, cx);
}
```

### 2. Forgetting Register Support

```rust
pub fn yank_with_register(&mut self, cx: &mut Context<Self>) {
    let register = self.take_register(cx).unwrap_or('"');
    // Store in specified register
}
```

### 3. Not Supporting Visual Mode

Ensure your operator works in all modes:
- Normal mode (with motion)
- Visual mode (on selection)
- Visual line mode
- Visual block mode

### 4. Breaking Dot Repeat

Always call `record_current_action` for repeatable actions.

## PR Checklist

- [ ] Action defined and registered
- [ ] Motion/operator implemented correctly
- [ ] Handles count correctly
- [ ] Handles registers if applicable
- [ ] Works in all relevant modes
- [ ] Neovim-backed tests added
- [ ] Tests pass
- [ ] Dot repeat works (if applicable)
- [ ] Documented with doc comments
- [ ] Matches Neovim behavior

## Resources

- [Vim crate source](../../../crates/vim/src/)
- [Vim test examples](../../../crates/vim/src/test/)
- [Neovim documentation](https://neovim.io/doc/)
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)
