# VO Turf List

A single-file web app for playing [Turf](https://turfgame.com) outdoors with VoiceOver on iPhone. Built to be fast, accessible, and usable with one hand while walking.

Made by Conejo. Last updated 2026-05-19.

---

## What it does

VO Turf List shows you the 30 nearest Turf zones from your current position, sorted by distance. For each zone it tells you:

- The zone name
- How far away it is in meters
- Which direction it is — given as a clock face (12 o'clock, 3 o'clock, etc.), relative to the direction you are actually facing if compass is enabled
- Who owns it
- Whether it is blocked, and exactly how long the block time remaining is, calculated from the zone owner's actual rank

Tapping a zone opens a detail page with full information and a list of all other zones that same owner currently holds.

---

## Designed for VoiceOver

The app is built from the ground up for VoiceOver on iPhone. Every zone in the list is a single swipe — one swipe, one zone, all key information read in one sentence. No sub-elements, no nested focusable controls, no noise.

The detail page uses a plain list where each item is one swipe group, for example:

- *Blockerad av TurferX — 8m 12s kvar*
- *178 m — Klockan 3, öst*
- *Nuvarande ägare TurferX rank 55, Ägarens blocktid 23m 45s*

It has also been tested with JAWS on desktop. The structure avoids heavy ARIA use, relying instead on semantic HTML that works with any screen reader.

---

## Features

**Navigation**
- GPS position acquired on first tap — no automatic background requests
- Compass support on iOS 13 and later: directions are relative to where you are facing, not north
- Both GPS and compass permissions are requested with the same single button tap

**Zone list**
- 30 nearest zones shown by default
- Load 10 more at a time with a button at the bottom
- Sticky toolbar with Refresh and Filter buttons visible while scrolling

**Filter**
- All zones (default)
- Available zones only
- Blocked zones only
- Filter persists across auto-refreshes within the same session

**Block time**
- Fetches the zone owner's profile from the Turf API to get their exact block time in seconds
- Calculates remaining block time precisely: owner's block time minus elapsed time since the zone was last taken
- Block countdown ticks live every second

**Zone detail page**
- Opens via native browser navigation — VoiceOver two-finger scrub, Safari back swipe, and back button all work naturally
- Shows block status, distance, direction, owner name and rank, last taken time, elapsed time, points, zone type, region, and creation date
- Lists all zones currently owned by the same owner, sorted by distance from you

**Language**
- Swedish and English, switchable at any time
- Language preference saved across sessions
- All directions and labels translate fully including clock-face directions

**Accessibility**
- Fully compatible with iOS Dynamic Type — if you use large text in iOS Settings, the entire app scales with it
- No fixed pixel font sizes anywhere
- High contrast dark theme
- Tested with VoiceOver on iPhone and JAWS on desktop

---

## How to use

1. Open `turf-nearby.html` in Safari on your iPhone
2. Tap **Starta navigering** (or **Start navigation** in English)
3. Allow location access when Safari asks
4. Allow compass access when Safari asks (optional but recommended)
5. The 30 nearest zones load automatically and update every 30 seconds

To switch language, scroll to the bottom of the zone list and tap **SV** or **EN**.

---

## Hosting

The app is a single HTML file with no dependencies, no build step, and no server required. You can host it anywhere that serves static files.

The easiest option is [Netlify Drop](https://app.netlify.com/drop): drag the file onto the page and you get a live URL in about 30 seconds, for free. Open that URL in Safari on your iPhone and it works immediately.

> **Note:** GPS does not work when opening an HTML file directly from the Files app on iPhone. The file needs to be served over HTTPS for location permissions to work in Safari. Netlify Drop handles this automatically.

---

## How block time is calculated

Turf blocks a zone after it is taken. The block duration depends on the rank of the player who took it, not your own rank. The formula is:

```
block time (minutes) = rank / 4 + 10, maximum 25 minutes
```

This app fetches the current owner's profile directly from the Turf API to get their exact block time in seconds, then calculates:

```
remaining = owner's block time − seconds elapsed since dateLastTaken
```

If the remaining time is zero or negative, the zone shows as **Available**.

---

## Data source

All zone and user data comes from the official [Turf API v5](https://api.turfgame.com/v5). The app makes the following API calls:

- `POST /zones` with a bounding box around your position — to fetch nearby zones
- `POST /users` with owner IDs — to fetch block times and ranks for zone owners
- `POST /zones` with a list of zone IDs — to fetch zones owned by the current zone's owner on the detail page

The app respects the API's rate limit of one request per second per resource.

---

## File structure

```
turf-nearby.html    — the entire app, one file
README.md           — this file
```

---

## Requirements

- iPhone with Safari (iOS 13 or later recommended for compass support)
- Internet connection (to fetch zone data from the Turf API)
- A Turf account is not required to use this app

---

## License

Do whatever you want with this. Credit is appreciated but not required.
