# VO Turf List

An accessible companion app for the [Turf](https://turfgame.com) game. Built specifically for screen reader users — navigate nearby zones with VoiceOver on iPhone, get live block time countdowns, and hear zone updates read aloud automatically.

Vibe coded with Claude by Conejo.

-----

## What it does

VO Turf List shows you the 30 nearest Turf zones from your current position, sorted by distance. For each zone it tells you in a single VoiceOver swipe:

- Zone name
- Distance in meters
- Direction as a clock face relative to where you are facing (12 o’clock, 3 o’clock etc.)
- Who owns it
- Whether it is blocked and exactly how long the block time remaining is

Tapping a zone opens a full detail page. Saving your Turf username unlocks your personal profile page with stats and all your currently owned zones.

-----

## Designed for VoiceOver

Every zone in the list is one swipe — one swipe, one zone, all key information in one sentence. No sub-elements, no nested controls, no noise.

The detail page uses a plain list where each item is one swipe, for example:

- *Blockerad av TurferX — 8m 12s kvar*
- *178 m — Klockan 3, öst*
- *Nuvarande ägare TurferX rank 55, Ägarens blocktid 23m 45s*

The app has been tested with VoiceOver on iPhone and JAWS on desktop. It avoids heavy ARIA use and relies on semantic HTML that works naturally with any screen reader.

-----

## Features

### Navigation

- GPS and compass permissions requested with the same single button tap
- Compass support on iOS 13 and later — directions are relative to where you are facing, not north
- Clock-face directions: 12 o’clock north, 3 o’clock east, 6 o’clock south, 9 o’clock west

### Zone list

- 30 nearest zones shown by default
- Load 10 more at a time with a button at the bottom
- Sticky toolbar with Refresh and Filter buttons always visible while scrolling

### Filter

- All zones (default)
- Available zones only
- Blocked zones only
- Filter persists across auto-refreshes within the same session

### Block time

- Fetches the zone owner’s profile from the Turf API to get their exact block time in seconds
- Calculates remaining block time precisely: owner’s block time minus elapsed time since last taken
- Block countdown ticks live every second

### Zone detail page

- Opens via native browser navigation
- Back button works in both Safari and standalone home screen mode
- Shows block status, distance, direction, owner name and rank, last taken, elapsed time, points, zone type, region, and creation date
- Lists all zones currently owned by the same owner, sorted by distance from you

### User profile

- Enter your Turf username once under Settings
- Profile page shows rank and rank name, points to next rank, total points, points per hour, unique zones taken, total takeovers, total medals, owned zone count, region, and global place
- Your own zones appear in yellow in the zone list and read as “Din zon” / “Your zone” — no redundant block time or owner info

### Voice mode

- Reads the nearest zone aloud on every auto-update
- 440Hz wake tone before speech to activate Bluetooth headphones
- Language matches the interface automatically (Swedish or English)
- Adjustable speech speed from 1.0 to 2.5, default 1.4
- Toggle in the toolbar, state saved across sessions

### Update interval

- Adjustable from 5 to 60 seconds, step 5, default 30
- Restart countdown immediately when adjusted
- Saved across sessions

### Screen wake lock

- Silent audio loop prevents iOS from locking the screen while the app is active
- Automatically resumes if interrupted by a phone call

### Language

- Swedish and English, switchable at any time
- Language preference saved across sessions
- All directions, labels, and voice announcements translate fully

### Accessibility

- Fully compatible with iOS Dynamic Type — scales with your system text size setting
- No fixed pixel font sizes
- High contrast dark theme
- Tested with VoiceOver on iPhone and JAWS on desktop

-----

## Setup — one time steps

### 1. Allow location permanently in Safari

So the app never asks for GPS permission again:

1. Open the app in Safari
1. Tap the **Page menu** to the left of the address bar
1. Tap **More** at the bottom right of the menu
1. Tap **Website Settings**
1. Set **Location** to **Allow**

### 2. Save to home screen

For the best experience as a standalone app:

1. Open the app in Safari
1. Tap the **Share** button
1. Choose **Add to Home Screen**
1. Confirm

The app will open without browser controls. Use the **back button inside the app** to navigate back — the browser swipe gesture is not available in standalone mode.

-----

## How to use

1. Open `index.html` in Safari on your iPhone
1. Tap **Starta navigering** / **Start navigation**
1. Allow location access when Safari asks
1. Allow compass access when Safari asks (optional but recommended for accurate directions)
1. The 30 nearest zones load automatically and update every 30 seconds

To switch language, scroll to the bottom of the zone list and tap **SV** or **EN** under Settings.

To add your profile, enter your Turf username in the Settings section and tap **Lägg till** / **Add**.

-----

## Hosting

The app is a single file — `index.html` — with no dependencies, no build step, and no server required.

The easiest hosting option is [Netlify Drop](https://app.netlify.com/drop): drag `index.html` onto the page and you get a live HTTPS URL in about 30 seconds, for free. Open that URL in Safari on your iPhone and it works immediately.

You can also host it on GitHub Pages by pushing `index.html` to the root of a repository and enabling Pages in the repo settings.

> **Note:** GPS does not work when opening an HTML file directly from the Files app on iPhone. The file needs to be served over HTTPS for location permissions to work in Safari. Netlify Drop and GitHub Pages both handle this automatically.

-----

## How block time is calculated

Block time belongs to the player who took the zone, not you. The app fetches the current zone owner’s profile from the Turf API to get their exact block time in seconds, then calculates:

```
remaining = owner's block time in seconds − seconds elapsed since dateLastTaken
```

If remaining time is zero or negative, the zone shows as available.

Block time by rank: `rank / 4 + 10 minutes`, maximum 25 minutes at rank 60.

-----

## Rank names

All 61 rank names are built in, from rank 0 Newbie to rank 60 Turfalicious. The profile page shows your current rank name and exactly how many points you need to reach the next rank.

-----

## API calls

All data comes from the official [Turf API v5](https://api.turfgame.com/v5). The app makes the following calls:

- `POST /zones` with a bounding box — fetch nearby zones
- `POST /users` with owner IDs — fetch block times and ranks for zone owners
- `POST /zones` with zone IDs — fetch zones owned by a detail page zone’s owner
- `POST /users` with your username — fetch your profile and owned zones

The app respects the API rate limit of one request per second per resource.

-----

## File structure

```
index.html    — the entire app, one file
README.md     — this file
```

-----

## Requirements

- iPhone with Safari (iOS 13 or later recommended for compass support)
- Internet connection (to fetch zone data from the Turf API)
- A Turf account is not required — but adding your username unlocks the profile and owned zones features

-----

## Known limitations

- The app requires the screen to stay unlocked and Safari open to function. Keep auto-lock disabled during play or save to home screen for best results.
- In standalone home screen mode, use the back button inside the app — the browser swipe gesture is not available.
- Zone history (which specific zones you have personally taken before) is not available through the Turf API, so unique zones per player cannot be filtered at this time.

-----

## License

Do whatever you want with this. Credit is appreciated but not required.