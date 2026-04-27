# SQL on HPC: Late-Night Fare Analysis

You're a data analyst at the NYC Taxi & Limousine Commission. The commission wants to know whether late-night taxi fares should be raised. To answer, you'll work with all 38 million yellow cab trips from 2023 — a 6.3 GB SQLite database that lives on TACC Vista (a supercomputer).

**Start here:** https://ashleyscruse.github.io/gosha-sql-assignment/

There are two pages on the site:

| Page | What it's for |
|------|---------------|
| [Walkthrough](https://ashleyscruse.github.io/gosha-sql-assignment/walkthrough.html) | Walks you through what we did in class, step by step. Use this if you missed the lecture, showed up late, or want to review before the homework. |
| [Homework](https://ashleyscruse.github.io/gosha-sql-assignment/homework.html) | Your assignment. Same dataset, different question. Has the prompt, sub-questions, what to submit, and how it's graded. |

---

## Quick start (on Vista)

The database is pre-built and shared. You don't need to download or build anything.

```bash
ssh USERNAME@vista.tacc.utexas.edu
cd $WORK
mkdir -p sql-demo/data
cd sql-demo
cp /work/10539/ashleyscruse/vista/gosha-sql-assignment/data/nyc_taxi.db data/
idev -p gg -m 60
sqlite3 data/nyc_taxi.db
```

Inside SQLite:

```sql
.mode column
.headers on
.tables
```

You should see `trips` and `zones`. You're ready to query.

---

## The database

**trips** (~38 million rows)

| Column | Type | Description |
|--------|------|-------------|
| pickup_time | TEXT | Pickup date and time |
| dropoff_time | TEXT | Dropoff date and time |
| passengers | INTEGER | Number of passengers |
| distance_miles | REAL | Trip distance |
| pickup_zone_id | INTEGER | Pickup location (zone ID) |
| dropoff_zone_id | INTEGER | Dropoff location (zone ID) |
| fare | REAL | Base fare amount |
| tip | REAL | Tip amount |
| total | REAL | Total charged |
| payment_type | INTEGER | 1=Credit card, 2=Cash, 3=No charge, 4=Dispute |

**zones** (70 rows, Manhattan neighborhoods)

| Column | Type | Description |
|--------|------|-------------|
| zone_id | INTEGER | Zone ID (matches pickup/dropoff zone IDs) |
| zone_name | TEXT | Neighborhood name |

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

---

---

*Instructors: a more general template version of this module lives at [sql-on-hpc](https://github.com/morehouse-supercomputing/sql-on-hpc). This repo holds the class-specific assignment, walkthrough, and shared database path.*
