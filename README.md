# Online Retail Customer Analytics & Segmentation

Capstone project. I analysed the transactions of an online retailer, built RFM scores for the customers and used clustering to split them into segments. Each segment has a suggested marketing action.

## Business questions

- Which products and customers bring in the most revenue?
- When do people buy (month, weekday, hour)?
- Which countries matter most?
- How common are cancellations and returns?
- Are there signs of wholesale / bulk buying?
- Can customers be grouped into segments that marketing can act on?

## Dataset

UCI Online Retail dataset (about 540,000 invoice lines, Dec 2010 to Dec 2011). Each row is an invoice line, not an order.

## Project structure

```
online-retail-analysis/
│
├── data/
│   └── Online Retail.xlsx
│
├── notebooks/
│   ├── 01_cleaning.ipynb
│   ├── 02_eda.ipynb
│   ├── 03_rfm.ipynb
│   └── 04_clustering.ipynb
│
├── README.md
├── requirements.txt
└── .gitignore
```

| File | What it does |
|---|---|
| `01_cleaning.ipynb` | Business understanding, data audit, cleaning decisions, feature engineering. Saves the clean tables in `data/` |
| `02_eda.ipynb` | Answers the 10 business questions and lists the business insights |
| `03_rfm.ipynb` | Builds one row per customer, calculates RFM scores and groups |
| `04_clustering.ipynb` | Log transform, scaling, K-Means / Hierarchical / DBSCAN, segment profiling, recommendations, limitations, conclusion |


## How to run

1. Install the packages:
```
pip install -r requirements.txt
```
2. Put `Online Retail.xlsx` in the `data/` folder.
3. Run the notebooks **in order** (01, 02, 03, 04). Each one reads the files saved by the one before it.

Or only recreate the data files from the terminal:
```
python analysis.py
```

## Main cleaning decisions

- **Missing CustomerID:** these are whole guest invoices, not random gaps. I kept them for sales, product, time and country analysis, and left them out of RFM and clustering.
- **Returns (invoices starting with C):** not deleted. Sales and returns are kept in separate tables, and net revenue = sales + returns.
- **Zero / negative prices and non-product codes (POST, BANK CHARGES...):** moved to a separate `odd` table, because they are adjustments and not real sales.
- **Duplicates:** exact duplicate rows were removed. This is the only real deletion.
- **Outliers:** not removed, because big wholesale buyers are real customers. I only apply a log transform to a copy of the data when needed.

## Method

- **RFM:** Recency (days since last purchase), Frequency (unique invoices), Monetary (net revenue). Each one is scored 1 to 5 with `pd.qcut`. For Recency, 5 means most recent.
- **Clustering:** log transform and StandardScaler on R, F, M, then K-Means.
- **Choosing k:** silhouette score and elbow method, plus the business need for a manageable number of groups. I chose k = 4 (silhouette about 0.34).
- **Other algorithms:** Hierarchical clustering on a sample of 500 customers gave a similar silhouette (0.33). DBSCAN found 2 clusters and 80 outliers (1.9%), so it is used to spot extreme customers, not to build the segments.

## Main findings

The exact numbers are in the notebooks (`02_eda` section 7 and `04_clustering` section 6). In short:

- Revenue is concentrated in a small part of the products and a small part of the customers.
- The UK is the main market, and sales are strongest in the last months of the year.
- Sales happen on weekdays, mostly around midday.
- There are signs of bulk / wholesale buying.
- The 4 customer segments each get a different action, for example rewarding the best customers and sending win-back offers to valuable customers who stopped buying.

## Limitations

- Only about one year of data, so I cannot confirm the seasonality repeats. December 2011 is incomplete.
- Guest customers are not in the segmentation.
- The data is descriptive, so I cannot make causal claims.
- There is no cost or profit data, only revenue.
- The silhouette score is moderate, so the segments overlap a bit.

## Requirements

Python 3 and the packages in `requirements.txt` (pandas, numpy, matplotlib, seaborn, scikit-learn, scipy, openpyxl, jupyter).
