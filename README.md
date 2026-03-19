# FlowSure

### AI-Powered Parametric Income Insurance for Q-Commerce Delivery Partners
**Guidewire DEVTrails 2026** | Persona: Grocery / Q-Commerce (Zepto & Blinkit) | Platform: Web + Mobile

---

## Repository Structure

```
gigshield/
├── README.md               ← Idea document (this file)
├── docs/                   ← Architecture diagrams, wireframes (added in Phase 2)
├── src/
│   ├── backend/            ← Python + FastAPI
│   ├── mobile/             ← Flutter (worker app)
│   └── web/                ← React (insurer dashboard)
└── video-link.txt          ← Phase 1 strategy video link
```

---

## Table of Contents

1. [Problem Definition](#1-problem-definition)
2. [Persona Definition](#2-persona-definition)
3. [Income Model](#3-income-model-mathematical)
4. [Income Loss Model](#4-income-loss-model-surge-corrected)
5. [Dark Store as the Risk Unit](#5-dark-store-as-the-risk-unit)
6. [Parametric Trigger System](#6-parametric-trigger-system)
7. [Income Drop Detection Mechanism](#7-income-drop-detection-mechanism)
8. [Simulation Strategy](#8-simulation-strategy)
9. [Pricing Model](#9-pricing-model)
10. [Adverse Selection Controls](#10-adverse-selection-controls)
11. [Fraud Detection System](#11-fraud-detection-system-7-layers)
12. [System Workflow](#12-system-workflow-end-to-end)
13. [AI/ML Integration](#13-aiml-integration)
14. [Tech Stack](#14-tech-stack)
15. [Platform Justification](#15-platform-justification)
16. [Differentiation](#16-differentiation)
17. [Future Scope](#17-future-scope)

---

## 1. Problem Definition

**Who is affected:**
Platform-based delivery riders working for Zepto and Blinkit — India's 10–20 minute grocery delivery networks. These riders are permanently assigned to a single dark store and operate within a 2–4 km radius of it.

**What kind of loss:**
Income loss only. When external disruptions cause the platform to pause operations or reduce order volume at a dark store, riders earn nothing for the duration. This lost hourly wage is the only thing GigShield covers.

**Why current systems fail:**
- No insurance product today compensates gig workers for lost daily wages caused by external disruptions
- Traditional insurance requires manual claim filing, documents, and days of processing — a rider cannot wait
- Riders are informal workers: no employer liability, no leave pay, no savings buffer
- Weekly platform payouts mean even a 3-hour disruption causes an immediate cash crisis that cannot be deferred

**What is explicitly excluded:**
Vehicle repair, health, accidents, medical bills, or any loss not directly attributable to lost delivery income during a verified disruption event.

---

## 2. Persona Definition

**Name:** Ravi
**Platform:** Zepto (dark store model)
**City:** Bengaluru (primary) / Chennai (secondary)
**Device:** Android smartphone, UPI-enabled

| Attribute | Detail |
|---|---|
| Active hours/day | 8–10 hours |
| Days worked/week | 6 days |
| Peak demand windows | 8–10 AM, 12–2 PM, 6–9 PM |
| Off-peak windows | 2–5 PM |
| Delivery radius | 2–4 km from assigned dark store |
| Avg delivery time | 12–18 min per order |
| Dark store assignment | Fixed — cannot switch to a busier zone |

**Why fixed dark store assignment matters:**
Unlike Zomato/Swiggy riders who can relocate to busier zones during a disruption, Zepto/Blinkit riders are tethered to one store. When that store pauses, Ravi has zero income alternatives. This makes parametric protection both more valuable and more actuarially clean — the risk unit is unambiguous.

---

## 3. Income Model (Mathematical)

**Assumptions:**
- Orders/hour (normal, no disruption): **3 orders/hr**
- Earnings/order (delivery fee + incentive average): **₹55/order**
- Platform efficiency factor (idle wait, app lag, traffic): **0.85**

**Hourly income — normal conditions:**
```
Hourly_Income = 3 × ₹55 × 0.85 = ₹140/hr
```

**Daily income (8-hour shift):**
```
Daily_Income = 8 × ₹140 = ₹1,120/day
```

**Weekly income (6 working days):**
```
Weekly_Income = 6 × ₹1,120 = ₹6,720/week
```

**Monthly income:**
```
Monthly_Income ≈ ₹6,720 × 4.3 = ₹28,896/month
```

Aligns with public data on Zepto/Blinkit rider earnings in metro cities (₹26,000–₹32,000/month).

---

## 4. Income Loss Model (Surge-Corrected)

**Critical correction from naive models:**
During partial disruptions, Zepto and Blinkit activate surge pricing — per-order pay increases to keep riders working in difficult conditions. A model that ignores surge will systematically overestimate income loss, leading to overpriced premiums and incorrect loss ratios.

GigShield uses a two-case loss model.

---

### Case A: Platform Pause (Primary Event)

Zepto/Blinkit officially halts order dispatch from a dark store zone.

```
Orders dispatched = 0
Surge pay        = N/A
Hourly Loss      = ₹140/hr (100% loss — no calculation needed)
```

Clean, binary, unambiguous. No estimation required.
Estimated frequency: ~60% of total disruption hours in major weather events.

---

### Case B: Partial Disruption (Platform Active, Reduced Volume)

Platform continues dispatching at reduced volume. Surge pay activates.

```
Disrupted_orders/hr         = 1.5   (50% of normal)
Surge_pay/order             = ₹70   (Zepto surge: ~27% above base ₹55)
Disrupted_Hourly_Income     = 1.5 × ₹70 × 0.85 = ₹89.25/hr

True_Hourly_Loss = ₹140 − ₹89.25 = ₹50.75/hr
```

Estimated frequency: ~40% of total disruption hours.

---

### Weighted True Hourly Loss

```
Weighted_Hourly_Loss = (0.60 × ₹140) + (0.40 × ₹50.75)
                     = ₹84.00 + ₹20.30
                     = ₹104.30/hr
```

---

### Payout Per Disruption Event (after 30-min deductible)

| Disruption Type | Avg Raw Duration | Payable Hours | Weighted Loss | 80% Payout |
|---|---|---|---|---|
| Heavy rain | 2.5 hrs | 2.0 hrs | ₹208.60 | ₹166.88 |
| Extreme heat | 3.0 hrs | 2.5 hrs | ₹260.75 | ₹208.60 |
| Flash flood | 4.0 hrs | 3.5 hrs | ₹365.05 | ₹292.04 |
| Severe AQI | 5.0 hrs | 4.5 hrs | ₹469.35 | ₹375.48 |
| Curfew | variable | variable | ₹140/hr (full pause) | ₹112/hr |

---

### Rain Demand Increase — Explicit Rule

Rain sometimes increases Zepto orders. GigShield handles this explicitly:

```
IF Platform_Pause_Signal == False
   AND Current_Income_Rate > Baseline_Income_Rate:
   → Income is above baseline
   → No trigger, no payout
   → Policy remains active for rest of week
```

This is stated in policy terms at onboarding. It is also why the dual condition
(external signal + income drop) is essential — rain alone cannot fire a claim.

---

## 5. Dark Store as the Risk Unit

**The most important architectural decision in GigShield.**

All 10–20 riders attached to one dark store share:
- The same operating zone (same weather, same road conditions, same flooding risk)
- The same platform pause events (when Zepto pauses a store, every rider at that store is affected simultaneously)
- The same historical disruption frequency (a store in a low-lying Bengaluru zone has measurably higher flood risk than one in Whitefield)

GigShield prices risk at the **dark store level first**, then adjusts for individual riders on top.

---

### Dark Store Risk Score (DSRS)

Each dark store is assigned a DSRS (0–100) at onboarding:

| Factor | Weight | Data Source |
|---|---|---|
| Zone flood / waterlogging history (5-year) | 30% | IMD historical data (mocked) |
| Proximity to water bodies / low elevation | 20% | OpenStreetMap |
| Historical AQI exceedance days | 15% | WAQI historical API |
| Road drainage quality (pincode-level proxy) | 15% | Flood complaint density (mocked) |
| Historical platform pause frequency at store | 20% | Simulated disruption log |

```
DSRS = Σ (factor_score × weight)   →   range 0–100
```

| DSRS Band | Risk Level | Premium Multiplier |
|---|---|---|
| 0–30 | Low | 0.85× |
| 31–55 | Medium | 1.00× |
| 56–75 | High | 1.18× |
| 76–100 | Very High | 1.35× |

---

### Final Premium Formula

```
Final_Premium = Tier_Base × DSRS_Multiplier × Rider_Multiplier
```

Individual rider factors applied on top of DSRS:

| Rider Factor | Adjustment |
|---|---|
| Shift during peak disruption window (12–4 PM) | +5% |
| 0 claims in last 8 weeks (loyalty) | −10% |
| New worker, < 4 weeks tenure | +5% |
| Monsoon season active (Jun–Sep) | +10% |
| Delhi AQI season (Oct–Jan) | +12% |

DSRS also anchors fraud detection: a claim at a store with DSRS = 12 (near-zero historical disruption) and no platform pause signal is an automatic anomaly before any ML layer runs.

---

## 6. Parametric Trigger System

**Core rule:**

```
TRIGGER fires if:
    store_pause_signal == True
    OR (External_Threshold_Breached AND Surge_Corrected_Income_Drop >= 25%)
```

Platform pause alone is sufficient — it is binary, platform-verified, and tamper-proof.
The dual condition applies only when the platform has NOT issued a pause.

---

### Trigger 1: Platform Pause Signal (Primary)

| Parameter | Value |
|---|---|
| Source | Zepto/Blinkit store pause API (mocked) |
| Signal | `store_id → paused: true` with timestamp |
| Income Drop Condition | Not required — pause = 100% halt |
| Coverage | All riders registered to that store_id |
| Loss Case | Case A — full ₹140/hr |
| Fraud Resistance | Highest — platform-issued, not worker-reported |

---

### Trigger 2: Heavy Rainfall

| Parameter | Value |
|---|---|
| External Condition | Rainfall > 25mm/hr sustained ≥ 30 min |
| Data Source | OpenWeatherMap API (free tier, live) |
| Income Drop Condition | Surge-corrected drop ≥ 25% vs baseline |
| Zone Scope | Worker's registered 3km zone polygon |
| Loss Case | Case B — ₹50.75/hr true loss |

---

### Trigger 3: Extreme Heat

| Parameter | Value |
|---|---|
| External Condition | Temperature > 43°C for ≥ 2 consecutive hours |
| Data Source | OpenWeatherMap API |
| Income Drop Condition | Surge-corrected drop ≥ 25% |
| Zone Scope | City-level |
| Loss Case | Case B |

---

### Trigger 4: Severe AQI / Pollution

| Parameter | Value |
|---|---|
| External Condition | AQI > 400 (Severe, CPCB scale) for ≥ 3 hours |
| Data Source | WAQI API (free tier, live) |
| Income Drop Condition | Surge-corrected drop ≥ 25% |
| Zone Scope | City-level |
| Loss Case | Mixed — depends on whether store also pauses |

---

### Trigger 5: Flash Flood / Waterlogging

| Parameter | Value |
|---|---|
| External Condition | IMD flood alert Level 2+ in zone |
| Data Source | IMD API (mocked) |
| Income Drop Condition | Not required if pause fires; otherwise ≥ 50% drop |
| Zone Scope | Pincode/ward polygon from alert |
| Loss Case | Typically Case A |

---

### Trigger 6: Curfew / Section 144

| Parameter | Value |
|---|---|
| External Condition | Official curfew notification in operating zone |
| Data Source | Govt alert feed (mocked) |
| Income Drop Condition | Not required — curfew = 100% halt |
| Zone Scope | Exact affected zone from notification |
| Loss Case | Case A — full ₹140/hr |

---

## 7. Income Drop Detection Mechanism

Used for Triggers 2–4 when no platform pause signal has been issued.
The system independently verifies that income is genuinely reduced before a claim fires.

---

### Step 1: Baseline Definition

```
Baseline(worker, zone, hour_slot, day_type) = expected_orders_per_hour
```

Baseline is dynamic — accounts for:
- Time-of-day demand curve (peak: 8–10 AM, 12–2 PM, 6–9 PM)
- Day type (weekends: +15% demand)
- Zone density (residential vs commercial catchment)

Default for first 4 weeks (no personal data):
```
Baseline = 3.0 orders/hr
```

After 4 weeks: updated to worker's personal 4-week rolling average.

**Baseline spike correction:**
People pre-order before a cyclone warning, temporarily inflating the baseline.
If left uncorrected, the disruption drop appears smaller and the trigger may not fire.

```
IF IMD or weather API issues a forecast alert within 48 hours:
    → Baseline LOCKS at the 14-day rolling average
    → Spike window (48 hrs before event) is excluded from baseline calculation
```

---

### Step 2: Surge-Corrected Income Rate (Computed Every 15 Minutes)

```python
import numpy as np

def compute_income_rate(mode, surge_active=False):
    order_rates = {
        "normal":      3.0,
        "light_rain":  2.5,
        "heavy_rain":  1.5,
        "heat":        1.5,
        "aqi_severe":  1.0,
        "pause":       0.0,
    }
    surge_pay  = 70 if surge_active else 55
    efficiency = 0.85
    lam        = order_rates[mode] / 4          # per 15-min window
    orders     = np.random.poisson(lam)
    current_hr = orders * 4                      # extrapolate to /hr
    current_income   = current_hr * surge_pay * efficiency
    baseline_income  = 3.0 * 55 * efficiency     # ₹140/hr fixed
    return current_income, baseline_income
```

---

### Step 3: Drop Detection

```
Income_Drop_Percent = (Baseline_Income − Current_Income) / Baseline_Income × 100

IF External_Signal_Active AND Income_Drop_Percent >= 25%:
    → CLAIM INITIATED
```

**Example — heavy rain, surge active:**
```
Baseline_Income   = ₹140/hr
Current_Income    = 1.5 × ₹70 × 0.85 = ₹89.25/hr
Drop_Percent      = (140 − 89.25) / 140 × 100 = 36.25%

36.25% >= 25% AND rain signal active → TRIGGER FIRES
```

**Example — light rain, surge active (demand holds):**
```
Baseline_Income   = ₹140/hr
Current_Income    = 2.5 × ₹70 × 0.85 = ₹148.75/hr
Drop_Percent      = −6.25%

Negative drop → NO TRIGGER
```

**Why 25% threshold?**
Below 25%, income variance is within normal hourly fluctuation (traffic, demand lull, personal break).
At 25%+ with surge already applied, the worker is genuinely losing meaningful income.
This threshold is also robust to Poisson noise — a single unlucky 15-min window will not fire a claim.

---

### Step 4: Disruption Duration Tracking

```
Disruption_start = first timestamp where conditions are met
Disruption_end   = timestamp where Drop_Percent < 15% for 30+ min sustained
Disruption_hours = (end − start) in decimal hours
Payable_hours    = Disruption_hours − 0.5  (30-min deductible)
```

---

## 8. Simulation Strategy

Real Zepto/Blinkit order APIs are not publicly accessible.
All order flow in non-pause scenarios is simulated.

### Simulation Components

| Component | Method | Calibration |
|---|---|---|
| Platform pause signal | Admin toggle + auto-trigger from weather APIs | Mocked API |
| Order arrival rate | Poisson process (λ varies by condition) | Q-Commerce operational data: 2.5–3.5 orders/hr normal |
| Surge pay | Boolean flag, fires when external threshold active | Industry norm: surge at ~27% above base |
| Weather events | OpenWeatherMap (live) + admin override | Live API |
| Flood / curfew | Admin-triggered zone flag | Mocked |

### Lambda Table (per 15-min window)

```
Normal:          λ = 0.75  →  3.0 orders/hr   baseline
Light rain:      λ = 0.625 →  2.5 orders/hr   surge on → income above baseline → no trigger
Heavy rain:      λ = 0.375 →  1.5 orders/hr   surge on → 36% drop → TRIGGER
Extreme heat:    λ = 0.375 →  1.5 orders/hr   surge on → 36% drop → TRIGGER
Severe AQI:      λ = 0.25  →  1.0 orders/hr   surge on → 51% drop → TRIGGER
Platform pause:  λ = 0.0   →  0.0 orders/hr   TRIGGER via pause signal (no simulation)
```

**Why Poisson is the right model:**
Poisson distribution is the standard model for arrival processes (calls, deliveries, transactions).
It correctly captures stochastic variance — not every 15-min window is identical under identical conditions.
This means the simulation produces realistic noise without triggering false claims, because the
dual condition (external signal + income drop) requires both to be simultaneously true.

### Demo Disruption Simulator (Admin Panel)

For judging purposes, the admin panel includes a Disruption Simulator that:
1. Selects a zone polygon on map
2. Chooses disruption type (rain / heat / AQI / flood / curfew / store pause)
3. Sets duration
4. Injects event into the live system

This triggers the complete pipeline end-to-end — signal → fraud check → claim → payout —
demonstrable in real time during the 5-min demo video.

---

## 9. Pricing Model

### Step 1: Expected Weekly Loss

| City | Events/Year | Avg Payable Hours | Weighted Loss/Event | Expected Annual Loss |
|---|---|---|---|---|
| Bengaluru | 28 | 2.0 hrs | ₹208.60 | ₹5,841 |
| Chennai | 35 | 2.5 hrs | ₹260.75 | ₹9,126 |
| Delhi | 20 | 3.5 hrs | ₹365.05 | ₹7,301 |
| Mumbai | 40 | 3.0 hrs | ₹312.90 | ₹12,516 |

**Weekly disruption probability (Bengaluru):**
```
P(disruption this week) = 28 / 52 = 0.538
```

**Expected weekly loss — Max tier, 80% coverage:**
```
Expected_Weekly_Loss = 0.538 × ₹208.60 × 0.80 = ₹89.78
```

---

### Step 2: Premium with Margins

```
Premium = Expected_Weekly_Loss × (1 + Risk_Margin) × (1 + Fraud_Buffer) × (1 + Op_Cost)
        = ₹89.78 × 1.15 × 1.08 × 1.12
        = ₹124.60  →  Max tier set at ₹119/week
```

| Margin | Rate | Justification |
|---|---|---|
| Risk Margin | 15% | Buffer for above-average disruption years |
| Fraud Buffer | 8% | Estimated residual fraud rate after 7-layer detection |
| Operational Cost | 12% | API costs, infra, Celery queue, payment processing |

---

### Step 3: Pricing Tiers

| Tier | Coverage | Weekly Premium | Max Weekly Payout |
|---|---|---|---|
| Basic | 50% income replacement | ₹59 | ₹700 |
| Standard | 65% income replacement | ₹89 | ₹910 |
| Max | 80% income replacement | ₹119 | ₹1,120 |

---

### Step 4: Example Final Premium

Ravi, High-DSRS Bengaluru store (DSRS = 72), Standard tier, July:
```
₹89 × 1.18 (DSRS High) × 1.10 (monsoon) = ₹115.61/week
```

---

### Step 5: Business Viability

**Why will a rider pay?**
```
Expected annual disruption loss (Bengaluru, no insurance) = ₹4,673/year
Annual Max tier cost = ₹119 × 52 = ₹6,188/year
```

The rider pays slightly above their statistical expected loss — but the value is
**same-day liquidity**. A ₹209 loss on Tuesday becomes a ₹167 credit within minutes.
For a zero-savings worker on a weekly payout cycle, timing matters more than annual return.

**Why does the insurer survive at 100 riders?**
```
Weekly premium collected = 100 × ₹119           = ₹11,900
Expected claims/week     = 53.8 riders × ₹166.88 = ₹8,978
Weekly operating margin  = ₹11,900 − ₹8,978      = ₹2,922  (24.6%)
```

---

## 10. Adverse Selection Controls

**The problem:** If riders can subscribe and cancel weekly, they will subscribe in June
(monsoon starts) and cancel in December (dry season). This concentrates claims in
high-disruption periods and destroys the annual loss ratio.

---

### Mechanism 1: Minimum 4-Week Commitment

Riders commit to a 4-week minimum on first activation.
First 4 weekly premiums collected upfront at onboarding.
No mid-subscription cancellation before week 4.
After week 4: cancellation effective at end of current week. No refund for active week.

---

### Mechanism 2: Seasonal Pre-Commitment Discount

Year-round coverage distributes risk across dry and wet seasons — which is what the insurer needs.

```
12-week pre-commitment → 15% discount on all weekly premiums
26-week pre-commitment → 22% discount
```

---

### Mechanism 3: Post-Cancellation Exclusion Window

```
IF worker cancelled coverage within 7 days of a disruption event at their store
   AND attempts to re-enrol within 7 days of that cancellation:
   → Re-enrolment allowed
   → 14-day exclusion period applies from re-enrolment date
   → No claims payable during those 14 days
```

This prevents: worker sees cyclone warning → cancels premium → re-enrols 2 days later to claim.

---

## 11. Fraud Detection System (7 Layers)

**Design principle:** A genuine disruption affects all riders at a dark store simultaneously.
Isolated anomalies are fraud signals. Every layer exploits this collective disruption property.

---

### Layer 1: Parametric-Only Claim Initiation

Workers cannot file claims. All claims are system-generated.
A claim fires only if the platform pause signal is active, or external API + income drop both confirm.
This eliminates fabricated claim filing at the architecture level.

---

### Layer 2: Time-Based Deductible

```
No payout for first 30 minutes of any disruption event.
Payout begins from minute 31 onwards.
```

Filters out normal operational variance, Poisson noise, and micro-events not worth insuring.

---

### Layer 3: Dark Store Peer Correlation

```
IF store_pause_signal == False
   AND < 30% of riders at the same dark store show equivalent drop:
   → REJECT (worker is an outlier at their own store)

IF >= 30% of riders at the same dark store show equivalent drop:
   → PROCEED to Layer 4
```

If a disruption is real, it affects everyone at the store. An isolated claim is definitionally suspicious.

---

### Layer 4: Zone Peer Correlation

```
IF >= 30% of all active workers in the same zone show Income_Drop >= 25%:
   → Zone-wide disruption confirmed → PROCEED

IF only one store is affected but neighbouring zone workers show normal rates:
   → Elevated scrutiny — store-specific anomaly
```

---

### Layer 5: GPS Zone Validation

- Worker GPS sampled every 10 min during active shift
- Must be within registered store zone polygon (±500m) during claimed window
- Outside registered zone → automatic rejection
- GPS unavailable > 30 min during claimed window → flagged for manual review
- GPS in different city → automatic rejection + account flag

---

### Layer 6: ML Anomaly Detection — Isolation Forest

```
Model:   Isolation Forest (unsupervised, scikit-learn)

Features:
  - claim_frequency_last_4_weeks
  - zone_match_score              (GPS vs registered zone, 0–1)
  - store_peer_correlation        (fraction of store peers showing drop)
  - zone_peer_correlation         (fraction of zone peers showing drop)
  - disruption_overlap_score      (claim window vs trigger window alignment, 0–1)
  - surge_reconciliation_delta    (claimed loss vs surge-adjusted true loss)
  - dsrs_claim_ratio              (claim frequency vs store DSRS expectation)

Output:  Anomaly score (0–1)
         score > 0.15 → flagged for insurer review (24hr SLA)
         score > 0.40 → auto-rejected
```

---

### Layer 7: Temporal Pattern Detection

```
Flag if ANY of the following:
  - Claims in >= 3 consecutive weeks on the same day, inconsistent with DSRS
  - Claim onset times cluster at minute 31–35 (gaming the 30-min deductible boundary)
  - Claim onset times correlate with weekly payout cycle, not randomly distributed
  - Same rider registered to two geographically distinct zone polygons in one week
```

Temporal flags are scored and fed into Layer 6 as additional features — not standalone rejections.

---

### Layer 7b: Surge Pay Reconciliation

Post-event, if platform data confirms the worker was receiving surge-priced orders during
the claimed disruption window:

```
Payout = recalculated using actual surge-adjusted loss (not estimated)
```

Prevents over-insurance where a rider earns ₹120/hr during a "disruption" but claims at ₹0.

---

### Duplicate Prevention

```
event_id = zone_id + store_id + timestamp_bucket + disruption_type
Constraint: one payout per (worker_id, event_id)
Duplicate attempt → rejected at DB level before any business logic runs
```

---

## 12. System Workflow (End-to-End)

```
[1] ONBOARDING
    Worker registers via Flutter mobile app
    Phone + Aadhaar-lite KYC
    Links platform (Zepto/Blinkit), selects assigned dark store
    Zone polygon auto-set from store_id → zone mapping
    DSRS fetched for that store
    Income baseline set (default 3.0 orders/hr, personal after 4 weeks)
    4-week minimum commitment accepted

[2] WEEKLY PREMIUM CALCULATION
    AI pricing model runs: zone, season, DSRS, shift_timing,
                           claim_history_4w, tenure_weeks
    Dynamic premium computed
    Worker selects tier (Basic / Standard / Max)
    Weekly premium deducted via UPI autopay
    Policy active for 7-day rolling window

[3] ACTIVE MONITORING (every 15 min, per active worker)
    Platform pause API polled per store_id
    OpenWeatherMap + WAQI APIs polled per zone
    Simulated order stream runs (Poisson, mode = current conditions)
    Surge flag set if external threshold is active
    Current_Income computed (surge-corrected)
    Income_Drop_Percent computed vs locked baseline

[4] TRIGGER EVALUATION
    IF store_pause_signal == True:
        Event created → Case A, full ₹140/hr loss
    ELIF External_Threshold_Active AND Income_Drop >= 25%:
        Event created → Case B, surge-corrected loss
    ELSE:
        No action

    event_id generated (zone + store + timestamp + type)
    All active workers at affected store/zone enrolled into event

[5] FRAUD CHECK (automated, target < 2 seconds)
    L1: Claim system-generated?              always yes by architecture
    L2: Payable_hours > 0?                   30-min deductible check
    L3: >= 30% store peers show drop?        dark store correlation
    L4: >= 30% zone peers show drop?         zone correlation
    L5: Worker GPS within zone polygon?      GPS validation
    L6: Anomaly score < 0.15?               Isolation Forest
    L7: No temporal pattern flags?           temporal detection

    All pass        → APPROVED
    L6 score 0.15–0.40 → FLAGGED (insurer review within 24hr)
    L6 > 0.40 OR L3/L4/L5 fail → REJECTED

[6] CLAIM PROCESSING
    Payable_hours = Disruption_hours − 0.5
    Payout = Weighted_Hourly_Loss × Payable_hours × Coverage_Factor × Tier_Factor
    Surge reconciliation applied

[7] INSTANT PAYOUT
    Razorpay Test Mode / UPI Simulator executes transfer
    Push notification: "₹167 credited — rain disruption 14:30–17:00"
    Claim logged to worker earnings history

[8] INSURER DASHBOARD UPDATE
    Claim recorded, loss ratio refreshed
    DSRS recalibration flagged if store shows > 2× expected claim rate
    Fraud flags surfaced for review
    Weekly claim forecast updated
```

**No manual steps. Zero-touch for the worker.**

---

## 13. AI/ML Integration

Four models. Each has a defined input, model type, and output.

---

### Model 1: Dark Store Risk Score (DSRS)

```
Input:   [zone_flood_history, elevation_proxy, proximity_water_body,
          road_drainage_score, historical_pause_frequency]
Model:   Weighted scoring + Random Forest classifier (for DSRS band)
Output:  DSRS (0–100), band, premium multiplier
Data:    Simulated store-zone dataset calibrated to IMD / OSM public data
```

### Model 2: Dynamic Premium Pricing

```
Input:   [city, dsrs_score, season_code, shift_time_band,
          claim_history_4w, worker_tenure_weeks, day_type]
Model:   Random Forest Regressor (scikit-learn)
Output:  Risk multiplier (float, 0.85–1.35)
Data:    Synthetic 15,000 rider-store-week records,
         calibrated to IMD / WAQI historical disruption data
```

### Model 3: Fraud Anomaly Detection

```
Input:   [claim_frequency, zone_match_score, store_peer_correlation,
          zone_peer_correlation, disruption_overlap_score,
          surge_reconciliation_delta, dsrs_claim_ratio]
Model:   Isolation Forest (unsupervised, scikit-learn)
Output:  Anomaly score (0–1); 0.15 = flag, 0.40 = reject
Data:    Simulated normal + anomalous claim records (10,000 rows)
```

### Model 4: Weekly Disruption Forecast

```
Input:   [7-day weather forecast, historical disruption calendar,
          active store count, DSRS distribution of active stores]
Model:   Weighted probability (rule-based)
Output:  Predicted claim count + expected payout for next 7 days
Use:     Insurer reserve management, shown on admin dashboard
```

---

## 14. Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Mobile App | Flutter + Dart + Riverpod | Worker: onboarding, policy, disruption alerts, payout history |
| Web Dashboard | React + Vite + Tailwind + Recharts | Insurer: analytics, fraud queue, DSRS map, forecast |
| Backend API | Python + FastAPI | All business logic, trigger monitoring, ML inference |
| Database | PostgreSQL (SQLAlchemy) | Workers, stores, policies, events, claims |
| Task Queue | Celery + Redis | 15-min monitoring loop per active worker |
| ML | scikit-learn + pandas + numpy | DSRS scoring, pricing, fraud detection |
| Weather | OpenWeatherMap free tier | Rain + heat triggers (live) |
| AQI | WAQI free tier | Pollution trigger (live) |
| Flood / Curfew | IMD / Govt feed | Mocked |
| Platform API | Zepto/Blinkit pause signal | Mocked |
| Payments | Razorpay Test Mode | Simulated UPI payouts |
| Auth | Firebase Auth (OTP) | Phone-number login |

---

## 15. Platform Justification

| Platform | Users | Justification |
|---|---|---|
| Flutter Mobile App | Workers | Riders are mobile-first. App provides real-time disruption alerts, policy status, payout history, and instant credit notifications. Single codebase for Android + iOS. |
| React Web Dashboard | Insurer / Admin | Policy analytics, fraud flag queues, DSRS risk maps, and predictive loss dashboards are best consumed on desktop. React allows fast iteration on data-heavy views. |

A single Python/FastAPI backend serves both platforms via REST API.

---

## 16. Differentiation

**Three decisions no other team will make:**

**1. Dark Store as the risk unit.**
Every other team will price per-rider. GigShield prices per-store first (DSRS), then per-rider.
This is how actual underwriting works — shared exposure priced together.
It also creates the strongest fraud signal: if a disruption is real, all riders at the store are affected.

**2. Platform Pause Signal as the primary trigger.**
Most teams will simulate order drops as their only trigger.
GigShield uses a binary store-pause API (mocked) as the primary — cleaner, unambiguous,
platform-verified, and architecturally honest to how Q-Commerce actually operates.

**3. Surge-corrected income loss model.**
Every other team will calculate loss as `baseline − disrupted` without accounting for surge pay.
This overstates loss by 30–40% during partial disruptions.
GigShield separates full pause (100% loss) from partial disruption (surge-corrected true loss)
and prices accordingly. This is the difference between a model an actuary would accept and one they wouldn't.

---

## 17. Future Scope

- Real Zepto/Blinkit API integration (replace mocked pause signal with live data)
- Expansion to food delivery (Zomato/Swiggy) and e-commerce (Amazon/Flipkart) — each needs its own income and trigger model
- WhatsApp bot interface for workers without smartphones
- IRDAI regulatory compliance layer for production deployment
- Dynamic DSRS recalibration as real claim data accumulates per store

---


*Built for Guidewire DEVTrails 2026 — Seed. Scale. Soar.*
