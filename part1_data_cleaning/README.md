# Part 1: Order Data Cleaning Project

## 1. Problem Summary

This project cleans and validates a 932-row export of retail order data pulled from multiple internal systems. The raw export contained inconsistent text formatting, four different date formats mixed within the same columns, duplicate and conflicting records, missing values, invalid discount entries, and mismatches between recorded and recalculated sales/profit figures. The goal was to produce an analysis-ready dataset, a data quality report, and summary pivot tables suitable for business review.

## 2. Dataset Description

- **Source file:** `data/raw_orders.xlsx`
- **Size:** 932 order records, 21 columns
- **Fields:** order identifiers and dates, customer details (name, segment), location (region, state, city), product details (category, sub-category, product name), shipping info (ship mode, ship date), financials (quantity, unit price, discount, sales, cost, profit), and status fields (payment status, order status)

## 3. Tools Used

- Microsoft Excel (formulas: `TRIM`, `PROPER`, `DATE`, `DATEVALUE`, `IFERROR`, `COUNTIF`/`COUNTIFS`, `TEXTJOIN`, nested `IF`, PivotTables, Calculated Fields)

## 4. Cleaning Steps Performed

1. Preserved the original file untouched; all work performed in a separate `cleaned_orders.xlsx`.
2. Standardized all text fields using `TRIM` + `PROPER` to remove spacing inconsistencies and normalize casing.
3. Parsed `order_date` and `ship_date` out of four mixed formats (`MM/DD/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`, `DD Mon YYYY`) into consistent Excel date values.
4. Identified and removed 21 exact duplicate rows; flagged (without deleting) 22 rows belonging to 11 `order_id`s with conflicting data.
5. Filled missing `region`/`ship_mode` values with `"Unknown"` and flagged them.
6. Standardized `discount` into a new `cleaned_discount` column, correcting missing, negative, and percent-text values, while still flagging out-of-range entries as Invalid.
7. Recalculated `calculated_sales`, `calculated_profit`, and `profit_margin` directly from base fields rather than trusting the original `sales`/`profit` columns.
8. Added `shipping_delay_days`, `order_month`, `order_year`, and a composite `data_quality_flag` (Clean / Warning / Invalid).

Full details are in `outputs/cleaning_log.md`.

## 5. Business Rules Applied

- Missing `region`/`ship_mode` → filled `"Unknown"`, flagged as Warning.
- Missing `discount` → treated as `0` (only where all other sales fields were valid).
- Negative or out-of-range discount → flagged as Invalid.
- Cancelled orders and Failed payments → excluded from completed-sales totals.
- Refunded orders → summarized separately.
- Ship date earlier than order date → flagged as an Invalid shipping record.

## 6. Summary of Data Quality Issues Found

| Category | Count |
|---|---|
| Missing region | 26 |
| Missing ship_mode | 22 |
| Missing discount | 18 |
| Negative discounts | 16 |
| Percent-text discounts | 8 |
| Discounts above allowed range | 8 |
| Exact duplicate rows removed | 20 |
| Conflicting duplicate rows flagged | 24 |
| Ship date before order date | 21 |
| Sales calculation mismatches | 64 |
| Profit calculation mismatches | 64 |
| **Final: Clean / Warning / Invalid (of 912 rows)** | **643 / 203 / 66** |

## 7. Summary of Final Pivot Reports

- **Sales & profit by region:** South leads in total sales (~₹15.5L), but East has the highest profit margin (~29.9%) and West the second-highest (~28.6%) — South's higher revenue carries a noticeably thinner margin (~24.4%).
- **Sales & profit by category:** Technology and Furniture both contribute strongly; full category-by-sub-category sales and profit are broken out in `pivot_summary.xlsx`.
- **Profit margin by segment:** Corporate customers carry the highest profit margin (~29.2%), followed closely by Home Office (~28.9%), then Small Business (~27.5%); Consumer is lowest (~26.4%).
- **Order count by ship mode:** Fairly evenly split across Standard Class (242), First Class (223), Second Class (222), and Same Day (204), with 21 orders missing a ship mode (filled as "Unknown").
- **Refunded/cancelled/failed orders by region:** North has the highest count of problem orders (88 combined), followed by South (75); West and East are lower (57–58).
- **Monthly sales trend:** Sales fluctuate month to month across the 22-month window with no single consistent seasonal pattern; see `pivot_summary.xlsx` sheet 6 for the full month-by-month breakdown.

## 8. Key Business Insights

- Roughly **22% of all orders (203 of 912)** carry a data-quality Warning (mostly missing region/ship_mode or failed/cancelled status) and another **7% (66 orders)** are flagged Invalid — meaning nearly 3 in 10 orders need some form of review before being trusted at face value.
- The original `sales` and `profit` columns disagree with a from-scratch recalculation in roughly **7% of rows (64 each)**, which is high enough that downstream reporting should rely on the recalculated `calculated_sales`/`calculated_profit` fields rather than the raw export values.
- Corporate is the most profitable customer segment by margin, while East and West regions run the leanest, highest-margin operations — together suggesting the business's strongest margin lever is Corporate-segment sales in those regions.
- North region has the most refund/cancellation/failure activity, which may be worth investigating operationally (fulfillment, payment processing, or regional demand mismatch).

## 9. Assumptions and Limitations

- The 0.65 discount ceiling used to flag "above allowed range" discounts was inferred from the data itself, not a stated policy — should be confirmed with the business.
- Conflicting duplicate `order_id`s were flagged, not resolved; a human decision is still needed on which version of each record is authoritative.
- Date parsing is pattern-based (by separator and position) and only covers the four formats found in this dataset.

## 10. Screenshots Included

See `screenshots/raw_data_preview.png`, `screenshots/cleaned_data_preview.png`, `screenshots/pivot_summary_1.png`, and `screenshots/pivot_summary_2.png` for visual confirmation of the raw vs. cleaned data and the pivot outputs.
