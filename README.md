# Hydra Desktop (Tauri)

Desktop version of [Hydra](https://github.com/hydra-synth/hydra) - live coding video synthesizer, built with Tauri for native performance.

> **Original Hydra**: https://hydra.ojack.xyz  
> **Original Repository**: https://github.com/hydra-synth/hydra

## What's Different from Original Hydra

This fork adds desktop application features:

### 1. Local File Support
Load local video and image files using helper functions:

```javascript
// Load local video
loadVideo(s0, "C:/Users/YourName/Videos/video.mp4")
src(s0).out()

// Load local image
loadImage(s1, "C:/Users/YourName/Pictures/image.jpg")
src(s1).out()

// Or use file picker dialog
const fileUrl = await pickFile()
if (fileUrl) {
  s0.initVideo(fileUrl)
  src(s0).out()
}
```

### 2. Fullscreen Mode
- **F11** or **Alt+Enter**: Toggle fullscreen
- **ESC**: Exit fullscreen
- Or call `toggleFullscreen()` in console

### 3. Developer Tools
- **F12**: Open developer console (debug builds only)

### 4. Available Helper Functions
- `localFile(path)`: Convert Windows path to asset URL
- `loadVideo(source, path)`: Load local video file
- `loadImage(source, path)`: Load local image file
- `pickFile(options)`: Open file picker dialog
- `toggleFullscreen()`: Toggle fullscreen mode

## Building

### Prerequisites
- Linux (Arch Linux recommended)
- Node.js and npm
- Rust toolchain (`rustup`)

### Common Setup

```bash
# Clone and install dependencies
git clone <your-repo-url>
cd hydra
npm install
```

### Building for Linux

#### Install Linux Dependencies (Arch Linux)

```bash
# Install Tauri dependencies
sudo pacman -S webkit2gtk-4.1 gtk3 libayatana-appindicator

# Optional: NSIS for creating installer packages
sudo pacman -S nsis
```

#### Build

```bash
# Build Linux packages
npm run tauri build
```

#### Output Files

- **AppImage**: `src-tauri/target/release/bundle/appimage/Hydra_*_amd64.AppImage` (portable, requires linuxdeploy)
- **DEB package**: `src-tauri/target/release/bundle/deb/Hydra_*_amd64.deb`
- **RPM package**: `src-tauri/target/release/bundle/rpm/Hydra-*.x86_64.rpm`
- **Binary**: `src-tauri/target/release/app`

#### Running on Linux

```bash
# Run the binary directly
./src-tauri/target/release/app

# Or install DEB package (Debian/Ubuntu)
sudo dpkg -i src-tauri/target/release/bundle/deb/Hydra_*_amd64.deb

# Or run AppImage (if built)
chmod +x src-tauri/target/release/bundle/appimage/Hydra_*_amd64.AppImage
./src-tauri/target/release/bundle/appimage/Hydra_*_amd64.AppImage
```

#### Example: Local video on Linux

```javascript
// Linux path format
loadVideo(s0, "/home/username/Videos/video.mp4")
src(s0).out()
```

### Building for Windows (Cross-compile from Linux)

#### Setup

```bash
# Add Windows target
rustup target add x86_64-pc-windows-gnu

# Install MinGW cross-compiler (Arch Linux)
sudo pacman -S mingw-w64-gcc nsis

# Configure Cargo for cross-compilation
cat > ~/.cargo/config.toml << 'EOF'
[target.x86_64-pc-windows-gnu]
linker = "x86_64-w64-mingw32-gcc"
ar = "x86_64-w64-mingw32-ar"
EOF
```

#### Build

```bash
# Build Windows installer
npx tauri build --target x86_64-pc-windows-gnu
```

#### Output Files

- **Installer**: `src-tauri/target/x86_64-pc-windows-gnu/release/bundle/nsis/Hydra_*_x64-setup.exe`
- **Executable**: `src-tauri/target/x86_64-pc-windows-gnu/release/app.exe`

#### Example: Local video on Windows

```javascript
// Windows path format (use forward slashes or escaped backslashes)
loadVideo(s0, "C:/Users/YourName/Videos/video.mp4")
src(s0).out()
```

## Original Hydra Documentation

For Hydra syntax, functions, and examples, see:
- [Getting Started](https://github.com/hydra-synth/hydra#getting-started)
- [Function Reference](https://github.com/hydra-synth/hydra/blob/main/docs/funcs.md)
- [Examples](https://github.com/hydra-synth/hydra/tree/main/examples)
- [User Patterns](https://twitter.com/hydra_patterns)

### Quick Start

```javascript
// Oscillator
osc(20, 0.1, 0.8).out()

// Webcam
s0.initCam()
src(s0).out()

// Kaleidoscope
s0.initCam()
src(s0).kaleid(4).out()

// Local video (desktop only)
loadVideo(s0, "C:/path/to/video.mp4")
src(s0).kaleid(4).out()
```

### Keyboard Shortcuts

**Web version:**
- `Ctrl+Enter`: Run line
- `Ctrl+Shift+Enter`: Run all code
- `Alt+Enter`: Run block
- `Ctrl+Shift+H`: Hide/show code

**Desktop additions:**
- `F11` or `Alt+Enter`: Fullscreen
- `F12`: Developer tools

## License

AGPL-3.0 (same as original Hydra)

## Credits

Original Hydra by [Olivia Jack](https://github.com/ojack)  
Desktop port modifications: Local file support, fullscreen toggle, Tauri integration
