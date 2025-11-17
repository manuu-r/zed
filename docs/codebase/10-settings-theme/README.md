# Settings and Theme System

**Path:** `/home/user/zed/crates/settings/` and `/home/user/zed/crates/theme/`
**Purpose:** Configuration and theming

---

## Overview

Manages user settings and theme system.

## Documentation Files

- **[Settings System](./settings-system.md)** - How settings work
- **[Themes](./themes.md)** - Theme system
- **README.md** - This file

## Settings File

`~/.config/zed/settings.json`:

```json
{
  "theme": "One Dark",
  "buffer_font_size": 14,
  "vim_mode": true,
  "format_on_save": "on"
}
```

## Theme File

`themes/one-dark.json`:

```json
{
  "name": "One Dark",
  "appearance": "dark",
  "style": {
    "background": "#282c34",
    "foreground": "#abb2bf"
  }
}
```

## Further Reading

- [UI](../07-ui/README.md)
- [GPUI](../01-gpui/README.md)
