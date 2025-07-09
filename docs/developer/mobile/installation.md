# Instalation Guide

This guide provides comprehensive, step-by-step instructions for setting up and running the NextNonce mobile application on your local machine for development and testing purposes.

## Hardware Specifications

#### **Scenario 1: Android-only version of NextNonce  (Windows / Linux / macOS)**

This setup focuses purely on building the Android version of the **NextNonce** application using Android Studio.

- **Operating System:**
    
    - **Windows:** 64-bit Microsoft Windows 10 (version 20H2 or later) or Windows 11.
        
    - **Linux:** Any 64-bit Linux distribution that supports Gnome, KDE, or Unity desktop environments (e.g., Ubuntu 22.04 LTS or Fedora 39). GNU C Library (glibc) 2.31 or later.
        
    - **macOS:** macOS 12 (Monterey) or higher.
        
- **Processor (CPU):**
    
    - **Intel:** 8th generation Intel Core i5 or newer (e.g., i5-8xxx, i7-8xxx, i9-9xxx series or later) with at least 4 cores/8 threads. Crucially, **Intel VT-x and EPT** (Extended Page Tables) must be enabled in the BIOS/UEFI for efficient emulator performance. CPUs with 'H', 'HK', or 'HX' suffixes are often better in laptops for sustained performance.
        
    - **AMD:** AMD Zen 2 architecture (Ryzen 3000 series) or newer (e.g., Ryzen 5 3600, Ryzen 7 5800X, Ryzen 9 7900X) with at least 6 cores/12 threads. **AMD-V (SVM)** must be enabled in the BIOS/UEFI.
        
    - **Apple Silicon (macOS):** Apple M1 chip or newer (M2, M3, M4 series). 
        
    - **General:** A clock speed of 2.5 GHz or higher is beneficial. More cores and threads will significantly speed up Gradle builds.
        
- **RAM (Memory):**
        
    - **Minimum:** **16 GB RAM or more.** 
        
    - **Recommended (Good Experience):** **32+ GB RAM.**
        
    - **Type:** DDR4 (3200 MHz or faster) or DDR5 RAM is highly recommended.
        
- **Disk Space (Storage):**
    
    - **Minimum:** 20 GB of available free space (for IDE installation, Android SDK, and a few emulator images).
        
    - **Recommended:** **256 GB NVMe Solid-State Drive (SSD) with at least 50 GB free space.** An NVMe SSD drastically improves IDE startup times, project loading, build times, and emulator performance.
        
    - **Ideal:** **500 GB to 1 TB NVMe SSD.**.
        
- **Graphics (GPU):**
    
    - **Integrated Graphics (e.g., Intel Iris Xe, AMD Radeon Graphics, Apple integrated GPU):** Sufficient.
        
    - **Dedicated GPU (Optional but Beneficial):** A dedicated GPU with at least **4GB of VRAM** (e.g., NVIDIA GeForce GTX 1050/RTX 3050 equivalent or AMD Radeon RX 5600/RX 6600 equivalent).
        
- **Screen Resolution:**
    
    - **Minimum:** 1280 x 800 pixels.
        
    - **Recommended:** 1920 x 1080 (Full HD) or higher.
        Minimum

#### **Scenario 2: Android + iOS versions of NextNonce (macOS is Essential !!!)**

Developing for iOS necessitates a macOS environment due to Apple's ecosystem requirements (Xcode, iOS Simulator). Therefore, this scenario primarily focuses on **Mac hardware**.

- **Operating System:**
    
    - **macOS:** macOS 13 (Ventura) or **newer** is **highly** recommended for compatibility with the latest Xcode versions and iOS SDKs.
        
- **Processor (CPU):**
    
    - **Apple Silicon (M-series):** **Apple M1 Pro, M1 Max, M2 Pro, M2 Max, M3 Pro, M3 Max, or newer.** 
        
- **RAM (Unified Memory on Apple Silicon):**
    
    - **Minimum:** 16 GB Unified Memory. You will feel constraints with multiple simulators/emulators running simultaneously.
        
    - **Recommended (Good Experience):** **32+ GB Unified Memory.**
        
- **Disk Space (Storage):**
    
    - **Minimum:** 256 GB NVMe SSD with at least 50 GB free space. This will fill up _very quickly_ with Android SDKs, iOS SDKs, Xcode, Android Studio, and various project dependencies.
        
    - **Recommended:** A 500 GB NVMe SSD with at least 100 GB of free space provides a more comfortable working environment.
        
- **Screen Resolution:**
    
    - **Minimum:** 1440 x 900 pixels.
        
    - **Recommended:** A high-resolution Retina display (on MacBooks) or an external 4K monitor. Dual monitors are highly beneficial for simultaneously viewing Android Studio, Xcode, and their respective simulators/emulators.
        


## Software Prerequisites

#### **Scenario 1: Android-Only Development**

- **Android Studio:** The latest stable version of Android Studio.
    
- **Android SDK:** Installed via Android Studio's SDK Manager. Ensure you have:
    
    - The latest Android Platform SDK.
        
    - Android SDK Platform-Tools.
        
    - Android SDK Build-Tools.
    
- **Git:** For version control and cloning the [NextNonce repository](https://github.com/NextNonce/Mobile.git).
    

#### **Scenario 2: Android + iOS / Cross-Platform Development (Recommended)**

**One more time: A macOS machine is mandatory for iOS development.**
        
- **Homebrew (macOS):** Essential package manager for macOS to install various development tools.
    
- **Android Studio:** The latest stable version of Android Studio.
    
- **Android SDK:** Installed via Android Studio's SDK Manager (same components as Android-Only scenario).
    
- **Xcode:** The latest stable version of Xcode, downloaded from the Mac App Store. This includes the iOS SDK, compilers, and iOS Simulator.
    
- **Command Line Tools for Xcode:** Install these after Xcode via `xcode-select --install` in your terminal.
    
- **Git:** For version control and cloning the [NextNonce repository](https://github.com/NextNonce/Mobile.git).

### Configuration

1. **Clone the repository:**
    
```shell
git clone https://github.com/NextNonce/Mobile.git
cd Mobile
```
    
2. **Create `local.properties` file:** Create a `local.properties` file in the root directory of the project. This file is essential for providing API keys and environment-specific variables required by the app.
    
3. **Add Configuration Keys:** Copy the contents of `local.properties.example` and paste them into the new `local.properties` file. Your own values must be provided for the following properties:
    
```shell
# Supabase Credentials
SUPABASE_URL="YOUR_SUPABASE_URL"
SUPABASE_ANON_KEY="YOUR_SUPABASE_ANON_KEY"

# Google Auth for Supabase
GOOGLE_WEB_CLIENT_ID="YOUR_GOOGLE_WEB_CLIENT_ID"
```

#### On macOS (for Android & iOS)

Install **[Kotlin Multiplatform Plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)**

### Running the App

The process for running the application depends on your operating system.

#### On macOS (for Android & iOS)

1. **Check Environment:** The easiest way to ensure your environment is set up correctly is by using **KDoctor**. Open your terminal and run the following commands:
    
```shell
# Install KDoctor via Homebrew (if you don't have it)
brew install kdoctor

# Run KDoctor and check the output
kdoctor
```

KDoctor will analyze your system and provide instructions if any part of your setup (like Xcode, command-line tools, or Android Studio) needs attention. Address all issues it reports.
    
2. **Build in Android Studio:** Once KDoctor reports a clean environment, open the project in Android Studio. You can now run both platforms directly from the IDE:
    
    - For **iOS**, select the `iosApp` configuration and choose an available iOS Simulator or a connected physical device.
        
    - For **Android**, select the `composeApp` configuration and choose an Android emulator or a connected device.

#### On Windows / Linux (for Android only)

1. **Open Project:** Launch Android Studio and open the project folder.
    
2. **Build `composeApp`:** Android Studio will handle the download and setup of the necessary Android SDKs.
    
3. **Run:** Select the `composeApp` configuration and run it on an Android emulator or a connected physical device.


You have successfully set up, configured, and run the NextNonce mobile app on your local machine. Happy coding!