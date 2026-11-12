# TV Rotator – Project Documentation

**Owner:** Carlo (BDR Group – Data & Reporting)
**Scope:** Office TV dashboards managed in UniFi Connect (console *BDR TV Cloud Key*)
**Status:** Stratford migrated (13 TVs). KEN, HEM and COL still on the old automations.

## 1. Problem

Each office TV runs a UC Cast Pro managed in UniFi Connect. The report rotation (a different Power BI report every 10 minutes) was built from UniFi automations:

- 52 automations named `Daily 0850` … `Daily 1720` (one per 10-minute slot, weekdays), plus `Stratford TVs Off`.
- Each automation contained ~22 devices, each with its own hard-coded report URL.
- Result: roughly 1,100 URL entries. Changing one report meant editing many automations by hand.

The UniFi documentation and API were checked: there is **no import/export or template for automations**, and the official UniFi API does **not** cover UniFi Connect, so the configuration could not be automated through the API.

## 2. Solution

Move the rotation logic out of UniFi and into a small web page.

- Each TV is pointed **once** to its own URL: `rotator.html?tv=<device-id>`.
- The page reads `config.json` (report links + time × TV schedule) and switches report at each scheduled time.
- UniFi only keeps two automations per TV: `Daily 0850` (sets the rotator URL and powers on) and `Stratford TVs Off`.
- All future changes are made in `config.json` on GitHub.

## 3. Tools used

| Tool | Use |
|---|---|
| **UniFi Connect** (unifi.ui.com → BDR TV Cloud Key → Connect) | Device and automation management. |
| **Power BI – Publish to web** | Report links that open without login on the Casts (`app.powerbi.com/view?r=...`). Approved because the reports contain no sensitive data. |
| **GitHub Pages** (`bdr-group-uk.github.io/tv-rotator`) | Free hosting for `rotator.html` and `config.json`. Public repo. |
| **rotator.html** | Plain HTML/JS page; double-iframe cross-fade, 30 s clock check, config reload every 5 min, nightly reload at 04:00. |
| **WallboardsRotation_v2.xlsx** | Schedule prepared first: devices × time slots with the report link for each, plus the report-name lookup table. Source of the config. |
| **config.json** | Generated from `WallboardsRotation_v2.xlsx` (13 Stratford devices, 52 time slots 08:50–17:20, 19 reports). |
| **Claude + Claude in Chrome** | Built the files, generated the config from the spreadsheet, and automated the UniFi clicks (removing devices from automations, setting URLs). |

## 4. What was done

1. **Schedule preparation** – the rotation was first reorganised in the spreadsheet `WallboardsRotation_v2.xlsx` so it reflected exactly how the rotation should run: one column per device, one row per 10-minute slot (08:50–17:20), each cell holding the report link, plus a lookup table of report names and links. This spreadsheet is the source for everything that follows.
2. **Analysis** – reviewed UniFi Connect docs and API; confirmed no bulk/import option.
3. **Built the rotator** – `rotator.html` + `config.json`. First version broke on GitHub because of accented characters; file was made pure ASCII.
4. **Config from spreadsheet** – converted `WallboardsRotation_v2.xlsx` (columns = devices, rows = times) into `config.json`, using the report names from the spreadsheet's lookup table.
5. **Hosting** – created the public repo `tv-rotator` and enabled GitHub Pages (branch `main`, root).
6. **Browser test** – `rotator.html?tv=...&debug=1` confirmed the correct report per time slot.
7. **Device test (1 h)** – DCP-10 removed from Daily 1100–1200 and pointed to the rotator; switched reports correctly every 10 minutes.
8. **Second device test** – DCP-06 (IMT Big, includes the 8x8 dashboards) tested successfully.
9. **Migration, one TV at a time** – DCP-10, DCP-02, DCP-03, DCP-05 and DCP-06 (DCP-01 done manually):
   - removed the device from every `Daily` automation except `Daily 0850`;
   - set the device's URL in `Daily 0850` to its rotator link (**Apply**, then **Save**);
   - set the same URL on the device itself.
10. **Migration, batch** – the remaining 7 TVs (DCP-07, 08, 09, 11, 12, 04, 22) were removed from all 51 `Daily` automations in a single pass (all selected together in *Manage*), then their links set in `Daily 0850` and on each device.
11. **Verification** – all 13 Stratford devices confirmed with the rotator link in `Daily 0850`; device URLs confirmed (DCP-04 was offline and will pick up the link from `Daily 0850`).

## 5. Current state

| Item | State |
|---|---|
| Stratford TVs (13) | Only in `Daily 0850` (rotator link) and `Stratford TVs Off`. |
| `Daily 0900`–`Daily 1720` | Still active; now contain only KEN, HEM and COL devices (9). |
| KEN / HEM / COL | Unchanged – still use the old per-slot automations. |
| DCP-04 Telephony | Offline during migration; URL is set in `Daily 0850`. |

## 6. Lessons learned

- In an automation's device panel, a URL change is only kept after clicking **Apply** and then **Save**.
- A device left in any old `Daily` automation gets its URL overwritten every 10 minutes.
- UniFi device lists are virtualised; scroll to the bottom to see/select the last devices.
- Keep `rotator.html` ASCII-only.
- Removing several devices at once per automation (*Manage* → select all → *Remove* → *Save*) is far faster than one device at a time.

## 7. Next steps

1. Get the rotation spreadsheets for KEN, HEM and COL.
2. Add their devices and schedules to `config.json`.
3. Test one TV per site.
4. Migrate the remaining devices, then disable the `Daily 0900`–`Daily 1720` automations (keep them disabled, not deleted, until stable).

## 8. Rollback

For any TV: put its old fixed Power BI URL back on the device and add it back to the `Daily` automations (or re-enable them). Keep a copy of `WallboardsRotation_v2.xlsx` as the source of the old links.
