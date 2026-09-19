# Configuration

Overwatch uses three JSON files in the server profile directory. None is ever sent to a
client.

| File | Purpose | Written by Overwatch? |
|---|---|---|
| `Overwatch_Admins.json` | tiers and the `gmTier` threshold | **yes** — rewritten on every grant/revoke |
| `Overwatch_Bans.json` | active bans | **yes** |
| `Overwatch_Config.json` | optional features: restart countdown, MOTD | **no** — created once, then read only |

That last column is the important one and is explained under
[Two traps that will cost you an evening](#two-traps-that-will-cost-you-an-evening).

---

## Where `$profile:` actually resolves

This is the single most common source of "I edited the file and nothing changed".

`$profile:` is not a fixed path. It resolves to whatever profile directory the running
process was started with, and that is **different** between Workbench and a dedicated
server.

| Where you run it | `$profile:` resolves to |
|---|---|
| **Workbench / Play in Editor** | your local Reforger profile, typically `%LOCALAPPDATA%\Arma Reforger\profile\` |
| **Dedicated server** | the directory passed as `-profile` on the command line |

On a typical panel-managed host that means something like:

```
container/profile/profile/Overwatch_Admins.json
```

If you are unsure, do not guess. Start the server once and read the log — Overwatch prints
the full resolved path when it loads or creates the file:

```
[Overwatch] Loaded 4 admins from $profile:Overwatch_Admins.json
[Overwatch] CONFIG: loaded $profile:Overwatch_Config.json (version 1).
```

**Editing the wrong copy is the number one config problem.** A Workbench test and a live
server are reading two entirely separate files.

---

## `Overwatch_Admins.json`

Created automatically on first start if absent. Full shape:

```json
{
  "version": 1,
  "gmTier": 2,
  "admins": {
    "bbe7b313-580b-4350-9709-18583139e0f7": {
      "tier": 3,
      "name": "Michael"
    },
    "7d0919b9-4c2a-4f13-9b81-2e5a71c04d3a": {
      "tier": 2,
      "name": "Bravo"
    }
  }
}
```

**`admins` is an object keyed by UID, not a list.** The UID is the key itself — there is no
`uid` field inside each entry. A file written as an array will not load, and Overwatch will
fail closed with nobody holding any permission, which looks identical to a missing file.

If you are editing by hand, the safest approach is to let the server write the file first and
then add entries in the shape it produced.

### Fields

| Field | Type | Meaning |
|---|---|---|
| `version` | int | config format version. Leave it alone. |
| `gmTier` | int | lowest tier that gets Game Master automatically. Default `2`. |
| *(key)* | string | Bohemia identity UID. **This is what authorises.** |
| `tier` | int | 1 Moderator, 2 Admin, 3 Owner |
| `name` | string | label for logs and menus only. Never used for permission. |

`name` is cosmetic. Changing it does nothing but change what the log says. The UID key is
what makes the system safe against name spoofing — two players can share a display name, but
not a UID.

### Finding a UID

Three ways, in order of convenience:

1. `!ow playerinfo <partial name>` in game — prints the UID.
2. `!ow players` — lists everyone with their UID.
3. The server log at connect: the `IdentityId=` field on the join line.

A UID looks like `bbe7b313-580b-4350-9709-18583139e0f7`. It is stable for that Bohemia
account forever.

---

## `gmTier`

`gmTier` is the lowest tier that receives the vanilla Game Master editor automatically, on
connect and on every respawn.

| Value | Effect |
|---|---|
| `0` | automatic grant **off**. Nobody gets it from their tier. `!ow gm` still works. |
| `1` | Moderators and above |
| `2` | **default** — Admins and Owners |
| `3` | Owners only |

The test in code is `tier >= gmTier`, with `gmTier = 0` meaning off. Set it to `3` if you
want Admins to have the command set but not world-editing power.

`!ow admins` reports the current threshold in its output, so you can check it in game
without touching the file:

```
Game Master: tier 2 (Admin) and above (gmTier 2).
```

`!ow grant` also tells you what the new tier will and will not receive, at the moment you
grant it — so promoting someone to Admin on a server with `gmTier 3` says so explicitly
rather than leaving you to wonder.

### Why this exists as a separate setting

Game Master is not a command. It is the vanilla editor, and it bypasses Overwatch entirely.
Someone with it can spawn, delete and teleport anything, and **none of it is written to the
Overwatch log**. The grant line is the last record you get.

That is why the threshold is configurable and why it is documented this prominently.
Handing out Admin is a decision about commands. Handing out Game Master is a much larger
decision, and `gmTier` is what keeps them separate.

---

## `Overwatch_Config.json`

Optional features live here. It is created on first start with **every feature off and every
content field empty**, so a fresh install changes nothing until you decide otherwise.

Overwatch **never rewrites this file**. It is yours, and hand-editing it is always safe —
unlike `Overwatch_Admins.json`. See the traps section below for why that distinction exists.

```json
{
  "version": 1,

  "restart": {
    "enabled": false,
    "useUtc": false,
    "times": [],
    "countdownSeconds": 300,
    "message": "SERVER RESTART IN {time}"
  },

  "motd": {
    "enabled": false,
    "title": "",
    "subtitle": "",
    "bannerImage": "{33D1918C78EF7999}UI/Textures/OW_MotdBanner.edds",
    "rules": [],
    "footer": "",
    "showOn": "welcome",
    "showDelaySeconds": 2
  }
}
```

### Restart countdown

**Overwatch does not restart your server.** It shows players a countdown so they are not
caught mid-firefight. Your host — AMP, a panel, a cron job — still does the restarting, which
means the two can never disagree about when it happens.

| Field | Meaning |
|---|---|
| `enabled` | master switch. Ships `false`. |
| `useUtc` | `false` = times are in the **server machine's local timezone**. `true` = UTC. |
| `times` | 24-hour `"HH:MM"` strings, e.g. `["04:00", "10:00", "16:00", "22:00"]` |
| `countdownSeconds` | how long the counter is on screen. `300` = the final five minutes. |
| `message` | counter text. `{time}` becomes the remaining time as `M:SS`. |

Set `times` to match whatever your host already does. Nothing enforces that they agree —
Overwatch only warns.

**`useUtc` is the field most likely to catch you out**, because a rented server is often in a
different timezone from the community using it. You do not have to wait for a restart to
check: the startup log prints the current time as Overwatch sees it, and the next scheduled
restart.

```
[Overwatch] RESTART: it is now 19:49:55 (server local time). Next scheduled restart 19:54, in 4:05.
```

If that time is not your wall clock, set `useUtc` and convert your times.

A time that does not parse, or an empty `times` with `enabled` set, disables the countdown and
says so in the log — rather than silently skipping one restart out of four.

### Message of the day

A rules panel shown once per session, before the player spawns in.

| Field | Meaning |
|---|---|
| `enabled` | master switch. Ships `false`. |
| `title` | heading under the banner. Your server's name works well here. |
| `subtitle` | one line under that — community name, Discord link, whatever. |
| `bannerImage` | banner texture. See below. |
| `rules` | the body. One array entry per line, in order. |
| `footer` | a line pinned at the bottom, e.g. `"Press Deploy when ready"`. |
| `showOn` | `"welcome"`, `"briefing"` or `"firstSpawn"`. See below. |
| `showDelaySeconds` | applies to `"firstSpawn"` only. |

An empty string in `rules` is a deliberate blank line. That is how you separate sections —
there is no markup, on purpose.

```json
"rules": [
  "1. No team killing.",
  "2. Follow your squad lead.",
  "",
  "COMMS",
  "3. Keep the command net clear."
]
```

#### `showOn`

| Value | When the panel appears |
|---|---|
| `"welcome"` | **default.** The first screen of the deploy flow, before the map. The player has arrived and has not started choosing anything yet, so covering the screen costs them nothing. |
| `"briefing"` | the deploy map screen. Works, but the panel covers the map, so players cannot see where they are deploying while it is up. |
| `"firstSpawn"` | after the player takes control of a character. |

Three values because which screens exist depends on your game mode and on what other UI mods
you run. If `"welcome"` produces nothing on your server, try `"firstSpawn"` — it is a config
edit, not a bug report.

The panel dismisses itself: on a screen trigger when the player moves on, and on
`"firstSpawn"` after a short hold. It is shown **once per session**, so players who have read
it are not shown it again every time they redeploy.

#### `bannerImage`

This is an Enfusion **resource name**, not a URL and not a file path:

```
"bannerImage": "{33D1918C78EF7999}UI/Textures/OW_MotdBanner.edds"
```

A `https://` link, a Discord CDN link or a path into your profile folder **cannot work** —
Reforger has no way to fetch an image at runtime for UI. There is no setting that changes
this.

The value shipped in the template points at Overwatch's own banner, so the feature looks
finished out of the box. To use your own artwork:

1. Put the image in **your own mod** — one your players already download. The panel is drawn
   on the client, so the texture has to exist on the client, not just the server.
2. Import it in the Workbench (power-of-two dimensions, e.g. 1024×256).
3. Right-click the resulting `.edds` → **Copy Resource Name**, and paste that whole string in.

Leave `bannerImage` empty for a text-only panel; the banner area simply disappears.

An enabled MOTD with no `title` and no `rules` disables itself and says so, rather than
showing every player an empty box.

---

## Two traps that will cost you an evening

### 1. `SaveConfig()` rewrites the whole admin file from memory

Any command that changes admin data — `!ow grant`, `!ow revoke` — rewrites
`Overwatch_Admins.json` **in full** from what the server currently holds in memory.

So if you hand-edit that file while the server is running, and then anyone runs a grant or a
revoke, your edit is silently overwritten. No error. No warning. The file just reverts.

**Stop the server before hand-editing `Overwatch_Admins.json`.** Every time.

If you must change something live, use the commands rather than the file.

**`Overwatch_Config.json` is deliberately exempt.** Overwatch only ever creates it, never
rewrites it, which is precisely so that the file you edit by hand is safe from this. The two
files are separated for that reason — roster data the mod owns, feature settings you own.
Changes to it still need a restart to take effect.

### 2. An Owner cannot be demoted by command

`!ow revoke` refuses to act on a tier 3. This is deliberate: it means a single compromised
Owner account cannot strip every other Owner and take the server.

The consequence is that removing an Owner requires stopping the server, editing
`Overwatch_Admins.json` by hand, and starting it again. That is the intended cost.

An Owner *can* create another Owner. Creating is reversible by hand; being locked out of
your own server is not.

---

## Failure behaviour

Overwatch **fails closed**. Every failure path grants nobody anything, and every optional
feature stays off.

| Situation | What happens |
|---|---|
| Admin file missing | template written, log line tells you the path, nobody has permission |
| Admin file malformed | load aborts, error logged, nobody has permission |
| UID not in the list | denied, logged as `UID not in the list` |
| Tier too low | denied, logged as `tier too low` |
| Player not signed in | denied, logged as `not signed in` |
| `Overwatch_Config.json` missing | template written with everything off, logged, nothing enabled |
| `Overwatch_Config.json` malformed | **not overwritten**, error logged, all optional features off |
| Restart time unparseable | countdown disabled, the offending value named in the log |
| MOTD enabled but empty | MOTD disabled rather than showing an empty panel |

Those denial reasons are logged as distinct strings on purpose. They are completely different
problems that otherwise look identical from in game — "the command did nothing".

The malformed-config case is worth calling out: Overwatch will **not** replace a file it
cannot parse. That file is your configuration with a typo in it, and replacing it with
defaults would throw your settings away at the exact moment you could least afford it. Fix the
typo and restart.

---

## `Overwatch_Bans.json`

Written and maintained by `OW_BanManagerComponent`. You should not need to edit it by hand,
but it is plain JSON if you do — and the same "stop the server first" rule applies.

Bans are keyed by UID, store the reason, the banning admin and the expiry, and survive
restarts. `!ow bans` lists them; `!ow unban` removes one.

---
