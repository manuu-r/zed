# Platform-Specific Development

Guide for developing platform-specific features in Zed.

## Platform Overview

Zed supports:
- **macOS** - Primary development platform
- **Linux** - Wayland and X11
- **Windows** - In development

## Platform-Specific Code

### Conditional Compilation

```rust
#[cfg(target_os = "macos")]
fn platform_specific_function() {
    // macOS implementation
}

#[cfg(target_os = "linux")]
fn platform_specific_function() {
    // Linux implementation
}

#[cfg(target_os = "windows")]
fn platform_specific_function() {
    // Windows implementation
}
```

### Feature Flags

```toml
# Cargo.toml
[target.'cfg(target_os = "linux")'.dependencies]
wayland-client = "0.31"

[target.'cfg(target_os = "macos")'.dependencies]
core-foundation = "0.9"
```

## macOS-Specific

### Using macOS APIs

```rust
#[cfg(target_os = "macos")]
mod macos {
    use core_foundation::*;

    pub fn get_system_appearance() -> Appearance {
        // Use macOS APIs
    }
}
```

### Menu Bar Integration

```rust
#[cfg(target_os = "macos")]
impl App {
    fn setup_menu_bar(&self) {
        // macOS menu bar
    }
}
```

### Window Management

```rust
#[cfg(target_os = "macos")]
impl Window {
    fn enable_native_titlebar(&mut self) {
        // macOS-specific window features
    }
}
```

## Linux-Specific

### Wayland vs X11

```rust
#[cfg(all(target_os = "linux", feature = "wayland"))]
mod wayland {
    // Wayland implementation
}

#[cfg(all(target_os = "linux", not(feature = "wayland")))]
mod x11 {
    // X11 implementation
}
```

### Building for Linux

```bash
# Wayland (default)
cargo build --release

# X11
cargo build --release --no-default-features --features x11
```

### Linux Dependencies

```bash
# Ubuntu/Debian
sudo apt install \
    libfontconfig1-dev \
    libfreetype6-dev \
    libxcb-composite0-dev \
    libxkbcommon-dev \
    libwayland-dev

# Arch
sudo pacman -S \
    fontconfig \
    freetype2 \
    xcb-util \
    libxkbcommon \
    wayland
```

## Windows-Specific

### Windows Development

```rust
#[cfg(target_os = "windows")]
mod windows {
    use windows::*;

    pub fn windows_specific_feature() {
        // Windows implementation
    }
}
```

### Building for Windows

```bash
cargo build --target x86_64-pc-windows-msvc
```

## Testing Across Platforms

### CI Testing

All platforms are tested in CI:
- macOS (latest)
- Linux (Ubuntu)
- Windows (when supported)

### Local Testing

Use virtual machines or CI for testing other platforms:

```bash
# Test on Linux with Docker
docker run -it ubuntu:latest

# Test on macOS with GitHub Actions (push to branch)
```

## Platform Features

### Clipboard

```rust
#[cfg(target_os = "macos")]
fn copy_to_clipboard(text: &str) {
    use cocoa::appkit::NSPasteboard;
    // macOS clipboard
}

#[cfg(target_os = "linux")]
fn copy_to_clipboard(text: &str) {
    use x11_clipboard::Clipboard;
    // Linux clipboard
}
```

### File System

```rust
#[cfg(unix)]
fn get_home_dir() -> PathBuf {
    std::env::var("HOME").unwrap().into()
}

#[cfg(windows)]
fn get_home_dir() -> PathBuf {
    std::env::var("USERPROFILE").unwrap().into()
}
```

### Keyboard Shortcuts

```json
// default-macos.json
{
  "cmd-s": "workspace::Save"
}

// default-linux.json
{
  "ctrl-s": "workspace::Save"
}
```

## Resources

- [GPUI Platform Code](../../../crates/gpui/src/platform/)
- [Platform-Specific Tests]
