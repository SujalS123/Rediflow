# RideFlow – Problem Understanding Note
**Author:** Engineering Intern  
**Document:** docs/problem_understanding.md

---

## The Problem

India's urban commute is fundamentally multi-modal. No one in a city like Pune, Hyderabad, or Bengaluru travels from home to work using exactly one vehicle. The typical journey involves walking, a bus, a metro, and a last-mile auto or cab — often in that exact combination.

Yet every system these commuters depend on is built in isolation:

- The BEST app tells you about bus routes but not about the metro connection at the end
- Google Maps approximates but doesn't book, doesn't issue a pass, doesn't track in real time
- Ola and Rapido cover last-mile but have no idea you just got off a Blue Line metro
- NCMC cards exist but work at metro turnstiles, not on city buses in most cities

The result is that the commuter carries the integration burden in their own head, every single day.

## Who Carries This Burden

The people most affected are not infrequent travelers — they are the people who take the same journey 22 days a month. The daily office commuter who knows Bus 335E but doesn't know if there's a faster alternative. The student who takes the metro because they don't know that a direct bus would save them 20 minutes. The first-time visitor who stands at the metro exit not knowing how to find an auto at 10pm.

These are not edge cases. They are the majority of public transit users in Indian cities.

## Why Route Comparison Matters

The same journey between two points can be done in dramatically different ways. One user wants the cheapest. Another wants the fastest. A third is eight months pregnant and wants to minimize walking. A fourth is an environmentalist and wants the lowest carbon footprint.

A system that shows only one "best" route is imposing a single value judgment on every user. RideFlow shows ranked options and lets the user choose based on what matters to them — today, for this journey.

## Why Booking and Payment Are Simulated

The MVP cannot access real ticketing APIs. BEST bus ticketing, metro QR systems, and ride-hailing APIs (Ola, Rapido, Uber) require formal partnerships, MoUs with transport authorities, and often government approvals.

The simulation serves a specific purpose: it demonstrates how the system would work. A product reviewer can see the complete flow — from route selection to unified pass — and evaluate whether the concept is sound, even if the underlying integrations are mocked. This is standard practice in transport tech prototyping.

## Why Live Tracking Is Simulated

Real-time vehicle tracking requires GTFS-RT feeds from transport operators. Most Indian public transit operators either do not publish these feeds publicly or do so with significant latency and coverage gaps. For an MVP, simulated updates demonstrate the interaction model — what the user sees, when, and in what format — without requiring infrastructure that isn't available.

## MVP Scope

The MVP proves one thing: a commuter can go from "I need to get somewhere" to "I have everything I need for my journey" in a single app, in under two minutes.

Everything beyond that — real payments, live tracking, multi-city support — is the growth roadmap that this MVP makes possible.

---
---
---

# RideFlow – Architecture Document
**Document:** docs/architecture.md

---

## System Architecture

RideFlow MVP uses a clean three-layer architecture: Frontend (React.js), Backend (FastAPI), and Data (JSON files). There are no external dependencies, no third-party APIs, and no database server required for the MVP. This makes it trivially runnable and demoable on any laptop.

```
┌─────────────────────────────────────────────────────────┐
│                     FRONTEND (React.js)                  │
│                                                          │
│  HomeScreen → RouteOptions → RouteDetails → Booking      │
│  → Payment → JourneyPass → LiveTracking                  │
│                                                          │
│  api.js (Axios/Fetch calls to backend)                   │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTP REST (localhost:8000)
┌───────────────────────▼─────────────────────────────────┐
│                    BACKEND (FastAPI)                      │
│                                                          │
│  ┌─────────────────┐  ┌──────────────────────────────┐  │
│  │  Route Search   │  │  Route Scoring Engine         │  │
│  │  /routes/search │  │  (scoring.py)                 │  │
│  └────────┬────────┘  │  Weighted formula + normalize │  │
│           │           └──────────────────────────────┘  │
│  ┌────────▼────────┐  ┌──────────────────────────────┐  │
│  │  Route Details  │  │  Explanation Engine           │  │
│  │  /routes/{id}   │  │  (explanation.py)             │  │
│  └─────────────────┘  │  Rule-based NL generation    │  │
│                       └──────────────────────────────┘  │
│  ┌─────────────────┐  ┌──────────────────────────────┐  │
│  │  Booking Module │  │  Payment Module               │  │
│  │  /bookings/     │  │  /payments/pay                │  │
│  │  create         │  │  Mock wallet deduction        │  │
│  └─────────────────┘  └──────────────────────────────┘  │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Live Update Simulation                          │    │
│  │  /journey/{bookingId}/updates                    │    │
│  │  Returns pre-defined update sequence from JSON   │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Data Loader (mock_data_loader.py)               │    │
│  │  Reads all JSON files at startup                 │    │
│  │  Stores in memory as Python dicts                │    │
│  └─────────────────────────────────────────────────┘    │
└───────────────────────┬─────────────────────────────────┘
                        │ File I/O at startup
┌───────────────────────▼─────────────────────────────────┐
│                    DATA LAYER (JSON Files)                │
│                                                          │
│  locations.json    stops.json    transport_legs.json      │
│  routes.json       last_mile_options.json                 │
│  live_updates.json                                        │
└─────────────────────────────────────────────────────────┘
```

## Module Responsibilities

**Route Search (search.py)**
Receives source, destination, preference. Filters routes.json for matching source-destination pairs. Passes to scoring engine.

**Scoring Engine (scoring.py)**
Normalizes all route metrics. Applies preference-specific weights. Computes final score per route. Sorts ascending. Assigns tags.

**Explanation Engine (explanation.py)**
Takes a route object and all competing routes. Produces a plain-English explanation using rule-based template logic. No external API calls.

**Booking Module (booking.py)**
Creates a booking record. Generates random ticket IDs per bookable leg. Sets status to "Pending Payment". Stores in memory dict.

**Payment Module (payment.py)**
Checks wallet balance. Deducts amount. Generates Journey Pass ID. Updates booking status to "Paid". Returns pass details.

**Live Update Simulation (journey.py)**
Returns the pre-defined update sequence for the relevant journey type. Frontend polls or receives all updates at once and displays them with timers.

---
---
---

# RideFlow – Data Model Document
**Document:** docs/data_model.md

---

## Overview

All data in the RideFlow MVP is stored as static JSON files. These files are loaded into memory at server startup. There is no live database.

## File Descriptions

### locations.json
Named places a user can select as source or destination. 8–10 entries covering all demo journeys.

**Fields:** locationId, name, type, latitude (optional), longitude (optional), availableModes

**Types:** railway_station, metro_station, bus_depot, tech_hub, residential, airport_road, commercial, university

### stops.json
Physical boarding/alighting points associated with a location. A location can have multiple stops (e.g., Central Station has a Bus Stop, a Train Platform, and a Walk exit point).

**Fields:** stopId, name, locationId, mode, platform, gate

### transport_legs.json
Individual transport segments. Each leg is one mode between two stops. This is the atomic unit of a journey.

**Fields:** legId, from, fromStopId, to, toStopId, mode, routeNumber, durationMinutes, fareRupees, distanceMeters, walkingDistanceMeters, reliabilityScore (0–100), carbonScore (0–100), safetyScore (0–100)

### routes.json
Pre-assembled route combinations. Each route is an ordered list of leg IDs. Routes are keyed by source-destination pair.

**Fields:** routeId, source, destination, legs (array of legIds), summary, totalTimeMinutes, totalFareRupees, transferCount, totalWalkingMeters

Note: carbonLabel, safetyLabel, reliabilityLabel, tag, and score are computed at runtime by the scoring engine — not stored in this file.

### last_mile_options.json
Available auto/cab options from specific locations. Used to populate last-leg data.

**Fields:** optionId, fromLocationId, mode, provider, estimatedFareRupees, estimatedDurationMinutes, availabilityStatus

### live_updates.json
Pre-defined update message sequences. Keyed by route type or journeyPattern.

**Fields:** patternId, updates (array of: order, message, type, delayMinutes)

---

## Entity Relationships

```
Location (1) ──── (many) Stop
Stop (1) ──── (many) TransportLeg.from / TransportLeg.to
TransportLeg (many) ──── (1) Route.legs[]
Route (1) ──── (1) Booking
Booking (1) ──── (1) Payment
Payment (1) ──── (1) JourneyPass
```

---

## Sample Data – Demo Journey

The primary demo journey is: Central Railway Station → Tech Park

Three routes are pre-defined for this pair:

**RF-R1 (Recommended):** Walk(LEG-W1) → Bus 335E(LEG-B1) → Blue Line Metro(LEG-M1) → Auto(LEG-A1)
Total: 54 min, ₹82, 2 transfers, 650m walking

**RF-R2 (Cheapest):** Walk(LEG-W2) → Train(LEG-T1) → Bus(LEG-B2) → Walk(LEG-W3)
Total: 68 min, ₹54, 2 transfers, 1200m walking

**RF-R3 (Fastest):** Auto(LEG-A2) → Blue Line Metro(LEG-M2) → Walk(LEG-W4)
Total: 42 min, ₹130, 1 transfer, 300m walking

---
---
---

# RideFlow – API Documentation
**Document:** docs/api_documentation.md

---

## Base URL
Development: http://localhost:8000
All responses: Content-Type: application/json

## Authentication
None required for MVP. All endpoints are open.

---

## POST /routes/search

Search and rank routes between two locations.

**Request Body:**
```
{
  "source": string,        // Must match a name in locations.json
  "destination": string,   // Must match a name in locations.json
  "preference": string     // One of: balanced, fastest, cheapest, least_walking, fewest_transfers, eco_friendly
}
```

**Success Response (200):**
Returns array of ranked route summary objects, sorted by ascending score.

**Error Responses:**
- 400: Source equals destination
- 404: No routes found for pair
- 422: Missing or invalid fields

---

## GET /routes/{routeId}

Get full step-by-step details for a specific route.

**Path Parameter:** routeId (e.g., RF-R1)

**Success Response (200):**
Returns route object with expanded steps array. Each step includes mode, from, to, durationMinutes, fareRupees, instruction text, platform/gate info.

**Error Response:**
- 404: Route not found

---

## GET /routes/{routeId}/explanation

Get the rule-based explanation for a route.

**Success Response (200):**
```
{
  "routeId": string,
  "tag": string,
  "explanation": string    // 2-4 sentence explanation grounded in route metrics
}
```

---

## POST /bookings/create

Create a mock booking for a selected route.

**Request Body:**
```
{
  "routeId": string,
  "userId": string         // Use "demo-user" for MVP
}
```

**Success Response (200):**
Returns bookingId, array of legs with ticketIds, totalFare, status: "Pending Payment"

---

## POST /payments/pay

Execute mock wallet payment for a booking.

**Request Body:**
```
{
  "bookingId": string,
  "paymentMethod": string   // Use "Mock NCMC Wallet"
}
```

**Success Response (200):**
Returns paymentStatus: "Success", journeyPassId, qrCodeText, walletBalance (after deduction)

**Error Response (400):**
Returns paymentStatus: "Failed", reason, walletBalance, requiredAmount

---

## GET /journey/{bookingId}/updates

Get simulated live updates for a booked journey.

**Success Response (200):**
Returns array of update objects with order, message, type fields.

---
---
---

# RideFlow – Demo Flow Document
**Document:** docs/demo_flow.md

---

## Demo Overview

This is the script for the 3–5 minute internal demo video submission. Follow this sequence exactly. Do not improvise. All demo data must be in place before recording.

**Demo Journey:** Central Railway Station → Tech Park · Preference: Balanced

---

## Step-by-Step Demo Script

**[0:00 – 0:20] Open the app**
Show the Home screen. Point out the RideFlow branding and tagline. Explain: "RideFlow is a unified multi-modal journey planner for Indian urban commuters."

**[0:20 – 0:45] Enter source and destination**
Select "Central Railway Station" as source. Select "Tech Park" as destination. Leave preference as "Balanced". Click "Find Routes".

**[0:45 – 1:20] Route Options Screen**
Three route cards appear. Point out: "RideFlow shows multiple options — Recommended, Cheapest, and Fastest — each with time, cost, transfers, walking distance, carbon and safety scores." Highlight the Recommended card. Click "View Route" on Recommended.

**[1:20 – 2:00] Route Details Screen**
Show the step-by-step timeline. Walk through each step: Walk 400m, Bus 335E for 18 min, Blue Line Metro for 22 min, Auto for 8 min. Point out: "Each step has instructions, platform info, and fare." Scroll down to explanation. Read it aloud or summarize. Click "Book This Journey".

**[2:00 – 2:30] Booking Summary**
Show ticket IDs: BUS-RF-3021, MTR-RF-8841, AUTO-RF-2219. Show fare breakdown totalling ₹82. Click "Proceed to Payment".

**[2:30 – 3:00] Mock Payment**
Show wallet balance ₹500. Show journey cost ₹82. Click "Pay ₹82". Show success screen with remaining balance ₹418 and Journey Pass ID RF-PASS-9182. Click "View Journey Pass".

**[3:00 – 3:30] Journey Pass**
Show the pass ID, QR placeholder, and all ticket IDs together. Say: "This is the single unified pass for the whole journey." Click "Start Journey".

**[3:30 – 4:30] Live Tracking**
Show updates appearing sequentially: bus arrival, boarding, metro platform, delay alert, auto pickup, journey completed. Highlight the delay notification: "RideFlow proactively tells you about delays so you can adjust."

**[4:30 – 5:00] Closing**
Return to home screen or show completion state. Close with: "RideFlow turns a fragmented daily commute into one intelligent, bookable, explainable journey."

---

## Demo Checklist (Run Before Recording)

- Backend running on localhost:8000
- Frontend running on localhost:3000
- Demo journey data loaded (verify /routes/search returns 3 routes)
- Wallet balance reset to ₹500
- No stale booking IDs from previous test runs
- Screen recording tool active before starting
- Phone/browser in landscape or portrait as preferred
- No terminal or dev tools visible in recording

---
---
---

# RideFlow – MVP vs Future Scope
**Document:** docs/mvp_vs_future_scope.md

---

## What Is Real in the MVP

The following are genuine, functional implementations:

- Frontend React application with 7 working screens
- FastAPI backend with 6 functional endpoints
- Route scoring and ranking engine with preference-based weights
- Rule-based route explanation generation
- Mock booking record creation with generated ticket IDs
- Mock payment processing with wallet deduction logic
- Journey pass generation with pass ID
- Simulated live journey updates from pre-loaded data
- Static transport dataset covering the demo journey

## What Is Simulated / Mocked in the MVP

The following are intentionally simulated and do not involve real systems:

| Feature | What Is Mocked | Why |
|---|---|---|
| Ticket booking | Ticket IDs are randomly generated | No access to BEST/Metro ticketing APIs |
| NCMC Payment | Wallet balance is in-memory, no real card | Requires RBI-regulated fintech integration |
| Auto/Cab booking | Booking ID is generated | No Ola/Rapido/Uber API access |
| Live vehicle tracking | Updates are pre-written sequences | No GTFS-RT feeds available |
| Safety scores | Static values in JSON | No real crime or safety data API |
| Carbon scores | Static values in JSON | No emissions calculation API |
| User accounts | No auth, single demo-user | No user management system built |
| Fare calculation | Hardcoded in JSON | No live fare API from transport operators |

## Future Scope (Post-MVP Roadmap)

**Phase 2 – Real Data Layer**
- Integrate GTFS-RT feeds for live bus/metro schedules
- Partner with metro operators for real fare and schedule data
- Integrate city-specific mobility APIs where available

**Phase 3 – Real Booking and Payment**
- NCMC card integration (National Common Mobility Card standard)
- Ola/Rapido/Uber API integration for last-mile booking
- BEST/PMPML bus ticketing API integration
- UPI deep link for payment

**Phase 4 – Intelligence Layer**
- ML-based delay prediction using historical GTFS data
- Crowding prediction per bus/metro line
- Personalized route preference learning per user
- Dynamic re-routing when disruptions are detected

**Phase 5 – Platform Expansion**
- Multi-city coverage (Bengaluru, Hyderabad, Chennai, Mumbai)
- Open API for third-party developers
- Accessibility routing (wheelchair, visual impairment)
- Carbon tracking and gamification

---
---
---

# RideFlow – Test Checklist
**Document:** docs/test_checklist.md

---

## How to Use This Checklist

Run through all items before final submission. Each item must be marked PASS. Any FAIL must be fixed or documented with a known limitation note.

---

## Backend Tests

- [ ] Backend starts without errors (uvicorn app.main:app --reload)
- [ ] GET / returns 200 (health check)
- [ ] POST /routes/search with valid source/destination returns 3 routes
- [ ] POST /routes/search with invalid source returns 404
- [ ] POST /routes/search with same source and destination returns 400
- [ ] POST /routes/search with preference "fastest" returns routes sorted by time
- [ ] POST /routes/search with preference "cheapest" returns routes sorted by fare
- [ ] GET /routes/RF-R1 returns full step-by-step route
- [ ] GET /routes/INVALID returns 404
- [ ] GET /routes/RF-R1/explanation returns non-empty explanation text
- [ ] Explanation text contains at least one numeric metric (time, fare, or distance)
- [ ] POST /bookings/create returns bookingId and ticket IDs
- [ ] Ticket IDs follow format: BUS-RF-XXXX, MTR-RF-XXXX, AUTO-RF-XXXX
- [ ] POST /payments/pay returns success when balance >= fare
- [ ] POST /payments/pay returns failure when balance < fare
- [ ] Journey Pass ID generated on success (format RF-PASS-XXXX)
- [ ] Wallet balance correctly decremented after payment
- [ ] GET /journey/RF-BKG-1001/updates returns at least 5 updates
- [ ] All API responses are valid JSON
- [ ] All API responses return in under 500ms

---

## Frontend Tests

- [ ] Home screen loads without errors
- [ ] Source and destination dropdowns populated with location list
- [ ] Preference pills selectable (only one active at a time)
- [ ] Validation error shown when source == destination
- [ ] Loading state shown during route search
- [ ] Route options screen shows 3 route cards
- [ ] Recommended card visually distinct
- [ ] All route metrics visible on cards (time, fare, transfers, walking, carbon, safety)
- [ ] Clicking a route card navigates to Route Details
- [ ] Route Details shows timeline with correct number of steps
- [ ] Each step shows mode, from, to, duration, fare, instruction
- [ ] Explanation section visible on Route Details
- [ ] Book This Journey button navigates to Booking Summary
- [ ] Booking Summary shows all ticket IDs
- [ ] Fare breakdown sums to correct total
- [ ] Proceed to Payment button works
- [ ] Payment screen shows wallet balance
- [ ] Payment screen shows journey cost
- [ ] Pay button triggers payment and shows success state
- [ ] Success state shows remaining balance and pass ID
- [ ] Journey Pass screen shows Pass ID, QR placeholder, ticket IDs
- [ ] Start Journey button navigates to Live Tracking
- [ ] Live Tracking shows updates appearing sequentially
- [ ] Delay alert banner appears with delay update
- [ ] Journey Completed state shown after last update
- [ ] Back navigation works on all screens
- [ ] No console errors in browser during full flow

---

## End-to-End Flow Test

Run the full demo flow exactly as described in demo_flow.md:

- [ ] Search Central Railway Station → Tech Park, Balanced
- [ ] View Recommended route details
- [ ] Book the journey
- [ ] Pay with mock wallet
- [ ] View journey pass
- [ ] Track live updates to completion
- [ ] No errors, no crashes, no broken UI at any step

---

## Known Limitations (Document Before Submission)

List any items that do not pass and the reason:

1. [Item] — [Reason] — [Impact]

---
---
---

# RideFlow – README
**Document:** README.md

---

# RideFlow 🚇

**Intelligent multi-modal journey planner for Indian urban commuters**

RideFlow helps users plan, compare, and simulate booking of combined journeys using Metro, Bus, Train, Walking, Auto, and Cab — all in one place.

Search → Compare → Select → Understand → Book → Pay → Track

---

## Problem Statement

Urban commuters in India use multiple transport modes every day but have no single platform to plan, book, and track their complete journey. RideFlow solves this with a unified intelligent commute assistant.

---

## Solution Overview

RideFlow provides:
- Multi-modal route planning across bus, metro, train, auto, walk, and cab
- Preference-based route ranking (fastest, cheapest, eco-friendly, least walking, etc.)
- Rule-based route explanations
- Mock end-to-end booking and payment
- Unified journey pass with QR placeholder
- Simulated live journey tracking

---

## MVP Features

1. Source and destination input with predefined locations
2. Multi-modal route options (minimum 3 per journey)
3. Route comparison cards with time, fare, transfers, walking, safety, carbon, reliability
4. Preference-based route ranking
5. Step-by-step route details with instructions
6. Rule-based route explanation
7. Mock booking with generated ticket IDs
8. Mock NCMC wallet payment
9. Unified journey pass generation
10. Simulated live journey updates

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React.js, Tailwind CSS |
| Backend | Python 3.11, FastAPI |
| Data | JSON files (static) |
| Route Engine | Python (scoring formula) |
| Explanation | Rule-based Python logic |

---

## Architecture

See docs/architecture.md for full diagram.

Three-layer architecture: React frontend → FastAPI backend → JSON data files. No external APIs. No database. Fully runnable offline.

---

## Setup Instructions

### Prerequisites
- Node.js 18+
- Python 3.11+
- pip

### Backend Setup

```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

Backend runs at: http://localhost:8000
API docs (auto-generated): http://localhost:8000/docs

### Frontend Setup

```bash
cd frontend
npm install
npm start
```

Frontend runs at: http://localhost:3000

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | /routes/search | Search and rank routes |
| GET | /routes/{routeId} | Get route details |
| GET | /routes/{routeId}/explanation | Get route explanation |
| POST | /bookings/create | Create mock booking |
| POST | /payments/pay | Execute mock payment |
| GET | /journey/{bookingId}/updates | Get live updates |

Full request/response schemas: docs/api_documentation.md

---

## Demo Flow

See docs/demo_flow.md for the complete demo script.

Quick summary: Open app → Enter Central Railway Station to Tech Park → View 3 route options → Select Recommended → View step-by-step details → Book journey → Pay ₹82 from mock wallet → View unified journey pass → Track simulated live updates → Journey complete.

---

## What Is Real vs Simulated

### Real (Functional)
- React frontend with 7 screens
- FastAPI backend with 6 endpoints
- Route scoring and ranking engine
- Rule-based explanation engine
- Mock booking record system
- Mock payment and wallet logic
- Journey pass generation
- Simulated update UI

### Simulated (Mocked)
- Ticket booking (IDs are randomly generated)
- NCMC payment (in-memory wallet only)
- Ride-hailing booking (no Ola/Rapido API)
- Real-time vehicle tracking (pre-written sequences)
- Safety scores (static values)
- Carbon scores (static values)

---

## Future Scope

See docs/mvp_vs_future_scope.md for the full roadmap.

Key items: GTFS-RT live feeds, real NCMC payment, Ola/Rapido API, ML-based delay prediction, multi-city expansion, accessibility routing, personalization.

---

## Known Limitations

- Wallet balance resets to ₹500 on server restart
- Only the demo journey (Central Station → Tech Park) is fully populated in the dataset
- Route data is static and does not reflect real schedules or fares
- QR code is a visual placeholder only
- No user authentication; single demo-user session only
- Live updates are pre-scripted, not dynamically generated

---

## Project Structure

```
rideflow/
├── frontend/       React app
├── backend/        FastAPI app + JSON data
├── docs/           All documentation
└── README.md
```

---

*Built for the Aerovox AI Engineer Internship Assignment.*
*RideFlow turns fragmented daily commute into one intelligent, bookable, explainable journey.*