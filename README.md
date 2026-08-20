# Disbursement Performance Analytics — World Bank Lending Portfolio

Flagging **slow-disbursing projects** across the World Bank's IBRD portfolio, and surfacing
what drives them — built from public loan-level data, with an interactive Tableau dashboard.

> **Why this project.** In donor-funded programmes the disbursement ratio (funds paid out ÷
> funds committed) is *the* implementation KPI. Undisbursed commitments lock up scarce capital
> and are the leading signal of projects heading for restructuring or cancellation. This is the
> same withdrawal-application / SOE logic used inside a Project Coordination Unit (PCU), applied
> to the entire Bank portfolio.

## Question
Which projects, countries and sectors disburse committed funds efficiently — and can we
predict, from characteristics known early, which loans will fall behind schedule?

## Key findings
- Of 9,513 IBRD loans, **672 are in the active disbursement phase**; **38.7% of them are behind
  their implied disbursement pace** — nearly 4 in 10.
- The strongest *actionable* driver of slow disbursement is a long **approval → effectiveness
  lag**: loans slow to become effective are slow to disburse.
- An interpretable **logistic model (test ROC-AUC 0.75)** stays competitive with — and far more
  defensible than — a Random Forest that overfits the small sample. The honest number wins.

## Dashboard
Interactive portfolio dashboard on **[Tableau Public](https://public.tableau.com/app/profile/tarteel.abu.ghazaleh/viz/IBRDDisbursement/IBRDDisbursementPerformanceAnalytics)** —
regional disbursement rates, an early-warning scatter (effectiveness lag vs disbursement gap),
an undisbursed-balance treemap by country, and slow-loan counts by approval vintage.

![Dashboard preview](reports/dashboard.png)

## Data
**IBRD Statement of Loans — Latest Snapshot** (World Bank Group Finances), resource `sfv5-tf7p`
on `finances.worldbank.org`, one row per loan. Licensed **CC-BY 4.0**. Fields include original
principal, disbursed / undisbursed / cancelled amounts, Board approval, effectiveness and
closing dates, region, country, instrument and loan status.

```bash
python src/fetch_data.py          # writes data/statement_of_loans.csv
```
A synthetic sample (`data/sample_statement_of_loans.csv`) is bundled so the notebook runs
offline before the real pull.

## Approach
1. **Feature engineering with financial meaning** — disbursement ratio, undisbursed ratio,
   approval→effectiveness lag, elapsed share of planned life, and a disbursement *gap*.
2. **Population definition** — restrict to loans genuinely in the disbursement phase
   (Disbursing, Effective, Signed, Approved); exclude Repaying/Fully Disbursed/Cancelled, whose
   disbursement pace is no longer a live question.
3. **Target** — a management-style rule: a loan is *slow* when it lags its implied pace by more
   than 25 percentage points.
4. **Modelling** — logistic baseline → SMOTE for class imbalance → PCA experiments → Random
   Forest comparison, with feature importance for the drivers.

## Methodology note
An initial run that kept *Repaying* loans in the population produced an inflated AUC (~0.90) and
let loan age dominate — the model was mostly separating old, repaid loans from young ones.
Tightening the population to the genuinely disbursement-phase portfolio dropped AUC to a truthful
0.75 and let the meaningful effectiveness-lag signal surface. *The more honest number was the
more useful one.*

## Structure
```
wb-disbursement-analytics/
├── README.md
├── requirements.txt
├── src/fetch_data.py                      # SODA API pull
├── notebooks/01_disbursement_analytics.ipynb
├── data/                                  # sample bundled; real pull is git-ignored
└── reports/                               # dashboard screenshot / figures
```

## Reproduce
```bash
pip install -r requirements.txt
python src/fetch_data.py
jupyter lab notebooks/01_disbursement_analytics.ipynb
```

## Caveats
Small active sample (~613 modelled rows) — the model is illustrative, not production-grade.
"Slow disburser" is a management heuristic, not an official World Bank classification. Scope is
IBRD only (middle-income borrowers); adding the IDA Statement of Credits & Grants would extend
coverage to the poorest countries and grant financing.

---
*Data: World Bank Group Finances — IBRD Statement of Loans (CC-BY 4.0). Modified: feature
engineering, filtering, and classification applied. Independent analysis; not endorsed by the
World Bank.*
