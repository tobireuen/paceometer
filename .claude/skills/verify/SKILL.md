---
name: verify
description: Build the project and run all tests to verify changes are correct before finishing. Use this before marking any implementation task done.
---

Run the full verification suite for paceometer:

1. **Build** — compile the project and confirm there are no errors or warnings introduced by recent changes.
   ```bash
   DEVELOPER_DIR=/Applications/Xcode-new.app/Contents/Developer xcodebuild \
     -scheme Paceometer-Package \
     -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
     build 2>&1
   ```

2. **Test** — run the full test suite.
   ```bash
   DEVELOPER_DIR=/Applications/Xcode-new.app/Contents/Developer xcodebuild \
     -scheme Paceometer-Package \
     -destination 'platform=iOS Simulator,name=iPhone 17 Pro' \
     test 2>&1
   ```

3. **Report** — summarise:
   - Any build errors or warnings
   - Test pass/fail count
   - Any new failures introduced by recent changes

Important: always use `/Applications/Xcode-new.app` (not `/Applications/Xcode.app`). The scheme is `Paceometer-Package`. Do not use `swift build` or `swift test` — they target macOS and fail on iOS-only APIs.
