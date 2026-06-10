# RideFlow – Product Requirements Document (PRD)
**Version:** 1.0  
**Author:** Product Management  
**Status:** Final – MVP Scope  
**Last Updated:** June 2026

---

## 1. Executive Summary

RideFlow is an intelligent multi-modal journey planner designed for Indian urban commuters. The product solves the fragmented commute problem: today, a commuter who travels by bus, metro, and auto must coordinate across three separate apps, three separate ticketing systems, and three separate payment flows — all mentally, in real time, every single day.

RideFlow collapses this friction into a single experience: Search → Compare → Select → Understand → Book → Pay → Track.

This PRD defines the MVP scope, feature requirements, acceptance criteria, and out-of-scope boundaries for the first engineering build.

---

## 2. Problem Statement

### 2.1 The User Pain

Daily commuters in Indian cities — students, office workers, professionals — rarely use a single mode of transport. A typical commute looks like this:

- Walk 500m to the nearest bus stop
- Take Bus 335E to a metro station (15 min)
- Take the Blue Line metro (25 min)
- Take an auto from the destination station (10 min)

Each of these steps requires the user to:
- Know the correct bus number and timing
- Know the metro line and interchange
- Hail or pre-book a last-mile cab or auto
- Pay separately at each point
- Hold multiple tickets or scan multiple QR codes
- Mentally estimate arrival times across four different modes

There is no single app in the Indian market that does all of this in one place, especially not one that explains its recommendations or simulates end-to-end booking.

### 2.2 Why This Matters Now

Urban India is seeing rapid metro expansion across cities like Pune, Hyderabad, Bengaluru, and Chennai. Yet last-mile connectivity remains the weakest link. NCMC (National Common Mobility Card) standards are now being rolled out by the government. The infrastructure for a unified commute layer is being laid — RideFlow positions itself as the intelligent interface on top of that infrastructure.

---

## 3. Goals and Non-Goals

### 3.1 Goals (MVP)

- Let a user enter source and destination and receive multiple ranked multi-modal route options
- Let the user compare routes by time, cost, transfers, walking distance, safety, reliability, and carbon score
- Let the user understand why a route is recommended (rule-based explanation)
- Let the user simulate booking each leg of the journey
- Let the user complete a mock payment via a simulated wallet
- Generate a unified journey pass after payment
- Simulate live journey updates post-booking

### 3.2 Non-Goals (MVP)

- Real NCMC payment integration
- Live GPS or vehicle tracking
- Real Ola/Uber/Rapido API booking
- Real ticketing APIs from transport authorities
- OTP-based authentication
- Multi-city route coverage
- Accessibility-specific routing
- Full map-based input (street-level search)

---

## 4. Target Users

### Primary Users

**Daily Office Commuter**
Commutes 5 days a week. Knows their route but wants to plan for delays or try faster alternatives. Values time over cost. Wants one app instead of three.

**Student**
Budget-sensitive. Uses buses and metro. Needs the cheapest route. Has time but not money. Wants clear explanations so they learn the city.

**Occasional City Traveler**
Visits the city for work or appointments. Doesn't know the local transport system. Needs guided step-by-step routing with clear instructions.

**Tourist**
First time in the city. Needs handholding. Values safety, simplicity, and not getting lost.

### Secondary Users

**Product Evaluators / Internal Demo Reviewers**
The Aerovox AI team evaluating this MVP. They need to see a clean, believable, end-to-end product flow. They are assessing technical execution, product thinking, and demo readiness.

---

## 5. User Stories

**US-01:** As a daily commuter, I want to enter my source and destination so that I can see route options without opening multiple apps.

**US-02:** As a budget traveler, I want to filter routes by cheapest fare so that I spend the minimum possible.

**US-03:** As a time-pressed professional, I want to see the fastest route highlighted so that I don't waste time comparing manually.

**US-04:** As an occasional traveler, I want step-by-step instructions for each journey leg so that I don't get confused mid-journey.

**US-05:** As a user, I want to understand why a route is recommended so that I trust the app's suggestion.

**US-06:** As a user, I want to book my full journey in one click so that I don't need to go to separate booking screens.

**US-07:** As a user, I want to pay from a single wallet so that I don't need to carry exact change or multiple payment apps.

**US-08:** As a user, I want a single journey pass so that I have everything I need in one place.

**US-09:** As a user, I want live-style updates after booking so that I know what's happening at each step of my journey.

**US-10:** As a user who cares about the environment, I want to compare carbon scores so that I can choose the more eco-friendly route.

---

## 6. Feature Requirements

### Feature 1: Source and Destination Input

**Description:** The entry point of the application. Users select a source and a destination from a predefined list of locations.

**Requirements:**
- Display a source selector (dropdown or searchable input)
- Display a destination selector (dropdown or searchable input)
- Display a route preference selector with options: Balanced, Fastest, Cheapest, Least Walking, Fewest Transfers, Eco-Friendly
- Show a Search / Find Routes button
- Validate that source and destination are different before search
- Display a loading state while routes are being fetched

**Predefined Locations (MVP dataset):**
Central Railway Station, Metro Central, City Bus Depot, Tech Park, University Campus, Residential Area North, Airport Road, Shopping Mall

**Acceptance Criteria:**
- User can select source and destination
- User can select a preference
- Clicking Search triggers the route search API
- Error message shown if source equals destination
- Loading indicator shown during fetch

---

### Feature 2: Multi-Modal Route Options

**Description:** After search, the app presents at least 3 route options as cards.

**Requirements:**
- Show a minimum of 3 route option cards
- Each card must display: Route ID, Mode sequence (e.g., Walk → Bus → Metro → Auto), Total estimated time, Total estimated fare, Number of transfers, Walking distance, Carbon score, Safety score, Reliability indicator, Recommendation tag (Recommended / Fastest / Cheapest / Eco-Friendly)
- The top-ranked route must be visually highlighted
- Cards should be tap/clickable to navigate to Route Details

**Acceptance Criteria:**
- At least 3 route cards visible on screen
- All required fields populated on each card
- Recommended tag shown on highest-scoring route
- Clicking a card navigates to Route Details screen

---

### Feature 3: Route Ranking System

**Description:** The backend must score and rank all available routes using a weighted formula. The ranking changes based on user preference.

**Scoring Formula:**

Route Score = (Wt × Nt) + (Wf × Nf) + (Wtr × Ntr) + (Ww × Nw) + reliability_penalty + carbon_penalty + safety_penalty

Where:
- Nt = normalized travel time (0–1 scale)
- Nf = normalized fare (0–1 scale)
- Ntr = normalized transfer count (0–1 scale)
- Nw = normalized walking distance (0–1 scale)
- W = weight per metric, adjusted by preference

Lower score = better route.

**Preference Weight Profiles:**

| Preference | Time | Fare | Transfers | Walking | Reliability | Carbon | Safety |
|---|---|---|---|---|---|---|---|
| Balanced | 0.25 | 0.20 | 0.15 | 0.15 | 0.10 | 0.10 | 0.05 |
| Fastest | 0.50 | 0.10 | 0.15 | 0.10 | 0.10 | 0.03 | 0.02 |
| Cheapest | 0.10 | 0.55 | 0.10 | 0.10 | 0.08 | 0.05 | 0.02 |
| Least Walking | 0.15 | 0.15 | 0.15 | 0.45 | 0.05 | 0.03 | 0.02 |
| Fewest Transfers | 0.15 | 0.15 | 0.50 | 0.10 | 0.05 | 0.03 | 0.02 |
| Eco-Friendly | 0.15 | 0.10 | 0.10 | 0.10 | 0.05 | 0.45 | 0.05 |

**Acceptance Criteria:**
- Routes are sorted by ascending score
- Selecting a different preference reorders cards
- Score values are returned in the API response

---

### Feature 4: Route Details Page

**Description:** A full step-by-step breakdown of a selected route, along with the AI/rule-based explanation and a Book button.

**Requirements:**
- Show route summary at top (same as card)
- Show a visual timeline of each journey step
- Each step must include: Mode icon, Start point name, End point name, Route name or number (if applicable), Estimated duration, Estimated fare, Platform/gate info (if applicable), Instruction text
- Show explanation section below timeline
- Show Book This Journey CTA at bottom

**Acceptance Criteria:**
- All journey steps rendered in correct sequence
- Each step shows all required fields
- Explanation text is visible and relevant to route metrics
- Book button navigates to Booking Summary screen

---

### Feature 5: Route Explanation

**Description:** Rule-based natural language explanation for why a route is ranked the way it is.

**Logic Rules (examples):**

- If route is fastest: "This route has the shortest total travel time of X minutes. It saves Y minutes compared to the next fastest option."
- If route is cheapest: "This route costs ₹X, which is ₹Y less than the recommended route."
- If route is recommended (balanced): "This route offers the best overall balance of time, cost, and walking distance. It completes the journey in X minutes at ₹Y with only Z transfers."
- If route has high walking distance: "Note: This route involves X meters of walking."
- If route has low reliability: "Note: This route includes services with variable reliability."

**Rules:**
- Explanations must only reference actual route metric values
- No text may be generated that isn't grounded in the route data
- If AI (LLM) is used, the prompt must strictly constrain the model to summarize only provided data

**Acceptance Criteria:**
- Every route has an explanation
- Explanation references at least two route metrics
- No fabricated data appears in explanation text

---

### Feature 6: Mock Booking Flow

**Description:** After selecting a route, the user initiates a booking that generates mock ticket IDs for each leg.

**Requirements:**
- POST /bookings/create called with routeId and userId
- Response includes one ticket ID per bookable leg (Bus, Metro, Auto/Cab)
- Walking legs do not require ticket IDs
- Booking status set to "Pending Payment"
- Display booking summary with all ticket IDs and fare breakdown

**Ticket ID Format:**
- Bus: BUS-RF-{4-digit random}
- Metro: MTR-RF-{4-digit random}
- Auto: AUTO-RF-{4-digit random}
- Train: TRN-RF-{4-digit random}

**Acceptance Criteria:**
- Booking created on click
- Ticket IDs visible in booking summary
- Total fare matches route card fare
- Continue to Payment button visible

---

### Feature 7: Mock Payment / Wallet Flow

**Description:** A simulated wallet payment screen.

**Requirements:**
- Display current wallet balance (mock, default ₹500)
- Display journey total cost
- Show Pay button
- On payment: deduct cost from wallet, show success screen
- Display remaining balance after payment
- Generate Journey Pass ID (format: RF-PASS-{4-digit random})
- Handle insufficient balance with an error state

**Acceptance Criteria:**
- Wallet balance displayed correctly
- Pay button triggers payment API
- Success screen shown with remaining balance
- Journey Pass ID generated and displayed
- Error shown if balance insufficient

---

### Feature 8: Unified Journey Pass

**Description:** A single-screen pass generated after successful payment.

**Requirements:**
- Journey Pass ID prominently displayed
- QR code placeholder (generated from pass ID text or a static visual)
- List of all ticket IDs
- Route summary (mode sequence)
- Total fare paid
- Start Journey button to proceed to live tracking

**Acceptance Criteria:**
- Journey Pass screen accessible after payment
- All ticket IDs listed
- QR placeholder visible
- Start Journey button navigates to Live Tracking

---

### Feature 9: Simulated Live Journey Updates

**Description:** After starting the journey, the app shows live-style status updates in sequence.

**Requirements:**
- Timer-based updates (one new update every 8–12 seconds, or loaded from mock data)
- Updates must include at minimum: vehicle arrival alerts, platform/gate information, delay notifications, last-mile pickup notifications, journey completed message
- Current journey step must be highlighted
- Simulated ETA updated dynamically
- Journey completion state shown at the end

**Acceptance Criteria:**
- Updates appear sequentially after starting journey
- At least 5 distinct update messages shown
- Journey completed state reachable
- No crash during update sequence

---

## 7. API Requirements Summary

| Endpoint | Method | Purpose |
|---|---|---|
| /routes/search | POST | Search and rank routes |
| /routes/{routeId} | GET | Get full route details |
| /routes/{routeId}/explanation | GET | Get route explanation text |
| /bookings/create | POST | Create mock booking |
| /payments/pay | POST | Execute mock payment |
| /journey/{bookingId}/updates | GET | Get simulated live updates |

Full request/response schemas are defined in the SRD.

---

## 8. Data Requirements

The MVP requires the following static datasets:

- locations.json — 8–10 named locations
- stops.json — associated stops/stations per location
- transport_legs.json — individual legs (mode, from, to, fare, time, distance, reliability, carbon, safety)
- routes.json — assembled route combinations (3+ routes per source-destination pair)
- last_mile_options.json — auto and cab options for last-mile legs
- live_updates.json — pre-written update message sequences

All data must support at least one complete end-to-end demo journey.

---

## 9. Out of Scope

The following will not be built in this MVP and must be documented as future scope:

- Real NCMC or UPI payment
- Ola/Uber/Rapido API integration
- Real-time vehicle tracking (GPS, GTFS-RT)
- Live bus/metro schedules from transport authority APIs
- OTP login / user accounts
- Map-based location input
- Multi-city support
- Accessibility routing (wheelchair, visual impairment)
- Carbon emissions API (real data)
- Real crime/safety data feeds
- Admin dashboard
- Push notifications

---

## 10. Success Metrics (MVP Review)

| Metric | Target |
|---|---|
| Full demo flow completable without error | 100% |
| Route options returned per search | ≥ 3 |
| Preference filter visibly changes route order | Yes |
| Time from search to journey pass | < 90 seconds |
| All 7 screens functional | Yes |
| README sufficient for a new engineer to run the project | Yes |

---

## 11. Assumptions and Constraints

- All transport data is static and hardcoded for MVP; no live data feeds
- Wallet balance is hardcoded at ₹500 per session; no persistence between sessions
- The demo journey (Central Railway Station → Tech Park) must work reliably and repeatedly
- QR codes are visual placeholders only; no real payment QR encoding required
- Route explanation is rule-based; LLM integration is optional if used responsibly
- The system is single-user; no authentication required for MVP

---

## 12. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Routing logic too complex to finish in time | Medium | High | Pre-define all routes in JSON; route engine assembles them, not generates them |
| LLM explanation hallucinating route facts | Medium | High | Constrain prompt strictly to route data; fall back to rule-based |
| UI polish consuming too much time | High | Medium | Use component library (Tailwind + shadcn); prioritize function over aesthetics |
| Demo data inconsistency | Low | High | Lock demo journey early (Day 2); test it every day |

---

*End of PRD*