# SQL Assignment: Late-Night Fare Analysis

You are a data analyst at the NYC Taxi & Limousine Commission. The commission is considering whether to adjust fare rates for late-night trips (10 PM to 5 AM). Your job is to analyze 2023 trip data and write a recommendation: should late-night fares increase, decrease, or stay the same?

## Getting Started

### Option 1: Build the database yourself

```bash
cd $WORK
git clone https://github.com/ashleyscruse/gosha-sql-assignment.git
cd gosha-sql-assignment
bash scripts/setup_data.sh
```

This downloads 12 months of NYC taxi data and loads it into a SQLite database. Takes about 10-15 minutes. Requires about 9 GB of disk space on `$WORK`.

### Option 2: Use the shared database

The database is available on Vista. Copy it into your repo:

```bash
cd $WORK
git clone https://github.com/ashleyscruse/gosha-sql-assignment.git
cd gosha-sql-assignment
mkdir -p data
cp /work/10539/ashleyscruse/vista/gosha-sql-assignment/data/nyc_taxi.db data/
```

### Open the database

```bash
sqlite3 data/nyc_taxi.db
```

To make output easier to read:

```sql
.mode column
.headers on
```

## The Database

**trips** (about 38 million rows)

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

## The Assignment

Work through the questions below in order. Write your SQL queries and record the results. You will use these results to write your final recommendation.

For this assignment, "late-night" means trips with a pickup time between 10 PM and 4:59 AM. Everything else is "daytime."

**Hint:** You can extract the hour from pickup_time like this:

```sql
CAST(SUBSTR(pickup_time, 12, 2) AS INTEGER)
```

---

### Part 1: Understanding the Data

Before making any recommendations, you need to understand what you're working with.

**Q1.** How many total trips are in the database?

**Q2.** How many trips are late-night (10 PM to 4:59 AM) vs daytime? What percentage of all trips are late-night?

**Q3.** What is the average fare, average tip, and average distance for late-night trips compared to daytime trips?

---

### Part 2: When and Where

Now look at patterns across time and location.

**Q4.** Break down the number of trips by hour of day. Which hours have the most and fewest trips?

**Q5.** What are the top 10 busiest pickup zones for late-night trips? Use a JOIN with the zones table to show neighborhood names.

**Q6.** How does average trip distance change by hour of day? Are late-night trips typically longer or shorter?

---

### Part 3: Revenue and Tipping

The commission cares about money. Dig into the financials.

**Q7.** What is the total revenue (sum of the total column) for late-night trips vs daytime trips?

**Q8.** Compare tipping behavior between late-night and daytime. What is the average tip and tip percentage (tip divided by fare) for each? Break this down by payment type (credit card vs cash).

**Q9.** Which late-night pickup zones generate the most total revenue? Show the top 10 with neighborhood names.

---

### Part 4: Patterns Worth Investigating

Look for things that might affect a fare adjustment decision.

**Q10.** Are late-night trips more likely to be paid by credit card or cash compared to daytime trips? Show the percentage breakdown for each.

**Q11.** Compare airport trips (zone IDs 132, 138, 1) to non-airport trips during late-night hours. How do they differ in average fare, distance, and tip?

**Q12.** Look at late-night trip volume by month. Are there seasonal patterns? Which months have the most and fewest late-night trips?

---

### Part 5: Your Recommendation

Using what you found in Parts 1 through 4, write a short recommendation (1 to 2 paragraphs) to the NYC Taxi & Limousine Commission.

Your recommendation should:

- State clearly whether late-night fares should increase, decrease, or stay the same
- Reference at least 3 specific findings from your queries
- Consider the impact on both drivers and riders
- Acknowledge any limitations in the data or your analysis

---

## Submission

Turn in:

1. A file with all 12 SQL queries and their results
2. Your written recommendation (Part 5)

## Reference

| Task | Command |
|------|---------|
| Open database | `sqlite3 data/nyc_taxi.db` |
| Count rows | `SELECT COUNT(*) FROM trips;` |
| Show tables | `.tables` |
| Show columns | `.schema trips` |
| Pretty output | `.mode column` then `.headers on` |
| Export to CSV | `.mode csv` then `.output results.csv` then run query |
| Exit SQLite | `.quit` |
