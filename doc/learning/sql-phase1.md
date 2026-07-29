# Phase 1: SQL Fundamentals

---

## 1. Filtering & Aggregation

**Concepts:**
- `WHERE` filters *before* aggregation (operates on individual rows)
- `HAVING` filters *after* aggregation (operates on grouped results)
- Aggregate functions: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`
- `GROUP BY` collapses rows into groups — every column in `SELECT` must either be in `GROUP BY` or wrapped in an aggregate

**Logical execution order (memorize this):**

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT
```

This order explains *every* confusing SQL behavior. Aliases defined in `SELECT` don't exist in `WHERE` because `WHERE` runs first. Window functions can't be in `WHERE` because they run after `WHERE`. `HAVING` can filter on aggregates because it runs after `GROUP BY`.

```sql
-- WHERE vs HAVING
SELECT user_id, COUNT(*) as txn_count
FROM transactions
WHERE status = 'APPROVED'        -- filters rows first
GROUP BY user_id
HAVING COUNT(*) > 10             -- filters groups after aggregation
```

---

### 1a. Conditional Aggregation

One of the most powerful patterns: use `CASE WHEN` inside an aggregate to compute multiple metrics in one pass.

```sql
-- Instead of multiple subqueries, do it in one scan
SELECT
    user_id,
    COUNT(*)                                          AS total_txns,
    COUNT(CASE WHEN status = 'APPROVED' THEN 1 END)  AS approved_count,
    COUNT(CASE WHEN status = 'DECLINED' THEN 1 END)  AS declined_count,
    SUM(CASE WHEN status = 'APPROVED' THEN amount ELSE 0 END) AS approved_spend,
    AVG(CASE WHEN status = 'APPROVED' THEN amount END)        AS avg_approved  -- NULLs excluded automatically
FROM transactions
GROUP BY user_id
```

**Why this matters:** This pattern avoids multiple self-joins or subqueries. In Spark on large tables, one scan vs three scans is the difference between minutes and seconds.

The `FILTER` clause (SQL standard, supported in Spark 3+) is a cleaner alternative:

```sql
SELECT
    user_id,
    COUNT(*) FILTER (WHERE status = 'APPROVED') AS approved_count,
    SUM(amount) FILTER (WHERE status = 'APPROVED') AS approved_spend
FROM transactions
GROUP BY user_id
```

---

### 1b. COUNT(DISTINCT ...) and Approximate Counting

```sql
-- Exact distinct count — expensive on large tables (requires dedup)
SELECT COUNT(DISTINCT user_id) FROM transactions

-- Approximate distinct count — much faster, ~2% error, fine for dashboards
SELECT APPROX_COUNT_DISTINCT(user_id) FROM transactions  -- Spark SQL
```

**When to use approximate:** Any dashboard metric where 1-2% error is acceptable and the table has >10M rows. Exact `COUNT(DISTINCT)` forces a full shuffle in distributed systems.

---

### 1c. ROLLUP, CUBE, and GROUPING SETS

**The problem they solve**

Say you have this data:

| region | category | amount |
|--------|----------|--------|
| US | Food | 100 |
| US | Travel | 200 |
| EU | Food | 150 |
| EU | Travel | 50 |

You want three things in one result:
1. Total per `(region, category)` — the detail rows
2. Total per `region` — subtotals
3. Grand total — one number

Without these features you'd write three separate queries and `UNION ALL` them:

```sql
SELECT region, category, SUM(amount) FROM t GROUP BY region, category
UNION ALL
SELECT region, NULL,     SUM(amount) FROM t GROUP BY region
UNION ALL
SELECT NULL,   NULL,     SUM(amount) FROM t
```

That works but scans the table three times. ROLLUP/CUBE/GROUPING SETS do this in **one pass**.

---

**ROLLUP — hierarchical subtotals**

`ROLLUP(region, category)` says: "give me every level of this hierarchy, from most granular to grand total." It **drops columns from right to left**, one level at a time.

```sql
SELECT region, category, SUM(amount)
FROM t
GROUP BY ROLLUP(region, category)
```

Result:

| region | category | sum |
|--------|----------|-----|
| EU | Food | 150 |
| EU | Travel | 50 |
| EU | NULL | 200 ← subtotal: all EU |
| US | Food | 100 |
| US | Travel | 200 |
| US | NULL | 300 ← subtotal: all US |
| NULL | NULL | 500 ← grand total |

`ROLLUP(a, b, c)` generates: `(a,b,c)`, `(a,b)`, `(a)`, `()`

**Use it when your dimensions have a natural hierarchy** — year → month → day, company → department → employee, region → country → city.

---

**CUBE — every possible combination**

`CUBE(region, category)` says: "give me every possible combination of these dimensions." It generates all 2ⁿ combinations where n = number of columns.

```sql
SELECT region, category, SUM(amount)
FROM t
GROUP BY CUBE(region, category)
```

Result:

| region | category | sum |
|--------|----------|-----|
| EU | Food | 150 |
| EU | Travel | 50 |
| EU | NULL | 200 ← subtotal: all EU |
| US | Food | 100 |
| US | Travel | 200 |
| US | NULL | 300 ← subtotal: all US |
| NULL | Food | 250 ← subtotal: all Food |
| NULL | Travel | 250 ← subtotal: all Travel |
| NULL | NULL | 500 ← grand total |

Notice the two extra rows vs ROLLUP: `(NULL, Food)` and `(NULL, Travel)` — totals by category alone. ROLLUP doesn't give you those because category has no meaning without region in a hierarchy. CUBE gives you all slices in every direction.

**Use it when dimensions are independent** and you want every possible slice — like a pivot table.

---

**GROUPING SETS — exact control**

You tell it exactly which combinations you want. No more, no less.

```sql
SELECT region, category, SUM(amount)
FROM t
GROUP BY GROUPING SETS (
    (region, category),  -- detail
    (region),            -- by region only
    ()                   -- grand total
)
```

ROLLUP and CUBE are just shorthand for GROUPING SETS:

```sql
-- ROLLUP(region, category) is shorthand for:
GROUP BY GROUPING SETS ((region, category), (region), ())

-- CUBE(region, category) is shorthand for:
GROUP BY GROUPING SETS ((region, category), (region), (category), ())
```

**Use it when you want some combinations but not others** — e.g. detail + grand total but not intermediate subtotals.

---

**The NULL ambiguity problem**

The gotcha: what if `region` legitimately contains NULL in your data? You can't tell if a NULL in the result means "subtotal row" or "the region value was actually NULL."

Fix it with `GROUPING()` — returns `1` if that column was rolled up (a subtotal NULL), `0` if it's a real value:

```sql
SELECT
    CASE WHEN GROUPING(region)   = 1 THEN 'ALL REGIONS'    ELSE region   END AS region,
    CASE WHEN GROUPING(category) = 1 THEN 'ALL CATEGORIES' ELSE category END AS category,
    SUM(amount)
FROM t
GROUP BY ROLLUP(region, category)
```

---

**Summary**

| Feature | What it gives you | Use when |
|---------|------------------|----------|
| `ROLLUP(a,b,c)` | Hierarchy: `(a,b,c)` → `(a,b)` → `(a)` → `()` | Dimensions have a natural parent-child order |
| `CUBE(a,b,c)` | All 2ⁿ combinations | Dimensions are independent, you want every slice |
| `GROUPING SETS` | Exactly what you specify | You want precise control over which groupings appear |

---

### Quiz

1. You want users who made more than 5 transactions. Should you use `WHERE` or `HAVING`?
2. Can you `WHERE` on an alias defined in `SELECT`? Why or why not?
3. What does `COUNT(*)` vs `COUNT(column)` do differently?
4. You run `SELECT user_id, status, COUNT(*) FROM transactions GROUP BY user_id` — does this work? Why not?
5. Write a single query that gives you, per user: total transactions, number of approved, number of declined. No subqueries, no joins.
6. `COUNT(DISTINCT user_id)` vs `APPROX_COUNT_DISTINCT(user_id)` — when would you choose each?
7. A query has `WHERE amount > 100` and `HAVING SUM(amount) > 1000`. Which filter reduces rows first? What's the execution order?
8. You need a grand total row alongside grouped rows in one result set. Which feature do you use?
9. What does `SUM(CASE WHEN status = 'APPROVED' THEN amount END)` return for a user who has zero approved transactions — `0` or `NULL`?
10. Can you use `HAVING` without `GROUP BY`? What does it mean if you do?

<details>
<summary>Answers</summary>

1. `HAVING COUNT(*) > 5` — you can't `WHERE` on an aggregate because aggregation hasn't happened yet.
2. No. `SELECT` is evaluated after `WHERE` in the logical order (`FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY`), so the alias doesn't exist at `WHERE` time.
3. `COUNT(*)` counts all rows including NULLs. `COUNT(column)` counts only non-NULL values in that column.
4. No — `status` is neither in `GROUP BY` nor aggregated. The engine doesn't know which `status` value to show when multiple rows collapse into one `user_id` group.
5. `SELECT user_id, COUNT(*) AS total, COUNT(CASE WHEN status='APPROVED' THEN 1 END) AS approved, COUNT(CASE WHEN status='DECLINED' THEN 1 END) AS declined FROM transactions GROUP BY user_id`
6. `COUNT(DISTINCT ...)` when exactness is required (billing, reconciliation, compliance). `APPROX_COUNT_DISTINCT` when the table is huge and ~2% error is acceptable (dashboards, estimates).
7. `WHERE` runs first (before `GROUP BY`), so it reduces rows before aggregation happens. `HAVING` only sees the aggregated groups. This means `WHERE` is more efficient — push as much filtering to `WHERE` as possible.
8. `ROLLUP` or `GROUPING SETS` with an empty group `()`.
9. `NULL` — `CASE WHEN` returns `NULL` for non-matching rows, and `SUM(NULL values only)` returns `NULL`. Use `COALESCE(SUM(...), 0)` to force zero.
10. Yes — it treats the entire table as a single group. `SELECT COUNT(*) FROM transactions HAVING COUNT(*) > 1000` returns a row only if the table has >1000 rows.

</details>

---

### Hands-on Exercises

**Exercise 1.1** — Basic aggregation

Write a query against `transactions(user_id, amount, status, created_at)` that returns users with total approved spend > $1000, ordered by spend descending.

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    SUM(amount) AS total_spend
FROM transactions
WHERE status = 'APPROVED'
GROUP BY user_id
HAVING SUM(amount) > 1000
ORDER BY total_spend DESC
```

</details>

---

**Exercise 1.2** — Conditional aggregation

From the same `transactions` table, return a single row per user containing:
- `total_txns`: all transactions
- `approved_spend`: sum of approved amounts
- `declined_count`: count of declined transactions
- `approval_rate`: approved_count / total_count as a decimal

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    COUNT(*)                                                    AS total_txns,
    COALESCE(SUM(CASE WHEN status = 'APPROVED' THEN amount END), 0) AS approved_spend,
    COUNT(CASE WHEN status = 'DECLINED' THEN 1 END)             AS declined_count,
    ROUND(
        COUNT(CASE WHEN status = 'APPROVED' THEN 1 END) * 1.0
        / NULLIF(COUNT(*), 0),
        4
    )                                                           AS approval_rate
FROM transactions
GROUP BY user_id
```

</details>

---

**Exercise 1.3** — ROLLUP

Given `transactions(region, merchant_category, amount)`, produce a result showing total spend broken down by region + category, by region alone, and a grand total — all in one query.

<details>
<summary>Answer</summary>

```sql
SELECT
    COALESCE(region, 'ALL')            AS region,
    COALESCE(merchant_category, 'ALL') AS category,
    SUM(amount)                        AS total_spend
FROM transactions
GROUP BY ROLLUP(region, merchant_category)
ORDER BY region, category
```

</details>

---

**Exercise 1.4** — Daily and weekly aggregation

From `transactions(txn_id, user_id, amount, created_at)`, compute for each calendar week:
- The number of distinct users who transacted
- Total transaction volume
- Average transaction size

Assume Spark SQL — use `DATE_TRUNC('week', created_at)` to normalize to week start.

<details>
<summary>Answer</summary>

```sql
SELECT
    DATE_TRUNC('week', created_at)  AS week_start,
    COUNT(DISTINCT user_id)         AS active_users,
    SUM(amount)                     AS total_volume,
    AVG(amount)                     AS avg_txn_size
FROM transactions
GROUP BY DATE_TRUNC('week', created_at)
ORDER BY week_start
```

</details>

---

## 2. JOINs

**Concepts:**
- `INNER JOIN` — only rows that match in **both** tables
- `LEFT JOIN` — all rows from left, NULLs for unmatched right
- `RIGHT JOIN` — all rows from right, NULLs for unmatched left (rarely used; just swap the tables and use LEFT JOIN)
- `FULL OUTER JOIN` — all rows from both, NULLs where no match
- `CROSS JOIN` — cartesian product (every row × every row), use intentionally

```sql
-- Classic LEFT JOIN pattern: find users with NO transactions
SELECT u.user_id
FROM users u
LEFT JOIN transactions t ON u.user_id = t.user_id
WHERE t.user_id IS NULL   -- NULL means no match was found
```

---

### 2a. ON vs WHERE on Outer Joins

This is the most commonly misunderstood JOIN behavior:

```sql
-- Filter in ON: rows not matching the condition get NULL instead of disappearing
SELECT u.user_id, t.amount
FROM users u
LEFT JOIN transactions t
    ON u.user_id = t.user_id
    AND t.status = 'APPROVED'    -- non-approved txns → NULL, user still appears
-- Result: ALL users, with amount=NULL for those who have no approved transactions

-- Filter in WHERE: drops NULLs AFTER the join — LEFT JOIN becomes INNER JOIN
SELECT u.user_id, t.amount
FROM users u
LEFT JOIN transactions t ON u.user_id = t.user_id
WHERE t.status = 'APPROVED'     -- drops rows where t.* is NULL
-- Result: only users who have at least one approved transaction
```

**Rule:** If you want to keep all left-side rows (the whole point of LEFT JOIN), put the right-table filter in `ON`. Putting it in `WHERE` silently converts it to an INNER JOIN.

---

### 2b. Self-Joins

A table joined to itself — used for hierarchical data or row-to-row comparisons within the same table.

```sql
-- Find employees and their managers from the same table
SELECT
    e.employee_id,
    e.name         AS employee_name,
    m.name         AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
-- LEFT JOIN so employees with no manager (CEO) still appear
```

```sql
-- Find pairs of users in the same company
SELECT a.user_id AS user_a, b.user_id AS user_b, a.company_id
FROM users a
JOIN users b
    ON a.company_id = b.company_id
    AND a.user_id < b.user_id   -- avoids (A,B) and (B,A) duplicates and (A,A) self-pairs
```

---

### 2c. Semi-Join and Anti-Join Patterns

These are extremely common in analytics — "give me rows from table A that DO (or DON'T) have a match in table B."

```sql
-- Semi-join: users who HAVE made a transaction (don't inflate with JOIN)
-- Option 1: EXISTS (often best for clarity)
SELECT u.*
FROM users u
WHERE EXISTS (
    SELECT 1 FROM transactions t WHERE t.user_id = u.user_id
)

-- Option 2: IN
SELECT * FROM users WHERE user_id IN (SELECT user_id FROM transactions)

-- Option 3: INNER JOIN + DISTINCT (careful — JOIN can inflate before DISTINCT)
SELECT DISTINCT u.*
FROM users u
JOIN transactions t ON u.user_id = t.user_id


-- Anti-join: users who have NEVER transacted
-- Option 1: NOT EXISTS (clearest)
SELECT u.*
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM transactions t WHERE t.user_id = u.user_id
)

-- Option 2: LEFT JOIN + IS NULL
SELECT u.*
FROM users u
LEFT JOIN transactions t ON u.user_id = t.user_id
WHERE t.user_id IS NULL
```

**Performance note:** In Spark, `NOT IN` with a subquery can behave unexpectedly if the subquery contains NULLs — `NOT IN (1, 2, NULL)` returns nothing because `x NOT IN (...NULL...)` is always NULL. Prefer `NOT EXISTS` or the LEFT JOIN anti-join pattern.

---

### 2d. Multi-Condition Joins and Non-Equi Joins

```sql
-- Join on multiple columns (composite key)
SELECT *
FROM orders o
JOIN order_items oi
    ON o.order_id = oi.order_id
    AND o.customer_id = oi.customer_id   -- composite key

-- Non-equi join: find which fee tier applies to each transaction
SELECT
    t.txn_id,
    t.amount,
    f.fee_percent
FROM transactions t
JOIN fee_tiers f
    ON t.amount >= f.min_amount
    AND t.amount < f.max_amount
-- Each transaction falls into exactly one tier — no GROUP BY needed IF tiers don't overlap
```

---

### Quiz

1. You LEFT JOIN users → transactions. The result has more rows than users. What happened?
2. What's the difference between filtering in `ON` vs `WHERE` on a LEFT JOIN?
3. You want all expense cards, and for each card the most recent transaction (or NULL if none). Which join?
4. Two tables each have 100 rows. A CROSS JOIN produces how many rows?
5. You need all users in the same company who are NOT the same person. Write the join condition.
6. `IN (subquery)` vs `EXISTS (subquery)` — is there a performance difference? When does it matter?
7. You LEFT JOIN users to transactions, then filter `WHERE t.status = 'APPROVED'`. You expect to see all users. Why might some users disappear?
8. You have a `fee_tiers` table with `min_amount` and `max_amount`. What kind of join do you use to assign a fee tier to each transaction?
9. You're joining three tables: users → cards → transactions. Should you always start with the biggest table? Why or why not?
10. What happens to NULLs in a JOIN key? Does `user_id = NULL` match another `user_id = NULL`?

<details>
<summary>Answers</summary>

1. Some users have multiple transactions — each transaction creates a new row. JOIN doesn't aggregate; it produces one row per matching pair.
2. Filtering in `ON` happens *during* the join (excluded right-side rows become NULL instead of disappearing). Filtering in `WHERE` happens *after* — rows with NULL are dropped, effectively turning a LEFT JOIN into an INNER JOIN for those filtered rows.
3. `LEFT JOIN` — you want all cards regardless of whether a transaction exists.
4. 100 × 100 = 10,000 rows.
5. `ON a.company_id = b.company_id AND a.user_id <> b.user_id` (or `< b.user_id` to deduplicate pairs).
6. In most modern query engines they're equivalent after optimization. `EXISTS` short-circuits on first match; `IN` may materialize the full subquery. For large subqueries with NULLs, `EXISTS` is safer because `NOT IN` breaks silently on NULLs.
7. The `WHERE t.status = 'APPROVED'` runs after the join. Users with no transactions have `t.status = NULL`, which fails the WHERE condition and drops them. Put the filter in the `ON` clause instead: `ON u.user_id = t.user_id AND t.status = 'APPROVED'`.
8. Non-equi join (range join): `ON t.amount >= f.min_amount AND t.amount < f.max_amount`.
9. It doesn't matter for correctness; the query optimizer reorders joins. But if your optimizer is weak, putting the most selective join first reduces intermediate result size. In Spark, use broadcast hints for small tables.
10. `NULL = NULL` evaluates to NULL (not TRUE), so NULL keys never match in joins. Rows with NULL join keys in the left table produce a NULL row (LEFT JOIN) or disappear (INNER JOIN).

</details>

---

### Hands-on Exercises

**Exercise 2.1** — Basic LEFT JOIN with aggregation

Given `cards(card_id, user_id)` and `transactions(txn_id, card_id, amount, created_at)`, return every card and its total spend (0 if no transactions).

<details>
<summary>Answer</summary>

```sql
SELECT
    c.card_id,
    c.user_id,
    COALESCE(SUM(t.amount), 0) AS total_spend
FROM cards c
LEFT JOIN transactions t ON c.card_id = t.card_id
GROUP BY c.card_id, c.user_id
```

</details>

---

**Exercise 2.2** — Anti-join

From `users(user_id, created_at)` and `transactions(txn_id, user_id, created_at)`, find users who registered more than 30 days ago but have never made a transaction.

<details>
<summary>Answer</summary>

```sql
SELECT u.user_id, u.created_at AS registered_at
FROM users u
WHERE u.created_at < CURRENT_DATE - INTERVAL 30 DAYS
  AND NOT EXISTS (
      SELECT 1
      FROM transactions t
      WHERE t.user_id = u.user_id
  )
```

</details>

---

**Exercise 2.3** — ON vs WHERE trap

Given `users(user_id, country)` and `transactions(txn_id, user_id, amount, status)`:

Write two queries:
1. All users with their total approved spend (users with no approved transactions should show 0)
2. Only users who have at least one approved transaction, with their total spend

Explain why these require different query structures.

<details>
<summary>Answer</summary>

```sql
-- Query 1: All users, approved spend only (filter in ON to preserve nulls)
SELECT
    u.user_id,
    COALESCE(SUM(t.amount), 0) AS approved_spend
FROM users u
LEFT JOIN transactions t
    ON u.user_id = t.user_id
    AND t.status = 'APPROVED'    -- filter here keeps all users
GROUP BY u.user_id

-- Query 2: Only users with approved transactions (filter in WHERE drops non-matchers)
SELECT
    u.user_id,
    SUM(t.amount) AS approved_spend
FROM users u
JOIN transactions t ON u.user_id = t.user_id
WHERE t.status = 'APPROVED'      -- INNER JOIN + WHERE is cleaner when you want this
GROUP BY u.user_id
```

Query 1 needs `LEFT JOIN` with filter in `ON` so that users with zero approved transactions still appear (with `amount=NULL`, converted to `0` by `COALESCE`). Query 2 can use `INNER JOIN` because we only want users who have at least one match.

</details>

---

**Exercise 2.4** — Multi-table join + self-join

Tables: `employees(emp_id, name, manager_id, department_id)`, `departments(dept_id, dept_name, budget)`.

Return each employee's name, their department name, their manager's name (NULL for the top of the hierarchy), and the department budget.

<details>
<summary>Answer</summary>

```sql
SELECT
    e.name            AS employee_name,
    d.dept_name,
    m.name            AS manager_name,
    d.budget
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
LEFT JOIN employees m ON e.manager_id = m.emp_id   -- LEFT so CEO (no manager) appears
ORDER BY d.dept_name, e.name
```

</details>

---

**Exercise 2.5** — Non-equi join

Table `transactions(txn_id, merchant_country, amount)`. Table `fx_rates(from_currency, to_currency, rate, valid_from, valid_to)`. Rates are time-bounded (no overlaps per pair).

Convert each transaction's amount to USD using the rate valid at the time of the transaction.

<details>
<summary>Answer</summary>

```sql
SELECT
    t.txn_id,
    t.amount,
    t.merchant_country,
    t.amount * f.rate AS amount_usd
FROM transactions t
JOIN fx_rates f
    ON f.from_currency = t.merchant_country
    AND f.to_currency = 'USD'
    AND t.created_at >= f.valid_from
    AND t.created_at <  f.valid_to
```

</details>

---

## 3. Window Functions

**Concepts:**
- Like aggregates, but they **don't collapse rows** — each row keeps its own result alongside the window computation
- Syntax: `function() OVER (PARTITION BY ... ORDER BY ... frame_clause)`
- `PARTITION BY` = "reset the window per group" (like GROUP BY but without collapsing)
- `ORDER BY` inside `OVER` = defines sequence within the window
- `frame_clause` = controls which rows are included in the window relative to the current row

---

### 3a. Ranking Functions

```sql
-- All three on the same data: amounts [100, 100, 200] per user
SELECT
    user_id,
    amount,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rn,
    -- 1, 2, 3 — always unique, tiebreaks are arbitrary
    RANK()       OVER (PARTITION BY user_id ORDER BY amount DESC) AS rnk,
    -- 1, 1, 3 — ties get the same rank, then a gap
    DENSE_RANK() OVER (PARTITION BY user_id ORDER BY amount DESC) AS dense_rnk
    -- 1, 1, 2 — ties get the same rank, no gap
FROM transactions
```

**When to use which:**
- `ROW_NUMBER`: deduplication, "latest record per key" patterns — you need exactly one row per partition
- `RANK`: leaderboards where ties should share a rank and the count of items at a given rank matters
- `DENSE_RANK`: leaderboards where you want a rank that tells you "you are N-th unique score"

---

### 3b. LAG and LEAD

Access values from preceding or following rows without a self-join.

```sql
SELECT
    user_id,
    created_at,
    amount,
    LAG(amount, 1)  OVER (PARTITION BY user_id ORDER BY created_at) AS prev_amount,
    LEAD(amount, 1) OVER (PARTITION BY user_id ORDER BY created_at) AS next_amount,
    amount - LAG(amount, 1) OVER (PARTITION BY user_id ORDER BY created_at) AS delta
FROM transactions
```

**LAG/LEAD signature:** `LAG(column, offset, default_if_null)` — the third argument fills NULLs at partition boundaries:

```sql
LAG(amount, 1, 0) OVER (PARTITION BY user_id ORDER BY created_at)
-- First row per user gets 0 instead of NULL
```

**Common use cases:**
- Day-over-day / week-over-week change
- Detecting gaps in sequences
- Session analysis (time since last event)

---

### 3c. Frame Specification — the Most Misunderstood Part

The frame determines which rows the window function sees relative to the current row. Without specifying it, the default differs based on whether `ORDER BY` is present:

| Clause | Default Frame |
|--------|--------------|
| No `ORDER BY` | `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` (whole partition) |
| With `ORDER BY` | `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (cumulative up to current) |

**ROWS vs RANGE:**
- `ROWS`: physical row offsets — `ROWS BETWEEN 1 PRECEDING AND 1 CURRENT ROW` = literally the row before and current row
- `RANGE`: logical value range — `RANGE BETWEEN 1 PRECEDING AND CURRENT ROW` = rows where the ORDER BY column is within 1 unit of the current row's value. Affects behavior on ties.

```sql
-- Running total (cumulative sum)
SUM(amount) OVER (
    PARTITION BY user_id
    ORDER BY created_at
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)

-- 7-day rolling average (physical rows)
AVG(amount) OVER (
    PARTITION BY user_id
    ORDER BY created_at
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW   -- current + 6 rows before = 7 rows
)

-- Entire partition total (no frame restriction, useful for % of total)
SUM(amount) OVER (PARTITION BY user_id)  -- no ORDER BY = whole partition

-- Future 3-day window
AVG(amount) OVER (
    ORDER BY created_at
    ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING
)
```

---

### 3d. NTILE, PERCENT_RANK, CUME_DIST

```sql
SELECT
    user_id,
    amount,
    NTILE(4)        OVER (ORDER BY amount DESC) AS quartile,
    -- Assigns rows to 4 roughly equal buckets: 1=top, 4=bottom
    PERCENT_RANK()  OVER (ORDER BY amount DESC) AS pct_rank,
    -- (rank - 1) / (total_rows - 1): 0.0 for first row, 1.0 for last
    CUME_DIST()     OVER (ORDER BY amount DESC) AS cum_dist
    -- Fraction of rows with value <= current row's value (0.0–1.0)
FROM transactions
```

**When to use:**
- `NTILE(n)`: bucketing users into percentile groups, A/B cohorts, decile reporting
- `PERCENT_RANK`: "this transaction is in the top X% of amounts" — excludes the current row from numerator
- `CUME_DIST`: "X% of transactions are at or below this amount" — includes current row

---

### 3e. FIRST_VALUE, LAST_VALUE, NTH_VALUE

```sql
SELECT
    user_id,
    created_at,
    amount,
    FIRST_VALUE(amount) OVER (
        PARTITION BY user_id ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS first_txn_amount,
    LAST_VALUE(amount) OVER (
        PARTITION BY user_id ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_txn_amount
FROM transactions
```

**Critical gotcha with LAST_VALUE:** The default frame is `UNBOUNDED PRECEDING TO CURRENT ROW`, so `LAST_VALUE` without an explicit frame just returns the current row's value (useless). Always specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` when you want the true last value of the partition.

---

### 3f. Common Patterns

**Pattern 1: Deduplication — keep latest record per key**
```sql
WITH deduped AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY updated_at DESC) AS rn
    FROM user_snapshots
)
SELECT * FROM deduped WHERE rn = 1
```

**Pattern 2: Running total + % of total**
```sql
SELECT
    user_id,
    created_at,
    amount,
    SUM(amount) OVER (PARTITION BY user_id ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
    amount / SUM(amount) OVER (PARTITION BY user_id) AS pct_of_user_total
FROM transactions
```

**Pattern 3: Session detection (gap > 30 min = new session)**
```sql
WITH flagged AS (
    SELECT *,
        CASE WHEN UNIX_TIMESTAMP(created_at) -
             LAG(UNIX_TIMESTAMP(created_at)) OVER (PARTITION BY user_id ORDER BY created_at) > 1800
             THEN 1 ELSE 0 END AS new_session_flag
    FROM events
),
sessions AS (
    SELECT *,
        SUM(new_session_flag) OVER (PARTITION BY user_id ORDER BY created_at
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS session_id
    FROM flagged
)
SELECT user_id, session_id, MIN(created_at) AS session_start, COUNT(*) AS events_in_session
FROM sessions
GROUP BY user_id, session_id
```

**Pattern 4: Gaps and Islands (find consecutive date ranges)**
```sql
-- Find continuous blocks of active subscription days
WITH ranked AS (
    SELECT user_id, active_date,
        active_date - INTERVAL (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY active_date)) DAYS AS grp
    FROM active_days
)
SELECT user_id, MIN(active_date) AS streak_start, MAX(active_date) AS streak_end
FROM ranked
GROUP BY user_id, grp
```

---

### Quiz

1. What's the difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` when there are ties?
2. You want the previous transaction amount for each transaction per card. Which function?
3. `SUM(amount) OVER (PARTITION BY user_id)` vs `SUM(amount) OVER (PARTITION BY user_id ORDER BY created_at)` — what's different?
4. Can you use a window function in a `WHERE` clause directly?
5. What is the default frame when `ORDER BY` is specified inside `OVER`? Why does this matter for `LAST_VALUE`?
6. You want to bucket users into 10 equal groups by spend. Which function?
7. `LAG(amount, 1, 0)` — what does the third argument `0` do?
8. You want to compute "this transaction's amount as a % of the user's total spend." Write the window expression.
9. What's the difference between `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` and `RANGE BETWEEN 6 PRECEDING AND CURRENT ROW` when `ORDER BY` is on a date column?
10. You have duplicate events and want to keep only the latest per `(user_id, event_type)`. Which window function and pattern?
11. `PERCENT_RANK()` returns `0.0` for the first row. Why?
12. How do you compute a 30-day rolling average in a window function?

<details>
<summary>Answers</summary>

1. Given values [100, 100, 200] ordered DESC:
   - `ROW_NUMBER` → 1, 2, 3 (arbitrary tiebreak, always unique)
   - `RANK` → 1, 1, 3 (tied rows share the same rank, then a gap equal to the number of tied rows)
   - `DENSE_RANK` → 1, 1, 2 (tied rows share the same rank, no gap)

2. `LAG(amount, 1) OVER (PARTITION BY card_id ORDER BY created_at)` — returns the value from the previous row in the partition.

3. Without `ORDER BY`: total sum for the whole partition (every row in the partition gets the same value). With `ORDER BY`: running cumulative sum up to and including the current row — the value increases as you go down the partition.

4. No — window functions execute after `WHERE` in the logical order. Wrap in a subquery or CTE and filter the outer query.

5. Default frame with `ORDER BY` is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This means `LAST_VALUE` just returns the current row's own value (the frame only goes up to the current row). Always specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` to get the true last value.

6. `NTILE(10) OVER (ORDER BY total_spend DESC)` — assigns each user to one of 10 roughly equal buckets.

7. The third argument is the default value when there is no previous row (i.e., the first row in the partition). Without it, `LAG` returns NULL for boundary rows.

8. `amount / SUM(amount) OVER (PARTITION BY user_id)` — the denominator uses no `ORDER BY`, so it computes the total for the entire partition.

9. `ROWS` counts physical rows: exactly 6 rows before the current one. `RANGE` counts logical value distance: all rows where the ORDER BY column is within 6 units of the current row's value. On dates, `RANGE BETWEEN 6 PRECEDING AND CURRENT ROW` includes all rows where the date is within 6 days — which could be more than 6 rows if there are multiple rows per day.

10. `ROW_NUMBER() OVER (PARTITION BY user_id, event_type ORDER BY created_at DESC) AS rn`, then filter `WHERE rn = 1`.

11. `PERCENT_RANK = (rank - 1) / (total_rows - 1)`. The first row has rank 1, so `(1-1)/(n-1) = 0.0`.

12. `AVG(amount) OVER (PARTITION BY user_id ORDER BY created_at ROWS BETWEEN 29 PRECEDING AND CURRENT ROW)` — 29 preceding + current row = 30 rows. Note: this is a 30-row rolling window, not calendar-day rolling. For true 30-calendar-day rolling, use `RANGE BETWEEN INTERVAL 29 DAYS PRECEDING AND CURRENT ROW` (Spark 3+ supports interval ranges).

</details>

---

### Hands-on Exercises

**Exercise 3.1** — Top-N per group

From `transactions(txn_id, user_id, amount, created_at)`, return each user's top 2 transactions by amount.

<details>
<summary>Answer</summary>

```sql
WITH ranked AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rn
    FROM transactions
)
SELECT * FROM ranked WHERE rn <= 2
```

</details>

---

**Exercise 3.2** — Running total and % of total

Return each transaction with:
- A cumulative spend total per user (ordered by `created_at`)
- The transaction's amount as a % of the user's all-time total spend

<details>
<summary>Answer</summary>

```sql
SELECT
    txn_id,
    user_id,
    created_at,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                                     AS running_total,
    ROUND(
        amount / SUM(amount) OVER (PARTITION BY user_id) * 100,
        2
    )                                                     AS pct_of_user_total
FROM transactions
```

</details>

---

**Exercise 3.3** — 7-day rolling average

Given `daily_spend(user_id, spend_date, total_amount)`, compute a 7-day rolling average spend per user.

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    spend_date,
    total_amount,
    AVG(total_amount) OVER (
        PARTITION BY user_id
        ORDER BY spend_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_7d_avg
FROM daily_spend
```

</details>

---

**Exercise 3.4** — LAG for change detection

From `transactions(txn_id, user_id, merchant_id, amount, created_at)`, find all transactions where the amount is more than 3x larger than the user's previous transaction (suspicious large transactions).

<details>
<summary>Answer</summary>

```sql
WITH with_prev AS (
    SELECT *,
        LAG(amount) OVER (PARTITION BY user_id ORDER BY created_at) AS prev_amount
    FROM transactions
)
SELECT txn_id, user_id, amount, prev_amount,
       ROUND(amount / prev_amount, 2) AS multiplier
FROM with_prev
WHERE prev_amount IS NOT NULL
  AND amount > prev_amount * 3
ORDER BY multiplier DESC
```

</details>

---

**Exercise 3.5** — Percentile bucketing

From `users(user_id, total_lifetime_spend)`, assign each user to a spend decile (1=top 10%, 10=bottom 10%) and compute the average spend per decile.

<details>
<summary>Answer</summary>

```sql
WITH deciles AS (
    SELECT
        user_id,
        total_lifetime_spend,
        NTILE(10) OVER (ORDER BY total_lifetime_spend DESC) AS decile
    FROM users
)
SELECT
    decile,
    COUNT(*)          AS user_count,
    AVG(total_lifetime_spend) AS avg_spend,
    MIN(total_lifetime_spend) AS min_spend,
    MAX(total_lifetime_spend) AS max_spend
FROM deciles
GROUP BY decile
ORDER BY decile
```

</details>

---

## 4. CTEs vs Subqueries

**Concepts:**
- Both let you break a query into steps
- CTE (`WITH name AS (...)`) — named, reusable within the query, reads top-to-bottom
- Subquery — inline, anonymous, can be harder to read when nested
- In Spark SQL, CTEs are preferred — they map cleanly to named DataFrames and help the optimizer

```sql
-- Subquery (harder to read when complex)
SELECT * FROM (
    SELECT user_id, SUM(amount) AS total FROM transactions GROUP BY user_id
) WHERE total > 1000

-- CTE (cleaner)
WITH user_spend AS (
    SELECT user_id, SUM(amount) AS total FROM transactions GROUP BY user_id
)
SELECT * FROM user_spend WHERE total > 1000
```

**Quiz:**

1. Can you reference a CTE more than once in the same query?
2. What's a "recursive CTE"? Name one use case.
3. When would a subquery be preferable to a CTE?
4. In Spark, does a CTE guarantee the result is computed only once, or can it be re-evaluated?

<details>
<summary>Answers</summary>

1. Yes — that's one of CTEs' main advantages over subqueries.
2. A CTE that references itself, used for hierarchical/tree data (e.g. org charts, category trees). Spark SQL supports it with `WITH RECURSIVE`.
3. Correlated subqueries in `WHERE` (though rare in analytics), or when the subquery is used exactly once and inlining makes the logic clearer.
4. Spark may re-evaluate a CTE multiple times unless you `CACHE` it or materialise it as a temp view. It's a logical alias, not a materialised result by default.

</details>

**Hands-on:**

Using CTEs, find users who have both approved AND declined transactions.

<details>
<summary>Answer</summary>

```sql
WITH approved AS (
    SELECT DISTINCT user_id FROM transactions WHERE status = 'APPROVED'
),
declined AS (
    SELECT DISTINCT user_id FROM transactions WHERE status = 'DECLINED'
)
SELECT a.user_id
FROM approved a
INNER JOIN declined d ON a.user_id = d.user_id
```

</details>

---

## 5. NULL Semantics

**Concepts:**
- NULL means *unknown* — it's not zero, not empty string, not false
- Any comparison with NULL returns NULL (not true/false): `NULL = NULL` → `NULL`
- Use `IS NULL` / `IS NOT NULL`, never `= NULL`
- `COALESCE(a, b, c)` — returns first non-NULL value
- `NULLIF(a, b)` — returns NULL if `a = b`, else `a` (useful to avoid divide-by-zero)
- Aggregates ignore NULLs (except `COUNT(*)`)

```sql
-- Divide-by-zero safe pattern
SELECT total_revenue / NULLIF(total_users, 0) AS revenue_per_user

-- Replace NULLs in output
SELECT COALESCE(company_name, 'Individual') AS payer
```

**Quiz:**

1. `SELECT 1 WHERE NULL = NULL` — returns a row or not?
2. `COUNT(*)` vs `COUNT(amount)` on a column with some NULLs — which is larger?
3. What does `COALESCE(NULL, NULL, 3, NULL)` return?
4. You `ORDER BY created_at DESC` and some rows have NULL. Where do NULLs appear — first or last? (Hint: it's database-dependent — what does Spark do?)

<details>
<summary>Answers</summary>

1. No row returned. `NULL = NULL` evaluates to `NULL` (not `TRUE`), so the `WHERE` condition fails.
2. `COUNT(*)` is larger (or equal) — it counts every row. `COUNT(amount)` skips NULLs.
3. `3` — first non-NULL value.
4. Spark SQL: NULLs sort as **largest** by default, so they appear **first** with `DESC`. Use `ORDER BY created_at DESC NULLS LAST` to push them to the end.

</details>

**Hands-on:**

Given `transactions(txn_id, user_id, amount, fee)` where `fee` can be NULL, return each user's average fee rate (`fee / amount`), treating NULL fees as 0.

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    AVG(COALESCE(fee, 0) / NULLIF(amount, 0)) AS avg_fee_rate
FROM transactions
GROUP BY user_id
```

`COALESCE(fee, 0)` treats NULL fee as zero. `NULLIF(amount, 0)` avoids divide-by-zero if amount is ever zero.

</details>

---

---

# Advanced SQL: Window Functions — Deep Dive and Internals

---

## A. How Window Functions Work Under the Hood

Understanding the implementation explains the performance profile and why certain queries are expensive.

### A1. The Three Phases of Window Execution

Every window function execution goes through three phases:

**Phase 1: Partition**
The engine partitions the data by `PARTITION BY` columns. In a single-node database this is a sort; in a distributed system like Spark this is a **shuffle** — data with the same partition key is routed to the same executor node. This is the most expensive phase for large datasets.

**Phase 2: Sort**
Within each partition, rows are sorted by the `ORDER BY` clause. This is an in-memory or spill-to-disk sort.

**Phase 3: Frame Evaluation**
The engine scans the sorted partition, maintaining a sliding frame pointer, and computes the function value for each row.

```
Input data (distributed across 3 nodes)
         ↓
   Shuffle by PARTITION BY key   ← EXPENSIVE: network I/O
         ↓
   Sort within each partition    ← O(n log n)
         ↓
   Slide frame, compute function ← O(n) per partition (for most functions)
         ↓
   Output (same row count as input)
```

### A2. The Cost of Multiple Window Functions

If you have multiple window functions with **the same PARTITION BY + ORDER BY**, a good optimizer (Spark 3+) can pipeline them through the same sorted partition pass — one shuffle, one sort:

```sql
-- Efficient: same window spec — optimizer pipelines these
SELECT
    ROW_NUMBER()  OVER w AS rn,
    RANK()        OVER w AS rnk,
    LAG(amount)   OVER w AS prev_amt,
    SUM(amount)   OVER w AS running_total
FROM transactions
WINDOW w AS (PARTITION BY user_id ORDER BY created_at)
```

But if you have different `PARTITION BY` or `ORDER BY`, the engine must shuffle and sort **separately for each unique window spec**:

```sql
-- Expensive: two different window specs = two shuffles + two sorts
SELECT
    RANK() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rank_by_amount,
    RANK() OVER (PARTITION BY merchant_id ORDER BY created_at) AS rank_by_time
FROM transactions
```

### A3. Frame Evaluation Algorithms

**Cumulative aggregates** (`UNBOUNDED PRECEDING TO CURRENT ROW`): The engine maintains a running accumulator — O(n) total per partition.

**Sliding window aggregates** (`N PRECEDING TO CURRENT ROW`): Requires maintaining a window queue. For `SUM` and `COUNT`, the engine can add the entering row and subtract the exiting row — O(n). For `MIN`/`MAX`, it needs a more complex structure (monotone deque) to handle the case where the current min/max drops off the window — O(n) amortized.

**RANGE-based frames with duplicates**: More expensive because the engine must find all rows in the value range, not just a fixed count.

### A4. Spark-Specific Behavior

In Spark, window functions always trigger a **shuffle** unless:
- There is no `PARTITION BY` (all data goes to one executor — dangerous on large tables)
- The data is already partitioned on the `PARTITION BY` key (e.g., you repartitioned before)

```python
# In PySpark, you can pre-partition to avoid the shuffle penalty
df.repartition("user_id") \
  .withColumn("rn", F.row_number().over(Window.partitionBy("user_id").orderBy("created_at")))
```

Spark also has a configurable spill threshold: if a partition's data doesn't fit in executor memory, it spills to disk. A partition skew problem (one user with 10M transactions vs others with 100) can cause OOM or extreme slowdown on a single executor.

---

## B. How Aggregation Works Internally

### B1. Hash Aggregation vs Sort-Based Aggregation

**Hash aggregation** (default in most engines for GROUP BY):
1. Build an in-memory hash map: key → accumulator (running sum, count, etc.)
2. For each row, hash the GROUP BY key, update the accumulator
3. At the end, emit one row per key

Complexity: O(n) time, O(distinct keys) space. Fails if the hash map exceeds memory — engine falls back to sort-based.

**Sort-based aggregation**:
1. Sort all rows by GROUP BY key
2. Scan sorted rows, emit aggregate when the key changes

Complexity: O(n log n) time, O(1) additional space. Used when: hash table exceeds memory, or data is already sorted.

```sql
-- In Spark, you can force sort-based aggregation (rarely needed):
SET spark.sql.execution.useObjectHashAggregateExec=false;
```

### B2. Partial Aggregation (Map-Side Combiner)

Spark's key optimization for distributed aggregation:

```
Stage 1 (per partition, no shuffle yet):
  Each executor pre-aggregates its local data → partial sums/counts

Stage 2 (after shuffle):
  Aggregate partial results from all executors → final result
```

This reduces shuffle data volume dramatically. `COUNT(DISTINCT ...)` can't be partially aggregated this way (you can't combine exact counts), which is why it forces a full shuffle and is expensive.

---

## C. How Joins Work Internally

### C1. Hash Join

Best for large table joined to a hashable table:
1. Build phase: scan the smaller table, build a hash map of `join_key → row`
2. Probe phase: scan the larger table, probe the hash map for each row

O(n) time, O(smaller table size) memory. Fails if the build-side table doesn't fit in memory.

### C2. Sort-Merge Join

For two large tables:
1. Sort both tables on the join key (triggers shuffle in distributed systems)
2. Merge the two sorted streams in one pass (like merge sort)

O(n log n) for the sort, O(n) for the merge. Both tables can be larger than memory. Default for large-large joins in Spark.

### C3. Broadcast Join (Spark-specific)

If one table is small enough (default < 10MB in Spark), Spark broadcasts it to all executors — no shuffle of the large table needed:

```sql
-- Force broadcast hint in Spark SQL
SELECT /*+ BROADCAST(small_table) */ *
FROM large_table
JOIN small_table ON large_table.id = small_table.id
```

This is the fastest join type when applicable. The key insight: instead of sending portions of the large table to where the small table's data is, you send the entire small table to where the large table already is.

### C4. What EXPLAIN Tells You

```sql
EXPLAIN SELECT * FROM transactions t JOIN cards c ON t.card_id = c.card_id;
```

In the output, look for:
- `BroadcastHashJoin` — small table was broadcast (good)
- `SortMergeJoin` — both tables shuffled and sorted (expensive but correct for large-large)
- `Exchange` — a shuffle happened here
- `Sort` — a sort happened here
- `FileScan` with `PushedFilters` — partition pruning is working

---

## D. Query Optimization: How the Planner Thinks

### D1. Predicate Pushdown

The optimizer moves WHERE filters as early as possible — ideally into the table scan itself (file-level pruning):

```sql
-- The optimizer will push this filter to the file scan level
-- (only read Parquet files/partitions where created_at >= 2024-01-01)
SELECT * FROM transactions
WHERE created_at >= '2024-01-01'
  AND user_id = 'abc123'
```

In a Parquet/Delta table partitioned by `created_at`, this means the engine doesn't even read older files. This is called **partition pruning** and can reduce I/O by orders of magnitude.

### D2. Column Pruning

Columnar formats (Parquet, Delta) store each column in separate chunks. The optimizer only reads columns referenced in the query:

```sql
-- Only reads the user_id and amount columns from disk
SELECT user_id, SUM(amount) FROM transactions GROUP BY user_id
-- Even though the table has 20 columns
```

### D3. Statistics and Cardinality Estimation

The optimizer uses table statistics (row counts, column distinct counts, min/max values) to:
- Choose join order (put the most selective join first)
- Decide hash vs sort-merge join
- Estimate intermediate result sizes

In Spark: `ANALYZE TABLE transactions COMPUTE STATISTICS FOR ALL COLUMNS` — run this after loading large tables to give the optimizer better information.

---

---

# Capstone Practice Problems

These are end-to-end problems. For each one: design and create the schema, load conceptual data, then answer the query challenges. These mirror real work.

---

## Problem 1: Expense Management Platform

**Scenario:** You're building the data layer for a corporate expense management system. Companies have employees who use corporate cards to make transactions. Finance admins review and approve/reject expense reports. You need to support analytics, fraud detection, and policy enforcement.

### Step 1: Design the schema

Before writing any SQL, answer these design questions:
1. What entities do you need?
2. What are the cardinalities? (1:many, many:many)
3. What columns need indexes?
4. What columns are good partition keys for a data warehouse?

Then create the database:

```sql
-- PostgreSQL / Spark SQL compatible

CREATE TABLE companies (
    company_id      VARCHAR(36)     NOT NULL,
    company_name    VARCHAR(255)    NOT NULL,
    country         VARCHAR(2)      NOT NULL,
    plan_tier       VARCHAR(20)     NOT NULL,   -- 'FREE', 'STARTER', 'ENTERPRISE'
    created_at      TIMESTAMP       NOT NULL,
    PRIMARY KEY (company_id)
);

CREATE TABLE users (
    user_id         VARCHAR(36)     NOT NULL,
    company_id      VARCHAR(36)     NOT NULL,
    email           VARCHAR(255)    NOT NULL,
    role            VARCHAR(20)     NOT NULL,   -- 'EMPLOYEE', 'ADMIN', 'OWNER'
    department      VARCHAR(100),
    created_at      TIMESTAMP       NOT NULL,
    PRIMARY KEY (user_id)
);

CREATE TABLE cards (
    card_id         VARCHAR(36)     NOT NULL,
    user_id         VARCHAR(36)     NOT NULL,
    company_id      VARCHAR(36)     NOT NULL,
    card_type       VARCHAR(20)     NOT NULL,   -- 'VIRTUAL', 'PHYSICAL'
    spend_limit     DECIMAL(15, 2)  NOT NULL,
    status          VARCHAR(20)     NOT NULL,   -- 'ACTIVE', 'FROZEN', 'CANCELLED'
    issued_at       TIMESTAMP       NOT NULL,
    PRIMARY KEY (card_id)
);

CREATE TABLE transactions (
    txn_id              VARCHAR(36)     NOT NULL,
    card_id             VARCHAR(36)     NOT NULL,
    user_id             VARCHAR(36)     NOT NULL,
    company_id          VARCHAR(36)     NOT NULL,
    amount              DECIMAL(15, 2)  NOT NULL,
    currency            VARCHAR(3)      NOT NULL,
    amount_usd          DECIMAL(15, 2),
    merchant_name       VARCHAR(255),
    merchant_category   VARCHAR(100),
    merchant_country    VARCHAR(2),
    status              VARCHAR(20)     NOT NULL,  -- 'APPROVED', 'DECLINED', 'PENDING', 'REVERSED'
    declined_reason     VARCHAR(100),              -- NULL if approved
    created_at          TIMESTAMP       NOT NULL,
    PRIMARY KEY (txn_id)
);

CREATE TABLE expense_reports (
    report_id       VARCHAR(36)     NOT NULL,
    user_id         VARCHAR(36)     NOT NULL,
    company_id      VARCHAR(36)     NOT NULL,
    title           VARCHAR(255)    NOT NULL,
    status          VARCHAR(20)     NOT NULL,   -- 'DRAFT', 'SUBMITTED', 'APPROVED', 'REJECTED'
    submitted_at    TIMESTAMP,
    reviewed_at     TIMESTAMP,
    reviewer_id     VARCHAR(36),
    PRIMARY KEY (report_id)
);

CREATE TABLE report_transactions (
    report_id   VARCHAR(36)     NOT NULL,
    txn_id      VARCHAR(36)     NOT NULL,
    PRIMARY KEY (report_id, txn_id)
);
```

---

### Step 2: Query Challenges

**Challenge 1.1 — Monthly spend by company and category**

Write a query that shows, for each company, for each month, the total spend broken down by `merchant_category`. Also include a subtotal per company per month (all categories combined) and a grand total row. Return `company_name`, `month`, `category`, `total_spend`.

<details>
<summary>Answer</summary>

```sql
SELECT
    c.company_name,
    DATE_TRUNC('month', t.created_at)    AS month,
    COALESCE(t.merchant_category, 'ALL') AS category,
    SUM(t.amount_usd)                    AS total_spend
FROM transactions t
JOIN companies c ON t.company_id = c.company_id
WHERE t.status = 'APPROVED'
GROUP BY GROUPING SETS (
    (c.company_name, DATE_TRUNC('month', t.created_at), t.merchant_category),
    (c.company_name, DATE_TRUNC('month', t.created_at)),
    ()
)
ORDER BY c.company_name, month, category
```

</details>

---

**Challenge 1.2 — Card utilization rate**

For each card that has been active for at least 30 days, compute:
- Total spend in the last 30 days
- The card's `spend_limit`
- Utilization rate as a percentage
- Whether it is "LOW" (<30%), "MEDIUM" (30–80%), or "HIGH" (>80%) utilization

<details>
<summary>Answer</summary>

```sql
WITH recent_spend AS (
    SELECT
        card_id,
        SUM(amount_usd) AS spend_30d
    FROM transactions
    WHERE status = 'APPROVED'
      AND created_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
    GROUP BY card_id
)
SELECT
    ca.card_id,
    ca.user_id,
    ca.spend_limit,
    COALESCE(rs.spend_30d, 0)                        AS spend_30d,
    ROUND(COALESCE(rs.spend_30d, 0) / NULLIF(ca.spend_limit, 0) * 100, 2) AS utilization_pct,
    CASE
        WHEN COALESCE(rs.spend_30d, 0) / NULLIF(ca.spend_limit, 0) < 0.30 THEN 'LOW'
        WHEN COALESCE(rs.spend_30d, 0) / NULLIF(ca.spend_limit, 0) < 0.80 THEN 'MEDIUM'
        ELSE 'HIGH'
    END                                              AS utilization_band
FROM cards ca
LEFT JOIN recent_spend rs ON ca.card_id = rs.card_id
WHERE ca.status = 'ACTIVE'
  AND ca.issued_at <= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
```

</details>

---

**Challenge 1.3 — Suspicious velocity detection**

Flag transactions where a user has spent more than 3x their own 90-day average daily spend in a single day. Return the user, the suspicious date, that day's spend, and their 90-day average.

<details>
<summary>Answer</summary>

```sql
WITH daily_spend AS (
    SELECT
        user_id,
        DATE(created_at)    AS spend_date,
        SUM(amount_usd)     AS day_total
    FROM transactions
    WHERE status = 'APPROVED'
    GROUP BY user_id, DATE(created_at)
),
with_avg AS (
    SELECT
        user_id,
        spend_date,
        day_total,
        AVG(day_total) OVER (
            PARTITION BY user_id
            ORDER BY spend_date
            ROWS BETWEEN 89 PRECEDING AND CURRENT ROW
        ) AS rolling_90d_avg
    FROM daily_spend
)
SELECT
    user_id,
    spend_date,
    day_total,
    ROUND(rolling_90d_avg, 2) AS avg_90d_daily,
    ROUND(day_total / rolling_90d_avg, 2) AS multiple
FROM with_avg
WHERE day_total > rolling_90d_avg * 3
ORDER BY multiple DESC
```

</details>

---

**Challenge 1.4 — Expense report approval funnel**

For each company and each month reports were submitted, compute:
- Total reports submitted
- Reports approved within 3 days
- Reports still pending after 7 days
- Average days to approval (for approved reports only)

<details>
<summary>Answer</summary>

```sql
SELECT
    er.company_id,
    DATE_TRUNC('month', er.submitted_at)    AS submit_month,
    COUNT(*)                                AS total_submitted,
    COUNT(CASE
        WHEN er.status = 'APPROVED'
          AND DATEDIFF(er.reviewed_at, er.submitted_at) <= 3
        THEN 1
    END)                                    AS approved_within_3d,
    COUNT(CASE
        WHEN er.status IN ('SUBMITTED', 'DRAFT')
          AND DATEDIFF(CURRENT_TIMESTAMP, er.submitted_at) > 7
        THEN 1
    END)                                    AS pending_over_7d,
    ROUND(AVG(CASE
        WHEN er.status = 'APPROVED'
        THEN DATEDIFF(er.reviewed_at, er.submitted_at)
    END), 1)                                AS avg_days_to_approval
FROM expense_reports er
WHERE er.submitted_at IS NOT NULL
GROUP BY er.company_id, DATE_TRUNC('month', er.submitted_at)
ORDER BY er.company_id, submit_month
```

</details>

---

**Challenge 1.5 — First, second, and third transaction per user (the hard one)**

For each user, return their 1st, 2nd, and 3rd approved transactions as separate columns in a single row: `user_id`, `first_txn_amount`, `first_txn_date`, `second_txn_amount`, `second_txn_date`, `third_txn_amount`, `third_txn_date`. Users with fewer than 3 transactions should show NULL for missing values.

<details>
<summary>Answer</summary>

```sql
WITH ranked AS (
    SELECT
        user_id,
        amount_usd,
        created_at,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) AS rn
    FROM transactions
    WHERE status = 'APPROVED'
)
SELECT
    user_id,
    MAX(CASE WHEN rn = 1 THEN amount_usd END) AS first_txn_amount,
    MAX(CASE WHEN rn = 1 THEN created_at  END) AS first_txn_date,
    MAX(CASE WHEN rn = 2 THEN amount_usd END) AS second_txn_amount,
    MAX(CASE WHEN rn = 2 THEN created_at  END) AS second_txn_date,
    MAX(CASE WHEN rn = 3 THEN amount_usd END) AS third_txn_amount,
    MAX(CASE WHEN rn = 3 THEN created_at  END) AS third_txn_date
FROM ranked
WHERE rn <= 3
GROUP BY user_id
```

</details>

---

## Problem 2: Ride-Sharing Analytics Platform

**Scenario:** You work on the analytics team for a ride-sharing company. Drivers complete trips for riders. You need to support driver performance analytics, surge pricing analysis, and revenue attribution.

### Step 1: Create the Schema

```sql
CREATE TABLE drivers (
    driver_id       VARCHAR(36)     NOT NULL,
    city            VARCHAR(100)    NOT NULL,
    rating          DECIMAL(3, 2),            -- 1.00 to 5.00
    vehicle_type    VARCHAR(20)     NOT NULL,  -- 'STANDARD', 'PREMIUM', 'XL'
    onboarded_at    TIMESTAMP       NOT NULL,
    is_active       BOOLEAN         NOT NULL,
    PRIMARY KEY (driver_id)
);

CREATE TABLE riders (
    rider_id        VARCHAR(36)     NOT NULL,
    city            VARCHAR(100)    NOT NULL,
    created_at      TIMESTAMP       NOT NULL,
    PRIMARY KEY (rider_id)
);

CREATE TABLE trips (
    trip_id             VARCHAR(36)     NOT NULL,
    driver_id           VARCHAR(36)     NOT NULL,
    rider_id            VARCHAR(36)     NOT NULL,
    city                VARCHAR(100)    NOT NULL,
    vehicle_type        VARCHAR(20)     NOT NULL,
    status              VARCHAR(20)     NOT NULL,  -- 'COMPLETED', 'CANCELLED', 'NO_SHOW'
    requested_at        TIMESTAMP       NOT NULL,
    started_at          TIMESTAMP,
    completed_at        TIMESTAMP,
    pickup_lat          DECIMAL(9, 6),
    pickup_lon          DECIMAL(9, 6),
    dropoff_lat         DECIMAL(9, 6),
    dropoff_lon         DECIMAL(9, 6),
    distance_km         DECIMAL(6, 2),
    base_fare           DECIMAL(10, 2),
    surge_multiplier    DECIMAL(4, 2)   DEFAULT 1.0,
    total_fare          DECIMAL(10, 2),
    driver_payout       DECIMAL(10, 2),
    PRIMARY KEY (trip_id)
);

CREATE TABLE driver_ratings (
    rating_id   VARCHAR(36)     NOT NULL,
    trip_id     VARCHAR(36)     NOT NULL,
    driver_id   VARCHAR(36)     NOT NULL,
    stars       INT             NOT NULL,   -- 1 to 5
    comment     TEXT,
    created_at  TIMESTAMP       NOT NULL,
    PRIMARY KEY (rating_id)
);
```

---

### Step 2: Query Challenges

**Challenge 2.1 — Driver performance ranking**

For each driver, compute over the last 90 days:
- Total completed trips
- Average rating (from `driver_ratings`)
- Total earnings (`driver_payout`)
- Cancellation rate (cancelled / total requested)
- Rank each driver within their city by total earnings

<details>
<summary>Answer</summary>

```sql
WITH driver_stats AS (
    SELECT
        t.driver_id,
        t.city,
        COUNT(*)                                                           AS total_trips,
        COUNT(CASE WHEN t.status = 'COMPLETED' THEN 1 END)                AS completed_trips,
        COUNT(CASE WHEN t.status = 'CANCELLED' THEN 1 END)                AS cancelled_trips,
        SUM(CASE WHEN t.status = 'COMPLETED' THEN t.driver_payout END)    AS total_earnings,
        ROUND(
            COUNT(CASE WHEN t.status = 'CANCELLED' THEN 1 END) * 1.0
            / NULLIF(COUNT(*), 0),
            4
        )                                                                  AS cancellation_rate
    FROM trips t
    WHERE t.requested_at >= CURRENT_TIMESTAMP - INTERVAL 90 DAYS
    GROUP BY t.driver_id, t.city
),
avg_ratings AS (
    SELECT driver_id, AVG(stars) AS avg_rating
    FROM driver_ratings
    WHERE created_at >= CURRENT_TIMESTAMP - INTERVAL 90 DAYS
    GROUP BY driver_id
)
SELECT
    ds.driver_id,
    ds.city,
    ds.completed_trips,
    ROUND(ar.avg_rating, 2)                                              AS avg_rating,
    ROUND(ds.total_earnings, 2)                                          AS total_earnings,
    ROUND(ds.cancellation_rate * 100, 2)                                 AS cancellation_pct,
    RANK() OVER (PARTITION BY ds.city ORDER BY ds.total_earnings DESC)   AS earnings_rank_in_city
FROM driver_stats ds
LEFT JOIN avg_ratings ar ON ds.driver_id = ar.driver_id
```

</details>

---

**Challenge 2.2 — Surge pricing windows**

For every hour of every day, compute:
- Number of completed trips
- Average surge multiplier
- Revenue vs what it would have been at base fare (no surge)
- "Surge lift": the additional revenue generated by surge pricing

Then identify the top 5 hour-of-day / city combinations by average surge multiplier.

<details>
<summary>Answer</summary>

```sql
WITH hourly AS (
    SELECT
        city,
        DATE_TRUNC('hour', started_at)  AS hour_bucket,
        HOUR(started_at)                AS hour_of_day,
        COUNT(*)                        AS completed_trips,
        AVG(surge_multiplier)           AS avg_surge,
        SUM(total_fare)                 AS actual_revenue,
        SUM(base_fare)                  AS base_revenue
    FROM trips
    WHERE status = 'COMPLETED'
      AND started_at IS NOT NULL
    GROUP BY city, DATE_TRUNC('hour', started_at), HOUR(started_at)
),
ranked AS (
    SELECT *,
        ROUND(actual_revenue - base_revenue, 2)          AS surge_lift,
        RANK() OVER (ORDER BY avg_surge DESC)             AS surge_rank
    FROM hourly
)
SELECT
    city,
    hour_of_day,
    completed_trips,
    ROUND(avg_surge, 3)     AS avg_surge_multiplier,
    ROUND(actual_revenue, 2) AS actual_revenue,
    ROUND(base_revenue, 2)   AS base_revenue,
    surge_lift
FROM ranked
WHERE surge_rank <= 5
ORDER BY avg_surge_multiplier DESC
```

</details>

---

**Challenge 2.3 — Rider cohort retention**

Define a rider's "cohort" as the month they completed their first ever trip. For each cohort, compute how many riders completed at least one trip in each of the following 6 months (months 0–6 relative to their cohort month). This is a cohort retention analysis.

<details>
<summary>Answer</summary>

```sql
WITH first_trips AS (
    SELECT
        rider_id,
        DATE_TRUNC('month', MIN(completed_at)) AS cohort_month
    FROM trips
    WHERE status = 'COMPLETED'
    GROUP BY rider_id
),
rider_activity AS (
    SELECT
        t.rider_id,
        DATE_TRUNC('month', t.completed_at)     AS activity_month
    FROM trips t
    WHERE status = 'COMPLETED'
    GROUP BY t.rider_id, DATE_TRUNC('month', t.completed_at)
),
cohort_activity AS (
    SELECT
        f.cohort_month,
        DATEDIFF(MONTH, f.cohort_month, ra.activity_month) AS months_since_cohort,
        COUNT(DISTINCT ra.rider_id)             AS active_riders
    FROM first_trips f
    JOIN rider_activity ra ON f.rider_id = ra.rider_id
    WHERE DATEDIFF(MONTH, f.cohort_month, ra.activity_month) BETWEEN 0 AND 6
    GROUP BY f.cohort_month, DATEDIFF(MONTH, f.cohort_month, ra.activity_month)
),
cohort_sizes AS (
    SELECT cohort_month, COUNT(*) AS cohort_size
    FROM first_trips
    GROUP BY cohort_month
)
SELECT
    ca.cohort_month,
    cs.cohort_size,
    ca.months_since_cohort,
    ca.active_riders,
    ROUND(ca.active_riders * 100.0 / cs.cohort_size, 1) AS retention_pct
FROM cohort_activity ca
JOIN cohort_sizes cs ON ca.cohort_month = cs.cohort_month
ORDER BY ca.cohort_month, ca.months_since_cohort
```

</details>

---

## Problem 3: E-Commerce Inventory and Sales Analytics

**Scenario:** You're building analytics for an e-commerce platform. Products have inventory across multiple warehouses. Orders come in from customers and are fulfilled by the nearest warehouse. You need to support inventory health reports, sales velocity analysis, and warehouse efficiency metrics.

### Step 1: Create the Schema

```sql
CREATE TABLE products (
    product_id      VARCHAR(36)     NOT NULL,
    sku             VARCHAR(100)    NOT NULL,
    product_name    VARCHAR(255)    NOT NULL,
    category        VARCHAR(100)    NOT NULL,
    subcategory     VARCHAR(100),
    unit_cost       DECIMAL(10, 2)  NOT NULL,
    retail_price    DECIMAL(10, 2)  NOT NULL,
    weight_kg       DECIMAL(6, 3),
    PRIMARY KEY (product_id),
    UNIQUE (sku)
);

CREATE TABLE warehouses (
    warehouse_id    VARCHAR(36)     NOT NULL,
    warehouse_name  VARCHAR(100)    NOT NULL,
    country         VARCHAR(2)      NOT NULL,
    city            VARCHAR(100)    NOT NULL,
    capacity_units  INT             NOT NULL,
    PRIMARY KEY (warehouse_id)
);

CREATE TABLE inventory (
    inventory_id    VARCHAR(36)     NOT NULL,
    product_id      VARCHAR(36)     NOT NULL,
    warehouse_id    VARCHAR(36)     NOT NULL,
    quantity        INT             NOT NULL,
    reorder_point   INT             NOT NULL,   -- trigger reorder when quantity <= this
    reorder_qty     INT             NOT NULL,   -- how many to order
    updated_at      TIMESTAMP       NOT NULL,
    PRIMARY KEY (inventory_id),
    UNIQUE (product_id, warehouse_id)
);

CREATE TABLE customers (
    customer_id     VARCHAR(36)     NOT NULL,
    country         VARCHAR(2)      NOT NULL,
    city            VARCHAR(100)    NOT NULL,
    created_at      TIMESTAMP       NOT NULL,
    PRIMARY KEY (customer_id)
);

CREATE TABLE orders (
    order_id        VARCHAR(36)     NOT NULL,
    customer_id     VARCHAR(36)     NOT NULL,
    warehouse_id    VARCHAR(36)     NOT NULL,   -- fulfilling warehouse
    status          VARCHAR(20)     NOT NULL,   -- 'PENDING', 'SHIPPED', 'DELIVERED', 'CANCELLED'
    ordered_at      TIMESTAMP       NOT NULL,
    shipped_at      TIMESTAMP,
    delivered_at    TIMESTAMP,
    PRIMARY KEY (order_id)
);

CREATE TABLE order_items (
    order_item_id   VARCHAR(36)     NOT NULL,
    order_id        VARCHAR(36)     NOT NULL,
    product_id      VARCHAR(36)     NOT NULL,
    quantity        INT             NOT NULL,
    unit_price      DECIMAL(10, 2)  NOT NULL,   -- price at time of sale
    discount_pct    DECIMAL(5, 2)   DEFAULT 0,
    PRIMARY KEY (order_item_id)
);
```

---

### Step 2: Query Challenges

**Challenge 3.1 — Inventory health dashboard**

For each product at each warehouse, compute:
- Current quantity on hand
- Average daily sales velocity (units sold per day, over the last 30 days)
- Days of supply remaining (`quantity / avg_daily_velocity`)
- A health flag: `CRITICAL` (<7 days), `LOW` (7–14 days), `OK` (14–30 days), `EXCESS` (>30 days)
- Whether it is currently below the reorder point

<details>
<summary>Answer</summary>

```sql
WITH recent_sales AS (
    SELECT
        oi.product_id,
        o.warehouse_id,
        SUM(oi.quantity) / 30.0 AS avg_daily_velocity
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.order_id
    WHERE o.status IN ('SHIPPED', 'DELIVERED')
      AND o.ordered_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
    GROUP BY oi.product_id, o.warehouse_id
)
SELECT
    p.sku,
    p.product_name,
    w.warehouse_name,
    i.quantity,
    i.reorder_point,
    ROUND(COALESCE(rs.avg_daily_velocity, 0), 2)  AS avg_daily_velocity,
    CASE
        WHEN COALESCE(rs.avg_daily_velocity, 0) = 0 THEN NULL
        ELSE ROUND(i.quantity / rs.avg_daily_velocity, 1)
    END                                            AS days_of_supply,
    CASE
        WHEN COALESCE(rs.avg_daily_velocity, 0) = 0 THEN 'NO_MOVEMENT'
        WHEN i.quantity / rs.avg_daily_velocity < 7  THEN 'CRITICAL'
        WHEN i.quantity / rs.avg_daily_velocity < 14 THEN 'LOW'
        WHEN i.quantity / rs.avg_daily_velocity < 30 THEN 'OK'
        ELSE 'EXCESS'
    END                                            AS health_flag,
    i.quantity <= i.reorder_point                  AS needs_reorder
FROM inventory i
JOIN products p   ON i.product_id   = p.product_id
JOIN warehouses w ON i.warehouse_id = w.warehouse_id
LEFT JOIN recent_sales rs
    ON i.product_id   = rs.product_id
    AND i.warehouse_id = rs.warehouse_id
ORDER BY health_flag, days_of_supply
```

</details>

---

**Challenge 3.2 — Sales velocity trend and week-over-week change**

For each product and each week, compute total revenue. Also compute the week-over-week revenue change (absolute and %) and a 4-week rolling average revenue. Flag products where the current week's revenue is more than 20% below their 4-week rolling average (declining products).

<details>
<summary>Answer</summary>

```sql
WITH weekly_revenue AS (
    SELECT
        oi.product_id,
        DATE_TRUNC('week', o.ordered_at)                AS week_start,
        SUM(oi.quantity * oi.unit_price * (1 - oi.discount_pct / 100)) AS revenue
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.order_id
    WHERE o.status IN ('SHIPPED', 'DELIVERED')
    GROUP BY oi.product_id, DATE_TRUNC('week', o.ordered_at)
),
with_metrics AS (
    SELECT
        product_id,
        week_start,
        revenue,
        LAG(revenue) OVER (PARTITION BY product_id ORDER BY week_start) AS prev_week_revenue,
        AVG(revenue) OVER (
            PARTITION BY product_id
            ORDER BY week_start
            ROWS BETWEEN 3 PRECEDING AND CURRENT ROW
        ) AS rolling_4w_avg
    FROM weekly_revenue
)
SELECT
    p.product_name,
    p.category,
    wm.week_start,
    ROUND(wm.revenue, 2)                                       AS revenue,
    ROUND(wm.prev_week_revenue, 2)                             AS prev_week_revenue,
    ROUND(wm.revenue - wm.prev_week_revenue, 2)                AS wow_change,
    ROUND((wm.revenue - wm.prev_week_revenue)
          / NULLIF(wm.prev_week_revenue, 0) * 100, 1)          AS wow_pct_change,
    ROUND(wm.rolling_4w_avg, 2)                                AS rolling_4w_avg,
    wm.revenue < wm.rolling_4w_avg * 0.80                      AS is_declining
FROM with_metrics wm
JOIN products p ON wm.product_id = p.product_id
ORDER BY p.product_name, wm.week_start
```

</details>

---

**Challenge 3.3 — Warehouse efficiency: fulfillment time and throughput**

For each warehouse, compute:
- Orders fulfilled per day (throughput) — rolling 30-day average
- Average days from `ordered_at` to `shipped_at` (processing time)
- P50 and P95 shipping time (use percentile approximation)
- The warehouse's total revenue processed in the last 30 days

<details>
<summary>Answer</summary>

```sql
WITH warehouse_orders AS (
    SELECT
        o.warehouse_id,
        o.order_id,
        o.ordered_at,
        o.shipped_at,
        DATEDIFF(o.shipped_at, o.ordered_at)  AS processing_days,
        SUM(oi.quantity * oi.unit_price * (1 - oi.discount_pct / 100)) AS order_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.status IN ('SHIPPED', 'DELIVERED')
      AND o.shipped_at IS NOT NULL
    GROUP BY o.warehouse_id, o.order_id, o.ordered_at, o.shipped_at
)
SELECT
    w.warehouse_name,
    w.city,
    COUNT(wo.order_id)                                  AS orders_last_30d,
    ROUND(COUNT(wo.order_id) / 30.0, 2)                AS avg_daily_throughput,
    ROUND(AVG(wo.processing_days), 2)                  AS avg_processing_days,
    PERCENTILE_APPROX(wo.processing_days, 0.50)        AS p50_days,
    PERCENTILE_APPROX(wo.processing_days, 0.95)        AS p95_days,
    ROUND(SUM(wo.order_revenue), 2)                    AS total_revenue_30d
FROM warehouses w
LEFT JOIN warehouse_orders wo
    ON w.warehouse_id = wo.warehouse_id
    AND wo.ordered_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
GROUP BY w.warehouse_id, w.warehouse_name, w.city
ORDER BY avg_daily_throughput DESC
```

</details>

---

**Challenge 3.4 — Product affinity (customers who bought X also bought Y)**

Find all product pairs that were purchased together (in the same order) at least 50 times. Return the two product names, co-occurrence count, and co-occurrence rate (count / total orders containing either product). This is a market basket / product affinity analysis.

<details>
<summary>Answer</summary>

```sql
WITH order_products AS (
    SELECT DISTINCT order_id, product_id
    FROM order_items
),
product_pairs AS (
    SELECT
        a.product_id AS product_a,
        b.product_id AS product_b,
        COUNT(DISTINCT a.order_id) AS co_occurrences
    FROM order_products a
    JOIN order_products b
        ON a.order_id = b.order_id
        AND a.product_id < b.product_id   -- avoid duplicates and self-pairs
    GROUP BY a.product_id, b.product_id
    HAVING COUNT(DISTINCT a.order_id) >= 50
),
product_order_counts AS (
    SELECT product_id, COUNT(DISTINCT order_id) AS total_orders
    FROM order_products
    GROUP BY product_id
)
SELECT
    pa.product_name                   AS product_a_name,
    pb.product_name                   AS product_b_name,
    pp.co_occurrences,
    -- co-occurrence rate: co-occurrences / orders containing either product
    ROUND(
        pp.co_occurrences * 1.0
        / (poc_a.total_orders + poc_b.total_orders - pp.co_occurrences),
        4
    )                                 AS jaccard_similarity
FROM product_pairs pp
JOIN products pa ON pp.product_a = pa.product_id
JOIN products pb ON pp.product_b = pb.product_id
JOIN product_order_counts poc_a ON pp.product_a = poc_a.product_id
JOIN product_order_counts poc_b ON pp.product_b = poc_b.product_id
ORDER BY pp.co_occurrences DESC
```

</details>

---

---

# Targeted Drill Problems

These are standalone problems — no big schema to set up, just a table definition and a question. Each one targets a specific concept. Do them without looking at the answers first. If you get it wrong, understand *why* before moving on.

---

## Section 1 Drills — Filtering & Aggregation

**Table:** `transactions(txn_id, user_id, merchant_id, amount, status, created_at)`
Statuses: `'APPROVED'`, `'DECLINED'`, `'REVERSED'`

---

**Drill 1.1**

Return the top 3 merchants by total approved transaction volume. If two merchants are tied on volume, order them alphabetically by `merchant_id`.

<details>
<summary>Answer</summary>

```sql
SELECT
    merchant_id,
    SUM(amount) AS total_volume
FROM transactions
WHERE status = 'APPROVED'
GROUP BY merchant_id
ORDER BY total_volume DESC, merchant_id ASC
LIMIT 3
```

</details>

---

**Drill 1.2**

For each user, return their approval rate (approved / total) and their reversal rate (reversed / total). Only include users who have made at least 10 transactions total. Round both rates to 4 decimal places.

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    COUNT(*)                                                          AS total_txns,
    ROUND(COUNT(CASE WHEN status = 'APPROVED'  THEN 1 END) * 1.0
          / NULLIF(COUNT(*), 0), 4)                                   AS approval_rate,
    ROUND(COUNT(CASE WHEN status = 'REVERSED'  THEN 1 END) * 1.0
          / NULLIF(COUNT(*), 0), 4)                                   AS reversal_rate
FROM transactions
GROUP BY user_id
HAVING COUNT(*) >= 10
```

</details>

---

**Drill 1.3**

Return a single result row with four columns:
- `total_txns`: all transactions
- `total_approved_volume`: sum of approved amounts
- `pct_approved_by_count`: % of transactions that were approved (by count)
- `pct_approved_by_volume`: % of total volume that was approved

No `GROUP BY` — this is a single aggregate over the whole table.

<details>
<summary>Answer</summary>

```sql
SELECT
    COUNT(*)                                                              AS total_txns,
    SUM(CASE WHEN status = 'APPROVED' THEN amount ELSE 0 END)            AS total_approved_volume,
    ROUND(COUNT(CASE WHEN status = 'APPROVED' THEN 1 END) * 100.0
          / NULLIF(COUNT(*), 0), 2)                                       AS pct_approved_by_count,
    ROUND(SUM(CASE WHEN status = 'APPROVED' THEN amount ELSE 0 END) * 100.0
          / NULLIF(SUM(amount), 0), 2)                                    AS pct_approved_by_volume
FROM transactions
```

</details>

---

**Drill 1.4**

For each day of the week (Monday, Tuesday, etc.) compute the average transaction amount and the total transaction count. Order by day of week (Monday = 1 through Sunday = 7). Use `DAYOFWEEK()` or `DATE_FORMAT(created_at, 'EEEE')` in Spark.

<details>
<summary>Answer</summary>

```sql
SELECT
    DAYOFWEEK(created_at)                AS day_of_week_num,
    DATE_FORMAT(created_at, 'EEEE')      AS day_name,
    COUNT(*)                             AS txn_count,
    ROUND(AVG(amount), 2)                AS avg_amount
FROM transactions
GROUP BY DAYOFWEEK(created_at), DATE_FORMAT(created_at, 'EEEE')
ORDER BY day_of_week_num
```

</details>

---

**Drill 1.5 — The tricky one**

You want to find merchants where the ratio of declined volume to total volume is greater than 30% — but only consider merchants that have had at least 5 approved transactions. Return merchant_id, total transactions, decline rate, sorted by decline rate descending.

Think carefully: which filters go in `WHERE` and which in `HAVING`?

<details>
<summary>Answer</summary>

```sql
SELECT
    merchant_id,
    COUNT(*)                                                              AS total_txns,
    ROUND(SUM(CASE WHEN status = 'DECLINED' THEN amount ELSE 0 END) * 100.0
          / NULLIF(SUM(amount), 0), 2)                                    AS decline_rate_pct
FROM transactions
GROUP BY merchant_id
HAVING COUNT(CASE WHEN status = 'APPROVED' THEN 1 END) >= 5
   AND SUM(CASE WHEN status = 'DECLINED' THEN amount ELSE 0 END) * 1.0
       / NULLIF(SUM(amount), 0) > 0.30
ORDER BY decline_rate_pct DESC
```

Both conditions go in `HAVING` because both depend on aggregates. You cannot use `WHERE` here — the filter on `>= 5 approved` requires `COUNT` which doesn't exist until after `GROUP BY`.

</details>

---

## Section 2 Drills — JOINs

**Tables:**
- `users(user_id, company_id, country, created_at)`
- `companies(company_id, company_name, plan_tier)` — tiers: `'FREE'`, `'PAID'`, `'ENTERPRISE'`
- `transactions(txn_id, user_id, amount, status, created_at)`

---

**Drill 2.1**

Return all companies on the `'PAID'` or `'ENTERPRISE'` plan alongside the number of users they have and their total approved transaction volume. Include companies with zero users (show 0). Include companies whose users have zero approved transactions (show 0).

<details>
<summary>Answer</summary>

```sql
SELECT
    c.company_id,
    c.company_name,
    c.plan_tier,
    COUNT(DISTINCT u.user_id)                             AS user_count,
    COALESCE(SUM(CASE WHEN t.status = 'APPROVED'
                      THEN t.amount END), 0)              AS approved_volume
FROM companies c
LEFT JOIN users u ON c.company_id = u.company_id
LEFT JOIN transactions t ON u.user_id = t.user_id
WHERE c.plan_tier IN ('PAID', 'ENTERPRISE')
GROUP BY c.company_id, c.company_name, c.plan_tier
```

Two LEFT JOINs because you want all companies regardless of whether they have users, and all users regardless of whether they have transactions.

</details>

---

**Drill 2.2**

Find all pairs of users from the same company who are both from different countries. Return `user_a_id`, `user_b_id`, `company_id`. Avoid returning both `(A, B)` and `(B, A)`.

<details>
<summary>Answer</summary>

```sql
SELECT
    a.user_id    AS user_a_id,
    b.user_id    AS user_b_id,
    a.company_id
FROM users a
JOIN users b
    ON a.company_id = b.company_id
    AND a.user_id < b.user_id          -- deduplicate pairs
    AND a.country <> b.country         -- different countries
```

</details>

---

**Drill 2.3**

Find users who registered in the last 90 days AND have never made a transaction AND whose company is on the `'ENTERPRISE'` plan. These are enterprise users who onboarded but never activated.

<details>
<summary>Answer</summary>

```sql
SELECT u.user_id, u.created_at AS registered_at, c.company_name
FROM users u
JOIN companies c ON u.company_id = c.company_id
WHERE c.plan_tier = 'ENTERPRISE'
  AND u.created_at >= CURRENT_TIMESTAMP - INTERVAL 90 DAYS
  AND NOT EXISTS (
      SELECT 1 FROM transactions t WHERE t.user_id = u.user_id
  )
```

</details>

---

**Drill 2.4 — The ON vs WHERE trap**

Write a query that returns every company and their total approved spend in the last 30 days. Companies with no approved transactions in the last 30 days should show 0, not disappear.

Then write a second query that returns only companies that DO have at least one approved transaction in the last 30 days.

Explain why they require different structures.

<details>
<summary>Answer</summary>

```sql
-- Query 1: ALL companies, zero for those with no recent approved spend
SELECT
    c.company_id,
    c.company_name,
    COALESCE(SUM(t.amount), 0) AS approved_spend_30d
FROM companies c
LEFT JOIN transactions t
    ON c.company_id = (SELECT company_id FROM users WHERE user_id = t.user_id)  -- via users
    AND t.status = 'APPROVED'           -- in ON clause: keeps all companies
    AND t.created_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
GROUP BY c.company_id, c.company_name

-- Cleaner version joining through users:
SELECT
    c.company_id,
    c.company_name,
    COALESCE(SUM(t.amount), 0) AS approved_spend_30d
FROM companies c
LEFT JOIN users u ON c.company_id = u.company_id
LEFT JOIN transactions t
    ON u.user_id = t.user_id
    AND t.status = 'APPROVED'
    AND t.created_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
GROUP BY c.company_id, c.company_name

-- Query 2: ONLY companies with approved transactions in last 30 days
SELECT
    c.company_id,
    c.company_name,
    SUM(t.amount) AS approved_spend_30d
FROM companies c
JOIN users u ON c.company_id = u.company_id
JOIN transactions t ON u.user_id = t.user_id
WHERE t.status = 'APPROVED'
  AND t.created_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
GROUP BY c.company_id, c.company_name
```

Query 1 needs LEFT JOINs with filters in `ON` so companies with no matching transactions still appear. Query 2 can use INNER JOINs + WHERE because you only want companies that have matches.

</details>

---

## Section 3 Drills — Window Functions

**Table:** `transactions(txn_id, user_id, merchant_id, amount, status, created_at)`

---

**Drill 3.1**

For each transaction, show:
- The transaction's amount
- The user's running total spend (approved only, ordered by `created_at`)
- The user's all-time total approved spend
- The transaction's amount as a % of the user's all-time total

Only include approved transactions.

<details>
<summary>Answer</summary>

```sql
SELECT
    txn_id,
    user_id,
    created_at,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    )                                                     AS running_total,
    SUM(amount) OVER (PARTITION BY user_id)               AS all_time_total,
    ROUND(amount / SUM(amount) OVER (PARTITION BY user_id) * 100, 2) AS pct_of_total
FROM transactions
WHERE status = 'APPROVED'
```

</details>

---

**Drill 3.2**

Find each user's single largest transaction. If a user has two transactions with the same maximum amount, return both. Do not use `ROW_NUMBER` — use `RANK` or `DENSE_RANK` instead, and explain why.

<details>
<summary>Answer</summary>

```sql
WITH ranked AS (
    SELECT *,
        RANK() OVER (PARTITION BY user_id ORDER BY amount DESC) AS rnk
    FROM transactions
    WHERE status = 'APPROVED'
)
SELECT * FROM ranked WHERE rnk = 1
```

`RANK` is correct here because tied amounts should both be returned (they share rank 1). `ROW_NUMBER` would arbitrarily pick one of the tied transactions and drop the other.

</details>

---

**Drill 3.3**

For each user, compute the amount difference between each transaction and their previous transaction (ordered by `created_at`). Flag transactions where the amount increased by more than 50% vs the previous one as `'SPIKE'`, decreased by more than 50% as `'DROP'`, and everything else as `'NORMAL'`. Exclude the first transaction per user (no previous to compare).

<details>
<summary>Answer</summary>

```sql
WITH with_prev AS (
    SELECT *,
        LAG(amount) OVER (PARTITION BY user_id ORDER BY created_at) AS prev_amount
    FROM transactions
    WHERE status = 'APPROVED'
)
SELECT
    txn_id,
    user_id,
    created_at,
    amount,
    prev_amount,
    ROUND((amount - prev_amount) / prev_amount * 100, 1) AS pct_change,
    CASE
        WHEN amount > prev_amount * 1.5  THEN 'SPIKE'
        WHEN amount < prev_amount * 0.5  THEN 'DROP'
        ELSE 'NORMAL'
    END AS change_flag
FROM with_prev
WHERE prev_amount IS NOT NULL
```

</details>

---

**Drill 3.4**

Assign each transaction to a spend decile within its merchant (1 = top 10% by amount, 10 = bottom 10%). Then return the average amount per decile per merchant. Two steps: first assign deciles, then aggregate.

<details>
<summary>Answer</summary>

```sql
WITH deciles AS (
    SELECT
        txn_id,
        merchant_id,
        amount,
        NTILE(10) OVER (PARTITION BY merchant_id ORDER BY amount DESC) AS decile
    FROM transactions
    WHERE status = 'APPROVED'
)
SELECT
    merchant_id,
    decile,
    COUNT(*)          AS txn_count,
    ROUND(AVG(amount), 2) AS avg_amount,
    MIN(amount)       AS min_amount,
    MAX(amount)       AS max_amount
FROM deciles
GROUP BY merchant_id, decile
ORDER BY merchant_id, decile
```

</details>

---

**Drill 3.5 — The hardest one**

For each user, find the longest consecutive streak of days where they made at least one approved transaction. Return `user_id` and `longest_streak_days`.

Hint: use the gaps-and-islands pattern.

<details>
<summary>Answer</summary>

```sql
WITH daily AS (
    -- one row per user per day they transacted
    SELECT DISTINCT
        user_id,
        DATE(created_at) AS txn_date
    FROM transactions
    WHERE status = 'APPROVED'
),
islands AS (
    -- subtract row number from date to get a group label per streak
    SELECT
        user_id,
        txn_date,
        txn_date - INTERVAL (
            ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY txn_date)
        ) DAYS AS grp
    FROM daily
),
streaks AS (
    SELECT
        user_id,
        grp,
        COUNT(*) AS streak_length
    FROM islands
    GROUP BY user_id, grp
)
SELECT
    user_id,
    MAX(streak_length) AS longest_streak_days
FROM streaks
GROUP BY user_id
ORDER BY longest_streak_days DESC
```

</details>

---

## Section 4 Drills — CTEs

**Tables:** same as Section 2 (`users`, `companies`, `transactions`)

---

**Drill 4.1**

Using only CTEs (no subqueries), find users who:
- Are in a company on the `'ENTERPRISE'` plan
- Have made more than 10 approved transactions
- Have an approval rate below 70%

Return `user_id`, `company_name`, `total_txns`, `approved_count`, `approval_rate`.

<details>
<summary>Answer</summary>

```sql
WITH enterprise_users AS (
    SELECT u.user_id, c.company_name
    FROM users u
    JOIN companies c ON u.company_id = c.company_id
    WHERE c.plan_tier = 'ENTERPRISE'
),
user_stats AS (
    SELECT
        user_id,
        COUNT(*)                                                    AS total_txns,
        COUNT(CASE WHEN status = 'APPROVED' THEN 1 END)            AS approved_count,
        ROUND(COUNT(CASE WHEN status = 'APPROVED' THEN 1 END) * 1.0
              / NULLIF(COUNT(*), 0), 4)                            AS approval_rate
    FROM transactions
    GROUP BY user_id
)
SELECT
    eu.user_id,
    eu.company_name,
    us.total_txns,
    us.approved_count,
    us.approval_rate
FROM enterprise_users eu
JOIN user_stats us ON eu.user_id = us.user_id
WHERE us.total_txns > 10
  AND us.approval_rate < 0.70
ORDER BY us.approval_rate ASC
```

</details>

---

**Drill 4.2**

Using CTEs, compute a "company health score" defined as:
- +1 point if the company has more than 5 active users (users with at least 1 transaction)
- +1 point if average approval rate across all users is above 80%
- +1 point if total approved volume in the last 30 days is above $10,000

Return `company_id`, `company_name`, and `health_score` (0–3).

<details>
<summary>Answer</summary>

```sql
WITH active_users AS (
    SELECT company_id, COUNT(DISTINCT user_id) AS active_user_count
    FROM users u
    WHERE EXISTS (SELECT 1 FROM transactions t WHERE t.user_id = u.user_id)
    GROUP BY company_id
),
approval_rates AS (
    SELECT
        u.company_id,
        AVG(
            COUNT(CASE WHEN t.status = 'APPROVED' THEN 1 END) * 1.0
            / NULLIF(COUNT(*), 0)
        ) OVER (PARTITION BY u.company_id)   -- window to get company-level avg
        AS avg_approval_rate
    FROM users u
    JOIN transactions t ON u.user_id = t.user_id
    GROUP BY u.company_id, u.user_id
),
-- simpler: just aggregate directly
company_approval AS (
    SELECT
        u.company_id,
        AVG(CASE WHEN t.status = 'APPROVED' THEN 1.0 ELSE 0 END) AS avg_approval_rate
    FROM users u
    JOIN transactions t ON u.user_id = t.user_id
    GROUP BY u.company_id
),
recent_volume AS (
    SELECT
        u.company_id,
        SUM(CASE WHEN t.status = 'APPROVED' THEN t.amount ELSE 0 END) AS approved_30d
    FROM users u
    JOIN transactions t ON u.user_id = t.user_id
    WHERE t.created_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAYS
    GROUP BY u.company_id
)
SELECT
    c.company_id,
    c.company_name,
    (CASE WHEN COALESCE(au.active_user_count, 0) > 5  THEN 1 ELSE 0 END
   + CASE WHEN COALESCE(ca.avg_approval_rate, 0) > 0.80 THEN 1 ELSE 0 END
   + CASE WHEN COALESCE(rv.approved_30d, 0) > 10000    THEN 1 ELSE 0 END
    ) AS health_score
FROM companies c
LEFT JOIN active_users    au ON c.company_id = au.company_id
LEFT JOIN company_approval ca ON c.company_id = ca.company_id
LEFT JOIN recent_volume   rv ON c.company_id = rv.company_id
ORDER BY health_score DESC
```

</details>

---

## Section 5 Drills — NULLs

**Table:** `users(user_id, full_name, email, phone, company_id, referrer_id, created_at)`
- `phone` can be NULL
- `referrer_id` can be NULL (user was not referred)
- `full_name` can be NULL (legacy records)

---

**Drill 5.1**

Return each user's display name using this priority: `full_name` → email prefix (part before `@`) → `'Anonymous'`. Also return a `contact_method` column: `'phone'` if they have a phone, `'email'` otherwise.

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    COALESCE(
        full_name,
        SPLIT(email, '@')[0],     -- Spark: array index 0
        'Anonymous'
    )                             AS display_name,
    CASE
        WHEN phone IS NOT NULL THEN 'phone'
        ELSE 'email'
    END                           AS contact_method
FROM users
```

</details>

---

**Drill 5.2**

Count:
- Total users
- Users with a phone number
- Users without a phone number
- Users who were referred (non-NULL `referrer_id`)
- Users who were NOT referred

All in a single row, no `GROUP BY`.

<details>
<summary>Answer</summary>

```sql
SELECT
    COUNT(*)                                          AS total_users,
    COUNT(phone)                                      AS has_phone,
    COUNT(*) - COUNT(phone)                           AS no_phone,
    COUNT(referrer_id)                                AS was_referred,
    COUNT(*) - COUNT(referrer_id)                     AS not_referred
FROM users
```

Key insight: `COUNT(column)` skips NULLs, so `COUNT(*) - COUNT(column)` gives you the NULL count without any `CASE WHEN`.

</details>

---

**Drill 5.3**

Find users who referred other users (their `user_id` appears in someone else's `referrer_id`). For each referrer, show how many users they referred. Include referrers even if all their referred users have since been deleted (handle NULLs carefully).

<details>
<summary>Answer</summary>

```sql
SELECT
    r.user_id              AS referrer_id,
    r.full_name            AS referrer_name,
    COUNT(u.user_id)       AS referred_count
FROM users r
JOIN users u ON r.user_id = u.referrer_id
GROUP BY r.user_id, r.full_name
ORDER BY referred_count DESC
```

Note: if you want to include referrers with 0 current referrals (referred users were deleted), you'd need a separate table tracking referral events, not just the current `referrer_id` column.

</details>

---

**Drill 5.4 — NULL in aggregation**

Given `transactions(txn_id, user_id, amount, fee)` where `fee` is NULL for transactions with no fee:

Write a query that returns per user:
- `avg_fee_when_present`: average fee only for transactions that had a fee (NULLs excluded)
- `avg_fee_treating_null_as_zero`: average fee treating NULL as 0
- `fee_coverage_rate`: % of transactions that had a fee at all

Explain why `avg_fee_when_present` and `avg_fee_treating_null_as_zero` will always differ unless coverage is 100%.

<details>
<summary>Answer</summary>

```sql
SELECT
    user_id,
    AVG(fee)                                                       AS avg_fee_when_present,
    AVG(COALESCE(fee, 0))                                          AS avg_fee_treating_null_as_zero,
    ROUND(COUNT(fee) * 100.0 / NULLIF(COUNT(*), 0), 2)            AS fee_coverage_rate
FROM transactions
GROUP BY user_id
```

They differ because `AVG(fee)` divides by the count of non-NULL rows only, while `AVG(COALESCE(fee, 0))` divides by all rows — including the zero-fee ones. If 50% of transactions have no fee, `avg_fee_treating_null_as_zero` will be roughly half of `avg_fee_when_present`.

</details>

---

## Mixed Drills — All Concepts Combined

These require you to pull from multiple sections in one query.

---

**Mixed Drill 1**

**Tables:** `users(user_id, company_id)`, `transactions(txn_id, user_id, amount, status, created_at)`

For each company, find the single user with the highest total approved spend. If two users in the same company are tied, return both. Return `company_id`, `user_id`, `total_spend`, `rank_in_company`.

<details>
<summary>Answer</summary>

```sql
WITH user_spend AS (
    SELECT
        u.company_id,
        t.user_id,
        SUM(t.amount) AS total_spend
    FROM transactions t
    JOIN users u ON t.user_id = u.user_id
    WHERE t.status = 'APPROVED'
    GROUP BY u.company_id, t.user_id
),
ranked AS (
    SELECT *,
        RANK() OVER (PARTITION BY company_id ORDER BY total_spend DESC) AS rank_in_company
    FROM user_spend
)
SELECT company_id, user_id, total_spend, rank_in_company
FROM ranked
WHERE rank_in_company = 1
```

</details>

---

**Mixed Drill 2**

**Tables:** `transactions(txn_id, user_id, amount, status, created_at)`

For each user, compute their month-over-month approved spend change for every month they transacted. Return `user_id`, `month`, `monthly_spend`, `prev_month_spend`, `mom_change_pct`. Only return rows where a previous month exists.

<details>
<summary>Answer</summary>

```sql
WITH monthly AS (
    SELECT
        user_id,
        DATE_TRUNC('month', created_at)  AS month,
        SUM(amount)                      AS monthly_spend
    FROM transactions
    WHERE status = 'APPROVED'
    GROUP BY user_id, DATE_TRUNC('month', created_at)
),
with_prev AS (
    SELECT *,
        LAG(monthly_spend) OVER (PARTITION BY user_id ORDER BY month) AS prev_month_spend
    FROM monthly
)
SELECT
    user_id,
    month,
    ROUND(monthly_spend, 2)       AS monthly_spend,
    ROUND(prev_month_spend, 2)    AS prev_month_spend,
    ROUND(
        (monthly_spend - prev_month_spend)
        / NULLIF(prev_month_spend, 0) * 100,
        2
    )                             AS mom_change_pct
FROM with_prev
WHERE prev_month_spend IS NOT NULL
ORDER BY user_id, month
```

</details>

---

**Mixed Drill 3**

**Tables:** `users(user_id, company_id, created_at)`, `transactions(txn_id, user_id, amount, status, created_at)`

A user is "churned" if they made at least one transaction in their first 30 days after signup, but zero transactions in the 60 days before today. Find all churned users. Return `user_id`, `first_txn_date`, `last_txn_date`, `days_since_last_txn`.

<details>
<summary>Answer</summary>

```sql
WITH user_activity AS (
    SELECT
        u.user_id,
        u.created_at                    AS signup_date,
        MIN(t.created_at)               AS first_txn_date,
        MAX(t.created_at)               AS last_txn_date,
        COUNT(CASE
            WHEN t.created_at <= u.created_at + INTERVAL 30 DAYS
            THEN 1
        END)                            AS txns_in_first_30d,
        COUNT(CASE
            WHEN t.created_at >= CURRENT_TIMESTAMP - INTERVAL 60 DAYS
            THEN 1
        END)                            AS txns_in_last_60d
    FROM users u
    JOIN transactions t
        ON u.user_id = t.user_id
        AND t.status = 'APPROVED'
    GROUP BY u.user_id, u.created_at
)
SELECT
    user_id,
    first_txn_date,
    last_txn_date,
    DATEDIFF(CURRENT_DATE, DATE(last_txn_date)) AS days_since_last_txn
FROM user_activity
WHERE txns_in_first_30d > 0
  AND txns_in_last_60d = 0
ORDER BY days_since_last_txn DESC
```

</details>

---

**Mixed Drill 4 — The hardest one**

**Tables:** `transactions(txn_id, user_id, amount, status, created_at)`, `users(user_id, company_id)`

For each company, bucket users into three tiers based on their total approved spend:
- `'HIGH'`: top 20% of spenders in the company
- `'MID'`: middle 60%
- `'LOW'`: bottom 20%

Then compute per company: the number of users in each tier, average spend per tier, and what % of total company spend each tier accounts for.

<details>
<summary>Answer</summary>

```sql
WITH user_spend AS (
    SELECT
        u.company_id,
        t.user_id,
        SUM(t.amount) AS total_spend
    FROM transactions t
    JOIN users u ON t.user_id = u.user_id
    WHERE t.status = 'APPROVED'
    GROUP BY u.company_id, t.user_id
),
tiered AS (
    SELECT *,
        NTILE(5) OVER (PARTITION BY company_id ORDER BY total_spend DESC) AS quintile
    FROM user_spend
),
with_tier AS (
    SELECT *,
        CASE
            WHEN quintile = 1           THEN 'HIGH'   -- top 20%
            WHEN quintile IN (2, 3, 4)  THEN 'MID'    -- middle 60%
            ELSE                             'LOW'    -- bottom 20%
        END AS spend_tier
    FROM tiered
)
SELECT
    company_id,
    spend_tier,
    COUNT(*)                                                     AS user_count,
    ROUND(AVG(total_spend), 2)                                   AS avg_spend,
    ROUND(
        SUM(total_spend) * 100.0
        / SUM(SUM(total_spend)) OVER (PARTITION BY company_id),
        2
    )                                                            AS pct_of_company_spend
FROM with_tier
GROUP BY company_id, spend_tier
ORDER BY company_id, spend_tier
```

Note: `SUM(SUM(total_spend)) OVER (PARTITION BY company_id)` — the inner `SUM` is the GROUP BY aggregate, the outer `SUM` is a window function over the grouped results. This is a nested aggregate + window pattern.

</details>

---

*End of Phase 1. Move to Phase 2 when you can solve all capstone and drill problems without referencing the answers.*
