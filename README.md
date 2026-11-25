# My Spotify Wrapped

A personal "Wrapped before Wrapped": my own Spotify listening year, modelled and tested with dbt and published as a dashboard, weeks before Spotify releases its version.

It's built from Spotify's free **Download your data** export. It uses the same stack as my [Strava running stats](https://github.com/Jnyambok/My-Strava-Running-Stats) project, and one of the marts joins my listening history to my runs.

> Not affiliated with or endorsed by Spotify. "Spotify" is a trademark of Spotify AB.

---

## The problem

Spotify Wrapped arrives once a year, in early December, and only shows what Spotify chooses to show. Your full listening history is already downloadable, but it arrives as raw JSON nobody reads. This project turns that export into a tested data model and a dashboard I can refresh whenever I want. It also answers questions Wrapped never asks: how often I skip, which songs I was obsessed with for a few weeks, and what I listen to while I run.

## Stack

| Step | Tool |
|---|---|
| Extract and load | Python (reads the export zip, loads raw tables) |
| Warehouse | BigQuery sandbox (free, no billing account) |
| Transform and test | dbt (staging → intermediate → marts) |
| Export | Python (marts → `docs/data.json`) |
| Dashboard | Static HTML, CSS and Chart.js on GitHub Pages |

## The data

**Account Data** export, requested in Settings → Privacy → Download your data (arrives in about 5 days):

| File | Contents | Used? |
|---|---|---|
| `StreamingHistory_music_*.json` | Every music play in the last 12 months: `endTime` (UTC), `artistName`, `trackName`, `msPlayed` | Yes, the core dataset |
| `StreamingHistory_podcast_*.json` | Podcast plays | Yes, a side stat |
| `Wrapped2025.json` | Spotify's own 2025 Wrapped figures | Yes, as a reconciliation test |
| `YourLibrary.json`, `Playlist1.json` | Liked songs and playlists | Yes, "saved vs actually played" |
| `Marquee.json` | Spotify's listener segment per artist | Maybe |
| `SearchQueries`, `MessageData`, `Inferences`, `Payments`, `Identifiers`, `Identity`, `UserAttributes` | Private account data | **Never loaded** |

**Extended Streaming History** has also been requested (it arrives in up to 30 days). It adds my full history since I joined, plus track IDs, real skip and shuffle flags, and platform. The staging layer is designed so those files slot in later without changing the marts.

### Limits of the account export

- There are only four fields per play. There's no album, genre, track ID or skip flag, so skips are defined as plays under 30 seconds.
- Timestamps are UTC, rounded to the minute, and they mark when a play *ended*. Staging converts them to Nairobi time.
- It covers only the last 12 months. The extended history fixes that.

## First look (export of 24 Nov 2026)

- **23,844 plays** between 24 Nov 2025 and 24 Nov 2026: **709 hours**, 4,628 tracks, 1,247 artists.
- **Top artists by hours:** Drake (92 h), Tory Lanez (51 h), Central Cee (33 h), Dave (24 h), Sus (19 h).
- **Top tracks by full plays:** Plot Twist by Drake (100), then The Color Violet by Tory Lanez (99).
- **Skip time:** 52% of plays ran past 90 seconds, 6% stopped between 30 and 90 seconds, and **42% stopped within 30 seconds**.
- **Busiest month:** April 2026 (2,789 plays). Listening peaks around 2 pm and 6 pm Nairobi time.

## What the dashboard will show

- **Header:** hours listened, plays, artists, tracks.
- **Top tracks table:** rank, track, artist, full plays, time listened.
- **Top artist panel:** the #1 artist with their most-played tracks.
- **Stream activity:** listening by weekday, with the busiest day highlighted. It has a day / week / month / year toggle.
- **Skip time:** a donut chart of plays under 30 s, 30–90 s and over 90 s.
- **Listening heatmap:** hour × weekday.
- **Obsessions:** tracks that spiked hard for a few weeks and then dropped off.
- **Monthly listening:** hours per month.
- **Saved but never played:** liked songs with zero plays.
- **Run soundtrack:** what plays during my Strava runs.

## Design

The look is based on a clean music-dashboard layout:

- White cards on a soft off-white background, with generous whitespace.
- A main column holding a hero header and a top tracks table with rounded rows; the highlighted row is tinted.
- A right-hand sidebar card stacking the artist panel, stream activity bars (rounded, with the top day filled in the accent colour) and the skip-time donut.
- One accent colour, used sparingly. Green replaces the reference's red.
- **A soft green translucent glow behind the dashboard that follows the cursor.** It's a radial gradient positioned with CSS variables set on `mousemove`. It stays still on touch devices and when the visitor prefers reduced motion.
- The export has no album art, so track and artist tiles use generated initials on gradients.

## Privacy

- The export zip and the unzipped folder are git-ignored and never committed.
- Only the streaming, podcast, library, playlist and Wrapped files are loaded. Searches, messages, inferences, payments and identifiers are never read.
- The public page shows aggregates: top artists and tracks, patterns and totals. It never shows a raw play-by-play log.

## Project structure (planned)

```
My-Spotify-Wrapped/
├── data/raw/              # export zip goes here (git-ignored)
├── extract/load.py        # zip -> BigQuery raw tables
├── spotify_dbt/           # dbt project: staging, intermediate, marts, tests
├── export/export_data.py  # marts -> docs/data.json
├── docs/                  # the dashboard (GitHub Pages)
└── pipeline.py            # load -> dbt build -> export, in one command
```

## Progress

### Phase 1: my own dashboard
- [x] Request the account data and extended streaming history exports
- [x] Profile the export and plan the build
- [ ] Set up the repo, venv and BigQuery dataset
- [ ] Load the streaming, podcast, library, playlist and Wrapped files to BigQuery
- [ ] dbt staging models (UTC → Nairobi, cleaned names, skip flag) with tests
- [ ] Intermediate and mart models (top tracks and artists, activity, skips, obsessions, monthly)
- [ ] Reconciliation test against Spotify's own Wrapped total
- [ ] Export to `docs/data.json`
- [ ] Build the dashboard (design above, cursor glow)
- [ ] Publish on GitHub Pages
- [ ] One-command `pipeline.py`

### Phase 2: run soundtrack
- [ ] Join plays to Strava runs by time window
- [ ] Run soundtrack section on the dashboard

### Phase 3: public upload tool (after Phase 1 ships)
- [ ] A site where anyone drops in their own export and gets their dashboard
- [ ] Runs entirely in the browser (DuckDB-WASM), so no file ever leaves the visitor's computer
- [ ] Its own name and branding, not Spotify's

### Later
- [ ] Switch to the extended streaming history when it arrives

## Key Attributes

*Filled in as the build goes: the decisions worth explaining in an interview.*
