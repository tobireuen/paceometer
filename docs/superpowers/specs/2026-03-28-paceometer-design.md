# Paceometer — Product Design Spec

**Date:** 2026-03-28
**Platform:** iOS 17+, SwiftUI, MVVM

---

## Overview

Paceometer is a real-time GPS driving dashboard that reframes speed as pace (time per distance unit). Research shows drivers systematically overestimate the time savings of driving faster because they think in speed (km/h) rather than pace (min/km). Paceometer makes this bias visceral by putting pace front and centre alongside a rolling "time saved" callout — no trip setup required.

---

## Core Concept

The app shows a live instrument cluster while driving. The primary metric is current pace (e.g. "4:32 min/km"), not speed. A secondary callout shows how much time the user is saving (or not saving) per 100 km compared to a configurable reference speed. The data is neutral and ephemeral — no history, no judgment, no onboarding. The numbers make the point themselves.

---

## Architecture

### Layers

**`LocationService`** (`@Observable` class)
- Wraps `CLLocationManager`
- Streams `CLLocation` updates on `@MainActor`
- Exposes: `currentSpeedMS: Double`, `isAuthorized: Bool`, `sessionStartDate: Date?`
- Uses Swift concurrency (`async`/`await`) — no `DispatchQueue.main` calls
- Requests "When In Use" location permission only

**`SettingsStore`** (`@Observable` class)
- Reads/writes to `UserDefaults`
- Exposes: `unitSystem: UnitSystem` (`.metric` / `.imperial`), `referenceSpeed: Double`, `paceRangeMin: Double`, `paceRangeMax: Double`, `keepScreenAwake: Bool`
- Defaults: unit system from device locale, reference speed 100 km/h (metric) / 60 mph (imperial), pace range 2:00–10:00 min/km

**`PaceViewModel`** (`@Observable` class)
- Consumes `LocationService` and `SettingsStore`
- Derives all display values:
  - `currentPace: String` — formatted pace (e.g. "4:32") or "-- : --" when speed < 5 km/h
  - `currentSpeed: String` — formatted speed with unit
  - `averagePace: String` — rolling session average
  - `timeSavedCallout: TimeSavedResult` — delta vs reference speed per 100 km, with sign and colour intent
  - `gaugeSpeedFraction: Double` — 0.0–1.0 for inner speed needle
  - `gaugePaceFraction: Double` — 0.0–1.0 for outer pace arc
- No business logic in views

**`DashboardView`** (SwiftUI)
- Single full-screen view, no navigation bar or tab bar
- Reads from `PaceViewModel`
- Decomposed into focused subviews:
  - `DualGaugeView` — the dual-concentric instrument dial (SwiftUI `Canvas` / `Path`)
  - `PaceLabelView` — pace value beneath the gauge
  - `TimeSavedBannerView` — time-saved callout
  - `SessionAverageView` — session average pace
  - `TopBarView` — settings icon + unit toggle

**`SettingsView`** (SwiftUI)
- Sheet presented from top bar
- Writes directly to `SettingsStore`

### Data Flow

```
CLLocationManager → LocationService → PaceViewModel → DashboardView subviews
                                   ↑
                         SettingsStore (UserDefaults)
```

### GPS Usage

CoreLocation only — no MapKit, no network calls. The app works fully offline. Background location is not requested; tracking is foreground-only.

---

## Main Dashboard Screen

### Layout

```
┌─────────────────────────────┐
│  ⚙️                   km/mi │  ← minimal top bar, low opacity
├─────────────────────────────┤
│                             │
│      ╭───────────╮          │
│    ╭─┤  87 km/h  ├─╮        │
│    │ │  ● needle │ │        │  ← inner: speed needle (classic sweep)
│    │ ╰───────────╯ │        │
│    ╰───────────────╯        │  ← outer arc: pace ring
│      4:32 min/km            │  ← pace value label below dial
│                             │
├─────────────────────────────┤
│  At this pace:              │
│  −1:48 per 100 km           │  ← time-saved banner
│  vs. 100 km/h               │
├─────────────────────────────┤
│  Avg  4:51 min/km           │  ← session average, smaller
└─────────────────────────────┘
```

### Visual Design

- Full-bleed dark canvas, no navigation chrome
- Subtle radial gradient background (deep charcoal / near-black)
- Instrument-cluster aesthetic with analogue/HUD character
- Pace label uses a large rounded monospaced font
- **Dual-concentric gauge:**
  - Outer ring: pace arc, sweeps across user-configured pace range (default 2:00–10:00 min/km)
  - Inner dial: speed needle, sweeps 0–200 km/h / 0–120 mph
  - Rendered with SwiftUI `Canvas` for smooth animation
  - Gauge dominates the upper ~60% of the screen
- **Time-saved banner accent colour:**
  - Green: going faster than reference speed (negative delta — saving time)
  - Neutral/white: at or near reference speed
  - Amber: going slower than reference speed (positive delta — losing time; not red — avoid alarming colour while driving)
- Screen kept awake via `.keepScreenAwake` modifier while GPS is active (configurable)

---

## Time-Saved Callout

Purely illustrative — no trip or destination required.

**Formula:** `timeDelta = (1/currentSpeed - 1/referenceSpeed) × 100 × 60` minutes per 100 km

- Negative result (currentSpeed > referenceSpeed): user is saving time — displayed as e.g. "−1:48 per 100 km"
- Positive result (currentSpeed < referenceSpeed): user is losing time — displayed as e.g. "+2:14 per 100 km"
- When speed < 5 km/h, callout is hidden

---

## Settings Sheet

| Setting | Control | Default |
|---|---|---|
| Units | Segmented picker: km / mi | Device locale |
| Reference speed | Numeric stepper | 100 km/h / 60 mph |
| Gauge pace min | Stepper (min/km or min/mi) | 2:00 |
| Gauge pace max | Stepper (min/km or min/mi) | 10:00 |
| Keep screen awake | Toggle | On |

All settings persisted to `UserDefaults` via `SettingsStore`. Changes take effect immediately.

---

## Error Handling & Edge Cases

**GPS permission denied:**
- Dashboard shows a friendly full-screen prompt with a deep-link to Settings
- No crash, no broken state

**GPS signal loss:**
- Gauge freezes at last known value
- Subtle "Signal lost" indicator appears
- Tracking resumes automatically when signal returns

**Stationary / very slow speed (< 5 km/h):**
- Pace label displays "-- : --"
- Time-saved callout is hidden
- Gauge needles animate to zero

**Background behaviour:**
- No background location — tracking is foreground-only
- Session average resets on next foreground activation

**Unit change mid-session:**
- Respects stored `UserDefaults` value; does not switch silently

---

## Future Considerations (Out of Scope Now)

- **CarPlay support** — architecture is designed to support this; `PaceViewModel` is display-agnostic
- **Live Activity / Dynamic Island** — natural addition once core is stable
- **Session history** — explicitly excluded; app is ephemeral by design

---

## Testing

### Unit Tests (`XCTest`)

- `PaceViewModel`: speed-to-pace conversion, unit switching, time-saved calculation, edge cases (zero speed, very high speed, below-threshold display)
- `SettingsStore`: read/write round-trips to `UserDefaults`, locale-based defaults

### UI Tests

- Dashboard renders correctly with mocked `LocationService` (injected via protocol)
- Gauge values reflect mocked speed inputs
- "-- : --" displays at zero/low speed
- Settings sheet: unit toggle updates dashboard labels, reference speed change updates callout

### Device Testing

- GPS accuracy and gauge animation smoothness verified on physical device
- Simulator location simulated via GPX files for CI

No snapshot tests — instrument-cluster visual design expected to iterate.
