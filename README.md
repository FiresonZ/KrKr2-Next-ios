<p align="center">
  <h1 align="center">KrKr2 Next</h1>
  <p align="center">面向 iOS 的下一代 KiriKiri2（吉里吉里2）运行环境</p>
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

**语言 / Language**: 中文 | [English](README_EN.md)

> 🙏 本项目基于 [krkr2](https://github.com/2468785842/krkr2) 重构，作为新项目引用其上游，感谢原作者的贡献。

## 简介

**KrKr2 Next** 是 [KiriKiri2 (吉里吉里2)](https://zh.wikipedia.org/wiki/%E5%90%89%E9%87%8C%E5%90%89%E9%87%8C2) 视觉小说引擎的现代化运行环境，**专注 iOS 平台**（macOS 作为开发/调试目标）。它完全兼容原版游戏脚本，通过 ANGLE（Metal 后端）+ IOSurface 实现零拷贝硬件加速渲染，并在渲染性能与脚本执行效率上做了大量优化。

项目采用「C++ 引擎 + Flutter 壳」架构：C++ 引擎离屏渲染到 IOSurface，Flutter 以原生纹理零拷贝显示，UI 完全由 Flutter 构建。

## 架构

```
C++ 引擎 (cpp/core, TJS2) ──engine_api C ABI──> Dart FFI (flutter_engine_bridge)
        │ ANGLE EGL/GLES2 离屏渲染                     │ Flutter Texture
        └──────────────> IOSurface（零拷贝）───────────┘ 显示
```

- **渲染管线**：引擎通过 ANGLE（Metal 后端）的 EGL Pbuffer Surface 离屏渲染（OpenGL ES 2.0），
  结果经 **IOSurface** 零拷贝传递给 Flutter 纹理显示，iOS 与 macOS 一致。
- **桥接层**：`bridge/engine_api` 提供稳定 C ABI（`engine_create` / `engine_tick` / `engine_destroy` 等）；
  Dart 优先走 FFI，MethodChannel 为兜底。

> 📖 面向开发者/AI Agent 的完整技术文档见 **[docs/dev/](docs/dev/README.md)**（技术栈、架构、关键引用、构建、约定陷阱）。

## 平台支持

| 平台 | 状态 | 图形后端 | 纹理共享 | 引擎形态 |
|------|------|----------|----------|----------|
| iOS | 🚧 主目标，开发中 | Metal | IOSurface | 静态库链接进 Runner |
| macOS | ✅ 开发目标 | Metal | IOSurface | dylib 打包进 Frameworks |

> Android / Linux / Windows 已从仓库移除。

## 系统要求（iOS）

| 项 | 要求 |
|----|------|
| 系统版本 | **iOS / iPadOS 15.0 及以上** |
| 架构 | 仅 64 位（**arm64**） |
| 设备 | iPhone 6s 及以上；iPad Air 2 / iPad mini 4 及以上；iPod touch（第 7 代） |
| 芯片 | A8 / A9 及以上（即所有支持 iOS 15 的 64 位设备） |

> 构建与运行链路（引擎静态库、vcpkg 依赖、Flutter）均按 `arm64`、部署目标 `iOS 15.0` 配置，
> 见 `CMakePresets.json`、`vcpkg/triplets/arm64-ios.cmake` 与 `ios/Podfile`。

## 构建

```bash
./build.sh ios release   # 构建 iOS
./build.sh macos debug   # 构建 macOS（开发）
```

详见 [docs/dev/build.md](docs/dev/build.md) 与 [build.sh](build.sh)。

## CI 打包（GitHub Actions）

仓库内置在线打包工作流 [ios_package.yml](.github/workflows/ios_package.yml)：

- **触发方式**：
  - 手动：Actions → *iOS 打包* → Run workflow（可选 debug / release）
  - 自动：推送 `v*` 标签（如 `v1.0.0`）
- **产物**：`KrKr2-Next-iOS-<release|debug>.zip`（未签名的 `Runner.app`，保留 14 天）
- **安装到真机**：下载 zip → 解压出 `Runner.app` → 用自己的 Apple 开发者证书签名
  （推荐用 Xcode 打开 `apps/flutter_app/ios/Runner.xcworkspace` 配置 Team 后运行），
  或用 `flutter run -d <device>` 开发调试。
- **首次构建**：vcpkg 需全量编译 `arm64-ios` 依赖（FFmpeg/OpenCV/ANGLE 等），耗时较长；
  已启用 vcpkg 二进制缓存，后续运行秒级还原。

## 开发进度

| 模块 | 状态 | 说明 |
|------|------|------|
| C++ 引擎核心编译 | ✅ 完成 | KiriKiri2 核心引擎可编译 |
| ANGLE 渲染层迁移 | ✅ 基本完成 | EGL/GLES 离屏渲染，替代旧 Cocos2d-x + GLFW 管线 |
| engine_api 桥接层 | ✅ 完成 | 稳定 C ABI，含启动/主循环/输入/内存统计等 |
| Flutter 插件（IOSurface） | ✅ 基本完成 | 零拷贝纹理 + RGBA 兼容路径 |
| Flutter 调试 UI | ✅ 基本完成 | FPS 控制、引擎生命周期、渲染状态监控 |
| 输入事件转发 | ✅ 基本完成 | 触控 / 指针事件坐标映射转发 |
| 引擎性能优化 | 🔨 进行中 | SIMD 像素混合（Highway）、GPU 合成管线等 |
| 游戏兼容性优化 | 🔨 进行中 | 补全解析引擎、插件，目标与 Z 闭源版兼容持平 |

## 相关文档

- 开发文档（AI Agent 速查）：[docs/dev/](docs/dev/README.md)
- 上游项目：<https://github.com/2468785842/krkr2>

## 许可证

本项目基于 GNU General Public License v3.0 (GPL-3.0) 开源，详见 [LICENSE](./LICENSE)。
