# Optimizing the Netezza Data Warehouse Cost

**Solution to the business questions**

1. Calculate top 10 accounts by the volume of traded per year.

```sql
-- Top 10 accounts by total volume traded (quantity) per year
SELECT da.account_id, dd.year, SUM(ft.quantity) AS total_quantity
FROM (
    SELECT account_id, date_id, quantity FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.fact_transactions
    UNION ALL
    SELECT account_id, date_id, quantity FROM nz_catalog.equity_transactions_ly.fact_transactions
) ft
JOIN (
    SELECT account_id FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_account
    UNION ALL
    SELECT account_id FROM nz_catalog.equity_transactions_ly.dim_account
) da ON ft.account_id = da.account_id
JOIN (
    SELECT date_id, year FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_date
    UNION ALL
    SELECT date_id, year FROM nz_catalog.equity_transactions_ly.dim_date
) dd ON ft.date_id = dd.date_id
GROUP BY da.account_id, dd.year
ORDER BY total_quantity DESC
LIMIT 10;
```
2. Identify the Top 10 accounts by transaction value per year.

```sql
-- Top 10 accounts by total value per year
SELECT da.account_id, dd.year, SUM(ft.total_value) AS total_value
FROM (
    SELECT account_id, date_id, total_value FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.fact_transactions
    UNION ALL
    SELECT account_id, date_id, total_value FROM nz_catalog.equity_transactions_ly.fact_transactions
) ft
JOIN (
    SELECT account_id FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_account
    UNION ALL
    SELECT account_id FROM nz_catalog.equity_transactions_ly.dim_account
) da ON ft.account_id = da.account_id
JOIN (
    SELECT date_id, year FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_date
    UNION ALL
    SELECT date_id, year FROM nz_catalog.equity_transactions_ly.dim_date
) dd ON ft.date_id = dd.date_id
GROUP BY da.account_id, dd.year
ORDER BY total_value DESC
LIMIT 10;

```
3. Determine the Average transaction price for each of the stocks, including current year (2025) trades.

```sql
SELECT ds.stock_id, AVG(ft.price) AS avg_price
FROM (
    SELECT stock_id, date_id, price FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.fact_transactions
    UNION ALL
    SELECT stock_id, date_id, price FROM nz_catalog.equity_transactions_ly.fact_transactions
) ft
JOIN (
    SELECT stock_id FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_stock
    UNION ALL
    SELECT stock_id FROM nz_catalog.equity_transactions_ly.dim_stock
) ds ON ft.stock_id = ds.stock_id
JOIN (
    SELECT date_id, year FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_date
    UNION ALL
    SELECT date_id, year FROM nz_catalog.equity_transactions_ly.dim_date
) dd ON ft.date_id = dd.date_id
GROUP BY ds.stock_id
ORDER BY ds.stock_id;
```
4. Determine the Number of transactions the took place in each of exchange by year.

```sql
SELECT de.exchange_name, dd.year, COUNT(ft.transaction_id) AS transaction_count
FROM (
    SELECT exchange_id, transaction_id, date_id FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.fact_transactions
    UNION ALL
    SELECT exchange_id, transaction_id, date_id FROM nz_catalog.equity_transactions_ly.fact_transactions
) ft
JOIN (
    SELECT exchange_id, exchange_name FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_exchange
    UNION ALL
    SELECT exchange_id, exchange_name FROM nz_catalog.equity_transactions_ly.dim_exchange
) de ON ft.exchange_id = de.exchange_id
JOIN (
    SELECT date_id, year FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_date
    UNION ALL
    SELECT date_id, year FROM nz_catalog.equity_transactions_ly.dim_date
) dd ON ft.date_id = dd.date_id
GROUP BY de.exchange_name, dd.year
ORDER BY de.exchange_name, dd.year;
```

5. List all of the stocks traded by account_id, 215 during the year 2024 and 2025.

```sql
SELECT DISTINCT ds.stock_symbol
FROM (
    SELECT stock_id, account_id, date_id FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.fact_transactions
    UNION ALL
    SELECT stock_id, account_id, date_id FROM nz_catalog.equity_transactions_ly.fact_transactions
) ft
JOIN (
    SELECT stock_id, stock_symbol FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_stock
    UNION ALL
    SELECT stock_id, stock_symbol FROM nz_catalog.equity_transactions_ly.dim_stock
) ds ON ft.stock_id = ds.stock_id
JOIN (
    SELECT date_id, year FROM iceberg_data.<SCHEMA_DWH_OFFLOAD>.dim_date
    UNION ALL
    SELECT date_id, year FROM nz_catalog.equity_transactions_ly.dim_date
) dd ON ft.date_id = dd.date_id
WHERE ft.account_id = 215 AND dd.year BETWEEN 2024 AND 2025;
```

