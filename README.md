# SQL on HPC: Late-Night Fare Analysis

A guest lecture and homework module for a database systems course. Students play a data analyst at the NYC Taxi & Limousine Commission, working with all 38 million yellow cab trips from 2023 (a 6.3 GB SQLite database hosted on TACC Vista).

**Live student materials:** https://ashleyscruse.github.io/gosha-sql-assignment/

The site has two pages:

| Page | What it's for |
|------|---------------|
| [Walkthrough](https://ashleyscruse.github.io/gosha-sql-assignment/walkthrough.html) | Self-guided tutorial of the in-class investigation. Use if you missed the lecture, came late, or want to review at your own pace. |
| [Homework](https://ashleyscruse.github.io/gosha-sql-assignment/homework.html) | Graded assignment with sub-questions, submission requirements, and rubric. |

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

## For instructors adapting this

This repo is paired with the more general [sql-on-hpc template](https://github.com/morehouse-supercomputing/sql-on-hpc), which has the database setup script and a generic instructor guide. This repo holds the specific assignment + walkthrough used for this class.
