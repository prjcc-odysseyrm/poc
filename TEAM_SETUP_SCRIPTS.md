# Team Setup Scripts & Commands

## 🔧 AUTOMATED SETUP SCRIPTS

### Script 1: Full Developer Setup
**File**: `setup-full-dev.sh`
```bash
#!/bin/bash
set -e

echo "🚀 Setting up full Chromium development environment..."

# Verify prerequisites
echo "Checking prerequisites..."
if ! command -v git &> /dev/null; then
    echo "❌ Git not found. Please install Xcode Command Line Tools"
    exit 1
fi

if ! xcodebuild -version &> /dev/null; then
    echo "❌ Xcode not found. Please install Xcode from App Store"
    exit 1
fi

# Check disk space
AVAILABLE=$(df -g . | tail -1 | awk '{print $4}')
if [ "$AVAILABLE" -lt 100 ]; then
    echo "❌ Insufficient disk space. Need 100GB, have ${AVAILABLE}GB"
    exit 1
fi

echo "✅ Prerequisites checked"

# Create build directory
BUILD_DIR="$HOME/chromium-build"
mkdir -p "$BUILD_DIR"/{depot_tools,chromium,build-output}
cd "$BUILD_DIR"

# Install depot_tools
echo "📥 Installing depot_tools..."
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git depot_tools
export PATH="$BUILD_DIR/depot_tools:$PATH"

# Configure gclient
echo "⚙️ Configuring build environment..."
cd chromium
gclient config --name src https://chromium.googlesource.com/chromium/src.git

# Download source (background process)
echo "📦 Starting source download (this takes 2-4 hours)..."
echo "You can monitor progress with: du -sh $BUILD_DIR/chromium/"
nohup gclient sync --no-history --shallow > download.log 2>&1 &

echo "✅ Setup initiated. Check download.log for progress."
echo "Next: Run build-chromium.sh when download completes"
```

### Script 2: Build Chromium
**File**: `build-chromium.sh`  
```bash
#!/bin/bash
set -e

BUILD_DIR="$HOME/chromium-build"
export PATH="$BUILD_DIR/depot_tools:$PATH"

cd "$BUILD_DIR/chromium/src"

echo "🏗️ Configuring debug build..."
gn gen out/Debug --args='
    is_debug=true
    symbol_level=1
    enable_nacl=false
    target_cpu="arm64"
'

echo "🚀 Starting compilation (this takes 3-5 hours)..."
echo "Progress: ninja -C out/Debug chrome"
time ninja -C out/Debug chrome

if [ $? -eq 0 ]; then
    echo "✅ Build successful!"
    echo "Launch: open out/Debug/Chromium.app"
else
    echo "❌ Build failed. Check output above."
    exit 1
fi
```

### Script 3: Source-Only Setup
**File**: `setup-source-only.sh`
```bash
#!/bin/bash
set -e

echo "📝 Setting up source-only development environment..."

TEAM_REPO="https://github.com/your-org/custom-browser.git"
CHROMIUM_MIRROR="https://github.com/chromium/chromium.git"

# Clone team repository
git clone "$TEAM_REPO" browser-dev
cd browser-dev

# Add upstream Chromium as remote
git remote add upstream "$CHROMIUM_MIRROR"

# Setup sparse checkout for essential directories only
git config core.sparseCheckout true
cat > .git/info/sparse-checkout << EOF
/chrome/browser/
/chrome/common/
/content/browser/
/content/renderer/
/ui/
/components/
/*.md
/*.gn
/build/
EOF

git read-tree -m -u HEAD

echo "✅ Source-only setup complete"
echo "Development ready in: $(pwd)"
```

### Script 4: Quick Test Setup
**File**: `setup-testing.sh`
```bash
#!/bin/bash
set -e

echo "🧪 Setting up testing environment..."

# Create testing directory
mkdir -p browser-testing
cd browser-testing

# Download latest artifacts (placeholder for actual implementation)
echo "📦 Downloading latest build artifacts..."
# curl -L https://your-build-server/latest/chromium-debug.tar.gz | tar xz

echo "✅ Testing environment ready"
echo "Launch browser: open Chromium.app"
```

## 🔄 DAILY DEVELOPMENT COMMANDS

### Common Git Workflow
```bash
# Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/my-feature

# Regular development cycle
git add .
git commit -m "feat: implement feature component"
git push origin feature/my-feature

# Create pull request (GitHub CLI)
gh pr create --title "Add new feature" --body "Description..."

# After PR approval and merge
git checkout develop
git pull origin develop
git branch -d feature/my-feature
```

### Build Commands
```bash
# Quick incremental build (5-30 minutes)
export PATH="$HOME/chromium-build/depot_tools:$PATH"
cd $HOME/chromium-build/chromium/src
ninja -C out/Debug chrome

# Release build (optimized, slower compile)
gn gen out/Release --args='is_debug=false'
ninja -C out/Release chrome

# Clean build (when needed)
rm -rf out/Debug
gn gen out/Debug --args='is_debug=true'
ninja -C out/Debug chrome
```

### Testing Commands
```bash
# Launch built browser
open out/Debug/Chromium.app

# Run unit tests
ninja -C out/Debug unit_tests
out/Debug/unit_tests

# Run browser tests
ninja -C out/Debug browser_tests
out/Debug/browser_tests --gtest_filter="YourTest*"
```

## 📝 DEVELOPMENT BEST PRACTICES

### Code Organization
```cpp
// File: chrome/browser/ui/custom_browser/
// Your custom browser features go here

// File: chrome/browser/custom_features/
// Core functionality modifications

// File: chrome/app/
// Application-level customizations
```

### Commit Message Format
```
type(scope): description

feat(ui): add custom navigation bar
fix(security): resolve authentication issue  
docs(readme): update installation guide
test(browser): add integration tests
```

### Feature Flag Strategy
```cpp
// Enable/disable features safely
#include "base/feature_list.h"

BASE_FEATURE(kCustomBrowserFeature,
             "CustomBrowserFeature",
             base::FEATURE_DISABLED_BY_DEFAULT);

// Usage in code
if (base::FeatureList::IsEnabled(kCustomBrowserFeature)) {
    // New feature code
}
```

## 🚦 CI/CD PIPELINE

### GitHub Actions Example
**File**: `.github/workflows/build.yml`
```yaml
name: Build Custom Browser

on:
  push:
    branches: [ develop, main ]
  pull_request:
    branches: [ develop ]

jobs:
  build:
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Xcode
      uses: maxim-lobanov/setup-xcode@v1
      with:
        xcode-version: latest
    
    - name: Build Chromium
      run: |
        ./scripts/ci-build.sh
    
    - name: Run Tests
      run: |
        ./scripts/run-tests.sh
    
    - name: Upload Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: browser-build
        path: out/Debug/Chromium.app
```

## 🔄 UPSTREAM SYNC STRATEGY

### Monthly Chromium Updates
```bash
# Update upstream tracking
git remote add upstream https://chromium.googlesource.com/chromium/src.git
git fetch upstream main

# Create update branch
git checkout -b update/chromium-$(date +%Y-%m)
git merge upstream/main

# Resolve conflicts and test
# Create PR for team review
```

### Security Update Process
```bash
# Emergency security patches
git checkout -b security/patch-$(date +%Y%m%d)
# Apply security commits
git cherry-pick <security-commit-hash>
# Fast-track review and merge
```

## 📋 TEAM ROLES & RESPONSIBILITIES

### Lead Developer
- Repository administration
- Upstream sync coordination
- Architecture decisions
- Security reviews

### Feature Developers  
- Feature implementation
- Unit testing
- Code reviews
- Documentation updates

### QA Engineers
- Integration testing
- Performance testing
- Release validation
- Bug reporting

### DevOps Engineer
- CI/CD maintenance
- Build optimization
- Infrastructure management
- Release automation

## 📞 SUPPORT & TROUBLESHOOTING

### Common Issues
1. **Build failures**: Check Xcode version, disk space
2. **Slow builds**: Use `ccache`, optimize flags
3. **Git conflicts**: Regular rebasing, small commits
4. **Large repository**: Use sparse-checkout, LFS

### Team Communication
- **Daily standups**: Progress and blockers
- **Weekly reviews**: Code quality and architecture
- **Monthly planning**: Feature roadmap updates
- **Quarterly retrospectives**: Process improvements

### Emergency Contacts
- **Build issues**: Lead Developer
- **Infrastructure**: DevOps Engineer  
- **Security**: Security Team Lead
- **Upstream**: Chromium community resources

This comprehensive setup enables your team to collaborate effectively while maintaining the isolation and build quality achieved in your initial setup.