# E-Commerce Marketplace — SQL Portfolio Project

A PostgreSQL database modeling a multi-seller marketplace — think Wildberries or Amazon, but smaller and actually understandable.

Built to demonstrate real data analyst skills: schema design, analytical SQL, and the kind of business questions that come up in e-commerce every day.

---

## What's inside

The database covers the full lifecycle of a marketplace transaction — from a customer browsing a product to a seller getting paid — across 15 tables:

- Customers and sellers with ratings, commission rates, and statuses
- Product catalog with a two-level category tree and SKU-level variants (color, size stored as JSONB)
- Orders with full status history so you can track how long each stage takes
- Payments split by method (card, SBP, BNPL, wallet) and provider
- Seller payouts showing gross revenue, marketplace commission, and net earnings per period
- Warehouses and inventory with real stock movement logs — every receipt, sale, return, and write-off
- Shipments with carrier tracking and delivery event history

---

## Getting started

You'll need PostgreSQL 16+ installed.

```bash
# Create the database
createdb portfolio_db

# Load schema + base data + analytical queries
psql portfolio_db -f ecommerce_marketplace.sql

# Load extended dataset (500 orders, 50 customers)
psql portfolio_db -f ecommerce_seed_extended.sql
```

If you're using pgAdmin: right-click the database → Query Tool → open the file (Cmd+O) → press F5.

---

## Dataset

The extended seed file generates realistic data for October 2024 through March 2025, with a December spike (about 1.8x the usual order volume — seasonal effect).

| Table | Records |
|---|---|
| customers | 60 |
| sellers | 5 |
| products / variants | 12 / 15 |
| orders | ~518 |
| order items | ~646 |
| payments | ~518 |
| shipments | ~455 |
| inventory movements | ~627 |
| seller payouts | 40 |

Order statuses are distributed realistically: ~85% delivered, ~10% cancelled, ~5% returned.

---

## Analytical queries

There are 8 queries in Section 4 of `ecommerce_marketplace.sql`, plus 4 pre-built views you can query directly.

The views:

```sql
-- Daily GMV broken down by category
SELECT * FROM v_gmv_daily WHERE order_date = '2024-12-15';

-- How each seller is performing
SELECT * FROM v_seller_performance ORDER BY gross_revenue DESC;

-- Which products are running low or out of stock
SELECT * FROM v_inventory_status WHERE stock_status != 'ok';

-- How fast each warehouse processes and ships orders
SELECT * FROM v_fulfillment_metrics;
```

The queries cover:

- GMV and take rate by month and category — how much the marketplace earns from each category
- Top sellers by revenue with their share of total GMV
- Payment method breakdown — card vs SBP vs BNPL, with percentage shares using window functions
- Inventory turnover — how many days of stock remain at the current sales pace
- Stock alerts — out-of-stock and low-stock SKUs by warehouse
- Fulfillment grading — Excellent / Good / Average / Poor based on total delivery time
- Repeat purchase rate — what share of customers come back for a second order
- Carrier performance — delivery success rate and average delivery time per carrier

---

## A few things worth noting

The schema uses some deliberate design choices that are worth calling out:

`order_items` stores `unit_price` and `commission_amount` at the time of the order. This matters because prices and commission rates change over time — you need to lock in the values so historical reports stay accurate.

`inventory_movements` is an append-only log with signed `qty_delta` (positive = stock in, negative = stock out). This lets you reconstruct stock levels at any point in time and calculate turnover properly, rather than just looking at a snapshot.

`order_status_history` tracks every status change with a timestamp, not just the current status. This is what makes it possible to calculate how long orders spend in each stage — useful for spotting bottlenecks in fulfillment.

---

## Known limitations

A few things that would exist in a production system but are simplified here:

- No `reviews` table — customer ratings for products and sellers
- Promo codes are stored as a plain string on orders, not linked to a campaigns table
- `inventory.qty_available` doesn't update automatically when orders come in — in production you'd handle this with triggers or application logic
- The views would need to be materialized (with scheduled refreshes) at any meaningful production scale

---

## Files

```
ecommerce_marketplace.sql      — schema, seed data, views, analytical queries
ecommerce_seed_extended.sql    — extended dataset (500 orders)
README.md                      — this file
```
