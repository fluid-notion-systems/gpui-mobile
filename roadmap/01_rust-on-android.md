# Roadmap: Building a Rust Android App - Step by Step

## Overview

This roadmap outlines the incremental development process for creating a Rust Android application. Each step follows the principle of small, tested, committed changes.

## Development Guidelines

**For every step:**
1. Make a small, focused change
2. Run `cargo check` to verify compilation
3. Run `cargo clippy` to check for improvements
4. Run `cargo test` where applicable
5. Commit the change with a descriptive message
6. **NO ONESHOTTING!** Each feature is built incrementally

## Phase 1: Foundation Setup

### Step 1.1: Initialize Rust Project
```bash
cargo new --lib rust-android-app
cd rust-android-app
```
- **Commit**: `Initial Rust library project setup`

### Step 1.2: Add Android Targets
```bash
rustup target add aarch64-linux-android
rustup target add armv7-linux-androideabi
rustup target add x86_64-linux-android
```
- **Commit**: `Add Android compilation targets`

### Step 1.3: Basic Cargo.toml Configuration
```toml
[package]
name = "rust-android-app"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]
```
- Run: `cargo check`
- **Commit**: `Configure library as C dynamic library for Android`

### Step 1.4: Add Android Dependencies
```toml
[dependencies]
jni = "0.21"
android_logger = "0.13"
log = "0.4"
```
- Run: `cargo check`
- Run: `cargo clippy`
- **Commit**: `Add core Android dependencies`

### Step 1.5: Create Basic JNI Entry Point
```rust
// src/lib.rs
use jni::JNIEnv;
use jni::objects::JClass;

#[no_mangle]
pub extern "system" fn Java_com_example_rustapp_NativeLib_hello(
    _env: JNIEnv,
    _class: JClass,
) {
    android_logger::init_once(
        android_logger::Config::default()
            .with_max_level(log::LevelFilter::Debug),
    );
    log::info!("Hello from Rust!");
}
```
- Run: `cargo check`
- Run: `cargo clippy`
- **Commit**: `Add basic JNI function with logging`

## Phase 2: Build System Integration

### Step 2.1: Install cargo-ndk
```bash
cargo install cargo-ndk
```
- **Commit**: `Document cargo-ndk requirement in README`

### Step 2.2: Create Build Script
```bash
#!/bin/bash
# build-android.sh
cargo ndk -t aarch64-linux-android build --release
```
- Make executable: `chmod +x build-android.sh`
- **Commit**: `Add Android build script`

### Step 2.3: Add More JNI Functions
```rust
#[no_mangle]
pub extern "system" fn Java_com_example_rustapp_NativeLib_add(
    _env: JNIEnv,
    _class: JClass,
    a: i32,
    b: i32,
) -> i32 {
    a + b
}
```
- Run: `cargo check`
- Run: `cargo clippy`
- **Commit**: `Add simple arithmetic JNI function`

### Step 2.4: Add String Handling
```rust
use jni::objects::JString;
use jni::sys::jstring;

#[no_mangle]
pub extern "system" fn Java_com_example_rustapp_NativeLib_greet(
    env: JNIEnv,
    _class: JClass,
    name: JString,
) -> jstring {
    let input: String = env
        .get_string(name)
        .expect("Couldn't get java string!")
        .into();
    
    let output = format!("Hello, {}!", input);
    
    env.new_string(output)
        .expect("Couldn't create java string!")
        .into_inner()
}
```
- Run: `cargo check`
- Run: `cargo clippy`
- **Commit**: `Add JNI string handling function`

## Phase 3: Android Project Setup

### Step 3.1: Create Android Directory Structure
```
android/
├── app/
│   ├── src/
│   │   └── main/
│   │       ├── java/
│   │       ├── AndroidManifest.xml
│   │       └── jniLibs/
│   └── build.gradle
├── build.gradle
└── settings.gradle
```
- **Commit**: `Add Android project directory structure`

### Step 3.2: Create Basic Android Manifest
```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.rustapp">
    
    <application
        android:label="Rust Android App"
        android:theme="@style/Theme.AppCompat">
        
        <activity android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```
- **Commit**: `Add Android manifest file`

### Step 3.3: Create Native Library Wrapper
```java
// android/app/src/main/java/com/example/rustapp/NativeLib.java
package com.example.rustapp;

public class NativeLib {
    static {
        System.loadLibrary("rust_android_app");
    }
    
    public static native void hello();
    public static native int add(int a, int b);
    public static native String greet(String name);
}
```
- **Commit**: `Add Java wrapper for native Rust library`

### Step 3.4: Create Basic MainActivity
```java
// android/app/src/main/java/com/example/rustapp/MainActivity.java
package com.example.rustapp;

import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        TextView tv = new TextView(this);
        tv.setText("2 + 3 = " + NativeLib.add(2, 3));
        setContentView(tv);
        
        NativeLib.hello(); // This will log
    }
}
```
- **Commit**: `Add basic Android activity`

## Phase 4: Graphics Foundation

### Step 4.1: Add Graphics Dependencies
```toml
[dependencies]
raw-window-handle = "0.5"
```
- Run: `cargo check`
- **Commit**: `Add raw-window-handle dependency`

### Step 4.2: Add Android NDK Dependencies
```toml
[target.'cfg(target_os = "android")'.dependencies]
ndk = "0.8"
ndk-sys = "0.5"
ndk-context = "0.1"
```
- Run: `cargo check`
- **Commit**: `Add Android NDK dependencies`

### Step 4.3: Create Window Handle Function
```rust
#[cfg(target_os = "android")]
use ndk::native_window::NativeWindow;

#[no_mangle]
pub extern "system" fn Java_com_example_rustapp_NativeLib_setSurface(
    env: JNIEnv,
    _class: JClass,
    surface: jobject,
) {
    // Store surface for later use
    log::info!("Surface received from Java");
}
```
- Run: `cargo check`
- Run: `cargo clippy`
- **Commit**: `Add surface handling function`

## Phase 5: Blade Integration Preparation

### Step 5.1: Add Blade as Dependency
```toml
[dependencies]
blade-graphics = { path = "../vendor/blade/blade-graphics" }
```
- Run: `cargo check`
- **Commit**: `Add Blade graphics dependency`

### Step 5.2: Create Blade Context Holder
```rust
use std::sync::Mutex;

static BLADE_CONTEXT: Mutex<Option<blade_graphics::Context>> = Mutex::new(None);
```
- Run: `cargo check`
- Run: `cargo clippy`
- **Commit**: `Add global Blade context holder`

### Step 5.3: Initialize Blade Function
```rust
#[no_mangle]
pub extern "system" fn Java_com_example_rustapp_NativeLib_initBlade(
    _env: JNIEnv,
    _class: JClass,
) -> bool {
    // Placeholder for Blade initialization
    log::info!("Initializing Blade graphics");
    true
}
```
- Run: `cargo check`
- **Commit**: `Add Blade initialization placeholder`

## Phase 6: Testing Infrastructure

### Step 6.1: Add Test Module
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_arithmetic() {
        // We can't test JNI functions directly, but we can test logic
        assert_eq!(2 + 3, 5);
    }
}
```
- Run: `cargo test`
- **Commit**: `Add basic test module`

### Step 6.2: Add Integration Test Structure
```
tests/
└── integration_test.rs
```
- **Commit**: `Add integration test directory`

### Step 6.3: Create Mock JNI Environment
```rust
// tests/common/mod.rs
pub fn setup() {
    android_logger::init_once(
        android_logger::Config::default()
            .with_max_level(log::LevelFilter::Debug),
    );
}
```
- Run: `cargo test`
- **Commit**: `Add test utilities module`

## Phase 7: Continuous Integration

### Step 7.1: Create GitHub Actions Workflow
```yaml
# .github/workflows/android.yml
name: Android CI

on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Install Rust
      uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
        target: aarch64-linux-android
    - name: Check
      run: cargo check --target aarch64-linux-android
```
- **Commit**: `Add GitHub Actions workflow for Android`

### Step 7.2: Add Clippy to CI
```yaml
    - name: Clippy
      run: cargo clippy --target aarch64-linux-android -- -D warnings
```
- **Commit**: `Add Clippy to CI pipeline`

## Progress Tracking

- [ ] Phase 1: Foundation Setup
  - [ ] Initialize project
  - [ ] Add targets
  - [ ] Configure Cargo.toml
  - [ ] Add dependencies
  - [ ] Create JNI entry point

- [ ] Phase 2: Build System
  - [ ] Install cargo-ndk
  - [ ] Create build script
  - [ ] Add JNI functions
  - [ ] String handling

- [ ] Phase 3: Android Project
  - [ ] Directory structure
  - [ ] Manifest
  - [ ] Native wrapper
  - [ ] MainActivity

- [ ] Phase 4: Graphics Foundation
  - [ ] Graphics dependencies
  - [ ] NDK dependencies
  - [ ] Window handling

- [ ] Phase 5: Blade Integration
  - [ ] Add Blade dependency
  - [ ] Context management
  - [ ] Initialization

- [ ] Phase 6: Testing
  - [ ] Unit tests
  - [ ] Integration tests
  - [ ] Test utilities

- [ ] Phase 7: CI/CD
  - [ ] GitHub Actions
  - [ ] Clippy integration

## Key Principles

1. **Small Commits**: Each logical change is a separate commit
2. **Always Check**: Run `cargo check` after every change
3. **Lint Everything**: Run `cargo clippy` before committing
4. **Test When Possible**: Write tests for testable logic
5. **Document Changes**: Clear commit messages
6. **No Big Bang**: Build features incrementally

## Next Steps

After completing this roadmap:
1. Implement actual Blade rendering
2. Add touch input handling
3. Create example scenes
4. Performance optimization
5. Release build configuration