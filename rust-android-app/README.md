# Rust Android App

A Rust library project for Android development, following incremental development principles.

## Prerequisites

This project requires the following Android compilation targets:

- `aarch64-linux-android` - 64-bit ARM (primary target for modern Android devices)
- `armv7-linux-androideabi` - 32-bit ARM (compatibility with older devices)
- `x86_64-linux-android` - 64-bit x86 (emulator and x86 devices)

## Setup

Install the required Android targets:

```bash
rustup target add aarch64-linux-android
rustup target add armv7-linux-androideabi
rustup target add x86_64-linux-android
```

## Development Guidelines

This project follows strict incremental development:

1. Make small, focused changes
2. Run `cargo check` to verify compilation
3. Run `cargo clippy` to check for improvements
4. Run `cargo test` where applicable
5. Commit each change with a descriptive message
6. **NO ONESHOTTING!** Each feature is built step by step

## Project Structure

```
rust-android-app/
├── Cargo.toml          # Project configuration
├── src/
│   └── lib.rs          # Main library entry point
└── README.md           # This file
```

## Progress

- [x] Initialize Rust library project
- [x] Add Android compilation targets
- [ ] Configure library as C dynamic library for Android
- [ ] Add core Android dependencies
- [ ] Create basic JNI entry point