# Overwatch — Command Reference

Type `/ow <command>` or `!ow <command>` in chat.

`/ow` is consumed by the chat panel, so **other players never see it** — prefer it. `!ow`
is sent as a normal chat message and is swallowed client-side before it leaves your
machine, so it is also hidden, but it depends on a hook that `/ow` does not. Keep `!ow` in
mind as a fallback for diagnosing which of the two paths is broken.

Bare `!ow` with no command shows help.

---

## All commands

| Command | Aliases | Tier | Usage |
|---|---|---|---|
| `help` | `commands` | 1 | `!ow help` |
| `menu` | `gui`, `admin` | 1 | `!ow menu` |
| `players` | `who`, `list` | 1 | `!ow players` |
| `playerinfo` | `info`, `pi` | 1 | `!ow playerinfo [player]` |
| `admins` | — | 1 | `!ow admins` |
| `bans` | `banlist` | 1 | `!ow bans` |
| `heal` | — | 1 | `!ow heal [player\|me]` |
| `broadcast` | `bc`, `announce` | 1 | `!ow broadcast [message]` |
| `kill` | — | 2 | `!ow kill [player\|me]` |
| `goto` | `tp`, `tpto` | 2 | `!ow goto [player]` |
| `bring` | `summon` | 2 | `!ow bring [player]` |
| `kick` | — | 2 | `!ow kick [player] [reason]` |
| `ban` | — | 2 | `!ow ban [player] [duration] [reason]` |
| `unban` | — | 2 | `!ow unban [uid\|name]` |
| `grant` | `promote`, `setrank` | 3 | `!ow grant [player] [1\|2\|3]` |
| `revoke` | `demote` | 3 | `!ow revoke [player]` |

Tier 1 = Moderator, 2 = Admin, 3 = Owner. Checks are `>=`, so an Owner can run everything.

`!ow help` lists only the commands the caller's own tier can actually use, so it doubles
as a quick check of what tier the server thinks you are.

---

## Targeting a player

Anywhere `[player]` appears, the query is resolved in this order:

1. **`me`** or **`self`** — you
2. **A numeric player id** — `!ow players` lists them
3. **A full name**, case-insensitive
4. **A partial name**, case-insensitive — must be **unique**, otherwise the ambiguity is
   reported rather than guessed at

Names containing spaces work **without quoting**, because everything after the command
word is joined back together:

```
!ow heal [AAO] Michael.M
```

The exception is `kick` and `ban`, where the **first** argument is the target and the rest
are the reason/duration — use a player id there if the name has spaces:

```
!ow players                     → 1: [AAO] Michael.M, 4: Someone Else
!ow kick 4 Team killing
```

---

## Moderator (tier 1)

### `help` — `!ow help`
Lists the commands available at your tier, with usage strings. Alias: `commands`.

### `menu` — `!ow menu`
Opens the admin menu on your client. Aliases: `gui`, `admin`.

The menu covers the things that are **not** about one specific player — the admin roster,
the ban list, connected players. Player-targeted actions live in the vanilla player list
instead, where you right-click a player. Each panel asks the server to run the equivalent
command and returns its text, so a panel and a typed command produce identical results,
permission checks and log entries.

Close with the Close button or Escape.

### `players` — `!ow players`
Every connected player as `id: name`. Aliases: `who`, `list`. Use this to grab an id for
commands where a name would be awkward.

### `playerinfo` — `!ow playerinfo [player]`
Detailed report on one player: id, name, identity UID, Overwatch tier, faction, and
whether they currently control a living character. Aliases: `info`, `pi`.

The quickest way to get someone's UID for the admin config.

### `admins` — `!ow admins`
The configured admin roster with tiers. Names and tiers only — UIDs are deliberately not
printed to chat.

### `bans` — `!ow bans`
Active bans with remaining time. Alias: `banlist`. **Truncates at 8 entries** — chat
replies get unreadable past a handful. Read `Overwatch_Bans.json` for the full list.

### `heal` — `!ow heal [player|me]`
Fully heals the target: all hitzones, bleeding included. **With no argument, heals you.**
Reports the before/after percentage, or tells you they were already at full health.

Fails if the target has no controlled character (dead or not deployed).

### `broadcast` — `!ow broadcast [message]`
On-screen announcement to **every** connected player. Aliases: `bc`, `announce`.

The announcement names the admin who sent it — anonymous server-wide messages invite abuse
and confusion. You receive your own broadcast, so there is no separate confirmation.

---

## Admin (tier 2)

### `kill` — `!ow kill [player|me]`
Kills the target's character.

**Deliberately does not default to you.** Unlike `heal`, a destructive command should
always name its target — use `!ow kill me` explicitly. The death is attributed to a Game
Master instigator, so it does not trigger friendly-fire punishment or pollute the kill
feed. Blocked against equal or higher tiers; self-kill stays allowed.

### `goto` — `!ow goto [player]`
Teleports **you** to the target. Aliases: `tp`, `tpto`.

### `bring` — `!ow bring [player]`
Teleports the target **to you**. Alias: `summon`.

Both refuse when either character is in a vehicle — teleporting a seated character
desyncs the compartment state. Get them out first. The moved character lands about 2 m to
the side of the anchor so the two don't spawn inside each other. The reply reports the
distance moved.

### `kick` — `!ow kick [player] [reason]`
Disconnects a player immediately, with **no reconnect block** — a kick means "stop that
and come back". Use `ban` for anything that should stick.

First argument is the target, everything after is the reason. Reason defaults to
"No reason given" and is recorded in the log with the actor and target UID.

### `ban` — `!ow ban [player] [duration] [reason]`
Bans by identity UID and kicks immediately.

| Duration | Meaning |
|---|---|
| `30s` | seconds |
| `45m` | minutes |
| `2h` | hours |
| `7d` | days |
| `perm` / `permanent` / `0` | never expires |

**The duration is required.** A permanent ban has to be typed as `perm` rather than being
what you get by forgetting an argument.

```
!ow ban 4 7d Repeated team killing
!ow ban Someone perm Cheating
```

Refused if you cannot target them (equal or higher tier), or if their UID cannot be
resolved. In Workbench the ban applies for the session but is **not saved** — the reply
tells you so.

### `unban` — `!ow unban [uid|name]`
Lifts a ban. Tries an exact UID first, then falls back to a unique name match against the
stored ban records. An ambiguous name lists the matches rather than guessing.

```
!ow unban a1b2c3d4-1111-2222-3333-444455556666
!ow unban Someone
```

The unbanned player can reconnect **immediately** — see the ban design note in
CONFIGURATION.md for why that matters.

---

## Owner (tier 3)

### `grant` — `!ow grant [player] [1|2|3]`
Sets a connected player's tier and writes it to `Overwatch_Admins.json`. Aliases:
`promote`, `setrank`.

```
!ow grant Someone 2
!ow grant 4 1
```

The last argument is the tier; everything before it is the player, so names with spaces
work. Their client is told the new tier straight away, so player-list actions update
**without a reconnect**.

Refused if: you target yourself, the tier is outside 1–3, they already hold that tier, or
you are trying to demote an existing Owner.

### `revoke` — `!ow revoke [player]`
Removes a connected player's admin entry and persists it. Alias: `demote`.

Refused if: you target yourself, they are not an admin, or **they are an Owner** — remove
an Owner by editing `Overwatch_Admins.json` directly.

---

## Why grant and revoke need the player online

A tier is keyed to an identity UID, and resolving one requires the player to be connected.
To pre-authorise someone who is offline, add them to `Overwatch_Admins.json` by hand.

---

## Every command is logged

Whether it succeeded, was refused or was not recognised:

```
[Overwatch] CMD OK      | player 1 ([AAO] Michael.M, UID bbe7b313-...) | '!ow menu'
[Overwatch] CMD DENIED  | player 4 (Someone Else, UID a1b2c3d4-...)    | 'kick'
[Overwatch] CMD UNKNOWN | player 1 ([AAO] Michael.M, UID bbe7b313-...) | 'nonsense'
```

Denials are preceded by a `DENY REASON` line naming the actual cause — see the
troubleshooting table in CONFIGURATION.md.
