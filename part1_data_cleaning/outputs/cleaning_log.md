# Cleaning Log — Order Data Cleaning Project

## 1. List of Issues Found

- **Text formatting:** Inconsistent capitalization (e.g. `"PRIYA MENON"`, `"mira das"`) and extra/leading/trailing/double spaces (e.g. `"Vikram  Iyer"`) across `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, and `order_status`.
- **Date formats:** `order_date` and `ship_date` were stored in four different formats within the same column: `MM/DD/YYYY`, `DD-MM-YYYY`, `YYYY-MM-DD`, and `DD Mon YYYY` (e.g. `21 Jul 2024`).
- **Duplicate records:** 31 `order_id` values appeared more than once (63 raw rows total belonged to a repeated order_id). 20 distinct rows were exact full-row duplicates and were removed (one order_id, `ORD-2024-10124`, had 3 copies where only 2 were identical, making it both an exact-duplicate case and a conflict case). After removing the 20 exact duplicates, 24 rows across 12 `order_id`s still had conflicting values in other columns and were flagged for review.
- **Missing values:** `region` (26 rows), `ship_mode` (22 rows), and `discount` (18 rows) had blank entries.
- **Invalid discounts:** 16 rows had negative discount values; 8 rows stored discount as percent text (e.g. `"70%"`, `"85%"`) which, once converted, exceeded the valid range observed elsewhere in the data (0–0.65).
- **Date logic errors:** 22 rows had a `ship_date` earlier than the `order_date`.
- **Order/payment status mix:** 146 Cancelled orders, 164 Returned orders, 72 Refunded payments, and 69 Failed payments need separate handling in summary reporting.
- **Calculation mismatches:** 65 rows where `sales` did not equal `quantity × unit_price × (1 − discount)`; 66 rows where `profit` did not equal `calculated_sales − cost`.

## 2. Cleaning Actions Performed

- Applied `TRIM` + `PROPER` to all text fields to remove extra spacing and standardize casing.
- Parsed all four date formats into true Excel dates using a format-detection formula (checking for `/`, position of `-`, or falling back to `DATEVALUE` for named-month text), then validated with `IFERROR`.
- Removed the 21 redundant exact-duplicate rows, keeping the first occurrence of each.
- Flagged (not deleted) the 22 rows belonging to the 11 conflicting `order_id` groups for manual business review.
- Filled missing `region` and `ship_mode` values with `"Unknown"`.
- Created a `cleaned_discount` column: missing → `0`; negative → absolute value; percent-text → divided by 100.
- Recalculated `calculated_sales` and `calculated_profit` directly from `quantity`, `unit_price`, `cleaned_discount`, and `cost` rather than trusting the original `sales`/`profit` columns.
- Added `profit_margin`, `shipping_delay_days`, `order_month`, `order_year`, and a composite `data_quality_flag` column.

## 3. Business Rules Applied

- Missing `region` / `ship_mode` → filled as `"Unknown"` and flagged as a Warning.
- Missing `discount` → treated as `0`, applied only because all other sales fields (quantity, unit_price, sales, cost, profit) were present and valid for those 18 rows.
- Negative discount → flagged as Invalid (value still corrected to its absolute value for calculation purposes, but the row remains marked Invalid for review).
- Discount above the allowed range (>0.65 after standardizing) → flagged as Invalid.
- Cancelled orders and Failed payments → excluded from final completed-sales totals in the pivot summary.
- Refunded orders → summarized separately rather than folded into completed sales.
- Ship date earlier than order date → flagged as an Invalid shipping record.

## 4. Assumptions Made

- Negative discounts were assumed to be sign-entry errors and corrected via absolute value, rather than treated as a different rule (e.g. a surcharge) — but they are still flagged as Invalid so the assumption is visible and reviewable, not hidden.
- The valid discount ceiling (0.65) was inferred empirically from the highest legitimate numeric discount values present elsewhere in the dataset, since no explicit business maximum was provided.
- For percent-text discounts (e.g. `"70%"`), the converted decimal was used for calculation, but since 0.70/0.85 exceed the inferred 0.65 ceiling, these are still flagged Invalid rather than silently accepted.
- Rows with conflicting `order_id` duplicates were left in the dataset rather than auto-merged or auto-deleted, since the correct resolution requires business judgment not available from the data alone.

## 5. Records Removed

- 20 rows removed: exact full-row duplicates (one of which, `ORD-2024-10124`, had 3 total copies where only 2 were identical).

## 6. Records Flagged

- 24 rows flagged as "Conflict – Review" (12 `order_id`s with differing field values across their duplicate entries).
- 21 rows flagged as Invalid for ship date earlier than order date.
- 16 rows flagged as Invalid for originally negative discounts.
- 8 rows flagged as Invalid for discount above the allowed range.
- 203 rows flagged as Warning (filled "Unknown" region/ship_mode, Failed payment, or Cancelled order status).
- Final tally: **643 Clean**, **203 Warning**, **66 Invalid** records out of 912 total (after removing the 20 exact-duplicate rows from the original 932).

## 7. Limitations of This Cleaning Process

- The 0.65 discount ceiling is an inferred threshold, not a stated business rule — if the actual company policy differs, the Invalid discount counts and flags would need to be revised.
- Conflicting duplicate records were flagged but not resolved; a human decision is still required to determine which version of each conflicting record is correct.
- Date parsing relies on positional pattern-matching (separator type and position) rather than a true locale-aware parser, so any date format outside the four identified patterns would not be handled correctly.
- Calculation mismatches between original and recalculated `sales`/`profit` were identified but the original values were not altered in place — analysis should use the `calculated_sales`/`calculated_profit` columns rather than the raw `sales`/`profit` columns going forward.
