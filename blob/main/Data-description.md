# Data Netezza description
> Data is generated and does not relate stock market situation

`INVESTMENTS` database `EQUITY_TRANSACTIONS` schema  
Contains information on stock market trading for 2019 - 2025 for different exchanges and stocks.

![ERD](attachments/2025-04-25-10-34-28-pasted-vscode.png)

## 📁 `fact_transactions` — Central fact table that logs all stock transactions
Total number of transactions is set to 1 200 000, we consider prices to be in one currency (USD) and will not calculate any conversions or exchange rate effects.

| Column Name       | Type           | Description                                                  | Possible Values / Notes                        |
|-------------------|----------------|--------------------------------------------------------------|------------------------------------------------|
| `transaction_id`  | INT            | Unique identifier for each transaction                      | Auto-increment or generated                    |
| `account_id`      | INT            | Reference to the `dim_account` table                        | Foreign key                                     |
| `stock_id`        | INT            | Reference to the `dim_stock` table                          | Foreign key                                     |
| `exchange_id`     | INT            | Reference to the `dim_exchange` table                       | Foreign key                                     |
| `date_id`         | INT            | Reference to the `dim_date` table                           | Foreign key                                     |
| `order_type`| VARCHAR(10)    | Type of transaction                                         | `'BUY'`, `'SELL'`                               |
| `quantity`          | INT            | Number of shares traded                                     | Positive integers                               |
| `price`           | DECIMAL(10,2)  | Price per share at time of transaction                      | Positive decimals                               |
| `total_value`     | DECIMAL(18,2)  | Calculated total value (`quantity * price`)                  | Auto-calculated                                 |

## 👤 `dim_account` — Account metadata including customer and trading details
Customers might have several accounts opened, in current dataset we have 123 000 customers and 246 301 accounts.

| Column Name         | Type           | Description                                                | Possible Values / Notes                      |
|---------------------|----------------|------------------------------------------------------------|----------------------------------------------|
| `account_id`        | INT            | Unique identifier for the account                         | Primary key                                   |
| `account_type`      | VARCHAR(50)    | Type of account                                           | `'Retail'`, `'Institutional'`, `'Margin'`     |
| `status`            | VARCHAR(20)    | Current status of the account                             | `'Active'`, `'Suspended'`, `'Closed'`         |
| `opening_date`      | DATE           | Date the account was opened                               | Past dates                                    |
| `risk_level`        | VARCHAR(10)    | Assigned risk profile                                     | `'Low'`, `'Medium'`, `'High'`                 |
| `balance`           | DECIMAL(18,2)  | Current account balance                                   | `$1,000 - $1,000,000`                          |
| `margin_enabled`    | BOOLEAN        | Indicates if margin trading is enabled                    | `True`, `False`                               |
| `trading_experience`| VARCHAR(20)    | User’s self-assessed trading experience                   | `'Beginner'`, `'Intermediate'`, `'Expert'`    |

## 📈 `dim_stock` — Information about individual stocks
Most of the stocks apart from four (BAC, IBM, AMZN, HD) that we will be used in the subsequent Labs. The stock symbols expect for the four mentioned are synthetic data (non existing companies), there are 100 fake companies.

| Column Name     | Type           | Description                                            | Possible Values / Notes            |
|------------------|----------------|--------------------------------------------------------|------------------------------------|
| `stock_id`       | INT            | Unique ID for each stock                              | Primary key                         |
| `stock_symbol`   | VARCHAR(10)    | Stock ticker symbol                                   | `'YATE'`, `'IBM'`, etc.             |
| `stock_name`     | VARCHAR(255)   | Full name of the stock/company                        | `'Yates-Rhodes.'`, etc.                |
| `sector`         | VARCHAR(100)   | Broad sector classification                           | `'Technology'`, `'Healthcare'`, etc.|
| `industry`       | VARCHAR(100)   | Specific industry                                     | `'Software'`, `'Banking'`, etc.     |
| `market_cap`     | DECIMAL(18,2)  | Market capitalization in USD                          | `$1B`, `$100M`, etc.                |

## 📅 `dim_date` — Date dimension for querying by different date granularity

In our current (full) data set we have dates from 1.1.2019 till 31.3.2025.

| Column Name        | Type         | Description                                    | Possible Values / Notes            |
|--------------------|--------------|------------------------------------------------|------------------------------------|
| `date_id`          | INT          | Surrogate key for the date                    | Primary key                        |
| `date` | DATE         | Actual date of the transaction execution                                   | `'2024-04-01'`, etc.               |
| `year`             | INT          | Year part                                     | `2024`, etc.                       |
| `quarter`          | INT          | Quarter of the year                           | `1`, `2`, `3`, `4`                  |
| `month`            | INT          | Month of the year                             | `1` to `12`                         |
| `day_of_week`              | INT          | Day of the month                              | `0` to `6`                         |
| `is_weekend`          | VARCHAR(15)  | If the day is a weekend                               | `True`, `False`, etc.      |

## 🏛️ `dim_exchange` — Stock exchange details
Information about stock exchanges were transactions were carries out -> information will be used later to calculate tax liability based on exchange country.
| Column Name     | Type           | Description                                      | Possible Values / Notes               |
|------------------|----------------|--------------------------------------------------|---------------------------------------|
| `exchange_id`    | INT            | Unique ID for the exchange                      | Primary key                            |
| `exchange_name`  | VARCHAR(100)   | Name of the stock exchange                      | `'NYSE'`, `'NASDAQ'`, `'SGX'`         |
| `country`        | VARCHAR(50)    | Country where the exchange is located           | `'USA'`, `'Singapore'`, `'UK'`, etc.  |
| `timezone`       | VARCHAR(10)    | Local timezone of the exchange                  | `'EST'`, `'GMT'`, `'SGT'`, etc.       |
| `currency`       | VARCHAR(10)    | Default trading currency                        | `'USD'`, `'SGD'`, `'GBP'`, etc.       |
