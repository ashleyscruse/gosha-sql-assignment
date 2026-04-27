---
layout: default
title: Late-Night Fares
tagline: The investigation we ran in class
---

[Home](./)  |  **Walkthrough**  |  [Homework](homework.html)

This page walks you through the technical setup and the six queries we ran during the lecture. Use it to replicate the demo on your own machine if you missed class, came in late, or want to review the SQL before the homework.

The interpretation, recommendation, and the broader lessons from the data are intentionally not on this page. Those came from the lecture. If you missed them, come to office hours.

By the end of this page, you'll have:

- Logged into TACC Vista
- Switched from a login node to a compute node
- Run six SQL queries against a 38-million-row taxi database
- Saved your query results as CSV files

---

## The setup

You're a data analyst at the NYC Taxi & Limousine Commission. Your boss has asked you whether late-night fares should be raised. **Late-night** = pickup time between 10 PM and 4:59 AM.

You have all of 2023's yellow cab trip data sitting in a SQLite database on the supercomputer at TACC.

---

## Step 1: Get on Vista

```bash
ssh USERNAME@vista.tacc.utexas.edu
cd $WORK
mkdir -p sql-demo/data
cd sql-demo
cp /work/10539/ashleyscruse/vista/gosha-sql-assignment/data/nyc_taxi.db data/
```

The copy is about 30-60 seconds. The database is 6.3 GB.

## Step 2: Move to a compute node

The login node is shared. Don't run heavy queries there. Switch to a compute node:

```bash
idev -p gg -m 60
```

`-p gg` selects the CPU queue. `-m 60` gives you a 60-minute interactive session. When your prompt changes (e.g., `c123-456`), you're on a compute node.

## Step 3: Open the database

```bash
sqlite3 data/nyc_taxi.db
```

Then in the SQLite shell, set up readable output:

```sql
.mode column
.headers on
.tables
.schema trips
.schema zones
```

You'll see two tables: `trips` (38M rows) and `zones` (70-row Manhattan neighborhood lookup).

---

## Phase 1: Meet the data

**Question:** What are we actually working with?

### Query 1: How many trips total?

```sql
.once query_1.csv
SELECT COUNT(*) FROM trips;
```

Run it. The number you see is the total count of trips for all of 2023.

### Query 2: What does a single trip look like?

```sql
.once query_2.csv
SELECT * FROM trips LIMIT 5;
```

Run it. As you look at the result, notice two things:

- `pickup_time` is stored as **text**, not a real timestamp. SQLite has no real timestamp type, so dates live as strings. We'll need string surgery to extract the hour later.
- `payment_type` is coded: 1 = Credit card, 2 = Cash, 3 = No charge, 4 = Dispute.

---

## Phase 2: When do trips happen?

**Question:** Are late-night trips even a meaningful slice of business?

### Query 3: Trips by hour of day

```sql
.once query_3.csv
SELECT
    CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER) AS hour_of_day,
    COUNT(*) AS num_trips
FROM trips
WHERE pickup_time IS NOT NULL
GROUP BY hour_of_day
ORDER BY hour_of_day;
```

`SUBSTR(pickup_time, 12, 2)` grabs 2 characters at position 12, which is where the hour lives in `2023-01-15 22:30:00`. `CAST(... AS INTEGER)` converts that text into a number we can compare and group by.

This query takes longer than the previous two because it scans all 38 million rows and does string surgery on each. That's a good sign you're on a compute node — a laptop would struggle with this.

### Query 4: Late-night vs daytime split

```sql
.once query_4.csv
SELECT
    CASE
        WHEN CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER) >= 22
          OR CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER) < 5
        THEN 'Late-Night'
        ELSE 'Daytime'
    END AS time_period,
    COUNT(*) AS num_trips,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM trips), 1) AS pct
FROM trips
WHERE pickup_time IS NOT NULL
GROUP BY time_period;
```

`CASE WHEN ... THEN ... ELSE ... END` is SQL's if/else. We're labeling each trip Late-Night or Daytime based on the pickup hour. The subquery `(SELECT COUNT(*) FROM trips)` runs first to give us the total, then we divide by it for the percentage.

---

## Phase 3: Where do late-night trips concentrate?

**Question:** Who's actually riding at night?

### Query 5: Top 10 late-night pickup zones (with JOIN)

```sql
.once query_5.csv
SELECT z.zone_name, COUNT(*) AS num_trips
FROM trips t
JOIN zones z ON t.pickup_zone_id = z.zone_id
WHERE CAST(SUBSTR(t.pickup_time, 12, 2) AS INTEGER) >= 22
   OR CAST(SUBSTR(t.pickup_time, 12, 2) AS INTEGER) < 5
GROUP BY z.zone_name
ORDER BY num_trips DESC
LIMIT 10;
```

`pickup_zone_id` in trips is a number. To turn it into a neighborhood name, we look it up in the zones table. JOIN matches every trip to its zone by the shared ID column. `t` and `z` are aliases — short names that let us write `t.column` instead of `trips.column`.

---

## Phase 4: How are late-night trips different?

**Question:** In fare, distance, and tipping, are late-night trips a different beast?

### Query 6: Avg fare, distance, tip by time period

```sql
.once query_6.csv
SELECT
    CASE
        WHEN CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER) >= 22
          OR CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER) < 5
        THEN 'Late-Night'
        ELSE 'Daytime'
    END AS time_period,
    ROUND(AVG(fare), 2) AS avg_fare,
    ROUND(AVG(distance_miles), 2) AS avg_distance,
    ROUND(AVG(tip), 2) AS avg_tip
FROM trips
WHERE pickup_time IS NOT NULL
GROUP BY time_period;
```

`AVG(...)` averages a column across the rows in each group. `ROUND(..., 2)` keeps the output to two decimal places.

When you read this result, ask yourself: are late-night and daytime trips fundamentally different in dollars and distance, or not? The answer matters for the recommendation we're building.

---

## Save your work, hand it off

Now exit SQLite and look at the files we saved:

```sql
.quit
```

```bash
ls *.csv
column -t -s, query_4.csv | less
column -t -s, query_5.csv | less
column -t -s, query_6.csv | less
```

This is the package you'd hand to your team. An analysis is only useful if someone else can pick it up.

When you're done with your compute node:

```bash
exit  # leaves the idev session
```

---

## Now do the homework

You've now run the same six queries we ran together in class. The interpretation, recommendation, and what the data does and doesn't support are what we worked through in the lecture — if you missed any of that, come to office hours.

The [homework](homework.html) gives you a different analyst question (airport pricing) on the same dataset. Same techniques you just used here, applied to a new question.
