# eBay Auction Competitiveness Analysis

Classifying eBay auctions as competitive (≥ 2 bids) or non-competitive using k-Nearest Neighbours and Decision Tree models, with exploratory data analysis and business-oriented model comparison.

**Best model accuracy: 0.715 (Decision Tree)** · Dataset: 1,972 auctions · Task: Binary classification

---

## Overview

This project analyses 1,972 eBay auctions from May–June 2004 to predict whether an auction will attract competitive bidding (defined as ≥ 2 bids). Using only features available at listing time — seller rating, opening price, category, duration, currency, and end day — the analysis develops two classifiers, compares their predictive and interpretive value, and derives actionable recommendations for sellers.

---

## Dataset

The dataset is publicly available and included in this repository:

| File | Description |
|------|-------------|
| `ebay_auction_data.xlsx` | 1,972 eBay auctions with features and competitiveness label |

**Features:**

| Feature | Type | Description |
|---------|------|-------------|
| `Category` | Categorical | Product type (18 categories) |
| `SellerRating` | Numeric | Seller's eBay reputation score |
| `Duration` | Numeric | Auction length in days (1–10) |
| `OpenPrice` | Numeric | Starting bid price |
| `ClosePrice` | Numeric | Final closing price (excluded from new-auction prediction) |
| `Currency` | Categorical | Listing currency (USD, GBP, EUR) |
| `EndDay` | Categorical | Day of week the auction closed |
| `Competitive` | Binary (target) | 1 if ≥ 2 bids placed, 0 otherwise |

No missing values were present — no imputation was required.

---

## Methodology

### 1 · Data Preprocessing

- **One-hot encoding** applied to all categorical variables (all levels retained — no baseline dropped — to preserve interpretability in tree and k-NN models)
- **Outlier flags** created for extreme values rather than removing records, to preserve data for EDA while allowing clean subsets for modelling
- **Log transforms** (`log1p`) applied to skewed features: `OpenPrice`, `ClosePrice`, `SellerRating`
- **Price ratio** engineered as `(1 + ClosePrice) / (1 + OpenPrice)` to capture relative bidding dynamics
- **Seller tiers** created by quantile-binning `SellerRating` into four groups: Low, Mid, High, Top
- **Currency normalisation** to USD for k-NN distance calculations

### 2 · Exploratory Data Analysis

Key findings from EDA:

- **Price ratio** is the strongest visual separator — auctions where the closing price far exceeds the opening price are consistently competitive
- **Seller reputation** matters: Top-tier sellers run significantly more competitive auctions, reflecting bidder trust
- **Category effects** are strong: Photography, Electronics, and Sporting Goods show the highest competitiveness rates; Health/Beauty and EverythingElse the lowest
- **Auction duration** positively correlates with competitiveness — longer listings give more buyers a chance to discover and bid
- **Currency and end day** show minimal standalone impact, though interaction effects with category may exist

Visualisations include bar charts by category/tier/duration, boxplots of log-transformed price distributions, and a heatmap of competitiveness by Category × Seller Tier.

### 3 · Model 1 — k-Nearest Neighbours (k-NN)

- **Features**: `Category`, `Currency`, `EndDay`, `SellerRating`, `Duration` (pre-listing only; `ClosePrice` excluded)
- **Tuning**: 5-fold cross-validation over k = 1–31; optimal **k = 5**
- **Split**: 60% train / 40% test

| Metric | Value |
|--------|-------|
| Test accuracy | 0.674 |
| Precision (competitive) | 0.717 |
| Recall (competitive) | 0.777 |
| F1 score | 0.745 |

The model successfully captures ~78% of truly competitive auctions but has a notable false-positive rate. Best used as a pre-screening tool when the cost of missing competitive auctions outweighs the cost of false positives.

### 4 · Model 2 — Decision Tree

- **Full tree**: trained on all predictors including `ClosePrice` (post-auction reference only)
- **Practical tree**: retrained excluding `ClosePrice` for real-world new-auction prediction; key predictors are `OpenPrice` and `SellerRating`
- **Tuning**: minimum 50 records per terminal node; cross-validated parameter search

| Metric | Value |
|--------|-------|
| Test accuracy | 0.715 |
| Precision (competitive) | 0.723 |
| Recall (competitive) | 0.776 |
| F1 score | 0.749 |

`OpenPrice` and `SellerRating` jointly explain over 95% of feature importance. The tree reveals that low opening prices and lower-rated sellers (who use aggressive pricing) are surprisingly associated with competitive auctions, while very high opening prices with high-rated sellers tend to be non-competitive.

---

## Model Comparison

| Criterion | k-NN (k=5) | Decision Tree |
|-----------|-----------|---------------|
| Test accuracy | 0.674 | **0.715** |
| Precision (competitive) | 0.717 | 0.723 |
| Recall (competitive) | **0.777** | 0.776 |
| F1 score | 0.745 | **0.749** |
| Interpretability | Low | **High** |
| Computation | Heavier at inference | Efficient |

The decision tree is superior in both predictive accuracy and interpretability, providing explicit threshold rules that are directly actionable by sellers. k-NN is useful as a complementary pre-screening tool given its marginally higher recall.

---

## Key Findings & Seller Recommendations

1. **Set a strategic opening price** — medium-range opening prices attract more bidders; very high prices deter participation, very low prices may signal low value
2. **Build and maintain seller rating** — top-tier sellers enjoy significantly higher competitiveness rates due to greater bidder trust
3. **List for longer durations** — 7–10 day auctions consistently outperform 3-day listings in competitiveness
4. **Category selection matters** — Photography, Electronics, and Sporting Goods are inherently more competitive; adjust expectations and pricing accordingly
5. **Price ratio over absolute price** — relative price growth is a stronger signal of competitiveness than raw prices; a low opening bid that escalates is the hallmark of a competitive auction

---

## Project Structure

```
ebay-auction-competitiveness/
├── ebay_auction_data.xlsx          # Public dataset (included)
├── ebay_auction_competitiveness.ipynb  # Full analysis: EDA, k-NN, Decision Tree
├── report.pdf                      # Full project report with findings and visuals
├── README.md
└── .gitignore
```

---

## Reproducing Results

The notebook is fully executable on **Google Colab** with no local setup required.

1. Upload `ebay_auction_competitiveness.ipynb` and `ebay_auction_data.xlsx` to Google Colab
2. Ensure both files are in the same directory (the notebook reads `ebayAuctions.xlsx` from the working directory)
3. Run all cells in order — total runtime is under 2 minutes

**Dependencies** (all pre-installed on Colab):
```
pandas, numpy, matplotlib, seaborn, scikit-learn, openpyxl, graphviz
```

---

## AI Usage Acknowledgement

Generative AI tools (ChatGPT, Claude, MS Copilot) were used as supplementary aids for code drafting, debugging, and documentation. All outputs were independently reviewed, validated, and adapted by the team. Refer to the AI Appendix in `report.pdf` for full disclosure of tools and prompts used.
