---
layout: default
title: Walkthrough
---

[Home](./)  |  **Walkthrough**  |  [Homework](homework.html)

# Walkthrough: Should the Commission Raise Late-Night Fares?

A self-guided tutorial of what we did in class. If you missed the lecture, came late, or just want to review at your own pace, work through this page top to bottom and you'll see the same investigation we ran together.

By the end, you'll have:

- Logged into TACC Vista
- Switched from a login node to a compute node
- Run six SQL queries against a 38-million-row taxi database
- Saved your query results as CSV files
- Formed a defensible recommendation with the data

---

## The setup

You're a data analyst at the NYC Taxi & Limousine Commission. The commission is the regulator that sets the rules every yellow cab in NYC follows.

Your boss walks in and asks:

> "Should we raise late-night fares? Take a position with evidence by Friday."

**Late-night** = pickup time between 10 PM and 4:59 AM.

You have all of 2023's yellow cab trip data sitting in a SQLite database on the supercomputer at TACC.

---

## How an analyst actually thinks about this

Before touching SQL, work through these in your head:

1. **Clarify the question.** Whose interest is the commission weighing? Driver welfare? Revenue? Riders?
2. **Argue both sides on paper.** Decide what would change your mind *before* looking at data, otherwise you'll cherry-pick.
3. **Ask: what data could settle each argument?** Each argument becomes a question. Each question becomes a query.
4. **Run the queries, interpret the results.** SQL is the tool, not the point.
5. **Acknowledge what the data CAN'T tell you.** Honest analysts are explicit about the limits.

For this question, the data we have can answer:

- Are late-night trips a meaningful slice of the business?
- Where do they happen?
- How do they differ in fare, distance, tipping?

The data cannot answer:

- Driver welfare (no driver IDs, no hours worked, no take-home pay)
- Cash tips (payment_type 2 records $0 tips, biasing tip analysis)
- Rider demographics or income
- Demand elasticity (we only see current prices)
- Alternative transit / Uber-Lyft competition
- Outer boroughs (zones table is mostly Manhattan + airports)

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

**Output:**

```
count(*)
--------
38310226
```

That's 38 million rows in 2023.

### Query 2: What does a single trip look like?

```sql
.once query_2.csv
SELECT * FROM trips LIMIT 5;
```

**Output:**

```
pickup_time          dropoff_time         passengers  distance_miles  pickup_zone_id  dropoff_zone_id  fare  tip   total  payment_type
-------------------  -------------------  ----------  --------------  --------------  ---------------  ----  ----  -----  ------------
2023-01-01 00:32:10  2023-01-01 00:40:36  1.0         0.97            161             141              9.3   0.0   14.3   2
2023-01-01 00:55:08  2023-01-01 01:01:27  1.0         1.1             43              237              7.9   4.0   16.9   1
2023-01-01 00:25:04  2023-01-01 00:37:49  1.0         2.51            48              238              14.9  15.0  34.9   1
2023-01-01 00:03:48  2023-01-01 00:13:25  0.0         1.9             138             7                12.1  0.0   20.85  1
2023-01-01 00:10:29  2023-01-01 00:21:19  1.0         1.43            107             79               11.4  3.28  19.68  1
```

A few things to notice:

- `pickup_time` is stored as **text**, not a real timestamp. SQLite has no real timestamp type, so dates live as strings.
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

**Output:**

```
hour_of_day  num_trips
-----------  ---------
0            1088628
1            731321
2            483366
3            319641
4            217492
5            226411
6            532181
7            1044241
8            1446062
9            1632601
10           1773717
11           1925489
12           2090720
13           2157093
14           2311519
15           2371342
16           2374464
17           2581999
18           2704217
19           2416756
20           2153613
21           2151209
22           1994411
23           1581733
```

Peak: 6 PM (2.7M trips). Trough: 4 AM (217K). Late-night is real, but it's the minority.

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

**Output:**

```
time_period  num_trips  pct
-----------  ---------  ----
Daytime      31893634   83.3
Late-Night   6416592    16.7
```

Late-night is **16.7% of total business**. Not trivial, not dominant. Big enough to matter for any fare-change decision.

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

`pickup_zone_id` in trips is a number. To turn it into a neighborhood name, we look it up in the zones table. JOIN matches every trip to its zone by the shared ID column.

**Output:**

```
zone_name                      num_trips
-----------------------------  ---------
JFK Airport                    416574
East Village                   402029
West Village                   339860
Clinton East                   281816
Sutton Place/Turtle Bay South  254150
Lower East Side                236074
Greenwich Village South        230367
Midtown Center                 206353
Lincoln Square East            199624
Penn Station/Madison Sq West   191701
```

This is a **mixed picture**, not just bar crowd. JFK leads (red-eyes, international arrivals). Heavy entertainment districts follow (East/West Village, Lower East Side, Greenwich Village South). Penn Station is on the list (late train arrivals). Late-night riders are not a single population.

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

**Output:**

```
time_period  avg_fare  avg_distance  avg_tip
-----------  --------  ------------  -------
Daytime      19.51     4.04          3.52
Late-Night   19.60     4.32          3.51
```

This is the surprise. The numbers are **remarkably similar**. Late-night fares are 9 cents higher (0.5%), distances are 0.28 miles longer (7%), tips are 1 cent lower (essentially identical). The intuition that "late-night must be wildly different" does not hold in this data.

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

## The recommendation

What the data shows:

- Late-night is **16.7%** of total trips (6.4M of 38.3M)
- Concentrated in **JFK Airport, entertainment districts, and transit hubs** — not a single rider population
- Late-night vs daytime averages are nearly identical: **fare $19.60 vs $19.51, distance 4.32 vs 4.04 mi, tip $3.51 vs $3.52**

What the data CANNOT tell us (repeating from above because it matters):

- Driver welfare
- Cash tips (biased data)
- Rider demographics or income
- Demand elasticity
- Alternative transit / Uber-Lyft competition
- Outer boroughs

**Recommendation, based on this data alone: don't raise late-night fares.** The dollar metrics tell us late-night and daytime trips are basically the same. Whatever case there is for raising fares cannot be made on these averages — it would have to come from data we don't have (driver wait times, safety, supply/demand imbalances).

The lesson: an honest analyst is honest about what the data does and doesn't support. The data sometimes refuses to support the obvious answer. Your job is to be honest about that.

---

## Now do the homework

You've seen the worked example. The [homework](homework.html) gives you a different analyst question (airport pricing) on the same dataset. Same shape, different question. Apply the same thinking.
