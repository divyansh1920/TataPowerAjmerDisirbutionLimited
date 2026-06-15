1 — README.mdmarkdown# TPADL Theft Detection Engine v3.0
### Predictive Meter Data Analytics for AT&C Loss Identification and Revenue Optimization

> **MBA Capstone Project** · Tata Power Ajmer Distribution Limited (TPADL) · 2025  
> **Author:** Divyansh Jain (084015)

---

## Project Overview

India's power distribution sector loses an estimated **$17 billion annually** to electricity theft, with national AT&C losses climbing back to **16.12% in FY 2023-24** after a brief decline. TPADL's estimated AT&C stands at ~18% — 3 percentage points above the RDSS national target.

This project builds a **three-layer, rule-based, interpretable theft detection framework** that scores all 111,792 consumers across 12 months of smart meter data, produces a prioritised inspection dispatch list, and quantifies the financial impact of AT&C loss reduction at both the subsidiary (TPADL) and group (Tata Power) level.

The framework is designed for **field operability** — every formula, weight, and threshold is documented so billing engineers can read, audit, and calibrate it without data science expertise.

---

## Architecture
Raw Dataset (164,076 consumers)

│

▼

┌─────────────────────────────────────────────────┐

│  PREPROCESSING                                   │

│  P1 → Null out months ≤10 kWh (noise)           │

│  P2 → Exclude consumers with ≤5 valid months    │

│  P3 → Compute Coverage Ratio for each consumer  │

└─────────────────────────────────────────────────┘

        │                        │

    111,792 SCORED           52,284 EXCLUDED

│                   (Watchlist)

▼

┌─────────────────────────────────────────────────┐

│  LAYER 1 — CPI (40%)  Consumption Pattern Index │

│  7 Variables: CTS · CVI · DSR · ZSB             │

│               SCAS · FCS · PDS  ← NEW in v3.0   │

└─────────────────────────────────────────────────┘

│

┌─────────────────────────────────────────────────┐

│  LAYER 2 — LCI (35%)  Load Compliance Index     │

│  4 Variables: LFR · MOI · MCGI · ZCSS           │

└─────────────────────────────────────────────────┘

     │

┌─────────────────────────────────────────────────┐

│  LAYER 3 — VHS (25%)  Violation History Score   │

│  1 Variable: Violation Count (step function)    │

└─────────────────────────────────────────────────┘

│

       ▼

Final_Score = (0.40 × CPI) + (0.35 × LCI) + (0.25 × VHS)

│

   OVERRIDE RULES

O1: Violation ≥2 AND CPI ≥55 → HIGH  (0 fired)

O2: MDI_Ratio ≥ 2.0          → HIGH  (49,859 fired)

O3: Load ≥25kW AND LFR <3%   → HIGH  (122 fired)

│

▼

HIGH (≥65) · MEDIUM (35–64) · LOW (<35)

│

FINANCIAL MODULE

Peer-benchmark leakage · AT&C scenarios

NPV · BCR · Inspection ROI

---

## Dataset

| Parameter | Value |
|---|---|
| Source | Tata Power Ajmer Distribution Limited (TPADL) |
| Raw consumer base | 164,076 |
| Scored pool (after preprocessing) | 111,792 |
| Excluded (watchlist) | 52,284 (31.9%) |
| Data period | 12 months (Jan–Dec) |
| Features per consumer | 34 raw → 39 engineered |
| Tariff source | AVVNL Tariff for Supply of Electricity 2023 |

**Rate category breakdown of scored pool:**

| Category | Consumers | Share |
|---|---|---|
| DS-LT1 (Domestic Single-Phase) | 94,970 | 84.95% |
| NDS-LT2 (Non-Domestic LT) | 14,574 | 13.04% |
| SP-LT5 (Special Purpose Industrial) | 1,339 | 1.20% |
| PSL-LT3 (Public Street Lighting) | 399 | 0.36% |
| AG-MS-LT4 (Agricultural) | 372 | 0.33% |
| Others (HT/Mixed) | 138 | 0.12% |

---

## Detection Layers

### Layer 1 — CPI (Consumption Pattern Index, 40%)

| Variable | Weight | Signal |
|---|---|---|
| CTS — Consumption Trend Slope | 20% | Sustained falling trend → bypass installed |
| CVI — Consumption Volatility Index | 20% | Erratic month-to-month → partial bypass |
| DSR — Drop Severity Ratio | 15% | Extreme peak-to-trough gap |
| ZSB — Z-Score Statistical Break | 10% | Single-month cliff → mid-year bypass |
| SCAS — Summer Consumption Anomaly ⭐ | 15% | Low summer vs off-season → Rajasthan bypass |
| FCS — Flat Consumption Score ⭐ | 10% | Near-zero CoV → tampered register |
| PDS — Peer Deviation Score ⭐ | 10% | Below peer median → under-recording |

⭐ New in v3.0 — fills gaps in original 4-variable CPI

### Layer 2 — LCI (Load Compliance Index, 35%)

| Variable | Weight | Signal |
|---|---|---|
| LFR — Load Factor Ratio | 30% | kWh << sanctioned load × hours |
| MOI — MDI Overdrawal Index | 25% | Peak demand > 2× sanctioned capacity |
| MCGI — MDI-Consumption Gap Index | 30% | kWh inconsistent with MDI-implied draw |
| ZCSS — Zero Coverage Suspicion Score | 15% | Missing months on large connections |

### Layer 3 — VHS (Violation History Score, 25%)

| Violation Count | VHS Score |
|---|---|
| 0 | 0 |
| 1 | 40 |
| 2 | 65 |
| 3 | 82 |
| ≥4 | 100 |

---

## Detection Results

| Category | Count | % of Scored Pool |
|---|---|---|
| **HIGH Risk** | **49,981** | **44.71%** |
| MEDIUM Risk | 550 | 0.49% |
| LOW Risk | 61,261 | 54.80% |
| EXCLUDED (watchlist) | 52,284 | — |

**Override breakdown:**
- Override O1 (Violation + CPI): **0 fires** — violation history too sparse (99.5% zero)
- Override O2 (MDI ≥ 2×): **49,859 fires** — primary detection mechanism
- Override O3 (Large load + near-zero LFR): **122 fires**

**Maximum computed Final Score:** 59.77 (no consumer reached 65 via pure scoring — data constraints)

**Top 10,000 consumers:** 7.15 MU estimated stolen energy · ₹5.25 Crore gross annual loss

---

## Financial Impact

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

*NPV at 10% WACC. PAT at 25% corporate tax. Distribution volume: 326.19 MU/year.*

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

## Repository Structure
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

│   ├── TPADL_Complete_Theft_Analysis_Report.csv    # Full scored consumer table

│   ├── TPADL_Complete_Theft_Analysis_Report_Top10000.csv

│   ├── TPADL_Complete_Theft_Analysis_Report_Dashboard.csv

│   ├── TPADL_Financial_Analysis_Data.csv           # Scenario financials

│   ├── TPADL_Financial_Analysis_Report_*.csv       # Module-wise outputs

│   └── TPADL_Framework_v3.docx                     # Framework documentation

│

├── 📁 website/

│   ├── index.html                  # Main project dashboard website

│   └── data.html                   # Source data sheets (embedded Google Sheets)

│

├── 📁 report/

│   └── Project_Report.docx         # Full academic research report

│

└── README.md

---

## Tech Stack

| Tool | Use |
|---|---|
| Python 3.x | Detection engine, financial module |
| Google Colab | Primary development environment |
| pandas / numpy | Data processing and scoring |
| openpyxl | Multi-sheet Excel workbook generation |
| Chart.js 4 | Interactive dashboard visualisations |
| HTML / CSS / JS | Project website (no framework) |
| Google Sheets | Live data embedding |
| Google Fonts (Rajdhani + Inter + JetBrains Mono) | Dashboard typography |

---

## How to Run

### Detection Engine
```bash
# Clone repository
git clone https://github.com/[username]/TPADL-Theft-Detection-Engine.git

# Install dependencies
pip install pandas numpy openpyxl

# Run detection engine
python python/tpadl_theft.py

# Run financial analysis
python python/tpadl_financials.py
```

### Website
Open `website/index.html` directly in a browser. No server required.  
`website/data.html` requires internet access (embedded Google Sheets via iframe).

---

## Key Findings

1. **Override O1 = 0 fires** — 99.5% of consumers have zero violation history, proving that enforcement-history-dependent detection systems fail in first-cycle Indian DISCOM contexts. Behavioral scoring is the only viable approach.

2. **Override O2 dominates** — 49,859 of 49,981 HIGH consumers were elevated because MDI exceeded double their sanctioned load. The MDI signal is the most powerful available indicator in monthly data without electrical parameter readings.

3. **Agriculture flag rate 30.9%** — AG-MS-LT4 consumers show the highest proportional flagging rate despite being only 0.33% of the scored pool. SCAS correctly identifies summer irrigation bypass patterns.

4. **Max score 59.77** — No consumer reached the 65-point HIGH threshold via pure computation. Phase 2 data (billing + DT mapping + V/I readings) is required to activate pure-score HIGH classifications.

5. **₹24.02 Crore net NPV, 2.91× BCR, 8.68-month payback** — The investment case is robust across all 16 parameter combinations tested. Even the most conservative scenario produces positive NPV.

---

## Three Research Contributions

1. **Interpretable rule-based architecture** — No black-box ML. Every rule is auditable by billing engineers, satisfying legal enforceability requirements for Indian revenue tribunals.

2. **Network validation via override rules** — Override O3 (large load + near-zero LFR) formalizes zone-level dormant connection detection, not addressed in prior single-study frameworks.

3. **Field-linked financial quantification** — The first study (to our knowledge) linking consumer-level detection outputs directly to ACS-ARR gap, subsidiary EBITDA, PAT addback, EPS accretion, BCR, and consumer-level break-even thresholds.

---

## Limitations

| Limitation | Impact | Phase 2 Fix |
|---|---|---|
| No monthly billing data | CDI and BMR uncomputable | P1 priority data request |
| No V/I/PF readings | CT tampering undetectable | P2 priority |
| No DT mapping | Zone-level balancing impossible | P1 priority |
| Monthly granularity only | Temporal evasion undetectable | Hourly data (P2) |
| Single year of data | Year-on-year comparison unavailable | Accumulates automatically |
| Estimated leakage only | Not for use in demand notices | Confirmed field evidence required |

---

## References

All academic references are cited in the full project report. Key sources:

- Power Finance Corporation (2024). *Report on the Performance of State Power Utilities 2023-24*
- Savian et al. (2021). Non-technical losses: A systematic review. *Renewable and Sustainable Energy Reviews*, 147, 111205
- Kawoosa et al. (2023). Machine learning ensemble for energy theft detection. *IET Generation, Transmission & Distribution*, 17(16)
- AVVNL. *Tariff for Supply of Electricity 2023* — All tariff figures sourced from this official order

---

## Author

**Divyansh Jain — 084015**  
MBA Capstone Project · Tata Power Ajmer Distribution Limited · 2025

> *This framework is a targeting tool, not a verdict. Every consumer flagged HIGH requires physical field verification before enforcement action. All financial leakage figures are peer-benchmark estimates and must not be used in demand notices without confirmed field evidence.*
