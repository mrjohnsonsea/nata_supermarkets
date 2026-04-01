# NATA Supermarkets — Customer Analytics

An end-to-end customer analytics project combining unsupervised segmentation and supervised spending prediction on a real-world supermarket dataset.

**Authors:** Dylan Lowndes, Emma Cranmer, Israel Mendoza, Mike Johnson, Lola Oshodi

## Overview

NATA Supermarkets wants to better understand its customer base to allocate marketing spend more effectively. This project answers two questions:

1. **Who are our customers?** — K-Means clustering segments customers by income, family structure, and purchasing behavior.
2. **How much will they spend?** — A linear regression model predicts annual spend from customer attributes, achieving R² = 0.76 on held-out data.

## Dataset

| | |
|---|---|
| Source | Ivey Business School (W33836) |
| Time Period | 2012–2014 |
| Records | 2,240 customers (2,205 after cleaning) |
| Features | 28 variables |
| Coverage | Demographics, product spend by category, channel usage, campaign response |

**Product categories:** Wines, Fruits, Meat, Fish, Sweets, Gold products  
**Purchase channels:** Web, Catalog, Store, Deals

**Key descriptive stats (post-cleaning):**
- Median income: $51,287 | Range: $1,730–$113,734
- Average customer age: ~56 years
- Average spend over 2 years: $606 | Range: $5–$2,525
- Average recency (days since last purchase): 49 days

## Methodology

```
Raw Data → Cleaning & Outlier Removal → EDA → K-Means Clustering → Linear Regression → Evaluation
```

**Data Preparation**
- Removed records with implausible birth years (pre-1940) — flagged as likely data entry errors
- Applied IQR-based outlier removal on Income (Q3 + 1.5×IQR threshold, ~$100K)
- 24 missing Income values resolved through outlier removal — no imputation required
- Consolidated irregular Marital_Status categories ("Absurd", "Alone", "YOLO" → Single)

**Customer Segmentation (Unsupervised)**
- Standardized all numeric features with `StandardScaler`
- One-hot encoded categorical variables
- Selected k=2 via elbow method on SSE curve

**Spending Prediction (Supervised)**
- Target: `total_purchases` (sum across all six product categories)
- Feature selection via correlation analysis — excluded multicollinear variables (NumCatalogPurchases, NumStorePurchases correlated with Income at r ≈ 0.70)
- Final features: `Income` + `cluster` membership (one-hot encoded)
- 80/20 train/test split · 10-fold cross-validation

## Results

### Customer Segments

| | Cluster 0 — Value Families | Cluster 1 — Affluent Professionals |
|---|---|---|
| Avg Income | ~$38,000 | ~$71,000 |
| Avg Children at Home | 0.71 | 0.07 |
| Avg Teens at Home | 0.55 | 0.45 |
| Behavior | Price-sensitive, lower spend | High-spend, fewer dependents |

### Predictive Model

| Metric | Train | Test |
|---|---|---|
| R² | 0.79 | 0.76 |
| RMSE | $279 | $280 |
| MAE | $202 | $208 |
| Cross-Val R² (10-fold) | 0.786 ± 0.031 | — |

**Coefficients:**
- `Income`: +$0.01 per $1 increase in annual income
- `Cluster 1 (High Income)`: +$664 vs. low-income segment

Minimal train/test gap confirms the model generalizes without overfitting.

### Key Insights

- **Wine and meat dominate spend** — these two categories account for ~75% of total customer spending over two years. Even small improvements in these categories meaningfully impact revenue.
- **Income is the primary spending driver** — the strongest single predictor of basket size.
- **Couples outspend singles by 15–20%** — married and "together" customers are a high-value segment.
- **Families with children spend less overall** — household budget constraints suppress discretionary spend (Kidhome correlation with total spend: r = -0.57).
- **High web visit frequency correlates with lower spend** — likely deal-seeking behavior, not purchase intent; may signal a conversion friction point.

## Recommendations

**Cluster 1 — Affluent Professionals:**
- Promote premium products and specialty categories (wines, meats, fish)
- Curated bundles (e.g., wine + meat pairings) to increase average basket size
- Personalized loyalty programs with lifestyle-aligned incentives and exclusive experiences

**Cluster 0 — Value-Oriented Families:**
- Lead with value messaging and essential-item discounts
- Household promotions ("meals under $20", family bundles)
- Gamified loyalty tied to spending thresholds to encourage incremental purchases

**Across Both Segments:**
- Align inventory and promotions to wine and meat category dominance
- Investigate web channel UX — high visit-to-purchase gap warrants A/B testing on pricing and checkout flow
- Run real-time campaigns calibrated to each segment's behavioral profile

## Project Structure

```
.
├── customer_analytics.ipynb      # Full analysis notebook
├── data/
│   ├── nata_supermarkets.xlsx    # Source dataset (Ivey Business School, W33836)
│   ├── written_report.pdf        # Group written analysis (April 2025)
│   └── case_study_brief.pdf      # Original Ivey case study document
└── README.md
```

## Setup

```bash
pip install pandas numpy scikit-learn matplotlib seaborn openpyxl
jupyter notebook customer_analytics.ipynb
```

Python 3.9+ required.
