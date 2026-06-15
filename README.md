# TPADL Theft Detection Engine v3.0
### Predictive Meter Data Analytics for AT&C Loss Identification & Revenue Optimization

> **MBA Capstone** · Tata Power Ajmer Distribution Limited (TPADL) · 2025  
> **Author:** Divyansh Jain (084015)

---

<div align="center">

![Python](https://img.shields.io/badge/Python-3.x-blue?style=for-the-badge&logo=python)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Engine-150458?style=for-the-badge&logo=pandas)
![Chart.js](https://img.shields.io/badge/Chart.js-Dashboard-FF6384?style=for-the-badge&logo=chartdotjs)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)

**111,792 consumers scored · ₹24.02 Cr net NPV · 2.91× BCR · 8.68-month payback**

</div>

---

## 📌 Overview

India's power sector loses **$17B annually** to electricity theft. National AT&C losses hit **16.12% in FY 2023-24**. TPADL's AT&C ~18% — 3 pts above RDSS target.

This project builds a **three-layer, rule-based, interpretable theft detection framework** that:
- Scores all 111,792 consumers across 12 months of smart meter data
- Produces prioritised inspection dispatch list
- Quantifies financial impact at TPADL and Tata Power group level

> Designed for **field operability** — every formula, weight, threshold documented. Billing engineers can read, audit, calibrate. No data science expertise required.

---

## 🏗️ Architecture

```
Raw Dataset (164,076 consumers)
│
▼
┌─────────────────────────────────────────────────┐
│  PREPROCESSING                                   │
│  P1 → Null out months ≤10 kWh (noise)           │
│  P2 → Exclude consumers with ≤5 valid months    │
│  P3 → Compute Coverage Ratio per consumer        │
└─────────────────────────────────────────────────┘
        │                        │
   111,792 SCORED           52,284 EXCLUDED (Watchlist)
        │
        ▼
┌─────────────────────────────────────────────────┐
│  LAYER 1 — CPI (40%)  Consumption Pattern Index │
│  CTS · CVI · DSR · ZSB · SCAS · FCS · PDS      │
└─────────────────────────────────────────────────┘
        │
┌─────────────────────────────────────────────────┐
│  LAYER 2 — LCI (35%)  Load Compliance Index     │
│  LFR · MOI · MCGI · ZCSS                        │
└─────────────────────────────────────────────────┘
        │
┌─────────────────────────────────────────────────┐
│  LAYER 3 — VHS (25%)  Violation History Score   │
│  Violation Count (step function)                 │
└─────────────────────────────────────────────────┘
        │
        ▼
  Final_Score = (0.40 × CPI) + (0.35 × LCI) + (0.25 × VHS)
        │
   OVERRIDE RULES
   O1: Violation ≥2 AND CPI ≥55  → HIGH  [0 fired]
   O2: MDI_Ratio ≥ 2.0           → HIGH  [49,859 fired]
   O3: Load ≥25kW AND LFR <3%   → HIGH  [122 fired]
        │
        ▼
   HIGH (≥65) · MEDIUM (35–64) · LOW (<35)
```

---

## 📊 Dataset

| Parameter | Value |
|---|---|
| Source | Tata Power Ajmer Distribution Limited |
| Raw consumer base | 164,076 |
| Scored pool (after preprocessing) | 111,792 |
| Excluded (watchlist) | 52,284 (31.9%) |
| Data period | Jan–Dec (12 months) |
| Features per consumer | 34 raw → 39 engineered |
| Tariff source | AVVNL Tariff 2023 |

**Rate category breakdown:**

| Category | Consumers | Share |
|---|---|---|
| DS-LT1 (Domestic Single-Phase) | 94,970 | 84.95% |
| NDS-LT2 (Non-Domestic LT) | 14,574 | 13.04% |
| SP-LT5 (Special Purpose Industrial) | 1,339 | 1.20% |
| PSL-LT3 (Public Street Lighting) | 399 | 0.36% |
| AG-MS-LT4 (Agricultural) | 372 | 0.33% |
| Others (HT/Mixed) | 138 | 0.12% |

---

## 🔍 Detection Layers

### Layer 1 — CPI · Consumption Pattern Index (40%)

| Variable | Weight | Signal |
|---|---|---|
| CTS — Consumption Trend Slope | 20% | Sustained falling trend → bypass installed |
| CVI — Consumption Volatility Index | 20% | Erratic month-to-month → partial bypass |
| DSR — Drop Severity Ratio | 15% | Extreme peak-to-trough gap |
| ZSB — Z-Score Statistical Break | 10% | Single-month cliff → mid-year bypass |
| ⭐ SCAS — Summer Consumption Anomaly | 15% | Low summer vs off-season → Rajasthan bypass |
| ⭐ FCS — Flat Consumption Score | 10% | Near-zero CoV → tampered register |
| ⭐ PDS — Peer Deviation Score | 10% | Below peer median → under-recording |

> ⭐ New in v3.0 — fills gaps in original 4-variable CPI

### Layer 2 — LCI · Load Compliance Index (35%)

| Variable | Weight | Signal |
|---|---|---|
| LFR — Load Factor Ratio | 30% | kWh << sanctioned load × hours |
| MOI — MDI Overdrawal Index | 25% | Peak demand > 2× sanctioned capacity |
| MCGI — MDI-Consumption Gap Index | 30% | kWh inconsistent with MDI-implied draw |
| ZCSS — Zero Coverage Suspicion Score | 15% | Missing months on large connections |

### Layer 3 — VHS · Violation History Score (25%)

| Violation Count | Score |
|---|---|
| 0 | 0 |
| 1 | 40 |
| 2 | 65 |
| 3 | 82 |
| ≥4 | 100 |

---

## 📈 Results

| Category | Count | % of Scored Pool |
|---|---|---|
| 🔴 HIGH Risk | 49,981 | 44.71% |
| 🟡 MEDIUM Risk | 550 | 0.49% |
| 🟢 LOW Risk | 61,261 | 54.80% |
| ⚪ EXCLUDED (watchlist) | 52,284 | — |

**Override breakdown:**
- O1 (Violation + CPI): **0 fires** — 99.5% consumers have zero violation history
- O2 (MDI ≥ 2×): **49,859 fires** — primary detection mechanism
- O3 (Large load + near-zero LFR): **122 fires**

> Max computed Final Score: **59.77** — no consumer reached 65 via pure scoring. Phase 2 data required to activate pure-score HIGH.

**Top 10,000 consumers:** 7.15 MU estimated stolen · ₹5.25 Crore gross annual loss

---

## 💰 Financial Impact

### Detection Module

| Metric | Value |
|---|---|
| Gross Annual Revenue Loss Detected | ₹3.56 Crore |
| Recoverable (60% factor) | ₹2.13 Crore |
| Detection Capture Rate | 15.26% of commercial loss pool |
| Total Commercial Loss Pool | ₹23.29 Crore |
| Blended Tariff (AVVNL 2023) | ₹7.14/kWh |

### AT&C Reduction Scenarios

| Scenario | AT&C Target | Annual Saving | 10-Yr NPV | PAT Impact | TPADL EBITDA |
|---|---|---|---|---|---|
| Conservative | 16.5% | ₹3.49 Cr | ₹21.47 Cr | ₹2.62 Cr | +16.3% |
| **Base (RDSS Target)** | **15.0%** | **₹6.99 Cr** | **₹42.94 Cr** | **₹5.24 Cr** | **+32.6%** |
| Optimistic | 13.0% | ₹11.65 Cr | ₹71.57 Cr | ₹8.74 Cr | +54.3% |
| Best Case | 12.0% | ₹13.98 Cr | ₹85.88 Cr | ₹10.48 Cr | +65.2% |

*NPV at 10% WACC · PAT at 25% corporate tax · Distribution volume: 326.19 MU/year*

### Investment Case (Base Scenario)

| Metric | Value |
|---|---|
| Net Programme NPV (10-year) | ₹24.02 Crore |
| Benefit-Cost Ratio | **2.91×** |
| Payback Period | **8.68 months** |
| Full Programme Cost | ₹15.16 Crore (50,531 visits × ₹3,000) |
| Year-1 Break-even Threshold | 58.35 kWh/consumer/month |
| Perpetuity Break-even Threshold | 3.50 kWh/consumer/month |

---

## 🗂️ Repository Structure

```
TPADL-Theft-Detection-Engine/
│
├── 📁 python/
│   ├── tpadl_theft.py              # Layer scoring engine (CPI + LCI + VHS)
│   └── tpadl_financials.py         # Financial analysis module
│
├── 📁 data/
│   ├── combined_anonymized_unique_id_summary.csv   # Raw 164,076-consumer dataset
│   ├── TPADL_Clean_Data.csv                        # Scored pool (111,792)
│   ├── TPADL_Layer1_CPI_v2.csv                     # Layer 1 outputs
│   └── TPADL_Layer2_LCI_v1_up.csv                  # Layer 2 outputs
│
├── 📁 outputs/
│   ├── TPADL_Complete_Theft_Analysis_Report.csv
│   ├── TPADL_Complete_Theft_Analysis_Report_Top10000.csv
│   ├── TPADL_Complete_Theft_Analysis_Report_Dashboard.csv
│   ├── TPADL_Financial_Analysis_Data.csv
│   ├── TPADL_Financial_Analysis_Report_*.csv
│   └── TPADL_Framework_v3.docx
│
├── 📁 website/
│   ├── index.html                  # Main project dashboard
│   └── data.html                   # Source data sheets (embedded Google Sheets)
│
├── 📁 report/
│   └── Project_Report.docx
│
└── README.md
```

---

## 🛠️ Tech Stack

| Tool | Use |
|---|---|
| Python 3.x | Detection engine, financial module |
| Google Colab | Primary development environment |
| pandas / numpy | Data processing and scoring |
| openpyxl | Multi-sheet Excel workbook generation |
| Chart.js 4 | Interactive dashboard visualisations |
| HTML / CSS / JS | Project website (no framework) |
| Google Sheets | Live data embedding |
| Google Fonts | Rajdhani + Inter + JetBrains Mono |

---

## 🚀 How to Run

### Detection Engine

```bash
# Clone repo
git clone https://github.com/[username]/TPADL-Theft-Detection-Engine.git

# Install dependencies
pip install pandas numpy openpyxl

# Run detection engine
python python/tpadl_theft.py

# Run financial analysis
python python/tpadl_financials.py
```

### Website

Open `website/index.html` directly in browser. No server required.  
`website/data.html` requires internet access (embedded Google Sheets).

---

## 💡 Key Findings

1. **Override O1 = 0 fires** — 99.5% consumers have zero violation history. Enforcement-history-dependent systems fail in first-cycle Indian DISCOM contexts. Behavioral scoring only viable approach.

2. **Override O2 dominates** — 49,859 of 49,981 HIGH consumers elevated because MDI exceeded 2× sanctioned load. MDI = most powerful indicator in monthly data without electrical parameter readings.

3. **Agriculture flag rate 30.9%** — AG-MS-LT4 highest proportional flagging despite 0.33% of scored pool. SCAS correctly identifies summer irrigation bypass patterns.

4. **Max score 59.77** — Phase 2 data (billing + DT mapping + V/I readings) required to activate pure-score HIGH classifications.

5. **₹24.02 Cr NPV · 2.91× BCR · 8.68-month payback** — Investment case robust across all 16 parameter combinations. Even most conservative scenario = positive NPV.

---

## 🔬 Research Contributions

| # | Contribution |
|---|---|
| 1 | **Interpretable rule-based architecture** — No black-box ML. Every rule auditable by billing engineers, satisfying legal enforceability for Indian revenue tribunals |
| 2 | **Network validation via override rules** — Override O3 formalizes zone-level dormant connection detection not addressed in prior frameworks |
| 3 | **Field-linked financial quantification** — First study linking consumer-level detection to ACS-ARR gap, EBITDA, PAT addback, EPS accretion, BCR, and consumer-level break-even thresholds |

---

## ⚠️ Limitations

| Limitation | Impact | Phase 2 Fix |
|---|---|---|
| No monthly billing data | CDI and BMR uncomputable | P1 priority data request |
| No V/I/PF readings | CT tampering undetectable | P2 priority |
| No DT mapping | Zone-level balancing impossible | P1 priority |
| Monthly granularity only | Temporal evasion undetectable | Hourly data (P2) |
| Single year of data | YoY comparison unavailable | Accumulates automatically |
| Estimated leakage only | Not for demand notices | Confirmed field evidence required |

---

## 📚 References

- Power Finance Corporation (2024). *Report on Performance of State Power Utilities 2023-24*
- Savian et al. (2021). Non-technical losses: systematic review. *Renewable & Sustainable Energy Reviews*, 147, 111205
- Kawoosa et al. (2023). ML ensemble for energy theft detection. *IET Generation, Transmission & Distribution*, 17(16)
- AVVNL. *Tariff for Supply of Electricity 2023*

---

## 👤 Author

**Divyansh Jain — 084015**  
MBA Capstone Project · Tata Power Ajmer Distribution Limited · 2025

---

> ⚠️ **Disclaimer:** This framework is a targeting tool, not a verdict. Every HIGH-flagged consumer requires physical field verification before enforcement. All leakage figures are peer-benchmark estimates — must not be used in demand notices without confirmed field evidence.
