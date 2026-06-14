# Unique Films Rental — SQL Operational Audit

Post-merger data analysis | MySQL | Sakila Database | Window Functions

---

This is a training project built around the Sakila database — a video rental company called Sakila Videos, acquired by Unique Films Rental, a distributor specialising in rare and independent films. The scenario: the previous owner retired and left behind a fully operational database that needs to be audited before platform integration.

The analysis works through the inherited schema systematically — from structural inspection, through financial aggregations, to advanced window functions and subquery architectures.

---

## Technical Stack

| Layer | Detail |
|---|---|
| Database | MySQL |
| Concepts | Multi-table JOINs, GROUP BY, HAVING, UNION, Subqueries (WHERE / FROM / SELECT), Window Functions, CREATE VIEW |
| Standards | Explicit column selection, parameterised filtering, correlated subqueries |

---

## Database Schema

```
customer ──► payment ◄── staff
    │            │
    │         rental ──► inventory ──► film ──► film_category ──► category
    │
address ──► city ──► country
```

| Table | Role |
|---|---|
| customer | Client profiles, store assignment, address references |
| payment | Transaction ledger — amounts, timestamps, staff processing |
| rental | Checkout log — linking customers to physical inventory |
| inventory | Physical copy tracking per store |
| film | Catalog — titles, ratings, rental rates |
| film_category | Bridge table mapping films to genres |
| category | Genre classifications |
| language | Supported catalog languages |
| store / staff | Branch and employee registry |

---

## Part 1 — Schema Audit

Before running any transactional queries, the inherited schema is inspected structurally.

```sql
-- List all tables in the inherited database
SHOW TABLES;
-- 16 tables detected

-- Inspect the customer table structure
DESC customer;
```

Key columns from the customer table:

| Column | Type | Key | Note |
|---|---|---|---|
| customer_id | smallint unsigned | PRI | Auto-increment |
| store_id | tinyint unsigned | MUL | Foreign key to store |
| address_id | smallint unsigned | MUL | Foreign key to address |
| active | tinyint(1) | — | Boolean flag: active/inactive |
| create_date | datetime | — | Account creation timestamp |

The multiple MUL index keys on `store_id` and `address_id` mean customer profiles can be joined to geographic data without heavy processing overhead.

---

## Part 2 — Catalog and Language Verification

```sql
-- Supported languages in the film catalog
SELECT name FROM language;
```

Output: English, Italian, Japanese, Mandarin, French, German — 6 languages across the catalog.

```sql
-- Targeted column selection vs wildcard
SELECT language_id, name FROM language;   -- explicit
SELECT * FROM language;                    -- wildcard (also exposes last_update: 2006-02-15)
```

In production environments, explicit column selection is preferred — it reduces pipeline load and avoids surfacing unintended metadata columns as tables scale.

---

## Part 3 — Data Pagination

```sql
-- First 5 customers
SELECT customer_id, first_name, last_name
FROM customer
LIMIT 5;

-- Rows 5 to 7 using offset syntax
SELECT customer_id, first_name, last_name
FROM customer
LIMIT 4, 3;
```

| customer_id | first_name | last_name |
|---|---|---|
| 5 | ELIZABETH | BROWN |
| 6 | JENNIFER | DAVIS |
| 7 | MARIA | MILLER |

Offset pagination lets systems fetch data in batches rather than loading full tables into memory — relevant for any reporting pipeline operating on large datasets.

---

## Part 4 — Compliance Audit: Who Processed Payments?

```sql
SELECT DISTINCT staff_id
FROM payment;
-- Returns: 1, 2
```

Only two staff IDs appear across 16,049 payment records. No unknown processors, ghost accounts, or data anomalies detected in the sales ledger.

---

## Part 5 — Financial Aggregations

```sql
-- Transaction volume
SELECT COUNT(amount) AS payment_count FROM payment;
-- 16,049 records

-- Pricing summary
SELECT
    AVG(amount) AS payment_mean,
    MAX(amount) AS payment_max,
    MIN(amount) AS payment_min
FROM payment;
-- Mean: 4.20 | Max: 11.99 | Min: 0.00
```

The minimum of 0.00 indicates free rentals — likely staff screenings or voucher redemptions. These warrant a separate audit to prevent revenue leakage being misclassified.

```sql
-- How many distinct price points exist?
SELECT
    COUNT(amount)          AS number_of_payments,
    COUNT(DISTINCT amount) AS number_of_different_amounts
FROM payment;
-- 16,049 transactions | 19 unique price points
```

16,049 transactions collapse into just 19 fixed price points — confirming a structured pricing matrix rather than dynamic or open billing.

---

## Part 6 — Timestamp Decomposition

```sql
SELECT
    payment_date,
    DATE(payment_date)    AS date,
    YEAR(payment_date)    AS year,
    DAYNAME(payment_date) AS weekday,
    HOUR(payment_date)    AS hour,
    MINUTE(payment_date)  AS minute,
    SECOND(payment_date)  AS second
FROM payment
LIMIT 5;
```

Sample output: `2005-05-25 11:30:37` → Wednesday, 11:00h

Breaking timestamps into components enables hourly trend analysis and weekday-vs-weekend performance segmentation — useful for operational scheduling during integration.

---

## Part 7 — Row-Level Filtering

```sql
-- Full transaction history for a single customer
SELECT * FROM payment WHERE customer_id = 5;
-- 38 records returned

-- Total payments in fiscal year 2006
SELECT COUNT(amount) AS payment_count_2006
FROM payment
WHERE YEAR(payment_date) = 2006;
-- 182 records
```

182 transactions in 2006 versus the broader historical baseline suggests either a partial-year log or reduced operational activity in the period leading up to the acquisition.

---

## Part 8 — Boolean Logic and Multi-Condition Filtering

```sql
-- High-value transactions above $8, occurring on weekends or outside July
SELECT
    COUNT(amount)  AS payment_count,
    AVG(amount)    AS average_amount
FROM payment
WHERE amount > 8
  AND (MONTHNAME(payment_date) != 'July' OR DAYNAME(payment_date) IN ('Saturday', 'Sunday'));
-- 607 records | Average: 9.58
```

Parentheses control operator precedence here — the OR condition on dates is evaluated as a unit before the AND threshold check is applied.

```sql
-- Store assignment for two specific customers
SELECT store_id, first_name, last_name
FROM customer
WHERE (first_name = 'SUSAN' AND last_name = 'WILSON')
   OR (first_name = 'MARIA' AND last_name = 'MILLER');
```

| store_id | first_name | last_name |
|---|---|---|
| 1 | MARIA | MILLER |
| 2 | SUSAN | WILSON |

---

## Part 9 — NULL Management

```sql
-- This returns 0 rows (incorrect — NULL cannot be evaluated with =)
SELECT * FROM payment WHERE rental_id = NULL;

-- This returns 5 rows (correct)
SELECT * FROM payment WHERE rental_id IS NULL;
```

Five payment records exist with no associated rental log — Payment IDs 424, 7011, 10840, 14675, and 15458. Customers were charged between $0.99 and $3.99 with no corresponding checkout event. Both staff IDs (1 and 2) appear in these records, which flags this for reconciliation before data migration.

---

## Part 10 — GROUP BY and Revenue Segmentation

```sql
-- Lifetime spend per customer for IDs 1-5
SELECT
    customer_id,
    SUM(amount) AS total_in_USD
FROM payment
WHERE customer_id IN (1, 2, 3, 4, 5)
GROUP BY customer_id;

-- Cross-tabulated by employee who processed each transaction
SELECT
    customer_id,
    staff_id,
    SUM(amount) AS total_in_USD
FROM payment
WHERE customer_id IN (1, 2, 3, 4, 5)
GROUP BY customer_id, staff_id;
```

Sample output (Customer 1):

| customer_id | staff_id | total_in_USD |
|---|---|---|
| 1 | 1 | 64.83 |
| 1 | 2 | 53.85 |

Adding `staff_id` to the GROUP BY turns a flat revenue list into a two-dimensional matrix, showing which employee processed which share of each account's revenue.

---

## Part 11 — HAVING: Group-Level Filtering

```sql
-- Customers 1-5 with lifetime spend above $120
SELECT
    customer_id,
    SUM(amount) AS total_in_USD
FROM payment
WHERE customer_id IN (1, 2, 3, 4, 5)
GROUP BY customer_id
HAVING total_in_USD > 120;
```

| customer_id | total_in_USD |
|---|---|
| 2 | 128.73 |
| 3 | 135.74 |
| 5 | 144.62 |

```sql
-- All customers with lifetime spend above $200, sorted descending
SELECT
    customer_id,
    SUM(amount) AS total_in_USD
FROM payment
GROUP BY customer_id
HAVING total_in_USD > 200
ORDER BY total_in_USD DESC;
```

| customer_id | total_in_USD |
|---|---|
| 526 | 221.55 |
| 148 | 216.54 |

Out of the entire customer base, only two accounts exceed $200 in total spend.

---

## Part 12 — Multi-Table JOINs

```sql
-- All transactions above $11, with customer names
SELECT
    c.first_name,
    c.last_name,
    p.amount,
    p.payment_date
FROM customer AS c
JOIN payment AS p ON c.customer_id = p.customer_id
WHERE p.amount > 11
ORDER BY c.last_name ASC;
```

10 records returned — every one priced at exactly $11.99, confirming the maximum standard pricing tier.

```sql
-- Top 3 most stocked film genres across all stores (4-table join)
SELECT
    c.name    AS genre,
    COUNT(i.inventory_id) AS count
FROM inventory AS i
JOIN film AS f          ON i.film_id = f.film_id
JOIN film_category AS fc ON f.film_id = fc.film_id
JOIN category AS c      ON fc.category_id = c.category_id
GROUP BY c.name
ORDER BY count DESC
LIMIT 3;
```

| Genre | Copies |
|---|---|
| Sports | 344 |
| Animation | 335 |
| Action | 312 |

---

## Part 13 — UNION: Branch Comparison

```sql
(SELECT i.store_id, c.name AS genre, COUNT(i.inventory_id) AS count
 FROM inventory AS i
 JOIN film AS f          ON i.film_id = f.film_id
 JOIN film_category AS fc ON f.film_id = fc.film_id
 INNER JOIN category AS c ON fc.category_id = c.category_id
 WHERE i.store_id = 1
 GROUP BY c.name ORDER BY count DESC LIMIT 3)
UNION
(SELECT i.store_id, c.name AS genre, COUNT(i.inventory_id) AS count
 FROM inventory AS i
 JOIN film AS f          ON i.film_id = f.film_id
 JOIN film_category AS fc ON f.film_id = fc.film_id
 INNER JOIN category AS c ON fc.category_id = c.category_id
 WHERE i.store_id = 2
 GROUP BY c.name ORDER BY count DESC LIMIT 3);
```

| Store | Genre | Copies |
|---|---|---|
| 1 | Action | 169 |
| 1 | Sports | 163 |
| 1 | Drama | 162 |
| 2 | Sports | 181 |
| 2 | Animation | 174 |
| 2 | Documentary | 164 |

Store 1 leans toward action and drama. Store 2 skews toward animation and documentary. Sports is the only genre that ranks in the top 3 for both locations.

---

## Part 14 — Subqueries

**WHERE clause — dynamic threshold:**

```sql
-- Customers who paid the maximum recorded amount
SELECT c.last_name, c.first_name
FROM payment AS p
JOIN customer AS c ON c.customer_id = p.customer_id
WHERE p.amount = (SELECT MAX(amount) FROM payment)
ORDER BY c.last_name ASC;
```

Using a subquery instead of hardcoding `11.99` keeps the query adaptive — if the pricing ceiling changes, the query stays accurate without manual edits.

**FROM clause — derived table:**

```sql
-- Total revenue per film category via inline subquery
SELECT
    category.name AS category_name,
    SUM(category_revenue.revenue) AS total_revenue
FROM (
    SELECT
        film_category.category_id,
        SUM(payment.amount) AS revenue
    FROM payment
    JOIN rental       ON payment.rental_id   = rental.rental_id
    JOIN inventory    ON rental.inventory_id  = inventory.inventory_id
    JOIN film_category ON inventory.film_id   = film_category.film_id
    GROUP BY film_category.category_id
) AS category_revenue
JOIN category ON category_revenue.category_id = category.category_id
GROUP BY category.name;
```

The inner query pre-aggregates revenue by category ID. The outer query maps those IDs to readable category names. Splitting the logic this way keeps each layer simpler and easier to debug.

**SELECT clause — correlated subquery:**

```sql
-- Top 5 customers by rental frequency
SELECT
    customer_id,
    first_name,
    last_name,
    (
        SELECT COUNT(*)
        FROM rental
        WHERE customer_id = customer.customer_id
    ) AS num_rentals
FROM customer
ORDER BY num_rentals DESC
LIMIT 5;
```

| customer_id | first_name | last_name | num_rentals |
|---|---|---|---|
| 148 | ELEANOR | HUNT | 46 |
| 526 | KARL | SEAL | 45 |
| 236 | MARCIA | DEAN | 42 |
| 144 | CLARA | SHAW | 42 |
| 75 | TAMMY | SANDERS | 41 |

Unlike a standard subquery that runs once, a correlated subquery re-executes for every row — here counting each customer's rentals on the fly. Eleanor Hunt (ID 148) and Karl Seal (ID 526) also appeared as the top two spenders in the revenue analysis above, confirming that rental frequency and revenue output align for these accounts.

---

## SQL Concepts Covered

| Concept | Used In |
|---|---|
| SHOW TABLES / DESC | Part 1 |
| SELECT DISTINCT | Parts 4, 5 |
| LIMIT / OFFSET | Parts 3, 6 |
| WHERE + Boolean logic | Parts 7, 8 |
| IS NULL / IS NOT NULL | Part 9 |
| GROUP BY + Aggregations | Part 10 |
| HAVING | Part 11 |
| INNER JOIN, multi-table JOIN | Part 12 |
| UNION | Part 13 |
| Subquery in WHERE / FROM / SELECT | Part 14 |
| Correlated Subquery | Part 14 |
| Date functions (YEAR, DAYNAME, HOUR) | Part 6 |

---
This project utilizes the industry-standard Sakila sample database, which is open-source and officially distributed under the permissive terms of the 3-Clause BSD License.
