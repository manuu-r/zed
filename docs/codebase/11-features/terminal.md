# Terminal Integration

**Last Updated:** 2025-11-16

---

## Overview

Zed includes an integrated terminal.

**Path:** `/home/user/zed/crates/terminal/` and `/home/user/zed/crates/terminal_view/`

## Terminal Component

```rust
pub struct Terminal {
    pty: Pty,
    screen: Screen,
}

impl Terminal {
    pub fn new(working_directory: Option<PathBuf>, cx: &mut Context<Self>) -> Self {
        let pty = Pty::new(working_directory)?;
        Self { pty, screen: Screen::new() }
    }
}
```

## PTY Integration

Uses pseudo-terminal for process communication:

```rust
let pty = Pty::new(working_directory)?;
pty.spawn(shell_command)?;

// Read output
let output = pty.read().await?;
screen.process_output(&output);

// Send input
pty.write(input.as_bytes())?;
```

## Rendering

```rust
impl Render for TerminalView {
    fn render(&mut self, window: &mut Window, cx: &mut Context<Self>) -> impl IntoElement {
        div()
            .children(self.terminal.screen.lines().map(|line| {
                render_terminal_line(line)
            }))
    }
}
```

## Further Reading

- [Workspace Items](../04-workspace/items.md)
