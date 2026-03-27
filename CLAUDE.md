# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Paceometer is an iOS-only SwiftUI app. Target minimum deployment: iOS 17+.

## Architecture

Use MVVM throughout:
- **Model**: Plain Swift types (structs preferred over classes)
- **ViewModel**: `@Observable` classes (iOS 17+); use `ObservableObject` only if targeting below iOS 17
- **View**: SwiftUI views, no business logic — delegate to ViewModel

Prefer `@Observable` (Observation framework) over `ObservableObject`/`@StateObject`/`@ObservedObject` for new ViewModels.

## SwiftUI Guidelines

- Prefer native SwiftUI APIs over UIKit/AppKit bridging
- Gate iOS 26+ APIs (Liquid Glass, etc.) with `#available(iOS 26, *)`
- Decompose views into small, focused subviews rather than large monolithic bodies
- Use Swift concurrency (`async`/`await`, `Task`, `@MainActor`) — avoid callbacks and DispatchQueue

## Git Workflow (Gitflow)

- `main` — stable releases only
- `develop` — integration branch; all features merge here
- `feature/<name>` — branch from `develop`, merge back via PR
- `hotfix/<name>` — branch from `main`, merge to both `main` and `develop`
- Commit messages: imperative mood, lowercase first letter, no trailing period, no brackets, no conventional-commit prefixes (e.g. no `feat:`, `fix:`, `docs:`), e.g. `add pace calculation logic`
