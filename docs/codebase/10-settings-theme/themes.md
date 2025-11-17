# Theme System

**Last Updated:** 2025-11-16

---

## Theme Structure

```rust
pub struct Theme {
    pub name: SharedString,
    pub appearance: Appearance,
    pub styles: ThemeStyles,
}

pub struct ThemeStyles {
    pub background: Hsla,
    pub foreground: Hsla,
    pub border: Hsla,
    pub syntax: SyntaxTheme,
}
```

## Syntax Highlighting

```rust
pub struct SyntaxTheme {
    pub highlights: Vec<(String, Hsla)>,
}

// Map token types to colors
let color = syntax_theme.color("function");
```

## Using Themes

```rust
let theme = cx.theme();

div()
    .bg(theme.background)
    .text_color(theme.foreground)
```

## Custom Themes

Create `~/.config/zed/themes/my-theme.json`:

```json
{
  "name": "My Theme",
  "appearance": "dark",
  "style": {
    "background": "#1e1e1e",
    "foreground": "#d4d4d4",
    "syntax": {
      "keyword": "#569cd6",
      "function": "#dcdcaa",
      "string": "#ce9178"
    }
  }
}
```

## Further Reading

- [Settings README](./README.md)
- [UI Styling](../07-ui/styling.md)
