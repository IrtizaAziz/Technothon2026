# App UI Design Prompt — Smart Plug Power Management Ecosystem

> **Instructions for Claude:** This document contains complete design specifications for the mobile app. Generate one screen at a time in order. For each screen, produce: (1) full ASCII wireframe, (2) component breakdown table, (3) interaction map, (4) all relevant states rendered. Use only the tokens from Section 1 and copy strings from Section 8. Do not invent colors, fonts, spacing, or text.

---

## Table of Contents

1. [Global Design System](#1-global-design-system)
   - 1.5. [Micro-Interactions & Animation Library](#15-micro-interactions--animation-library)
2. [Navigation Architecture](#2-navigation-architecture)
3. [Dashboard Screen](#3-dashboard-screen)
4. [Plug Detail Screen](#4-plug-detail-screen)
5. [Pairing Flow](#5-pairing-flow)
6. [AI Insights Screen](#6-ai-insights-screen)
7. [Community Grid Screen](#7-community-grid-screen)
8. [Settings Screen](#8-settings-screen)
9. [Reusable Component Specs](#9-reusable-component-specs)
   - Component Visual Polish & Micro-Interactions
10. [Data Display Conventions](#10-data-display-conventions)
   - 10.5. [Icon System & Emoji Specification](#105-icon-system--emoji-specification)
11. [Copy Bank](#11-copy-bank)
12. [Output Instructions for Claude](#12-output-instructions)

---

## 1. Global Design System

### Color Palette

| Token | Hex | Usage |
|-------|-----|-------|
| `--color-primary` | `#00C853` | Primary actions, ON state, success, eco indicators |
| `--color-primary-dark` | `#009624` | Pressed state, primary text on light |
| `--color-surface` | `#1E1E1E` | Card backgrounds, elevated surfaces |
| `--color-background` | `#121212` | App background, screen fill |
| `--color-background-alt` | `#181818` | Section dividers, subtle separation |
| `--color-text-primary` | `#FFFFFF` | Headlines, primary labels, active text |
| `--color-text-secondary` | `#B0B0B0` | Body text, captions, secondary labels |
| `--color-text-tertiary` | `#757575` | Placeholder, disabled text, metadata |
| `--color-online` | `#00E676` | Plug online status, connected indicators |
| `--color-offline` | `#757575` | Plug offline status, disconnected |
| `--color-warning` | `#FF6D00` | Medium consumption (200-800W), budget 70-90%, mild anomaly |
| `--color-danger` | `#FF1744` | High consumption (>800W), budget exceeded, critical anomaly |
| `--color-divider` | `#2C2C2C` | Thin borders, list dividers |
| `--color-overlay` | `#00000080` | Modal backdrop, 50% opacity |

### Typography

| Token | Font | Size | Weight | Usage |
|-------|------|------|--------|-------|
| `--font-display` | Inter | 32sp | Bold (700) | Large wattage readout, key metrics |
| `--font-headline` | Inter | 24sp | Bold (700) | Screen titles, section headers |
| `--font-title` | Inter | 18sp | SemiBold (600) | Card titles, dialog headers |
| `--font-body` | Inter | 14sp | Regular (400) | Paragraphs, descriptions, labels |
| `--font-caption` | Inter | 12sp | Regular (400) | Timestamps, metadata, helper text |
| `--font-mono` | JetBrains Mono | 32sp | Bold (700) | Wattage numbers (DeviceCard, PowerGauge center) |
| `--font-mono-small` | JetBrains Mono | 18sp | Medium (500) | Small numeric readouts |

### Spacing Scale

| Token | Value | Usage |
|-------|-------|-------|
| `--space-xs` | 4dp | Icon padding, tight gaps |
| `--space-sm` | 8dp | Chip gaps, inline spacing |
| `--space-md` | 12dp | Card inner padding, list item gaps |
| `--space-lg` | 16dp | Card outer margins, section gaps |
| `--space-xl` | 24dp | Screen padding, large section dividers |
| `--space-2xl` | 32dp | Top-of-screen spacing, major separators |

### Corner Radii

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 8dp | Chips, small badges, inline elements |
| `--radius-md` | 12dp | Buttons, input fields, small cards |
| `--radius-lg` | 16dp | Cards, sheets, modals |
| `--radius-xl` | 24dp | Large panels, bottom sheets |
| `--radius-full` | 999dp | Pills, status dots, circular gauges |

### Shadows & Elevation

| Token | Value | Usage |
|-------|-------|-------|
| `--elevation-card` | 0dp Y, 2dp blur, #00000040 | Standard card |
| `--elevation-card-hover` | 0dp Y, 8dp blur, #00000060 | Card on hover/press (elevated) |
| `--elevation-modal` | 0dp Y, 8dp blur, #00000060 | Modal/sheet backdrop |
| `--elevation-fab` | 0dp Y, 4dp blur, #00000050 | Floating action button |

### Typography Details

| Property | Spec | Usage |
|----------|------|-------|
| Line Height (body) | 1.5 | Body text, descriptions (14sp) |
| Line Height (headlines) | 1.2 | Headlines, titles (18sp+) |
| Letter Spacing (display) | +0.5% | Large wattage readouts (32sp mono) |
| Letter Spacing (headlines) | 0% | Section headers and titles |
| Font Smoothing | `-webkit-font-smoothing: antialiased` | All text on dark backgrounds |
| Text Truncation | max-lines: 2, ellipsis | Device names, card titles |

### Opacity & State Scale

| State | Opacity | Usage |
|-------|---------|-------|
| Default | 100% | Normal interactive elements, text |
| Hover | 85% | Cards, buttons on hover (slight reduction) |
| Active/Pressed | 70% | Buttons during press, dismissed items |
| Disabled | 50% | Disabled toggles, offline devices, read notifications |
| Faded | 60% | Error state text, last-known values |

---

## 1.5. Micro-Interactions & Animation Library

### Motion Principles
- **Quick interactions** (toggles, taps): Snappy, responsive feel — ease-out timing
- **Screen transitions** (navigation): Smooth, elegant — ease-in-out timing
- **Data changes** (value updates): Gradual reveal — ease-out-cubic timing
- **Loading states** (spinners, skeletons): Constant, hypnotic — linear or ease-in-out timing
- **Dismissals** (swipe away cards): Satisfying momentum — ease-in timing

### Standard Easing Curves
| Curve | Flutter Code | Duration | Use Case |
|-------|-------------|----------|----------|
| **Quick** | `Curves.easeOut` | 200ms | Button press, toggle, instant feedback |
| **Standard** | `Curves.easeInOutCubic` | 400ms | Screen transitions, modal entrance |
| **Smooth** | `Curves.easeOutCubic` | 500ms | Arc fills (PowerGauge), progress bars, value animations |
| **Bouncy** | `Curves.elasticOut` | 400ms | Achievement animations, success confirmations |
| **Linear** | `Curves.linear` | ∞ | Loading spinners, continuous animations |

### Global Transition Timings
| Interaction | Duration | Curve | Details |
|-------------|----------|-------|---------|
| Button tap (ripple/feedback) | 200ms | easeOut | Immediate, snappy response |
| Toggle switch | 200ms | easeOut | State change animation |
| Card elevation on hover | 150ms | easeOut | Lift effect, shadow expansion |
| Modal/Sheet entrance | 400ms | easeInOutCubic | Slide up from bottom, fade-in |
| Screen push transition | 400ms | easeInOutCubic | Horizontal slide + fade |
| Value changes (watts, cost) | 500ms | easeOutCubic | Smooth arc fill, number transitions |
| Card dismiss/swipe | 300ms | easeIn | Accelerating exit animation |
| Skeleton pulse | 1.2s | easeInOut | Subtle opacity pulse for loaders |
| Badge notification pulse | 2.0s | easeInOut | Attention-grabbing pulse on anomaly badge |

### Component-Specific Micro-Interactions

**DeviceCard:**
- Hover: Card lifts with shadow elevation, scale +2%, duration 150ms
- Press: Scale -1%, duration 100ms, then return with easeOut
- Tap toggle: Switch animates over 200ms, state changes immediately with optimistic update

**PowerGauge:**
- Arc fill on value change: Smooth arc animation from 0° to target angle, 500ms easeOutCubic
- Critical state glow: Red glow pulses when anomaly detected, 1.5s loop, opacity 100%→60%→100%
- Tooltip: Fade in on tap, opacity 0→100%, 200ms easeOut

**Consumption Chart:**
- Line draw animation: Animate path drawing on chart load, 600ms easeOut
- Data point highlight: Tap point → tooltip fades in, circle grows from point, 200ms easeOut
- Range toggle: Fade out old chart, fade in new data, 300ms crossfade

**BudgetBar:**
- Progress fill: Animate from current to new percentage, 500ms easeOutCubic
- Color transition: Green→Amber→Red fills smoothly as percentage increases
- Exceeded state: Shake animation (±2px) when budget exceeded, 100ms linear, 3 cycles

**NudgeCard & AnomalyCard:**
- Entrance: Fade in from 0→100%, slide up 16dp, 300ms easeOutCubic
- Dismiss swipe: Drag right or left → opacity fade to 0, slide exit, 250ms easeIn
- Action button press: Ripple effect + subtle scale, 200ms easeOut

**All Buttons:**
- Press ripple: Circle expands from tap point, 300ms easeOut, opacity 20%→0%
- Focus ring: Animated border highlight, 2dp stroke, `--color-primary` at 60% opacity
- Disabled: Opacity 50%, no ripple, non-interactive

**Loading Skeletons:**
- Pulse pattern: Opacity 60%→40%→60%, 1.2s easeInOut, continuous
- Shimmer (optional): Subtle left-to-right gradient sweep for premium feel

### 1.6. Dark Theme Refinement & Accessibility

#### Background & Surface Color Strategy
The app uses a dark theme with carefully chosen surface colors to create visual hierarchy and prevent eye fatigue.

| Element | Color | Hex | Contrast Ratio | Usage |
|---------|-------|-----|-----------------|-------|
| App background | Base dark | #121212 | — | Base canvas, full-screen fills |
| Card/surface (L1) | Elevated | #1E1E1E | 1.3:1 on bg | Default cards, panels |
| Section dividers | Subtle | #181818 | 1.1:1 on bg | Soft separation between sections |
| Elevated overlay | Light surface | #2A2A2A | 2:1 on bg | Modal backdrop fill, FAB background |
| Text on dark (primary) | White | #FFFFFF | 20:1 on all | Headlines, primary labels |
| Text on dark (secondary) | Grey | #B0B0B0 | 12:1 on all | Body text, descriptions |
| Text on dark (tertiary) | Dark grey | #757575 | 4.5:1 on surface | Captions, metadata (meets WCAG AA) |

**Accessibility Compliance:**
- All text meets WCAG AA contrast minimum (4.5:1 for body text, 3:1 for large text)
- Non-text elements (icons, borders, dots) meet Level AA in critical contexts
- Tested on multiple dark theme implementations (iOS Dark Mode, Android Q+, Material You)

#### Surface Color Differentiation (Optional Refinement)
For future versions, consider subtle color-coding of surfaces by functional area:

| Area | Surface Tint | Hex | Brightness | Usage |
|------|-------------|-----|------------|-------|
| Insights/Alerts | Subtle red tint | #1D1618 | -3% | AnomalyCard, anomaly sections |
| Community | Subtle green tint | #181D18 | -2% | Community Grid sections, leaderboard |
| Actions | Subtle blue tint | #18191F | -2% | Quick action chips, buttons |

**Current state:** Base surfaces (#1E1E1E, #181818) are neutral. Tinting is optional for phase 2.

#### Skeleton Loaders & Loading States
Skeleton loaders guide the eye toward incoming data without harsh transitions.

| Technique | Implementation | Animation | Duration | Notes |
|-----------|---|---|---|---|
| Opacity pulse | Opacity: 60% → 40% → 60% | easeInOut | 1.2s loop | Subtle, hypnotic effect |
| Shimmer (optional) | Left-to-right gradient sweep | Linear | 2.0s loop | Adds perceived activity; premium feel |
| Color shift | Neutral → slightly lighter | easeInOut | 1.0s loop | Minimal distraction |

**Recommended for MVP:** Opacity pulse only (simpler, equally effective).

#### Dark Theme Contrast Checklist
- [ ] All body text: minimum 4.5:1 contrast (WCAG AA)
- [ ] Large text (18sp+): minimum 3:1 contrast (WCAG AAA large text)
- [ ] Icon + background: minimum 3:1 contrast (visual elements)
- [ ] Focus rings: 3:1 minimum on --color-text-primary or --color-primary
- [ ] Text on colored backgrounds: Verify contrast on --color-primary, --color-warning, --color-danger
- [ ] Disabled states: 50% opacity meets contrast requirements for interactive elements

---

## 2. Navigation Architecture

### Bottom Tab Bar

Permanent bar at screen bottom. 4 tabs, white icons on dark background, active tab highlighted in `--color-primary`.

| Position | Icon | Label | Route | Screen |
|----------|------|-------|-------|--------|
| 1 | `house` | Home | `/dashboard` | Dashboard |
| 2 | `lightbulb` | Insights | `/insights` | AI Insights |
| 3 | `users-three` | Community | `/community` | Community Grid |
| 4 | `user` | Profile | `/settings` | Settings |

### Screen Hierarchy

```
BottomTabBar (always visible on all 4 root screens)
│
├── /dashboard
│   ├── Tap DeviceCard → /plug-detail/{id}  (push, with back arrow)
│   └── Tap "+ Add Plug" → PairingFlow (modal, slides up from bottom)
│
├── /insights
│   └── Tap AnomalyCard → /plug-detail/{id}  (push, with back arrow)
│
├── /community
│   ├── Tap "Derma" → DonationModal (modal, slides up from bottom)
│   └── Tap StreakBadgeRow → /badges (push, with back arrow)
│
├── /badges
│   └── Badge Collection (push from /community or /settings profile)
│
└── /settings
    ├── Tap plug in "Plugs Saya" → /plug-detail/{id}  (push, with back arrow)
    └── Tap Level/XP row → /badges (push, with back arrow)
```

### Back Navigation

- **Push (Plug Detail):** ← back arrow in app bar returns to previous screen
- **Modal (Pairing, Donation):** ✕ close button top-right or swipe down to dismiss
- **Tab switch:** No back stack between tabs. Tapping a tab always shows its root

### Deep Link Rule

When user taps an anomaly push notification, app opens directly to `/plug-detail/{id}` with the anomaly banner pre-expanded at top. Back arrow returns to Dashboard (no Insights screen in the stack).

---

## 3. Dashboard Screen

**Route:** `/dashboard`
**Purpose:** Real-time overview of all rooms and plugs. Home of the app. First screen after onboarding.

### Wireframe

```
┌──────────────────────────────────┐
│  Good morning, Aisyah           🔔│  ← App Bar (48dp height)
│  Home                          (2)│    Bell shows anomaly badge count
├──────────────────────────────────┤
│ ┌────────────────────────────────┐ │
│ │  420 W          RM 1.37 today │ │  ← Summary Card (height: 80dp)
│ │  ▲ 12% vs yesterday            │ │    Left: total watts + cost
│ │                                │ │    Right: trend arrow + percentage
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ [🚪 Leaving Home] [All Off] [All On]│  ← Quick Action Chips
│                                    │    Horizontal scrollable row
├──────────────────────────────────┤
│ 🏠 Living Room               180W │  ← Room header (tappable, collapses)
│ ┌──────┐ ┌──────┐ ┌──────┐       │
│ │  🖥️   │ │  💡  │ │  ❄️   │       │  ← DeviceCards
│ │  TV  │ │ Lamp │ │  AC  │       │    Horizontal scrollable row
│ │ 12 W │ │  8 W │ │160 W│       │
│ │  ●   │ │  ●   │ │  ●   │       │    ● = green dot = online/normal
│ │ [ON] │ │ [ON] │ │ [ON] │       │
│ └──────┘ └──────┘ └──────┘       │
├──────────────────────────────────┤
│ 🍽️ Kitchen                240 W │  ← Room header
│ ┌──────┐ ┌──────┐                │
│ │  🧊   │ │  🍚   │                │
│ │Fridge│ │RiceCk│                │
│ │ 80 W │ │160 W│                │
│ │  ●   │ │  🟠   │                │    🟠 = orange dot = high (200-800W)
│ │ [ON] │ │ [ON] │                │
│ └──────┘ └──────┘                │
├──────────────────────────────────┤
│ 🛏️ Bedroom                  8 W │  ← Room header
│ ┌──────┐                         │
│ │  🔌   │                         │
│ │Charger│                        │
│ │  8 W │                         │
│ │  ●   │                         │
│ │ [ON] │                         │
│ └──────┘                         │
├──────────────────────────────────┤
│ [🏠 Home] [💡] [👥 Comm] [👤 Prof]│  ← Bottom Nav
└──────────────────────────────────┘
```

### Component Breakdown

| Order | Component | Data Source | States |
|-------|-----------|-------------|--------|
| 1 | App Bar | User name, badge count from NudgeEngine | Normal, badge=0, badge=5+ |
| 2 | Summary Card | Sum of all plug wattages live, daily cost calculator | Normal, all plugs offline (show "--"), loading skeleton |
| 3 | Quick Action Chips | Static actions | Normal |
| 4 | Room Section | Plug list grouped by room | Expanded, collapsed |
| 5 | DeviceCard (per plug) | MQTT `tele/{id}/SENSOR` | Online-normal, online-high (orange dot), online-critical (red dot), offline, loading skeleton |
| 6 | "Add Plug" button | N/A | Visible when < 1 plug paired |

### Interactions

| Trigger | Action |
|---------|--------|
| Tap DeviceCard | Push to `/plug-detail/{id}` |
| Tap DeviceCard toggle | Publish `cmnd/{id}/POWER TOGGLE` → animate toggle instantly, reconcile on MQTT response |
| Tap "Leaving Home" chip | Publish OFF to all non-always-on plugs. Show summary toast. |
| Tap "All Off" chip | Publish OFF to all plugs. Show confirmation toast. |
| Tap "All On" chip | Publish ON to all plugs. Show confirmation toast. |
| Tap Room header | Collapse/expand room section |
| Tap "+ Add Plug" | Open Pairing Flow modal |
| Pull down | Refresh all MQTT states (re-request `stat/` for every plug) |
| Tap bell icon | Push to Insights screen, scrolled to anomaly section |

### States

**Empty (0 plugs):**
```
┌──────────────────────┐
│ Good morning, Aisyah │
├──────────────────────┤
│                      │
│       🔌             │
│  No plugs paired     │
│                      │
│  [ + Add Your First  │
│     Smart Plug  ]    │
│                      │
└──────────────────────┘
```

**All plugs offline:**
- Summary Card shows "-- W" and "-- today"
- Each DeviceCard: grey dot, "OFFLINE" label, toggle disabled (grey)
- "Leaving Home" chip disabled

**Loading skeleton:**
- Summary Card: pulsing grey placeholder rectangles
- DeviceCards: pulsing placeholder cards (same shape, no data)

**Error state:**
- Banner at top of Dashboard: "Connection lost. Reconnecting..." with animated dots
- Last-known values remain visible but fade to 60% opacity
- Pull-to-refresh retries connection

---

## 4. Plug Detail Screen

**Route:** `/plug-detail/{id}`
**Purpose:** Deep view of a single plug. Live meter, charts, schedules, budget, controls.

### Wireframe

```
┌──────────────────────────────────┐
│ ←  AC Bedroom             ✏️     │  ← App bar with back arrow + edit
├──────────────────────────────────┤
│           ┌───────────┐          │
│           │           │          │  ← PowerGauge (circular, 180dp)
│           │   160 W   │          │    Arc: 0-2400W
│           │ RM 0.35   │          │    Colors: green ≤200W, amber ≤800W,
│           │  per day   │          │            red >800W
│           └───────────┘          │    Center: large mono readout
├──────────────────────────────────┤
│  240 V    0.67 A   2.4 kWh  48 RM│  ← Stat Row
│  Voltage  Current  Today    Month│    4 equal columns, iconless
├──────────────────────────────────┤
│ ┌──────────────────────────────┐ │
│ │    [7 Days]  [30 Days]       │ │  ← Consumption Chart
│ │  W▲                          │ │    Toggle for range
│ │ 200│     ╱╲                  │ │    Line chart with filled area
│ │    │    ╱  ╲      ╱╲        │ │    X: time, Y: watts
│ │ 100│╲  ╱    ╲    ╱  ╲   ╱╲ │ │    Horizontal grid lines
│ │    │ ╲╱      ╲╲╱    ╲╲╱  ╲│ │    Tap on point → tooltip
│ │   0└──────────────────────  │ │
│ │   Mon  Tue  Wed  Thu  Fri   │ │
│ └──────────────────────────────┘ │
├──────────────────────────────────┤
│ Schedules (2)               + Add│  ← Schedule section header
│ ┌──────────────────────────────┐ │
│ │ Weekdays  ⏰ 8PM – 6AM  [ON]│ │  ← Schedule card
│ └──────────────────────────────┘ │    Swipe left to delete
│ ┌──────────────────────────────┐ │
│ │ Weekend   ⏰ 9AM – 11PM  [ON]│ │
│ └──────────────────────────────┘ │
├──────────────────────────────────┤
│ ┌──────────────────────────────┐ │
│ │ Budget                       │ │  ← Budget section
│ │ ▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░ RM 48/60 │ │    Green 0-70%, amber 70-90%,
│ │ 80% used · 12 days remaining │ │          red 90-100%
│ └──────────────────────────────┘ │
├──────────────────────────────────┤
│ 🦇 Vampire auto-off          [ON]│  ← Toggle chips
│ 📌 Always-on                 [ON]│    Each: label + switch
├──────────────────────────────────┤
│         [ ●━━━━━━━━━  ON  ]      │  ← Large ON/OFF toggle button
│                                  │    Full-width, 56dp tall
│                                  │    Green when ON, grey when OFF
└──────────────────────────────────┘
```

### Component Breakdown

| Order | Component | Data Source | States |
|-------|-----------|-------------|--------|
| 1 | App Bar | Plug name, room | Normal, editing name (inline text field) |
| 2 | PowerGauge | Live watts from MQTT, daily cost from calculator | Normal (green arc), warning (amber arc, 200-800W), critical (red arc, >800W), offline (grey, "-- W") |
| 3 | Stat Row | V, A, kWh_today, RM_month from MQTT + local accumulator | Normal, offline (all "--") |
| 4 | Consumption Chart | 7-day or 30-day query from TimescaleDB | 7-day data, 30-day data, loading skeleton, empty ("No data yet — check back tomorrow") |
| 5 | Schedule List | Local SQLite `schedules` table | Has schedules, no schedules ("No schedules set. Tap + to add your first.") |
| 6 | Budget Bar | Local accumulator vs. user-set budget | Not set ("Set a budget"), on track (green), warning (amber, 70-90%), exceeded (red, >100%) |
| 7 | Vampire Toggle | Local plug config | ON, OFF |
| 8 | Always-On Toggle | Local plug config | ON, OFF |
| 9 | ON/OFF Button | Live relay state from MQTT `stat/{id}/POWER` | ON (green fill), OFF (grey fill), toggling (spinner) |

### Interactions

| Trigger | Action |
|---------|--------|
| Tap back arrow | Pop to previous screen |
| Tap edit (✏️) | Inline rename: title becomes text field, keyboard opens, "Done" replaces ✏️ |
| Tap ON/OFF button | Publish `cmnd/{id}/POWER TOGGLE`. Button shows spinner until `stat/` response returns. |
| Tap chart toggle (7 Days / 30 Days) | Reload chart with selected range. Active toggle highlighted in `--color-primary`. |
| Tap "+ Add" (schedules) | Open schedule creation sheet (time picker, day selector, repeat toggle) |
| Tap schedule toggle | Enable/disable that schedule row. Toggle animates immediately. |
| Swipe schedule left | Show red "Delete" action. Tap to confirm. |
| Tap Budget bar | Open budget edit sheet (amount input in RM, "Save" button) |
| Tap Vampire toggle | Toggle vampire auto-off for this plug |
| Tap Always-on toggle | Mark/unmark plug as always-on (excluded from "Leaving Home" mass-off) |
| Tap appliance icon | Open icon picker grid (24 icons: TV, fridge, AC, lamp, fan, charger, microwave, kettle, etc.) |

### States

**Anomaly active state:**
- Thin red banner below app bar: "⚠️ This device is consuming 35% more than usual. [View Details]"
- PowerGauge arc shifts to red zone
- Consumed wattage label turns red

**Schedule active state:**
- If currently within an active schedule window, a subtle green pill shows below the plug name: "Scheduled — on until 6 AM"

**Plug offline state:**
- PowerGauge: grey arc, "-- W" center readout
- Stat Row: all "--"
- ON/OFF button disabled (grey)
- Chart: last known data shown, "Last updated: 2 hours ago" caption

**Vampire mode triggered:**
- Thin amber banner: "Vampire cutoff triggered — drawing 3W for 30+ min. [Undo]"
- ON/OFF button shows OFF
- "Undo" tap restores relay to ON

---

## 5. Pairing Flow

**Presentation:** Slides up as a bottom modal sheet, 90% screen height.
**Purpose:** Add a new smart plug. Entirely simulated — no real BLE or MQTT hardware needed.

### Step 1: Scanning

```
┌──────────────────────────────────┐
│ Add a Plug                   ✕   │
├──────────────────────────────────┤
│  Searching for nearby plugs...   │
│         ⟳ spinning               │
│                                  │
│  Make sure your plug is plugged  │
│  in and the LED is pulsing white.│
│                                  │
│ [Scan Again]                     │
└──────────────────────────────────┘
```

**Simulated scan duration:** 2 seconds, then auto-show results.

### Step 2: Device List

```
┌──────────────────────────────────┐
│ Select Your Plug              ✕  │
├──────────────────────────────────┤
│ ┌────────────────────────────────┐│
│ │ 🔌 Sonoff-S31-A3F2     ████   ││  ← Signal strength bars
│ │ Living Room                   ││    "Connect" button on right
│ │                        [Connect]│
│ └────────────────────────────────┘│
│ ┌────────────────────────────────┐│
│ │ 🔌 Plug-7B12           ███    ││
│ │ Kitchen                       ││
│ │                        [Connect]│
│ └────────────────────────────────┘│
│ ┌────────────────────────────────┐│
│ │ 🔌 Tasmota-9F01       ██      ││
│ │ Unknown                       ││
│ │                        [Connect]│
│ └────────────────────────────────┘│
│                                  │
│ [Scan Again]                     │
└──────────────────────────────────┘
```

**Mock device list:** Always show exactly 3-4 devices with these names. Signal bars are static decorative elements.

### Step 3: Wi-Fi Setup

```
┌──────────────────────────────────┐
│ Connect to Wi-Fi              ✕  │
├──────────────────────────────────┤
│ Wi-Fi Network                    │
│ ┌────────────────────────────────┐│
│ │ MyHomeWiFi_5G             (▼) ││  ← Pre-filled from phone's Wi-Fi
│ └────────────────────────────────┘│
│                                  │
│ Password                         │
│ ┌────────────────────────────────┐│
│ │ ••••••••••               👁️   ││  ← Password field with toggle
│ └────────────────────────────────┘│
│                                  │
│ [        Connect Plug        ]   │  ← Large primary button
└──────────────────────────────────┘
```

**SSID:** Pre-filled with "MyHomeWiFi_5G" (mock). Dropdown shows 2 other mock networks: "MyHomeWiFi_2G", "GuestNetwork".

### Step 4: Progress Animation

```
┌──────────────────────────────────┐
│ Setting Up Your Plug         ✕   │
├──────────────────────────────────┤
│                                  │
│  ✓  Connecting to Wi-Fi          │  ← Checkmarks appear
│  ✓  Configuring MQTT broker      │    sequentially with
│  ⟳  Testing relay...             │    1-second delay each
│     Assigning plug ID            │
│     Syncing with your account    │
│                                  │
│ ┌────────────────────────────────┐│
│ │     Progress bar (60%)         ││  ← Animated fill
│ └────────────────────────────────┘│
│                                  │
└──────────────────────────────────┘
```

**Duration:** ~4 seconds total. All steps auto-complete. Last checkmark triggers transition.

### Step 5: Name & Assign

```
┌──────────────────────────────────┐
│ Almost Done                  ✕   │
├──────────────────────────────────┤
│ Name                             │
│ ┌────────────────────────────────┐│
│ │ Living Room AC                 ││  ← Pre-filled suggestion
│ └────────────────────────────────┘│
│                                  │
│ Room                             │
│ ┌────────────────────────────────┐│
│ │ Living Room               (▼) ││  ← Dropdown
│ └────────────────────────────────┘│
│                                  │
│ Appliance Type                   │
│ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐  │
│ │ ❄️│ │ 🖥️│ │ 💡│ │ 🔌│ │ 🍚│  │  ← Icon grid, 4 per row
│ │ AC│ │ TV│ │Lamp│ │Chgr│ │Rice│  │    Scrollable if >8 options
│ └───┘ └───┘ └───┘ └───┘ └───┘  │
│ ┌───┐ ┌───┐ ┌───┐                │
│ │ 🧊│ │ 🌀│ │ 📺│                │
│ │Frdg│ │Fan│ │Micr│                │
│ └───┘ └───┘ └───┘                │
│                                  │
│ [        Finish Setup        ]   │
└──────────────────────────────────┘
```

### Step 6: Success

```
┌──────────────────────────────────┐
│         ✓                        │
│   Plug Added Successfully        │
│                                  │
│   Living Room AC is ready to     │
│   monitor and control.           │
│                                  │
│ [     Go to Dashboard       ]    │
└──────────────────────────────────┘
```

"Dismiss" or backswipe or "Go to Dashboard" closes the modal. New DeviceCard appears on Dashboard.

### Edge Case States

**Scan timeout (no plugs found):**
- "No plugs found nearby."
- "Make sure your Sonoff S31 is plugged in and the LED is pulsing white."
- "[Scan Again] [Cancel]"

**Connection failure (step 4):**
- Progress stops at the failed step.
- "✗ Could not connect to Wi-Fi. Check your password and try again."
- "[Try Again] [Cancel]"

**BLE permission denied (first-use only):**
- "Bluetooth permission is needed to discover nearby plugs."
- "[Open Settings] [Not Now]"

---

## 6. AI Insights Screen

**Route:** `/insights`
**Purpose:** Anomaly alerts, personalized nudges, and savings summary. The AI "brain" of the app.

### Wireframe

```
┌──────────────────────────────────┐
│ 💡 Insights                      │
├──────────────────────────────────┤
│ ⚠️ Anomalies                     │  ← Section header
│ ┌────────────────────────────────┐ │
│ │ 🧊  Fridge                ⚠️  │ │  ← AnomalyCard (critical)
│ │ Your fridge consumed 35% more  │ │    Red left-border accent
│ │ power yesterday. Estimated     │ │
│ │ impact: RM 18/month.           │ │
│ │ 2 hours ago                    │ │
│ │ [View Details] [Dismiss]       │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 🍚  Rice Cooker           🟠  │ │  ← AnomalyCard (warning)
│ │ Rice cooker energy pattern     │ │    Amber left-border accent
│ │ looks unusual. May indicate    │ │
│ │ a maintenance need.            │ │
│ │ Yesterday, 7:30 PM             │ │
│ │ [View Details] [Dismiss]       │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 💬 Suggestions                   │  ← Section header
│ ┌────────────────────────────────┐ │
│ │ 💡  Your AC has been running   │ │  ← NudgeCard
│ │ for 4 hours. Open windows?     │ │    Dismiss via swipe-left
│ │ It's 26°C outside.             │ │    or tap action button
│ │ just now                       │ │
│ │ [Open Schedule]                │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 📊  You used 18% less energy   │ │  ← NudgeCard (achievement)
│ │ this week. New personal        │ │    Green left-border accent
│ │ record!                        │ │
│ │ 3 days ago                     │ │
│ │ [View Stats]                   │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 💰 Savings Summary               │  ← Section header
│ ┌────────────────────────────────┐ │
│ │       ┌───────────┐            │ │
│ │       │  ○ 73%    │            │ │  ← Circular progress
│ │       │ RM 18.40  │            │ │    Center: "RM 18.40"
│ │       │  saved    │            │ │    Track: grey ring
│ │       └───────────┘            │ │    Fill: green arc
│ │  Goal: RM 25/month             │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ [🏠 Home] [💡] [👥 Comm] [👤 Prof]│
└──────────────────────────────────┘
```

### Component Breakdown

| Order | Component | Data Source | States |
|-------|-----------|-------------|--------|
| 1 | App Bar | "Insights" static title | Normal |
| 2 | Anomaly Section | NudgeEngine anomaly detection output | Has anomalies, no anomalies ("No anomalies detected. Your appliances are running smoothly ✅") |
| 3 | AnomalyCard | Anomaly data per device | Critical (red accent), warning (amber accent), read (reduced opacity), dismissed (removed from list) |
| 4 | Suggestions Section | NudgeEngine recommendation queue | Has nudges, no nudges ("No new suggestions right now.") |
| 5 | NudgeCard | Nudge data per event | Unread (full opacity), read (70% opacity), achievement (green accent), dismissed (removed) |
| 6 | Savings Summary | Local accumulator comparing current month vs. goal | Goal set (progress visible), no goal ("Set a monthly savings goal"), goal achieved ("🎉 Goal reached! RM 27.30 saved") |

### Interactions

| Trigger | Action |
|---------|--------|
| Tap "View Details" on AnomalyCard | Push to `/plug-detail/{id}` with anomaly banner expanded |
| Tap "Dismiss" on AnomalyCard | Fade out card. If last anomaly, show empty state. |
| Swipe NudgeCard left | Show red dismiss action. Tap to confirm. Card fades out. |
| Tap action button on NudgeCard | Executes suggested action (e.g., "Open Schedule" pushes to Plug Detail's schedule section) |
| Pull down | Refresh all sections |
| Tap savings goal | Open goal edit sheet (amount input in RM) |

### States

**No anomalies, no nudges (fresh install):**
```
┌────────────────────────────────┐
│ No anomalies detected          │
│ Your appliances are running    │
│ smoothly ✅                    │
│                                │
│ No new suggestions right now.  │
│ Check back tomorrow for        │
│ personalized tips.             │
│                                │
│ Set a monthly savings goal     │
│ to start tracking your impact. │
│ [Set Goal]                     │
└────────────────────────────────┘
```

**Anomaly banner on Dashboard (when navigated from notification):**
- Top of Plug Detail shows: "⚠️ This device is consuming 35% more than usual."
- Action: "Schedule maintenance check" or "Dismiss"

---

## 7. Community Grid Screen

**Route:** `/community`
**Purpose:** Neighborhood energy marketplace. Leaderboard, donations, community impact, gamification. Grounded in KL geography — real kawasan names, multi-ethnic persona names, BM-English bilingual content.

### Wireframe

```
┌──────────────────────────────────┐
│ 👥 Komuniti                    🔔│  ← App Bar
├──────────────────────────────────┤
│ ┌────────────────────────────────┐ │
│ │ 🏅 Badges & Streaks        ›  │ │  ← Streak + Badge Row (tappable → Badge Collection)
│ │ 🔥 7-Hari  🛡️ Jiran  🌙 Anak  │ │    Horizontal scrollable chips
│ │           🍃 Celik   ›        │ │    4 badge icons + names, right arrow for full collection
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ ┌────────────────────────────────┐ │
│ │ Kredit Tenaga Anda             │ │  ← Credits Card
│ │        14                      │ │    Large number
│ │    kredit tersedia             │ │
│ │  [     Derma Kredit      ]     │ │    Green button
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 🏆 Penderma Teratas — Julai      │  ← Leaderboard section
│ ┌────────────────────────────────┐ │
│ │ #1  👤 Kumar, Bangsar    22 cr│ │  ← KL neighborhood names
│ │     12 kpd 3 jiran — PPR      │ │    Multi-ethnic persona names
│ │     Kerinchi                   │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ #2  👤 Mei Ling, Cheras  16 cr│ │
│ │     9 kpd 2 jiran             │ │
│ └────────────────────────────────┘ │
│ ┌════════════════════════════════┐ │
│ ║ #3  👤 Anda (Aisyah)    14 cr ║ │  ← Highlighted row (your position)
│ ║     6 kpd 1 jiran — PP Lbg   ║ │    Green border, slightly elevated
│ ║     Subang                    ║ │
│ └════════════════════════════════┘ │
│ ┌────────────────────────────────┐ │
│ │ #4  👤 Raj, Damansara     9 cr │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ #5  👤 Fatimah, PPR K'chi 6 cr│ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ ⚡ Cabaran Langsung: Haze Shield  │  ← Neighborhood Challenge (visible during active challenge)
│ ┌────────────────────────────────┐ │
│ │ 🏘️ TTDI (38 rumah) ▓▓▓▓▓▓▓▓ 72%│ │
│ │ 🏘️ Damansara (29)   ▓▓▓▓▓░░░ 58%│ │
│ │                                │ │
│ │ 4 hari lagi · TTDI mendahului! │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 🌏 Impak Komuniti                │  ← Impact section
│ ┌────────────────────────────────┐ │
│ │ Bulan ini, kawasan KL kita     │ │  ← KL-specific impact stats
│ │ jimat:                         │ │
│ │                                │ │
│ │         1,240 kWh              │ │
│ │                                │ │
│ │ Setara dengan:                 │ │
│ │ 🏠 62 unit PPR sehari          │ │
│ │ ❄️ 11 AC selama 8 jam          │ │
│ │ 🚦 58 lampu Jln Bukit Bintang  │ │
│ │                                │ │
│ │ 84 kredit dikongsi ke 4 kawasan│ │
│ │                 [📤 Kongsi]    │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ [🏠] [💡] [👥 Komuniti] [👤 Profil]│
└──────────────────────────────────┘
```

### Component Breakdown

| Order | Component | Data Source | States |
|-------|-----------|-------------|--------|
| 1 | App Bar | "Komuniti" — BM title | Normal; bell shows anomaly count |
| 2 | Streak + Badge Row | User's active streak + 4 most recent badges (mock data) | Has badges, no badges ("Belum ada lencana. Jimat tenaga untuk dapatkan!") |
| 3 | Credits Card | User's current credit balance | Has credits, no credits ("Anda belum ada kredit. Kurangkan penggunaan bawah baseline untuk mula dapatkan.") |
| 4 | Leaderboard | Top donors from local state (mock KL data) | Has entries, empty ("Tiada derma bulan ini. Jadilah yang pertama!") |
| 5 | Neighborhood Challenge | Active monthly challenge between two kawasan (mock data) | Active challenge, no active challenge (hidden), challenge ended ("TTDI menang! 20% multiplier untuk Ogos 🔥") |
| 6 | Impact Card | Aggregate community stats (mock KL data) | Normal |

### Interactions

| Trigger | Action |
|---------|--------|
| Tap Streak + Badge Row | Push to Badge Collection screen (`/badges`) |
| Tap "Derma Kredit" | Open Donation Modal |
| Tap any leaderboard row | Highlight row briefly (no further action) |
| Tap Neighborhood Challenge | Expand to full challenge detail (inline) |
| Tap "Kongsi" on Impact Card | Simulate share action (mock system share sheet with KL-copy prefill) |

### Streak + Badge Row Widget Spec

**Component:** `StreakBadgeRow`
**Location:** Top of Community Grid, below App Bar, above Credits Card
**Layout:** Horizontal scrollable row of compact chips. Each chip = 1 badge icon + short name.
**Height:** 44dp (compact, non-intrusive)

```
┌────────────────────────────────────────────┐
│ 🔥 7-Hari  │ 🛡️ Jiran Terbaik  │ 🌙 A.Bulan  │ 🍃 Celik Tenaga  │ › │
└────────────────────────────────────────────┘
   ↑ 48dp wide    ↑ 72dp wide       ↑ 56dp      ↑ 72dp              ↑ chevron
   Streak chip    Badge chip        Badge chip  Badge chip           (tap → /badges)
```

**Streak chip (`🔥 7-Hari`):**
- Always visible if streak > 0, leftmost position
- Shows fire emoji + current streak count + "Hari"
- Pulsing opacity animation (1.2s loop) when streak is active
- If streak broken: greyed out, shows "🔥 0" with subtle shake animation on first view

**Badge chips:**
- Shows 4 most recently earned badges
- Each chip: emoji icon + Malay badge name (truncated to 12 chars)
- Tap entire row or chevron → `/badges`

**States:**
- No badges yet: Row shows single chip "🔒 4 lencana untuk dibuka" (grey, disabled)
- 1–3 badges: Shows earned badges + remaining grey placeholder chips
- 4+ badges: Shows 4 most recent + "›" chevron for full collection

### Neighborhood Challenge Widget Spec

**Component:** `NeighborhoodChallengeCard`
**Location:** Between leaderboard and impact card
**Visibility:** Only when an active monthly challenge exists between two kawasan
**Height:** 120dp (compact card, expands to 200dp on tap)

**Data model (mock for prototype):**
```dart
{
  challengeName: "Haze Shield",
  month: "Julai",
  neighborhoodA: "TTDI",
  homesA: 38,
  progressA: 0.72,  // 72% of target
  neighborhoodB: "Damansara",
  homesB: 29,
  progressB: 0.58,  // 58% of target
  daysRemaining: 4,
  leadingNeighborhood: "TTDI"
}
```

**Collapsed view:**
```
┌──────────────────────────────────────────┐
│ ⚡ CABARAN: Haze Shield (Julai)          │
│                                            │
│ 🏘️ TTDI (38)    ▓▓▓▓▓▓▓▓░░  72%          │
│ 🏘️ Damansara (29) ▓▓▓▓▓░░░░░  58%         │
│                                            │
│ 4 hari lagi  ·  TTDI mendahului!          │
└──────────────────────────────────────────┘
```

**Expanded view (on tap):**
```
┌──────────────────────────────────────────┐
│ ⚡ CABARAN: Haze Shield (Julai)     [✕]  │
│                                            │
│ 🏘️ TTDI (38 rumah)                       │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░ 72%  (864/1200 kWh) │
│ Penderma teratas: Raj (12 kredit)         │
│ 🔥 Rantaian: 14 hari                      │
│                                            │
│ 🏘️ Damansara Heights (29 rumah)           │
│ ▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░ 58%  (696/1200 kWh) │
│ Penderma teratas: Mei Ling (9 kredit)     │
│ 🔥 Rantaian: 8 hari                       │
│                                            │
│ ⏰ Tamat dalam 4 hari  |  Hadiah: Lencana │
│ "Jaguh Bulan Ini" + 20% multiplier        │
│                      [Kongsi Cabaran]      │
└──────────────────────────────────────────┘
```

**States:**
- **Active challenge** → Shows collapsed card (compact) or expanded (on tap)
- **Challenge ended — user's kawasan won** → "🏆 TTDI menang! 20% multiplier untuk Ogos. Kongsi kejayaan anda →"
- **Challenge ended — user's kawasan lost** → "Damansara menang Julai. Sertai cabaran Ogos: Merdeka 55 🇲🇾"
- **No active challenge** → Widget hidden entirely

### Donation Modal (KL-Updated)

```
┌──────────────────────────────────┐
│ Derma Kredit                 ✕   │
├──────────────────────────────────┤
│ Pilih penerima                   │
│ ┌────────────────────────────────┐ │
│ │ 🏠  Tabung Komuniti           │ │  ← Default, pre-selected
│ │     Diagih ke PPR sekitar KL  │ │
│ │     secara automatik          │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 🏠  PPR Kerinchi              │ │
│ │     3 keluarga sedang tunggu  │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 🏠  PPR Lembah Subang         │ │
│ │     2 keluarga sedang tunggu  │ │
│ └────────────────────────────────┘ │
│                                  │
│ Jumlah                           │
│ ┌────────────────────────────────┐ │
│ │  ●─────○──────○──────○──────○ │ │
│ │  0     3      6      9     14 │ │
│ │          5 kredit             │ │
│ └────────────────────────────────┘ │
│                                  │
│ Impak:                           │
│ ~RM 1.09 bil elektrik            │
│ Derma ke PPR = 2x XP lencana    │
│                                  │
│ [     Sahkan Derma          ]    │
└──────────────────────────────────┘
```

**After confirm — success animation (KL copy):**

```
┌──────────────────────────────────┐
│                                  │
│           🎉 🎉 🎉               │
│         confetti                 │
│                                  │
│   5 kredit didermakan!           │
│                                  │
│   Anda bantu ringankan           │
│   ~RM 1.09 untuk PPR Kerinchi.   │
│                                  │
│   🛡️ 3 lagi untuk lencana        │
│      Jiran Terbaik!              │
│                                  │
│   [Lihat Impak] [Selesai]        │
└──────────────────────────────────┘
```

### States

**No credits (first-time user):**
- Credits Card shows "0 kredit tersedia"
- "Derma Kredit" button disabled (grey)
- Info text: "Jimat tenaga untuk dapatkan kredit. Baseline anda sedang dikira — semak semula dalam 7 hari."

**Empty leaderboard:**
- "Tiada derma bulan ini. Jadilah yang pertama dan tuntut tempat #1!"

---

## 7.5. Badge Collection Screen

**Route:** `/badges`
**Purpose:** Full collection view of all earned and locked badges. Accessed by tapping the Streak + Badge Row on the Community Grid screen.

### Wireframe

```
┌──────────────────────────────────┐
│ ← Koleksi Lencana                │
├──────────────────────────────────┤
│ Tahap: Celik Tenaga (Lv.3)       │  ← Level badge + XP bar
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓░░░  1,840 / 3,000 XP│
│ 560 XP lagi ke Wira Hijau        │
├──────────────────────────────────┤
│ Diperolehi (4)                   │  ← Earned section
│ ┌────────┐ ┌────────┐ ┌────────┐ │
│ │   🔥   │ │   🛡️   │ │   🌙   │ │  ← 3-column grid
│ │7-Hari │ │ Jiran  │ │  Anak  │ │    Badge icon + name
│ │Rantaian│ │Terbaik │ │ Bulan  │ │    + unlocked date
│ │  2/7   │ │ 15 krd │ │  Hero  │ │
│ │  ✓     │ │  ✓     │ │  ✓     │ │
│ └────────┘ └────────┘ └────────┘ │
│ ┌────────┐                       │
│ │   🍃   │                       │
│ │ Celik  │                       │
│ │ Tenaga │                       │
│ │  Lv.3  │                       │
│ │  ✓     │                       │
│ └────────┘                       │
├──────────────────────────────────┤
│ Terkunci (8)                     │  ← Locked section
│ ┌────────┐ ┌────────┐ ┌────────┐ │
│ │   🌱   │ │   🇲🇾  │ │   ☕   │ │  ← Greyed out (opacity 40%)
│ │ Taugeh │ │Merdeka │ │Kopitiam│ │    Shows unlock condition
│ │Champion│ │ Saver  │ │Regular │ │    below icon
│ │  RM 50 │ │  55 kWh│ │30 hari │ │
│ └────────┘ └────────┘ └────────┘ │
│ ┌────────┐ ┌────────┐ ┌────────┐ │
│ │   🫓   │ │   🚗   │ │   🚆   │ │
│ │ Mamak  │ │ Balik  │ │  LRT   │ │
│ │ Squad  │ │Kampung │ │Warrior │ │
│ │10 mlm  │ │Hujung  │ │ 30 hrg │ │
│ └────────┘ └────────┘ └────────┘ │
│ ┌────────┐ ┌────────┐           │
│ │   🏅   │ │   👑   │           │
│ │  PPR   │ │ Dato'  │           │
│ │Champion│ │ Jimat  │           │
│ │ Top 3  │ │Lv.7 25k│           │
│ └────────┘ └────────┘           │
├──────────────────────────────────┤
│ [🏠] [💡] [👥 Komuniti] [👤 Profil]│
└──────────────────────────────────┘
```

### Component Breakdown

| Order | Component | Data Source | States |
|-------|-----------|-------------|--------|
| 1 | App Bar | "Koleksi Lencana" + back arrow | Normal |
| 2 | Level Card | User's level, XP, progress bar, next level name | Normal |
| 3 | Earned Grid | 4 earned badges in 3-column grid | Has badges, empty ("Belum ada lencana. Mulakan perjalanan jimat tenaga anda!") |
| 4 | Locked Grid | 8 locked badges in 3-column grid, greyed at 40% opacity, with unlock condition | Normal (always shown) |

### Badge Data Model (For Prototype Mock)

```dart
class Badge {
  String id;
  String nameBM;        // Malay name (e.g., "Anak Bulan Hero")
  String nameEN;        // English name (e.g., "Ramadan Champion")
  String emoji;         // Single emoji icon
  String unlockCondition; // e.g., "30 hari bawah baseline semasa Ramadan"
  bool isUnlocked;
  DateTime? unlockedDate;
}

// Prototype mock data
List<Badge> mockEarnedBadges = [
  Badge(id: 'streak', nameBM: '7-Hari Rantaian', nameEN: '7-Day Streak', emoji: '🔥', unlockCondition: '7 hari berturut-turut bawah baseline', isUnlocked: true, unlockedDate: DateTime(2026, 7, 14)),
  Badge(id: 'jiran', nameBM: 'Jiran Terbaik', nameEN: 'Best Neighbor', emoji: '🛡️', unlockCondition: 'Derma 15+ kredit ke PPR', isUnlocked: true, unlockedDate: DateTime(2026, 7, 8)),
  Badge(id: 'bulan', nameBM: 'Anak Bulan Hero', nameEN: 'Ramadan Champion', emoji: '🌙', unlockCondition: '30 hari bawah baseline semasa Ramadan', isUnlocked: true, unlockedDate: DateTime(2026, 4, 29)),
  Badge(id: 'celik', nameBM: 'Celik Tenaga', nameEN: 'Energy Literate', emoji: '🍃', unlockCondition: 'Capai Tahap 3', isUnlocked: true, unlockedDate: DateTime(2026, 6, 20)),
];

List<Badge> mockLockedBadges = [
  Badge(id: 'taugeh', nameBM: 'Taugeh Champion', nameEN: 'Bean Sprout Champ', emoji: '🌱', unlockCondition: 'Jimat RM 50+ sebulan', isUnlocked: false),
  Badge(id: 'merdeka', nameBM: 'Merdeka Saver', nameEN: 'Merdeka Saver', emoji: '🇲🇾', unlockCondition: 'Jimat 55 kWh semasa Ogos', isUnlocked: false),
  Badge(id: 'kopitiam', nameBM: 'Kopitiam Regular', nameEN: 'Kopitiam Regular', emoji: '☕', unlockCondition: '30 hari guna luar waktu puncak', isUnlocked: false),
  Badge(id: 'mamak', nameBM: 'Mamak Squad', nameEN: 'Mamak Squad', emoji: '🫓', unlockCondition: '10 malam jimat 8PM-12AM', isUnlocked: false),
  Badge(id: 'balik', nameBM: 'Balik Kampung', nameEN: 'Balik Kampung', emoji: '🚗', unlockCondition: 'Auto-tutup semasa minggu cuti perayaan', isUnlocked: false),
  Badge(id: 'lrt', nameBM: 'LRT Warrior', nameEN: 'LRT Warrior', emoji: '🚆', unlockCondition: '30 hari guna LRT untuk ulang-alik', isUnlocked: false),
  Badge(id: 'ppr', nameBM: 'PPR Champion', nameEN: 'PPR Champion', emoji: '🏅', unlockCondition: 'Top 3 Liga Rumah Pangsa', isUnlocked: false),
  Badge(id: 'dato', nameBM: 'Dato\' Jimat', nameEN: 'Sir Saves-a-Lot', emoji: '👑', unlockCondition: 'Capai Tahap 7 (25,000 XP)', isUnlocked: false),
];
```

### States

**Empty earned (first visit):**
- "Belum ada lencana. Mulakan perjalanan jimat tenaga anda!"
- Locked grid shown in full (all 12 badges visible as locked)

**Badge unlocked notification:**
- Toast at bottom: "🎉 Lencana baru: Jiran Terbaik! Ketik untuk lihat."
- Tapping toast navigates to `/badges` with new badge pulsing glow animation

---


## 8. Settings Screen

**Route:** `/settings`
**Purpose:** Profile, plug management, data sharing preferences, notification controls.

### Wireframe

```
┌──────────────────────────────────┐
│ 👤 Profil                        │
├──────────────────────────────────┤
│ ┌────────────────────────────────┐ │
│ │ 👤  Aisyah Binti Rahman        │ │  ← Profile section
│ │     aisyah.r@email.com         │ │
│ │ 📍  12, Jalan Bahagia, TTDI    │ │
│ │                                │ │
│ │ 🏅 Celik Tenaga (Tahap 3)      │ │  ← Level badge + XP progress
│ │ ▓▓▓▓▓▓▓▓▓▓▓░░░ 1,840 / 3,000  │ │    Compact progress bar
│ │ 560 XP lagi ke Wira Hijau  ›   │ │    Tappable → /badges
│ │                    [Edit Profil]│ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 🔌 Plugs Saya                      │
│ ┌────────────────────────────────┐ │
│ │ ❄️  AC Bedroom          ●     │ │  ← Plug row
│ │     Living Room         [ON]  │ │    Online dot (green/grey)
│ └────────────────────────────────┘ │    Tap → Plug Detail
│ ┌────────────────────────────────┐ │
│ │ 🖥️  TV                   ●     │ │
│ │     Living Room         [ON]  │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 🧊  Fridge               ●     │ │
│ │     Kitchen             [ON]  │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 📌 Sentiasa-On                   │  ← Editable list
│ ┌────────────────────────────────┐ │
│ │ 🧊  Fridge                    │ │  ← Each with swipe-to-remove
│ │ 📶  Router                    │ │    "Add" button at bottom
│ │ 🐠  Fish Tank                 │ │
│ │                         [+ Tambah]│
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 🔒 Data Sharing                  │  ← 3-tier selector
│ ┌────────────────────────────────┐ │
│ │ ○ Default                      │ │  ← Radio button group
│ │   No data shared. Local AI.    │ │    Selected = green fill
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ ● Eco Mode          [Selected] │ │
│ │   Anonymized aggregated data.  │ │
│ │   Unlocks Community Grid.      │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ ○ Grid Mode                    │ │
│ │   Disaggregated anonymized     │ │
│ │   data. Unlocks TNB rebates.   │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ 🔔 Notifications                 │  ← Toggle list
│ ┌────────────────────────────────┐ │
│ │ All notifications         [ON] │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ ⚠️  Anomaly alerts       [ON] │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 💰  Budget alerts        [ON] │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 💡  Energy tips         [OFF] │ │
│ └────────────────────────────────┘ │
│ ┌────────────────────────────────┐ │
│ │ 👥  Community updates   [ON] │ │
│ └────────────────────────────────┘ │
├──────────────────────────────────┤
│ [🏠 Home] [💡] [👥 Comm] [👤 Prof]│
└──────────────────────────────────┘
```

### Component Breakdown

| Order | Component | Data Source | States |
|-------|-----------|-------------|--------|
| 1 | App Bar | "Profile" static title | Normal |
| 2 | Profile Card | Local user data | Normal, editing (fields become editable) |
| 3 | My Plugs List | All paired plugs from local DB | Has plugs, no plugs ("No plugs paired yet. Add one from the Home screen.") |
| 4 | Always-On List | Whitelist from local DB | Has items, empty ("No always-on devices set.") |
| 5 | Data Sharing Selector | User preference from local | Default selected, Eco selected, Grid selected |
| 6 | Notification Toggles | User prefs from local | Master ON/OFF, each sub-type independent |

### Interactions

| Trigger | Action |
|---------|--------|
| Tap plug in My Plugs | Push to `/plug-detail/{id}` |
| Tap "Edit" on Profile | Fields become editable inline. "Save" replaces "Edit". |
| Tap "Add" in Always-On | Open plug picker (list of paired plugs not yet on whitelist). Tap to add. |
| Swipe always-on item left | "Remove" action. Tap to confirm. |
| Tap Data Sharing tier | Select that tier. Show brief confirmation toast. |
| Tap Grid Mode (if current is Default) | Show consent dialog: "Grid Mode shares anonymized appliance data with TNB. You'll qualify for rebates. Continue?" [Yes] [No] |
| Toggle any notification | Instant toggle. If master is OFF and user toggles a sub-type, sub-type follows master. |

### States

**No plugs paired:**
- My Plugs section: "No plugs paired yet." + "Add your first plug from the Home screen."

**Empty always-on list:**
- "No always-on devices set." + "Always-on devices won't turn off when you activate Leaving Home mode."

**Data sharing consent dialog (on switching to Grid Mode):**
- Title: "Enable Grid Mode?"
- Body: "Grid Mode shares anonymized appliance-level data with TNB. This helps improve Malaysia's energy grid. Your name and address are never shared."
- Buttons: "[Learn More] [Enable]"

---

## 9. Reusable Component Specs

### 9A — DeviceCard

**Where used:** Dashboard (inside Room Section), Settings (My Plugs list).

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `icon` | String | Yes | — | Emoji: 🖥️, 💡, ❄️, 🧊, etc. (28sp, centered) |
| `name` | String | Yes | — | Device name, max 12 chars (ellipsis if longer) |
| `watts` | Int | Yes | — | Current wattage (formatted per Section 10) |
| `status` | Enum | Yes | — | `online_normal`, `online_high`, `online_critical`, `offline` |
| `isOn` | Bool | Yes | — | Relay state |
| `onTap` | Function | Yes | — | Navigate to detail + ripple feedback |
| `onToggle` | Function | Yes | — | Toggle relay with optimistic update |

**Layout:**
```
┌──────────────────┐
│                  │
│      icon        │  ← 28sp emoji, centered, 100% opacity
│      name        │  ← --font-caption (12sp), centered, single line
│      XX W        │  ← --font-mono-small (18sp), centered, color-coded
│      ●           │  ← Status dot, 8dp, animated pulse if anomaly
│   [ON]/[OFF]     │  ← Toggle switch, full width
│                  │
└──────────────────┘
Width: 120dp
Height: 140dp
Background: --color-surface (#1E1E1E)
Border: 1dp, --color-divider, optional, for visual separation
Corner: --radius-lg (16dp)
Elevation: --elevation-card (2dp shadow on normal, 8dp on hover)
Padding: 8dp all sides
```

**Visual States:**

| State | Icon | Watts | Dot | Toggle | Background | Elevation | Notes |
|-------|------|-------|-----|--------|------------|-----------|-------|
| `online_normal` | 100% | Primary | Green | Active (green ON) | Surface | Card (2dp) | Default, healthy state |
| `online_high` | 100% | Amber | Amber | Active | Surface | Card (2dp) | Medium load warning |
| `online_critical` | 100% | Red | Red (pulse) | Active | Surface | Card (2dp) | High load or anomaly detected |
| `offline` | 40% | Tertiary | Grey | Disabled (40% opacity) | Surface (60% opacity) | Card (1dp) | Device disconnected |
| `hover` | 100% | Unchanged | Unchanged | Unchanged | Surface | Card-hover (8dp) | Card lifts, scale +2% |
| `pressed` | 100% | Unchanged | Unchanged | Ripple | Surface | Card (1dp) | Scale -1%, ripple effect |
| `skeleton` | Pulse 60%→40% | Pulse 60%→40% | Pulse circle | Pulse 60%→40% | Surface | Card (2dp) | Loading state, 1.2s loop |

**Micro-Interactions:**
- **Hover**: Elevation increases to `--elevation-card-hover`, scale +2%, 150ms easeOut
- **Press**: Scale -1%, 100ms easeOut, then snap back
- **Toggle tap**: Immediate visual feedback (switch animates 200ms), optimistic update of isOn state
- **Tap card**: Ripple from tap point (300ms easeOut, 20% opacity), navigate after ripple completes
- **Status change (critical)**: Red dot pulses 1.5s loop when anomaly active, opacity 100%→60%→100%

---

### 9B — PowerGauge

**Where used:** Plug Detail screen (center element).

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `currentWatts` | Int | Yes | — | Current wattage, 0-2400 |
| `maxWatts` | Int | Yes | 2400 | Maximum of the gauge range |
| `dailyCostRM` | Double | Yes | — | Calculated daily cost in RM |
| `status` | Enum | Yes | — | `normal`, `warning`, `critical`, `offline` |

**Layout:**
```
Circular gauge, 180dp diameter, centered.
Outer arc: 270° sweep (from 135° to 405°, bottom gap 90°).
Arc thickness: 12dp, smooth cap (strokeLineCap: round).
Track background: --color-divider (#2C2C2C), full 270° circle.
Arc fill: Gradient or solid based on status (see Visual States below).

Center content (layered):
  - Background circle: 140dp, --color-background, no shadow
  - Primary text: "160 W" in --font-mono (32sp, bold), --color-text-primary, centered
  - Secondary text: "RM 0.35 per day" in --font-caption (12sp), --color-text-secondary, centered
  - Line spacing: 4dp between primary and secondary text

Spacing:
  - Gauge from screen edges: 16dp horizontal padding
  - Center circle from arc: 20dp visual gap (concentric)
```

**Visual States:**

| State | Arc color | Arc animation | Center text color | Notes |
|-------|-----------|----------------|------------------|-------|
| `normal` (≤200W) | `--color-primary` (#00C853) | Smooth fill 500ms easeOutCubic | Primary (white) | Healthy state |
| `warning` (200-800W) | `--color-warning` (#FF6D00) | Smooth fill 500ms easeOutCubic | Primary (white) | Medium load |
| `critical` (>800W) | `--color-danger` (#FF1744) | Smooth fill 500ms easeOutCubic, then pulse | Danger (red) | Red glow pulse 1.5s loop, 100%→60% opacity |
| `offline` | `--color-divider` (#2C2C2C) | No animation | Tertiary + "-- W" text | Desaturated, no interaction |

**Micro-Interactions:**
- **Value change animation**: When `currentWatts` changes, arc smoothly animates from old angle to new angle over 500ms using easeOutCubic curve. Creates satisfying "sweep" effect.
- **Status transition**: Color change (green→amber→red) animates smoothly as wattage increases, no jarring jumps.
- **Critical state glow** (>800W): Red arc pulses with opacity animation: 100% → 60% → 100%, loop duration 1.5s, easeInOut curve. Creates attention-grabbing effect for anomalies.
- **On tap**: Add subtle scale animation ±3%, 200ms, for tactile feedback (if tappable for details).
- **Skeleton loader**: Opacity pulse 60%→40%→60%, 1.2s easeInOut loop, before real data arrives.

**Color Zones:**
- 0–200W: Solid --color-primary
- 201–400W: Solid --color-warning
- 401–800W: Solid --color-warning (darker shade, #FF8C00 optional)
- 801–2400W: Solid --color-danger with pulse effect

---

### 9C — ConsumptionChart

**Where used:** Plug Detail screen.

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `dataPoints` | Array<{time, watts}> | Yes | — | Time-series data |
| `range` | Enum | Yes | `7d` | `7d` or `30d` |
| `isLoading` | Bool | No | false | Show skeleton |
| `isEmpty` | Bool | No | false | Show empty state |

**Layout:**
```
Height: 200dp
X-axis: time labels (Mon/Tue/Wed for 7d; 1/5/10/15/20/25/30 for 30d)
Y-axis: wattage (0 to max value, labeled at 3-4 intervals)
Line: `--color-primary`, 2dp stroke width
Fill: `--color-primary` at 15% opacity, from line to x-axis
Grid: horizontal dashed lines at each Y label, `--color-divider`
Tooltip on tap: vertical line + label bubble showing "160W · 3:15 PM, Tue"
Toggle: two pill buttons above chart — [7 Days] [30 Days]
  - Selected: `--color-primary` background, white text
  - Unselected: transparent, `--color-text-secondary` text
```

**States:**

| State | Display |
|-------|---------|
| `loading` | Pulsing skeleton rectangle (200dp × full width), rounded `--radius-lg` |
| `empty` | Centered text: "No data yet — check back tomorrow.", `--font-body`, `--color-text-secondary` |
| `7d data` | 7 data points connected by line, Mon-Sun labels |
| `30d data` | Up to 30 data points, smoother line, date labels every 5 days |

---

### 9D — BudgetBar

**Where used:** Plug Detail screen.

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `spentRM` | Double | Yes | — | Amount spent this month |
| `budgetRM` | Double | Yes | — | Monthly budget cap |
| `daysRemaining` | Int | Yes | — | Days left in month |

**Layout:**
```
┌─────────────────────────────────┐
│ Budget                          │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░  80%    │  ← Progress bar
│ RM 48 / RM 60 · 12 days left    │  ← Caption
└─────────────────────────────────┘
Bar height: 12dp
Bar corner: `--radius-full`
Track: `--color-divider` (#2C2C2C)
Fill color logic:
  - 0–70%: `--color-primary` (#00C853)
  - 70–90%: `--color-warning` (#FF6D00)
  - 90–100%: `--color-danger` (#FF1744)
  - >100%: `--color-danger`, bar full, "Exceeded by RM X" replaces percentage
```

**States:**

| State | Bar color | Text |
|-------|-----------|------|
| `not_set` | Empty track | "Set a budget to track your spending." + "[Set Budget]" link |
| `on_track` (≤70%) | Green | "RM 24 / RM 60 · 18 days left" |
| `warning` (70-90%) | Amber | "RM 48 / RM 60 · 12 days remaining" |
| `exceeded` (90-100%) | Red | "RM 58 / RM 60 · 5 days remaining" |
| `over_budget` (>100%) | Red full | "RM 64 / RM 60 · Exceeded by RM 4" |

---

### 9E — NudgeCard

**Where used:** AI Insights screen (Suggestions section).

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `icon` | String | Yes | — | Emoji: 💡, 📊, 🌤️, etc. (24sp, leading) |
| `title` | String | Yes | — | Headline, max 50 chars, 14sp body weight |
| `body` | String | No | "" | Detail, max 100 chars, 12sp secondary color |
| `timestamp` | String | Yes | — | Relative time or date, 11sp tertiary color |
| `actionLabel` | String | No | "" | Button text, "Open Schedule", "View Stats" |
| `onAction` | Function | No | null | Button callback with haptic feedback |
| `onDismiss` | Function | Yes | — | Swipe dismiss callback, removes card |
| `type` | Enum | Yes | `tip` | `tip` (neutral), `achievement` (green), `comparison` (amber) |

**Layout:**
```
┌─────────────────────────────────────┐
│ ┃ 💡 Your AC has been running      │  ← Left border: 3dp, colored by type
│ ┃ for 4 hours. Open windows?       │    Padding: 16dp left/right, 12dp top/bottom
│ ┃ It's 26°C outside.               │
│ ┃                                  │
│ ┃ just now        [Open Schedule]  │  ← Timestamp left, action right
│ ┃                                  │
│ ┃ ← → swipe indicator              │  ← Subtle visual hint to swipe
└─────────────────────────────────────┘
Card height: auto (min 80dp, max-width expandable)
Corner radius: --radius-lg (16dp)
Background: --color-surface (#1E1E1E)
Elevation: --elevation-card (2dp)
Padding: --space-lg (16dp) all sides
Gap between rows: --space-md (12dp)
```

**Visual States:**

| State | Opacity | Border | Shadow | Swipeable | Notes |
|-------|---------|--------|--------|-----------|-------|
| `unread` | 100% | Colored (3dp) | Card (2dp) | Yes | Full visibility, bright |
| `read` | 70% | Colored (3dp) | Card (2dp) | Yes | Faded, lower priority |
| `achievement` | 100% | Green (3dp) | Card (2dp) | Yes | Success type, full visibility |
| `achievement-read` | 70% | Green (3dp) | Card (2dp) | Yes | Read achievement, faded |
| `dismissing` | Fade to 0% | Fade | None | No | Sliding right, exiting |
| `drag` | 90% | Unchanged | Lift (4dp) | — | While dragging left/right |

**Micro-Interactions:**
- **Entrance**: Fade in 0%→100%, slide up 16dp, 300ms easeOutCubic (staggered if list)
- **Swipe to dismiss**: Drag left/right → opacity fades 100%→0%, slide exit accelerates 250ms easeIn. Haptic feedback (light) on drag start, medium on dismiss.
- **Action button tap**: Ripple effect (300ms easeOut, 20% opacity), feedback haptic (light), callback fires immediately
- **Hover** (on web): Elevation increases to 4dp, subtle background lighten to --color-background-alt
- **On tap card body**: No action (swipe is primary), highlight color briefly (50ms flash to --color-primary at 10% opacity)

**Type-Specific Styling:**
- `tip`: Left border neutral (--color-divider), icon and text neutral
- `achievement`: Left border green (--color-primary), icon green-tinted, text white
- `comparison`: Left border amber (--color-warning), icon amber-tinted, text white

---

### 9F — AnomalyCard

**Where used:** AI Insights screen (Anomalies section).

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `icon` | String | Yes | — | Appliance emoji (24sp) |
| `deviceName` | String | Yes | — | "Fridge", "AC Bedroom" (18sp title) |
| `message` | String | Yes | — | Anomaly description (14sp body, 2 lines max) |
| `impactRM` | String | No | "" | Cost impact, e.g., "RM 18/month" (12sp caption) |
| `timestamp` | String | Yes | — | Relative time (11sp tertiary) |
| `severity` | Enum | Yes | — | `critical` (red), `warning` (amber) |
| `onViewDetails` | Function | Yes | — | Navigate to Plug Detail with ripple |
| `onDismiss` | Function | Yes | — | Dismiss callback with slide animation |

**Layout:**
```
┌────────────────────────────────────┐
│ ┃ 🧊 Fridge                  ⚠️  │  ← Left border: 4dp (thicker for anomaly), colored
│ ┃                                 │    Padding: 16dp all sides
│ ┃ Your fridge consumed 35% more  │
│ ┃ power yesterday. Estimated    │
│ ┃ impact: RM 18/month.          │
│ ┃                                 │
│ ┃ 2 hours ago                     │    Gap between sections: 12dp
│ ┃                                 │
│ ┃ [View Details]   [Dismiss]     │    Buttons aligned bottom, space-between
└────────────────────────────────────┘
Card height: auto (min 100dp)
Corner radius: --radius-lg (16dp)
Background: --color-surface (#1E1E1E)
Elevation: --elevation-card (2dp)
Left border: 4dp, severity-colored, rounded corners
```

**Visual States:**

| State | Border color | Severity icon | Pulsing | Notes |
|-------|-------------|----------------|---------|-------|
| `critical` | `--color-danger` (#FF1744, 4dp) | ⚠️ red | Yes (1.5s loop) | Attention-grabbing, high priority |
| `warning` | `--color-warning` (#FF6D00, 4dp) | 🟡 amber | No | Medium priority, stable |
| `resolved` | N/A | N/A | N/A | Card removed from list, slide-out animation |
| `hover` | Unchanged | Unchanged | Unchanged | Elevation lifts to 4dp, background lightens 5% |
| `dismissed` | Fade to 0% | Fade | Stop | Card slides down and out, 300ms easeIn |

**Micro-Interactions:**
- **Entrance**: Fade in 0%→100%, slide down 16dp (from top), 300ms easeOutCubic, staggered if multiple
- **Pulsing border** (critical only): Left border pulses in brightness: 100% → 60% → 100%, 1.5s easeInOut loop, creates urgent feel
- **View Details button**: Ripple effect (300ms easeOut, 20% opacity), haptic feedback (medium), navigate to Plug Detail with anomaly banner pre-expanded
- **Dismiss button**: Ripple effect (200ms easeOut), haptic (light), card slides down and fades out 300ms easeIn
- **Hover state**: Elevation increases to 4dp (--elevation-card-hover), 150ms easeOut transition, subtle background color shift
- **On swipe left**: Optional: show delete action, slide back on cancel

**Color Zones:**
- `critical` (>800W+ or 35%+ increase): Red border + pulsing effect, ⚠️ icon, high priority
- `warning` (200-800W or 20-35% increase): Amber border (solid, no pulse), 🟡 icon, medium priority

---

### 9G — StatusDot

**Where used:** DeviceCard, Plug Detail, My Plugs list.

**Props:**

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `status` | Enum | Yes | — | `online`, `offline`, `anomaly` |

**Layout:**
```
Circle, 8dp diameter.
  `online` → `--color-online` (#00E676), solid fill
  `offline` → `--color-offline` (#757575), solid fill
  `anomaly` → `--color-danger` (#FF1744), solid fill + subtle pulse animation (scale 1.0 → 1.3 → 1.0, 2s loop)
```

---

## 10. Data Display Conventions

### Wattage Formatting

| Range | Format | Example |
|-------|--------|---------|
| <1000W | `{value} W` | "12 W", "160 W", "850 W" |
| ≥1000W | `{value} kW` (1 decimal) | "1.2 kW", "2.4 kW" |
| Unknown | "-- W" | "-- W" |

### Currency Formatting

| Format | Example |
|--------|---------|
| Always: `RM {amount}` | "RM 18.40", "RM 1.37", "RM 0.35" |
| 2 decimal places always | "RM 5.00", "RM 0.10" |
| Null/unknown | "RM --.--" |

### Timestamp Formatting

| Age | Format | Example |
|-----|--------|---------|
| <1 minute | "just now" | "just now" |
| <60 minutes | "{N} min ago" | "3 min ago", "45 min ago" |
| <24 hours | "{N} hours ago" | "2 hours ago" |
| Yesterday | "Yesterday, {time}" | "Yesterday, 7:30 PM" |
| This week | "{day}, {time}" | "Tue, 3:15 PM" |
| Older | "{day} {month}" | "12 Apr 2026" |

### Chart Axis Conventions

- **Y-axis:** 3-4 labeled intervals. Max value auto-calculated from data (ceiling to nearest 50W).
- **X-axis (7d):** Mon, Tue, Wed, Thu, Fri, Sat, Sun
- **X-axis (30d):** Date labels every 5 days (1, 5, 10, 15, 20, 25, 30)
- **Grid lines:** Horizontal dashed, `--color-divider` at 20% opacity

### Universal Color-Coding Rule

Applied consistently across PowerGauge, DeviceCard, and StatusDot:

| Range | Color | Token |
|-------|-------|-------|
| 0–200W | Green | `--color-primary` / `--color-online` |
| 201–800W | Amber/Orange | `--color-warning` |
| 801W+ | Red | `--color-danger` |

---

## 10.5. Icon System & Emoji Specification

### Icon Philosophy
The Smart Plug app uses **emoji as the primary icon system** for all appliance, action, and status indicators. This approach provides:
- **Instant recognition** by users (universal symbols)
- **Cultural familiarity** (emoji are locale-aware)
- **Consistent rendering** across platforms (Flutter-native emoji support)
- **Reduced design/implementation overhead** vs. custom icon sets

### Appliance Icons (Device Types)
Used in DeviceCard, Settings "My Plugs", and Pairing flow.

| Appliance | Emoji | Alternative Emoji | Usage |
|-----------|-------|-------------------|-------|
| Air Conditioner | ❄️ | 🌬️ | Cooling appliances |
| TV | 🖥️ | 📺 | TVs, monitors, screens |
| Light/Lamp | 💡 | 🔆 | Lights, bulbs |
| Phone Charger | 🔌 | 🔋 | Chargers, power adapters |
| Refrigerator | 🧊 | ❄️ | Fridges, freezers |
| Rice Cooker | 🍚 | 🥘 | Cooking appliances |
| Fan | 🌀 | 🌬️ | Fans, ventilation |
| Microwave | 📺 | 🍲 | Microwave ovens |
| Washing Machine | 🌊 | 🧺 | Washing machines, dryers |
| Heater | 🔥 | 🌡️ | Heaters, water heaters |
| Router | 📶 | 🛰️ | Routers, network devices |
| Fish Tank | 🐠 | 🐡 | Aquariums, pumps |
| Kettle | 🫖 | ☕ | Kettles, hot water dispensers |
| Vacuum | 🏠 (generic) | 🧹 | Vacuums, cleaning appliances |

### Action & Section Icons (Tab Bar, Section Headers)
Used in navigation and section identifiers.

| Action/Section | Emoji | Usage Context |
|---|---|---|
| Home/Dashboard | 🏠 | Home tab, Living Room section header |
| Insights/Lightbulb | 💡 | AI Insights tab, suggestions |
| Community/Users | 👥 | Community Grid tab |
| Profile/User | 👤 | Settings tab |
| Plug | 🔌 | Generic plug icon, pairing, "Add Plug" |
| Leaving Home | 🚪 | Quick action chip for "Leaving Home" mode |
| Alert/Warning | ⚠️ | Anomaly card severity indicator |
| Community Pool | 🏘️ | Community Grid donation recipient |
| Impact/Globe | 🌏 | Community Grid impact view |
| Vampire/Bat | 🦇 | Vampire power auto-off feature |

### Status Indicators & Semantic Icons
**Never use emoji for these.** Use color-coded circles instead.

| Status | Icon | Color | Size | Usage |
|--------|------|-------|------|-------|
| Online/Normal | ● (circle) | `--color-online` (#00E676) | 8dp | DeviceCard, Plug Detail |
| Online/High Load | ● (circle) | `--color-warning` (#FF6D00) | 8dp | DeviceCard when 200-800W |
| Online/Critical | ● (circle) | `--color-danger` (#FF1744) | 8dp | DeviceCard when >800W, pulsing |
| Offline | ● (circle) | `--color-offline` (#757575) | 8dp | DeviceCard when disconnected |
| Anomaly Alert | ⚠️ (emoji) | `--color-danger` (#FF1744) | 20sp | AnomalyCard severity indicator |

### Emoji Sizing Guidelines

| Context | Size | Font Family | Usage |
|---------|------|------------|-------|
| **Card header** | 28sp | System emoji | DeviceCard, Plug Detail icons |
| **Tab bar** | 24sp | System emoji | Navigation tabs (Home, Insights, Community, Profile) |
| **Section header** | 24sp | System emoji | Room headers, section titles |
| **In-text** | 20sp | System emoji | Inline in cards, buttons, chips |
| **Small/metadata** | 16sp | System emoji | Minimal contexts, secondary info |
| **Floating action** | 32sp | System emoji | FABs, primary actions (if used) |

### Emoji Consistency Rules
1. **All emoji must be properly spaced**: Use zero-width joiner (ZWJ) sequences only when documented in design spec.
2. **Skin tone variants**: Use default (yellow) skin tone emoji for appliances. Use default for people emoji (👤, 👥).
3. **Variation selectors**: Ensure emoji render as full color (U+FE0F suffix). Test on iOS and Android.
4. **Accessibility**: Pair all emoji with text labels for screen reader users (aria-label on buttons, alt text in descriptions).
5. **Consistency across screens**: Same appliance = same emoji everywhere. No substitutes unless explicitly documented.
6. **Dark theme rendering**: Emoji render correctly on dark backgrounds (#121212). No special styling needed.

### Custom Icon Set (Future Consideration)
If Flutter team prefers a custom icon set over emoji in the future:
- Use **outlined** style (not filled) for consistency with Material Design 3
- Maintain same semantic sizing as emoji spec above
- Ensure WCAG AA contrast on dark backgrounds
- Test on 1x, 2x, 3x pixel densities for mobile
- Provide SVG source files for all icons

---

## 11. Copy Bank

### Dashboard

| Key | Text |
|-----|------|
| `greeting.morning` | "Good morning" |
| `greeting.afternoon` | "Good afternoon" |
| `greeting.evening` | "Good evening" |
| `summary.label` | "today" |
| `summary.trend_up` | "▲ {pct}% vs yesterday" |
| `summary.trend_down` | "▼ {pct}% vs yesterday" |
| `summary.trend_same` | "Same as yesterday" |
| `summary.no_data` | "-- W" |
| `chip.leaving_home` | "🚪 Leaving Home" |
| `chip.all_off` | "All Off" |
| `chip.all_on` | "All On" |
| `toast.leaving_home` | "Turned off {count} devices. Fridge and always-on devices still running." |
| `toast.all_off` | "All plugs turned off." |
| `toast.all_on` | "All plugs turned on." |
| `empty.no_plugs_title` | "No plugs paired" |
| `empty.no_plugs_body` | "Tap below to add your first smart plug and start saving energy." |
| `empty.no_plugs_button` | "+ Add Your First Smart Plug" |
| `error.connection_lost` | "Connection lost. Reconnecting..." |

### Plug Detail

| Key | Text |
|-----|------|
| `stat.voltage` | "Voltage" |
| `stat.current` | "Current" |
| `stat.kwh_today` | "Today" |
| `stat.rm_month` | "Month" |
| `chart.toggle_7d` | "7 Days" |
| `chart.toggle_30d` | "30 Days" |
| `chart.empty` | "No data yet — check back tomorrow." |
| `schedule.header` | "Schedules" |
| `schedule.add` | "+ Add" |
| `schedule.empty` | "No schedules set. Tap + to add your first." |
| `budget.not_set` | "Set a budget to track your spending." |
| `budget.set_button` | "Set Budget" |
| `budget.exceeded` | "Exceeded by RM {amount}" |
| `vampire.label` | "Vampire auto-off" |
| `always_on.label` | "Always-on" |
| `anomaly.banner` | "⚠️ This device is consuming {pct}% more than usual." |
| `anomaly.view_details` | "View Details" |
| `vampire.banner` | "Vampire cutoff triggered — drawing {watts}W for 30+ min." |
| `vampire.undo` | "Undo" |
| `schedule.active_pill` | "Scheduled — on until {time}" |

### Pairing Flow

| Key | Text |
|-----|------|
| `pairing.title` | "Add a Plug" |
| `pairing.scanning` | "Searching for nearby plugs..." |
| `pairing.instruction` | "Make sure your plug is plugged in and the LED is pulsing white." |
| `pairing.scan_again` | "Scan Again" |
| `pairing.select_title` | "Select Your Plug" |
| `pairing.no_plugs` | "No plugs found nearby." |
| `pairing.no_plugs_help` | "Make sure your Sonoff S31 is plugged in and the LED is pulsing white." |
| `pairing.wifi_title` | "Connect to Wi-Fi" |
| `pairing.wifi_label` | "Wi-Fi Network" |
| `pairing.password_label` | "Password" |
| `pairing.connect_button` | "Connect Plug" |
| `pairing.progress_title` | "Setting Up Your Plug" |
| `pairing.progress.step1` | "Connecting to Wi-Fi" |
| `pairing.progress.step2` | "Configuring MQTT broker" |
| `pairing.progress.step3` | "Testing relay" |
| `pairing.progress.step4` | "Assigning plug ID" |
| `pairing.progress.step5` | "Syncing with your account" |
| `pairing.name_title` | "Almost Done" |
| `pairing.name_label` | "Name" |
| `pairing.room_label` | "Room" |
| `pairing.type_label` | "Appliance Type" |
| `pairing.finish_button` | "Finish Setup" |
| `pairing.success_title` | "Plug Added Successfully" |
| `pairing.success_body` | "{name} is ready to monitor and control." |
| `pairing.success_button` | "Go to Dashboard" |
| `pairing.error_wifi` | "Could not connect to Wi-Fi. Check your password and try again." |
| `pairing.try_again` | "Try Again" |
| `pairing.cancel` | "Cancel" |
| `pairing.ble_permission` | "Bluetooth permission is needed to discover nearby plugs." |
| `pairing.open_settings` | "Open Settings" |
| `pairing.not_now` | "Not Now" |

### AI Insights

| Key | Text |
|-----|------|
| `insights.title` | "Insights" |
| `anomalies.header` | "⚠️ Anomalies" |
| `anomalies.empty` | "No anomalies detected. Your appliances are running smoothly ✅" |
| `suggestions.header` | "💬 Suggestions" |
| `suggestions.empty` | "No new suggestions right now. Check back tomorrow for personalized tips." |
| `savings.header` | "💰 Savings Summary" |
| `savings.no_goal` | "Set a monthly savings goal to start tracking your impact." |
| `savings.set_goal` | "Set Goal" |
| `savings.goal_reached` | "🎉 Goal reached! RM {amount} saved this month." |
| `savings.label` | "saved" |
| `nudge.action_schedule` | "Open Schedule" |
| `nudge.action_stats` | "View Stats" |
| `nudge.action_tips` | "See Tips" |
| `anomaly.action_view` | "View Details" |
| `anomaly.action_dismiss` | "Dismiss" |
| `anomaly.action_schedule` | "Schedule Check" |

### Community Grid (KL-Grounded)

| Key | BM Text | EN Text |
|-----|---------|---------|
| `community.title` | "Komuniti" | "Community" |
| `community.credits_label` | "Kredit Tenaga Anda" | "Your Energy Credits" |
| `community.credits_unit` | "kredit tersedia" | "credits available" |
| `community.credits_no` | "Anda belum ada kredit. Kurangkan penggunaan bawah baseline untuk mula dapatkan." | "You haven't earned credits yet. Reduce your usage below your baseline to start earning." |
| `community.credits_baseline` | "Jimat tenaga untuk dapatkan kredit. Baseline anda sedang dikira — semak semula dalam 7 hari." | "Save energy to earn credits. Your baseline is being calculated — check back in 7 days." |
| `community.donate_button` | "Derma Kredit" | "Donate Credits" |
| `community.leaderboard.header` | "🏆 Penderma Teratas — {month}" | "🏆 Top Donors — {month}" |
| `community.leaderboard.empty` | "Tiada derma bulan ini. Jadilah yang pertama dan tuntut tempat #1!" | "No donations this month. Be the first and claim the #1 spot!" |
| `community.leaderboard.you` | "Anda" | "You" |
| `community.leaderboard.donated_to` | "{count} kpd {count} jiran — {ppr}" | "{count} to {count} neighbors — {ppr}" |
| `community.impact.header` | "🌏 Impak Komuniti" | "🌏 Community Impact" |
| `community.impact.monthly` | "Bulan ini, kawasan KL kita jimat:" | "This month, our KL neighborhood saved:" |
| `community.impact.equiv1` | "🏠 {count} unit PPR sehari" | "🏠 {count} PPR units for a day" |
| `community.impact.equiv2` | "❄️ {count} AC selama 8 jam" | "❄️ {count} AC units for 8 hours" |
| `community.impact.equiv3` | "🚦 {count} lampu Jln Bukit Bintang" | "🚦 {count} Jln Bukit Bintang street lamps" |
| `community.impact.share` | "📤 Kongsi" | "📤 Share" |
| `community.impact.credits_shared` | "{count} kredit dikongsi ke {count} kawasan" | "{count} credits shared across {count} neighborhoods" |
| `donate.title` | "Derma Kredit" | "Donate Credits" |
| `donate.recipient_label` | "Pilih penerima" | "Choose a recipient" |
| `donate.pool` | "🏠 Tabung Komuniti" | "🏠 Community Pool" |
| `donate.pool_desc` | "Diagih ke PPR sekitar KL secara automatik" | "Distributed to KL PPR households automatically" |
| `donate.ppr_waiting` | "{count} keluarga sedang tunggu" | "{count} families waiting" |
| `donate.amount_label` | "Jumlah" | "Amount" |
| `donate.impact_preview` | "~RM {amount} bil elektrik" | "~RM {amount} electricity bill" |
| `donate.ppr_multiplier` | "Derma ke PPR = 2x XP lencana" | "PPR donation = 2x badge XP" |
| `donate.confirm` | "Sahkan Derma" | "Confirm Donation" |
| `donate.success_title` | "{count} kredit didermakan!" | "{count} credits donated!" |
| `donate.success_body` | "Anda bantu ringankan ~RM {amount} untuk {ppr}." | "You helped offset ~RM {amount} for {ppr}." |
| `donate.success_badge_hint` | "🛡️ {count} lagi untuk lencana Jiran Terbaik!" | "🛡️ {count} more for Jiran Terbaik badge!" |
| `donate.view_impact` | "Lihat Impak" | "View Impact" |
| `donate.done` | "Selesai" | "Done" |

### Gamification — Streaks & Challenges (KL-Grounded)

| Key | BM Text | EN Text |
|-----|---------|---------|
| `streak.label` | "🔥 {count}-Hari Rantaian" | "🔥 {count}-Day Streak" |
| `streak.nudge` | "Jangan patah rantaian! {count} hari berturut-turut ✓" | "Don't break your streak! {count} consecutive days ✓" |
| `streak.broken` | "Rantaian putus. Mula semula esok!" | "Streak broken. Start again tomorrow!" |
| `challenge.active` | "⚡ CABARAN: {name} ({month})" | "⚡ CHALLENGE: {name} ({month})" |
| `challenge.days_left` | "{count} hari lagi" | "{count} days remaining" |
| `challenge.leading` | "{kawasan} mendahului!" | "{kawasan} is leading!" |
| `challenge.won` | "🏆 {kawasan} menang! 20% multiplier untuk {nextMonth} 🔥" | "🏆 {kawasan} won! 20% multiplier for {nextMonth} 🔥" |
| `challenge.lost` | "{kawasan} menang {month}. Sertai cabaran {nextMonth}: {nextChallenge} 🇲🇾" | "{kawasan} won {month}. Join {nextMonth}'s challenge: {nextChallenge} 🇲🇾" |
| `challenge.expanded.reward` | "Hadiah: Lencana 'Jaguh Bulan Ini' + 20% multiplier" | "Reward: 'Champion of the Month' badge + 20% multiplier" |
| `challenge.share` | "Kongsi Cabaran" | "Share Challenge" |

### Badges — Koleksi Lencana (KL-Grounded)

| Key | BM Text | EN Text |
|-----|---------|---------|
| `badges.title` | "Koleksi Lencana" | "Badge Collection" |
| `badges.earned_header` | "Diperolehi ({count})" | "Earned ({count})" |
| `badges.locked_header` | "Terkunci ({count})" | "Locked ({count})" |
| `badges.empty` | "Belum ada lencana. Mulakan perjalanan jimat tenaga anda!" | "No badges yet. Start your energy saving journey!" |
| `badges.unlock_toast` | "🎉 Lencana baru: {name}! Ketik untuk lihat." | "🎉 New badge: {name}! Tap to view." |
| `badges.level_card` | "🏅 {levelName} (Tahap {level})" | "🏅 {levelName} (Level {level})" |
| `badges.xp_next` | "{xp} XP lagi ke {nextLevel}" | "{xp} XP to {nextLevel}" |

### KL Persona Names (For Prototype Mock Data)

| Persona | Name | Kawasan | League | Vibe |
|---------|------|---------|--------|------|
| User | Aisyah Binti Rahman | TTDI | Taman League | Working mom, 2 kids, energy-conscious |
| #1 | Kumar A/L Muthu | Bangsar | Taman League | Young professional, competitive saver |
| #2 | Mei Ling Wong | Cheras | Taman League | Retiree, garden enthusiast, steady saver |
| #3 | Raj A/L Selvam | Damansara Heights | Taman League | Family man, 3 kids, weekend warrior |
| #4 | Fatimah Binti Hassan | PPR Kerinchi | Rumah Pangsa | Single mom, 1 child, grateful recipient → becoming donor |

### Settings (KL-Updated)

| Key | BM Text | EN Text |
|-----|---------|---------|
| `settings.title` | "Profil" | "Profile" |
| `profile.edit` | "Edit Profil" | "Edit Profile" |
| `profile.save` | "Simpan" | "Save" |
| `profile.level_title` | "🏅 {levelName} (Tahap {level})" | "🏅 {levelName} (Level {level})" |
| `profile.level_xp` | "{current} / {required} XP" | "{current} / {required} XP" |
| `profile.level_next` | "{xp} XP lagi ke {nextLevel}" | "{xp} XP to {nextLevel}" |
| `plugs.header` | "🔌 Plugs Saya" | "🔌 My Plugs" |
| `plugs.empty` | "Tiada plug. Tambah dari skrin Utama." | "No plugs paired yet. Add one from the Home screen." |
| `always_on.header` | "📌 Sentiasa-On" | "📌 Always-On" |
| `always_on.empty` | "Tiada peranti sentiasa-on." | "No always-on devices set." |
| `always_on.add` | "+ Tambah" | "+ Add" |
| `notifications.header` | "🔔 Notifikasi" | "🔔 Notifications" |
| `notifications.anomalies` | "⚠️ Amaran anomali" | "⚠️ Anomaly alerts" |
| `notifications.budget` | "💰 Amaran bajet" | "💰 Budget alerts" |
| `notifications.tips` | "💡 Tip tenaga" | "💡 Energy tips" |
| `notifications.community` | "👥 Kemas kini komuniti" | "👥 Community updates" |
| `data_sharing.tier1` | "Default" |
| `data_sharing.tier1_desc` | "No data shared. Local AI only." |
| `data_sharing.tier2` | "Eco Mode" |
| `data_sharing.tier2_desc` | "Anonymized aggregated data. Unlocks Community Grid." |
| `data_sharing.tier3` | "Grid Mode" |
| `data_sharing.tier3_desc` | "Disaggregated anonymized data. Unlocks TNB rebates." |
| `data_sharing.consent_title` | "Enable Grid Mode?" |
| `data_sharing.consent_body` | "Grid Mode shares anonymized appliance-level data with TNB. This helps improve Malaysia's energy grid. Your name and address are never shared." |
| `data_sharing.learn_more` | "Learn More" |
| `data_sharing.enable` | "Enable" |

### General

| Key | Text |
|-----|------|
| `loading` | "Loading..." |
| `retry` | "Retry" |
| `cancel` | "Cancel" |
| `save` | "Save" |
| `delete` | "Delete" |
| `done` | "Done" |
| `back` | "←" |
| `close` | "✕" |
| `on` | "ON" |
| `off` | "OFF" |
| `yes` | "Yes" |
| `no` | "No" |

---

## 12. Output Instructions

**For Claude:** When generating designs from this document, follow these rules:

1. **Generate one screen at a time.** Work through sections 3-8 in order.
2. **Produce for each screen:**
   - Full ASCII wireframe showing every element using the layout described
   - Component breakdown table listing every element in order, its data source, and all possible states
   - Interaction map (what every tappable/swipeable element does)
   - All states rendered as separate wireframes (empty, loading, error, offline, anomaly, edge cases)
3. **For each component requested:** Render normal state + all visual states described in section 9.
4. **For each flow requested:** Render a step-by-step wireframe sequence showing every screen transition.
5. **Use only tokens from Section 1.** Do not invent new colors, fonts, or spacing values.
6. **Use only copy from Section 11.** Do not invent new text strings. Reference by key.
7. **Apply Section 10 conventions** for all numbers: wattage, currency, timestamps, chart axes.
8. **Do not generate code.** Pure visual design descriptions and wireframes only.
9. **Preserve the dark theme** — all surfaces use `--color-background` (#121212) and `--color-surface` (#1E1E1E).
10. **Smart plug icons are emoji** — use the emoji specified in each component/screen spec. Do not design new icons.

---

*Generated from App Design Spec v1.0 — Smart Plug Power Management Ecosystem*
