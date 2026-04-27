---
layout: default
title: SQL on HPC Homework
---

# Homework: Should Airport Fares Be Priced Differently?

You're a data analyst at the NYC Taxi & Limousine Commission. The commission has a follow-up question:

**Should airport fares be priced differently from non-airport fares?**

Same dataset (2023 NYC yellow cab trips). Different question. Take a position with evidence.

Airport zones: `pickup_zone_id IN (1, 132, 138)` — Newark, JFK, and LaGuardia.

---

## Step 1: Get on Vista

```bash
ssh USERNAME@vista.tacc.utexas.edu
cd $WORK
mkdir -p sql-homework/data
cd sql-homework
cp /work/10539/ashleyscruse/vista/gosha-sql-assignment/data/nyc_taxi.db data/
```

The copy takes 30-60 seconds.

## Step 2: Move to a compute node

Never run heavy queries on a login node — it's a shared resource. Always switch to a compute node first:

```bash
idev -p gg -m 60
```

`-p gg` selects the CPU queue (we don't need a GPU). `-m 60` gives you 60 minutes.

When your prompt changes (e.g., `c123-456`), you're on a compute node.

## Step 3: Open the database

```bash
sqlite3 data/nyc_taxi.db
```

Then make the output readable:

```sql
.mode column
.headers on
```

---

## Sub-questions to guide your queries

Use these to break the big question into queries you can write. You don't have to answer them in this order, but each one helps build the case.

### 1. How big a slice of the business are airport trips?

What's the count, and what percentage of total trips? Hint: use `CASE WHEN pickup_zone_id IN (1, 132, 138)` to bucket airport vs non-airport, then `COUNT(*)` and a subquery for the percentage.

### 2. How are airport trips different in fare, distance, and tip?

Compare averages. `AVG(fare)`, `AVG(distance_miles)`, `AVG(tip)` grouped by airport vs non-airport.

### 3. Are airport trips concentrated at certain hours of the day?

Filter to airport pickups, then `GROUP BY` hour. Use this pattern to extract the hour:

```sql
CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER)
```

### 4. Are airport riders more likely to pay by credit card vs cash?

Compare payment type breakdown for airport vs non-airport. Payment type codes: 1 = Credit card, 2 = Cash, 3 = No charge, 4 = Dispute.

You're welcome to ask additional questions of the data if they help your case.

---

## Save your query results

Before each query, tell SQLite to save the result to a CSV file:

```sql
.headers on
.mode csv
.once query_1.csv
SELECT ...your query...;
```

Repeat for each query (`query_2.csv`, `query_3.csv`, etc.). After you `.quit` SQLite, view them from the shell:

```bash
ls *.csv
column -t -s, query_1.csv | less
```

---

## Submission

Submit a single ZIP or folder containing:

1. **The CSV files** of your query outputs
2. **Your recommendation** (1-2 paragraphs) — should airport fares be priced differently? Cite specific numbers from your queries.
3. **A short reflection** (~150 words):
   - What finding surprised you?
   - What data did you wish you had?
   - When did HPC actually matter for this assignment, vs when it didn't?

---

## What good work looks like

A strong submission:

- Takes a clear position (yes / no / depends-on-X)
- Uses specific numbers from your queries to support the position
- Acknowledges what the data CAN'T tell you (driver welfare, demand elasticity, alternative transit, outer boroughs, etc.)
- Suggests what additional data would strengthen the case

There's no single correct answer. You're graded on the quality of reasoning and use of evidence, not the direction of the recommendation.

---

## Reference

| Task | Command |
|------|---------|
| Open database | `sqlite3 data/nyc_taxi.db` |
| Show tables | `.tables` |
| Show columns | `.schema trips` |
| Pretty output | `.mode column` then `.headers on` |
| Save next query as CSV | `.once filename.csv` then run query |
| View saved CSV | `column -t -s, filename.csv \| less` |
| Exit SQLite | `.quit` |
| Leave compute node | `exit` |
