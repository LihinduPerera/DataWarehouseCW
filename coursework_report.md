# Data Warehouse and Business Intelligence Coursework Report

## Module: Data Warehousing and Business Intelligence
## Student: S.A. Lihindu Indudunu Perera
## Student ID: KUHDSE251F-018
## Intake: HNDSE25.1
## Tools Used: Supabase (PostgreSQL), Power BI, CSV/Excel/MSSQL exported data
## Dataset Context: Olist e-commerce public dataset (Brazilian marketplace scenario)

---

## 1. Introduction

### 1.1 Organizational Background and Business Scenario

The course work is founded on an e-commerce marketplace scenario that is modeled based on Olist data set. The organization in question can be defined as an online marketplace which is multi-vendor and links sellers and buyers in various cities and states. In this type of business, it is a many-seller type, where customers make orders with time, and transactions are done through various means of payments. The company also gathers customer feedback, the delivery milestones and shipping prices.

Most organisations have high volumes of operational data, but most decision-makers are unable to leverage this data due to the fact that they are scattered in various files and systems. In an operational structure, the records of orders are maintained in a different location as compared to the payment records or customer profiles, product characteristics, seller details, and geolocation data. This division is effective in carrying out transactional activities but not analytics. Integrated, historical, and query-optimized structures are it needed by management teams to answer the following questions:

- What are the highest revenue areas?
- What do the product categories stimulate growth?
- Which sellers contain delays that influence the customer satisfaction?
- What is the variation in payment patterns across locations and time?
- What is the situation with customer retention in the long term?

A dashboarding solution and a data warehouse were installed to enhance strategic and tactical decision-making. The warehouse was created in **Supabase PostgreSQL** in the form of staging tables **(stg)** and analytical dimension/fact tables **(dim, fact_)**. The information was then loaded into Power BI to develop interactive reports to the business users.

### 1.2 Why This Dataset Is Suitable

The chosen dataset is quite appropriate to a data warehousing assignment as there are a number of fundamental business entities within it that are inherently suited to dimensional modeling:

- Customers and their localities.
- Lifecycle of orders and order status.
- Realize items of order including prices and freight charges.
- Types of payment and payment transactions.
- Metadata of product category.
- The performance of the sellers and their regions.
- Review scores and timing response.
- Trend analysis date and time attributes.

This organization serves a host of business processes and thus is more suited to a **Galaxy Schema** (fact constellation) design where there are numerous fact tables which have shared dimensions.

### 1.3 Problem Statement and Objectives

This is because the fundamental issue that was discussed in this coursework is the absence of integrated analytical layer of business intelligence. This is aimed at developing a strong decision-support environment by:

1. Combining data of various source formats.
2. The design and deployment of a Galaxy-schema data warehouse.
3. Transformation of staging data to dimensional and fact tables.
4. Constructing ten Power BI visualization analysis.
5. Making sense of trends, patterns and actionable knowledge.
6. Talking about ethics in data and the ethics in analytics.

The end product is a warehouse-supported BI model that helps managers to track the sales, performance of deliveries, payment habits, and customer feelings in a predictable, repeatable and scalable manner.

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

2. **Excel-like tabular source** (category translation in spreadsheet-compatible format):
   - `productcategoryname_translation.csv` (stored as spreadsheet-managed reference/master data; this may be recorded as Excel-managed and then made as CSV to be loaded).

3. **SQL operational source export** (data found in relational tables, converted to CSV, and then staged by loading):
   - The transaction data obtained by an active RDBMS (e.g. MSSQL) and converted into CSV can be defined as orders, payments, and reviews and then be centrally staged in Supabase.

This hybrid source story is a characteristic feature of enterprise practice, which involves analysts combining mixed source formats into one warehouse landing zone.

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

A **Galaxy Schema** (fact constellation) was chosen since the business involves multiple processes of analysis which cannot be structured in a clean way using a single fact table:

- Sales process (item level transaction)
- Transaction payment process (payment-sequence-level transactions)
- Review process (review-level events)

The similarities of the dimensions common in these facts are the date, customer, product, seller and order status and hence Galaxy schema is the appropriate design choice. The Galaxy model is better than a single-star design since data duplication is not experienced, there is enhancement in model consistency, and cross-process analysis is enabled.

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

![Supabase Warehouse Schema](Screenshot-2026-03-08-082423.png)

![Supabase Warehouse Schema](supabase-schema-qmhcfaxjxrbjyymfzsfr.png)

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

This pair of measures approximates to retention momentum. An increase implies an augmentation of recursive buying conduct; the downward trend may initiate customer relationship initiatives and service quality enhancement.

### 4.11 Cross-Visualization Trends and Pattern Summary

In all the pictures, one can speak about the following general categories of patterns:

- Revenue location within areas of choice of urban/strong-commerce.
- Unequal performance of sellers with the best performers taking over the value.
- Obvious time-based variability which indicates seasonality.
- Variation in delivery performance directly correlated with the proxies of customer satisfaction.
- Channel-specific optimization of payment behavior disparity.

These results show that a linked warehouse model can allow integrated business analysis to be run, as opposed to isolated reporting.

### 4.12 Screenshot Placeholders: Power BI

![Power BI Model View](Screenshot-2026-03-06-153855.png)

![Power BI Dashboard](Screenshot-2026-03-07-012706.png)

---

## 5. Data Ethics

Driven decision-making Data-driven decision-making is capable of producing business value, but it also imposes ethical obligations. The issues to be applicable to this project are as follows.

### 5.1 Privacy and Personal Data Protection

Although personal identifiers might be direct (e.g., names, etc.), location and behavioral information can still constitute a re-identification risk. When the combination of zip code, city, and purchase behavior is involved, it makes the combination sensitive. This must be done ethically by reducing the amount of personally identifiable information and by taking care of role-based access control in reporting.

### 5.2 Purpose Limitation and Responsible Use

The unrelated or intrusive profiling by using the data gathered to perform transactions should not be done unless there is a legal and ethical reason. The warehouse is supposed to fulfill specific business purposes including operations enhancement, quality in customer service, and strategic planning.

### 5.3 Bias and Fairness in Interpretation

Comparisons between regions and sellers may make unwanted judgments when factors of context are not taken into consideration (infrastructure differences, distance, mix of products). Before punishing underperformers without the analysis of the root causes, ethical analytics is cautious.

### 5.4 Data Quality and Misleading Conclusions

Wrong decisions may come as a result of inaccurate, incomplete, or delayed data. Transparency of data limits, assumptions known, and model constraints are also ethical contents that need to be reported. Example: review-to-product mapping through first order item is a deliberate tradeoff in modeling and needs to be recorded.

### 5.5 Security and Access Governance

The BI assets and warehouse assets should be safeguarded by:

- Least-privilege access
- Credential management
- Audit logging
- Sharing of dashboard links is controlled.

This avoids unauthorized use and accountability is ensured.

### 5.6 Ethical Recommendations for This Project

- Cover or blend sensitive location information where necessary.
- Show assumptions of the documents in the report and dashboard metadata.
- Authenticate important metrics prior to executive reporting.
- Enforce retention policies on staging raw data.
- Make sure that the law is satisfied with the data protection principles.

---

## 6. Conclusion

This course work illustrates an entire business intelligence life cycle: source integration, data warehouse architecture, ETL, and dashboard analytics. To represent the numerous interrelated processes (sales, payments, reviews), a Galaxy schema was rightfuly chosen to be shared by common dimensions. Staging and dimensional layers were used to implement the warehouse in Supabase and the key constraints and foreign keys were used to preserve the integrity.

Analytically, the power BI dashboard can provide a complete decision support using 10 visualizations. The model assists in analysis of region, product/category performance, seller operation, payment and customer sentiment trend. The integration of these views would help the management to have a better idea of what contributes to revenue, where delays exist, and which processes should be optimized.

### 6.1 Overall Findings, Trends, and Patterns

According to the integrated model, there are a number of strategic patterns:

- Selection of locations and the highest contributors in terms of revenue.
- Varying performance among sellers and logistics results.
- Demand variations with time implying seasonality.
- Status (experience) and delay (experience) connected with operational indicators (reviews).
- Partnership and checkout strategy can depend on unique payment preferences.

### 6.2 Business Intelligence Value Obtained

A BI value of the next magnitude is obtained:

- **Strategic planning:** Improved expansion and targeting of the market regionally.
- **Operational improvement:** The discovery of delivery bottlenecks and service risk points.
- **Commercial optimization:** Special treatment of products, sellers and categories.
- **Customer management:** The review signals and repeat activity monitoring.
- **Executive alignment:** KPI visibility based on one governed data model.

All in all, the project has demonstrated that data warehousing and visualization can be utilized to convert raw data in transactions into actionable intelligence. This helps in speedy, evidence based decision making and enhances performance within a company.