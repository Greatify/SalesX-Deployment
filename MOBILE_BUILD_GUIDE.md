# SalesX Mobile App - GitHub Actions Build Guide

Complete guide for building Android APK and iOS apps using GitHub Actions - **100% FREE!**

---

## 🚀 **Quick Start**

### **1. Trigger Build from GitHub**

1. Go to: https://github.com/Greatify/SalesX-Deployment
2. Click **"Actions"** tab
3. Select **"Build Mobile Apps"** workflow
4. Click **"Run workflow"** button
5. Choose options:
   - **Platform:** android | ios | both
   - **Build Type:** release | debug
   - **Create Release:** ✅ (recommended)
6. Click **"Run workflow"**

### **2. Wait for Build**

- **Android:** ~10-15 minutes
- **iOS:** ~15-20 minutes
- **Both:** ~20-25 minutes

### **3. Download Your Apps**

**Option A: From Artifacts (90 days)**
- Scroll to bottom of workflow run
- Click "Artifacts" section
- Download APK/iOS app

**Option B: From GitHub Release (Permanent)**
- Go to "Releases" tab
- Find latest release
- Download APK/iOS app
- **Public link - share with anyone!**

---

## 📱 **What Gets Built**

### **Android APK**

**File:** `SalesX-Android-v1.0.0-release.apk`

**Features:**
- ✅ Ready to install on Android devices
- ✅ No Google Play Store needed
- ✅ Works on Android 5.0+
- ✅ ~50-80 MB file size

**How to Install:**
1. Download APK to Android phone
2. Enable "Install from Unknown Sources"
3. Tap APK to install
4. Open SalesX app!

### **iOS App (Simulator)**

**File:** `SalesX-iOS-v1.0.0-release-simulator.app.zip`

**Features:**
- ✅ Works in iOS Simulator (Mac only)
- ⚠️ Does NOT work on real iPhones (no code signing)
- ✅ Good for testing on Mac
- ✅ ~100-150 MB file size

**How to Use:**
1. Download .app.zip on Mac
2. Unzip the file
3. Open Xcode → Simulator
4. Drag .app to simulator
5. Open SalesX app!

**For Real iPhone:**
- Use EAS Build (paid)
- Or build locally with Xcode + Apple Developer account

---

## 🎯 **Build Options Explained**

### **Platform**

| Option | What Builds | Time | Best For |
|--------|-------------|------|----------|
| `android` | Android APK only | ~10-15 min | Quick Android testing |
| `ios` | iOS simulator app only | ~15-20 min | Quick iOS testing |
| `both` | Android + iOS | ~20-25 min | Full release |

### **Build Type**

| Option | What It Means | Size | Performance |
|--------|---------------|------|-------------|
| `release` | Optimized, production-ready | Smaller | Faster |
| `debug` | With debug symbols | Larger | Slower |

**Recommendation:** Always use `release` for testing/distribution

### **Create Release**

| Option | Effect |
|--------|--------|
| ✅ Yes | Creates GitHub Release with public download link |
| ❌ No | Only uploads to artifacts (requires GitHub login) |

**Recommendation:** Enable for easy sharing

---

## 📥 **How to Download & Share**

### **Method 1: GitHub Artifacts (90 days, requires login)**

After build completes:

1. **Navigate to workflow run:**
   ```
   GitHub → Actions → Click on your workflow run
   ```

2. **Scroll to "Artifacts" section at bottom**

3. **Download:**
   - `android-apk` (contains APK)
   - `ios-app` (contains iOS app)

4. **Extract and use!**

**Sharing:**
- ❌ Cannot share artifacts directly
- ✅ Download and upload to your own cloud storage
- ✅ Or use GitHub Releases instead

---

### **Method 2: GitHub Releases (Permanent, public link)** ⭐ **RECOMMENDED**

After build completes:

1. **Go to Releases tab:**
   ```
   https://github.com/Greatify/SalesX-Deployment/releases
   ```

2. **Find latest release:**
   - Named: `SalesX Mobile - v2024.01.13-build-XX`

3. **Download apps:**
   - Click on APK or iOS app file
   - Direct download (no GitHub account needed!)

4. **Share the link:**
   ```
   https://github.com/Greatify/SalesX-Deployment/releases/latest
   ```
   - Anyone can access!
   - No GitHub login required!
   - Permanent link!

---

## 🔗 **Sharing Options**

### **Option 1: Share Release Link** (Easiest)

```
https://github.com/Greatify/SalesX-Deployment/releases/latest
```

**Pros:**
- ✅ Public link
- ✅ No login required
- ✅ Permanent
- ✅ Shows version history
- ✅ Professional looking

**Cons:**
- ❌ Requires public repo

---

### **Option 2: QR Code** (For Beta Testers)

1. Get release link
2. Create QR code: https://www.qr-code-generator.com/
3. Share QR code
4. Users scan → download APK

**Perfect for:**
- Field testing
- Beta testers
- Quick distribution

---

### **Option 3: Upload to Cloud Storage**

After downloading from GitHub:

**Google Drive:**
1. Upload APK to Google Drive
2. Right-click → Share → Get link
3. Change to "Anyone with the link"
4. Share link

**Dropbox:**
1. Upload to Dropbox
2. Create shared link
3. Share link

**AWS S3:**
1. Upload to S3 bucket
2. Make object public
3. Use public URL

---

## 📊 **Build Process**

### **What Happens When You Trigger:**

```
┌─────────────────────────────────────────┐
│  1. Checkout SalesX source code         │
│     (from main branch)                  │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│  2. Setup environment                   │
│     - Node.js 20                        │
│     - Android SDK (for Android)         │
│     - Xcode (for iOS)                   │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│  3. Install dependencies                │
│     npm ci --legacy-peer-deps           │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│  4. Generate native project             │
│     npx expo prebuild                   │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│  5. Build app                           │
│     Android: gradlew assembleRelease    │
│     iOS: xcodebuild                     │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│  6. Upload artifacts                    │
│     - APK/iOS app to artifacts          │
│     - Create GitHub Release             │
└─────────────────────────────────────────┘
```

---

## 💰 **Cost Breakdown (FREE!)**

### **GitHub Actions Minutes Used:**

| Platform | Runner | Time | Minutes Deducted | Cost |
|----------|--------|------|------------------|------|
| Android | Ubuntu | ~15 min | 15 min | **FREE** |
| iOS | macOS | ~20 min | 200 min* | **FREE** |
| Both | Mixed | ~25 min | 215 min* | **FREE** |

*macOS has 10x multiplier (20 min = 200 min deducted)

### **Free Tier Limits (Public Repo):**

- ✅ **Unlimited** minutes for public repos!
- ✅ **Unlimited** storage for artifacts (90 days)
- ✅ **Unlimited** releases
- ✅ **No credit card needed**

**Your repo is public, so this is 100% FREE!** 🎉

---

## 🛠️ **Advanced Usage**

### **Build Specific Branch**

To build from a different branch, update workflow:

```yaml
# In mobile-build.yaml, line 32
ref: 'develop'  # Change from 'main' to your branch
```

### **Customize App Version**

Version comes from `SalesX/mobile/app.json`:

```json
{
  "expo": {
    "version": "1.0.0"  ← This is used
  }
}
```

Update this to change app version in builds.

### **Auto-Build on Push**

Add this to trigger automatic builds:

```yaml
on:
  workflow_dispatch:
  push:
    branches: [main]
    paths:
      - 'SalesX/mobile/**'
```

Builds automatically when mobile code changes!

---

## 📋 **Troubleshooting**

### **Build Failed - Android**

**Error:** "SDK not found"
```yaml
# Workflow already handles this - should auto-fix
```

**Error:** "Out of memory"
```yaml
# Add to workflow (already included):
- name: Setup Gradle
  run: |
    echo "org.gradle.jvmargs=-Xmx4096m" >> gradle.properties
```

### **Build Failed - iOS**

**Error:** "Code signing required"
```yaml
# For simulator builds (current setup):
CODE_SIGNING_ALLOWED=NO  # Already set in workflow

# For device builds:
# Use EAS Build or local Xcode with certificates
```

**Error:** "Pod install failed"
```yaml
# Workflow uses Ruby 3.2 and auto-installs pods
# Should work automatically
```

### **Artifacts Not Found**

- Build must complete successfully
- Artifacts available for 90 days
- Check workflow logs for errors

---

## 📱 **Testing Your Builds**

### **Android APK Testing:**

1. **Download APK from release**
2. **Transfer to Android device:**
   - Email yourself
   - Google Drive
   - Direct USB transfer
3. **Enable Unknown Sources:**
   - Settings → Security → Unknown Sources
4. **Install APK**
5. **Test the app!**

### **iOS App Testing (Simulator):**

1. **Download .app.zip from release**
2. **Unzip on Mac**
3. **Open iOS Simulator:**
   ```bash
   open -a Simulator
   ```
4. **Drag .app to simulator**
5. **Test the app!**

---

## 🎯 **Best Practices**

### **For Beta Testing:**

1. ✅ Use `release` build type
2. ✅ Enable "Create Release"
3. ✅ Build `both` platforms
4. ✅ Share release link via QR code
5. ✅ Create new build for each version

### **For Internal Testing:**

1. ✅ Use `debug` build type initially
2. ✅ Build single platform as needed
3. ✅ Download from artifacts
4. ✅ Switch to `release` before wider testing

### **For Production:**

1. ✅ Always use `release` build
2. ✅ Test thoroughly before publishing
3. ✅ Keep release notes updated
4. ✅ Version number in app.json
5. ✅ Use EAS or local build for store submission

---

## 📊 **Workflow Features**

### **✅ What's Included:**

- [x] Manual trigger with options
- [x] Build Android APK
- [x] Build iOS simulator app
- [x] Upload artifacts (90 days)
- [x] Create GitHub Releases
- [x] Public download links
- [x] Version tracking
- [x] Build summaries
- [x] Concurrent build prevention

### **❌ What's NOT Included:**

- [ ] iOS device builds (requires code signing)
- [ ] Automatic Play Store/App Store submission
- [ ] TestFlight integration
- [ ] Push notification setup
- [ ] App signing for Play Store

For these, use EAS Build or local builds with proper certificates.

---

## 🔗 **Quick Links**

- **Workflow File:** `.github/workflows/mobile-build.yaml`
- **Trigger Build:** https://github.com/Greatify/SalesX-Deployment/actions
- **View Releases:** https://github.com/Greatify/SalesX-Deployment/releases
- **Source Code:** https://github.com/Greatify/SalesX

---

## 🆘 **Need Help?**

### **Common Questions:**

**Q: How do I share the APK with my team?**
A: Share the GitHub Release link - anyone can download!

**Q: Can I build for real iPhones?**
A: No, this builds simulator apps. Use EAS Build or local Xcode for real devices.

**Q: How long are builds available?**
A: Artifacts: 90 days, Releases: Forever!

**Q: Can I automate builds?**
A: Yes! Add `push` trigger to workflow for auto-builds.

**Q: Is this really free?**
A: Yes! Public repos have unlimited GitHub Actions minutes.

---

## 🎉 **You're All Set!**

To build your first app:

1. Go to https://github.com/Greatify/SalesX-Deployment/actions
2. Click "Build Mobile Apps"
3. Click "Run workflow"
4. Select `both` + `release` + ✅ Create Release
5. Wait ~25 minutes
6. Download from Releases tab!

**Happy Building!** 🚀📱

---

**Last Updated:** January 2026  
**Workflow Version:** 1.0  
**Supported Platforms:** Android 5.0+, iOS Simulator
