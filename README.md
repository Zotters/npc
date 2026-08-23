<div align="center">

# zotters_npc — FiveM NPC Pathing & Ambient Ped Spawner Script

**A configurable, NUI-driven ambient NPC and vehicle pathing system for FiveM servers.**
Build walking patrols, driving routes, and ambient wander zones with a live in-game editor — no server restarts, no manual JSON editing.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![FiveM](https://img.shields.io/badge/platform-FiveM-orange)](https://fivem.net)
[![Lua](https://img.shields.io/badge/lua-5.4-blue)](https://www.lua.org/)
[![Standalone](https://img.shields.io/badge/framework-standalone-green)]()
[![oxmysql](https://img.shields.io/badge/storage-JSON%20%7C%20oxmysql-lightgrey)](https://github.com/overextended/oxmysql)

Made with ❤️ by Zotters

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
- [Squads, Idle Variation & Respawn](#squads-idle-variation--respawn)
- [Proximity Alerts (Heist / Defense Hook)](#proximity-alerts-heist--defense-hook)
- [Alarm Zones](#alarm-zones)
- [Escalation & Alert Groups](#escalation--alert-groups)
- [Anti-Cheat Hardening](#anti-cheat-hardening)
- [Dashboard Tools: Search, Duplicate, Import/Export, Map Preview, Alert Log](#dashboard-tools-search-duplicate-importexport-map-preview-alert-log)
- [Debug Overlay](#debug-overlay)
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
- 🚨 **Proximity alerts** — an alert-enabled guard fires a global server event when a player gets close (optionally only if armed, only with line of sight) and switches into combat, so other resources (e.g. a bank heist script) can react — lock doors, dispatch police, fail the job
- 🪜 **Escalation tiers** — optionally investigate before committing to combat, and re-trigger within a window to escalate, instead of instant "aimbot guard" reactions
- 🔔 **Alarm zones** — a ped-less trigger volume for silent alarms/cameras, reusing the same alert/group system
- 🔗 **Alert groups** — link multiple guards/alarms together so tripping one alerts the whole group, even across different players' clients
- 👥 **Patrol squads** — spawn 2–4 peds walking the same route together instead of a lone patroller
- 🎲 **Idle variation** — walking peds randomly pause for a scenario animation at waypoints instead of beelining every time
- ♻️ **Auto-respawn** — optionally respawn a killed guard/wanderer after a delay instead of leaving the post empty
- 🔍 **Dashboard search, duplicate, import/export** — filter the route list, clone an existing route, or copy/paste a route as JSON between servers
- 🗺️ **In-game map preview** — see a route's waypoints/nodes/center as real blips while editing, instead of a plain coordinate list
- 📜 **Alert log** — a rolling in-dashboard log of recent proximity alerts for reviewing heist attempts after the fact
- 🕵️ **Debug overlay** — a local, visual-only blip overlay showing every currently active ped/vehicle/zone and its alert state
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

## Squads, Idle Variation & Respawn

Walking routes support a few realism options beyond a single patroller:

- **Squad size** (`route.pedCount`, 1–`Config.MaxSquadSize`) — spawns multiple peds walking the
  same waypoint list together, staggered so they don't stack on spawn. All squad members share
  the same alert/investigate state, so a proximity trigger sends the whole squad into combat, not
  just one member.
- **Idle variation** (`route.idleVariation`) — at each waypoint, there's a configurable chance
  (`Config.IdleVariationChance`) the ped pauses for a random scenario animation
  (`Config.IdleVariationScenarios`) for a few seconds before continuing, instead of always
  beelining straight through. Ignored if the route also has a full-time `scenario` set.
- **Auto-respawn** (`route.respawnOnDeath` + `route.respawnDelayMs`) — available on both walking
  and wander routes. If a ped dies, it respawns at its original position after the configured
  delay instead of leaving that patrol slot permanently empty until the whole route despawns and
  re-streams in.

---

## Proximity Alerts (Heist / Defense Hook)

Any **walking**, **wander**, or **alarm** route/zone can be turned into a defensive trigger —
e.g. a bank vault guard (or camera) who reacts when a player gets close during a heist run by
another resource. Enable it in the **Proximity Alert** section of the route editor:

| Setting | Description |
|---|---|
| Enable | Turns the feature on for this route/zone |
| Alert radius | Distance (m) from the route's anchor point that triggers the alert |
| Combat duration | How long the ped(s) stay in combat before reverting to patrol/wander |
| Cooldown | Minimum time between re-triggers, so it doesn't spam every tick while a player lingers |
| Require weapon drawn | Only trigger if the player has a weapon out, to avoid alerting on regular passersby |
| Require line of sight | Only trigger if a guard has a clear sightline to the player (walking/wander only) |
| Escalation (investigate first) | See [Escalation & Alert Groups](#escalation--alert-groups) below |
| Alert group | See [Escalation & Alert Groups](#escalation--alert-groups) below |

When triggered, two things happen simultaneously:

1. **The ped(s) react locally** — walking and wander routes switch the guard(s) into combat
   against the triggering player (`TaskCombatPed`) for the configured duration, then revert to
   their normal patrol/wander/hostility state. (Driving/alarm instances have no ped to react —
   they only do step 2.)
2. **A global server event fires** that any other resource can hook into, with zero coupling to
   this resource:

   ```lua
   -- in your heist resource's server-side code
   AddEventHandler('zotters_npc:alert:triggered', function(routeId, targetSrc, coords)
       if routeId ~= 'bank_vault_guard' then return end
       -- targetSrc: the server id of the player who triggered it
       -- coords: their position (vector3) at trigger time
       -- e.g.: lock the vault door, dispatch police, fail/escalate the heist, etc.
   end)
   ```

There's no permission gate on this event — it's a gameplay signal from a normal player action
(walking near a guard), not an admin operation, so any resource can safely listen for it.

---

## Alarm Zones

An **Alarm Zone** route spawns no ped or vehicle at all — it's a pure trigger volume, meant for
silent alarms, security cameras, or trip-wires. It uses the exact same Proximity Alert system as
guards (radius, cooldown, weapon requirement, groups), just without a "combat reaction" step since
there's nothing to fight with. Pair one with `alert.group` to have a silent alarm wake up a
group of guards elsewhere on the map the instant it's tripped.

## Escalation & Alert Groups

**Escalation** (`alert.investigateFirst`) softens the "instant aimbot guard" problem: instead of
going straight to combat, the first qualifying contact sends the guard(s) to *investigate* the
player's last-known position (`TaskGoToCoordAnyMeans`, no combat, no server event fired yet). Only
a **second** qualifying contact while still investigating escalates to full combat and fires the
server event. Leave it off for guards that should react instantly.

**Alert groups** (`alert.group`) link multiple routes together — e.g. two vault guards and a
silent alarm all sharing `group = 'bank_vault'`. Tripping any one of them:

1. Immediately triggers every other matching-group route this same client already owns
   (investigate-only, since there's no guaranteed live target reference for those other guards).
2. Notifies the server, which rebroadcasts to every other client so guards *they* own can react
   too — with full combat if that client can actually see the triggering player, or investigate
   otherwise.

This means a group of guards reacts together across multiple players' clients without any of them
needing to directly observe the triggering player themselves.

## Anti-Cheat Hardening

Proximity alert *decisions* happen client-side (the observing client checks distance/LOS locally
— there's no way around that for an ambient ped system without server-side hit-scanning every
player every tick). To keep a modified client from abusing that, `server/main.lua` validates every
report before trusting it:

- **Rate limiting** (`Config.AlertMinIntervalPerSourceMs`) — a single client can't flood the event.
- **Coordinate sanity check** (`Config.AlertMaxCoordDriftMeters`) — the claimed target player's
  position is compared against the server's own known position for that player
  (`GetEntityCoords` works server-side for networked peds); a claim that drifts too far is
  rejected and logged, so a spoofed target/position pair can't be used to falsely trigger a
  heist's fail state against a player who isn't actually there.

## Dashboard Tools: Search, Duplicate, Import/Export, Map Preview, Alert Log

- **Search** — filter the route list by label or type as you type.
- **Duplicate** — clone the currently open route (including all points) into a new unsaved copy;
  Save to persist it under a new id.
- **Export / Import** — Export copies the current route as formatted JSON (with a clipboard-copy
  button); Import pastes JSON back in to prefill the editor — handy for backing up a route or
  sharing one between servers.
- **Map preview** — click "🗺️ Preview on map" in the Walking/Driving/Wander/Alarm section to see
  the route's current points as real, numbered in-game blips instead of a plain coordinate list.
- **Alert log** — the 📜 button in the top bar shows a rolling log (`Config.AlertLogMaxEntries`)
  of recent proximity alerts server-side, for reviewing heist attempts after the fact.

## Debug Overlay

Run `/zotters_npc_debug` (`Config.DebugCommand`) to toggle a local, visual-only blip overlay of
every currently active ped/vehicle/alarm zone this client has spawned, colored by state: green
(idle), yellow (investigating), red (alerted/combat). No gameplay effect — it only surfaces
entities already visible in the world, and no ace gate is applied since it can't reveal anything
a player couldn't already see by walking up to it.

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

Config.AlertCheckIntervalMs    = 500            -- proximity alert poll rate
Config.AlertDefaultRadius      = 15.0
Config.AlertDefaultCooldownMs = 30000
Config.AlertDefaultDurationMs = 15000
Config.InvestigateDefaultDurationMs = 8000
Config.AlertMinIntervalPerSourceMs = 1000       -- anti-cheat: rate limit per observing client
Config.AlertMaxCoordDriftMeters    = 25.0       -- anti-cheat: reject implausible claimed positions
Config.AlertLogMaxEntries          = 100

Config.MaxSquadSize          = 4                -- cap on route.pedCount
Config.RespawnDefaultDelayMs = 20000
Config.DebugCommand          = 'zotters_npc_debug'

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
the task if it gets interrupted (e.g. by combat), and optionally respawns dead peds in place.

**Alarm zones** spawn nothing — `client/pathing.lua`'s `SpawnAlarm` just returns a lightweight
instance so the streaming lifecycle treats it like any other route, while `client/spawner.lua`'s
proximity-alert tick does all the actual work.

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

**My proximity alert never fires.**
Check: the route has `alert.enabled = true`, you're within `alert.radius` of the route's *anchor
point* (first waypoint for walking, zone center for wander/alarm — not necessarily the guard's
current position mid-patrol), `alert.requireWeaponDrawn`/`requireLineOfSight` aren't blocking you,
the route isn't still on cooldown from a previous trigger (`alert.cooldownMs`), and if
`investigateFirst` is on, remember the *first* contact only investigates — a second contact is
needed to actually escalate to combat and fire the server event.

**A group-linked guard didn't fully engage, just walked toward the last known position.**
Expected if that guard's client can't see the triggering player's ped entity (too far away/not
streamed in) — it falls back to investigating the coordinates rather than a full `TaskCombatPed`
against an entity it doesn't have a valid reference to.

---

## Project Structure

```
zotters_npc/
├── fxmanifest.lua          # resource manifest
├── config.lua              # all tunables + example route schema
├── shared/
│   └── utils.lua           # shared helpers (global ZottersNPC.Utils)
├── server/
│   ├── main.lua            # authoritative route store + NUI CRUD events, alert validation/log/groups
│   └── storage.lua         # JSON file / oxmysql persistence backends
├── client/
│   ├── main.lua            # dashboard open/close, route cache sync, capture mode, map preview blips
│   ├── spawner.lua         # streaming, proximity alerts, escalation, group propagation, debug overlay
│   ├── pathing.lua         # ped/vehicle spawn + movement loops, squads, respawn, alert reactions
│   ├── weathersync.lua     # Renewed-Weathersync GlobalState adapter
│   └── nui.lua              # NUI callback → server event bridge, alert log relay
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

Made with ❤️ by Zotters

</div>
