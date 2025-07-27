# Data Literacy with Polars: Business Questions to Code

## Setup Instructions

### Prerequisites
- Python 3.12 or higher
- Visual Studio Code
- uv (Python package manager) - Install from [astral.sh/uv](https://astral.sh/uv)

### Step 1: Clone the Repository
**Option A: Using GitHub Desktop (Recommended)**
1. Open GitHub Desktop
2. Click "Clone a repository from the Internet"
3. Enter the repository URL
4. Choose your local folder and click "Clone"

**Option B: Using Command Line**
```bash
git clone <repository-url>
cd opan5510-class5
```

### Step 2: Open in VS Code
Navigate to the project folder and open it in VS Code:
- In GitHub Desktop: Click "Open in Visual Studio Code"
- Or open VS Code and use File → Open Folder

### Step 3: Create Virtual Environment and Install Dependencies
Open a terminal in VS Code (Terminal → New Terminal) and run:
```bash
# Create virtual environment and install dependencies in one step
uv sync

# This creates a .venv folder and installs all packages from pyproject.toml
```

### Step 4: Install VS Code Extensions
Open VS Code and install these extensions:
1. **Jupyter** - For running notebook files
   - Extension ID: `ms-toolsai.jupyter`
2. **Data Wrangler** - For viewing dataframes interactively
   - Extension ID: `ms-toolsai.datawrangler`

### Step 5: Verify Installation
1. Open `class_example.ipynb` in VS Code
2. When prompted to select a kernel, choose the Python interpreter from `.venv` (should show `.venv` in the path)
3. Run the first cell to verify everything works - you should see "Dataset loaded successfully!"

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

1. **Transaction-level grain**: Each row = one transaction
   ```
   | transaction_id | customer | product | amount |
   |----------------|----------|---------|--------|
   | 1              | Alice    | Laptop  | 1200   |
   | 2              | Bob      | Mouse   | 25     |
   ```

2. **Customer-level grain**: Each row = one customer
   ```
   | customer | total_purchases | avg_amount |
   |----------|-----------------|------------|
   | Alice    | 5               | 240        |
   | Bob      | 3               | 180        |
   ```

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

## Summary

1. **Business questions contain keywords** that map to specific operations
2. **Grain is what each row represents** in your dataset
3. **group_by() changes the grain** to whatever columns you group by
4. **Always know your grain** before performing aggregations

Remember: Good data analysis starts with understanding what your data represents and translating business needs into the right operations.