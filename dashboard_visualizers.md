# Power BI Dashboard Visualizers and DAX Measures

This document describes ten useful visualizations you can build on top of the Supabase DW schema using the dim and fact tables. Each section includes a brief description, the required fields, and example DAX measures.

---

## 1. Map (Bubble or Filled)

**Purpose:** Show sales geographically by seller or customer location.

**Fields:**
- Location: `dim_geography[city]` (or `state_code`/`region_group`)
- Latitude: `dim_geography[latitude]`
- Longitude: `dim_geography[longitude]`
- Size (bubble): custom measure

**DAX Measure – Sales by Seller Geo**
```dax
Sales by Seller Geo =
CALCULATE(
    SUM(fact_sales[gross_item_value]),
    USERELATIONSHIP(
        fact_sales[geo_seller_key],
        dim_geography[geo_key]
    )
)
```

Use the same measure with `geo_customer_key` for customer-based analysis.

---

## 2. Clustered Bar Chart

**Purpose:** Compare sales across cities or sellers.

**Fields:**
- Axis (Y): `dim_geography[city]`
- Values (X): `Sales by Seller Geo` measure
- Legend: optional (`dim_seller[seller_id]`)

---

## 3. Line Chart – Sales Over Time

**Purpose:** Trend of gross sales by day, month or year.

**Fields:**
- Axis: `dim_date[full_date]` (or `year_num`, `month_num`)
- Values: `Total Gross Sales` measure

**DAX Measure – Total Gross Sales**
```dax
Total Gross Sales = SUM(fact_sales[gross_item_value])
```

Add a slicer on `dim_date[year_num]` or `dim_date[month_name]`.

---

## 4. Matrix / Table – Top Products

**Purpose:** Show top-selling products with quantities and revenues.

**Fields:**
- Rows: `dim_product[category_name_en]` / `dim_product[product_id]`
- Values: `Total Quantity`, `Total Revenue`

**DAX Measure – Total Quantity**
```dax
Total Quantity = SUM(fact_sales[item_count])
```

**DAX Measure – Total Revenue**
```dax
Total Revenue = SUM(fact_sales[gross_item_value])
```

Sort by `Total Revenue` descending.

---

## 5. Pie or Donut Chart – Payment Type Breakdown

**Purpose:** Visualize payment method distribution.

**Fields:**
- Legend: `dim_payment_type[payment_type]`
- Values: `Count of Payments` or `Sum of payment_value`

**DAX Measure – Count of Payments**
```dax
Count of Payments = COUNTROWS(fact_payments)
```

Optionally filter by year or order status.

---

## 6. Stacked Column Chart – Orders by Status

**Purpose:** Show how many orders are in each status group over time.

**Fields:**
- Axis: `dim_date[year_num]` or `dim_date[month_num]`
- Legend: `dim_order_status[status_group]`
- Values: `Count of Orders`

**DAX Measure – Count of Orders**
```dax
Count of Orders = DISTINCTCOUNT(fact_sales[order_id])
```

---

## 7. Scatter Chart – Delivery vs Delay

**Purpose:** Analyze delivery performance.

**Fields:**
- X axis: `Average Delivery Days`
- Y axis: `Average Delay Days`
- Details / Size: `dim_seller[seller_id]` or `dim_geography[city]`

**DAX Measure – Average Delivery Days**
```dax
Avg Delivery Days = AVERAGE(fact_sales[delivery_days])
```

**DAX Measure – Average Delay Days**
```dax
Avg Delay Days = AVERAGE(fact_sales[delay_days])
```

---

## 8. Card Visual – Key Metrics

**Purpose:** Display single-value KPIs.

**Examples:**
- Total Sales: `Total Gross Sales` (see measure earlier)
- Total Orders: `Total Orders` measure
- Average Review Score

**DAX Measure – Total Orders**
```dax
Total Orders = DISTINCTCOUNT(fact_sales[order_id])
```

**DAX Measure – Average Review Score**
```dax
Avg Review Score = AVERAGE(fact_reviews[review_score])
```

---

## 9. Line & Clustered Column Combo – Sales vs Payments

**Purpose:** Compare revenue to payment counts over time.

**Fields:**
- Shared Axis: `dim_date[full_date]`
- Column Values: `Total Gross Sales`
- Line Values: `Count of Payments`

Use the `Count of Payments` measure defined previously.

---

## 10. KPI Visual – Customer Churn / Retention

**Purpose:** Show current vs previous period numbers.

**Fields:**
- Indicator measure, Target measure, Trend

**DAX Measures:**
```dax
Customers This Period = DISTINCTCOUNT(fact_sales[customer_key])

Customers Last Period =
CALCULATE(
    [Customers This Period],
    DATEADD(dim_date[full_date], -1, MONTH)
)
```

You can use `CUSTOMERS THIS PERIOD` as indicator and `Customers Last Period` as target with arrow.

---

### Additional Tips

- Use slicers on `dim_date[year_num]`, `dim_geography[state_code]`, and `dim_order_status[status_group]`.
- Establish *Many-to-One* (*:1) relationships from facts to dimensions in the model view.
- Add the geo relationships:
  - `fact_sales[geo_customer_key]` → `dim_geography[geo_key]`
  - `fact_sales[geo_seller_key]` → `dim_geography[geo_key]`

With these visualizers and measures, you have a strong foundation for an insightful dashboard.