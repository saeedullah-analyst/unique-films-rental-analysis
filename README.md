# unique-films-rental-analysis
SQL Case Study: Post-merger operational audit and data integration for an entertainment niche acquisition scenario. SQL-Fallstudie: Post-Merger-Betriebsprüfung und Datenintegration für ein Akquisitions-Szenario in einer Mediennische.
# SQL Project: Unique Films Rental Case Study

## Profile & Core Competencies / Kompetenzprofil

### English Summary
*   **Project Goal:** Conducted a comprehensive post-merger operational data audit on the newly acquired Sakila Videos database to evaluate inventory scale, customer geography, and historical transaction logic.
*   **Technical Skills Applied:** Multi-table Joins (`INNER`, `LEFT`), Advanced Grouping (`GROUP BY`), Data Virtualization (`CREATE VIEW`), Subqueries across multiple clauses, and Advanced Window Functions (`OVER`, `PARTITION BY`, `DENSE_RANK`).
*   **Business Value:** Built a reusable reporting view to bridge conflicting entity definitions and designed an analytical ranking model to isolate premium high-value customer cohorts.

### Deutsche Zusammenfassung
*   **Projektziel:** Durchführung einer umfassenden Post-Merger-Betriebsprüfung der neu erworbenen Sakila Videos-Datenbank zur Bewertung von Bestandskonfigurationen, Kundenverteilung und Transaktionslogik.
*   **Angewandte Hard Skills:** Multi-Tabellen-Joins (`INNER`, `LEFT`), fortgeschrittene Gruppierung (`GROUP BY`), Datenvirtualisierung (`CREATE VIEW`), Subqueries über mehrere Klauseln und fortgeschrittene Window-Funktionen (`OVER`, `PARTITION BY`, `DENSE_RANK`).
*   **Geschäftlicher Nutzen:** Erstellung eines wiederverwendbaren Reporting-Views zur Bereinigung widersprüchlicher Entitäten und Entwicklung eines analytischen Ranking-Modells zur Isolierung wertvoller Kundenkohorten.

---

## 1. Project Overview & M&A Scenario
This project evaluates an independent data integration case study simulating a typical Mergers & Acquisitions (M&A) scenario. The acquiring firm, **Unique Films Rental**, is a global distributor specializing in rare, independent films. Following the retirement of the owner of friendly competitor **Sakila Videos**, Unique Films Rental acquired the company. 

Acting as the lead Data Analyst, my objective is to perform a deep exploratory dive into the inherited Sakila relational schema to audit data infrastructure assets, mapping out baseline operational logic for executive leadership before complete platform integration.

## 2. Technical Stack & SQL Concepts
*   **Database Engine:** PostgreSQL / MySQL (Python Notebook Interface Integration)
*   **Core SQL Competencies:** Relational Integrity Verification, Multi-table Data Merging, CTE Multi-stage Architectures, Windowing Over Partitions, View Engineering.

---
## 3. Data Exploration & Inherited Operational Audits
---

## 3. Structural Data Exploration & Table Auditing

Before running transactional data queries, I performed a structural schema audit to identify all database assets inherited from the Sakila Videos infrastructure.

### Task 1: Comprehensive Database Inventory Check
**Business Question:** Wie lasse ich mir eine Liste aller verfügbaren Tabellen anzeigen? (How do I retrieve a complete list of all available tables in the inherited database schema?)

```sql
SHOW TABLES;
```

*   **Execution Output:** 16 Tables detected.
*   **Business Takeaway:** Finding 16 distinct relational tables confirms that Sakila Videos runs a fully modular database infrastructure. This baseline list serves as the inventory checklist for auditing separate business sectors (such as customers, inventory, and payments) before initiating the core data migration.
### Aufgabe 1: Core Entity Structural Audit (`customer` Architecture)
**Business Question:** Lass dir die eine detaillierte Übersicht über die Tabelle `customer` anzeigen, die die Spaltennamen und ihre Datentypen sowie Primär- und Fremdschlüssel auflistet. (Extract a complete structural schema overview of the `customer` table, auditing columns, data formatting types, and indexing keys.)

```sql
DESC customer;
```

*   **Key Column Definitions Audited:**
    *   `customer_id`: Typ `smallint unsigned` | Key: `PRI` (Primary Key) | Extra: `auto_increment`
    *   `store_id`: Typ `tinyint unsigned` | Key: `MUL` (Foreign Key Index bridge to store branches)
    *   `first_name` & `last_name`: Typ `varchar(45)`
    *   `email`: Typ `varchar(50)`
    *   `address_id`: Typ `smallint unsigned` | Key: `MUL` (Foreign Key Index bridge to geographic locations)
    *   `active`: Typ `tinyint(1)` (Acts as an operational boolean flag for active/inactive users)
    *   `create_date` & `last_update`: Typ `datetime` & `timestamp`

*   **Business Takeaway:** Analyzing this structure proves that the customer database is highly normalized. The existence of multiple index keys (`MUL`) on columns like `store_id` and `address_id` ensures that we can quickly join customer profiles to geographical data layouts during platform integration without causing high processing stress or database slowdowns.
---

## 4. Fundamental Data Retrieval & Catalog Audits

### Task 3: Catalog Language Capabilities Check
**Business Question:** Wie frage ich die Spalte `name` der Tabelle `language` ab? (How do I extract all distinct operational languages supported in the inherited movie catalog database?)

```sql
SELECT name
FROM language;
```

*   **Execution Output (Complete Record Log):**
    *   English
    *   Italian
    *   Japanese
    *   Mandarin
    *   French
    *   German
*   **Business Takeaway:** This baseline query returns 6 distinct international languages supported by Sakila Videos. Verifying these text fields helps global expansion teams immediately confirm that the target company's media files match the localized linguistic requirements of our existing multi-region markets.
### Aufgabe 2: Specific Attributes vs. Table Wildcards (`SELECT *`)
**Business Question:** Schreibe zwei neue Queries. Mit der ersten Query fragst du die beiden Spalten `language_id` und `name` der Tabelle `language` ab. Mit der zweiten Query fragst du alle Spalten der Tabelle `language` ab. (Write two separate queries: one pulling explicitly targeted attributes, and a second pulling all structural columns using a wildcard modifier.)

**Query 2.1 (Targeted Selection):**
```sql
SELECT language_id, name
FROM language;
```

**Query 2.2 (System Wildcard Selection):**
```sql
SELECT * 
FROM language;
```

*   **Execution Metrics:** Both operations returned a compact matrix of 6 rows. Query 2.2 exposed the hidden tracking timestamp attribute `last_update` (Value: `2006-02-15`).
*   **Business Takeaway:** While `SELECT *` is convenient for raw inspection, enterprise data environments require explicitly defining target columns (as in Query 2.1). This habit minimizes unnecessary data pipeline load, optimizes server memory utilization, and speeds up analytical processing when tables scale up significantly.

---

### Codebeispiel III: Row-Level Constraints (`LIMIT`)
**Business Question:** Wie limitiere ich eine Datenausgabe auf eine überschaubare Anzahl an Zeilen? (How do I truncate data output results to prevent server memory overhead during large scale profiling?)

```sql
SELECT 
    customer_id, 
    first_name, 
    last_name
FROM customer
LIMIT 5;
```
### Aufgabe 3: Targeted Row Pagination (`LIMIT` with Row Offset)
**Business Question:** Schreibe eine Query, um die Spalten `customer_id`, `first_name` und `last_name` der Tabelle `customer` abzufragen. Lass dir nur die Zeilen 5 bis 7 vom Ergebnis ausgeben. (Formulate a query to extract customer identifiers and names, restricting the output exclusively to rows 5 through 7 by skipping the first four records.)

```sql
-- Utilizing the shorthand offset syntax (Skip 4 rows, return 3 rows)
SELECT 
    customer_id, 
    first_name, 
    last_name
FROM customer 
LIMIT 4, 3;
```

*   **Execution Output (Rows 5-7 Data Matrix):**
    *   5 | ELIZABETH | BROWN
    *   6 | JENNIFER | DAVIS
    *   7 | MARIA | MILLER
*   **Business Takeaway:** This approach demonstrates data pagination. Instead of loading an entire database table into memory, using offsets allows software systems and reporting pipelines to fetch data in small, highly optimized batches. It directly cuts down server processing costs and speeds up loading times for end-user applications.

---

### Phase 2: Cardinality & Uniqueness Audits (`SELECT DISTINCT`)
When exploring an inherited corporate database, identifying duplicate entry patterns and understanding the true number of unique attributes is critical. The `SELECT DISTINCT` modifier filters out redundant rows to reveal the exact unique categories present in a column.
### Aufgabe 4: Compliance & Authorization Audit (`SELECT DISTINCT`)
**Business Question:** Gab es Zahlungen, die von jemand anderem als den zwei uns bekannten Angestellten bearbeitet wurden? Gib die Antwort darauf, indem du die einzigartigen Werte der Spalte `staff_id` in der Tabelle `payment` abfragst. (Did any unauthorized or unknown personnel process customer transactions? Verify by extracting all unique employee identifiers recorded inside the transactional ledger.)

```sql
SELECT DISTINCT 
    staff_id 
FROM payment;
```

*   **Execution Output:**
    *   1
    *   2
*   **Business Takeaway & Audit Result:** The query returns a cardinality of exactly 2, showing only our known employee IDs (1 and 2). This successfully confirms that no ghost accounts, external payment processors, or data input anomalies have breached the sales ledger. System security and internal financial tracking are intact during the M&A transition phase.
---

## 5. Derived Business Logic & Currency Transformations

### Codebeispiel IV: Multi-Currency Exchange Calculations (`AS` Aliasing)
**Business Question:** Wie berechne ich Geldbeträge von US-Dollar in Euro und benenne die Spalten entsprechend? (How do I convert financial columns from USD to EUR inside a query using dynamic math operators and clear column aliases?)

```sql
SELECT 
    amount AS amount_usd, 
    amount * 0.85 AS amount_eur
FROM payment
LIMIT 5;
```

*   **Execution Output Sample (Top 5 Converted Lines):**
    *   2.99 USD | 2.5415 EUR
    *   0.99 USD | 0.8415 EUR
    *   5.99 USD | 5.0915 EUR
    *   0.99 USD | 0.8415 EUR
    *   9.99 USD | 8.4915 EUR
*   **Business Takeaway:** In multinational retail networks, processing raw source data in a uniform corporate currency is standard practice. Performing arithmetic operations directly inside the `SELECT` block and using clean aliases allows accounting teams to build multi-currency financial snapshots dynamically without running slow export routines or relying on manual spreadsheet formulas.
---

## 6. Descriptive Financial Aggregations

### Codebeispiel V: Transactional System Auditing (`COUNT`)
**Business Question:** Wie wird die Anzahl abgewickelter Zahlungen bestimmt? (How do I quantify the absolute volume of customer transactions processed across the entire payment ledger?)

```sql
SELECT 
    COUNT(amount) AS payment_count 
FROM payment;
```

*   **Execution Output:** 16,049 records
*   **Business Takeaway:** Finding 16,049 historical entries gives you a solid dataset to measure macro seasonal patterns, predict hardware capacity needs, and check how scalable the database will be before merging it into our core infrastructure.

### Aufgabe 5: Descriptive Sales Matrix (`AVG`, `MAX`, `MIN`)
**Business Question:** Gib den mittleren Zahlungsbetrag sowie die größte und kleinste Einzelzahlung aus. Berechne die Werte mit den Aggregationsfunktionen `AVG()`, `MAX()` und `MIN()`. Wähle sinnvolle Aliasse. (Extract structural pricing indicators by calculating the arithmetic mean, maximum charge value, and absolute baseline floor payment recorded in the system.)

```sql
SELECT 
    AVG(amount) AS payment_mean,
    MAX(amount) AS payment_max,
    MIN(amount) AS payment_min
FROM payment;
```

*   **Execution Output:**
    *   `payment_mean`: 4.200667
    *   `payment_max`: 11.99
    *   `payment_min`: 0.0
*   **Business Takeaway:** The average rental fee sits firmly around 4.20, with premium transactions topping out at 11.99. Crucially, the minimum payment boundary of `0.0` reveals the existence of free customer voucher promotions, complimentary employee rentals, or service adjustments. This requires a dedicated operational audit next to prevent revenue leakage.
---

## 7. Advanced Cardinality & Time-Series Granularity Extraction

### Codebeispiel VI: Pricing Tier Cardinality Audit (`COUNT(DISTINCT)`)
**Business Question:** Wie wird der DISTINCT-Modifikator in Kombination mit der Funktion COUNT() verwendet, um die Gesamtanzahl der Zahlungen mit den einzigartigen Zahlungsbeträgen zu vergleichen? (How do I isolate total transaction velocity against unique pricing point options available in the platform?)

```sql
SELECT 
    COUNT(amount) AS number_of_payments,
    COUNT(DISTINCT amount) AS number_of_different_amounts
FROM payment;
```

*   **Execution Output:**
    *   `number_of_payments`: 16,049 records
    *   `number_of_different_amounts`: 19 unique price options
*   **Business Takeaway:** While the ledger tracked 16,049 individual transaction sales records, they all fall under only 19 fixed price points. This indicates that Sakila Videos relies on a highly structured, standard pricing matrix (e.g., standard rental fees, premium movie charges, specific late fee tiers) rather than open or dynamic billing options.

---

### Codebeispiel VII: Chronological Feature Engineering (Time-Intelligence Parsing)
**Business Question:** Wie werden Zeit- und Datumsfunktionen genutzt, um unterschiedliche Informationen aus einem Zeitstempel zu extrahieren? (How do I isolate granular components from standard `DATETIME` timestamps to facilitate chronological pattern analysis?)

```sql
SELECT 
    payment_date,
    DATE(payment_date) AS date,
    YEAR(payment_date) AS Year,
    DAYNAME(payment_date) AS weekday,
    HOUR(payment_date) AS hour,
    MINUTE(payment_date) AS minute,
    SECOND(payment_date) AS second
FROM payment 
LIMIT 5;
```

*   **Execution Output Sample:**
    *   `payment_date`: 2005-05-25 11:30:37
    *   `date`: 2005-05-25 | `Year`: 2005 | `weekday`: Wednesday
    *   `hour`: 11 | `minute`: 30 | `second`: 37
*   **Business Takeaway:** Raw `DATETIME` fields are too dense for direct high-level analytics. Breaking a timestamp down into distinct components like `weekday` and `hour` allows us to perform precise hourly trend analysis. This data structure helps operations managers identify peak customer rental hours and optimize customer support shift scheduling during corporate integration.
---

## 8. Row-Level Filtering & Transactional Audits (`WHERE`)

### Codebeispiel VIII: Targeted Customer Lifecycle Extraction
**Business Question:** Wie ermittle ich alle Zahlungen einer Person anhand einer bestimmten Kundennummer? (How do I isolate and extract the full historical ledger of transactions processed for a single specific customer identifier?)

```sql
SELECT *
FROM payment
WHERE customer_id = 5;
```

*   **Execution Metrics:** Returned a complete transactional footprint of 38 distinct records for Customer ID 5.
*   **Business Takeaway:** Isolating individual customer timelines via the `WHERE` clause is critical for high-touch customer support diagnostics, fraudulent account reviews, or customer loyalty calculations. It lets us trace exactly when a specific client transacted, which clerk (`staff_id`) helped them, and how their individual order amounts fluctuated over time.
### Aufgabe 6: Fiscal Year Volume Aggregation (Time-Filtered Auditing)
**Business Question:** Wie viele Zahlungen wurden im Jahr 2006 getätigt? Schreibe eine Query, die nach dem Jahr 2006 filtert und die Anzahl an Zahlungen ausgibt. (Quantify the total transactional payment volume processed specifically during the 2006 fiscal year operational window.)

```sql
SELECT 
    COUNT(amount) AS payment_count_2006
FROM payment
WHERE YEAR(payment_date) = 2006;
```

*   **Execution Output:** 182 records
*   **Business Takeaway:** Isolating yearly transactional volumes lets the business measure growth or contraction over time. Seeing only 182 transactions in 2006 compared to the massive historical ledger baseline indicates that the system data for 2006 might be a partial-year log, or that operations slowed down right before the business acquisition took place.

---

### Phase 3: Advanced Boolean Logic & Multi-Condition Filtering
To run precise data audits, simple single-value filters are often not enough. Combining multiple rules using logical operators (`AND`, `OR`, `XOR`, `NOT`) allows us to narrow down specific business anomalies while managing operator priority through clean bracket isolation.
### Codebeispiel IX: Advanced Boolean Logic & Operator Precedence
**Business Question:** Ermittle Anzahl und Mittelwert der Zahlungen, deren Betrag größer als 8 Dollar ist und die entweder am Wochenende oder nicht im Juli erfolgten. (Extract the transaction count and average expenditure for high-value sales above $8 that occurred either during a weekend or outside the month of July.)

```sql
SELECT 
    COUNT(amount) AS payment_count,
    AVG(amount) AS average_amount
FROM payment
WHERE amount > 8 
  AND (MONTHNAME(payment_date) != 'July' OR DAYNAME(payment_date) IN ('Saturday', 'Sunday'));
```

*   **Execution Output:** 607 records | Average Amount: 9.584679
*   **Business Takeaway:** This query shows how to handle operator precedence using parentheses. By forcing the database to evaluate the date exceptions (`OR`) together before checking the pricing threshold (`AND`), we prevent data leaks and make sure our seasonal high-value customer tracking models are completely accurate.

### Aufgabe 7: Targeted Multi-Entity Verification (Text-Based Filtering)
**Business Question:** Welche Stammfiliale haben die Kundinnen SUSAN WILSON und MARIA MILLER? Lass dir die Spalten `store_id`, `first_name` und `last_name` aus der Tabelle `customer` ausgeben und filtere nach Vor- und Nachnamen. (Identify the designated home store branch location for specific customers Susan Wilson and Maria Miller by running a dual-attribute filter.)

```sql
SELECT 
    store_id, 
    first_name, 
    last_name
FROM customer
WHERE (first_name = 'SUSAN' AND last_name = 'WILSON')
   OR (first_name = 'MARIA' AND last_name = 'MILLER');
```

*   **Execution Output Matrix:**
    *   1 | MARIA | MILLER
    *   2 | SUSAN | WILSON
*   **Business Takeaway:** Using explicit logic combinations allows an analyst to lookup and audit specific high-value customer records instantly across different retail locations. The results reveal that Maria Miller is tied to Store Branch 1, while Susan Wilson is serviced by Store Branch 2, helping CRM teams assign localized customer success tasks properly during platform integration.

---

### Phase 4: Handling System Missing Values (`NULL` Management)
In database systems, missing or unrecorded information is represented by a `NULL` state. A common technical mistake is trying to filter these entries using standard comparison operators (like `= NULL`), which fails because a comparison against an unknown value always returns an unknown result. To handle these safely, specialized keywords like `IS NULL` or `IS NOT NULL` must be used instead.
### Codebeispiel X: Auditing Data Gaps via Non-Evaluative Assertions (`IS NULL`)
**Business Question:** Wie werden fehlende Werte in einer Tabelle korrekt identifiziert und warum schlagen herkömmliche Vergleichsoperatoren fehl? (Demonstrate how standard assignment operators fail when targeting unrecorded data states, and execute the correct syntax to locate missing entries.)

**Query 10.1 (Incorrect Syntax - Returns 0 Rows):**
```sql
SELECT *
FROM payment
WHERE rental_id = NULL;
```

**Query 10.2 (Correct Syntax - Returns 5 Rows):**
```sql
SELECT *
FROM payment
WHERE rental_id IS NULL;
```
*   **Execution Metrics & Anomalies Detected:** Query 10.1 failed to evaluate, returning an empty set. Query 10.2 successfully located exactly 5 transactional entries where the `rental_id` was completely absent (`None` / `NULL`), including Payment IDs `424`, `7011`, `10840`, `14675`, and `15458`.
*   **Business & Financial Audit Insight:** Discovering records with an empty `rental_id` inside a core sales ledger is a critical financial anomaly. It reveals that customers were charged fees (ranging from $0.99 to $3.99) without an associated physical film checkout log being generated. In an M&A scenario, identifying these records alerts the data integration team to system bugs or manual override actions by specific clerks (`staff_id` 1 and 2) that require immediate operational reconciliation.


*   **Execution Output Sample (Top 5 Consumer Records):**
    *   1 | MARY | SMITH
    *   2 | PATRICIA | JOHNSON
    *   3 | LINDA | WILLIAMS
    *   4 | BARBARA | JONES
    *   5 | ELIZABETH | BROWN
*   **Business Takeaway:** In large-scale systems, running unconstrained requests on multi-million row transaction tables can cause system bottlenecks or crashes. Applying the `LIMIT` clause allows an analyst to safely scan record layouts, confirm naming patterns, and verify row integrity before executing computationally heavy queries.
