# Homework: Should Airport Fares Be Priced Differently?

You're still a data analyst at the NYC Taxi & Limousine Commission. The commission has a follow-up question:

**Should airport fares be priced differently from non-airport fares?**

Same dataset (2023 NYC yellow cab trips). Different question. Take a position with evidence.

Airport zones: `pickup_zone_id IN (1, 132, 138)` — that's Newark, JFK, and LaGuardia.

---

## Sub-questions to guide your queries

Use these to break the big question into queries you can write. You don't have to answer them in this order, but each one helps build the case.

1. **How big a slice of the business are airport trips?** What's the count, and what percentage of total trips?
2. **How are airport trips different in fare, distance, and tip?** Compare averages between airport and non-airport trips.
3. **Are airport trips concentrated at certain hours of the day?** Show trips by hour, filtered to airport pickups.
4. **Are airport riders more likely to pay by credit card vs cash?** Compare payment type breakdown for airport vs non-airport.

You're welcome to ask additional questions of the data if they help your case.

---

## Submission

Submit a single ZIP or folder containing:

1. **The CSV files** of your query outputs (use `.once filename.csv` in SQLite to save each)
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
- Acknowledges what the data CAN'T tell you (driver welfare, demand elasticity, alternative transit, etc.)
- Suggests what additional data would strengthen the case

There's no single correct answer. You're graded on the quality of reasoning and use of evidence, not the direction of the recommendation.

---

## Reminders

- The database is at `/work/10539/ashleyscruse/vista/gosha-sql-assignment/data/nyc_taxi.db`
- Copy it to your own `$WORK` directory before querying
- Always run queries on a compute node (`idev -m 60`), never on the login node
- Use `.once query_N.csv` in SQLite to save each query's output to a CSV file
