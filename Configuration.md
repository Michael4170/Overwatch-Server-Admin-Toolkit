# Overwatch — Configuration Guide

Everything Overwatch needs lives on the server. There is no client-side setup, no
optional client mod, and nothing for your players to download beyond the automatic mod
sync every Reforger server already performs.

Setup is two steps: **load the mod → add your UID.**

---

## 1. Load the mod

Add Overwatch to the `mods` array in your server config:

```json
{
  "game": {
    "mods": [
      {
        "modId": "REPLACE_WITH_WORKSHOP_ID",
        "name": "Overwatch — Server Admin Toolkit"
      }
    ]
  }
}
```

Clients receive the mod automatically on connect. Nothing else in your server config
needs to change.

---

## 2. Verify it loaded

**There is nothing to attach.** Overwatch ships its three components already on the base
game mode, so they come up automatically on any scenario that inherits from it:

| Component | Responsibility |
|---|---|
| `OW_PermissionManagerComponent` | Loads the admin config, answers all permission checks |
| `OW_CommandRouterComponent` | Parses `/ow` chat, gates by tier, dispatches commands |
| `OW_BanManagerComponent` | Loads and enforces the ban list, performs kicks |

Confirm all three initialised by watching the server log at startup:

```
[Overwatch] Loaded 2 admin entries from config.
[Overwatch] Command router ready — 13 commands registered.
[Overwatch] No ban list found — starting with an empty list.
```

If those lines never appear, your scenario is almost certainly running a custom game mode
prefab that doesn't derive from the base one. In that case — and only that case — add the
three components to your own game mode prefab manually.

---

## 3. Configure admins

### First run

Start the server once with no config present. Overwatch writes a template to the profile
directory and **grants nobody anything** until you edit it:

```
[Overwatch] No admin config found. Creating default at
            $profile:Overwatch_Admins.json — add your UID and restart.
```

The generated file looks like this:

```json
{
  "version": 1,
  "admins": {
    "PASTE_YOUR_IDENTITY_UID_HERE": {
      "tier": 3,
      "name": "REPLACE_WITH_YOUR_NAME"
    }
  }
}
```

The placeholder key is deliberately ignored on load, with a warning — a template left
unedited never becomes a working admin entry.

`$profile:` resolves to whatever directory your server was launched with via the
`-profile` parameter.

### File format

The `admins` object is keyed by **identity UID**. Each entry carries a numeric tier and
a display label:

```json
{
  "version": 1,
  "admins": {
    "bbe7b313-580b-4350-9709-18583139e0f7": {
      "tier": 3,
      "name": "Michael"
    },
    "a91c4f22-7d10-4e83-b5c2-9f0e11d47a3b": {
      "tier": 2,
      "name": "Dave"
    },
    "3f8b0c15-2a44-4d91-8e77-c0b5e2194d6a": {
      "tier": 1,
      "name": "Sam"
    }
  }
}
```

| Field | Type | Notes |
|---|---|---|
| *(key)* | string | Identity UID. The only thing that actually grants permission. |
| `tier` | int | `1` Moderator, `2` Admin, `3` Owner |
| `name` | string | Label only — shown by `!ow admins`. Never used for matching. |

The `name` field has **no security function whatsoever**. Renaming an entry changes
nothing; only the UID key matters.

### Finding a UID

- **From the server console** — identity IDs appear in the connection log lines when a
  player joins. The exact format varies between game builds.
- **From `!ow playerinfo [player]`** — the cleanest route once you have one admin
  configured. This is how you'd add everyone after yourself.
- Overwatch's own log lines include the UID on every command, so a single command from a
  prospective moderator surfaces theirs.

Bootstrapping the first Owner is the one case where you need the server console, since
you can't run `playerinfo` before you have any permissions.

### Applying changes

The config is read **once, at game mode start**. Editing the JSON on a running server has
no effect until restart.

---

## Tiers

| Tier | Name | Grants |
|:---:|---|---|
| 0 | *(player)* | Nothing. Any `!ow` command returns a permission denial. |
| 1 | Moderator | `help`, `admins`, `players`, `playerinfo`, `heal` |
| 2 | Admin | The above plus `kill`, `goto`, `bring`, `broadcast`, `kick`, `ban`, `unban`, `bans` |
| 3 | Owner | Everything, and the only tier that can action an Admin |

**Targeting rule:** commands that act on another player require that player to be at a
*strictly lower* tier. An Admin cannot kick, ban or kill another Admin. Give tier 3 only
to people you'd trust to action your own account.

### Overwatch tiers are not server admin

Overwatch is completely parallel to Reforger's built-in admin system. It never reads or
writes the engine's admin list.

- An Overwatch Moderator is, to the engine, an ordinary player — **no Game Master, no
  vanilla admin tools.**
- A server config admin has **zero** Overwatch commands unless their UID is also in
  `Overwatch_Admins.json`.

This is the point of the design: you can hand someone `heal` and the player roster
without handing them the world editor.

**One gotcha worth acting on.** Overwatch's tier checks only know about Overwatch tiers.
A vanilla server admin who isn't listed in your JSON is tier 0 to Overwatch, which means
an Overwatch Admin can kick or ban them. The fix is procedural: list every server config
admin in `Overwatch_Admins.json` at tier 3 as well, so both hierarchies agree.

---

## Ban storage

Bans are written to `$profile:Overwatch_Bans.json`, created and maintained automatically.
Hand-editing it is supported but unnecessary — `!ow ban`, `!ow unban` and `!ow bans`
cover normal operation.

Overwatch keeps its own ban store rather than using the engine's backend ban service, so
bans function on locally hosted and offline servers where that service is unavailable.

Enforcement is two-layered: the ban is checked when a connecting player's identity is
verified, **and** the remaining duration is handed to the engine's reconnect timeout at
kick time.

---

## Failure behaviour

The two config files fail in **opposite directions**, deliberately:

| File | On corruption | Reasoning |
|---|---|---|
| `Overwatch_Admins.json` | Fails **closed** — nobody has permissions | A broken file must never grant power |
| `Overwatch_Bans.json` | Fails **open** — nobody is banned | A broken file must never lock out your playerbase |

Both log loudly. Neither silently half-loads.

```
[Overwatch] FAILED to parse Overwatch_Admins.json — file is malformed.
            NO permissions granted. Fix or delete the file and restart.
```

If you ever see that line, every `!ow` command on the server is denied — including your
own. Delete the file to regenerate the template, or fix the JSON, then restart.

---

## Troubleshooting

| Symptom | Cause |
|---|---|
| Every command denied, including your own | Config failed to load, or your UID isn't in it. Check the startup log. |
| `Loaded 0 admin entries` | The placeholder UID was never replaced. |
| `kick` / `ban` say *managers unavailable* | The ban manager didn't initialise — usually a custom game mode that doesn't inherit from the base. |
| No response at all to `!ow` | Router didn't start — check for `Command router ready` at startup. |
| Config edits do nothing | Changes require a server restart. |
| Fewer than 13 commands registered | A command failed to register; check for duplicate-name warnings in the log. |

Every command is logged with its outcome, the caller's ID, name and UID, and the verbatim
command string, which makes the log the fastest place to diagnose anything:

```
[Overwatch] CMD OK      | player 2 (Michael, UID bbe7...) | '!ow heal Bob'
[Overwatch] CMD DENIED  | player 5 (Dave, UID a91c...)    | 'kill'
```

---

## Security notes

- **All permission checks are server-side.** Clients never receive the admin list, and
  the router's tier gate runs before any command body executes.
- **Do not commit your `Overwatch_Admins.json` to a public repository.** UIDs aren't
  secret, but publishing who holds which tier tells anyone probing your server exactly
  who to target.
- Commands are authenticated by verified identity UID, not by name — impersonating a
  moderator's display name grants nothing.
