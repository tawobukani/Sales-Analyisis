# Sales-Analyisis
A professional repository for sales analysis. Includes scripts, queries, statistical modeling and interactive dashboards designed to evaluate performance, identify trends, and support data-driven business decisions.

### Data Source
Primary dataset for this analysis is "retail_store_sales.csv" file containing detailed information sales.
[Link](https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning)

### Tools
- Python -Data cleaning
- SQL - Data Analysis
- R - Statistical Analysis
- Power BI - Creating reports

### Data cleaning
In the initial data preparation phase: The following steps are perfomed 
1. Data loading and inspection
2. handle missing values handle missing values
3. Data cleaning and formatting
4. Export data to csv file 

```Python
import numpy as np
import pandas as pd
df = pd.read_csv("retail_store_sales.csv")
df.head()
df.tail()
df.info()
df.describe()
df.describe(include="all")df.isnull().sum()
df["Item"] = df.groupby("Category")["Item"].transform(lambda x: x.fillna(x.mode()[0] if not x.mode().empty else "Unknown"))
df["Price Per Unit"] = df["Price Per Unit"].fillna(df["Price Per Unit"].mean())
df["Quantity"] = df["Quantity"].fillna(df["Quantity"].mean())
df["Total Spent"] = df["Total Spent"].fillna(df["Price Per Unit"] * df["Quantity"])
df["Discount Applied"] = df["Discount Applied"].fillna(df["Total Spent"] < df["Price Per Unit"] * df["Quantity"])
df["Transaction Date"] = pd.to_datetime(df["Transaction Date"])
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(" ","_")
df.head()
df.to_csv("retail_store_sales_cleaned.csv", index=False)
```

### Expainatory Data Analysis 
1. What kinds of products are really driving our sales and profits?
2. Who are our biggest spenders, and how do their shopping habits change over time?
3. How do our online sales stack up against what’s happening in-store?
4. When do people buy the most — are there clear monthly or seasonal peaks?
5. What happens when we run discounts — do they boost sales volume without hurting revenue too much?
6. On average, how much do customers spend in a single transaction, and does it vary by category ?  


### Data Analysis
import retail_store_sales.csv to sql and save as sales
```sql
SELECT category, SUM(total_spent) AS total_revenue
FROM sales
GROUP BY category
ORDER BY total_revenue DESC;
```

```sql
SELECT customer_id, strftime('%Y-%m', transaction_date) AS month, SUM(total_spent) AS monthly_spend
FROM sales
GROUP BY customer_id,month
ORDER BY monthly_spend DESC
LIMIT 5;
```

```sql
select payment_method , sum(total_spent) as total_spend , count(*) as number_of_transaction,
       ROUND(SUM(total_spent) * 100.0 / (SELECT SUM(total_spent) FROM sales), 2) AS revenue_pct
from sales
group by payment_method
order by payment_method desc;
```

```sql
SELECT strftime('%Y-%m', transaction_date) AS month,SUM(total_spent) AS monthly_revenue,COUNT(*) AS transaction_count,
FROM sales
GROUP BY month
ORDER BY monthly_revenue DESC;
```

```sql
SELECT (CASE WHEN discount_applied = true THEN 'yes' ELSE 'no' END) AS discount_applied,
       COUNT(*) AS transaction_count,
       SUM(total_spent) AS total_revenue,
       round(AVG(total_spent),2) AS avg_spend
FROM sales
GROUP BY discount_applied
ORDER BY total_revenue DESC;
```

```sql
SELECT category,ROUND(AVG(total_spent), 2) AS avg_spend
FROM sales
GROUP BY category
ORDER BY avg_spend DESC;
```

