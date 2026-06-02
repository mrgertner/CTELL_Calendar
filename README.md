# CTE–Linked Learning Calendar 2026–27

Auto-generated visual calendar for the CTE–Linked Learning Department, Division of Instruction, LAUSD.

## How it works

1. **Edit the data** → `CTE-LL_Calendar_Events_2026-27.csv`
2. **Push to GitHub** → the Action runs `generate_cte_ll_calendar.py`
3. **View the dashboard** → open `CTE-LL_Calendar_2026-27.html` (or via GitHub Pages)

## CSV format

| Column | Description |
|---|---|
| `Date` | `YYYY-MM-DD` for single day, `YYYY-MM-DD:YYYY-MM-DD` for multi-day |
| `Status` | Blank, `Confirmed`, or `Hold` |
| `Team` | Emoji code: 🧰 NTST · 🐦 CTE · ▶️ LL · 📰 DE · 📗 WBL · 💰 G&F · 📧 All · 🏫 Special |
| `Event Name` | Event title |
| `Audience` | Target audience |
| `Lead Contact` | `LAST, FIRST` |
| `Notes` | Additional context |

## To update

Edit the CSV directly on GitHub (pencil icon) or locally and push. The workflow auto-generates and commits the updated HTML.

## Manual run

Go to **Actions** → **Generate CTE-LL Calendar** → **Run workflow**.
