# Chrome-Odyssey Build and Test Guide

This guide provides step-by-step instructions for building and testing the Chrome-Odyssey browser locally.

## Quick Start Commands

### Prerequisites Check
```bash
# Verify system requirements
system_profiler SPSoftwareDataType | grep "System Version"
xcode-select -p
git --version
python3 --version
```

### One-Line Setup
```bash
# Complete setup (run each line separately)
mkdir -p ~/chromium-build && cd ~/chromium-build
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$HOME/chromium-build/depot_tools:$PATH"
gclient config --name src https://chromium.googlesource.com/chromium/src.git
gclient sync
```

## Build Process

### Step 1: Apply Chrome-Odyssey Branding
```bash
cd ~/chromium-build/chromium/src

# Clone your patches (replace with actual repo URL)
git clone https://github.com/your-username/Chrome-Source.git /tmp/chrome-source

# Apply branding patch
git apply /tmp/chrome-source/patches/001-chrome-odyssey-branding.patch

# Copy BRANDING file
cp /tmp/chrome-source/patches/chrome/app/theme/chromium/BRANDING \
   chrome/app/theme/chromium/BRANDING

# Verify branding was applied
grep -r "Chrome-Odyssey" chrome/app/chromium_strings.grd
cat chrome/app/theme/chromium/BRANDING
```

### Step 2: Configure Build
```bash
# Generate Release build configuration
gn gen ~/chromium-build/build-output/Release --args='
  is_debug=false
  symbol_level=1
  is_component_build=false
  is_official_build=false
'

# For faster Debug builds (optional)
gn gen ~/chromium-build/build-output/Debug --args='
  is_debug=true
  symbol_level=2
'
```

### Step 3: Build Chrome-Odyssey
```bash
# Start the build (Release version)
ninja -C ~/chromium-build/build-output/Release chrome

# Monitor build progress
tail -f ~/chromium-build/build-output/Release/.ninja_log
```

Expected build time:
- **First build**: 1-3 hours (depending on hardware)
- **Incremental builds**: 5-30 minutes
- **Clean builds**: 45-90 minutes

## Testing Procedures

### 1. Basic Functionality Tests

#### Launch Test
```bash
# Launch the browser
open ~/chromium-build/build-output/Release/Chrome-Odyssey.app

# Verify it launches without errors
# Check Console.app for any crash logs
```

#### Process Verification
```bash
# Check running processes
ps aux | grep "Chrome-Odyssey"

# Should show:
# Chrome-Odyssey
# Chrome-Odyssey Helper
# Chrome-Odyssey Helper (GPU)
# Chrome-Odyssey Helper (Renderer)
```

### 2. Branding Verification Tests

#### Visual Branding Check
1. **Application Name**: 
   - Dock should show "Chrome-Odyssey"
   - Activity Monitor should list "Chrome-Odyssey" processes

2. **Menu Bar**:
   - Menu should show "Chrome-Odyssey" → About Chrome-Odyssey
   - About dialog should display "Chrome-Odyssey"

3. **Window Title**:
   - New tab should show "New Tab - Chrome-Odyssey"
   - Websites should show "Page Title - Chrome-Odyssey"

#### Bundle Identifier Test
```bash
# Check bundle identifier
mdls -name kMDItemCFBundleIdentifier ~/chromium-build/build-output/Release/Chrome-Odyssey.app

# Expected: kMDItemCFBundleIdentifier = "com.odysseyrm.chrome-odyssey"
```

#### Helper Apps Test
```bash
# List all helper applications
find ~/chromium-build/build-output/Release -name "*Chrome-Odyssey*.app" -type d

# Should list:
# Chrome-Odyssey.app
# Chrome-Odyssey Helper.app  
# Chrome-Odyssey Helper (GPU).app
# Chrome-Odyssey Helper (Renderer).app
# Chrome-Odyssey Helper (Plugin).app
# Chrome-Odyssey Helper (Alerts).app
```

### 3. Functionality Tests

#### Basic Browser Features
1. **Navigation**: Navigate to https://google.com
2. **Tabs**: Open multiple tabs, close tabs
3. **Bookmarks**: Add/remove bookmarks
4. **Settings**: Access chrome-odyssey://settings
5. **Extensions**: Test extension loading
6. **Developer Tools**: Press F12 or Cmd+Option+I

#### Core Feature Tests
```bash
# Test internal pages
open ~/chromium-build/build-output/Release/Chrome-Odyssey.app --args \
  --new-window chrome-odyssey://settings

# Test with specific profile
open ~/chromium-build/build-output/Release/Chrome-Odyssey.app --args \
  --user-data-dir="/tmp/chrome-odyssey-test-profile"
```

### 4. Performance Tests

#### Memory Usage
```bash
# Monitor memory usage
top -pid $(pgrep "Chrome-Odyssey")

# Check for memory leaks during extended use
```

#### Startup Time
```bash
# Measure startup time
time open ~/chromium-build/build-output/Release/Chrome-Odyssey.app
```

## Automated Testing

### Running Unit Tests
```bash
cd ~/chromium-build/chromium/src

# Run base unit tests
ninja -C ~/chromium-build/build-output/Release base_unittests
~/chromium-build/build-output/Release/base_unittests

# Run browser tests (subset)
ninja -C ~/chromium-build/build-output/Release browser_tests
~/chromium-build/build-output/Release/browser_tests --gtest_filter="*Branding*"
```

### Integration Tests
```bash
# Run with test user profile
mkdir -p /tmp/chrome-odyssey-test
~/chromium-build/build-output/Release/Chrome-Odyssey.app/Contents/MacOS/Chrome-Odyssey \
  --user-data-dir="/tmp/chrome-odyssey-test" \
  --no-first-run \
  --disable-default-apps
```

## Regression Testing

### Before Code Changes
1. Document current behavior
2. Create test user profile
3. Take screenshots of branding elements
4. Record performance baselines

### After Code Changes
```bash
# Incremental build
ninja -C ~/chromium-build/build-output/Release chrome

# Re-run all tests above
# Compare with baseline behavior
```

## Build Verification Checklist

### ✅ Pre-Build
- [ ] Chromium source downloaded
- [ ] Patches apply cleanly
- [ ] BRANDING file copied
- [ ] Build configuration generated

### ✅ Build Process
- [ ] No compilation errors
- [ ] All 54,404+ targets built successfully
- [ ] Chrome-Odyssey.app created
- [ ] All helper apps present

### ✅ Post-Build Testing  
- [ ] Application launches
- [ ] Correct branding displayed
- [ ] Bundle ID is correct
- [ ] Helper processes named correctly
- [ ] Basic browsing works
- [ ] Settings accessible
- [ ] No crash logs in Console

### ✅ Sign-off
- [ ] All tests passed
- [ ] Performance acceptable
- [ ] No regressions identified
- [ ] Ready for distribution

## Troubleshooting Build Issues

### Common Build Errors

#### Disk Space
```bash
# Check available space
df -h ~/chromium-build

# Clean up if needed
rm -rf ~/chromium-build/chromium/src/out/Release
```

#### Xcode Issues
```bash
# Reset Xcode command line tools
sudo xcode-select --reset
sudo xcode-select --install
```

#### Patch Conflicts
```bash
# Check patch status
cd ~/chromium-build/chromium/src
git status

# Reset to clean state if needed
git reset --hard HEAD
git clean -fd

# Re-apply patches
git apply /tmp/chrome-source/patches/001-chrome-odyssey-branding.patch
```

### Performance Issues

#### Slow Build
```bash
# Use more parallel jobs (adjust for your CPU)
ninja -j8 -C ~/chromium-build/build-output/Release chrome

# Use ccache for faster rebuilds
export CCACHE_DIR=~/chromium-build/.ccache
```

#### Runtime Performance
```bash
# Build with optimizations
gn gen ~/chromium-build/build-output/Release --args='
  is_debug=false
  is_official_build=true
  symbol_level=0
'
```

## Continuous Integration Setup

### GitHub Actions (Example)
```yaml
name: Chrome-Odyssey Build
on: [push, pull_request]
jobs:
  build:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Chromium build
        run: |
          mkdir -p ~/chromium-build
          # ... setup steps
      - name: Build Chrome-Odyssey
        run: ninja -C ~/chromium-build/build-output/Release chrome
      - name: Test branding
        run: |
          # Run automated tests
```

---
**Build Environment**: macOS 14.6+, Xcode 15+, 100GB+ disk space
**Build Time**: 1-3 hours (first build), 5-30 minutes (incremental)
**Last Updated**: 2025-08-24