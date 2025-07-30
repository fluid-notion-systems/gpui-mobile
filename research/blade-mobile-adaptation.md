# Blade Mobile Adaptation: Technical Requirements and Implementation Strategy

## Table of Contents
1. [Overview](#overview)
2. [Current Blade Architecture Analysis](#current-blade-architecture-analysis)
3. [Mobile Platform Requirements](#mobile-platform-requirements)
4. [Implementation Roadmap](#implementation-roadmap)
5. [Code Examples](#code-examples)
6. [Testing Strategy](#testing-strategy)
7. [Performance Optimization](#performance-optimization)
8. [Platform-Specific Considerations](#platform-specific-considerations)

## Overview

This document outlines the technical requirements and implementation strategy for adapting Blade graphics library to mobile platforms, with primary focus on Android and secondary consideration for iOS.

## Current Blade Architecture Analysis

### Modular Structure
Blade's architecture is already well-suited for mobile adaptation:

```
blade/
├── blade-graphics/     # Core graphics abstraction
├── blade-render/       # High-level rendering
├── blade-asset/        # Asset management
├── blade-egui/         # egui integration
├── blade-macros/       # Procedural macros
└── blade-util/         # Utilities
```

### Graphics Backend Support

| Platform | Backend | Status | Mobile Ready |
|----------|---------|--------|--------------|
| Linux    | Vulkan  | ✅ Implemented | ✅ Yes |
| Windows  | Vulkan  | ✅ Implemented | N/A |
| macOS    | Metal   | ✅ Implemented | ✅ Yes* |
| Android  | Vulkan  | ⚠️ Mentioned | 🔧 Needs work |
| iOS      | Metal   | ⚠️ Possible | 🔧 Needs work |
| Web      | WebGL   | ✅ Implemented | ✅ Yes |

*Metal on macOS shares codebase with iOS

### Key Observations

1. **Vulkan Support**: Already implemented, needs Android surface integration
2. **Metal Support**: Already implemented, needs iOS-specific adjustments
3. **No Desktop Dependencies**: Core graphics layer is platform-agnostic
4. **Compute Shaders**: Supported on Vulkan/Metal, important for mobile GPU compute

## Mobile Platform Requirements

### Android Requirements

#### Minimum Specifications
- **API Level**: 24 (Android 7.0) for Vulkan support
- **Vulkan Version**: 1.0 minimum, 1.1 preferred
- **Architecture**: ARM64 primary, ARMv7 for compatibility

#### Required Extensions
```rust
// Already required by Blade - all available on Android
const REQUIRED_INSTANCE_EXTENSIONS: &[&str] = &[
    "VK_KHR_surface",
    "VK_KHR_android_surface",  // Additional for Android
    "VK_EXT_debug_utils",
    "VK_KHR_get_physical_device_properties2",
];
```

#### Surface Creation
```rust
pub struct AndroidSurface {
    window: *mut ANativeWindow,
    surface: vk::SurfaceKHR,
}

impl AndroidSurface {
    pub unsafe fn new(
        instance: &ash::Instance,
        window: *mut ANativeWindow,
    ) -> Result<Self, SurfaceError> {
        let create_info = vk::AndroidSurfaceCreateInfoKHR {
            s_type: vk::StructureType::ANDROID_SURFACE_CREATE_INFO_KHR,
            p_next: ptr::null(),
            flags: vk::AndroidSurfaceCreateFlagsKHR::empty(),
            window,
        };
        
        let surface = instance
            .create_android_surface_khr(&create_info, None)?;
        
        Ok(AndroidSurface { window, surface })
    }
}
```

### iOS Requirements

#### Minimum Specifications
- **iOS Version**: 11.0 for Metal 2
- **Architecture**: ARM64 only
- **Metal Feature Set**: iOS GPU Family 3 minimum

#### Metal Layer Setup
```swift
// Objective-C bridging required
@interface BladeiOSView : UIView
@property (nonatomic, readonly) CAMetalLayer *metalLayer;
@end

@implementation BladeiOSView
+ (Class)layerClass {
    return [CAMetalLayer class];
}
- (CAMetalLayer *)metalLayer {
    return (CAMetalLayer *)self.layer;
}
@end
```

## Implementation Roadmap

### Phase 1: Android Foundation (Week 1-2)
1. **Setup Build System**
   - Configure cargo-ndk
   - Create Android.mk/CMakeLists.txt
   - Setup JNI bindings

2. **Surface Management**
   - Implement ANativeWindow handling
   - Create Vulkan surface
   - Handle lifecycle events

3. **Basic Example**
   - Triangle rendering
   - Touch input
   - Lifecycle handling

### Phase 2: iOS Foundation (Week 3-4)
1. **Setup Build System**
   - Configure cargo-lipo
   - Create Xcode project
   - Setup Objective-C bridging

2. **Metal Integration**
   - CAMetalLayer setup
   - Drawable management
   - Display link integration

3. **Basic Example**
   - Triangle rendering
   - Touch input
   - Lifecycle handling

### Phase 3: Feature Parity (Week 5-6)
1. **Asset Loading**
   - Android: APK asset manager
   - iOS: Bundle resources
   - Unified API

2. **Input System**
   - Touch events
   - Gesture recognition
   - Accelerometer/gyroscope

3. **Platform Integration**
   - Notifications
   - Background/foreground
   - Memory warnings

### Phase 4: Optimization (Week 7-8)
1. **Performance**
   - Mobile GPU profiling
   - Power optimization
   - Thermal management

2. **Platform Features**
   - HDR support (where available)
   - Variable refresh rate
   - Platform-specific extensions

## Code Examples

### Android Activity Integration

```rust
use android_activity::{AndroidApp, InputStatus, MainEvent, PollEvent};
use blade_graphics::{Context, Surface};

#[no_mangle]
fn android_main(app: AndroidApp) {
    android_logger::init_once(
        android_logger::Config::default()
            .with_min_level(log::Level::Debug),
    );

    let mut blade_context: Option<Context> = None;
    let mut surface: Option<Surface> = None;

    loop {
        match app.poll_events(Some(std::time::Duration::from_millis(16))) {
            PollEvent::Wake => {}
            PollEvent::Timeout => {}
            PollEvent::Main(event) => match event {
                MainEvent::InitWindow { .. } => {
                    let window = app.native_window();
                    if let Some(window) = window {
                        // Initialize Blade
                        blade_context = Some(Context::new_android(&window)?);
                        surface = Some(Surface::from_android_window(
                            &blade_context.as_ref().unwrap(),
                            &window
                        )?);
                    }
                }
                MainEvent::TerminateWindow { .. } => {
                    surface = None;
                }
                MainEvent::Destroy => break,
                _ => {}
            }
            PollEvent::Io(_) => {}
        }

        // Render frame
        if let (Some(ctx), Some(surf)) = (&blade_context, &surface) {
            render_frame(ctx, surf);
        }
    }
}
```

### iOS View Controller Integration

```rust
use objc::{msg_send, sel, sel_impl};
use objc::runtime::Object;
use blade_graphics::{Context, Surface};

#[no_mangle]
pub extern "C" fn blade_ios_init(view: *mut Object) -> *mut Context {
    let layer: *mut Object = unsafe {
        msg_send![view, layer]
    };
    
    let context = Context::new_metal(layer).expect("Failed to create Metal context");
    Box::into_raw(Box::new(context))
}

#[no_mangle]
pub extern "C" fn blade_ios_render(
    context: *mut Context,
    drawable: *mut Object,
) {
    let context = unsafe { &*context };
    let surface = Surface::from_metal_drawable(context, drawable);
    
    render_frame(context, &surface);
}
```

### Unified Mobile API

```rust
pub trait MobilePlatform {
    fn create_context(&self) -> Result<Context, ContextError>;
    fn create_surface(&self, context: &Context) -> Result<Surface, SurfaceError>;
    fn handle_lifecycle(&mut self, event: LifecycleEvent);
    fn get_display_info(&self) -> DisplayInfo;
}

pub enum LifecycleEvent {
    Resume,
    Pause,
    LowMemory,
    OrientationChange(Orientation),
}

pub struct DisplayInfo {
    pub size: (u32, u32),
    pub scale_factor: f32,
    pub hdr_capable: bool,
    pub refresh_rate: f32,
}
```

## Testing Strategy

### Device Testing Matrix

#### Android
| Device Type | GPU | Priority | Test Focus |
|-------------|-----|----------|------------|
| Pixel 6+ | Mali-G78 | High | Vulkan 1.1, HDR |
| Samsung S21+ | Mali/Adreno | High | Performance |
| OnePlus 9 | Adreno 660 | Medium | Thermals |
| Pixel 4a | Adreno 618 | High | Low-end |

#### iOS
| Device Type | GPU | Priority | Test Focus |
|-------------|-----|----------|------------|
| iPhone 14 Pro | A16 | High | ProMotion, HDR |
| iPhone 12 | A14 | High | Baseline |
| iPhone SE 3 | A15 | Medium | Small screen |
| iPad Pro M2 | M2 | Low | Tablet |

### Automated Testing

```yaml
# CI/CD Pipeline
android-test:
  runs-on: macos-latest
  steps:
    - uses: reactivecircus/android-emulator-runner@v2
      with:
        api-level: 29
        arch: x86_64
        script: cargo test --target x86_64-linux-android

ios-test:
  runs-on: macos-latest
  steps:
    - run: xcrun simctl boot "iPhone 14"
    - run: cargo test --target aarch64-apple-ios-sim
```

## Performance Optimization

### Mobile-Specific Optimizations

1. **Tile-Based Rendering**
   ```rust
   // Optimize for TBDR GPUs
   pub struct RenderPass {
       load_op: LoadOp::DontCare,  // Don't load previous frame
       store_op: StoreOp::Store,   // Only store what's needed
   }
   ```

2. **Bandwidth Reduction**
   - Use compressed textures (ETC2/ASTC)
   - Implement LOD system
   - Reduce overdraw

3. **Power Management**
   ```rust
   pub enum PowerProfile {
       HighPerformance,  // 60+ FPS
       Balanced,         // 30-60 FPS
       PowerSaving,      // 30 FPS cap
   }
   ```

### Profiling Tools Integration

```rust
#[cfg(target_os = "android")]
pub fn begin_gpu_trace(name: &str) {
    unsafe {
        android_trace_begin_section(name.as_ptr());
    }
}

#[cfg(target_os = "ios")]
pub fn begin_gpu_trace(name: &str) {
    unsafe {
        os_signpost_interval_begin(name);
    }
}
```

## Platform-Specific Considerations

### Android Considerations

1. **Vulkan Variants**
   - Standard Vulkan
   - Vulkan on ANGLE (for older devices)
   - Fallback to GLES 3.0

2. **Surface Handling**
   ```rust
   // Handle surface recreation
   fn on_surface_changed(&mut self, new_surface: ANativeWindow) {
       self.wait_idle();
       self.destroy_swapchain();
       self.surface = Surface::new(new_surface);
       self.create_swapchain();
   }
   ```

3. **Memory Pressure**
   ```rust
   extern "C" fn on_low_memory() {
       // Release non-essential resources
       RESOURCE_CACHE.clear_unused();
       TEXTURE_CACHE.compress();
   }
   ```

### iOS Considerations

1. **Metal Features**
   - Memoryless render targets
   - Tile shaders
   - Metal Performance Shaders

2. **Display Sync**
   ```swift
   displayLink = CADisplayLink(target: self, selector: #selector(render))
   displayLink.preferredFramesPerSecond = 120 // ProMotion
   ```

3. **Background Handling**
   ```rust
   fn application_did_enter_background() {
       self.gpu_tasks.cancel_non_essential();
       self.reduce_memory_footprint();
   }
   ```

## Next Steps

1. **Prototype Development**
   - Start with Android Vulkan
   - Basic triangle example
   - Touch input handling

2. **Community Feedback**
   - RFC for mobile API design
   - Performance benchmarks
   - Use case validation

3. **Documentation**
   - Mobile setup guide
   - Platform-specific notes
   - Performance guidelines

## Conclusion

Blade's architecture is well-positioned for mobile adaptation. The main challenges are platform integration rather than fundamental graphics API issues. With Vulkan and Metal already supported, the path to mobile is primarily about:

1. Surface/window management
2. Lifecycle handling
3. Platform-specific optimizations
4. Build system integration

The modular architecture allows for incremental implementation, starting with basic rendering and progressively adding platform features.