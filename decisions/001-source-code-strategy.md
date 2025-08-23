# Decision Record 001: Source Code Strategy

**Date**: August 24, 2025  
**Status**: ✅ DECIDED  
**Decision Maker**: Project Lead  
**Context**: Team collaboration setup for custom browser development  

## 📋 **DECISION SUMMARY**

**Chosen Strategy**: **Option B - Patch-Based Approach**  
**Alternative Considered**: Option A - Fork Chromium on GitHub  

## 🤔 **PROBLEM STATEMENT**

Need to determine the best approach for sharing Chromium source code with the development team while enabling:
- Efficient team collaboration
- Easy upstream synchronization with Chromium updates
- Manageable repository sizes
- Fast development iteration

## ⚖️ **OPTIONS EVALUATED**

### Option A: Fork Chromium on GitHub
**Approach**: Fork https://github.com/chromium/chromium and add changes as commits

**Pros:**
- Standard Git workflow
- Complete version history
- Easy branch management within fork
- Industry-standard approach

**Cons:**
- **Repository size**: 15-20GB (productivity killer)
- **Merge conflicts**: Complex upstream integration
- **Infrastructure costs**: Git LFS charges for large files
- **Clone time**: 2-3 hours vs 30 seconds
- **GitHub performance**: Degrades with large repositories

### Option B: Patch-Based Approach ✅ CHOSEN
**Approach**: Keep only changes as patch files, apply to fresh Chromium downloads

**Pros:**
- **Small repository**: 1-50MB (only changes)
- **Fast operations**: 30-second clones
- **Simple upstream sync**: `gclient sync` + `git apply`
- **Cost effective**: No Git LFS costs
- **Team efficiency**: Focus on changes only
- **Build flexibility**: Easy A/B testing with/without patches

**Cons:**
- **Non-standard workflow**: Team needs training
- **Patch management**: Requires discipline in patch organization
- **Merge conflicts**: Must resolve in patch files

## 🎯 **DECISION RATIONALE**

### **Primary Factors:**
1. **Team Productivity**: 30-second clones vs 3-hour downloads
2. **Repository Management**: 50MB vs 20GB repositories  
3. **Upstream Integration**: Simple patch application vs complex merges
4. **Cost Management**: Zero Git LFS costs vs potential hundreds monthly

### **Risk Assessment:**
- **Low Risk**: Patch-based is proven approach (Linux kernel, many OSS projects)
- **Mitigation**: Automated scripts for patch application
- **Fallback**: Can convert patches to fork later if needed

### **Team Size Consideration:**
- **Current team**: Small (2-5 developers) - patches ideal
- **Future scaling**: Can migrate to fork if team grows to 20+ developers

## 🛠️ **IMPLEMENTATION PLAN**

### Repository Structure:
```
custom-browser-project/ (~10-50MB)
├── patches/
│   ├── 001-custom-branding.patch
│   ├── 002-privacy-features.patch
│   └── 003-ui-modifications.patch
├── scripts/
│   ├── apply-patches.sh
│   ├── build-browser.sh
│   └── setup-dev-env.sh
├── docs/ (existing documentation)
├── assets/ (icons, branding files)
└── configs/ (build configurations)
```

### Developer Workflow:
```bash
# 1. Setup Chromium base (one-time, 6-8 hours)
./scripts/setup-dev-env.sh

# 2. Apply team modifications (daily, 5 minutes)
./scripts/apply-patches.sh

# 3. Build and test (incremental, 10-30 minutes)
./scripts/build-browser.sh

# 4. Create new patches (as needed)
git diff > patches/004-new-feature.patch
```

### Upstream Sync Process:
```bash
# Monthly Chromium updates
gclient sync                     # Get latest Chromium
./scripts/apply-patches.sh       # Apply our changes
# Resolve any conflicts in patch files
# Test and commit updated patches
```

## 📊 **SUCCESS METRICS**

### **Immediate (Week 1-2):**
- [ ] Repository created and team can clone in <1 minute
- [ ] Automated patch application working
- [ ] First team member successfully builds browser

### **Short-term (Month 1):**
- [ ] All team members productive with patch workflow
- [ ] Successful upstream Chromium integration
- [ ] <10 minute overhead for patch application

### **Long-term (Quarter 1):**
- [ ] 5+ custom features implemented as patches
- [ ] Zero repository size issues
- [ ] Successful team scaling if needed

## 🔄 **REVIEW CONDITIONS**

**Re-evaluate if:**
- Team grows beyond 10 developers
- Patch conflicts become frequent (>weekly)
- Feature divergence becomes significant (>1000 files modified)
- Infrastructure costs exceed $500/month

**Migration to Fork:**
- Patches convert easily to fork commits
- Can be done anytime without losing work
- Provides upgrade path as project scales

## 📝 **NEXT ACTIONS**

1. **Create patch-based repository** (this week)
2. **Develop patch automation scripts** (next week)
3. **Train team on patch workflow** (week 2-3)
4. **Implement first feature as patch** (week 4)

## 🔗 **RELATED DECISIONS**

- **Pending**: Build infrastructure choice (local vs shared)
- **Pending**: CI/CD platform selection
- **Pending**: First feature priority

---

**Decision Final**: Patch-based approach chosen for optimal team productivity and repository management.

**Next Decision**: [002-build-infrastructure-strategy.md]