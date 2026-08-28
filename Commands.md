# Overwatch — Command Reference

Type `/ow <command>` or `!ow <command>` in chat.

`/ow` is consumed by the chat panel, so **other players never see it** — prefer it. `!ow`
is swallowed client-side before it leaves your machine, so it is also hidden, but it
depends on a hook that `/ow` does not. Keep it in mind as a fallback for working out which
of the two paths is broken.

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

`!ow help` lists only what the caller's own tier can use, so it doubles as a check of what
tier the server thinks you are.

---

## Targeting a player

Anywhere `[player]` appears, the query resolves in this order:

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
is the reason or duration — use a player id there if the name has spaces:

```
!ow players                     → 1: [AAO] Michael.M, 4: Someone Else
!ow kick 4 Team killing
```

---

## The admin menu

`!ow menu` (aliases `gui`, `admin`) opens the in-game admin menu on your client. Tier 1.

Also bound to **numpad minus** by default, rebindable in the game's keybind settings. The
key works while you control a character — not on the deploy or loading screen, where
`/ow menu` still does. Pressing it submits `!ow menu` for you, so it is audited identically.

Three columns:

**Left — panels.** *Admins* lists the configured roster with tiers. *Bans* lists active
bans with remaining time. *Refresh* rebuilds the player list.

**Middle — the player list.** One row per connected player, shown as `id: name`. Click a
row to select it; the selected row is marked with `>`.

**Right — detail and actions.** Selecting a player fills the pane with their id, UID, tier,
faction and alive state, fetched from the server. The six action buttons then act on that
selection:

| Button | Command | Tier |
|---|---|---|
| Heal | `heal` | 1 |
| Go To | `goto` | 2 |
| Bring | `bring` | 2 |
| Kill | `kill` | 2 |
| Kick | `kick` | 2 |
| Ban (Perm) | `ban … perm` | 2 |

Safe actions are the left button column, destructive ones the right.

**Owners** also get a **SET TIER** block under the nav column — Moderator, Admin, Owner and
Revoke — acting on the selected player, equivalent to `!ow grant` and `!ow revoke`. It is
hidden from anyone below Owner (cosmetic only; the server refuses regardless).

Every button routes through the command router exactly as a typed command does, so the
permission check, the tier-targeting rule and the audit log entry are identical. Clicking
with nobody selected prints a prompt rather than doing anything.

> **Destructive actions fire on a single click.** There is no confirmation step yet, and
> the Ban button applies a **permanent** ban with the reason "Banned from admin menu".
> Lift it with `!ow unban`, which takes effect immediately.

The roster is built client-side from replicated names and ids, so Refresh is instant and
costs no server call. Tiers are not available client-side — they live in a server-side
JSON that is deliberately never sent to clients — which is why a row shows name and id
only and the detail pane has to ask the server for the rest.

Close with the Close button or Escape.

---

## Moderator (tier 1)

### `help` — `!ow help`
Lists the commands available at your tier, with usage strings. Alias: `commands`.

### `players` — `!ow players`
Every connected player as `id: name`. Aliases: `who`, `list`.

### `playerinfo` — `!ow playerinfo [player]`
Id, name, identity UID, Overwatch tier, faction, and whether they control a living
character. Aliases: `info`, `pi`. The quickest way to get someone's UID for the config.

### `admins` — `!ow admins`
The configured admin roster with tiers. Names and tiers only — UIDs are deliberately not
printed to chat.

### `bans` — `!ow bans`
Active bans with remaining time. Alias: `banlist`. **Truncates at 8 entries** — chat
replies get unreadable past a handful. The menu's Bans panel and `Overwatch_Bans.json`
have the full list.

### `heal` — `!ow heal [player|me]`
Fully heals the target: all hitzones, bleeding included. **With no argument, heals you.**
Reports the before/after percentage, or says they were already at full health. Fails if
the target has no controlled character.

### `broadcast` — `!ow broadcast [message]`
On-screen announcement to **every** connected player. Aliases: `bc`, `announce`.

The announcement names the admin who sent it — anonymous server-wide messages invite abuse
and confusion. You receive your own broadcast, so there is no separate confirmation.

---

## Admin (tier 2)

### `kill` — `!ow kill [player|me]`
Kills the target's character.

**Deliberately does not default to you.** Unlike `heal`, a destructive command should name
its target — use `!ow kill me` explicitly. The death is attributed to a Game Master
instigator, so it does not trigger friendly-fire punishment or pollute the kill feed.
Blocked against equal or higher tiers; self-kill stays allowed.

### `goto` — `!ow goto [player]`
Teleports **you** to the target. Aliases: `tp`, `tpto`.

### `bring` — `!ow bring [player]`
Teleports the target **to you**. Alias: `summon`.

Both refuse when either character is in a vehicle — teleporting a seated character desyncs
the compartment. Dismount first. The moved character lands about 2 m to the side of the
anchor so the two don't spawn inside each other. The reply reports the distance moved.

### `kick` — `!ow kick [player] [reason]`
Disconnects a player immediately, with **no reconnect block** — a kick means "stop that and
come back". Use `ban` for anything that should stick.

First argument is the target, everything after is the reason. Reason defaults to
"No reason given" and is recorded with the actor and target UID.

### `ban` — `!ow ban [player] [duration] [reason]`
Bans by identity UID and kicks immediately.

| Duration | Meaning |
|---|---|
| `30s` | seconds |
| `45m` | minutes |
| `2h` | hours |
| `7d` | days |
| `perm` / `permanent` / `0` | never expires |

**The duration is required.** A permanent ban has to be typed as `perm`, so you cannot make
one by forgetting an argument.

```
!ow ban 4 7d Repeated team killing
!ow ban Someone perm Cheating
```

Refused if you cannot target them (equal or higher tier), or if their UID cannot be
resolved. In Workbench the ban applies for the session but is **not saved** — the reply
says so.

### `unban` — `!ow unban [uid|name]`
Lifts a ban. Tries an exact UID first, then a unique name match against the stored ban
records. An ambiguous name lists the matches rather than guessing.

```
!ow unban a1b2c3d4-1111-2222-3333-444455556666
!ow unban Someone
```

The unbanned player can reconnect **immediately** — see the ban design note in
Configuration.md for why that matters.

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
work. Their client is told the new tier straight away, so menu and player-list actions
update **without a reconnect**.

Refused if: you target yourself, the tier is outside 1–3, they already hold that tier, or
you are trying to demote an existing Owner.

### `revoke` — `!ow revoke [player]`
Removes a connected player's admin entry and persists it. Alias: `demote`.

Refused if: you target yourself, they are not an admin, or **they are an Owner** — remove
an Owner by editing `Overwatch_Admins.json` directly.

---

## Why grant and revoke need the player online

A tier is keyed to an identity UID, and resolving one requires the player to be connected.
To pre-authorise someone offline, add them to `Overwatch_Admins.json` by hand.

---

## Every command is logged

Whether it succeeded, was refused or was not recognised:

```
[Overwatch] CMD OK      | player 1 ([AAO] Michael.M, UID bbe7b313-...) | '!ow menu'
[Overwatch] CMD DENIED  | player 4 (Someone Else, UID a1b2c3d4-...)    | 'kick'
[Overwatch] CMD UNKNOWN | player 1 ([AAO] Michael.M, UID bbe7b313-...) | 'nonsense'
```

Denials are preceded by a `DENY REASON` line naming the actual cause — see the
troubleshooting table in Configuration.md.
