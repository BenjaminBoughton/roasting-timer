# Roast Timer — Claude Code Handoff v2

## Overview

A mobile-first PWA for logging coffee roasts at a small Hawaiian coffee roastery. Built as a single HTML file with vanilla JS and localStorage — no backend, no build step, no framework.

The core concept: a roaster loads a **profile** (the ideal roast recipe) before starting, and during the roast the app guides them through each phase while they log what **actually happens**. After the roast, actual vs. expected is saved for future reference.

---

## Current State (v1)

A working v1 exists in `index.html`. It covers:
- Home, setup, timer, summary, history, and bean manager screens
- Event logging for First Crack, Second Crack, Drop with optional temp entry via numpad overlay
- Summary screen with computed stats (dev time, dev%, event timeline)
- localStorage persistence (`rt_beans`, `rt_roasts`)
- PWA manifest

**What v1 is missing entirely:** profile/recipe system, guided roasting, airflow cues, gas setting guidance, ramp down logic, batch-size awareness.

**Known placeholder:** `viewRoast()` uses `alert()` — needs a proper detail screen.

---

## V2 Goal

Replace the generic bean selector with a **profile system**. During a roast, the app acts as a **guide** — telling the roaster what they should be doing at each phase — while they log actuals. Think: recipe on the left, reality on the right.

---

## Roast Process — Universal Rules

Understanding the roast flow is critical for building the UI correctly.

### Airflow — 3 positions
1. **Through cooling bin** (marked X) — always the starting position
2. **50/50** — split between cooling bin and drum; transition happens at 300–320°F (roaster judges exact temp)
3. **Through roasting drum** (marked →) — always triggered at First Crack

### Gas scale
Physical dial gauge (inches water column, "WC), scale 0–16. Logged as circled numbers in the notebook. The app should display these as plain numbers (0–10).

### Universal starting rule (ALL profiles except Dark Roast)
- Gas starts at **0** at charge
- Airflow starts **through cooling bin**

### Timestamp logging
- **:00** — charge temp logged (drum temp when beans are loaded)
- **:30** — gas setting logged
- **1:00** — gas setting logged
- **300–320°F transition** — temp logged, gas adjusted, airflow → 50/50
- **Ramp down temps** — temp + gas logged at each step
- **First Crack** — time + temp logged, gas + airflow change
- **Second Crack** (dark roast only) — time + temp logged, gas change
- **Drop** — time + temp logged

### Ramp down logic (most profiles)
Starting at a profile-specific temp, decrease gas by 1 unit every 10°F increase. The goal is a controlled approach to first crack — targeting ~10°F per minute rise. If the rate is faster or slower, the roaster adjusts in real time.

### Post-first-crack (most profiles)
- Gas goes back up to 10 (or profile-specific target)
- Airflow switches to through roasting drum
- Roaster watches time + temp to decide drop

---

## All 11 Roast Profiles

### Universal fields for all non-dark-roast profiles
- Charge: gas 0, airflow through cooling bin
- 300–320°F: airflow → 50/50

---

### 1. 100% Kona
- **Batch:** 2750g
- **Charge temp:** 410°F
- **Gas at :30:** 5 | **Gas at 1:00:** 10
- **50/50 transition:** 300–320°F, gas stays 10
- **Ramp down:** starts 350°F → 9, 360°F → 8, 370°F → 6, 380°F → 5
- **First crack:** ~385–395°F → gas 10, airflow through drum
- **Drop target:** watch temp, ~15 min total
- **Second crack:** No

---

### 2. Light Roast
- **Batch:** 2200g
- **Charge temp:** 380°F
- **Gas at :30:** 4 | **Gas at 1:00:** 8
- **50/50 transition:** 300–320°F → gas 9
- **Ramp down:** starts 360°F → 8, 370°F → 7, 380°F → 6
- **First crack:** ~385–395°F → gas 10, airflow through drum
- **Drop target:** ~415°F, MAX 2:00–2:15 post first crack
- **Second crack:** No
- **Note:** ⚠️ Strictly enforce post-crack time limit. Lowest drop temp of all profiles — intentional.

---

### 3. Maui Mokka
- **Batch:** 2200g
- **Charge temp:** 380°F
- **Gas at :30:** 4 | **Gas at 1:00:** 8
- **50/50 transition:** 300–320°F → gas 9
- **Ramp down:** starts 350°F → 8, 360°F → 7, 370°F → 6, 380°F → 5
- **First crack:** → gas 10, airflow through drum
- **Drop target:** ~425°F, ~2:30 post first crack
- **Second crack:** No
- **Note:** ⚠️ Newest profile, subject to change. First crack very difficult to hear — extra attention required.

---

### 4. Waialua Peaberry
- **Batch:** 1100g (most variable batch size)
- **Charge temp:** 300°F
- **Gas at :30:** 3 | **Gas at 1:00:** 6
- **50/50 transition:** 300–320°F → gas 9
- **Ramp down:** starts 350°F → 7, 360°F → 6, 370°F → 5
- **First crack:** ~375°F (earlier than most — small batch + bean type) → gas 8, airflow through drum
- **Drop target:** ~4:00 post first crack, 425–430°F
- **Second crack:** No
- **Note:** Primary cue is time post-crack, adjusted if temp rising too fast/slow.

---

### 5. 100% Kau
- **Batch:** 1100g
- **Charge temp:** 300°F
- **Gas at :30:** 3 | **Gas at 1:00:** 6
- **50/50 transition:** 300–320°F → gas 9
- **Ramp down:** starts 350°F → 8, 360°F → 7, 370°F → 6, 380°F → 5
- **First crack:** ~390°F → gas 7, airflow through drum
- **Drop target:** ~3:30 post first crack, 425–430°F
- **Second crack:** No

---

### 6. Waialua Medium
- **Batch:** 2750g
- **Charge temp:** 410°F
- **Gas at :30:** 5 | **Gas at 1:00:** 10
- **50/50 transition:** 300–320°F, gas stays 10
- **Ramp down:** starts 350°F → 9, 360°F → 8, 370°F → 7, 380°F → 6
- **First crack:** ~385–395°F → gas 10, airflow through drum
- **Drop target:** ~4:00 post first crack, 425–430°F
- **Second crack:** No

---

### 7. Downtown Blend
- **Batch:** 2750g
- **Charge temp:** 410°F
- **Gas at :30:** 5 | **Gas at 1:00:** 10
- **50/50 transition:** 300–320°F, gas stays 10
- **Ramp down:** starts 350°F → 9, 360°F → 8, 370°F → 7, 380°F → 6
- **First crack:** ~385°F (tends toward lower end) → gas 10, airflow through drum
- **Drop target:** ~3:30 post first crack, 420–430°F
- **Second crack:** No
- **Note:** Signature blend.

---

### 8. Medium Roast Blend
- **Batch:** 2750g
- **Charge temp:** 410°F
- **Gas at :30:** 5 | **Gas at 1:00:** 10
- **50/50 transition:** 300–320°F, gas stays 10
- **Ramp down:** starts 350°F → 9, 360°F → 8, 370°F → 7, 380°F → 6
- **First crack:** ~385–395°F → gas 10, airflow through drum
- **Drop target:** 3:30–4:00 post first crack, ~423°F
- **Second crack:** No

---

### 9. 60% Kauai Blend
- **Batch:** 2750g
- **Charge temp:** 410°F
- **Gas at :30:** 5 | **Gas at 1:00:** 10
- **50/50 transition:** 300–320°F, gas stays 10
- **Ramp down:** starts 360°F (later than most) → 9, 370°F → 8, 380°F → 7
- **First crack:** ~385–395°F → gas 10, airflow through drum
- **Drop target:** ~3:00 post first crack, 425–430°F
- **Second crack:** No

---

### 10. 20% Kona Blend
- **Batch:** 2750g
- **Charge temp:** 410°F
- **Gas at :30:** 5 | **Gas at 1:00:** 10
- **50/50 transition:** 300–320°F, gas stays 10
- **Ramp down:** starts 350°F → 9, 360°F → 8, 370°F → 7, 380°F → 6
- **First crack:** ~385–395°F → gas 10, airflow through drum
- **Drop target:** ~3:00 post first crack, 425–430°F
- **Second crack:** No

---

### 11. Dark Roast
- **Batch:** 2800g
- **Charge temp:** 400°F, gas **8** ⚠️ unique — starts at 8, not 0
- **Airflow at charge:** through cooling bin
- **Gas at :30:** not tracked | **Gas at 1:00:** not tracked
- **No ramp down**
- **300–320°F:** gas drops to 7, airflow → 50/50
- **First crack:** ~385–395°F → gas drops to 6, airflow through drum
- **Second crack:** ~425–430°F → gas drops to 5
- **Drop:** 2:00 after second crack, temp will exceed 440°F
- **Second crack:** YES — critical milestone
- **Note:** ⚠️ Entirely event-driven. No ramp down. Longest roast, highest drop temp. Second crack hard to hear.

---

## V2 Data Model

### Profile object (hardcoded on first load, editable later)
```json
{
  "id": "kona_2750",
  "name": "100% Kona",
  "batchGrams": 2750,
  "chargeTempF": 410,
  "chargeGas": 0,
  "earlyGas": { "30s": 5, "60s": 10 },
  "airflow50_50TempF": "300-320",
  "gasAt50_50": 10,
  "rampDown": {
    "startTempF": 350,
    "steps": [
      { "tempF": 350, "gas": 9 },
      { "tempF": 360, "gas": 8 },
      { "tempF": 370, "gas": 6 },
      { "tempF": 380, "gas": 5 }
    ]
  },
  "postFirstCrackGas": 10,
  "firstCrackExpectedTempF": "385-395",
  "dropTargetTempF": "425-430",
  "dropTargetPostCrackMinutes": 15,
  "trackSecondCrack": false,
  "isDarkRoast": false,
  "notes": ""
}
```

### Roast log object
```json
{
  "id": "1700000000000",
  "savedAt": "2024-11-14T10:30:00.000Z",
  "profileId": "kona_2750",
  "profileName": "100% Kona",
  "batchGrams": 2750,
  "actualChargeTempF": 410,
  "checkpoints": [
    { "label": "0:30", "seconds": 30, "gas": 5, "tempF": null },
    { "label": "1:00", "seconds": 60, "gas": 10, "tempF": null },
    { "label": "50/50 transition", "seconds": 185, "gas": 10, "tempF": 312 }
  ],
  "rampDownActual": [
    { "tempF": 350, "gas": 9, "seconds": 420 },
    { "tempF": 360, "gas": 8, "seconds": 480 },
    { "tempF": 370, "gas": 6, "seconds": 540 },
    { "tempF": 380, "gas": 5, "seconds": 600 }
  ],
  "events": [
    { "type": "first_crack", "seconds": 680, "tempF": 388, "gasAfter": 10, "airflowAfter": "drum" },
    { "type": "drop", "seconds": 890, "tempF": 426 }
  ],
  "totalSeconds": 890,
  "endWeightGrams": null,
  "notes": ""
}
```

---

## V2 Screen Flow

### 1. Home
- Grid/list of 11 profiles by name
- Recent roasts strip at bottom
- Link to full history

### 2. Pre-roast overview (new)
- Show full profile as a cheat sheet: charge temp, early gas targets, ramp down table, first crack expectations, drop target
- Confirm or override batch weight
- "Begin Roast" button

### 3. Active roast — guided timer (major rebuild)
Two persistent elements:
- **Large running timer** (always visible)
- **Current cue card** — what to do right now (e.g. "350°F → Set gas to 9")
- **Next cue preview** — what's coming (e.g. "Next: 360°F → Gas 8")
- **Airflow indicator** — shows current position (X / 50/50 / →), highlights when it needs to change

Tap-to-log buttons always visible at bottom:
- **Log Checkpoint** — for :30, 1:00, 50/50 transition, ramp down steps
- **First Crack** — always prominent
- **Second Crack** — visible only for Dark Roast
- **Drop** — ends the roast

Each tap opens numpad for temp/gas entry.

### 4. Summary (enhance existing)
- Actual vs. expected comparison table
- Dev time, dev%, all events with temps
- End weight field (optional)
- Tasting notes
- Save / Discard

### 5. History (enhance existing)
- List of saved roasts grouped by profile
- Tap → detail screen

### 6. Roast detail (replace alert() placeholder)
- Full stats, event timeline, actual vs. expected
- Editable tasting notes (auto-save on blur)
- Delete roast with confirmation

---

## Priority Build Order

1. **Seed 11 profiles into localStorage** on first load — hardcode as default dataset in JS
2. **Replace setup screen** with profile selector (name + batch size + notes flag for Maui Mokka)
3. **Add pre-roast overview screen** — profile cheat sheet before starting
4. **Rebuild timer screen** — current cue + next cue + airflow indicator
5. **Checkpoint logging** — :30, 1:00, 50/50 transition, ramp down steps
6. **Replace viewRoast() alert()** with proper detail screen
7. **Summary screen** — add actual vs. expected comparison
8. **Profile editor** — allow tweaking profiles without touching code

---

## Deployment

### Quick local test
Open `index.html` directly in browser — no server needed. localStorage works fine.

### GitHub Pages
```bash
git init && git add . && git commit -m "initial"
gh repo create roast-timer --public --push
# Enable Pages in repo Settings → deploy from main branch root
```

### Raspberry Pi (local network)
```bash
cd roast-timer
python3 -m http.server 8080
# Access at http://<pi-ip>:8080 from any device on same WiFi
```
For permanent Pi setup: Nginx serving static files, configured to start on boot.

### PWA install
Host anywhere → user visits in Safari/Chrome → "Add to Home Screen" → installs as standalone app, no App Store needed.

---

## V2 Progress

### Completed
- ✅ 11 profiles seeded with full roast data (Kona ramp corrected to 10→9→8→7→6)
- ✅ Profile selector home screen
- ✅ Pre-roast cheat sheet/overview ("Green Bean Drop", "Initial Gas Ramp", drop shows time-after-1st-crack + target temp)
- ✅ Guided timer with table-based cue system (columns: Cue, Airflow, Gas, Temp)
- ✅ Inline temp + gas entry at every step — no overlay numpad
- ✅ Rampdown steps prefill temp from profile; gas prefilled at all stages but editable
- ✅ Time-based cues (0:30, 1:00) auto-expand for temp entry instead of auto-skipping
- ✅ Airflow indicator, post-crack timer
- ✅ Live log + guide panel split
- ✅ Cancel out of events without committing (preview what a step entails, then back out)
- ✅ Weight loss calculation on summary (roasted weight input + auto % calc)
- ✅ Roast detail screen (replaced alert() placeholder) with editable notes, auto-save on blur, delete with confirmation, back button near profile name
- ✅ History filtering by profile + sorting by date/profile with month headers
- ✅ Timer persists across tab switches (Date.now()-based, not setInterval counter)
- ✅ Timer display fixed at top — split layout so clock never scrolls away
- ✅ Mobile button fixes (native confirm() replaced with custom modal for PWA compatibility)
- ✅ Deployed to GitHub Pages: https://benjaminboughton.github.io/roasting-timer/

### Work log

**Session 1 (Apr 2–3):** Built v2 from scratch — profile system, guided timer, cue cards, overview screen, summary enhancements. Iterative UI refinements: renamed sections (Charge → Green Bean Drop, Early Gas → Initial Gas Ramp), fixed ramp table layout, restructured Drop section, fixed Kona ramp data.

**Session 2 (Apr 3):** Major timer UI rework — replaced overlay numpad with inline inputs, split guide panel + live log, added cancel on events, added weight loss calc to summary. Deployed to GitHub Pages.

**Session 3 (Apr 5):** Replaced viewRoast() alert with full detail screen. Added history filtering/sorting. Fixed timer persistence across tab switches. Made timer display fixed (non-scrolling). Added prefilled temp/gas at all stages. Confirmed mobile button fixes. Updated handoff doc with scaling roadmap.

### Still to do
- **Multi-size profiles** — Each profile (e.g. 100% Kona) needs multiple batch-size variants (2750g, 2200g, 1100g) with different gas schedules for each. Currently you can override the weight number, but the guide still shows the default size's recipe. Implementation: add a `variants` array to each profile, keyed by batch size, each with its own `earlyGas`, `rampDown`, and `postFirstCrackGas`. On the pre-roast screen, selecting a batch size loads that variant's recipe. This is tedious data entry — someone who knows the actual recipes for each size needs to input them.
- **Profile editor** — allow editing/saving profiles on the site without touching code. Ties into multi-size profiles — the editor should let you add/edit batch-size variants.
- **Time cue warnings for non-time cues** — estimated warnings based on historical averages for temp-based cues (e.g. "based on your last 5 roasts, 50/50 usually happens around 3:05")
- **Service worker** for offline PWA support (critical for roasteries with spotty WiFi)

---

## Scaling Roadmap

### Current state: GitHub Pages (free, static)
- GitHub serves the single HTML file from their servers
- No backend, no database — all data in browser localStorage
- Data is per-device and per-browser (clearing browser = data gone)
- URL: `benjaminboughton.github.io/roasting-timer`

### Level 1: Custom domain + polish
- **Vercel or Netlify** (free tier) — same static hosting but with a custom domain (e.g. `roasttimer.app`)
- Add a **service worker** for offline support — critical for roasteries
- Add proper PWA icons and splash screens
- Still no backend — localStorage only, single-user per device

### Level 2: Backend + multi-device sync
- **Supabase** (free tier) — PostgreSQL database + auth + API, all managed
- Add user accounts (email/password or magic link)
- Roast data syncs across devices — log on your phone, review on laptop
- Multiple roasteries can each have their own account
- This is the minimum to sell to another roastery

### Level 3: Multi-user per roastery
- Roastery-level accounts with multiple operators
- Shared roast history, shared/customized profiles
- Role-based access (owner edits profiles, operators log roasts)
- Still Vercel + Supabase, just more schema

### Level 4: Scale
- **Cloudflare** in front for caching/CDN (free tier) — fast loading worldwide
- Analytics dashboard (roast trends, consistency tracking over time)
- Consider charging: SaaS model, ~$20/mo per roastery
- This is where a YC application becomes realistic

### Hosting glossary
| Service | What it does | When you need it |
|---------|-------------|-----------------|
| **GitHub Pages** | Serves static files for free | Now (you're here) |
| **Vercel / Netlify** | Static hosting + custom domains + auto-deploy | When you want `roasttimer.app` |
| **Supabase / Firebase** | Database + user auth + API (backend-as-a-service) | When multiple people/devices need shared data |
| **Cloudflare** | CDN + caching + DDoS protection, sits in front of your host | When you have real traffic to optimize |
| **AWS / Google Cloud** | Raw servers you configure yourself | Overkill until thousands of users |

### Getting traction
1. Use it yourself for a few weeks, fix what sucks in real roast sessions
2. Add backend + auth (1-2 weekends with Supabase)
3. Get 3-5 other roasters to try it (friends, local roasteries on island)
4. Iterate on their feedback
5. Post "Show HN: I built a guided roast timer for our small Hawaiian coffee roastery" — HN loves niche, authentic tools
6. Share on r/roasting, r/coffee, Home-Barista.com, Coffee Roasters Guild Slack
7. If it gets traction, add billing and go from there

---

## Miscellaneous Notes

- **Batch size varies day to day** based on inventory. Profiles are built around today's typical sizes. A future enhancement: allow batch size selection at roast start with adjusted early gas targets.
- **Maui Mokka** is the newest and most volatile profile — flag it visually (e.g. a "beta" badge). ✅ Done — has "beta" badge.
- **Dark Roast** is structurally different from all others. ✅ Done — same layout, different cues (no ramp down, second crack button visible).
- All temps are °F. A future enhancement could add a °F/°C toggle stored in localStorage.
- The gas gauge physically reads 0–16 "WC but roasters use it on a 0–10 scale in practice. The app uses 0–10.
