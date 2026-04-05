# Sales-Analyisis
A professional repository for sales analysis. Includes scripts, queries, and interactive dashboards designed to evaluate performance, identify trends, and support data-driven business decisions.

### Data Source
Primary dataset for this analysis is "retail_store_sales.csv" file containing detailed information sales.
[Link](https://www.kaggle.com/datasets/ahmedmohamed2003/retail-store-sales-dirty-for-data-cleaning)

### Tools
- Python -Data cleaning
- SQL - Data Analysis
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
df.shape
df.info()
df.describe()
df.describe(include="all")df.isnull().sum()
df["Item"] = df.groupby("Category")["Item"].transform(lambda x: x.fillna(x.mode()[0] if not x.mode().empty else "Unknown"))
df["Price Per Unit"] = df["Price Per Unit"].fillna(df["Price Per Unit"].mean())
df["Quantity"] = df["Quantity"].fillna(df["Quantity"].mean())
df["Total Spent"] = df["Total Spent"].fillna(df["Price Per Unit"] * df["Quantity"])
df["Discount Applied"] = df["Discount Applied"].fillna(df["Total Spent"] < df["Price Per Unit"] * df["Quantity"])
df["Transaction Date"] = pd.to_datetime(df["Transaction Date"])
df["month_name"] = df["Transaction Date"].dt.strftime("%B")
df = df.rename(columns={"Location": "shopping"})
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(" ","_")
df.head()
df.to_csv("retail_store_sales_cleaned.csv", index=False)
```

### Expainatory Data Analysis 
1. Which category are really driving our sales and profits?
2. Who are our biggest 3 customers, and how do their shopping habits change over time?
3. Which payment method is used the most and how much spent?
4. When do people buy the most — are there clear monthly or seasonal peaks?
5. What happens when we run discounts — do they boost sales volume without hurting revenue too much?
6. Which items sell the most by quantity?
7. Which shopping chanell is currently the most utilized by consumers?

### Data Analysis
import retail_store_sales.csv to sql and save as sales
```sql
SELECT category, round(SUM(total_spent),2) AS total_sales,
round(SUM(total_spent - (price_per_unit * quantity)),2)AS approx_profit
FROM sales
GROUP BY category
ORDER BY total_sales DESC
limit 3;
```

```sql
SELECT customer_id,month_name, round(SUM(total_spent),2) AS monthly_spent
FROM sales
GROUP BY customer_id, month_name
ORDER BY monthly_spent desc
limit 3;
```

```sql
SELECT payment_method,COUNT(*) AS usage_count,round(SUM(total_spent),2) AS total_spend
FROM sales
GROUP BY payment_method
ORDER BY usage_count DESC, total_spend DESC;
```

```sql
SELECT month_name, round(SUM(total_spent),2) AS total_sales, round(SUM(quantity),0) AS total_units
FROM sales
GROUP BY month_name
ORDER BY total_sales DESC;
```

```sql
SELECT (CASE WHEN discount_applied = TRUE THEN 'yes' ELSE 'no' END) AS discount_applied,
COUNT(*) AS transaction_count,round(SUM(total_spent),2) AS total_revenue,ROUND(AVG(total_spent), 2) AS avg_spend
FROM sales
GROUP BY discount_applied
order by total_revenue desc;
```

```sql
SELECT item,round(SUM(quantity),2) AS total_units,round(SUM(total_spent),2) AS total_revenue
FROM sales
GROUP BY item, category
ORDER BY total_units DESC
limit 5;
```

```sql
SELECT shopping,COUNT(*) AS usage_count,round(SUM(total_spent),2) AS total_sales,
ROUND(AVG(total_spent), 2) AS avg_transaction_value
FROM sales
GROUP BY shopping
ORDER BY usage_count DESC, total_sales DESC;
```


### Results
1. Butchers and Electric Household Essentials are the biggest revenue drivers. Beverages also contribute strongly, but mostly through high volume, lower value purchases.
2. Our top customers 3 CUST_03, CUST_12, and CUST_08 show peak activity in January, suggesting strong seasonal loyalty and a surge in spending at the start of the year.
3. Cash is still the most used, slightly ahead of Digital Wallet and Credit Card. However, the gap is small, showing that customers are comfortable with multiple payment options.
4. January is our strongest month, followed by July and December. These peaks align with holiday seasons and mid-year shopping cycles.
5. Discounts clearly boost transaction counts and units sold. Importantly, average spend per transaction remains almost identical to non-discounted sales — meaning discounts increase volume without eroding revenue.
6. Food, Milk, and Patisserie items dominate in terms of quantity sold. These staples are essential for driving basket size and repeat purchases.
7. Online shopping slightly outpaces In-store in both transactions and revenue. Consumers are leaning toward digital channels, though physical stores remain important.

### Recommendations
1. Double down on Butchers and Household Essentials: These categories are our revenue engines. Marketing and promotions should highlight them.
2. Protect and grow staples (Food, Milk, Patisserie): These items drive volume. Ensuring availability and competitive pricing will keep customers coming back.
3. Engage top customers in January: Loyalty programs or personalized offers for CUST_03, CUST_12, and CUST_08 can lock in their seasonal spending.
4. Encourage Digital Wallet adoption: It’s nearly tied with Cash and Credit Card, but aligns better with the growth of online shopping. Incentives could accelerate adoption.
5. Plan around seasonal peaks: Stock up and run campaigns in January, July, and December to maximize demand.
6. Use discounts strategically: Since they boost volume without hurting revenue, apply them to high-volume categories where they’ll have the biggest impact.
7. Invest in Online experience: With digital channels slightly ahead, improving delivery, promotions, and user experience will strengthen this advantage.


### Limitations
- The Item column had about 9% missing values, which is higher than the usual 5% threshold where dropping rows would have been acceptable. Removing them would have meant losing too much information and risked skewing the analysis. To keep the dataset intact, I filled those gaps using the mode (the most common item) within each category. The downside is that this can over‑represent popular items and reduce the variety in the data.
- For the numerical columns — Price Per Unit, Quantity, and Total Spent — I chose to fill missing values with the mean because the data appeared to be normally distributed. Using the mean keeps the averages consistent and avoids biasing the overall results. However, this approach can be sensitive to outliers. If the data had been skewed, the median would have been a safer choice.
