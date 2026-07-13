# The Fest Database

A personal portfolio project — a relational MySQL database of **The Fest**, Gainesville, FL's independent underground punk festival (2002–2025).

> *"THE FEST is an independent multiple-day, multiple-venue underground music festival held annually in Gainesville, FL. Established in 2002 with only 60 bands, four stages, and two days, THE FEST has experienced a massive growth rate and now is three days long, 15+ venues and over 350 bands."* — thefestfl.com

---

## What's in the database

| Table | Rows | Description |
|---|---|---|
| `fest_editions` | 24 | All 23 editions + 2020 (cancelled). Dates, venue counts, notes |
| `venues` | 60 | Every venue used across all editions — name, type, capacity, years active |
| `concerts` | 27 | Day-level events for Fest 15–23 (2016–2025) |
| `bands` | 242 | Unique band master list with appearance counts |
| `band_appearances` | 430 | Junction/fact table — one row per band per concert day |
| `top_performers` | 17 | Most appearances per setlist.fm + Grooveist charts |
| `genres` | 10 | Genre breakdown from Concert Archives |

**Band-level data is complete for Fest 15–23 (2016–2025).** Earlier editions (Fest 1–14, 2002–2015) have edition metadata and venue history but not full band lineups.

---

## Files

| File | Description |
|---|---|
| `fest_schema.sql` | Complete MySQL dump — `CREATE DATABASE`, all tables, all INSERT data, views, example queries |
| `the_fest_database_v2.xlsx` | Excel version with 8 colour-coded sheets, same data, for non-SQL analysis |
| `fest_data.py` | Raw Python data source (easy to extend) |
| `build_sql.py` | Script that generates `fest_schema.sql` from `fest_data.py` |
| `build_excel.py` | Script that generates the Excel file |

---

## Quick start (MySQL)

```bash
# Import the full database
mysql -u root -p < fest_schema.sql

# Connect and start querying
mysql -u root -p the_fest
```

---

## Example queries

**Who has played The Fest the most times?**
```sql
SELECT band_name, total_appearances, fest_editions, first_year, last_year
FROM bands
ORDER BY total_appearances DESC
LIMIT 15;
```

**How many unique bands per edition?**
```sql
SELECT fest_edition, year, COUNT(DISTINCT band_name) AS unique_bands
FROM band_appearances
GROUP BY fest_edition, year
ORDER BY fest_edition;
```

**Bands that played both Fest 15 (2016) AND Fest 23 (2025) — the true loyalists:**
```sql
SELECT DISTINCT a1.band_name
FROM band_appearances a1
JOIN band_appearances a2 ON a1.band_name = a2.band_name
WHERE a1.year = 2016 AND a2.year = 2025
ORDER BY a1.band_name;
```

**Which single concert day had the most bands?**
```sql
SELECT c.concert_name, c.concert_date, c.fest_edition,
       COUNT(DISTINCT a.band_name) AS band_count
FROM concerts c
JOIN band_appearances a USING (concert_id)
GROUP BY c.concert_id
ORDER BY band_count DESC
LIMIT 5;
```

**Festival growth over time (stages vs years):**
```sql
SELECT year, edition, num_stages, cancelled
FROM fest_editions
ORDER BY year;
```

**Bands with 3+ appearances (the Fest regulars):**
```sql
SELECT * FROM v_frequent_bands;
```

---

## 📐 Schema diagram

```
fest_editions ──────────────────────────────────┐
  edition (PK)                                  │
  year, name, start_date, end_date              │
  num_stages, cancelled, notes                  │
                                                │
concerts ──────────────────────────────────┐   │
  concert_id (PK)                          │   │
  fest_edition → fest_editions.edition ────┼───┘
  concert_name, day_label                  │
  concert_date, year, source_url           │
                                           │
band_appearances (junction / fact table)   │
  appearance_id (PK)                       │
  concert_id → concerts.concert_id ────────┘
  band_id    → bands.band_id ──────────────┐
  fest_edition, band_name                  │
  concert_date, year                       │
                                           │
bands ────────────────────────────────────┘
  band_id (PK)
  band_name (UNIQUE)
  total_appearances, fest_editions
  years_played, first_year, last_year

venues                      top_performers          genres
  venue_id (PK)               rank_position (PK)    genre_id (PK)
  name, type                  band_name             genre
  capacity_tier               setlistfm_count       performance_count
  years_active, notes         grooveist_rank, notes
```

---

## Data sources

| Source | What it contributed |
|---|---|
| [Concert Archives](https://www.concertarchives.org/venues/the-fest) | Full band-by-band concert data, Fest 15–23 |
| [setlist.fm](https://www.setlist.fm/festivals/the-fest-63d6aad3.html) | Complete edition history (all 23 years), venues per year, artist appearance chart |
| [thefestfl.com](https://thefestfl.com/what-is-fest/) | Official alumni band list, Fest 23 attendance stats |
| [Grooveist](https://grooveist.com/lineups/the-fest-lineups/) | Most appearances leaderboard |
| [Wikipedia](https://en.wikipedia.org/wiki/The_Fest) | Historical context, early edition details |
| [MusicBrainz](https://musicbrainz.org/series/8487c823-b129-4d38-a677-400ae0531a27) | External IDs, official links |

---

## Known gaps & extension ideas

- **Fest 1–14 (2002–2015):** Edition metadata is complete, but band-level data is not in this dataset. The [r/thefest Reddit thread](https://www.reddit.com/r/thefest/comments/17tzn1u/) has community-compiled data for earlier years.
- **Venue ↔ Concert linkage:** setlist.fm has individual venue-by-venue setlists per edition — could be used to add `venue_id` to each appearance row.
- **Setlists:** setlist.fm tracks actual songs played per band per show — a natural next step.
- **Headliners flag:** Could tag headliner bands per day for richer analysis.
- **Geography:** Band hometowns could be added from MusicBrainz API.

---

## Fun facts

- **Most appearances (setlist.fm):** Timeshares (20), A Wilhelm Scream (18), The Copyrights (18)
- **Most stages ever:** Fest 19 (2021) and Fest 18 (2019) — both had **15 stages**
- **Fest 23 (2025) attendance:** 9,197 — with 72% from outside Florida and 35% from outside the USA
- **Smallest edition:** Fest 7 (2008) — 4 venues only
- **Biggest band count:** Fest 12 (2013) — 400+ bands over 4 days, plus Big Pre-Fest in Ybor City

---

*Built July 2026 as a data engineering portfolio project. Not affiliated with The Fest or No Idea Records.*
