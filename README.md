# EarnSure Suraksha Policy
### Dynamic Income Protection for Q-Commerce Delivery Partners
**By Team 3.JS**

---

## Table of Contents

1. [Overview](#overview)
2. [Problem Statement](#problem-statement)
3. [Policy Features](#policy-features)
4. [Mathematical Models](#mathematical-models)
   - [Premium Model](#1-premium-model)
   - [Expected Income Regression Model](#2-expected-income-regression-model)
   - [Income Impact Regression Model](#3-income-impact-regression-model)
   - [Claim Validity Condition](#4-claim-validity-condition)
   - [Disruption Trigger & Severity](#5-disruption-trigger--severity)
   - [Coverage Multiplier (θ)](#6-coverage-multiplier-θ)
   - [Claim Amounts & Month Factor](#7-claim-amounts--month-factor)
   - [No-Claim Bonus](#8-no-claim-bonus)
   - [Final Claim Formula](#9-final-claim-formula)
5. [Application Architecture](#application-architecture)
6. [AI Architecture](#ai-architecture)
7. [Implementation Plan](#implementation-plan)
8. [Tech Stack](#tech-stack)
9. [Adversarial Defense & Anti-Spoofing Strategy](#adversarial-defense--anti-spoofing-strategy)

---

## Overview

**EarnSure Suraksha Policy** is a micro-insurance product purpose-built for gig economy delivery partners operating on Q-commerce platforms (Blinkit, Zepto, Swiggy Instamart, etc.). Unlike traditional insurance, EarnSure adapts its premiums and payouts dynamically each week based on actual income, disruption events, and area demand — ensuring that coverage is always proportional, affordable, and fair.

The policy bridges a critical gap: delivery partners face income volatility from weather, traffic disruptions, platform outages, and area-level demand shifts. Existing insurance products are designed for salaried workers with predictable income. EarnSure solves this by tying both premiums and claims directly to real earned income data.

---

## Problem Statement

Q-commerce delivery partners face:

- **Unpredictable income** due to external disruptions (weather, floods, traffic incidents, local shutdowns)
- **No income safety net** during involuntary low-earning weeks
- **Unaffordable traditional insurance** with flat premiums that ignore income volatility
- **High claim friction** due to documentation requirements and slow processing

EarnSure addresses all four issues with adaptive premiums, ML-driven disruption detection, automated claim validation, and instant payouts.

---

## Policy Features

| Feature | Description |
|---|---|
| **Weekly Adaptive Premium** | Premium is calculated weekly based on actual income and number of prior claims |
| **Income-Linked Coverage** | Claim amounts scale directly with the worker's predicted weekly income |
| **Disruption-Triggered Activation** | Claims are only triggered when a verified disruption event hits the worker's area |
| **No-Claim Bonus** | Workers who complete a 6-month policy period without claims receive a bonus of 25–50% of total premiums paid |
| **Month Factor Scaling** | Early-period claims (months 1–2) receive slightly reduced payouts to disincentivize gaming at onboarding |
| **Claim Waiver Post-Claim** | After a valid claim, the next week's premium is fully waived |
| **Payment Deadline** | Premiums are due every Sunday and are mandatory for coverage continuity |

---

## Mathematical Models

### 1. Premium Model

The weekly premium is a percentage of the worker's income for that week:

```
P_w = α_n × I_w
```

Where the premium rate `α_n` escalates with repeated claims:

```
α_n = 0.03 + (0.005 × N_c)
```

| Parameter | Meaning |
|---|---|
| `P_w` | Weekly premium (₹) |
| `α_n` | Premium rate (starts at 3%, increases by 0.5% per prior claim) |
| `I_w` | Actual income earned in week `w` |
| `N_c` | Number of claims made so far in the policy period |

**Key rules:**
- Base premium: **3%** of weekly income
- Increases by **+0.5%** per claim filed
- Payment due: **Every Sunday** (mandatory)
- If a claim occurs: **Next week's premium is waived**

---

### 2. Expected Income Regression Model

The model predicts what a worker *should* have earned in a given week, forming the baseline for claim calculations:

```
I_exp = f(past_income_2w, active_days, area_demand_factor)
```

**Inputs:**
- Past 2-week average income
- Active days per week
- Area demand factor (derived from Q-commerce platform order volumes)

**Output:**
- `I_exp` — Predicted Weekly Income (₹)

---

### 3. Income Impact Regression Model

Determines whether a worker actually suffered meaningful income loss relative to what was expected:

```
L_r = (I_exp - I_act) / I_exp
```

**Output:** Binary flag — `1` if `L_r ≥ 0.30`, else `0`

A loss ratio of **≥30%** is required for a claim to proceed. This prevents minor fluctuations from triggering payouts and filters opportunistic claims.

---

### 4. Claim Validity Condition

A claim is valid **only when all three conditions are simultaneously true**:

```
Claim Valid = (T = 1) ∧ (A = 1) ∧ (M = 1)
```

| Flag | Meaning |
|---|---|
| `T = 1` | A verified disruption event exists in the worker's area |
| `A = 1` | The worker was active (logged in, accepting orders) in the affected area |
| `M = 1` | The Income Impact Model outputs 1 (loss ratio ≥ 30%) |

All three must be true simultaneously. This is the core gating logic — any single flag failing blocks the claim.

---

### 5. Disruption Trigger & Severity

`T = 1` is set when a disruption event is detected in the worker's operational area.

Disruption severity `D_s` is categorized as:

| Disruption Level | D_s Score |
|---|---|
| Partial | 0.5 |
| Moderate | 0.75 |
| Full | 1.0 |

Severity is sourced from a combination of Traffic API data, Weather API alerts, and platform order volume drops.

---

### 6. Coverage Multiplier (θ)

The coverage multiplier boosts the claim amount based on how severe the disruption was, while penalizing frequent claimers and rewarding income stability:

```
θ = max(1.0, 1 + 0.15×D_s − 0.02×N_c + S_i)
```

| Variable | Meaning |
|---|---|
| `D_s` | Disruption severity (0.05, 0.10, or 0.15 at policy level — mapped from severity table) |
| `N_c` | Number of prior claims → deduction of 0.02 per claim |
| `S_i` | Stability bonus: +0.05 if income is stable, else 0 |

**Constraint:** `θ ≥ 1.0` (floor enforced by `max`)

---

### 7. Claim Amounts & Month Factor

**Base Claim:**
```
C_w = θ × I_exp
```

**Time-Based Adjustment:**
```
C_final = C_w × γ_m
```

Month Factor `γ_m` scales with policy maturity — rewarding long-term policyholders:

| Policy Period | γ_m |
|---|---|
| Month 1–2 | 0.80 |
| Month 3–4 | 0.85 |
| Month 5–6 | 0.90 |

---

### 8. No-Claim Bonus

At the end of a 6-month policy period, workers who filed no claims receive a refund bonus:

```
B = η × ΣP_w
```

| Payment Behaviour | Bonus Rate (η) |
|---|---|
| Paid fully on time | 50% |
| Partial / delayed | 25% |

---

### 9. Final Claim Formula

The consolidated payout formula:

```
C_final = max(1.0, 1 + 0.15×D_s − 0.02×N_c + S_i) × I_exp × γ_m
```

| Symbol | Meaning |
|---|---|
| `C_final` | Final weekly payout (₹) |
| `D_s` | Disruption severity (0.05/0.10/0.15) |
| `N_c` | Number of claims made |
| `S_i` | +0.05 if income stable, else 0 |
| `I_exp` | Predicted weekly income |
| `γ_m` | Month adjustment factor |

---

## Application Architecture

![Application Architecture Diagram](docs/app%20architecture.png)

The platform is a React-based SPA for delivery partners, backed by a Flask REST API server, with MongoDB for persistence. External data sources (Weather, Maps, Traffic, Platform) feed into the AI layer for disruption detection and income modeling.

---

## AI Architecture

![AI Architecture Diagram](docs/ai%20architecture.png)

---

## Implementation Plan

**Phase 1 (Current — MVP):**
- Policy engine with all 9 mathematical models
- React frontend for delivery partner onboarding and claims tracking
- MongoDB data store for profiles, claims, and premium records
- Integration with Weather API, Traffic API, and Google Maps for disruption detection
- AI/Fraud Detection Service (rule-based)
- CI/CD pipeline via GitHub Actions

**Phase 2 (Planned):**
- ML-powered income prediction model with live retraining
- Full multimodal disruption detection (image + location + telemetry)
- Multilingual voice navigation for low-literacy users
- Real-time analytics dashboard for platform operators
- Adversarial defense layer (GPS spoofing, claim ring detection — see below)

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React (SPA) |
| Backend | Python + Flask, REST API |
| Database | MongoDB |
| AI/ML | Python (scikit-learn / regression models) |
| DevOps | GitHub Actions, Cloud Deployment |
| External APIs | Weather API, Google Maps API, Traffic API, Q-Commerce Platform API, Payment Gateway |

---

## Adversarial Defense & Anti-Spoofing Strategy

**Differentiation:** Since claims are auto-triggered by external APIs, spoofing is structurally blocked at the source — a claim only fires when Weather API, Traffic API, and Google Maps jointly confirm a disruption in the worker's registered delivery zone, meaning a bad actor cannot trigger a claim at all without a real, verifiable, hyperlocal disruption event occurring first.

**Data Beyond GPS:** The system validates every auto-trigger against a multi-source confirmation threshold — Weather API rainfall intensity, Traffic API incident density, Google Maps road closure overlays, Q-commerce platform order volume drop, and news crawler signals must collectively agree before a claim is released, making coordinated fraud impossible without spoofing multiple independent external systems simultaneously.

**UX Balance:** Since claims are never filed manually, honest workers are never flagged or penalized — the only exception is a soft-hold triggered when a worker's registered zone partially overlaps a disruption boundary, which auto-resolves within 2 hours as APIs update their coverage polygons, ensuring no human intervention is ever needed.

---

*EarnSure Suraksha Policy — Building financial resilience for the gig economy, one week at a time.*
*Team 3.JS*