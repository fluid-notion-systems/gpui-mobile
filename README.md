# GPUI Mobile

A research project exploring the feasibility of running Blade graphics library on mobile platforms, with a focus on Android support.

## Overview

This project investigates the current state of Rust on Android and explores the technical requirements for porting Blade - a lean GPU abstraction library - to mobile platforms. Blade is currently used in projects like Zed editor and provides cross-platform graphics capabilities across Vulkan, Metal, and OpenGL ES3.

## Project Structure

```
gpui-mobile/
├── vendor/
│   ├── blade/        # Blade graphics library
│   └── zed/          # Zed editor source
├── research/         # Research documentation
│   └── rust-on-android.md
└── README.md         # This file
```

## Goals

1. **Research Rust on Android**: Deep dive into the current state of Rust development for Android
2. **Blade Mobile Compatibility**: Assess what's needed to get Blade running on mobile platforms
3. **Technical Feasibility**: Document challenges and potential solutions
4. **Proof of Concept**: Create a working example if feasible

## Current Status

- [ ] Research Rust on Android ecosystem
- [ ] Analyze Blade's architecture for mobile compatibility
- [ ] Identify required modifications
- [ ] Create proof of concept

## Key Considerations

### Blade's Current Platform Support

According to Blade's documentation:
- **Vulkan**: Supported on desktop Linux, Windows, and Android
- **Metal**: Supported on desktop macOS and iOS
- **OpenGL ES3**: Supported on the Web

This suggests that Blade already has some level of mobile support through Vulkan (Android) and Metal (iOS).

### Technical Requirements

For Android specifically:
- Rust toolchain with Android targets
- Android NDK
- JNI bindings for Android integration
- Surface management for rendering
- Input handling

## Research Topics

See the `research/` directory for detailed investigations on:
- [Rust on Android](research/rust-on-android.md) - Current state, tooling, and best practices
- Blade mobile adaptation requirements
- Performance considerations
- Platform-specific challenges

## Getting Started

### Prerequisites

- Rust toolchain (1.65+)
- Android Studio and NDK (for Android development)
- Xcode (for iOS development)

### Setup

1. Clone the repository with submodules:
```bash
git clone --recursive https://github.com/yourusername/gpui-mobile.git
cd gpui-mobile
```

2. Install Rust Android targets:
```bash
rustup target add aarch64-linux-android
rustup target add armv7-linux-androideabi
rustup target add x86_64-linux-android
```

3. Set up Android development environment:
```bash
export ANDROID_HOME=$HOME/Android/Sdk
export NDK_HOME=$ANDROID_HOME/ndk/[version]
```

## Contributing

This is a research project. Contributions in the form of:
- Additional research
- Code experiments
- Documentation improvements
- Bug reports

are all welcome!

## License

This research project follows the licensing of its vendored dependencies:
- Blade is licensed under MIT
- Zed is licensed under multiple licenses (check vendor/zed for details)

## References

- [Blade Graphics](https://github.com/kvark/blade)
- [Rust on Android](https://mozilla.github.io/firefox-browser-architecture/experiments/2017-09-21-rust-on-android.html)
- [Android NDK](https://developer.android.com/ndk)