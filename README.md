# Data Literacy with Polars: Business Questions to Code

## Setup Instructions

### Prerequisites
- Google Colab 

## Introduction

This class focuses on two critical data literacy concepts:
1. **Translating business questions into code operations**
2. **Understanding data grain (what each row represents)**

## 1. Business Terms → Code Operations

When analyzing data, business questions often contain keywords that map directly to specific operations:

### Common Mappings

| Business Term | Code Operation | Example Question | Polars Code |
|--------------|----------------|------------------|-------------|
| **"total"** | `sum()` | "What is the total revenue?" | `df.select(pl.col("revenue").sum())` |
| **"average"** | `mean()` | "What is the average price?" | `df.select(pl.col("price").mean())` |
| **"number of"** / **"count"** | `len()` or `count()` | "How many customers do we have?" | `df.select(pl.len())` |
| **"by"** / **"for each"** | `group_by()` | "What is the total sales by region?" | `df.group_by("region").agg(pl.col("sales").sum())` |
| **"maximum"** / **"highest"** | `max()` | "What is the highest price?" | `df.select(pl.col("price").max())` |
| **"minimum"** / **"lowest"** | `min()` | "What is the lowest score?" | `df.select(pl.col("score").min())` |
| **"unique"** / **"distinct"** | `n_unique()` | "How many unique products?" | `df.select(pl.col("product").n_unique())` |

### Practice Reading Business Questions

**Question**: "What is the average price for each product category?"
- **"average"** → `mean()`
- **"for each"** → `group_by()`
- **Answer**: `df.group_by("category").agg(pl.col("price").mean())`

## 2. Understanding Data Grain

### What is Grain?

**Grain** refers to what each row in your dataset represents. It's the level of detail of your data.

### Examples of Different Grains

1. **Single Grain (Good)**: Each row represents the same thing
   ```
   Monthly Sales - Each row = one salesperson's monthly total
   | salesperson | month   | sales_amount |
   |-------------|---------|--------------|
   | Alice       | 2024-01 | 15,000       |
   | Alice       | 2024-02 | 18,000       |
   | Bob         | 2024-01 | 12,000       |
   | Bob         | 2024-02 | 14,000       |
   ```

2. **Mixed Grains (Problematic)**: Different rows represent different things
   ```
   Sales Report with Mixed Monthly and Quarterly Rollups
   | salesperson | period     | period_type | sales_amount |
   |-------------|------------|-------------|--------------|
   | Alice       | 2024-01    | Monthly     | 15,000       |
   | Alice       | 2024-02    | Monthly     | 18,000       |
   | Alice       | 2024-03    | Monthly     | 22,000       |
   | Alice       | Q1 2024    | Quarterly   | 55,000       | ← Different grain!
   | Bob         | 2024-01    | Monthly     | 12,000       |
   | Bob         | 2024-02    | Monthly     | 14,000       |
   | Bob         | 2024-03    | Monthly     | 16,000       |
   | Bob         | Q1 2024    | Quarterly   | 42,000       | ← Different grain!
   | **TOTAL**   | Q1 2024    | Company     | 97,000       | ← Yet another grain!
   ```

   **Why mixed grains are problematic:**
   - Can't directly compare rows (monthly vs quarterly)
   - Aggregations will double-count (Q1 includes Jan, Feb, Mar)
   - Filtering becomes complex ("show me all March data" might accidentally include Q1)
   - Violates the principle that each row should represent the same type of entity

### How group_by() Changes Grain

When you use `group_by()`, you **change the grain** of your data:

```python
# Original: Each row = one transaction
original_grain = "one transaction"

# After grouping by customer
df.group_by("customer").agg(pl.col("amount").sum())
# New grain: Each row = one customer
new_grain = "one customer"
```

### Why Grain Matters

1. **Prevents incorrect aggregations**: You can't sum customer ages if each row is a transaction (same customer appears multiple times)
2. **Guides analysis decisions**: Know what level of detail you need for your business question
3. **Helps communicate results**: "This report shows data at the monthly grain" is clearer than "this is monthly data"

### The Golden Rule of Grain

**Before aggregating data, always ask: "What does each row represent?"**

If you're unsure about the grain:
```python
# Check the columns
print(df.columns)

# Look at a few rows
print(df.head())

# Check for unique identifiers
print(f"Number of unique transaction_ids: {df['transaction_id'].n_unique()}")
print(f"Total rows: {len(df)}")
# If these match, transaction_id defines the grain
```

## 3. Business Language to Polars Translation Guide

### Complete Business Phrase Mappings

This section helps translate common business questions and requests into Polars operations.

#### Aggregation Phrases

| Business Language | Polars Operation | Full Example |
|-------------------|------------------|--------------|
| "What's the total..." | `.sum()` | "What's the total revenue?" → `pl.col("revenue").sum()` |
| "Calculate the sum of..." | `.sum()` | "Calculate the sum of all orders" → `pl.col("order_amount").sum()` |
| "Add up all the..." | `.sum()` | "Add up all the sales" → `pl.col("sales").sum()` |
| "What's the average..." | `.mean()` | "What's the average customer age?" → `pl.col("age").mean()` |
| "Find the typical..." | `.mean()` or `.median()` | "Find the typical order size" → `pl.col("order_size").median()` |
| "Count how many..." | `.count()` or `pl.len()` | "Count how many orders we have" → `pl.len()` |
| "Number of unique..." | `.n_unique()` | "Number of unique customers" → `pl.col("customer_id").n_unique()` |
| "How many different..." | `.n_unique()` | "How many different products?" → `pl.col("product").n_unique()` |
| "Find the highest..." | `.max()` | "Find the highest sale amount" → `pl.col("sale_amount").max()` |
| "What's the maximum..." | `.max()` | "What's the maximum discount?" → `pl.col("discount").max()` |
| "Find the lowest..." | `.min()` | "Find the lowest price" → `pl.col("price").min()` |
| "What's the minimum..." | `.min()` | "What's the minimum order quantity?" → `pl.col("quantity").min()` |

#### Grouping Phrases

| Business Language | Polars Operation | Full Example |
|-------------------|------------------|--------------|
| "...by department" | `.group_by("department")` | "Total sales by department" → `df.group_by("department").agg(pl.col("sales").sum())` |
| "...for each region" | `.group_by("region")` | "Average price for each region" → `df.group_by("region").agg(pl.col("price").mean())` |
| "...per customer" | `.group_by("customer")` | "Number of orders per customer" → `df.group_by("customer").agg(pl.len())` |
| "...across categories" | `.group_by("category")` | "Sales trends across categories" → `df.group_by("category").agg(pl.col("sales").sum())` |
| "Break down by..." | `.group_by()` | "Break down revenue by product line" → `df.group_by("product_line").agg(pl.col("revenue").sum())` |
| "Split by..." | `.group_by()` | "Split costs by department" → `df.group_by("department").agg(pl.col("cost").sum())` |
| "...within each..." | `.group_by()` | "Top seller within each store" → `df.group_by("store").agg(pl.col("sales").max())` |

#### Filtering Phrases

| Business Language | Polars Operation | Full Example |
|-------------------|------------------|--------------|
| "Only show..." | `.filter()` | "Only show orders over $100" → `df.filter(pl.col("amount") > 100)` |
| "Exclude..." | `.filter(~...)` | "Exclude cancelled orders" → `df.filter(~pl.col("status").eq("cancelled"))` |
| "Remove..." | `.filter(~...)` | "Remove inactive customers" → `df.filter(~pl.col("active").eq(False))` |
| "Keep only..." | `.filter()` | "Keep only premium customers" → `df.filter(pl.col("tier") == "premium")` |
| "Where..." | `.filter()` | "Where sales exceed target" → `df.filter(pl.col("sales") > pl.col("target"))` |
| "...greater than..." | `> value` | "Revenue greater than 10000" → `pl.col("revenue") > 10000` |
| "...less than..." | `< value` | "Age less than 30" → `pl.col("age") < 30` |
| "...between X and Y" | `.is_between(X, Y)` | "Orders between Jan and Mar" → `pl.col("date").is_between(date(2024,1,1), date(2024,3,31))` |

#### Sorting and Ranking Phrases

| Business Language | Polars Operation | Full Example |
|-------------------|------------------|--------------|
| "Top 10..." | `.sort().head(10)` | "Top 10 customers by revenue" → `df.sort("revenue", descending=True).head(10)` |
| "Bottom 5..." | `.sort().head(5)` | "Bottom 5 performing products" → `df.sort("sales").head(5)` |
| "Rank by..." | `.rank()` | "Rank salespeople by performance" → `df.with_columns(pl.col("sales").rank().alias("rank"))` |
| "Sort from highest..." | `.sort(descending=True)` | "Sort from highest to lowest price" → `df.sort("price", descending=True)` |
| "Order by..." | `.sort()` | "Order by date" → `df.sort("date")` |
| "Find the best..." | `.sort().first()` | "Find the best performing store" → `df.sort("performance", descending=True).first()` |

#### Time-based Phrases

| Business Language | Polars Operation | Full Example |
|-------------------|------------------|--------------|
| "Year-to-date..." | `.filter()` with date logic | "Year-to-date sales" → `df.filter(pl.col("date") >= date(2024,1,1))` |
| "Last 30 days..." | `.filter()` with date math | "Orders in last 30 days" → `df.filter(pl.col("date") >= date.today() - timedelta(days=30))` |
| "Monthly..." | `.group_by_dynamic()` | "Monthly revenue trends" → `df.group_by_dynamic("date", every="1mo").agg(pl.col("revenue").sum())` |
| "Quarterly..." | `.group_by_dynamic()` | "Quarterly performance" → `df.group_by_dynamic("date", every="1q").agg(pl.col("sales").sum())` |
| "Year-over-year..." | Compare with `.shift()` | "Year-over-year growth" → Calculate with shifted data from previous year |

#### Calculation Phrases

| Business Language | Polars Operation | Full Example |
|-------------------|------------------|--------------|
| "Calculate the ratio..." | Column division | "Calculate profit margin" → `pl.col("profit") / pl.col("revenue")` |
| "Percentage of..." | Division with * 100 | "Percentage of total sales" → `(pl.col("sales") / pl.col("sales").sum()) * 100` |
| "Growth rate..." | `(new - old) / old` | "Monthly growth rate" → `(pl.col("current") - pl.col("previous")) / pl.col("previous")` |
| "Difference between..." | Subtraction | "Difference between actual and target" → `pl.col("actual") - pl.col("target")` |

### Complex Business Questions Example

**Business Question**: "Show me the top 5 products by revenue for each region, but only for products that sold more than 100 units last quarter"

**Translation Process**:
1. "top 5" → `.sort().head(5)`
2. "by revenue" → `.sort("revenue", descending=True)`
3. "for each region" → `.group_by("region")`
4. "sold more than 100 units" → `.filter(pl.col("units_sold") > 100)`
5. "last quarter" → `.filter()` with appropriate date range

**Polars Code**:
```python
(df
 .filter(pl.col("date").is_between(date(2024,1,1), date(2024,3,31)))
 .filter(pl.col("units_sold") > 100)
 .group_by("region")
 .agg([
     pl.col("product"),
     pl.col("revenue")
 ])
 .explode(["product", "revenue"])
 .sort(["region", "revenue"], descending=[False, True])
 .group_by("region")
 .head(5))
```

## Summary

1. **Business questions contain keywords** that map to specific operations
2. **Grain is what each row represents** in your dataset
3. **group_by() changes the grain** to whatever columns you group by
4. **Always know your grain** before performing aggregations

Remember: Good data analysis starts with understanding what your data represents and translating business needs into the right operations.