---
name: verify
description: Build the project and run all tests to verify changes are correct before finishing. Use this before marking any implementation task done.
---

Run the full verification suite for paceometer:

1. **Build** — compile the project and confirm there are no errors or warnings introduced by recent changes.
   - If using Xcode CLI: `xcodebuild -scheme paceometer -destination 'platform=iOS Simulator,name=iPhone 16' build`
   - If using Swift Package Manager: `swift build`
   - Adapt the scheme/destination to match what is configured in the project.

2. **Test** — run the full test suite.
   - If using Xcode CLI: `xcodebuild -scheme paceometer -destination 'platform=iOS Simulator,name=iPhone 16' test`
   - If using Swift Package Manager: `swift test`

3. **Report** — summarise:
   - Any build errors or warnings
   - Test pass/fail count
   - Any new failures introduced by recent changes

If the project's build system has not been configured yet, tell the user and suggest the next setup step (e.g. creating a Package.swift or Xcode project).
