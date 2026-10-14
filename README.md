# DexDumper - Memory-based DEX Extraction Library

---

## 📖 Overview
a
DexDumper is an advanced Android library that performs runtime memory analysis to detect and extract DEX files from within applications. Unlike traditional methods that require root privileges or external tools, DexDumper operates entirely within the application's own process space.

## ✨ Main Features

- **🕵️‍♂️ Non-Root Operation** - Works on standard Android devices without root access
- **🔒 Self-Contained** - Pure C implementation with no external dependencies
- **🎯 Smart Memory Scanning** - Intelligent region filtering and DEX signature detection
- **🛡️ Safe Memory Access** - Signal-handled memory reading prevents crashes
- **📊 Duplicate Prevention** - SHA1 checksum and inode-based duplicate detection
- **🧹 Exclusion Control** - SHA1-based exclusion list to skip unwanted DEX files
- **⚙️ Runtime Configuration** - Dynamic configuration without recompiling via external config files

## 💡 Why Choose DexDumper?

### 🚀 Special Advantages

**🔄 Complete Isolation Operation**
DexDumper is specifically designed for non-root devices.
If this library is implemented in a sandbox or virtual machine, it can dump the dex files of official apps without breaking their integrity. The process is simple:

- Implement the library in a sandbox or virtual machine.
- Clone and run the official/test apps inside the sandbox or virtual machine.
- The dex files of the official/test apps will be dumped and saved.

**🔧 Dynamic Runtime Configuration**
DexDumper features an advanced runtime configuration system that allows you to customize behavior without recompiling:

- Automatically detects and pairs configuration files with library names.
- If no config exists, DexDumper creates a detailed, commented configuration file.

*How it works:* The library automatically extracts its own filename (e.g., `libdexdumper.so` → `dexdumper.conf`) and searches for matching configuration files. This ensures that even when you rename the library for stealth, the configuration system remains functional.

## 🛠️ Build & Installation

### Prerequisites

- Android NDK (for native compilation)
- Termux app (for on-device building)

> 💡 Tip:
> If you encounter issues with NDK builds or Termux setup, you can use the GitHub Actions workflow to auto-compile the source — no manual setup needed, handles dependencies, and builds for all architectures.

### Build Instructions

#### Using NDK Build

```bash
# Clone the repository
git clone https://github.com/muhammadrizwan87/dexdumper.git
cd dexdumper

# Set NDK path (adjust according to your setup)
export NDK_HOME=/path/to/your/ndk
# export NDK_HOME=/data/data/com.termux/files/home/android-sdk/ndk/24.0.8215888

# Build for all architectures
$NDK_HOME/ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=./jni/Application.mk

# Output will be in libs/ directory
ls libs/
# armeabi-v7a/ arm64-v8a/ x86/ x86_64/
```

### Installation & Usage

Ensure that the native library is loaded within the class initializer of the application’s main entry-point class.

#### Option A: Integration in Android App

**1. Add to your project:**
```java
public class MyApp extends Application {
    static {
        System.loadLibrary("dexdumper");
    }

    @Override
    public void onCreate() {
        super.onCreate();
        // Dumping starts automatically when library loads
    }
}
```

**2. Update build.gradle:**
```gradle
android {
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

#### Option B: Patch on Android App

**1. Smali code to load library:**
```smali
.method static constructor <clinit>()V
    .registers 1
    
    const-string v0, "dexdumper"
    
    invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V

    return-void
.end method
```

**2. Add library in android app:**

- lib/armeabi-v7a/libdexdumper.so
- lib/arm64-v8a/libdexdumper.so
- lib/x86/libdexdumper.so
- lib/x86_64/libdexdumper.so

## 📁 Output Locations

DexDumper will automatically try these directories in order. You can change the order if you want, or add a custom directory.

1. `/data/data/[PACKAGE]/files/dex_dump/` (Primary)
2. `/data/user/0/[PACKAGE]/files/dex_dump/` (Multi-user)
3. `/storage/emulated/0/Android/data/[PACKAGE]/files/dex_dump/` (External)
4. `/sdcard/Android/data/[PACKAGE]/files/dex_dump/` (Legacy external)

## 🔧 Configuration

### Build-time Configuration (config.h)

```c
// Enable/disable region filtering
// Effect of disabling: Scans ALL memory regions including system areas
#define ENABLE_REGION_FILTERING 1 // Enabled

// Enable/disable second scan after initial dump
#define ENABLE_SECOND_SCAN 0 // Disabled by default

// Timing configuration (in seconds)
#define THREAD_INITIAL_DELAY 8   // Delay before first scan
#define SECOND_SCAN_DELAY 7      // Delay before second scan (if enabled)

// Output directory templates (order of preference)
#define OUTPUT_DIRECTORY_TEMPLATES { \
    "/data/data/%s/files/dex_dump", \
    "/data/user/0/%s/files/dex_dump", \
    "/storage/emulated/0/Android/data/%s/files/dex_dump", \
    "/sdcard/Android/data/%s/files/dex_dump" \
}

// SHA1 exclusion list
// Any DEX with matching SHA1 will be skipped
#define EXCLUDED_SHA1_LIST { \
    "da39a3ee5e6b4b0d3255bfef95601890afd80709", /* Empty file SHA1 */ \
    /* Add your excluded SHA1 hashes here */ \
}

// DEX file size limits
#define DEX_MIN_FILE_SIZE 1024
#define DEX_MAX_FILE_SIZE (50 * 1024 * 1024)

// Memory scanning limits  
#define DEFAULT_SCAN_LIMIT (2 * 1024 * 1024)
#define MAX_REGION_SIZE (200 * 1024 * 1024)
```

### Runtime Configuration ([LIBRARY_NAME].conf)

DexDumper automatically creates and reads configuration files at runtime. No recompilation needed!

**Configuration File Locations:**
- `/data/data/[PACKAGE]/files/[LIBRARY_NAME].conf`
- `/data/user/0/[PACKAGE]/files/[LIBRARY_NAME].conf` 
- `/storage/emulated/0/Android/data/[PACKAGE]/files/[LIBRARY_NAME].conf`

**First Run Behavior:**
- Automatically generates a detailed configuration file with explanations
- Uses compile-time defaults until you customize the file  
- File includes comprehensive comments for every setting

**To customize:** Edit the generated `.conf` file, save, and restart the app. The library will use your new settings immediately.

## 📊 Performance Considerations

- **Memory Usage**: Minimal impact (typically < 10MB)
- **CPU Usage**: Single background thread with yield operations
- **Storage**: Automatic cleanup of output directory
- **Battery**: Short-lived operation with sleep intervals

## 🛡️ Security & Privacy

### What DexDumper DOES NOT Do:
- ❌ No network communication
- ❌ No data exfiltration
- ❌ No root escalation attempts
- ❌ No app modification
- ❌ No permanent system changes

### Safety Features:
- ✅ Signal-protected memory access
- ✅ Bounds checking on all operations
- ✅ Safe directory traversal
- ✅ Resource cleanup on completion
- ✅ No persistent background services

## 🤝 Contributing

We welcome contributions! Please see our contributing guidelines:

1. Fork the repository
2. Create a feature branch
3. Follow the code style and add comments
4. Test thoroughly on multiple architectures
5. Submit a pull request

## 🔮 TODO / Roadmap

### 🎯 Short Term Goals
- [ ] **Lite Version** - Optimized for low memory and battery usage
- [ ] **Enhanced Stealth** - Improved anti-analysis techniques
- [ ] **Performance Metrics** - Scanning performance optimization

### 🚀 Long Term Vision
- [ ] **Root Version** - Full system memory scanning capabilities
- [ ] **Encrypted DEX Support** - Brute force to runtime decrypt and dump
- [ ] **GUI Interface** - User-friendly analysis dashboard with thread start/stop controls, support for standard and deep scanning modes, and multi-scan capabilities
- [ ] **Crash Prevention Plugin** - Helper module to prevent app crashes or premature exits, ensuring dex files can load and be dumped
- [ ] **SO Dumper (Idea Phase)** - Concept for dumping native so (shared object) libraries, planned for future exploration

**⭐ If you find this project useful, please consider giving it a star!**

---

## 📄 License

This project is licensed under the **MIT License**. You can view the license details in the [LICENSE](https://github.com/muhammadrizwan87/dexdumper/blob/main/LICENSE) file.

*Disclaimer: This library is intended for educational purposes, security research, and legitimate reverse engineering activities. Users are responsible for complying with applicable laws and terms of service.*

---

## Author

**MuhammadRizwan**


- **Telegram Channel**: [TDOhex](https://TDOhex.t.me)
- **Second Channel**: [Android Patches](https://Android_Patches.t.me)
- **Discussion Group**: [Discussion of TDOhex](https://TDOhex_Discussion.t.me)
- **GitHub**: [MuhammadRizwan87](https://github.com/MuhammadRizwan87)

---
