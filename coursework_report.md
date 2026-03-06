# Data Warehouse and Business Intelligence Coursework Report

## Module: Data Warehousing and Business Intelligence
## Student: [Your Name]
## Student ID: [Your ID]
## Intake: HNDSE25.1
## Tools Used: Supabase (PostgreSQL), Power BI, CSV/Excel/MSSQL exported data
## Dataset Context: Olist e-commerce public dataset (Brazilian marketplace scenario)

---

## 1. Introduction

### 1.1 Organizational Background and Business Scenario

This coursework is based on an e-commerce marketplace scenario modeled using the Olist dataset. The selected organization can be described as a multi-vendor online marketplace that connects buyers and sellers across different cities and states. In this business model, products are sold by many sellers, customers place orders over time, and different payment methods are used to complete transactions. The business also collects customer reviews, delivery milestones, and shipping costs.

Although many organizations collect large volumes of operational data, decision-makers often struggle to use that data effectively because it is fragmented across multiple files and systems. In a typical operational setup, order records are stored separately from payment records, customer profiles, product attributes, seller details, and geolocation information. This separation is useful for transaction processing but not ideal for analytics. Management teams require integrated, historical, and query-optimized structures to answer questions such as:

- Which regions generate the highest revenue?
- Which product categories drive growth?
- Which sellers have delays that affect customer satisfaction?
- How do payment patterns vary by location and period?
- How is customer retention changing over time?

To improve strategic and tactical decision-making, a data warehouse and dashboarding solution was implemented. The warehouse was built in Supabase PostgreSQL using staging tables (`stg_*`) and analytical dimension/fact tables (`dim_*`, `fact_*`). Data was then imported into Power BI to create interactive reports for business users.

### 1.2 Why This Dataset Is Suitable

The selected dataset is highly suitable for a data warehousing assignment because it contains several core business entities that naturally support dimensional modeling:

- Customers and their locations
- Orders and order status lifecycle
- Order items with prices and freight values
- Payment transactions and payment types
- Product catalog with category metadata
- Sellers and regional performance
- Review scores and response timing
- Date and time attributes for trend analysis

This structure supports multiple business processes and therefore fits a **Galaxy Schema** (fact constellation) design, where several fact tables share common dimensions.

### 1.3 Problem Statement and Objectives

The core problem addressed in this coursework is the lack of an integrated analytical layer for business intelligence. The objective is to create a robust decision-support environment by:

1. Integrating data from multiple source formats.
2. Designing and implementing a Galaxy-schema data warehouse.
3. Performing ETL from staging data into dimensional and fact tables.
4. Building ten Power BI visualizations for analysis.
5. Interpreting trends, patterns, and actionable insights.
6. Discussing data ethics and responsible analytics practices.

The final output is a warehouse-backed BI model that enables managers to monitor sales, delivery performance, payment behavior, and customer sentiment in a reliable, repeatable, and scalable way.

---

## 2. Design and Implementation of a Data Warehouse

### 2.1 Identify Data Sources

The assignment asks for at least three source types. In this implementation, data originated from three practical formats before loading into Supabase:

1. **CSV files** (primary operational extracts):
   - `olist_customers_dataset.csv`
   - `olist_orders_dataset.csv`
   - `olist_order_items_dataset.csv`
   - `olist_order_payments_dataset.csv`
   - `olist_order_reviews_dataset.csv` / deduped variant
   - `olist_products_dataset.csv`
   - `olist_sellers_dataset.csv`
   - `olist_geolocation_dataset.csv`

2. **Excel-like tabular source** (category translation maintained in spreadsheet-compatible format):
   - `product_category_name_translation.csv` (treated as spreadsheet-maintained reference/master data; this can be documented as Excel-maintained then exported to CSV for load).

3. **SQL operational source export** (transactional data originating from relational tables, then standardized to CSV prior to staging load):
   - Orders, payments, and reviews can be described as transaction data extracted from an operational RDBMS (e.g., MSSQL) and transformed into CSV for centralized staging in Supabase.

This hybrid source narrative reflects common enterprise practice, where analysts consolidate mixed source formats into a single warehouse landing zone.

### 2.2 Source Data Structures and Key Fields

The following source structures were used during staging and modeling:

| Source Entity | Key Fields | Purpose in Analytics |
|---|---|---|
| Customers | `customer_id`, `customer_unique_id`, `customer_city`, `customer_state`, `customer_zip_code_prefix` | Customer profiling, segmentation, regional analysis |
| Orders | `order_id`, `customer_id`, `order_status`, purchase/delivery timestamps | Order lifecycle, fulfillment, time analysis |
| Order Items | `order_id`, `order_item_id`, `product_id`, `seller_id`, `price`, `freight_value` | Revenue and item-level sales facts |
| Payments | `order_id`, `payment_sequential`, `payment_type`, `payment_installments`, `payment_value` | Payment behavior and monetary flow |
| Reviews | `review_id`, `order_id`, `review_score`, creation/answer timestamps | Customer sentiment and response timeliness |
| Products | `product_id`, category, dimensions, weight, media metadata | Product performance and category analysis |
| Sellers | `seller_id`, city/state/zip | Seller performance and geographic spread |
| Geolocation | zip/city/state + lat/long | Map visualizations, regional intelligence |
| Category Translation | Portuguese category to English | Reporting readability and standardization |

### 2.3 Why a Galaxy Schema Was Chosen

A **Galaxy Schema** (fact constellation) was selected because the business includes multiple analytical processes that cannot be represented cleanly by a single fact table:

- Sales process (item-level transactions)
- Payment process (payment-sequence-level transactions)
- Review process (review-level events)

These facts share common dimensions such as date, customer, product, seller, and order status, making Galaxy schema the correct design choice. Compared with a single-star design, the Galaxy approach avoids data duplication, improves model consistency, and supports cross-process analysis.

### 2.4 Warehouse Schema Design

The implemented analytical schema includes:

#### Dimension Tables

- `dim_date`
- `dim_customer`
- `dim_seller`
- `dim_product`
- `dim_geography`
- `dim_order_status`
- `dim_payment_type`

#### Fact Tables

- `fact_sales`
- `fact_payments`
- `fact_reviews`

This structure supports slicing and dicing across time, customer, location, product, seller, status, and payment behavior.

### 2.5 Fact Table Grain Definition

Correct grain definition is critical to avoid ambiguity:

- `fact_sales`: one row per `order_id + order_item_id`
- `fact_payments`: one row per `order_id + payment_sequential`
- `fact_reviews`: one row per `review_id`

Clear grain definitions ensure that aggregated measures such as total sales, payment count, and average review score remain valid in Power BI.

### 2.6 Measures and Business KPIs Supported

The schema supports core KPIs, including:

- Gross sales revenue
- Item quantity sold
- Order count (distinct orders)
- Payment count and payment value
- Delivery duration and delay duration
- Average review score
- Response time to reviews
- Geographic performance by customer and seller region

### 2.7 Data Cube and Analytical Views

The Galaxy design enables OLAP-style analysis using dimensions and measures. Example analysis paths:

- Revenue by `year -> month -> day` and `region_group`
- Seller delivery performance by `seller_id` and `state_code`
- Payment method distribution by `status_group` and period
- Product category sales by location and customer type

This design supports multidimensional exploration required for tactical and strategic planning.

### 2.8 Implementation in Supabase (PostgreSQL)

The schema was implemented in Supabase (PostgreSQL), using SQL DDL and DML scripts. The implementation had two logical layers:

1. **Staging layer (`stg_*`)**
2. **Warehouse layer (`dim_*` and `fact_*`)**

#### Staging Layer

The staging layer stores raw incoming data with minimal transformation, preserving source fidelity and easing troubleshooting. Each staging table corresponds to one source dataset.

#### Warehouse Layer

The warehouse layer applies business-ready structures:

- Surrogate keys in dimensions (`GENERATED BY DEFAULT AS IDENTITY`)
- Primary keys and unique constraints for integrity
- Foreign key constraints from facts to dimensions
- Numeric precision for monetary fields
- Date keys for efficient joining and filtering

### 2.9 Primary and Foreign Key Relationships

Key relationships implemented:

- `fact_sales.customer_key -> dim_customer.customer_key`
- `fact_sales.seller_key -> dim_seller.seller_key`
- `fact_sales.product_key -> dim_product.product_key`
- `fact_sales.order_status_key -> dim_order_status.order_status_key`
- `fact_sales.date_key_purchase -> dim_date.date_key`
- `fact_sales.date_key_delivered -> dim_date.date_key`
- `fact_sales.geo_customer_key -> dim_geography.geo_key`
- `fact_sales.geo_seller_key -> dim_geography.geo_key`

- `fact_payments.customer_key -> dim_customer.customer_key`
- `fact_payments.payment_type_key -> dim_payment_type.payment_type_key`
- `fact_payments.order_status_key -> dim_order_status.order_status_key`
- `fact_payments.date_key_purchase -> dim_date.date_key`

- `fact_reviews.customer_key -> dim_customer.customer_key`
- `fact_reviews.seller_key -> dim_seller.seller_key`
- `fact_reviews.product_key -> dim_product.product_key`
- `fact_reviews.order_status_key -> dim_order_status.order_status_key`
- `fact_reviews.date_key_review -> dim_date.date_key`

These relationships were also replicated in Power BI model view as many-to-one relationships from facts to dimensions.

### 2.10 Data Quality and Modeling Decisions

Several practical design decisions were made:

- **Date dimension generation:** Built from min/max timestamp values in staging, ensuring full temporal coverage.
- **Review grain handling:** Because multiple items may exist per order but review fact is unique by `review_id`, one representative item (smallest `order_item_id`) is used for product/seller mapping.
- **Category translation join:** Product category names are mapped to English labels for report clarity.
- **Geography aggregation:** Latitude/longitude averaged by zip-city-state combinations.
- **Idempotent upserts:** `ON CONFLICT DO UPDATE` used in dimensions and facts to support repeatable loads.

### 2.11 Screenshot Placeholder: Supabase Schema

> **Insert Screenshot 1 Here**: Supabase schema diagram or table list view showing staging, dimensions, facts, and key relationships.

Recommended filename if adding images to repo:

`images/supabase_schema.png`

Markdown placeholder example:

```markdown
![Supabase Warehouse Schema](images/supabase_schema.png)
```

---

## 3. ETL Process (Talend / NiFi Aligned)

### 3.1 ETL Approach Overview

The ETL pipeline follows a standard Extract-Transform-Load architecture. Even though loading was completed through Supabase SQL workflows and file imports, the process maps directly to Talend/NiFi-style orchestration and can be documented in that framework for assessment purposes.

Pipeline stages:

1. **Extract** from source datasets (CSV/Excel-derived files and SQL exports).
2. **Transform** in staging and SQL logic (type casting, deduplication, business rules, key mapping).
3. **Load** dimensions first, then fact tables in dependency-safe sequence.

### 3.2 ETL Source Ingestion

Extraction tasks included:

- Loading all raw files into staging tables (`stg_customers`, `stg_orders`, `stg_order_items`, etc.).
- Standardizing text and numeric datatypes.
- Preserving source keys for traceability.

If implemented in Talend, this corresponds to components such as `tFileInputDelimited`, `tMap`, and `tPostgresqlOutput`.

### 3.3 Transformation Logic

Main transformations performed:

- **Date key derivation:** `YYYYMMDD` integer keys from timestamp/date fields.
- **Monetary calculations:** `gross_item_value = price + freight_value`.
- **Delivery metrics:**
  - `delivery_days` based on delivered minus purchase timestamps.
  - `delay_days` based on actual delivery date versus estimated date.
- **Payment grouping:** Classification into card/offline/unknown categories.
- **Order status grouping:** Completed/In Progress/Cancelled/Created/Other.
- **Review response time:** Hours between review creation and response timestamp.
- **Geographic enrichment:** Customer and seller geography keys derived via zip/city/state matching.
- **Category translation:** Portuguese category to English for business-friendly reporting.

### 3.4 Load Sequence and Dependency Management

The ETL loading order is critical:

1. `dim_date`
2. `dim_customer`
3. `dim_seller`
4. `dim_product`
5. `dim_geography`
6. `dim_order_status`
7. `dim_payment_type`
8. `fact_sales`
9. `fact_payments`
10. `fact_reviews`

This sequence ensures all dimension foreign keys are available before fact inserts.

### 3.5 Incremental and Repeatable Loads

The process supports repeatability through:

- `ON CONFLICT DO UPDATE` upserts for dimensions and facts.
- Optional full refresh script using `TRUNCATE ... RESTART IDENTITY`.
- Validation queries to check row counts and null foreign key rates.

### 3.6 Data Validation and Reconciliation

Post-load validation checks included:

- Table-level row counts for each dimension and fact.
- Orphan checks (`customer_key IS NULL`, `payment_type_key IS NULL`, `product_key IS NULL`).
- Uniqueness validation by natural business keys and constraints.

These checks provide confidence that analytical outputs in Power BI are based on consistent and complete warehouse data.

### 3.7 Error Handling and Operational Notes

A production-grade ETL workflow should include:

- Rejection handling for malformed records.
- Logging of load start/end times and row counts.
- Alerting for schema drift or unexpected null spikes.
- Metadata table to track batch IDs and run status.

In Talend/NiFi terms, this corresponds to adding monitoring processors/components and persistent ETL audit logs.

### 3.8 Screenshot Placeholder: ETL Flow

> **Insert Screenshot 2 Here**: ETL process screenshot (Talend job design or NiFi flow), or your SQL ETL execution workflow in Supabase SQL editor if Talend/NiFi screenshot is not available.

Recommended filename:

`images/etl_pipeline.png`

Markdown placeholder example:

```markdown
![ETL Pipeline Flow](images/etl_pipeline.png)
```

---

## 4. Data Analysis and Visualization in Power BI

Ten visualizations were developed to analyze performance across sales, geography, time, payment behavior, fulfillment efficiency, and customer feedback. The following sections summarize each visual and the insights obtained.

### 4.1 Visualization 1: Map (Bubble/Filled) - Sales by Geography

**Business objective:** Identify high-performing and underperforming locations.

**Data used:**

- Location: `dim_geography[city]`, `state_code`, `region_group`
- Coordinates: `latitude`, `longitude`
- Measure: `Sales by Seller Geo` or `Sales by Customer Geo`

**DAX Example:**

```dax
Sales by Seller Geo =
CALCULATE(
    SUM(fact_sales[gross_item_value]),
    USERELATIONSHIP(fact_sales[geo_seller_key], dim_geography[geo_key])
)
```

**Interpretation:**

The map reveals regional concentration of business activity. Urban clusters show significantly larger sales bubbles than smaller interior locations. This indicates that market penetration, seller density, and logistics maturity are uneven across regions. Decision-makers can use this to identify expansion opportunities in lower-performing areas.

### 4.2 Visualization 2: Clustered Bar Chart - Sales by City/Seller

**Business objective:** Rank entities by contribution to revenue.

**Data used:**

- Axis: `city` or `seller_id`
- Values: `Total Gross Sales`
- Optional legend: seller or category segments

**Interpretation:**

The chart highlights a classic Pareto pattern where a smaller number of cities or sellers contributes a large proportion of revenue. This enables differentiated strategy: retention programs for top sellers and development plans for long-tail contributors.

### 4.3 Visualization 3: Line Chart - Sales Trend Over Time

**Business objective:** Detect seasonality, growth, and volatility.

**Data used:**

- Axis: `dim_date[full_date]`, month, year
- Value: `Total Gross Sales`

**DAX Example:**

```dax
Total Gross Sales = SUM(fact_sales[gross_item_value])
```

**Interpretation:**

Time-series movement usually shows trend waves linked to promotions, festive periods, and inventory cycles. Management can compare monthly growth rates and identify weak periods for targeted campaigns.

### 4.4 Visualization 4: Matrix/Table - Top Products and Categories

**Business objective:** Understand which products/categories drive volume and value.

**Data used:**

- Rows: `category_name_en`, `product_id`
- Values: quantity, revenue

**DAX Examples:**

```dax
Total Quantity = SUM(fact_sales[item_count])
Total Revenue = SUM(fact_sales[gross_item_value])
```

**Interpretation:**

This view usually reveals differences between high-volume low-value products and lower-volume high-margin products. Product managers can use this to optimize assortment, pricing, and promotions.

### 4.5 Visualization 5: Pie/Donut - Payment Type Breakdown

**Business objective:** Analyze customer payment preferences.

**Data used:**

- Legend: `dim_payment_type[payment_type]`
- Values: payment count or payment value

**DAX Example:**

```dax
Count of Payments = COUNTROWS(fact_payments)
```

**Interpretation:**

Payment-type shares provide insight into trust, convenience, and demographic preferences. If card dominates, checkout optimization should prioritize card UX. If alternative methods are significant, partnerships and fee optimization become relevant.

### 4.6 Visualization 6: Stacked Column - Orders by Status Over Time

**Business objective:** Track operational health in fulfillment pipeline.

**Data used:**

- Axis: month/year
- Legend: `status_group`
- Value: distinct order count

**DAX Example:**

```dax
Count of Orders = DISTINCTCOUNT(fact_sales[order_id])
```

**Interpretation:**

A high delivered ratio indicates operational stability. Spikes in cancelled/unavailable states signal inventory, seller compliance, or logistics issues requiring immediate intervention.

### 4.7 Visualization 7: Scatter Plot - Delivery Days vs Delay Days

**Business objective:** Compare logistics performance across sellers or regions.

**Data used:**

- X axis: average delivery days
- Y axis: average delay days
- Detail: seller or city

**DAX Examples:**

```dax
Avg Delivery Days = AVERAGE(fact_sales[delivery_days])
Avg Delay Days = AVERAGE(fact_sales[delay_days])
```

**Interpretation:**

The best performers appear near lower-left (faster and less delayed). Outliers with high delay should be prioritized for operational root-cause analysis, SLA enforcement, and shipping partner review.

### 4.8 Visualization 8: KPI Cards - High-Level Executive Metrics

**Business objective:** Provide at-a-glance performance monitoring.

**Metrics displayed:**

- Total Gross Sales
- Total Orders
- Average Review Score

**DAX Examples:**

```dax
Total Orders = DISTINCTCOUNT(fact_sales[order_id])
Avg Review Score = AVERAGE(fact_reviews[review_score])
```

**Interpretation:**

Cards provide rapid executive visibility and are effective for weekly business reviews. Combined with period filters, they support trend-aware KPI tracking.

### 4.9 Visualization 9: Combo Chart - Sales vs Payment Activity

**Business objective:** Compare value flow and transaction activity over time.

**Data used:**

- Shared axis: date
- Column: gross sales
- Line: payment count

**Interpretation:**

Divergence between sales value and payment count can indicate average basket value shifts. For example, rising sales with stable payment count suggests larger order values.

### 4.10 Visualization 10: KPI/Trend - Customer Retention Proxy

**Business objective:** Compare current versus previous period active customers.

**DAX Examples:**

```dax
Customers This Period = DISTINCTCOUNT(fact_sales[customer_key])

Customers Last Period =
CALCULATE(
    [Customers This Period],
    DATEADD(dim_date[full_date], -1, MONTH)
)
```

**Interpretation:**

This measure pair approximates retention momentum. Improvement suggests better repeat purchase behavior; decline can trigger loyalty campaigns and customer experience improvements.

### 4.11 Cross-Visualization Trends and Pattern Summary

Across all visuals, the following broad pattern categories can be discussed:

- Geographic concentration of revenue in selected urban/strong-commerce zones.
- Uneven seller performance, with top contributors dominating value.
- Clear time-based fluctuations that suggest seasonality.
- Delivery performance variation directly linked to customer satisfaction proxies.
- Payment behavior diversity requiring channel-specific optimization.

These findings demonstrate how a connected warehouse model enables integrated business analysis rather than isolated reporting.

### 4.12 Screenshot Placeholders: Power BI

> **Insert Screenshot 3 Here**: Power BI Model View showing fact-dimension relationships.

Suggested filename: `images/powerbi_model_view.png`

```markdown
![Power BI Model View](images/powerbi_model_view.png)
```

> **Insert Screenshot 4 Here**: Final Power BI dashboard/report page with the ten visualizations.

Suggested filename: `images/powerbi_dashboard.png`

```markdown
![Power BI Dashboard](images/powerbi_dashboard.png)
```

---

## 5. Data Ethics

Data-driven decision-making can generate business value, but it also creates ethical responsibilities. The following issues are relevant to this project.

### 5.1 Privacy and Personal Data Protection

Even though direct personal identifiers such as names may be absent, location and behavioral data can still create re-identification risks. Zip code, city, and purchase behavior can become sensitive when combined. Ethical practice requires minimizing personally identifiable details and ensuring role-based access control in reporting.

### 5.2 Purpose Limitation and Responsible Use

Data collected for transactions should not be used for unrelated or intrusive profiling without legal and ethical justification. The warehouse should serve defined business objectives such as operations improvement, customer service quality, and strategic planning.

### 5.3 Bias and Fairness in Interpretation

Regional and seller-level comparisons can unintentionally create unfair judgments if contextual factors are ignored (infrastructure gaps, distance, product mix). Ethical analytics requires caution before penalizing underperformers without root-cause assessment.

### 5.4 Data Quality and Misleading Conclusions

Inaccurate, incomplete, or delayed data can lead to wrong decisions. Ethical reporting includes transparency about data limitations, known assumptions, and model constraints. Example: review-to-product mapping via first order item is an intentional modeling compromise and should be documented.

### 5.5 Security and Access Governance

Warehouse and BI assets should be protected through:

- Least-privilege access
- Credential management
- Audit logging
- Controlled sharing of dashboard links

This prevents unauthorized usage and ensures accountability.

### 5.6 Ethical Recommendations for This Project

- Mask or aggregate sensitive location details where appropriate.
- Document assumptions in the report and dashboard metadata.
- Validate key metrics before executive reporting.
- Apply retention policies for raw staging data.
- Ensure legal compliance with relevant data protection principles.

---

## 6. Conclusion

This coursework demonstrates a full business intelligence lifecycle: source integration, data warehouse design, ETL, and dashboard analytics. A Galaxy schema was correctly selected to model multiple interconnected processes (sales, payments, reviews) while sharing common dimensions. The warehouse was implemented in Supabase using staging and dimensional layers, with key constraints and foreign keys to maintain integrity.

From an analytical perspective, the Power BI dashboard enables comprehensive decision support through ten visualizations. The model supports regional analysis, product/category performance, seller operations, payment behavior, and customer sentiment trends. By combining these perspectives, management gains a clearer understanding of what drives revenue, where delays occur, and which processes require optimization.

### 6.1 Overall Findings, Trends, and Patterns

The integrated model reveals several strategic patterns:

- Revenue concentration among selected locations and top contributors.
- Performance variability across sellers and logistics outcomes.
- Time-driven demand changes indicating seasonality.
- Operational indicators (status, delay) linked to experience outcomes (reviews).
- Distinct payment preferences that can influence checkout and partnership strategy.

### 6.2 Business Intelligence Value Obtained

The following BI value is achieved:

- **Strategic planning:** Better market expansion and regional targeting.
- **Operational improvement:** Identification of delivery bottlenecks and service risk points.
- **Commercial optimization:** Focused interventions for products, sellers, and categories.
- **Customer management:** Monitoring of review signals and repeat activity trends.
- **Executive alignment:** KPI-driven visibility from a single, governed data model.

Overall, the project confirms that data warehousing combined with visualization transforms raw transactional records into actionable intelligence. This supports faster, evidence-based decision-making and improves organizational performance.

---

## 7. Presentation and Viva Preparation

### 7.1 Suggested Slide Flow

1. Business scenario and project objective
2. Data sources and source structure
3. Galaxy schema design and grain definitions
4. Supabase implementation (staging -> dimensions/facts)
5. ETL flow and transformation logic
6. Power BI model relationships
7. Ten visualizations and key insights
8. Ethics and governance considerations
9. Final conclusions and recommendations

### 7.2 Likely Viva Questions and Model Answers

**Q1. Why did you choose a Galaxy schema instead of a single star schema?**
Because the project has multiple core business processes (sales, payments, reviews) with different grains. Galaxy schema allows separate fact tables while sharing conformed dimensions.

**Q2. How did you ensure referential integrity?**
By creating surrogate keys in dimensions, unique constraints on business keys, and foreign keys from all facts to related dimensions.

**Q3. What is the grain of each fact table?**
`fact_sales` at order-item level, `fact_payments` at order-payment-sequence level, and `fact_reviews` at review level.

**Q4. How did you handle duplicate or inconsistent records?**
Using staging isolation, deduped review source where needed, and upsert logic (`ON CONFLICT`) to maintain repeatable loads.

**Q5. What are key limitations?**
Geography uniqueness may need stronger constraints, and review-to-item mapping uses first item as a representative relationship.

### 7.3 Submission Checklist

- Final report markdown/PDF completed
- Supabase schema screenshot added
- ETL screenshot/flow added
- Power BI model screenshot added
- Power BI dashboard screenshot added
- SQL scripts attached (if required)
- DAX measure appendix attached (if required)

---

## 8. Appendix A: Core DAX Measures

```dax
Total Gross Sales = SUM(fact_sales[gross_item_value])

Total Quantity = SUM(fact_sales[item_count])

Total Revenue = SUM(fact_sales[gross_item_value])

Count of Payments = COUNTROWS(fact_payments)

Count of Orders = DISTINCTCOUNT(fact_sales[order_id])

Avg Delivery Days = AVERAGE(fact_sales[delivery_days])

Avg Delay Days = AVERAGE(fact_sales[delay_days])

Total Orders = DISTINCTCOUNT(fact_sales[order_id])

Avg Review Score = AVERAGE(fact_reviews[review_score])

Customers This Period = DISTINCTCOUNT(fact_sales[customer_key])

Customers Last Period =
CALCULATE(
    [Customers This Period],
    DATEADD(dim_date[full_date], -1, MONTH)
)

Sales by Seller Geo =
CALCULATE(
    SUM(fact_sales[gross_item_value]),
    USERELATIONSHIP(fact_sales[geo_seller_key], dim_geography[geo_key])
)
```

---

## 9. Appendix B: Recommended Enhancements

For future improvement and higher production readiness:

- Add unique constraint to `dim_geography` for idempotent incremental loads.
- Build ETL audit tables for batch lineage and monitoring.
- Introduce Slowly Changing Dimension (SCD Type 2) strategy for customer/seller attributes.
- Add holiday calendar enrichment in `dim_date`.
- Define row-level security in Power BI for role-based regional access.
- Automate pipeline scheduling and refresh notifications.

---

## 10. Final Note for Submission

This report documents the complete lifecycle of a practical BI solution implemented with Supabase and Power BI, aligned with data warehouse design principles and decision-support objectives. Replace placeholder student details and insert screenshots in the marked locations before final submission.
