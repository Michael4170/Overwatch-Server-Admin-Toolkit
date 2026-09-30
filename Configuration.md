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

On a typical AMP or panel-managed host that means something like:

```
/AMP/arma-reforger/1874900/AReforgerMaster/profile/Overwatch_Admins.json
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

This file is yours, and hand-editing it is always safe — unlike `Overwatch_Admins.json`. See
the traps section below for why that distinction exists.

Overwatch rewrites it in exactly **one** case: when a new version adds a settings block your
file does not have yet. See [Upgrading](#upgrading-an-older-config) below. It never touches a
file it could not parse, and it never changes a value you set.

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
  },

  "discord": {
    "enabled": false,
    "alertWebhook": "",
    "activityWebhook": "",
    "serverName": "",
    "announceStartup": true,

    "alertCooldownSeconds": 300,
    "alertGlobalCooldownSeconds": 60,
    "alertMinConnectedSeconds": 60,
    "alertMaxLength": 300,
    "notifyOnlineStaff": true,

    "logJoins": true,
    "logLeaves": true,
    "logAdminActions": true,
    "showUids": false,
    "batchSeconds": 10
  }
}
```

If your config predates the Discord release it will have no `discord` block. Overwatch adds it
for you on the next start, keeping everything else — see [Upgrading](#upgrading-an-older-config).

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

### Discord

Two webhooks, two purposes. Both optional, and each works without the other.

| Feed | What goes in it | Timing |
|---|---|---|
| `alertWebhook` | player alerts raised in game | **immediate** |
| `activityWebhook` | joins, leaves, admin actions | batched every `batchSeconds` |

**Why two and not one.** An alert staff are meant to react to cannot share a channel with a
stream of joins and leaves. The alert gets buried, staff stop reading the channel, and the
feature quietly stops working without anything looking broken.

#### Getting the webhook URL

In Discord: **Server Settings → Integrations → Webhooks → New Webhook**, pick the channel,
then **Copy Webhook URL**. Paste it straight into the config.

**Treat that URL as a password.** Anyone who has it can post into that channel as often as they
like. It belongs in `Overwatch_Config.json` in your server's profile folder and **nowhere
else** — never in a mod, never in a screenshot, never in a support thread. If one leaks,
delete the webhook in Discord and make a new one; the old URL dies with it.

#### Channel permissions are your job, not Overwatch's

The alert channel should be one only staff can see. **Overwatch cannot check that, and cannot
enforce it.** A webhook posts perfectly happily into a channel the whole server can read. Set
the channel permissions in Discord yourself.

Overwatch never pings anyone. Every post it makes has Discord's mention parsing switched off
entirely, so a player cannot get `@everyone` into your Discord through an alert.

#### Fields

| Field | Meaning |
|---|---|
| `enabled` | master switch. Ships `false`. |
| `alertWebhook` | webhook URL for player alerts. Empty disables that feed. |
| `activityWebhook` | webhook URL for the activity feed. Empty disables that feed. |
| `serverName` | shown in the Discord message, so one channel can serve several servers. |
| `announceStartup` | post a line when the server starts. Cheap proof the integration still works, every restart. Goes to the activity feed. |
| `logJoins` / `logLeaves` | join and leave lines in the activity feed. |
| `logAdminActions` | admin commands that **changed** something. Read-only commands like `!ow players` are never posted. |
| `showUids` | include player UIDs. **Off by default** — the activity channel is usually visible to more people than the alert channel, and a UID is an account identifier. Turn it on if your staff copy UIDs out of Discord for ban commands. |
| `batchSeconds` | how often the activity queue is flushed. Clamped to 1–300. |

The alert abuse controls — `alertCooldownSeconds`, `alertGlobalCooldownSeconds`,
`alertMinConnectedSeconds`, `alertMaxLength`, `notifyOnlineStaff` — belong to the player alert
command and are documented with it.

#### `batchSeconds` is not a performance tuning knob

Discord rate-limits a webhook to roughly **5 requests every 2 seconds**. A 40-player server
going through a map change produces 40 joins in a few seconds; sent one at a time that is an
instant rate-limit, and a rate-limited post is not retried — the event is gone. One message
carrying forty lines stays far under the limit and loses nothing. Leave it at 10 unless you
have a reason.

#### Checking it works

`!ow discordtest` (Owner only) posts a test message. It takes one feed per invocation:

```
!ow discordtest             → alert channel (the default)
!ow discordtest activity    → activity channel
!ow discordtest both        → both
```

The post is asynchronous, so the in-game reply only confirms it was sent. **The server log is
where the answer is.** Every failure gets a line naming the cause:

| Log says | What to do |
|---|---|
| `webhook REJECTED (HTTP 401/403)` | the URL's token is wrong or has been regenerated. Make a new webhook. That feed stays off until you restart with a corrected URL. |
| `webhook NOT FOUND (HTTP 404)` | the webhook was deleted in Discord, or the URL is mistyped. |
| `RATE LIMITED (HTTP 429)` | raise `batchSeconds`. |
| `got no HTTP response at all` | the request never reached Discord. Check that your host permits **outbound HTTPS**. Some rented hosts block it, and nothing in the mod can work around that. |
| `Discord REJECTED the request body (HTTP 400)` | a bug in Overwatch, not in your config. Please report it. |

---

### Upgrading an older config

When a new version of Overwatch adds a settings block — as the Discord release did — your
existing config does not have it. The mod still runs, because missing settings fall back to
their defaults internally. But **the keys are not in your file**, so there is nothing to edit
to switch the new feature on.

Overwatch fixes that for you. On the first start after upgrading:

1. It notices the block is missing.
2. It copies your current file to `Overwatch_Config.json.bak`.
3. It rewrites `Overwatch_Config.json` with the new block added at its defaults, and **every
   value you had set left exactly as it was**.
4. It says so in the log, naming which block was added.

```
CONFIG: added the discord block(s) to $profile:Overwatch_Config.json with default
values — your existing settings were kept, and the previous file was saved as
$profile:Overwatch_Config.json.bak. Edit the new block and restart to use it.
```

Then edit the new block and restart. **You do not need to delete your config** — doing that
throws away your MOTD text and restart times, which is exactly what this exists to prevent.

Three things worth knowing:

- **It only runs when a whole block is missing.** A complete file is never rewritten, so this
  does not churn a backup on every server start.
- **It never touches a config it could not parse.** A file with a typo in it still fails loudly
  and is left alone, so a broken file is never replaced by one you did not write.
- **Your file gets reformatted** to Overwatch's standard layout — same keys, same values, tidier
  indentation. If you keep your own formatting, `Overwatch_Config.json.bak` has it.

Individual *fields* added to an existing block are not backfilled the same way; they take their
default silently and will not appear in your file. The field tables on this page are the
reference for what exists.

---

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
| A settings block missing after an upgrade | block added at defaults, your values kept, previous file saved as `.bak` |
| Restart time unparseable | countdown disabled, the offending value named in the log |
| MOTD enabled but empty | MOTD disabled rather than showing an empty panel |
| `discord` block missing entirely | Discord off, logged, everything else loads normally |
| Webhook URL not shaped like one | that feed disabled, logged **without** printing the URL |
| Webhook rejected by Discord | that feed switched off for the session, cause named in the log |

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

## How Overwatch attaches itself

**You do not add anything to your game mode prefab.** Overwatch ships an override of
`Prefabs/MP/Modes/GameMode_Base.et` — the base prefab every Reforger game mode inherits
from — carrying its components:

```
OW_PermissionManagerComponent
OW_CommandRouterComponent
OW_BanManagerComponent
OW_ConfigManagerComponent
OW_MotdComponent
OW_RestartSchedulerComponent
OW_DiscordComponent
```

Loading the mod is the whole installation. Any game mode built on `GameMode_Base` picks them
up automatically.

The one case where this does not apply is a game mode that does **not** inherit from
`GameMode_Base`. That is unusual, but if Overwatch is loaded and the startup banner never
appears, that is the first thing to check — add the components to that game mode prefab by
hand.

Confirm the mod is live by looking for the startup banner:

```
[Overwatch] Command router ready — 20 commands registered. v0.2.13
```
