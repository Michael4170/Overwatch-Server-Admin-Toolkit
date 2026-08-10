# Overwatch — Command Reference

All Overwatch commands are typed into normal in-game chat and are prefixed with `!ow` or `/ow`.
Everything is parsed, permission-checked and executed **server-side only** — the client
never decides whether a command is allowed.

Replies come back as an on-screen hint panel visible only to the admin who issued the
command, so responses stay readable against terrain and don't leak into public chat.

```
/ow <command> [arguments]
!ow <command> [arguments]
```


/ow <command> — recommended, hidden from chat

!ow <command> — still works, visible to other players

Typing a bare `!ow` is treated as `!ow help`. Command names and aliases are
case-insensitive.

---

## Permission tiers

Admins are defined by identity UID in `$profile:Overwatch_Admins.json`. Each entry
carries a numeric tier:

| Tier | Name | Typical scope |
|:---:|---|---|
| 0 | *(player)* | No Overwatch access |
| 1 | Moderator | Information and non-destructive commands |
| 2 | Admin | Destructive commands, teleport, kick/ban |
| 3 | Owner | Everything, including actioning other Admins |

Two rules govern every command:

1. **Minimum tier** — you must hold at least the command's listed tier to run it.
2. **Strictly-lower targeting** — commands that act on another player only work if that
   player's tier is *lower than yours*. An Admin cannot kick, ban or kill another Admin;
   only an Owner can. Ordinary players are tier 0, so any admin can action them.

The permission file **fails closed**: if it's missing or malformed, nobody is granted
anything and the reason is written to the server log.

---

## Targeting a player

Any command that takes a `[player]` argument accepts:

| Form | Example | Notes |
|---|---|---|
| `me` or `self` | `!ow heal me` | Always resolves to the caller |
| Player ID | `!ow heal 3` | IDs come from `!ow players` |
| Exact name | `!ow heal Michael` | Case-insensitive |
| Partial name | `!ow heal mich` | Must match exactly one connected player |
| Names with spaces | `!ow heal John Smith` | Remaining arguments are rejoined automatically |

If a partial name matches more than one player, the command **fails with an error**
rather than guessing. Only connected players can be resolved by name.

---

## Quick reference

| Command | Aliases | Tier | Usage |
|---|---|:---:|---|
| `help` | `commands` | 1 | `/ow help` |
| `admins` | — | 1 | `/ow admins` |
| `players` | `list`, `who` | 1 | `/ow players` |
| `playerinfo` | `info`, `pi` | 1 | `/ow playerinfo [player]` |
| `heal` | — | 1 | `/ow heal [player\|me]` |
| `kill` | — | 2 | `/ow kill [player]` |
| `goto` | `tp` | 2 | `/ow goto [player]` |
| `bring` | `summon` | 2 | `/ow bring [player]` |
| `broadcast` | `bc`, `announce` | 2 | `/ow broadcast [message]` |
| `kick` | — | 2 | `/ow kick [player] [reason]` |
| `ban` | — | 2 | `/ow ban [player] [duration] [reason]` |
| `unban` | — | 2 | `/ow unban [uid\|name]` |
| `bans` | — | 2 | `/ow bans` |

---

## General

### `/ow help` — *alias `commands`* — Tier 1

Lists every command **the caller is actually allowed to use**, with usage strings. A
Moderator and an Owner see different lists, so the output doubles as a tier check.

### `/ow admins` — Tier 1

Lists the configured admin roster with display names and tiers, e.g. `Michael [T3]`.
UIDs are deliberately **not** printed in-game.

---

## Information

### `/ow players` — *aliases `list`, `who`* — Tier 1

Flat roster of connected players as `id: name`. This is the lookup step before any
targeted command — IDs are always unambiguous where names may not be.

### `/ow playerinfo [player]` — *aliases `info`, `pi`* — Tier 1

Detailed readout for one player: player ID, identity UID, Overwatch tier, faction, and
alive/dead state.

---

## Player management

### `/ow heal [player|me]` — Tier 1

Fully heals the target's character — all hitzones, bleeding included. With no argument,
heals the caller. Fails clearly if the target has no controlled character (dead or not
yet spawned).

### `/ow kill [player]` — Tier 2

Kills the target's character. The death is attributed to the issuing admin as a Game
Master instigator, so it appears correctly in the kill feed and logs.

Self-kill is allowed. Killing another player requires them to be at a strictly lower
tier. The command verifies the target actually died and reports if it didn't.

---

## Teleport

Both teleport commands require the *other* player to be at a strictly lower tier.

### `/ow goto [player]` — *alias `tp`* — Tier 2

Teleports **you** to the target player.

### `/ow bring [player]` — *alias `summon`* — Tier 2

Teleports the target player **to you**.

---

## Communication

### `/ow broadcast [message]` — *aliases `bc`, `announce`* — Tier 2

Sends a message to every connected player. Everything after the command word is treated
as the message, so no quoting is needed.

```
/ow broadcast Server restarting in 10 minutes.
```

---

## Moderation

### `/ow kick [player] [reason]` — Tier 2

Immediately disconnects a player. The first argument is the target; everything after it
is the reason. If no reason is given, `No reason given` is recorded.

The kick is written to the server log with the target's name, UID, the issuing admin and
the reason.

### `/ow ban [player] [duration] [reason]` — Tier 2

Bans a player and kicks them immediately.

```
/ow ban Bob 7d griefing
/ow ban 4 perm cheating
```

**The duration argument is mandatory.** There is no way to hand out a permanent ban by
forgetting an argument — permanent must be typed as `perm`.

Accepted durations:

| Format | Meaning |
|---|---|
| `30s` | seconds |
| `45m` | minutes |
| `2h` | hours |
| `7d` | days |
| `perm` | permanent |

Bans are enforced in two layers: the ban is recorded in `$profile:Overwatch_Bans.json`
and checked when a player's identity is verified on connect, **and** the remaining
duration is passed to the engine's reconnect timeout at kick time.

Banned players see Reforger's generic "kicked" message rather than a ban reason. 
The ban itself works correctly; only the notification is generic.

Overwatch keeps its own ban store rather than using the engine's backend ban service, so
bans work on local and offline-hosted servers.

### `/ow unban [uid|name]` — Tier 2

Lifts a ban by identity UID or by the name recorded at ban time. Use `!ow bans` to see
the list first.

### `/ow bans` — Tier 2

Lists all active bans: name, UID, reason, who issued it, and remaining time. Expired
entries are ignored.

Unlike the permission file, the ban list **fails open** — a corrupt or unreadable ban
file bans nobody and logs loudly. A malformed config should never lock a playerbase out
of a server.

---

## Logging

Every command — successful, denied, unknown, or a usage error — is written to the server
log with the outcome, the caller's player ID, name and UID, and the verbatim command
string:

```
[Overwatch] CMD OK      | player 2 (Michael, UID bbe7...) | '!ow heal Bob'
[Overwatch] CMD DENIED  | player 5 (Dave, UID a91c...)    | 'kill'
[Overwatch] CMD UNKNOWN | player 5 (Dave, UID a91c...)    | 'banana'
```

Kick and ban actions additionally emit a dedicated `KICK` / `BAN` log line.

---

## Configuration files

| File | Purpose | On failure |
|---|---|---|
| `$profile:Overwatch_Admins.json` | Admin UIDs, display names, tiers | Fails **closed** — nobody has access |
| `$profile:Overwatch_Bans.json` | Active bans | Fails **open** — nobody is banned |

Both are created and maintained on the server. Neither requires a client download.
