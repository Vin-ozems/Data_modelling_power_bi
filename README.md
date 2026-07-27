# Retail & E-Commerce Sales Analytics — Power BI Data Model
<img width="957" height="479" alt="image" src="https://github.com/user-attachments/assets/032352a7-87ed-4712-a2ac-b7bfb3148e09" />

A star-schema semantic model built in Power BI for retail/e-commerce reporting, covering sales, inventory, marketing campaigns, order fulfillment, and regional sales targets. The model implements row-level security (RLS) to restrict data access by user region.

## 📐 Model Overview

The model follows a standard star schema pattern: multiple fact tables connected to shared dimension tables, enabling analysis across sales, campaigns, inventory, and order operations from a common set of filters (product, geography, customer, date, campaign).

## 🗂️ Tables

### Dimension Tables

| Table | Key Columns | Description |
|---|---|---|
| **dim_products** | `product_key`, `product_code`, `product_name`, `category`, `subcategory`, `brand`, `primary_supplier`, `unit_price` | Product catalog with pricing and categorization |
| **dim_customers** | `customer_id`, `customer`, `contacts_name`, `contacts_email`, `phone`, `region`, `segment`, `credit_limit`, `payment_terms` | Customer master data |
| **dim_geo** | `city_key`, `city`, `region` | Geographic hierarchy (city → region) |
| **dim_date** | `Date` | Standard date table for time intelligence |
| **dim_campaign** | `campaign_key`, `name`, `channel`, `budget`, `start_date`, `end_date` | Marketing campaign metadata |
| **dim_order_flags** | `flag_key`, `Order_channel`, `channels`, `priority`, `status` | Order channel and priority/status flags |
| **security** | `user_email`, `region` | Maps report users (by email) to their authorized region, used for RLS |

### Fact Tables

| Table | Key Columns | Description |
|---|---|---|
| **fact_sales** | `OrderID`, `LineID`, `order_date`, `product_key`, `ship_to_city_key`, `Quantity`, `unit_price`, `unit_cost`, `line_total` | Core sales transactions at line-item grain |
| **fact_order_process** | `order_id`, `customer_id`, `order_date`, `invoice_date`, `pay_date`, `shipment_date`, `delivery_date` | Order lifecycle timestamps for fulfillment tracking |
| **fact_sales_targets** | `date`, `target_revenue` | Revenue targets over time |
| **sales_targets** | `Period`, `TargetRevenue` | Period-level target revenue (summary/staging) |
| **fact_inventory** | `date`, `product_key`, `unit` | Inventory unit levels over time |
| **fact_campaign_spend** | `campaign_key`, `date`, `clicks`, `impressions`, `spend` | Daily marketing spend and engagement metrics |
| **fact_promotion_coverage** | `campaign_key`, `product_key` | Bridge table mapping which products were covered by which campaigns |

## 🔗 Key Relationships

- `dim_products[product_key]` → `fact_sales`, `fact_inventory`, `fact_promotion_coverage`
- `dim_customers[customer_id]` → `fact_order_process`
- `dim_customers[region]` → `security[region]` (drives RLS)
- `dim_geo[city_key]` → `fact_sales[ship_to_city_key]`
- `dim_date[Date]` → all fact tables with a date column
- `dim_campaign[campaign_key]` → `fact_campaign_spend`, `fact_promotion_coverage`
- `dim_order_flags` → `fact_sales` / `fact_order_process` (order channel & status context)

## 🔒 Row-Level Security (RLS)

The `security` table maps each user's email to a region. A DAX filter is applied on the relevant table(s) to restrict visible rows to the user's assigned region:

```dax
[region] = LOOKUPVALUE(security[region], security[user_email], USERPRINCIPALNAME())
```

This ensures regional sales managers only see data for their own region when the report is published to the Power BI Service.

**Notes / assumptions:**
- Each user in `security` must have exactly one region (LOOKUPVALUE requires a single match).
- If a user needs access to multiple regions, this should be reworked using a bridge table + `TREATAS` instead of `LOOKUPVALUE`.

## 📊 Use Cases

This model supports analysis such as:
- Sales performance vs. targets by region, product, and time period
- Order fulfillment lead time (order → invoice → payment → shipment → delivery)
- Marketing campaign ROI (spend, clicks, impressions vs. sales/promotion coverage)
- Inventory levels by product over time
- Customer segmentation and credit exposure

## 🛠️ Tools

- Power BI Desktop (data modeling, relationships, DAX, RLS)
- Star schema design principles

---
*Model diagram and RLS DAX included in this repo as reference screenshots.*
