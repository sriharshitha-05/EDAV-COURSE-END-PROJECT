# Sample Superstore Dataset

Order-line transactions of a fictional US office-products retailer, covering **3 January 2014 to 30 December 2017**. Each row is one product on one order, with customer, location, product, sales, quantity, discount and profit. It is a common practice dataset for retail analytics, dashboards and EDA.

> The file does not state its origin. It matches the well-known "Sample - Superstore" dataset distributed with Tableau; cite the source you downloaded it from.

## At a glance

| Item | Value |
|---|---|
| File | `Sample__Superstore.csv` (UTF-8, comma separated) |
| Rows / columns | 9,994 / 19 |
| Orders | 5,009 unique `Order ID`s |
| Customers | 793 |
| Products | 1,862 `Product ID`s (1,850 distinct names) |
| Geography | United States only: 49 states (incl. District of Columbia), 531 cities, 4 regions |
| Order dates | 2014-01-03 to 2017-12-30 (last ship date 2018-01-05) |
| Missing values / duplicate rows | 0 / 0 |
| Total sales / total profit | 2,297,200.86 / 286,397.02 (profit margin about 12.5%) |

## Columns

| Column | Type | Description |
|---|---|---|
| `Row ID` | Integer | Row number, 1 to 9,994 |
| `Order ID` | Text | Order identifier, e.g. `CA-2016-152156`; one order has several rows |
| `Order Date` | Text | Date the order was placed (see **Date format** below) |
| `Ship Date` | Text | Date the order was shipped (see **Date format** below) |
| `Ship Mode` | Text | Standard Class, Second Class, First Class, Same Day |
| `Customer ID` | Text | Customer identifier, e.g. `CG-12520` |
| `Segment` | Text | Consumer, Corporate, Home Office |
| `Country` | Text | Always "United States" |
| `City` | Text | Customer city (531 values) |
| `State` | Text | Customer state (49 values) |
| `Region` | Text | Central, East, South, West |
| `Product ID` | Text | Product identifier, e.g. `FUR-BO-10001798` (prefix = category and sub-category) |
| `Category` | Text | Furniture, Office Supplies, Technology |
| `Sub-Category` | Text | 17 values, e.g. Bookcases, Chairs, Phones, Binders |
| `Product Name` | Text | Product description |
| `Sales` | Number | Line revenue (mean 229.86, median 54.49, max 22,638.48); currency not stated, presumably USD |
| `Quantity` | Integer | Units on the line (1 to 14, mean 3.79) |
| `Discount` | Number | Discount fraction, 0 to 0.8; 0 on 4,798 lines, 0.2 on 3,657 lines |
| `Profit` | Number | Line profit; negative means a loss (min and max are about -6,600 and 8,400) |

## Things to know before analysing

1. **Mixed date formats.** `Order Date` and `Ship Date` use two layouts in the same column: `M/D/YYYY` (for example `4/15/2017`) and `MM-DD-YYYY` with dashes (for example `11-08-2016`). Both are **month first**; treating the dashed dates as day first produces 1,708 ship dates earlier than order dates, while month-first gives none. Parse them explicitly:

   ```python
   import pandas as pd

   def parse(s):
       dash = s.str.contains('-')
       slash = pd.to_datetime(s.where(~dash), format='%m/%d/%Y')
       return slash.fillna(pd.to_datetime(s.where(dash), format='%m-%d-%Y'))

   df = pd.read_csv('Sample__Superstore.csv')
   df['Order Date'] = parse(df['Order Date'])
   df['Ship Date'] = parse(df['Ship Date'])
   df['Ship Days'] = (df['Ship Date'] - df['Order Date']).dt.days   # 0 to 7
   ```

2. **Order lines, not orders.** `Sales`, `Profit` and `Quantity` are per line. Group by `Order ID` for order-level views.
3. **No postal code column.** Location goes down to city only.
4. **Single country.** `Country` has one value and carries no information.
5. **Product ID versus name.** There are more Product IDs than distinct names (1,862 against 1,850), so a few names map to several IDs. Use `Product ID` as the key.
6. **Losses are common.** 1,871 lines (18.7%) have negative profit, and 98.9% of them carry a discount of 20% or more. No line sold at full price (discount 0) made a loss. Discount and profit are negatively related (Spearman -0.54), so analyse them together.
7. **Uneven growth.** Lines per year: 1,993 (2014), 2,102 (2015), 2,587 (2016), 3,312 (2017); sales rose from about 484k to 733k.

## Quick facts

| Category | Sales | Profit |
|---|---|---|
| Technology | 836,154 | 145,455 |
| Furniture | 741,000 | 18,451 |
| Office Supplies | 719,047 | 122,491 |

- **Regions by sales:** West 725k, East 679k, Central 501k, South 392k.
- **Segments by sales:** Consumer 1.16M, Corporate 706k, Home Office 430k.
- **Loss-making sub-categories (total profit):** Tables -17,725; Bookcases -3,473; Supplies -1,189.

## Ideas for analysis

- Sales and profit trends by month, quarter and year; seasonality.
- Profitability by category, sub-category, region and state; why Tables lose money.
- Effect of discount on profit and on quantity sold.
- Shipping performance by ship mode (days between order and ship).
- Customer segmentation (RFM) and repeat purchase behaviour.
- Maps of sales and profit by state; Pareto analysis of products and customers.
- Forecasting monthly sales.

## Loading the data

```python
import pandas as pd
df = pd.read_csv('Sample__Superstore.csv')
print(df.shape)          # (9994, 19)
print(df.dtypes)
```
