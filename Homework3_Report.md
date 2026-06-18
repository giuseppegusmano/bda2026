# Homework #3 - Financial Transactions
## Star Schema and Interactive Dashboard - Brief Report

## 1. Modeling choices

**Business process and grain.** The fact table models financial transactions at the grain of *one row per transaction* from the account statement.

**Star schema.** One fact table (`Fact_Transactions`) and four dimensions (`Dim_Time`, `Dim_Geography`, `Dim_Symbol`, `Dim_TransactionType`). The schema is kept as a pure star: the geography link is resolved during ETL (symbol -> company country -> country.csv), so that `geography_key` sits directly in the fact with no snowflaking.

**Keys.** Each dimension has a surrogate primary key. The fact also uses a surrogate key (`transaction_key`) because `IDTransaction` is **not unique** (only 1,136 distinct values across 2,281 rows, with the same id reused on different transactions). `IDTransaction` is kept as a *degenerate dimension* (descriptive attribute) inside the fact.

**Measures.** `units` (quantity traded) and `transaction_count` (a constant 1 used to count transactions).

**Avoiding duplication.** `country`, `region` and `sub_region` live only in `Dim_Geography`, not in `Dim_Symbol`.

**Hierarchies.**
- `Dim_Time`: Day -> Month -> Quarter -> Year
- `Dim_Geography`: Country -> Region -> Sub-region
- `Dim_Symbol`: Sector -> Industry -> Company

**Transaction types.** `DIVIDENT` (a typo for DIVIDEND) is a dividend event, not a trade. It is kept in the fact (consistent with the grain) but is naturally excluded by the BUY/SELL questions.

## 2. Data quality issues found

- **Empty rows and phantom column.** The account statement contained 464 completely empty rows and a trailing empty column caused by a final `;` separator. Both were removed, leaving **2,281 valid transactions**.
- **Dates.** Parsed from the format `%d/%m/%Y %H:%M:%S`; all transactions fall within 2024.
- **Unmapped symbols.** 212 transactions (**9.3%**, from 18 distinct symbols) use a symbol that does not exist in `symbols.csv`. They cannot be mapped to sector / industry / country and were excluded from the dimensional analysis, leaving **2,069 mapped transactions**.
- **Non-unique `IDTransaction`.** Handled with a surrogate key, as described above.
- **Country name alignment.** Two company-country names did not match the ISO 3166 names in `country.csv` and were corrected: `Taiwan -> Taiwan, Province of China` and `Turkey -> Türkiye`. After this, every company country maps.
- **Missing region for Taiwan.** In `country.csv`, "Taiwan, Province of China" has an empty `region` and `sub-region` (the ISO table leaves them blank, as it also does for Antarctica), even though Taiwan is traded (94 transactions). These values were **left as missing rather than filled with invented data**, and documented here. This has no effect on the five questions answered, since none of them use region or sub-region.

## 3. Main analytical results

The following five questions were answered: Q1, Q3, Q4, Q6, Q7.

**Q1 - Top 5 sectors by SELL transactions in the US (2024).** Out of 389 US sell transactions: Technology (158), Communication Services (58), Financial Services (55), Healthcare (50), Consumer Cyclical (48). Technology alone is about 40% of US selling.

**Q3 - Quarters of 2024 ranked by BUY+SELL transactions.** Q1 (968) > Q2 (522) > Q3 (242) > Q4 (241), out of 1,973 total. Activity drops steeply after Q1, and the two final quarters are essentially tied.

**Q4 - Top 10 countries by SELL transactions (2024).** Out of 989 sells: United States (389), United Kingdom (130), China (112), Brazil (69), Taiwan (50), Netherlands (46), Switzerland (37), Ireland (31), Luxembourg (27), Canada (22). Selling is strongly concentrated in the US.

**Q6 - Top 5 sectors by total units traded (BUY+SELL) in Q3 2024.** Out of 16,962 units: Technology (4,461), Healthcare (3,572), Consumer Cyclical (2,667), Financial Services (2,054), Communication Services (1,863). Measured by volume, the top is less concentrated than by transaction count.

**Q7 - Top 10 symbols by BUY+SELL transactions (2024).** ARM (100), AMD (97), TSM (77), TIMB (76), GOOG (48), AMZN (47), MSFT (45), ARDX (43), BRFS (42), RDDT (40). The most actively traded names are semiconductor and big-tech stocks.

## Deliverables

1. Jupyter notebook with ETL logic, star schema creation and the five analytical queries.
2. Star schema diagram (`star_schema.png`).
3. Streamlit application (`streamlit_app.zip`) - Page 1, Time Analysis.
4. This report.
