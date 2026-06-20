# PluginBuilder — Beta

A macOS app that handles the full build-to-distribution pipeline for Audio Unit and VST3 plugins — no shell scripts required.

> **Beta notice:** This is a pre-release build. Please report bugs, unexpected behavior, or feature requests in the [feedback repository](https://github.com/ShmKnd/PluginBuilder_feedback/issues).

---

## Requirements

- macOS 14.0 (Sonoma) or later — Apple Silicon only
- Xcode and Command Line Tools installed
- A JUCE-generated `.xcodeproj` or `.xcworkspace`
- For PKG creation:
  - Developer ID Application certificate
  - Developer ID Installer certificate
  - Apple ID with App-Specific Password, or a `notarytool` Keychain Profile

App Sandbox is disabled. PluginBuilder launches build tools, reads and writes project and plugin bundle files, and communicates with Apple's notarization service. Mac App Store distribution is not supported.

---

## What it does

PluginBuilder runs the following steps in sequence, entirely from the GUI:

1. `xcodebuild`
2. Collect built AU / VST3 bundles
3. Copy to `UniBinary` / `AppleSilicon` folders alongside your project
4. `codesign`
5. `pkgbuild`
6. `productbuild`
7. `xcrun notarytool submit` (upload)
8. `xcrun notarytool wait` (runs in background while next build starts)
9. `xcrun stapler staple` on approval
10. Copy to your configured PKG output folder

Disabling **Create PKG** stops after step 3 — useful for development builds.

---

## Key features

**Universal Binary + Apple Silicon, both in one run**
Produces `PluginName_UniBinary_v1.0.0.pkg` (arm64 + x86_64) and `PluginName_AppleSilicon_v1.0.0.pkg` (arm64) from a single build execution.

**Multi-plugin queue**
Select multiple plugins in the sidebar and hit Build. They run sequentially, one at a time. Notarization waits run in the background so the next build can start immediately.

**Combined PKG**
Bundle multiple plugins into a single installer with per-format (AU / VST3) selection at install time.

**Package Builder**
Drop an existing `.component` or `.vst3` into the sub-window (`⌘⇧P`) to generate a PKG without going through a full build. When the required signing and notarization credentials are available, PluginBuilder signs and notarizes the package from the GUI.

**Notarization Profile Manager**
Create, list, and delete `notarytool` Keychain profiles directly from the app. No terminal required. Open from Settings → Notarization → Manage Profiles.

**Auto-detection**
JUCE version string and VST3 category are read from your project headers automatically when you drop in a `.xcodeproj`.

**Build Cache**
Optionally reuse per-plugin DerivedData for faster incremental builds. Disable and rebuild clean if a source change isn't reflected correctly.

**Build Jobs**
Set `xcodebuild -jobs` per plugin. `0` defers to Xcode's default scheduling.

---

## Initial setup

Open **Settings** (toolbar icon) and fill in the shared values. These are used as defaults when individual plugin configurations leave a field blank.

The settings window is divided into four categories:

**Install** — Default AU and VST3 install paths.

**Package** — PKG output destination (`Project / dist` or a global folder). Also controls the version mismatch warning for Combined PKG.

**Signing** — Developer name, Team ID, and Developer ID Application / Installer certificate values. Enter your Developer name and Team ID, then press `Apply` to auto-generate the standard certificate strings. You can also edit the two Developer ID fields directly using the common name form.

**Notarization** — Apple ID plus either App-Specific Password or Keychain Profile, shared across plugins. Use `Manage Profiles...` to create and manage Keychain profiles from the GUI.

---

## Adding a plugin

Click **Add Plugin** at the bottom of the sidebar, or drag and drop a `.xcodeproj` / `.xcworkspace` onto the sidebar. AU and VST3 schemes are detected automatically.

Per-plugin configuration includes:

- Display name and project name
- `.xcodeproj` / `.xcworkspace` path
- AU and VST3 scheme names
- AU / VST3 install paths (overrides global defaults)
- Developer ID certificates (overrides global defaults)
- Notarization credentials (overrides global defaults)
- Build configuration (default: Release)
- Universal Binary toggle
- Build Cache and Build Jobs
- Create PKG toggle
- Create Combined PKG toggle

---

## Running a build

Use the **RunBar** at the bottom of the main window and press **Build**.

The version is stored per plugin configuration, auto-filled from JUCE headers when available, and shown read-only in the RunBar. If needed, update it from the plugin configuration before building.

To build multiple plugins, select them in the sidebar before pressing Build. They run in sidebar order, one at a time.

---

## Combined PKG

Enable **Create Combined PKG** on each plugin you want to bundle. When a build starts, all checked plugins are collected automatically — you don't need to select all of them in the sidebar.

A dialog will prompt for the installer name before the build starts. The name applies to that build only and is not saved.

The installer presents AU and VST3 as top-level choices, with individual plugins expandable under each format.

If plugin versions don't match, a warning dialog appears before the build. You can suppress future warnings from that dialog; re-enable from Settings → Package → Version Warning.

Output example:
```
Combined_UniBinary_v1.2.0.pkg
Combined_AppleSilicon_v1.2.0.pkg
```

---

## Notarization Profile Manager

Open from **Settings → Notarization → Manage Profiles**.

- **New Profile** — runs `xcrun notarytool store-credentials` internally. macOS may show a Keychain access dialog.
- **Add Existing** — register a profile by name if it was created outside PluginBuilder.
- **Delete** — removes the Keychain entry and PluginBuilder's metadata (for profiles created in PluginBuilder).
- **Remove** — removes only PluginBuilder's metadata, leaving the Keychain entry intact (for profiles added via Add Existing).
- **Edit** — available for Add Existing entries so you can update PluginBuilder's display metadata without changing the Keychain credential itself.

Credentials are stored in Keychain. No passwords are written to disk in plaintext.

---

## Build log

Logs are saved per plugin and persist across app restarts. The last log is restored on next launch.

Filter by: `All` / `Success` / `Warning` / `Error`

Build results are shown per variant and format:
```
[ok ] Build succeeded: UniBinary / AU (PluginName – AU)
[ok ] Build succeeded: AppleSilicon / VST3 (PluginName – VST3)
```

Log retention: 30 days from last save, up to 5 MB per plugin.

---

## Sidebar build history

Expand a plugin entry in the sidebar to see the last build result:

- **Last Build** — date and time (your locale and timezone)
- **Last Build Version** — version string and scheme names
- **Status** — per-step result for Build / Sign / Package / Notarize

---

## Notes

- Builds cannot start if required credentials are missing from the effective configuration. In the current beta, Team ID is still required even when Keychain Profile authentication is selected.
- A `notarytool` 403 error usually indicates an issue with your Apple Developer Program membership or credentials. Check App Store Connect.
- Do not quit PluginBuilder while notarization or stapling is in progress. Background tasks are not restored on relaunch.

---

## Feedback

Please open an issue at **[github.com/ShmKnd/PluginBuilder_feedback](https://github.com/ShmKnd/PluginBuilder_feedback)** for:

- Bugs or crashes
- Unexpected behavior with your project setup
- Feature requests

Especially useful: notarization flow with different setups (App-Specific Password vs Keychain Profile), non-standard scheme names, and `.xcworkspace` projects.
