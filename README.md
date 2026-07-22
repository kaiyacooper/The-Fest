# The Fest Database

<img width="200" height="200" alt="The Fest logo" src="https://github.com/user-attachments/assets/c61de144-30b6-4478-b558-4f7631cbcb93" />

A personal portfolio project — a relational MySQL database of **The Fest**, Gainesville, FL's independent underground punk festival (2002–2025), with a Power BI dashboard built on top.

> *"THE FEST is an independent multiple-day, multiple-venue underground music festival held annually in Gainesville, FL. Established in 2002 with only 60 bands, four stages, and two days, THE FEST has experienced a massive growth rate and now is three days long, 15+ venues and over 350 bands."* — thefestfl.com

---

## What's in the database

| Table | Rows | Description |
|---|---|---|
| `fest_editions` | 24 | All 23 editions + 2020 (cancelled). Dates, venue counts, notes |
| `venues` | 60 | Every venue used across all editions — name, type, capacity, years active |
| `concerts` | 27 | Day-level events for Fest 15–23 (2016–2025) |
| `bands` | 242 | Unique band master list with appearance counts + hometown city/lat/lon |
| `band_appearances` | 430 | Junction/fact table — one row per band per concert day |
| `band_origins` | 132 | Map-ready table — verified hometown city, state, country, latitude, longitude |
| `top_performers` | 17 | Most appearances per setlist.fm + Grooveist charts |
| `genres` | 10 | Genre breakdown from Concert Archives |

**Band-level data is complete for Fest 15–23 (2016–2025).** Earlier editions (Fest 1–14, 2002–2015) have edition metadata and venue history but not full band lineups. 132 of 242 bands have verified hometown and coordinates data sourced from MusicBrainz and Wikipedia.

---

## Files

| File | Description |
|---|---|
| `fest_schema_v3.sql` | Complete MySQL dump — `CREATE DATABASE`, all tables, INSERT data, 5 views, example queries |
| `the_fest_database_v3.xlsx` | Excel workbook with 9 colour-coded sheets including the map-ready `band_origins` sheet |
| `fest_data.py` | Raw Python data source covering concerts, venues, genres, editions |
| `band_hometowns.py` | Hometown + coordinates data for 132 bands (MusicBrainz / Wikipedia sourced) |
| `build_sql.py` | Generates `fest_schema_v3.sql` from the Python data sources |
| `build_excel.py` | Generates the Excel workbook |

---

## Quick start (MySQL)

```bash
# Import the full database
mysql -u root -p < fest_schema_v3.sql

# Connect and start querying
mysql -u root -p the_fest
```

---

## Example queries

**Who has played The Fest the most times?**
```sql
SELECT band_name, total_appearances, hometown_city,
       hometown_state, hometown_country, fest_editions
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

**Which US states sent the most bands?**
```sql
SELECT hometown_state, hometown_country,
       COUNT(*) AS band_count,
       SUM(total_appearances) AS total_appearances
FROM bands
WHERE hometown_country = 'USA'
  AND hometown_state IS NOT NULL
GROUP BY hometown_state, hometown_country
ORDER BY band_count DESC;
```

**Which city has produced the most Fest bands?**
```sql
SELECT hometown_city, hometown_state, hometown_country,
       COUNT(*) AS bands,
       SUM(total_appearances) AS total_appearances
FROM bands
WHERE hometown_city IS NOT NULL
GROUP BY hometown_city, hometown_state, hometown_country
ORDER BY bands DESC
LIMIT 15;
```

**International (non-US) bands:**
```sql
SELECT band_name, hometown_city, hometown_country, total_appearances
FROM bands
WHERE hometown_country != 'USA'
  AND hometown_country IS NOT NULL
ORDER BY hometown_country, total_appearances DESC;
```

**Gainesville locals vs touring bands:**
```sql
SELECT
  SUM(CASE WHEN hometown_city = 'Gainesville' THEN 1 ELSE 0 END) AS local_bands,
  SUM(CASE WHEN hometown_city != 'Gainesville'
           AND hometown_city IS NOT NULL THEN 1 ELSE 0 END) AS touring_bands
FROM bands;
```

**Festival growth over time:**
```sql
SELECT year, edition, num_stages, cancelled
FROM fest_editions
ORDER BY year;
```

**All map data (for BI tools):**
```sql
SELECT * FROM v_map_data;
```

**Bands with 3+ appearances (the Fest regulars):**
```sql
SELECT * FROM v_frequent_bands;
```

---

## Schema diagram

```
fest_editions ───────────────────────────────────┐
  edition (PK)                                   │
  year, name, start_date, end_date               │
  num_stages, cancelled, notes                   │
                                                 │
concerts ───────────────────────────────────┐   │
  concert_id (PK)                           │   │
  fest_edition → fest_editions.edition ─────┼───┘
  concert_name, day_label                   │
  concert_date, year, source_url            │
                                            │
band_appearances (junction / fact table)    │
  appearance_id (PK)                        │
  concert_id → concerts.concert_id ─────────┘
  band_id    → bands.band_id ───────────────┐
  fest_edition, band_name                   │
  concert_date, year                        │
                                            │
bands ─────────────────────────────────────┘
  band_id (PK)
  band_name (UNIQUE)
  total_appearances, fest_editions
  years_played, first_year, last_year
  hometown_city, hometown_state, hometown_country
  latitude, longitude

venues                 top_performers          genres
  venue_id (PK)          rank_position (PK)    genre_id (PK)
  name, type             band_name             genre
  capacity_tier          setlistfm_count       performance_count
  years_active, notes    grooveist_rank, notes

band_origins (map-ready view)
  band_name, hometown_city, hometown_state
  hometown_country, latitude, longitude
  total_appearances, first_year, last_year
```

---

## Pre-built views

| View | Purpose |
|---|---|
| `v_frequent_bands` | Bands with 3+ appearances, sorted by count |
| `v_edition_summary` | Each edition with stage count, duration, band count |
| `v_map_data` | All location-verified bands with lat/lon — ready for any BI tool |
| `v_bands_by_state` | Band count and total appearances grouped by US state |
| `v_bands_by_country` | USA vs Canada vs UK breakdown |

---

## Data sources

| Source | What it contributed |
|---|---|
| [Concert Archives](https://www.concertarchives.org/venues/the-fest) | Full band-by-band concert data, Fest 15–23 |
| [setlist.fm](https://www.setlist.fm/festivals/the-fest-63d6aad3.html) | Complete edition history (all 23 years), venues per year, artist appearance chart |
| [thefestfl.com](https://thefestfl.com/what-is-fest/) | Official alumni band list, Fest 23 attendance stats |
| [Grooveist](https://grooveist.com/lineups/the-fest-lineups/) | Most appearances leaderboard |
| [Wikipedia](https://en.wikipedia.org/wiki/The_Fest) | Historical context, early edition details |
| [MusicBrainz](https://musicbrainz.org/series/8487c823-b129-4d38-a677-400ae0531a27) | Band hometown and location data |

---

## Known gaps & extension ideas

- **Fest 1–14 (2002–2015):** Edition metadata is complete but full band lineups are not in this dataset. The [r/thefest community thread](https://www.reddit.com/r/thefest/comments/17tzn1u/) has compiled data for earlier years.
- **110 bands without location data:** Smaller or newer acts where hometown could not be verified with confidence. Honesty over guessing — these are excluded from the map sheet rather than approximated.
- **Venue ↔ concert linkage:** setlist.fm has individual venue-by-venue setlists per edition — a natural next step to add `venue_id` to each appearance row.
- **Setlists:** Actual songs played per band per show are tracked on setlist.fm.
- **Headliner flag:** Tagging headliners per day would enable richer analysis.
- **Attendance data:** Fest 23 is the only edition with a published attendance figure (9,197).

---

## Fun facts from the data

- **Most appearances (setlist.fm):** Timeshares (20), A Wilhelm Scream (18), The Copyrights (18)
- **Most stages ever:** Fest 19 (2021) and Fest 18 (2019) — both had **15 stages**
- **Fest 23 (2025) attendance:** 9,197 — 72% from outside Florida, 35% from outside the USA
- **Smallest edition:** Fest 7 (2008) — only 4 venues
- **Most represented city:** New Brunswick, NJ — home of Screaming Females, The Bouncing Souls, Thursday, Streetlight Manifesto, The Ergs!, and more
- **Gainesville locals:** Against Me!, Hot Water Music, Less Than Jake, Dikembe, Pool Kids, and Dollar Signs all call the host city home
- **UK acts:** The Murderburgers and You Vandal both made the trip from Edinburgh, Scotland. Nervus represented London.

---

*Built July 2026 as a data engineering portfolio project. Not affiliated with The Fest or No Idea Records.*
