# TV Rotator
 
Rotates Power BI (and other web) dashboards on the office TVs driven by **UniFi Connect** devices (UC Cast Pro).
Each TV loads one fixed URL; the schedule of which report shows on which TV, and when, lives in `config.json` in this repository.
 
**Live site:** `https://bdr-group-uk.github.io/tv-rotator/`
 
## How it works
 
```
UniFi Connect (Daily 0850)  ──►  UC Cast Pro (TV)  ──►  rotator.html?tv=<device-id>
                                                            │
                                                            ├─ reads config.json every 5 min
                                                            └─ shows the report scheduled for <device-id> at the current time
```
 
- `rotator.html` checks the clock every 30 seconds and switches report when the next time slot in `config.json` starts (every 10 minutes by default).
- Reports are loaded in an iframe and cross-faded, so the screen never goes blank between reports.
- Changes pushed to `config.json` reach the TVs within ~5 minutes. No UniFi changes needed.
- The page reloads itself at 04:00 to keep the Cast's memory clean.
## Source of the schedule
 
The rotation was first defined in `WallboardsRotation_v2.xlsx` (one column per TV, one row per 10-minute slot, plus a lookup table of report names and links). `config.json` was generated from that spreadsheet, and the rotator reads `config.json`.
 
## Files
 
| File | Purpose |
|---|---|
| `rotator.html` | The page every TV opens. No changes needed in normal use. |
| `config.json` | Report links and the schedule (time × TV). **This is the file you edit.** |
 
## config.json
 
```json
{
  "configReloadMinutes": 5,
  "devices": { "str-0f-dcp-10": "STR 0F DCP-10 | FINANCE DEPT" },
  "reports": {
    "Billing": "https://app.powerbi.com/view?r=..."
  },
  "schedule": [
    { "time": "08:50", "tvs": { "str-0f-dcp-10": "Billing" } }
  ]
}
```
 
| Key | Meaning |
|---|---|
| `configReloadMinutes` | How often the page re-reads `config.json`. Does **not** affect the rotation interval. |
| `devices` | Reference list of TV ids and labels (documentation only). |
| `reports` | Report name → URL. Each URL is defined once and reused by name. |
| `schedule` | List of time slots (`HH:MM`, 24h). From that time until the next slot, each TV in `tvs` shows the named report. A slot can also have `"default"` for TVs not listed. |
 
Before the first slot of the day the first slot is shown; after the last slot the last one stays on screen until the TVs are switched off.
 
## Common changes
 
**Change a report's link** – edit its URL in `reports`. Every slot using that name updates.
 
**Change what a TV shows at a given time** – in `schedule`, find the time and change the report name for that TV id.
 
**Add a report** – add `"New name": "https://..."` to `reports`, then use `"New name"` in `schedule`.
 
**Edit on GitHub:** open `config.json` → pencil icon → edit → *Commit changes*. Validate the JSON (e.g. jsonlint.com) before committing; invalid JSON stops updates (TVs keep the last valid config while the page is open).
 
## TV URLs (Stratford)
 
Each device has its own URL; only the `tv=` value changes.
 
| Device | URL |
|---|---|
| STR 0F DCP-06 IMT Dept Big | `https://bdr-group-uk.github.io/tv-rotator/rotator.html?tv=str-0f-dcp-06` |
| STR 0F DCP-07 IMT Dept Small | `...rotator.html?tv=str-0f-dcp-07` |
| STR 0F DCP-08 Business Support | `...rotator.html?tv=str-0f-dcp-08` |
| STR 0F DCP-09 Billing Dept | `...rotator.html?tv=str-0f-dcp-09` |
| STR 0F DCP-10 Finance Dept | `...rotator.html?tv=str-0f-dcp-10` |
| STR 0F DCP-11 Finance Big TV | `...rotator.html?tv=str-0f-dcp-11` |
| STR 0F DCP-12 Indi | `...rotator.html?tv=str-0f-dcp-12` |
| STR 1F DCP-01 Bahman's Office | `...rotator.html?tv=str-1f-dcp-01` |
| STR 1F DCP-02 Sales Dept TV | `...rotator.html?tv=str-1f-dcp-02` |
| STR 1F DCP-03 IT Support Dept TV | `...rotator.html?tv=str-1f-dcp-03` |
| STR 1F DCP-04 Telephony Dept TV | `...rotator.html?tv=str-1f-dcp-04` |
| STR 1F DCP-05 Malek's TV | `...rotator.html?tv=str-1f-dcp-05` |
| STR 1F DCP-22 GFB TV | `...rotator.html?tv=str-1f-dcp-22` |
 
## Testing
 
Add `&debug=1` to any TV URL and open it in a browser. A label in the bottom-right corner shows the TV id and the active report link. Never use `debug=1` on the real TVs.
 
## Troubleshooting
 
| Symptom | Likely cause / fix |
|---|---|
| Black screen, no text | JavaScript error. Open the URL in Chrome, F12 → Console. Non-ASCII characters in `rotator.html` caused this once; keep the file plain ASCII. |
| "Erro ao ler config.json" | `config.json` missing or invalid JSON. |
| "Sem relatorio para ..." | `tv=` id not in the schedule, or typo in the id. |
| A report shows blank | The site blocks iframes (check 8x8 links) or the Power BI *Publish to web* code was deleted. |
| TV stuck on one report | The device is still in an old "Daily HHMM" automation that overwrites its URL. Remove it (see SOP). |
 
## Notes
 
- Power BI reports use **Publish to web** links (`app.powerbi.com/view?r=...`): no login needed on the Cast, but anyone with the link can view them. Only use it for non-sensitive reports.
- This repository is public because GitHub Pages is free only for public repos. It contains only links that are already public.
 
