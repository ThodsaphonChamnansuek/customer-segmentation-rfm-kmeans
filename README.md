# Customer Segmentation: RFM Analysis vs. K-Means Clustering

Segmenting 5,881 online-retail customers with two approaches, rule-based **RFM scoring** and machine-learning **K-Means clustering**, then comparing them to see where the business rules and the data disagree.

**[View the interactive dashboard on Tableau Public](https://public.tableau.com/shared/J485TR38N)** | [Author's Tableau Public profile](https://public.tableau.com/app/profile/thodsaphon.chamnansuek)

<!-- Add a dashboard screenshot here: ![Dashboard](images/dashboard.png) -->

---

## Business Question

Which customers generate the most revenue, which valuable customers are at risk of leaving, and does a standard RFM scoring rule identify "top customers" accurately?

## Dataset

- **Source:** [Online Retail II](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci) (UCI / Kaggle), transactions from a UK-based online retailer
- **Period:** December 2009 to December 2011
- **Size:** 1,067,371 rows, 8 columns (Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID, Country)
- The raw file is not included in this repo because of its size. Download it from the link above.

## Tools

Python (pandas, scikit-learn, matplotlib, seaborn), Google Colab, Tableau Public

## Methodology

### 1. Data cleaning

| Step                                            | Rows remaining | Removed |
| ----------------------------------------------- | -------------- | ------- |
| Raw data                                        | 1,067,371      | -       |
| Drop rows with missing Customer ID              | 824,364        | 243,007 |
| Drop cancelled orders (Invoice starts with `C`) | 805,620        | 18,744  |

- Parsed `InvoiceDate` with `format='mixed', dayfirst=True` because the column contained more than one date format.
- Created `TotalPrice = Quantity x Price`.
- Verified that no negative quantities remained after cleaning.
- Result: **805,620 transactions from 5,881 customers**.

### 2. RFM analysis

- **Recency:** days since last purchase (reference date = last date in data + 1 day)
- **Frequency:** number of distinct invoices
- **Monetary:** total spend
- Each dimension scored 1-5 by quintile (`qcut`), then mapped to segments: Champions, Loyal Customers, New Customers, At Risk, Lost, Others.

### 3. K-Means clustering

- Standardized R, F, M with `StandardScaler` (otherwise Monetary dominates the distance calculation).
- Chose **K = 5** using the Elbow Method and Silhouette Score.
- K = 2 had the highest silhouette score (0.916) but was rejected: it only split off 22 extreme outliers from 5,859 other customers, which is not a useful segmentation. K = 5 scored 0.600 and matched the elbow.

## Key Findings

**1. Revenue is highly concentrated.**
RFM "Champions" are 22% of customers but generate **68% of revenue** (about 12.1M of 17.7M GBP), a Pareto-style pattern.

**2. At Risk customers are worth saving.**
Only 354 customers (6%), but they average about 3,188 GBP each, second only to Champions, and total about 1.1M GBP in historical revenue. They are the best target for a retention campaign.

**3. RFM's "Champions" label is too broad.**
Comparing the two methods shows that **932 of the 1,300 RFM Champions (72%) fall into the K-Means "Regular" cluster**, with behavior close to an average customer. RFM quintiles rank customers relative to each other and ignore how large the gaps are.

**4. A tiny group drives extreme value.**
K-Means isolated **28 customers (0.5%)**, which are 4 "Super VIP" and 24 "VIP" customers. They buy 120-210+ times and spend 100K-437K GBP on average. This pattern is consistent with wholesale or reseller accounts rather than ordinary retail shoppers.

| Cluster   | Customers | Avg Recency (days) | Avg Frequency | Avg Monetary (GBP) |
| --------- | --------- | ------------------ | ------------- | ------------------ |
| Super VIP | 4         | 3.5                | 212.8         | 436,836            |
| VIP       | 24        | 22.5               | 119.8         | 100,927            |
| Loyal     | 383       | 28.1               | 28.5          | 13,935             |
| Regular   | 3,552     | 75.7               | 5.1           | 1,911              |
| Dormant   | 1,918     | 471.5              | 2.2           | 755                |

## Business Recommendations

- Use **K-Means VIP / Super VIP** as the list for personalized, account-based management (about 28 accounts).
- Use **RFM segments** for broad marketing communication, where easy-to-explain rules matter more than precision.
- Launch a **retention campaign for At Risk customers** first. They have high past value and are still recoverable, unlike the Lost segment.
- Re-define the RFM thresholds using the K-Means cluster boundaries, and reduce the oversized "Others" group (23% of customers).

## Lessons Learned

- **Dirty data hides in formats.** `InvoiceDate` mixed two date formats; the default parser failed and a wrong `dayfirst` guess would have silently shifted dates.
- **Don't trust a metric blindly.** The highest silhouette score (K = 2) came from outliers, not meaningful groups. Checking cluster sizes exposed it.
- **Check dashboard numbers against the notebook.** A "Total Customers" card showed 90 million because Tableau summed Customer IDs instead of counting distinct values.

## Limitations

- Data ends in December 2011, so "recency" is measured against that date, not today.
- About 23% of transactions had no Customer ID and could not be segmented.
- The "wholesale" interpretation of the top clusters is inferred from purchasing behavior; the dataset has no customer-type field to confirm it.
- No predictive model (e.g. churn) was built; this project is descriptive segmentation.

## Repository Structure

```
.
├── README.md
├── customer_segmentation_analysis.ipynb   # full analysis (cleaning, RFM, K-Means, charts)
├── data/
│   └── rfm_final.csv                      # customer-level output used in the dashboard
└── images/
    └── dashboard.png                      # dashboard screenshot
```

## How to Reproduce

1. Download `online_retail_II.csv` from the dataset link above.
2. Open the notebook in Google Colab (or Jupyter) and point the file path to your copy.
3. Run all cells from top to bottom.

## Author

**Thodsaphon Chamnansuek**
[Tableau Public](https://public.tableau.com/app/profile/thodsaphon.chamnansuek)
