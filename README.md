<div align="center">

# zotters_npc — FiveM NPC Pathing & Ambient Ped Spawner Script

**A configurable, NUI-driven ambient NPC and vehicle pathing system for FiveM servers.**
Build walking patrols, driving routes, and ambient wander zones with a live in-game editor — no server restarts, no manual JSON editing.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![FiveM](https://img.shields.io/badge/platform-FiveM-orange)](https://fivem.net)
[![Lua](https://img.shields.io/badge/lua-5.4-blue)](https://www.lua.org/)
[![Standalone](https://img.shields.io/badge/framework-standalone-green)]()
[![oxmysql](https://img.shields.io/badge/storage-JSON%20%7C%20oxmysql-lightgrey)](https://github.com/overextended/oxmysql)

</div>

---

## What is zotters_npc?

`zotters_npc` is a **FiveM ambient AI ped and NPC pathing script** for server owners who want
believable, performant background population — night guards on patrol, daytime commuter traffic,
market crowds — without hand-coding routes or hardcoding coordinates. Everything is authored live
through an in-game **NUI dashboard**: drop waypoints by walking to them, set active hours and
weather conditions, tune hostility and combat behavior, and save instantly for every player on
the server.

It's built as a **standalone FiveM resource** (no ESX/QBCore dependency) with distance-based
streaming, near-zero idle resmon cost, and native [Renewed-Weathersync](https://github.com/Renewed-Development/Renewed-Weathersync)
integration for time-of-day and weather-gated spawns.

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Waypoint Capture Mode](#waypoint-capture-mode)
- [Configuration](#configuration)
- [How the Pathing Logic Works](#how-the-pathing-logic-works)
- [Renewed-Weathersync Integration](#renewed-weathersync-integration)
- [Data Persistence](#data-persistence)
- [Permissions](#permissions)
- [Troubleshooting / FAQ](#troubleshooting--faq)
- [Project Structure](#project-structure)
- [Contributing](#contributing)
- [License](#license)

---

## Features

- 🖥️ **Live NUI dashboard** — create, edit, toggle, and delete NPC routes/zones in real time, no `/restart` needed
- 🚶 **Walking routes** — ordered waypoint patrols with looping, scenarios, and configurable move speed
- 🚗 **Driving routes** — vehicle node-to-node navigation with per-route driving style and speed
- 🧍 **Ambient wander zones** — radius-based crowd simulation using GTA's native wander AI
- 🎯 **In-world waypoint capture mode** — walk or drive to a spot and press a key to drop a point, instead of alt-tabbing into a menu
- 🕒 **Time-of-day & weather conditions** — night-only guards, daytime commuters, weather-restricted spawns
- 🔫 **Full behavior control** — ped models (fixed or random pool), weapons, hostility (passive/defensive/aggressive), relationship groups, scenario animations
- 📡 **Distance-based streaming** — peds and vehicles only exist near players; automatic culling keeps server/client load low
- ⚡ **Near 0.00ms idle cost** — movement is delegated to native GTA tasks (`TaskFollowNavMeshToCoord`, `TaskVehicleDriveToCoordLongrange`, `TaskWanderStandard`), not per-frame Lua loops
- 💾 **Flexible persistence** — JSON file storage out of the box, or `oxmysql` for multi-server/database-backed setups
- 🌦️ **Renewed-Weathersync aware** — reads authoritative synced time/weather instead of the native clock
- 🔐 **ACE-gated admin access** — dashboard and all CRUD actions are permission-checked server-side
- ✅ **Toast feedback** — every save/delete/toggle reports success or failure back to the editor, so nothing fails silently

---

## Requirements

| Dependency | Required? | Notes |
|---|---|---|
| [FiveM](https://fivem.net) server build | ✅ Required | `cerulean` fxmanifest, Lua 5.4 |
| [oxmysql](https://github.com/overextended/oxmysql) | ⚠️ Optional | Only needed if `Config.UseMySQL = true` |
| [Renewed-Weathersync](https://github.com/Renewed-Development/Renewed-Weathersync) | ⚠️ Optional | Enables accurate time/weather gating; falls back gracefully if absent |
| ESX / QBCore / any framework | ❌ Not required | Fully standalone |

---

## Installation

1. Drop the `zotters_npc` folder into your server's `resources` directory.
2. Add it to `server.cfg` (after `oxmysql`/`Renewed-Weathersync` if you use either):
   ```cfg
   ensure zotters_npc
   ```
3. Grant dashboard access via ACE permissions:
   ```cfg
   add_ace group.zotters_npc_admin zotters_npc.admin allow
   add_principal identifier.license:YOUR_LICENSE_IDENTIFIER group.zotters_npc_admin
   ```
   Or in-game, from the server console:
   ```
   zotters_npc_grant <server id>
   ```
4. (Optional) Set `Config.UseMySQL = true` in `config.lua` to persist routes via `oxmysql`
   instead of the default `data/routes.json` file. The `zotters_npc_routes` table is
   created automatically on first boot.
5. Start your server and open the dashboard with `/zotters_npc` or the `F7` keybind.

---

## Usage

| Action | Command / Key | Notes |
|---|---|---|
| Open/close dashboard | `/zotters_npc` or `F7` | Rebindable in FiveM's Key Bindings settings under "zotters_npc" |
| Create a route/zone | **+ New** button in the dashboard | Choose Walking, Driving, or Ambient Wander |
| Save changes | **Save Route** button | Broadcasts to all connected clients instantly |
| Delete a route | **Delete** button | Requires the `zotters_npc.admin` ACE |
| Enable/disable a route | Toggle in the sidebar | Does not delete saved data |

## Waypoint Capture Mode

Manually typing coordinates is slow and error-prone. Instead, click **🎯 Capture Mode** in the
Walking or Driving editor to walk or drive freely while dropping points on the fly:

| Key | Action |
|---|---|
| `E` | Drop a point at your current position/heading |
| `Backspace` | Undo the last dropped point |
| `F6` | Finish capture mode and return to the editor |

Capture mode uses raw keyboard input (not FiveM's rebindable keymap system) so it works
consistently regardless of other resources' keybindings. Your in-progress form data is preserved
the entire time — closing the dashboard (`F7`) always safely exits capture mode as well.

---

## Configuration

All tunables live in [`config.lua`](config.lua). Key options:

```lua
Config.Command               = 'zotters_npc'  -- command to open the dashboard
Config.Keybind                = 'F7'
Config.AdminOnly              = true
Config.AdminAce                = 'zotters_npc.admin'
Config.UseMySQL                = false          -- false = JSON file, true = oxmysql

Config.SpawnRadius             = 80.0           -- streaming distance (spawn)
Config.DespawnRadius           = 120.0          -- streaming distance (despawn, hysteresis)
Config.SpawnCheckIntervalMs   = 1500
Config.MaxPedsPerZone         = 6
Config.MaxActiveZones         = 40

Config.UseRenewedWeathersync = true            -- see integration section below
```

See [`Config.ExampleRoutes`](config.lua) for the full per-route/zone schema (models, hostility,
weapons, time-of-day windows, weather filters, driving styles, etc.).

---

## How the Pathing Logic Works

**Streaming** (`client/spawner.lua`) — a single lightweight thread ticks every
`Config.SpawnCheckIntervalMs` (default 1.5s), comparing each route's anchor point against every
player's position. Routes within `Config.SpawnRadius` spawn; active routes beyond
`Config.DespawnRadius` despawn. The gap between the two radii is deliberate hysteresis, preventing
peds from flickering in and out at the boundary. Time-of-day and weather conditions are
re-evaluated on every pass.

**Walking routes** use `TaskFollowNavMeshToCoord` against an ordered waypoint list. A per-route
monitor thread polls every 400ms (not every frame) for arrival, then advances — looping back to
the first waypoint if `loop` is enabled. A stall watchdog re-issues the task if a ped gets stuck,
rather than relying on a force-timeout (which causes visible teleporting).

**Driving routes** use `TaskVehicleDriveToCoordLongrange` node-to-node, with a configurable
driving-style bitmask (`Config.DrivingStyles`) and target speed.

**Ambient wander zones** spawn a configurable number of peds inside a radius and hand them to
GTA's native `TaskWanderStandard` — no custom movement math, just a slow watchdog that re-issues
the task if it gets interrupted (e.g. by combat).

**Performance**: because actual steering is delegated entirely to native game tasks, no Lua loop
runs faster than ~400ms per active instance, and the top-level streaming tick is 1.5s — idle
resource usage sits at effectively **0.00ms**.

---

## Renewed-Weathersync Integration

Time-of-day and weather gating (`client/weathersync.lua`) read directly from
[Renewed-Weathersync](https://github.com/Renewed-Development/Renewed-Weathersync)'s
`GlobalState.currentTime.hour` and `GlobalState.weather.weather`, rather than FiveM's native
clock/weather:

- The native clock only reflects Renewed-Weathersync's simulated time for a player if that
  player's own `syncWeather` toggle is enabled — `GlobalState.currentTime.hour` is authoritative
  regardless of that per-player setting.
- Valid weather filter values match whatever Renewed-Weathersync cycles through: `BLIZZARD`,
  `CLEAR`, `CLEARING`, `CLOUDS`, `EXTRASUNNY`, `FOGGY`, `NEUTRAL`, `OVERCAST`, `RAIN`, `SMOG`,
  `SNOW`, `SNOWLIGHT`, `THUNDER`, `XMAS`.

This is a **soft dependency** — `client/weathersync.lua` checks `GetResourceState('Renewed-Weathersync')`
at runtime. If it isn't running (or `Config.UseRenewedWeathersync = false`), time gating falls
back to the native game clock and weather filters are simply ignored rather than erroring.

---

## Data Persistence

Routes/zones are plain Lua tables identified by a string `id` (see `Config.ExampleRoutes` in
`config.lua` for the schema). The dashboard authors these client-side and sends them through
`client/nui.lua` → server events (`zotters_npc:server:*`) in `server/main.lua`, which validates,
persists (JSON or `oxmysql`, via `server/storage.lua`), and broadcasts the update to every
connected client (`zotters_npc:client:syncRoutes`) — changes apply instantly, server-wide.

## Permissions

All create/update/delete/toggle actions are gated server-side by the `zotters_npc.admin` ACE
(`Config.AdminAce`), independent of anything the client claims. Set `Config.AdminOnly = false`
to disable the gate entirely (not recommended for production).

---

## Troubleshooting / FAQ

**Save button does nothing.**
Check for a toast notification in the dashboard — success and failure are both reported. The
most common cause is a missing `zotters_npc.admin` ACE grant (see [Installation](#installation)).

**Peds are teleporting / snapping around.**
Make sure you're on the latest version — earlier releases used a hard task-completion timeout on
`TaskFollowNavMeshToCoord` that force-warped stuck peds. This is fixed by using `-1` (no forced
completion) plus a stall-detection watchdog.

**Defensive/passive peds aren't reacting to being attacked.**
Also fixed in the current version — hostility flags were previously being overwritten by a
stray `SetPedFleeAttributes` call after the behavior switch.

**Capture mode keys don't respond.**
Capture mode uses raw keyboard polling, not FiveM's remappable keybind system, specifically to
avoid this class of issue. If it's still unresponsive, confirm no other resource is intercepting
raw input, and use `F7` to force-close the dashboard (this also always cancels capture mode).

---

## Project Structure

```
zotters_npc/
├── fxmanifest.lua          # resource manifest
├── config.lua              # all tunables + example route schema
├── shared/
│   └── utils.lua           # shared helpers (global ZottersNPC.Utils)
├── server/
│   ├── main.lua            # authoritative route store + NUI CRUD network events
│   └── storage.lua         # JSON file / oxmysql persistence backends
├── client/
│   ├── main.lua            # dashboard open/close, route cache sync, capture mode
│   ├── spawner.lua         # distance-based streaming + condition gating
│   ├── pathing.lua         # ped/vehicle spawn + movement loops
│   ├── weathersync.lua     # Renewed-Weathersync GlobalState adapter
│   └── nui.lua              # NUI callback → server event bridge
└── html/                   # dashboard UI (vanilla HTML/CSS/JS, no build step)
```

---

## Contributing

Issues and pull requests are welcome. Please include your FiveM server build, whether
`Config.UseMySQL`/`Config.UseRenewedWeathersync` are enabled, and console/resmon output when
reporting a bug.

## License

Released under the [MIT License](LICENSE).

---

<div align="center">

**Keywords:** FiveM NPC script, FiveM ped spawner, ambient AI peds FiveM, NPC pathing script,
FiveM NUI dashboard, waypoint editor FiveM, FiveM vehicle pathing, standalone FiveM resource,
oxmysql NPC storage, Renewed-Weathersync integration

</div>
