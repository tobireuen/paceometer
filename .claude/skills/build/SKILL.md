---
name: build
description: Build the Paceometer project and run tests using xcodebuild. Use this skill whenever the user asks to build, compile, run tests, check for build errors, or verify that the project compiles. Also trigger when the user says "does it build?", "run the tests", "check compilation", or any variation. This skill knows the correct Xcode toolchain, build targets, and test configuration — always prefer it over manual build commands.
---

# Build — Paceometer

This skill builds the Paceometer iOS app and runs its test suite. It knows the project's exact targets, toolchain requirements, and how to interpret results.

## Xcode Toolchain

This project uses a specific Xcode installation. Every `xcodebuild` command **must** be prefixed with:

```
DEVELOPER_DIR=/Applications/Xcode-new.app/Contents/Developer
```

The default `/Applications/Xcode.app` is an older version used for other projects — using it here will cause version mismatches or build failures.

## Why xcodebuild, not swift build/test

This is an iOS-only SPM project (platform: `.iOS(.v17)`). The `swift build` and `swift test` CLI commands target the macOS host platform, which causes availability errors for iOS-only APIs like SwiftUI modifiers. Use `xcodebuild` with the iOS Simulator destination instead — this properly resolves iOS API availability.

## Project Structure

Paceometer is a Swift Package Manager project (no .xcodeproj). The `Package.swift` at the repo root defines two targets:

| Target | Type | Path | Purpose |
|--------|------|------|---------|
| **PaceometerLib** | Library | `Paceometer/` | Core app logic — Models, Services, ViewModels, Views |
| **PaceometerTests** | Test | `PaceometerTests/` | Unit tests for PaceometerLib |

The auto-generated xcodebuild scheme is **`Paceometer-Package`** (not `Paceometer` or `PaceometerLib`).

**PaceometerLib** excludes two files that only make sense in a full app context:
- `PaceometerApp.swift` — the SwiftUI app entry point
- `Services/LocationService.swift` — requires CoreLocation (excluded for testability; the protocol `LocationProviding` is included instead)

**PaceometerTests** contains:
- `PaceViewModelTests.swift` (8 tests)
- `PaceFormattingTests.swift` (9 tests)
- `SettingsStoreTests.swift` (5 tests)

Total: 22 tests.

Swift settings: `StrictConcurrency` experimental feature is enabled on PaceometerLib.

## Build Commands

### Build the library

```bash
DEVELOPER_DIR=/Applications/Xcode-new.app/Contents/Developer xcodebuild \
  -scheme Paceometer-Package \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
  build 2>&1
```

Check for `** BUILD SUCCEEDED **` or `** BUILD FAILED **` in the output.

### Run tests

```bash
DEVELOPER_DIR=/Applications/Xcode-new.app/Contents/Developer xcodebuild \
  -scheme Paceometer-Package \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
  test 2>&1
```

Check for `** TEST SUCCEEDED **` or `** TEST FAILED **`. The output includes individual test pass/fail lines and a summary with total executed count and failures.

## Workflow

1. **Build** — run the xcodebuild build command and check for compilation errors or warnings
2. **Test** — run the xcodebuild test command and check for test failures
3. **Report** — summarize:
   - Build status (success/failure) and any errors or warnings
   - Test results: pass count, fail count, and names of any failing tests
   - If a test fails, show the relevant assertion message to help diagnose the issue

If the build fails, focus on the first error — cascading errors are usually noise from the root cause.
