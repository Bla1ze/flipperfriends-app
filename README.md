<div align="center">


# Flipper Friends — Tournament Viewer

<img width="1920" height="1080" alt="vlcsnap-2026-07-30-01h22m53s293" src="https://github.com/user-attachments/assets/7f5684ec-b290-4db3-bd88-77e804fe2bc0" />



**View the Flipper Friends pinball tournament leaderboards natively on your AtGames cabinet.**

[![Platform](https://img.shields.io/badge/target-aarch64%20Linux-1f6feb)](../../README.md#-device-constraints--gotchas)
[![Graphics](https://img.shields.io/badge/graphics-SDL2-1d7874)](#)
[![Data](https://img.shields.io/badge/data-live%20HTTPS%20API-2ea043)](#-data-source)
[![Offline](https://img.shields.io/badge/offline-cached-orange)](#-caching--offline)

</div>

---

The Flipper Friends website is a JavaScript web app, and **no web browser will run on the cabinet's firmware** (old glibc + old SDL2). So instead of rendering the site, this app talks to the site's **public JSON API** directly and draws the standings natively — fast, joystick-friendly, and offline-tolerant.

> This is the reference pattern for putting *any* web-hosted data source on the cabinet: **reimplement the view natively against the same API**, rather than trying to embed a browser.

## Table of Contents

- [Features](#-features)
- [Screens](#-screens)
- [Controls](#-controls)
- [Data Source](#-data-source)
- [Caching & Offline](#-caching--offline)
- [Building & Installing](#-building--installing)
- [Architecture](#-architecture)
- [Limitations](#-limitations)
- [Credits](#-credits)

## ✨ Features

- 🏆 **Live standings** — tournaments, the games in each, and full rankings (rank · player · signature · score · device · date), fetched over HTTPS on launch.
- 🔎 **Filter** — Active / All / Expired tournaments.
- 🔄 **Auto + manual refresh** — background refresh (list every 60 s, rankings every 30 s) plus an on-demand **refresh** button.
- 📴 **Offline cache** — last-known standings are saved to USB and shown instantly, even with no network, behind an **"OFFLINE — updated N ago"** notice.
- 🎨 **Neon UI** — teal→violet gradients, glowing headers, and the Bebas Neue display font; scores formatted like `1,253,160`, top-3 ranks in gold.
- 🕹️ **No keyboard needed** — the whole app drives from an arcade joystick + buttons.

## 📱 Screens

```
Tournaments  ──▶  Games  ──▶  Rankings
(filterable)      (per         (scrollable
                  tournament)   leaderboard)
```

1. **Tournaments** — the tournament list, with an Active / All / Expired filter.
2. **Games** — the games contained in the selected tournament, each showing its score count.
3. **Rankings** — the leaderboard for a game: two-line rows with `#rank  player  score` and `signature · device · date` beneath.

## 🎮 Controls

| Input | Action |
|---|---|
| **Joystick / D-pad** | Move selection · scroll |
| **A / Start** | Open / select |
| **Left / Right** | Cycle the filter (Tournaments screen) |
| **Y** | Refresh now |
| **B / Back** | Go back · exit (with confirmation) |

> Button labels vary by cabinet — the SDK's [`Controls`](../../sdk/controls) helper maps physical inputs to these logical actions.

## 🌐 Data Source

This is AtGames Legends pinball tournament data. Nested JSON is parsed with a small dependency-free scanner (`Json.h`) — no JSON library.

## 💾 Caching & Offline

- Every **successful** response is cached to the USB drive:
  ```text
  /media/usb0/external/flipperfriends-app/data/
  ├── tournaments.json
  └── detail-<id>.json
  ```
- On launch, the cached list loads **instantly** (before the network responds); opening a tournament loads its cached rankings the same way.
- A **failed** fetch never blanks the screen — it keeps the cached data and shows an amber **`OFFLINE — UPDATED 5M AGO`** badge under the header.
- All caching is best-effort: if the USB isn't writable, the app just runs live-only, never crashes.

## 🔧 Building & Installing

Copy the output to a USB drive and plug it into the cabinet:

```text
USB root/
└── external/
    └── flipperfriends-app/
        ├── flipperfriends-app.elf
        ├── flipperfriends-app.png
        └── flipperfriends-app.xml
```

It appears under **External Applications** as **Flipper Friends**.

## 🏗️ Architecture

Built on the shared SDK app framework (portrait canvas, `Controls`, embedded fonts, `sdk/http` helper).

```text
src/
├── App.{h,cpp}            # SDL window, input, main loop
├── TournamentsApp.{h,cpp} # the app: screens, fetching, caching, rendering
├── Models.h               # Tournament / GameEntry / RankEntry structs
├── Json.h                 # dependency-free JSON scanner (+ nested arrays)
├── Theme.h                # shared Neon theme (gradients, cards, dialogs, glow)
├── AppFont.{h,cpp}        # two-face bitmap text (Noto body + Bebas display)
└── Menu / Utils / AppConfig / TextUtils / main
```

Network fetches run on detached background threads so the UI never blocks; fetched data is mutex-guarded so a background refresh can swap it in while the current view stays on screen.

## ⚠️ Limitations

- **Text only, no box art.** Tournament/game artwork is PNG, and the firmware's core SDL2 can't decode images — the data all shows; the artwork doesn't.
- **Depends on the Flipper Friends proxy.** It reads through the site's Cloudflare proxy, so it relies on that deployment staying up.
- **"Updated N ago" is approximate** on a cold offline start (derived from the cache file's timestamp).

## 🙏 Credits

- Underlying tournament data: **AtGames** Legends pinball.
- Fonts: **Noto Sans** & **Bebas Neue** (SIL Open Font License).
- Display info: @n-i-x

Part of the [AtGames External Application SDK](../../README.md).
