# 🪙 Bitcoin AML Transaction Monitoring with Graph Analytics & Explainable Risk Scoring

## Overview
This project implements an **explainable, graph-based Anti–Money Laundering (AML) transaction monitoring system for Bitcoin** using the public Elliptic dataset.

Instead of training a black-box classifier, the system follows **real-world AML design principles**:
- risk scoring instead of prediction
- graph-based behavioral signals
- bounded risk propagation
- fully explainable alert reasons

The output closely mirrors what a **production transaction monitoring (TM) system** would generate for compliance analysts.

---

## Key Design Principles
- **Risk scoring, not classification**  
  Transactions are prioritized for investigation rather than labeled as illicit.

- **Explainability first**  
  Every alert includes human-readable reasons tied to AML typologies.

- **Bounded risk propagation**  
  Illicit exposure is limited to 1-hop and strict 2-hop neighborhoods to control false positives.

- **Vendor-agnostic logic**  
  Core signals are derived from transaction graph structure, not proprietary features.

---

## Dataset
**Elliptic Bitcoin Dataset (Public Research Release)**

### Raw files used
- `elliptic_txs_features.csv`
- `elliptic_txs_edgelist.csv`
- `elliptic_txs_classes.csv`

### Raw columns actually used
| File | Columns |
|----|----|
| Features | `txId`, `time_step` |
| Edgelist | `txId1`, `txId2` |
| Classes | `txId`, `class` |

> Elliptic engineered features (`f_*`) were explored during EDA but intentionally **not used for risk scoring** to keep logic domain-driven and explainable.

---

## Concept: `time_step`
`time_step` represents a **discrete AML monitoring window**, not a blockchain timestamp.

It is used to:
- order transactions temporally
- analyze transaction volume and illicit clustering over time

This mirrors how banks monitor activity in rolling time windows.

---

## Feature Engineering

### 1️⃣ Graph Structure Features
Computed from the transaction edgelist:

- **`fan_in_1hop`**  
  Number of upstream transactions feeding into a transaction  
  → aggregation / mixing signal

- **`fan_out_1hop`**  
  Number of downstream transactions funded by a transaction  
  → distribution / layering / batching signal

These features exhibit **heavy-tailed distributions**, consistent with Bitcoin transaction graphs.

---

### 2️⃣ Illicit Exposure Features
Using an **undirected transaction graph** for proximity analysis:

#### 1-hop exposure
- `illicit_nbr_ratio_1hop`  
  Fraction of immediate neighbors labeled illicit

#### Strict 2-hop exposure
- `illicit_nbr_ratio_2hop_strict`  
  Fraction of transactions *exactly two hops away* that are illicit  
  (excluding self and immediate neighbors)

> Strict 2-hop exposure isolates **layering behavior** while preventing uncontrolled risk propagation.

---

## Why 2-Hop Is the Sweet Spot
- Captures intentional laundering behavior (layering)
- Prevents graph explosion and excessive false positives
- Aligns with industry crypto AML practices
- Defensible to regulators and model validators

---

## Risk Scoring Engine

### Calibration (`fit_risk_config`)
- Percentile-based thresholds (p95 / p99)
- Robust to heavy-tailed graph features
- Option to calibrate using known labels only

### Scoring (`score_transaction`)
Each transaction is scored using:
- illicit exposure (strongest signal)
- graph structure (context signal)

Scores are additive and fully explainable.

### Severity Bands
| Score | Severity |
|----|----|
| 0–2 | Low |
| 3–4 | Medium |
| 5–7 | High |
| ≥8 | Critical |

---

## Alert Output (`get_alerts`)
The system produces an **investigation-ready alert table** containing:
- transaction ID
- severity
- risk score
- graph structure features
- illicit exposure metrics
- human-readable alert reasons

Typical alert distribution:
- ~96% Low  
- ~2% Medium  
- ~1–2% High  
- ~0.2% Critical  

This closely mirrors realistic AML alert volumes.
