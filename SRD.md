# RideFlow – System Requirements Document (SRD)
**Version:** 1.0  
**Author:** Engineering / Product  
**Status:** Final – MVP Scope  
**Last Updated:** June 2026

---

## 1. Purpose

This document defines the technical system requirements for the RideFlow MVP. It covers backend architecture, API contracts, data models, route scoring logic, and integration points. This document is intended for the engineering team building the system.

---

## 2. System Architecture Overview

The RideFlow MVP consists of three layers:

**Layer 1: Frontend (React.js)**
- Client-side web application
- 7 screens covering the full user journey
- Communicates with backend via REST API
- State management via React Context or useState
- No real authentication; demo-mode only

**Layer 2: Backend (Python FastAPI)**
- RESTful API server
- Route Search Module
- Route Scoring and Ranking Module
- Route Explanation Module
- Booking Module
- Payment Module
- Live Update Simulation Module
- All data loaded from JSON files at startup

**Layer 3: Data Layer (JSON Files)**
- Static datasets: locations, stops, transport legs, routes, last-mile options, live updates
- No database required for MVP
- Data loaded into memory at server start
- Optional: SQLite for persistence if booking/payment state needs to survive restarts

---

## 3. Repository Structure

```
rideflow/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── RouteCard.jsx
│   │   │   ├── JourneyTimeline.jsx
│   │   │   ├── WalletScreen.jsx
│   │   │   ├── JourneyPass.jsx
│   │   │   ├── LiveUpdateCard.jsx
│   │   │   └── PreferenceSelector.jsx
│   │   ├── screens/
│   │   │   ├── HomeScreen.jsx
│   │   │   ├── RouteOptionsScreen.jsx
│   │   │   ├── RouteDetailsScreen.jsx
│   │   │   ├── BookingSummaryScreen.jsx
│   │   │   ├── PaymentScreen.jsx
│   │   │   ├── JourneyPassScreen.jsx
│   │   │   └── LiveTrackingScreen.jsx
│   │   ├── services/
│   │   │   └── api.js
│   │   └── assets/
│   ├── package.json
│   └── tailwind.config.js
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── routes/
│   │   │   ├── search.py
│   │   │   ├── booking.py
│   │   │   ├── payment.py
│   │   │   └── journey.py
│   │   ├── services/
│   │   │   ├── route_engine.py
│   │   │   ├── scoring.py
│   │   │   ├── explanation.py
│   │   │   └── mock_data_loader.py
│   │   └── models/
│   │       ├── route_models.py
│   │       ├── booking_models.py
│   │       └── payment_models.py
│   ├── data/
│   │   ├── locations.json
│   │   ├── stops.json
│   │   ├── transport_legs.json
│   │   ├── routes.json
│   │   ├── last_mile_options.json
│   │   └── live_updates.json
│   └── requirements.txt
└── docs/
    ├── architecture.png
    ├── PRD.md
    ├── SRD.md
    ├── UX_Flow.md
    ├── api_documentation.md
    ├── data_model.md
    ├── demo_flow.md
    ├── problem_understanding.md
    ├── mvp_vs_future_scope.md
    └── test_checklist.md
```

---

## 4. Data Models

### 4.1 Location

```json
{
  "locationId": "LOC001",
  "name": "Central Railway Station",
  "type": "railway_station",
  "latitude": 18.5294,
  "longitude": 73.8742,
  "availableModes": ["train", "bus", "auto", "walk"]
}
```

**Types:** railway_station, metro_station, bus_depot, tech_hub, residential, airport_road, commercial

---

### 4.2 Stop

```json
{
  "stopId": "STP001",
  "name": "Central Railway Station Bus Stop",
  "locationId": "LOC001",
  "mode": "bus",
  "platform": "Bay 3"
}
```

---

### 4.3 Transport Leg

A transport leg is a single mode segment between two stops.

```json
{
  "legId": "LEG001",
  "from": "Central Railway Station",
  "fromStopId": "STP001",
  "to": "Metro Central",
  "toStopId": "STP005",
  "mode": "bus",
  "routeNumber": "335E",
  "durationMinutes": 18,
  "fareRupees": 15,
  "distanceMeters": 4200,
  "walkingDistanceMeters": 150,
  "reliabilityScore": 80,
  "carbonScore": 35,
  "safetyScore": 75
}
```

**Mode values:** walk, bus, metro, train, auto, cab, bike

**Score ranges:** 0–100 (higher = better)

---

### 4.4 Route

A route is an ordered list of legs forming a complete source-to-destination journey.

```json
{
  "routeId": "RF-R1",
  "source": "Central Railway Station",
  "destination": "Tech Park",
  "legs": ["LEG001", "LEG005", "LEG009"],
  "summary": "Walk → Bus → Metro → Auto",
  "totalTimeMinutes": 54,
  "totalFareRupees": 82,
  "transferCount": 2,
  "totalWalkingMeters": 650,
  "carbonLabel": "Low",
  "safetyLabel": "Medium",
  "reliabilityLabel": "High",
  "tag": "Recommended",
  "score": 0.0
}
```

Note: `score` is computed at runtime by the scoring engine based on preference. It is not stored statically.

---

### 4.5 Last Mile Option

```json
{
  "optionId": "LM001",
  "fromLocationId": "LOC003",
  "mode": "auto",
  "provider": "Local Auto",
  "estimatedFareRupees": 50,
  "estimatedDurationMinutes": 8,
  "availabilityStatus": "Available"
}
```

---

### 4.6 Booking

```json
{
  "bookingId": "RF-BKG-1001",
  "routeId": "RF-R1",
  "userId": "demo-user",
  "createdAt": "2026-06-10T09:00:00Z",
  "status": "Pending Payment",
  "legs": [
    {
      "legId": "LEG001",
      "mode": "bus",
      "ticketId": "BUS-RF-3021",
      "fareRupees": 15
    },
    {
      "legId": "LEG005",
      "mode": "metro",
      "ticketId": "MTR-RF-8841",
      "fareRupees": 35
    },
    {
      "legId": "LEG009",
      "mode": "auto",
      "ticketId": "AUTO-RF-2219",
      "fareRupees": 32
    }
  ],
  "totalFareRupees": 82
}
```

---

### 4.7 Payment

```json
{
  "paymentId": "PAY-001",
  "bookingId": "RF-BKG-1001",
  "method": "Mock NCMC Wallet",
  "amountRupees": 82,
  "status": "Success",
  "journeyPassId": "RF-PASS-9182",
  "qrCodeText": "RF-PASS-9182",
  "walletBalanceAfterRupees": 418,
  "paidAt": "2026-06-10T09:02:00Z"
}
```

---

### 4.8 Live Update

```json
{
  "updateId": "UPD001",
  "bookingId": "RF-BKG-1001",
  "sequenceOrder": 1,
  "message": "Bus 335E arriving in 4 minutes at Bay 3.",
  "type": "arrival",
  "delayMinutes": 0
}
```

**Update types:** arrival, departure, platform, delay, last_mile, completed

---

## 5. API Contracts

### 5.1 POST /routes/search

**Request:**
```json
{
  "source": "Central Railway Station",
  "destination": "Tech Park",
  "preference": "balanced"
}
```

**Response (200):**
```json
{
  "routes": [
    {
      "routeId": "RF-R1",
      "summary": "Walk → Bus → Metro → Auto",
      "totalTimeMinutes": 54,
      "totalFareRupees": 82,
      "transferCount": 2,
      "walkingDistanceMeters": 650,
      "carbonLabel": "Low",
      "safetyLabel": "Medium",
      "reliabilityLabel": "High",
      "tag": "Recommended",
      "score": 0.312
    },
    {
      "routeId": "RF-R2",
      "summary": "Walk → Train → Bus → Walk",
      "totalTimeMinutes": 68,
      "totalFareRupees": 54,
      "transferCount": 2,
      "walkingDistanceMeters": 1200,
      "carbonLabel": "Low",
      "safetyLabel": "High",
      "reliabilityLabel": "Medium",
      "tag": "Cheapest",
      "score": 0.441
    },
    {
      "routeId": "RF-R3",
      "summary": "Auto → Metro → Walk",
      "totalTimeMinutes": 42,
      "totalFareRupees": 130,
      "transferCount": 1,
      "walkingDistanceMeters": 300,
      "carbonLabel": "Medium",
      "safetyLabel": "High",
      "reliabilityLabel": "High",
      "tag": "Fastest",
      "score": 0.389
    }
  ]
}
```

**Error (400):**
```json
{
  "error": "Source and destination cannot be the same."
}
```

**Error (404):**
```json
{
  "error": "No routes found for the given source-destination pair."
}
```

---

### 5.2 GET /routes/{routeId}

**Response (200):**
```json
{
  "routeId": "RF-R1",
  "source": "Central Railway Station",
  "destination": "Tech Park",
  "summary": "Walk → Bus → Metro → Auto",
  "totalTimeMinutes": 54,
  "totalFareRupees": 82,
  "steps": [
    {
      "stepNumber": 1,
      "mode": "walk",
      "from": "Current Location",
      "to": "Bus Stop – Bay 3, Central Railway Station",
      "distanceMeters": 400,
      "durationMinutes": 6,
      "fareRupees": 0,
      "instruction": "Walk 400m south to Bus Stop Bay 3."
    },
    {
      "stepNumber": 2,
      "mode": "bus",
      "from": "Bus Stop – Bay 3, Central Railway Station",
      "to": "Metro Central Bus Stop",
      "routeNumber": "335E",
      "durationMinutes": 18,
      "fareRupees": 15,
      "instruction": "Board Bus 335E. Alight at Metro Central."
    },
    {
      "stepNumber": 3,
      "mode": "metro",
      "from": "Metro Central Station",
      "to": "Tech Park Metro Station",
      "lineName": "Blue Line",
      "platform": "Platform 2",
      "durationMinutes": 22,
      "fareRupees": 35,
      "instruction": "Take Blue Line towards Airport Road. Alight at Tech Park."
    },
    {
      "stepNumber": 4,
      "mode": "auto",
      "from": "Tech Park Metro Station Gate 1",
      "to": "Tech Park Main Entrance",
      "durationMinutes": 8,
      "fareRupees": 32,
      "instruction": "Take auto from Gate 1. Estimated ₹32."
    }
  ]
}
```

---

### 5.3 GET /routes/{routeId}/explanation

**Response (200):**
```json
{
  "routeId": "RF-R1",
  "tag": "Recommended",
  "explanation": "This route is recommended because it offers the best overall balance of time, cost, and walking distance. It completes the journey in 54 minutes at ₹82 with 2 transfers and 650m of walking. It is 12 minutes faster than the cheapest option and requires 900m less walking than the train route. The reliability of this route is rated High based on historical service performance."
}
```

---

### 5.4 POST /bookings/create

**Request:**
```json
{
  "routeId": "RF-R1",
  "userId": "demo-user"
}
```

**Response (200):**
```json
{
  "bookingId": "RF-BKG-1001",
  "routeId": "RF-R1",
  "status": "Pending Payment",
  "legs": [
    { "mode": "bus", "ticketId": "BUS-RF-3021", "fareRupees": 15 },
    { "mode": "metro", "ticketId": "MTR-RF-8841", "fareRupees": 35 },
    { "mode": "auto", "ticketId": "AUTO-RF-2219", "fareRupees": 32 }
  ],
  "totalFareRupees": 82
}
```

---

### 5.5 POST /payments/pay

**Request:**
```json
{
  "bookingId": "RF-BKG-1001",
  "paymentMethod": "Mock NCMC Wallet"
}
```

**Response (200 – Success):**
```json
{
  "paymentStatus": "Success",
  "journeyPassId": "RF-PASS-9182",
  "qrCodeText": "RF-PASS-9182",
  "walletBalance": 418
}
```

**Response (400 – Insufficient Balance):**
```json
{
  "paymentStatus": "Failed",
  "reason": "Insufficient wallet balance.",
  "walletBalance": 30,
  "requiredAmount": 82
}
```

---

### 5.6 GET /journey/{bookingId}/updates

**Response (200):**
```json
{
  "bookingId": "RF-BKG-1001",
  "currentStep": 2,
  "updates": [
    { "order": 1, "message": "Bus 335E arriving in 4 minutes at Bay 3.", "type": "arrival" },
    { "order": 2, "message": "You are on board Bus 335E. Next stop: Metro Central.", "type": "departure" },
    { "order": 3, "message": "Metro expected on Platform 2. Blue Line departing in 3 minutes.", "type": "platform" },
    { "order": 4, "message": "Minor delay detected on Blue Line. ETA updated by 5 minutes.", "type": "delay" },
    { "order": 5, "message": "Auto driver arriving at Gate 1. Booking ID: AUTO-RF-2219.", "type": "last_mile" },
    { "order": 6, "message": "You have arrived at Tech Park. Journey complete.", "type": "completed" }
  ]
}
```

---

## 6. Route Scoring Engine – Technical Specification

### 6.1 Normalization

Each metric is normalized across all routes in the result set:

```python
normalized_value = (value - min_value) / (max_value - min_value)
```

If all values are equal (max == min), normalized value = 0 for all routes.

### 6.2 Penalty Scores

Penalties are additive constants based on label values:

**Reliability Penalty:**
- High reliability → 0.00
- Medium reliability → 0.05
- Low reliability → 0.15

**Carbon Penalty:**
- Low carbon → 0.00
- Medium carbon → 0.05
- High carbon → 0.12

**Safety Penalty:**
- High safety → 0.00
- Medium safety → 0.04
- Low safety → 0.10

### 6.3 Label Conversion

Carbon label is derived from carbonScore (0–100):
- 0–33 → Low
- 34–66 → Medium
- 67–100 → High

Safety label from safetyScore:
- 75–100 → High
- 50–74 → Medium
- 0–49 → Low

Reliability label from reliabilityScore:
- 80–100 → High
- 60–79 → Medium
- 0–59 → Low

### 6.4 Tag Assignment

After scoring:
- Rank 1 by balanced score → tag = "Recommended"
- Lowest fare → tag = "Cheapest" (if not already Recommended)
- Lowest time → tag = "Fastest" (if not already tagged)
- Lowest carbon → tag = "Eco-Friendly" (if not already tagged)

---

## 7. Explanation Engine – Technical Specification

The explanation engine is rule-based. It takes the scored route object and produces a human-readable paragraph using the following logic:

```python
def generate_explanation(route, all_routes, preference):
    parts = []
    
    sorted_by_time = sorted(all_routes, key=lambda r: r.totalTimeMinutes)
    sorted_by_fare = sorted(all_routes, key=lambda r: r.totalFareRupees)
    
    if route.tag == "Recommended":
        parts.append(f"This route offers the best overall balance of time, cost, and walking distance.")
    
    time_rank = sorted_by_time.index(route) + 1
    fare_rank = sorted_by_fare.index(route) + 1
    
    parts.append(f"It completes the journey in {route.totalTimeMinutes} minutes at ₹{route.totalFareRupees} with {route.transferCount} transfer(s).")
    
    if time_rank == 1:
        parts.append(f"This is the fastest available route.")
    else:
        faster = sorted_by_time[0]
        parts.append(f"It is {faster.totalTimeMinutes - route.totalTimeMinutes} minutes slower than the fastest route but ₹{route.totalFareRupees - faster.totalFareRupees} cheaper.")
    
    if route.reliabilityLabel == "High":
        parts.append("The services on this route have high historical reliability.")
    
    if route.totalWalkingMeters > 800:
        parts.append(f"Note: This route involves {route.totalWalkingMeters}m of walking.")
    
    return " ".join(parts)
```

**Rule:** If an LLM is used, the system prompt must read: "You are a route explanation assistant. Summarize the following route data in 2–3 sentences. Do not add any facts that are not present in the data provided. Do not invent times, fares, or distances."

---

## 8. Mock Wallet State

The wallet is a server-side in-memory dictionary keyed by userId:

```python
wallet_balances = {
    "demo-user": 500
}
```

On payment:
1. Check if balance >= booking total
2. If yes: deduct, generate pass ID, return success
3. If no: return failure with current balance

Wallet state resets to ₹500 on server restart (acceptable for MVP demo).

---

## 9. Error Handling Requirements

| Scenario | HTTP Status | Error Message |
|---|---|---|
| Source == Destination | 400 | "Source and destination cannot be the same." |
| No routes found | 404 | "No routes found for this journey." |
| Invalid routeId | 404 | "Route not found." |
| Invalid bookingId | 404 | "Booking not found." |
| Insufficient wallet balance | 400 | "Insufficient wallet balance." |
| Missing required fields | 422 | FastAPI default validation error |
| Server error | 500 | "Internal server error. Please try again." |

---

## 10. CORS and Environment Configuration

The backend must enable CORS for localhost during development:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Frontend environment variable: `REACT_APP_API_BASE_URL=http://localhost:8000`

---

## 11. Tech Stack Summary

| Component | Technology | Rationale |
|---|---|---|
| Frontend | React.js + Tailwind CSS | Fast development, wide familiarity, clean UI |
| Backend | Python FastAPI | Async, auto-docs, fast to prototype |
| Route Engine | Python (pure logic) | Scoring formula is math, not graph traversal for MVP |
| Data Storage | JSON files | No DB setup overhead; clean for MVP |
| State Management | React useState / Context | Sufficient for 7-screen linear flow |
| Build Tool | Vite or CRA | Standard React setup |

---

## 12. Performance Requirements (MVP)

| Metric | Target |
|---|---|
| Route search API response time | < 500ms |
| Route details API response time | < 200ms |
| Frontend initial load | < 3 seconds |
| Full demo flow (search to journey pass) | < 2 minutes |
| Server uptime during demo | 100% (single demo session) |

---

## 13. Security Considerations (MVP)

This is an internal demo MVP. The following security measures are intentionally deferred:

- No authentication required
- No input sanitization beyond Pydantic validation
- No HTTPS required (localhost only)
- No secrets management required (no real API keys)

These must be addressed before any public deployment.

---

*End of SRD*