<p align="center">
  <h1 align="center">KrKr2 Next</h1>
  <p align="center">A Next-Generation KiriKiri2 Runtime for iOS</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-In%20Development-orange" alt="Status">
  <img src="https://img.shields.io/badge/platform-iOS%20%7C%20macOS-blue" alt="Platform">
  <img src="https://img.shields.io/badge/engine-KiriKiri2-blue" alt="Engine">
  <img src="https://img.shields.io/badge/framework-Flutter-02569B" alt="Flutter">
  <img src="https://img.shields.io/badge/graphics-ANGLE(Metal)-red" alt="Graphics">
  <img src="https://img.shields.io/badge/license-GPL--3.0-blue" alt="License">
</p>

---

**Language / 语言**: [中文](README.md) | English

> 🙏 This project is a refactor based on [krkr2](https://github.com/2468785842/krkr2). It is developed as a new project referencing that upstream. Thanks to the original author.

## Overview

**KrKr2 Next** is a modern runtime for the [KiriKiri2](https://en.wikipedia.org/wiki/KiriKiri) visual novel engine, **focused on iOS** (macOS is kept as the development/debug target). It is fully compatible with original game scripts, uses ANGLE (Metal backend) + IOSurface for zero-copy hardware-accelerated rendering, and includes numerous optimizations for both rendering performance and script execution.

The project follows a "C++ engine + Flutter shell" architecture: the C++ engine renders offscreen to an IOSurface, Flutter displays it via a native texture with zero-copy transfer, and the UI is fully built with Flutter.

## Architecture

```
C++ Engine (cpp/core, TJS2) ──engine_api C ABI──> Dart FFI (flutter_engine_bridge)
        │ ANGLE EGL/GLES2 offscreen                  │ Flutter Texture
        └──────────────> IOSurface (zero-copy) ──────┘ display
```

- **Rendering pipeline**: the engine renders offscreen via ANGLE's EGL Pbuffer Surface (OpenGL ES 2.0, Metal backend);
  the result is passed to a Flutter texture through **IOSurface** with zero-copy transfer, identical on iOS and macOS.
- **Bridge layer**: `bridge/engine_api` exposes a stable C ABI (`engine_create` / `engine_tick` / `engine_destroy`, etc.);
  Dart prefers FFI, with MethodChannel as a fallback.

> 📖 Full developer/AI-agent documentation: **[docs/dev/](docs/dev/README.md)** (tech stack, architecture, key references, build, conventions).

## Platform Support

| Platform | Status | Graphics Backend | Texture Sharing | Engine Binary |
|----------|--------|------------------|-----------------|---------------|
| iOS | 🚧 Primary target, in development | Metal | IOSurface | Static library linked into Runner |
| macOS | ✅ Development target | Metal | IOSurface | dylib bundled into Frameworks |

> Android / Linux / Windows have been removed from the repository.

## Build

```bash
./build.sh ios release   # Build iOS
./build.sh macos debug   # Build macOS (development)
```

See [docs/dev/build.md](docs/dev/build.md) and [build.sh](build.sh).

## Development Progress

| Module | Status | Notes |
|--------|--------|-------|
| C++ Engine Core Build | ✅ Done | KiriKiri2 core engine compiles |
| ANGLE Rendering Migration | ✅ Mostly Done | EGL/GLES offscreen rendering, replacing the legacy Cocos2d-x + GLFW pipeline |
| engine_api Bridge Layer | ✅ Done | Stable C ABI: startup, main loop, input, memory stats, etc. |
| Flutter Plugin (IOSurface) | ✅ Mostly Done | Zero-copy texture + RGBA fallback path |
| Flutter Debug UI | ✅ Mostly Done | FPS control, engine lifecycle, rendering monitor |
| Input Event Forwarding | ✅ Mostly Done | Touch / pointer coordinate mapping and forwarding |
| Engine Performance | 🔨 In Progress | SIMD pixel blending (Highway), GPU compositing pipeline, etc. |
| Game Compatibility | 🔨 In Progress | Completing the script parser and plugins; target parity with Z's closed-source build |

## Related Docs

- Development docs (AI-agent quick reference): [docs/dev/](docs/dev/README.md)
- Upstream project: <https://github.com/2468785842/krkr2>

## License

This project is licensed under the GNU General Public License v3.0 (GPL-3.0). See [LICENSE](./LICENSE) for details.
