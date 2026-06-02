#!/usr/bin/env python3
"""
generate_cte_ll_calendar.py
Reads CTE-LL_Calendar_Events_2026-27.csv and produces CTE-LL_Calendar_2026-27.html

Layout:
  - 13 months stacked in a single column (Jul 2026 – Jul 2027)
  - Each month: LEFT = event list  |  RIGHT = traditional day-grid
  - Event list format: "day# – Event Name emoji"
  - Legend: 🧰 NTST · 🐦 CTE · ▶️ LL · 📰 DE · 📗 WBL · 💰 G&F · 📧 ALL · 🏫 Special Projects
  - Confirmed (●) and Hold (◆) status indicators

Usage:
    python generate_cte_ll_calendar.py [csv_path] [html_path]
"""

import csv, sys, os, html as html_mod
from datetime import datetime, date, timedelta
from collections import defaultdict
from calendar import monthrange, monthcalendar

CSV_DEFAULT  = "CTE-LL_Calendar_Events_2026-27.csv"
HTML_DEFAULT = "CTE-LL_Calendar_2026-27.html"

# LAUSD brand
NAVY   = "#00237A"
GOLD   = "#F2A900"
WHITE  = "#FFFFFF"

# Team color map  →  (text-color, bg-color)
TEAM_META = {
    "🧰":  ("#1A4FA0", "#EAF0FB", "NTST"),
    "🐦":  ("#7A5C00", "#FFF7E0", "CTE"),
    "▶️":  ("#1A6B1A", "#E6F5E6", "LL"),
    "📰":  ("#6B1A8A", "#F5E6FA", "DE"),
    "📗":  ("#0A6060", "#E0F5F5", "WBL"),
    "💰":  ("#8A1A1A", "#FAE6E6", "G&F"),
    "📧":  ("#444444", "#F0F0F0", "ALL TEAMS"),
    "🏫":  ("#8A4000", "#FFF0E0", "Special Projects"),
}
DEFAULT_STYLE = ("#444444", "#F0F0F0")

# Fiscal year: Jul 2026 = index 0, Jun 2027 = index 11, Jul 2027 = 12 (overflow)
MONTHS = [
    (2026,7),(2026,8),(2026,9),(2026,10),(2026,11),(2026,12),
    (2027,1),(2027,2),(2027,3),(2027,4),(2027,5),(2027,6),
    (2027,7),
]
MONTH_LABELS = [
    "JULY 2026","AUGUST 2026","SEPTEMBER 2026","OCTOBER 2026","NOVEMBER 2026","DECEMBER 2026",
    "JANUARY 2027","FEBRUARY 2027","MARCH 2027","APRIL 2027","MAY 2027","JUNE 2027",
    "JULY 2027",
]
DOW = ["S","M","T","W","Th","F","S"]




# ── Parse CSV ──────────────────────────────────────────────────────────────────
def parse_events(csv_path):
    by_date = defaultdict(list)
    with open(csv_path, newline="", encoding="utf-8-sig") as f:
        for row in csv.DictReader(f):
            raw = row.get("Date","").strip()
            if not raw: continue
            if ":" in raw:
                parts = raw.split(":")
                try:
                    d = datetime.strptime(parts[0].strip(),"%Y-%m-%d").date()
                    end = datetime.strptime(parts[1].strip(),"%Y-%m-%d").date()
                except ValueError: continue
                while d <= end:
                    by_date[d].append(row)
                    d += timedelta(days=1)
            else:
                try:
                    by_date[datetime.strptime(raw,"%Y-%m-%d").date()].append(row)
                except ValueError: continue
    return by_date


def team_style(tc):
    tc = (tc or "").strip()
    for emoji,(fg,bg,_) in TEAM_META.items():
        if emoji in tc:
            return fg, bg
    return DEFAULT_STYLE


def status_badge(status):
    s = (status or "").strip().lower()
    if s == "confirmed":
        return '<span class="badge confirmed" title="Confirmed">●</span>'
    if s == "hold":
        return '<span class="badge hold" title="Hold">◆</span>'
    return ""


# ── Event list for one month ───────────────────────────────────────────────────
def month_event_list(year, month, by_date):
    items = []
    _, days_in_month = monthrange(year, month)
    for day in range(1, days_in_month+1):
        d = date(year, month, day)
        evts = by_date.get(d, [])
        # deduplicate multi-day events: only show on first day of range
        for e in evts:
            raw = e.get("Date","")
            if ":" in raw:
                start_str = raw.split(":")[0].strip()
                try:
                    start_d = datetime.strptime(start_str,"%Y-%m-%d").date()
                    if start_d != d:
                        continue  # only show on start day
                except ValueError:
                    pass
            items.append((day, e))

    if not items:
        return '<p class="no-events">—</p>'

    html_parts = []
    for day, e in items:
        name  = html_mod.escape(e.get("Event Name",""))
        team  = (e.get("Team") or "").strip()
        notes = html_mod.escape(e.get("Notes","") or "")
        lead  = html_mod.escape(e.get("Lead Contact","") or "")
        status = e.get("Status","")
        fg, bg = team_style(team)

        # Build tooltip
        tip_parts = []
        if lead:  tip_parts.append(f"Lead: {lead}")
        if notes: tip_parts.append(notes)
        tip = html_mod.escape(" · ".join(tip_parts)) if tip_parts else ""
        tip_attr = f' title="{tip}"' if tip else ""

        badge = status_badge(status)
        # multi-day range label
        raw_date = e.get("Date","")
        range_label = ""
        if ":" in raw_date:
            parts = raw_date.split(":")
            try:
                s = datetime.strptime(parts[0].strip(),"%Y-%m-%d").date()
                en = datetime.strptime(parts[1].strip(),"%Y-%m-%d").date()
                if s.month == en.month:
                    range_label = f"{s.day}–{en.day}"
                else:
                    range_label = f"{s.day}/{s.month}–{en.day}/{en.month}"
            except ValueError:
                range_label = str(day)
        else:
            range_label = str(day)

        html_parts.append(
            f'<div class="evt-row"{tip_attr}>'
            f'<span class="evt-day">{range_label}</span>'
            f'<span class="evt-pill" style="color:{fg};background:{bg};">'
            f'{badge}{name} {team}</span>'
            f'</div>'
        )

    return "\n".join(html_parts)


# ── Traditional day-grid for one month ────────────────────────────────────────
def month_grid(year, month, by_date):
    cal = monthcalendar(year, month)
    # header row
    hdr = "".join(f'<th>{d}</th>' for d in DOW)
    rows = [f'<thead><tr>{hdr}</tr></thead>']
    rows.append("<tbody>")
    for week in cal:
        cells = []
        for dow_i, day in enumerate(week):
            if day == 0:
                cells.append('<td class="empty"></td>')
                continue
            d = date(year, month, day)
            evts = by_date.get(d, [])
            cls = "grid-day"
            if dow_i in (0,6): cls += " wknd"
            if d == date.today(): cls += " today"
            dot_html = ""
            if evts:
                dots = ""
                seen_teams = set()
                for e in evts:
                    tc = (e.get("Team") or "").strip()
                    key = tc if tc else "other"
                    if key in seen_teams: continue
                    seen_teams.add(key)
                    fg,_ = team_style(tc)
                    dots += f'<span class="dot" style="background:{fg};"></span>'
                dot_html = f'<div class="dots">{dots}</div>'
            cells.append(f'<td class="{cls}"><span class="gday">{day}</span>{dot_html}</td>')
        rows.append(f'<tr>{"".join(cells)}</tr>')
    rows.append("</tbody>")
    return "\n".join(rows)


# ── Build full HTML ────────────────────────────────────────────────────────────
def generate_html(csv_path, html_path):
    by_date = parse_events(csv_path)
    ts = datetime.now().strftime("%B %d, %Y")

    # Legend
    legend_parts = []
    for emoji,(_,bg,label) in TEAM_META.items():
        legend_parts.append(f'<span class="leg-item">{emoji} <strong>{label}</strong></span>')
    legend_html = " &nbsp;·&nbsp; ".join(legend_parts)

    # Single-column month rows
    months_html = ""
    for i,(y,m) in enumerate(MONTHS):
        label = MONTH_LABELS[i]
        evts  = month_event_list(y, m, by_date)
        grid  = month_grid(y, m, by_date)
        months_html += f"""
        <div class="month-row" id="month-{i}">
            <div class="month-event-col">
                <div class="month-name-badge">{label}</div>
                <div class="event-list">{evts}</div>
            </div>
            <div class="month-grid-col">
                <table class="cal-grid">{grid}</table>
            </div>
        </div>\n"""

    html = f"""<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>2026–27 CTE–Linked Learning Calendar</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap" rel="stylesheet">
<style>
*, *::before, *::after {{ box-sizing: border-box; margin:0; padding:0; }}
body {{
    font-family: 'Poppins', sans-serif;
    font-size: 15px;
    background: #fff;
    color: #1a1a2e;
    line-height: 1.5;
}}

/* ── Cover ── */
.cover {{
    background: {NAVY};
    color: {WHITE};
    padding: 28px 52px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 24px;
    flex-wrap: wrap;
}}
.cover-left {{
    display: flex;
    align-items: center;
    gap: 20px;
}}
.cover-seal {{
    height: 80px;
    width: 80px;
    flex-shrink: 0;
}}
.cover-wordmark {{
    height: 28px;
    width: auto;
    display: block;
    margin-bottom: 8px;
    filter: brightness(0) invert(1);
}}
.cover-text {{
    display: flex;
    flex-direction: column;
}}
.cover-title {{
    font-size: 22px;
    font-weight: 700;
    letter-spacing: -0.3px;
    line-height: 1.2;
}}
.cover-title span {{ color: {GOLD}; }}
.cover-sub {{
    font-size: 12px;
    color: rgba(255,255,255,0.55);
    margin-top: 5px;
}}
.cover-right {{
    text-align: right;
}}
.cover-year {{
    font-size: 60px;
    font-weight: 700;
    color: {GOLD};
    line-height: 1;
    letter-spacing: -2px;
    opacity: 0.9;
}}

/* ── Legend bar ── */
.legend-bar {{
    background: #f7f8fc;
    border-bottom: 2px solid {NAVY};
    padding: 11px 52px;
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    position: sticky;
    top: 0;
    z-index: 50;
    box-shadow: 0 2px 6px rgba(0,0,35,0.06);
}}
.legend-bar .lbl {{
    font-weight: 700;
    color: {NAVY};
    margin-right: 4px;
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: .5px;
}}
.leg-item {{
    font-size: 12px;
    color: #444;
    white-space: nowrap;
}}
.status-note {{
    margin-left: auto;
    font-size: 11px;
    color: #666;
}}
.badge.confirmed {{ color: #1a7a1a; font-weight: 800; }}
.badge.hold      {{ color: #b02020; font-weight: 800; }}

/* ── Calendar body ── */
.calendar-body {{
    padding: 0 52px 48px;
    max-width: 1100px;
    margin: 0 auto;
}}

/* ── Each month row ── */
.month-row {{
    display: grid;
    grid-template-columns: 1fr 260px;
    gap: 24px;
    padding: 20px 0 22px;
    border-bottom: 2px solid {NAVY};
}}
.month-row:first-child {{
    border-top: 2px solid {NAVY};
}}

/* ── Event list ── */
.month-event-col {{
    min-width: 0;
}}
.month-name-badge {{
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 1.2px;
    text-transform: uppercase;
    color: {WHITE};
    background: {NAVY};
    display: inline-block;
    padding: 4px 14px;
    border-radius: 4px;
    margin-bottom: 12px;
}}
.event-list {{
    display: flex;
    flex-direction: column;
    gap: 4px;
}}
.evt-row {{
    display: flex;
    align-items: baseline;
    gap: 8px;
    line-height: 1.4;
    cursor: default;
}}
.evt-day {{
    font-size: 12px;
    font-weight: 700;
    color: {NAVY};
    min-width: 30px;
    text-align: right;
    flex-shrink: 0;
}}
.evt-pill {{
    font-size: 12px;
    padding: 2px 7px;
    border-radius: 4px;
    font-weight: 500;
    line-height: 1.45;
    word-break: break-word;
}}
.no-events {{
    color: #aaa;
    font-size: 13px;
    font-style: italic;
}}

/* ── Mini calendar grid ── */
.month-grid-col {{
    flex-shrink: 0;
    align-self: start;
    padding-top: 4px;
}}
.cal-grid {{
    width: 100%;
    border-collapse: collapse;
    table-layout: fixed;
}}
.cal-grid thead th {{
    font-size: 11px;
    font-weight: 600;
    color: #888;
    text-align: center;
    padding: 3px 2px 4px;
    border-bottom: 1px solid #ddd;
}}
.cal-grid tbody td {{
    height: 30px;
    text-align: center;
    vertical-align: middle;
    padding: 2px 1px;
    border: 0.5px solid #ebebeb;
    position: relative;
}}
.cal-grid tbody td.empty {{ background: #f9f9f9; }}
.cal-grid tbody td.wknd  {{ background: #f5f5f5; }}
.cal-grid tbody td.today {{ outline: 2px solid {GOLD}; outline-offset: -2px; }}
.gday {{
    font-size: 11px;
    font-weight: 500;
    color: #444;
    display: block;
    line-height: 1;
}}
.dots {{
    display: flex;
    justify-content: center;
    gap: 2px;
    margin-top: 2px;
}}
.dot {{
    width: 4px;
    height: 4px;
    border-radius: 50%;
    display: inline-block;
}}

/* ── Footer ── */
.footer {{
    border-top: 2px solid {NAVY};
    padding: 16px 52px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 11px;
    color: #666;
    flex-wrap: wrap;
    gap: 8px;
    max-width: 1100px;
    margin: 0 auto;
}}
.footer-left {{ font-weight: 600; color: {NAVY}; }}

/* ── Print ── */
@media print {{
    .legend-bar {{ position: static; box-shadow: none; }}
    .month-row {{ break-inside: avoid; page-break-inside: avoid; }}
    .calendar-body {{ padding: 0 20px 20px; max-width: none; }}
    .cover {{ padding: 20px; }}
    .footer {{ max-width: none; padding: 12px 20px; }}
}}
@media (max-width: 700px) {{
    .month-row {{ grid-template-columns: 1fr; }}
    .month-grid-col {{ display: none; }}
    .calendar-body {{ padding: 0 16px 24px; }}
    .cover {{ padding: 20px; }}
    .legend-bar {{ padding: 10px 16px; }}
}}
</style>
</head>
<body>

<div class="cover">
    <div class="cover-left">
        <img src="LAUSD_seal.svg" class="cover-seal" alt="LAUSD Seal">
        <div class="cover-text">
            <img src="LAUSD_wordmark_RGB.svg" class="cover-wordmark" alt="LAUSD">
            <div class="cover-title">CTE – Linked Learning<br><span>Department Calendar</span></div>
            <div class="cover-sub">Division of Instruction &nbsp;·&nbsp; Los Angeles Unified School District</div>
        </div>
    </div>
    <div class="cover-right">
        <div class="cover-year">2026–27</div>
        <div class="cover-sub" style="color:rgba(255,255,255,0.45);text-align:right;margin-top:6px;">Generated {ts}</div>
    </div>
</div>

<div class="legend-bar">
    <span class="lbl">Teams:</span>
    {legend_html}
    <span class="status-note">
        <span class="badge confirmed">●</span> Confirmed &nbsp;
        <span class="badge hold">◆</span> Hold
    </span>
</div>

<div class="calendar-body">
    {months_html}
</div>

<div class="footer">
    <span class="footer-left">CTE–Linked Learning Department · Division of Instruction · LAUSD</span>
    <span>Auto-generated from <code>CTE-LL_Calendar_Events_2026-27.csv</code> · {ts}</span>
</div>

</body>
</html>"""

    with open(html_path, "w", encoding="utf-8") as f:
        f.write(html)

    unique = set()
    for evts in by_date.values():
        for e in evts:
            unique.add((e.get("Date"), e.get("Event Name")))
    print(f"✅ Calendar ready: {html_path}")
    print(f"   {len(unique)} unique events across {len(by_date)} days")


if __name__ == "__main__":
    csv_path  = sys.argv[1] if len(sys.argv) > 1 else CSV_DEFAULT
    html_path = sys.argv[2] if len(sys.argv) > 2 else HTML_DEFAULT
    if not os.path.isfile(csv_path):
        print(f"❌ CSV not found: {csv_path}")
        sys.exit(1)
    generate_html(csv_path, html_path)
