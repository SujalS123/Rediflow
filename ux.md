# RideFlow – UX Flow Document
**Version:** 1.0  
**Author:** Product / Design  
**Status:** Final – MVP Scope  
**Last Updated:** June 2026

---

## 1. Overview

This document defines the complete user experience flow for the RideFlow MVP. It covers all 7 screens, the transitions between them, the component-level layout, interaction states, and copy guidelines. This document is the source of truth for the frontend developer.

The primary user flow is linear and sequential:

**Home → Route Options → Route Details → Booking Summary → Payment → Journey Pass → Live Tracking**

There are no branching flows in the MVP. The user proceeds forward through the journey. Back navigation is supported on all screens.

---

## 2. Design Principles

**Principle 1: One Thing Per Screen**
Each screen has a single primary action. The user always knows what to do next. There are no screens with multiple competing CTAs.

**Principle 2: Show, Don't Tell**
Numbers and data do the communicating. Route cards show time, cost, and distance — the user should not have to read paragraphs to understand the options.

**Principle 3: Trust Through Transparency**
Explanations are honest and grounded in data. The app never overpromises. Mock states are clearly a simulation — not presented as real bookings.

**Principle 4: Mobile-First**
All screens designed for a mobile viewport (375px–430px wide). The web version is a responsive adaptation.

**Principle 5: Demo-Friendly**
Every screen must read clearly in a 3–5 minute video demo. Key information must be visible without scrolling on a standard phone screen.

---

## 3. Color System and Visual Language

**Primary:** Deep Indigo (#3730A3) — trust, intelligence, transit
**Accent:** Amber (#F59E0B) — energy, urgency, action
**Success:** Emerald (#10B981) — completion, payment success
**Error:** Red (#EF4444) — payment failure, delay alerts
**Neutral Background:** Slate-50 (#F8FAFC)
**Card Background:** White (#FFFFFF)
**Text Primary:** Slate-900 (#0F172A)
**Text Secondary:** Slate-500 (#64748B)

**Transport Mode Colors:**
- Walk → Slate-400 (neutral)
- Bus → Blue-500
- Metro → Indigo-600
- Train → Violet-700
- Auto → Amber-500
- Cab → Orange-500
- Bike → Green-500

---

## 4. Typography

**Headings:** Inter Bold, 20–28px
**Body:** Inter Regular, 14–16px
**Labels / Captions:** Inter Medium, 12px
**Numbers / Data:** Inter Semibold (for fare, time, distance)

---

## 5. Screen-by-Screen UX Specification

---

### Screen 1: Home / Search Screen

**Purpose:** Entry point. User selects source, destination, and preference.

**Layout:**

```
┌─────────────────────────────────┐
│  🚇  RideFlow                   │ ← App name + icon, top left
│  Smart multi-modal journeys     │ ← Tagline, subtle
├─────────────────────────────────┤
│                                 │
│  Where are you?                 │ ← Section label
│  ┌───────────────────────────┐  │
│  │ 📍 Select source         ▼│  │ ← Dropdown
│  └───────────────────────────┘  │
│                                 │
│  Where do you want to go?       │
│  ┌───────────────────────────┐  │
│  │ 🏁 Select destination    ▼│  │ ← Dropdown
│  └───────────────────────────┘  │
│                                 │
│  Route preference               │
│  ┌──────┐ ┌───────┐ ┌────────┐ │
│  │Balan.│ │Fastest│ │Cheapest│ │ ← Pill selectors
│  └──────┘ └───────┘ └────────┘ │
│  ┌──────────┐ ┌──────┐ ┌────┐  │
│  │Less Walk.│ │Fewer.│ │Eco │  │
│  └──────────┘ └──────┘ └────┘  │
│                                 │
│  ┌───────────────────────────┐  │
│  │     🔍 Find Routes        │  │ ← Primary CTA, full width
│  └───────────────────────────┘  │
│                                 │
│  Popular routes:                │ ← Optional: quick launch chips
│  [Central → Tech Park]          │
│  [Residential → University]     │
└─────────────────────────────────┘
```

**Component States:**

- Default: both dropdowns show placeholder text, Balanced preference pre-selected
- Validation error: red border + "Please select a source" / "Source and destination must be different"
- Loading: button shows spinner, text changes to "Finding routes..."
- Hover on preference pill: slight background change

**Interactions:**

- Tapping a preference pill selects it (single select, only one active at a time)
- Tapping Find Routes validates inputs then calls /routes/search
- On success: navigate to Route Options Screen, passing route results
- On error: show toast notification with error message

**Copy:**
- App tagline: "Smart multi-modal journeys for every Indian commuter"
- Button text: "Find Routes"
- Loading text: "Planning your journey..."

---

### Screen 2: Route Options Screen

**Purpose:** Show multiple ranked route options as comparable cards. Let user choose.

**Layout:**

```
┌─────────────────────────────────┐
│  ← Back                         │
│  Central Station → Tech Park    │ ← Journey header
│  3 routes found • Balanced      │ ← Subtitle with preference
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │ ⭐ RECOMMENDED          │    │ ← Tag badge, colored
│  │ Walk → Bus → Metro → Auto│   │ ← Mode sequence (icons)
│  │                         │    │
│  │ ⏱ 54 min  💰 ₹82       │    │ ← Time and fare, large
│  │ 🔄 2 transfers          │    │
│  │ 🚶 650m walking         │    │
│  │ 🌿 Low carbon  🛡 Medium safety│
│  │ ████████░ Reliability: High  │ ← Simple bar or label
│  │                 View Route → │ ← Secondary CTA
│  └─────────────────────────┘    │
│                                 │
│  ┌─────────────────────────┐    │
│  │ 💸 CHEAPEST             │    │
│  │ Walk → Train → Bus → Walk│   │
│  │                         │    │
│  │ ⏱ 68 min  💰 ₹54       │    │
│  │ 🔄 2 transfers          │    │
│  │ 🚶 1200m walking        │    │
│  │ 🌿 Low carbon  🛡 High safety │
│  │                 View Route → │
│  └─────────────────────────┘    │
│                                 │
│  ┌─────────────────────────┐    │
│  │ ⚡ FASTEST              │    │
│  │ Auto → Metro → Walk     │    │
│  │                         │    │
│  │ ⏱ 42 min  💰 ₹130      │    │
│  │ 🔄 1 transfer           │    │
│  │ 🚶 300m walking         │    │
│  │ 🌿 Med. carbon  🛡 High safety│
│  │                 View Route → │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

**Component States:**

- Recommended card: border-color = Primary indigo, slightly elevated shadow
- Cheapest card: border-color = Emerald
- Fastest card: border-color = Amber
- Hover/tap: card elevates with subtle scale animation
- Active selection: card gets a filled checkmark top-right

**Interactions:**

- Tapping anywhere on a card navigates to Route Details for that route
- No need to select before viewing details (tap = view)
- Changing preference (if accessible from this screen via filter icon) re-sorts cards

**Copy:**
- Section header: "{N} routes found · {Preference}"
- Tag labels: "Recommended", "Cheapest", "Fastest", "Eco-Friendly"
- CTA: "View Route →"
- If no routes: "No routes found for this journey. Try different locations."

---

### Screen 3: Route Details Screen

**Purpose:** Deep dive into a single route — step-by-step timeline, explanation, and booking CTA.

**Layout:**

```
┌─────────────────────────────────┐
│  ← Back                         │
│  Walk → Bus → Metro → Auto      │ ← Route summary header
│  54 min · ₹82 · ⭐ Recommended  │
├─────────────────────────────────┤
│  JOURNEY TIMELINE               │ ← Section header
│                                 │
│  ●── 🚶 Walk · 6 min           │
│  │   Current Location           │
│  │   ↓ 400m to Bus Stop Bay 3   │
│  │                              │
│  ●── 🚌 Bus 335E · 18 min · ₹15│
│  │   Bus Stop Bay 3             │
│  │   ↓ to Metro Central Stop    │
│  │                              │
│  ●── 🚇 Blue Line · 22 min · ₹35│
│  │   Metro Central Station      │
│  │   Platform 2                 │
│  │   ↓ to Tech Park Metro       │
│  │                              │
│  ●── 🛺 Auto · 8 min · ₹32    │
│      Tech Park Metro Gate 1     │
│      ↓ to Tech Park Entrance    │
│                                 │
├─────────────────────────────────┤
│  WHY THIS ROUTE?                │ ← Explanation section
│  ┌─────────────────────────┐    │
│  │ 💡 This route offers the    │ │
│  │ best balance of time, cost, │ │
│  │ and walking. 54 min, ₹82,  │ │
│  │ 2 transfers. High           │ │
│  │ reliability.                │ │
│  └─────────────────────────┘    │
│                                 │
├─────────────────────────────────┤
│  ┌───────────────────────────┐  │
│  │    📋 Book This Journey   │  │ ← Primary CTA
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

**Component States:**

- Timeline dots: colored by mode (bus = blue, metro = indigo, auto = amber, walk = slate)
- Connecting line: vertical dashed line between stops
- Explanation card: light indigo background, 💡 icon, italicized or regular text
- Book button: full width, primary color, bold text

**Interactions:**

- Each timeline step is expandable (tap to see full instruction text)
- Scrollable if content exceeds screen
- Book This Journey → calls /bookings/create → navigate to Booking Summary

**Copy:**
- Timeline header: "Journey Timeline"
- Explanation header: "Why this route?"
- Book button: "Book This Journey"
- Loading: "Creating your booking..."

---

### Screen 4: Booking Summary Screen

**Purpose:** Show the generated mock tickets and fare breakdown before payment.

**Layout:**

```
┌─────────────────────────────────┐
│  ← Back                         │
│  Booking Summary                │
│  Booking ID: RF-BKG-1001        │ ← Generated booking ID
├─────────────────────────────────┤
│  JOURNEY TICKETS                │
│                                 │
│  🚌 Bus 335E                    │
│  Ticket ID: BUS-RF-3021         │
│  Fare: ₹15                      │
│  ─────────────────────────────  │
│  🚇 Blue Line Metro             │
│  Ticket ID: MTR-RF-8841         │
│  Fare: ₹35                      │
│  ─────────────────────────────  │
│  🛺 Auto – Last Mile            │
│  Booking ID: AUTO-RF-2219       │
│  Fare: ₹32                      │
│  ─────────────────────────────  │
│  🚶 Walking legs: Free          │
│                                 │
├─────────────────────────────────┤
│  FARE SUMMARY                   │
│  Bus             ₹15            │
│  Metro           ₹35            │
│  Auto            ₹32            │
│  ─────────────────              │
│  Total           ₹82            │
│                                 │
├─────────────────────────────────┤
│  ┌───────────────────────────┐  │
│  │   💳 Proceed to Payment   │  │ ← Primary CTA
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

**Component States:**

- Ticket rows: clean list, each with mode icon, ticket ID, and fare
- Walking legs shown as "Free" entry (for completeness)
- Total row: bold, slightly larger font
- Proceed button: full width, primary color

**Interactions:**

- No editing on this screen — summary only
- Proceed to Payment → navigate to Payment Screen with bookingId

---

### Screen 5: Mock Payment Screen

**Purpose:** Show mock wallet, allow user to pay, and confirm success or failure.

**Layout (Pre-Payment):**

```
┌─────────────────────────────────┐
│  ← Back                         │
│  Mock NCMC Wallet               │
├─────────────────────────────────┤
│                                 │
│       Your Wallet Balance       │
│           ₹500                  │ ← Large, prominent
│                                 │
├─────────────────────────────────┤
│  PAYMENT DETAILS                │
│                                 │
│  Journey: Central → Tech Park   │
│  Route: Walk → Bus → Metro → Auto│
│  Amount to Pay: ₹82             │
│                                 │
│  Payment Method: Mock NCMC Wallet│ ← Static, no options needed
│                                 │
├─────────────────────────────────┤
│  ┌───────────────────────────┐  │
│  │       💳 Pay ₹82          │  │ ← Primary CTA
│  └───────────────────────────┘  │
│                                 │
│  Note: This is a simulated      │ ← Disclaimer, small text
│  payment. No real money moves.  │
└─────────────────────────────────┘
```

**Layout (Post-Payment Success):**

```
┌─────────────────────────────────┐
│                                 │
│       ✅                        │ ← Large success icon
│   Payment Successful!           │
│                                 │
│   Journey Pass ID: RF-PASS-9182 │
│   Amount Paid: ₹82              │
│   Remaining Balance: ₹418      │
│                                 │
│  ┌───────────────────────────┐  │
│  │     🎫 View Journey Pass  │  │ ← Navigate to Journey Pass
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

**Layout (Insufficient Balance):**

```
│       ❌                        │
│   Payment Failed                │
│   Insufficient wallet balance   │
│   Required: ₹82 · Balance: ₹30  │
│                                 │
│  [Top Up Wallet] ← disabled/mock │
│  [Choose Cheaper Route] ← back nav│
```

**Interactions:**

- Pay button → calls /payments/pay → shows success or failure state
- Success → show pass ID, remaining balance, navigate CTA
- Failure → show reason, offer back navigation

---

### Screen 6: Journey Pass Screen

**Purpose:** The all-in-one pass for the booked journey. This is what the user shows at each leg.

**Layout:**

```
┌─────────────────────────────────┐
│  Your Journey Pass              │
│  ─────────────────────────────  │
│                                 │
│  RF-PASS-9182                   │ ← Pass ID, large bold
│                                 │
│  ┌─────────────────────────┐    │
│  │                         │    │
│  │     [QR Placeholder]    │    │ ← Box with QR visual or 
│  │       ▓▓▓▓▓▓▓▓▓         │    │   generated QR from pass ID
│  │       ▓░░░░░▓▓          │    │
│  │       ▓░▓▓░░▓▓          │    │
│  │       ▓░░░░░▓▓          │    │
│  │       ▓▓▓▓▓▓▓▓          │    │
│  └─────────────────────────┘    │
│                                 │
│  JOURNEY SUMMARY                │
│  Central Station → Tech Park    │
│  Walk → Bus → Metro → Auto      │
│  Total Fare: ₹82                │
│                                 │
│  TICKET IDs                     │
│  Bus:   BUS-RF-3021             │
│  Metro: MTR-RF-8841             │
│  Auto:  AUTO-RF-2219            │
│                                 │
│  ─────────────────────────────  │
│                                 │
│  ┌───────────────────────────┐  │
│  │   ▶️  Start Journey        │  │ ← Navigate to Live Tracking
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

**Component States:**

- Pass ID: largest text on screen, monospace or bold
- QR placeholder: either a pre-rendered QR image from pass ID text, or a stylized placeholder box
- Ticket IDs: small, monospace font, distinct visual treatment

**Interactions:**

- Start Journey → navigate to Live Tracking Screen
- (Optional) Share/Download button for pass screenshot

---

### Screen 7: Live Tracking Screen

**Purpose:** Simulate live journey updates after booking begins.

**Layout:**

```
┌─────────────────────────────────┐
│  Live Journey                   │
│  Walk → Bus → Metro → Auto      │ ← Route summary
│  Central Station → Tech Park    │
├─────────────────────────────────┤
│  CURRENT STATUS                 │
│                                 │
│  Step 2 of 4: Bus 335E          │ ← Highlighted current step
│  Estimated arrival: 22 min      │ ← Simulated ETA countdown
│                                 │
├─────────────────────────────────┤
│  LIVE UPDATES                   │
│                                 │
│  ✅ 09:02  Walk completed       │ ← Past updates, faded
│  ──────────────────────────── │
│  🔵 09:06  Bus 335E arriving    │ ← Current update, highlighted
│            in 4 minutes at Bay 3│
│  ──────────────────────────── │
│  ⏳ --:--  Metro – Blue Line    │ ← Future updates, muted
│            Platform 2           │
│  ⏳ --:--  Auto pickup at Gate 1│
│                                 │
├─────────────────────────────────┤
│  🟡 Minor delay: +5 min ETA    │ ← Alert banner (conditional)
│                                 │
├─────────────────────────────────┤
│  JOURNEY LEGS                   │
│  ✅ Walk    ●───○ Bus ○───○ Metro ○ Auto │
│  ─────────────────────────────  │
│                                 │
│  Booking: RF-BKG-1001           │
│  Pass: RF-PASS-9182             │
└─────────────────────────────────┘
```

**Component States:**

- Past updates: check icon, faded text, timestamp shown
- Current update: blue/indigo highlight, larger text
- Future updates: grey, no timestamp, clock icon
- Delay alert banner: amber background, shown conditionally when delay update fires
- Completion state: all steps checked, green success message "Journey Complete 🎉"

**Animation / Timing:**

- Updates appear one at a time, every 8–12 seconds (configurable in code)
- ETA countdown updates every 30 seconds (simulated decrement)
- Delay alert banner appears with the delay update and auto-dismisses after 10 seconds
- Journey Completed state shown as a full-screen celebration on last update

**Interactions:**

- No user input required on this screen
- Updates are automatic / timer-driven
- Journey Complete state shows a "Plan Another Journey" button → navigate to Home

---

## 6. Navigation Map

```
Home (Screen 1)
    │
    │ [Find Routes]
    ▼
Route Options (Screen 2)
    │
    │ [View Route / Tap card]
    ▼
Route Details (Screen 3)
    │
    │ [Book This Journey]
    ▼
Booking Summary (Screen 4)
    │
    │ [Proceed to Payment]
    ▼
Payment Screen (Screen 5)
    │
    │ [Pay ₹XX] ─── Success ──→ Journey Pass (Screen 6)
    │                               │
    │                               │ [Start Journey]
    │                               ▼
    │                          Live Tracking (Screen 7)
    │                               │
    │                               │ [Journey Complete]
    └───────────────────────────────│────────────────────────▶ Home
```

Back navigation: every screen has a back button returning to the previous screen.

---

## 7. Loading and Empty States

| Screen | Loading State | Empty / Error State |
|---|---|---|
| Home | Find Routes button shows spinner | Toast: "No routes found for this pair" |
| Route Options | Skeleton cards (3) while loading | Empty state illustration + "Try different locations" |
| Route Details | Skeleton timeline | "Route not found" + back button |
| Booking Summary | "Creating booking..." overlay | "Booking failed. Try again." |
| Payment | "Processing payment..." overlay | "Payment failed." with reason |
| Journey Pass | Immediate (data already loaded) | N/A |
| Live Tracking | "Starting journey..." for 2 seconds | N/A (updates always available from mock data) |

---

## 8. Toast / Notification System

Global toast notifications shown at bottom of screen:

- **Info:** blue background — "3 routes found for your journey"
- **Success:** green background — "Payment successful"
- **Error:** red background — "Source and destination are the same"
- **Warning:** amber background — "Delay detected on your route"

Toasts auto-dismiss after 3 seconds.

---

## 9. Responsive Behavior

The MVP is primarily mobile. On desktop:

- Max content width: 480px, centered
- Background: Slate-100
- Cards and buttons maintain mobile sizing
- No layout restructuring for MVP

---

## 10. Accessibility (MVP Minimum)

- All interactive elements keyboard-focusable
- Color not the only differentiator (use icons + text)
- Font size minimum 14px
- Touch targets minimum 44px height
- Alt text on all icons

---

## 11. Copy Guidelines

- Use plain English, no jargon
- Use ₹ symbol not "Rs" or "INR"
- Time format: "54 min" not "54 minutes" (space efficiency)
- Distance: "650m" not "0.65 km"
- Always show exact numbers, not approximate ranges
- Error messages explain the problem AND what to do next

---

*End of UX Flow Document*