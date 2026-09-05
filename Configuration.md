# Configuration

Overwatch reads and writes two JSON files in the server profile directory. Neither is ever
sent to a client.

| File | Purpose |
|---|---|
| `Overwatch_Admins.json` | tiers and the `gmTier` threshold |
| `Overwatch_Bans.json` | active bans |

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
```

**Editing the wrong copy is the number one config problem.** A Workbench test and a live
server are reading two entirely separate files.

---

## `Overwatch_Admins.json`

Created automatically on first start if absent. Full shape:

```json
{
  "gmTier": 2,
  "admins": [
    {
      "uid": "bbe7b313-580b-4350-9709-18583139e0f7",
      "name": "Michael",
      "tier": 3
    },
    {
      "uid": "7d0919b9-4c2a-4f13-9b81-2e5a71c04d3a",
      "name": "Bravo",
      "tier": 2
    }
  ]
}
```

### Fields

| Field | Type | Meaning |
|---|---|---|
| `gmTier` | int | lowest tier that gets Game Master automatically. Default `2`. |
| `admins[].uid` | string | Bohemia identity UID. **This is what authorises.** |
| `admins[].name` | string | label for logs and menus only. Never used for permission. |
| `admins[].tier` | int | 1 Moderator, 2 Admin, 3 Owner |

`name` is cosmetic. Changing it does nothing but change what the log says. `uid` is the key,
and it is what makes the system safe against name spoofing — two players can share a display
name, but not a UID.

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

## Two traps that will cost you an evening

### 1. `SaveConfig()` rewrites the whole file from memory

Any command that changes admin data — `!ow grant`, `!ow revoke` — rewrites
`Overwatch_Admins.json` **in full** from what the server currently holds in memory.

So if you hand-edit the file while the server is running, and then anyone runs a grant or a
revoke, your edit is silently overwritten. No error. No warning. The file just reverts.

**Stop the server before hand-editing.** Every time.

If you must change something live, use the commands rather than the file.

### 2. An Owner cannot be demoted by command

`!ow revoke` refuses to act on a tier 3. This is deliberate: it means a single compromised
Owner account cannot strip every other Owner and take the server.

The consequence is that removing an Owner requires stopping the server, editing
`Overwatch_Admins.json` by hand, and starting it again. That is the intended cost.

An Owner *can* create another Owner. Creating is reversible by hand; being locked out of
your own server is not.

---

## Failure behaviour

Overwatch **fails closed**. Every failure path grants nobody anything.

| Situation | What happens |
|---|---|
| File missing | template written, log line tells you the path, nobody has permission |
| File malformed | load aborts, error logged, nobody has permission |
| UID not in the list | denied, logged as `UID not in the list` |
| Tier too low | denied, logged as `tier too low` |
| Player not signed in | denied, logged as `not signed in` |

Those four denial reasons are logged as distinct strings on purpose. They are four
completely different problems that otherwise look identical from in game — "the command did
nothing".

---

## `Overwatch_Bans.json`

Written and maintained by `OW_BanManagerComponent`. You should not need to edit it by hand,
but it is plain JSON if you do — and the same "stop the server first" rule applies.

Bans are keyed by UID, store the reason, the banning admin and the expiry, and survive
restarts. `!ow bans` lists them; `!ow unban` removes one.

---

## Required components

Overwatch needs three components on your game mode prefab. Missing any one of them is
logged at startup.

```
OW_PermissionManagerComponent
OW_CommandRouterComponent
OW_BanManagerComponent
```

Confirm the mod is live by looking for the startup banner:

```
[Overwatch] Command router ready — 20 commands registered. v0.7.3
```

