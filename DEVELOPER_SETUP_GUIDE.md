# Chrome-Odyssey Developer Setup Guide

This guide helps developers set up the Chrome-Odyssey browser build environment and test the custom branding locally.

## Prerequisites

### System Requirements
- **macOS**: 14.0+ (tested on 14.6.0)
- **Xcode**: Latest version with command line tools
- **Disk Space**: ~100GB free space for full build
- **RAM**: 16GB+ recommended (32GB for optimal performance)
- **Architecture**: Intel x64 or Apple Silicon (ARM64)

### Required Tools
```bash
# Install Xcode command line tools
xcode-select --install

# Verify git is installed
git --version

# Python 3 should be available
python3 --version
```

## Quick Setup

### 1. Clone the Project
```bash
git clone https://github.com/your-username/Chrome-Source.git
cd Chrome-Source
```

### 2. Set Up Build Environment
```bash
# Create build directory structure
mkdir -p ~/chromium-build
cd ~/chromium-build

# Get depot_tools (Google's build tools)
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

# Add depot_tools to PATH
export PATH="$HOME/chromium-build/depot_tools:$PATH"
echo 'export PATH="$HOME/chromium-build/depot_tools:$PATH"' >> ~/.zshrc
```

### 3. Download Chromium Source
```bash
cd ~/chromium-build

# Configure gclient for Chromium
gclient config --name src https://chromium.googlesource.com/chromium/src.git

# Sync the source (this will take 30-60 minutes)
gclient sync
```

## Building Chrome-Odyssey

### 1. Apply Custom Branding
```bash
cd ~/chromium-build/chromium/src

# Apply branding patches from your project
git apply /path/to/Chrome-Source/patches/001-chrome-odyssey-branding.patch

# Copy branding file
cp /path/to/Chrome-Source/patches/chrome/app/theme/chromium/BRANDING \
   chrome/app/theme/chromium/BRANDING
```

### 2. Configure Build
```bash
# Generate build configuration
gn gen ~/chromium-build/build-output/Release --args='
  is_debug=false
  symbol_level=1
  is_component_build=false
'
```

### 3. Build the Browser
```bash
# Start the build (this takes 1-3 hours depending on hardware)
ninja -C ~/chromium-build/build-output/Release chrome
```

### 4. Launch Chrome-Odyssey
```bash
# Open the built browser
open ~/chromium-build/build-output/Release/Chrome-Odyssey.app
```

## Verification Steps

### 1. Check Application Bundle
```bash
ls -la ~/chromium-build/build-output/Release/Chrome-Odyssey*.app
```

Expected output:
```
Chrome-Odyssey Helper (Alerts).app/
Chrome-Odyssey Helper (GPU).app/
Chrome-Odyssey Helper (Plugin).app/
Chrome-Odyssey Helper (Renderer).app/
Chrome-Odyssey Helper.app/
Chrome-Odyssey.app/
```

### 2. Verify Branding in Browser
1. Launch Chrome-Odyssey.app
2. Check **About** menu - should show "Chrome-Odyssey"
3. Verify window title shows "Chrome-Odyssey" 
4. Check Activity Monitor - processes should show "Chrome-Odyssey" names

### 3. Bundle Identifier Check
```bash
# Check bundle ID
mdls -name kMDItemCFBundleIdentifier ~/chromium-build/build-output/Release/Chrome-Odyssey.app

# Should output: kMDItemCFBundleIdentifier = "com.odysseyrm.chrome-odyssey"
```

## Development Workflow

### Making Changes
1. Edit source files in `~/chromium-build/chromium/src/`
2. Run incremental build: `ninja -C ~/chromium-build/build-output/Release chrome`
3. Test changes in Chrome-Odyssey.app

### Creating New Patches
```bash
cd ~/chromium-build/chromium/src
git add .
git diff --cached > /path/to/Chrome-Source/patches/new-feature.patch
```

### Updating Chromium Base
```bash
cd ~/chromium-build
gclient sync
# May need to resolve conflicts with custom patches
```

## Troubleshooting

### Build Errors
- **Disk space**: Ensure 100GB+ free space
- **Memory**: Close other applications during build
- **Xcode issues**: Run `sudo xcode-select -r` to reset

### Patch Application Issues
```bash
# Check what files would be affected
git apply --check /path/to/patch.patch

# Apply with 3-way merge for conflicts
git apply --3way /path/to/patch.patch
```

### Performance Optimization
```bash
# Use more parallel jobs (adjust based on CPU cores)
ninja -j16 -C ~/chromium-build/build-output/Release chrome

# For faster incremental builds
gn gen ~/chromium-build/build-output/Debug --args='is_debug=true'
```

## Environment Isolation

This setup keeps Chrome-Odyssey completely isolated:
- **Build files**: `~/chromium-build/`
- **No system changes**: Uses local depot_tools
- **Separate bundle**: Won't conflict with regular Chrome
- **Clean removal**: Delete `~/chromium-build/` to remove everything

## Support

For issues:
1. Check build logs in `~/chromium-build/build-output/Release/build.ninja`
2. Verify system requirements are met
3. Try clean rebuild: `rm -rf ~/chromium-build/build-output && gn gen ...`

---
**Last Updated**: 2025-08-24  
**Tested Environments**: macOS 14.6.0 (Intel & ARM64)