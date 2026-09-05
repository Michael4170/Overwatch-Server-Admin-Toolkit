# Commands

Every command is prefixed `!ow` or `/ow`. Both are identical — `!ow` is typed in normal chat
and the message is swallowed so other players never see it; `/ow` is the engine chat command
and is consumed by the chat panel.

Player arguments accept **partial names**, case-insensitive. `!ow heal bra` matches "Bravo".
An ambiguous match is refused and lists the candidates rather than guessing.

Every command is checked server-side. The client never decides anything.

---

## The tier rule

**You may only act on someone at a strictly lower tier.**

An Admin cannot kick, ban, kill, teleport or spectate another Admin. Only an Owner can. This
also means destructive commands cannot target yourself.

Informational commands (`players`, `playerinfo`, `admins`, `bans`) are not restricted this
way.

---

## Moderator — tier 1

| Command | Aliases | What it does |
|---|---|---|
| `!ow help` | `commands` | lists the commands you personally can use |
| `!ow admins` | — | lists staff and their tiers, plus the current `gmTier` |
| `!ow heal <player>` | — | restores health |
| `!ow players` | `who`, `list` | every connected player with id and UID |
| `!ow playerinfo <player>` | `info`, `pi` | one player in detail — UID, tier, position, health |
| `!ow broadcast <message>` | `bc`, `announce` | server-wide message to everyone |
| `!ow bans` | `banlist` | active bans with reason, admin and expiry |
| `!ow menu` | `gui`, `admin` | opens the admin menu |

`!ow help` is filtered to your tier. A Moderator running it does not see the ban commands at
all, which keeps it short and stops people trying things they cannot do.

`!ow menu` exists because the keybind (**Numpad minus**) is bound to
`CharacterGeneralContext` and therefore does not fire on the deploy or loading screen. Type
`/ow menu` there instead.

---

## Admin — tier 2

Everything above, plus:

| Command | Aliases | What it does |
|---|---|---|
| `!ow kill <player>` | — | kills the player |
| `!ow goto <player>` | `tp`, `tpto` | teleports **you** to them |
| `!ow bring <player>` | `summon` | teleports **them** to you |
| `!ow spectate <player>` | `spec`, `watch` | attaches your camera to them |
| `!ow unspectate` | `unspec`, `stopspectate` | ends spectate |
| `!ow kick <player> [reason]` | — | disconnects them; reason is shown to them |
| `!ow ban <player> <duration> [reason]` | — | bans by UID |
| `!ow unban <uid>` | — | lifts a ban |
| `!ow gm <player>` | `givegm`, `gamemaster` | grants Game Master for this session only |
| `!ow ungm <player>` | `takegm` | removes a session grant |

### Ban durations

```
!ow ban Bravo 7d repeated team killing
```

| Format | Meaning |
|---|---|
| `30s` | seconds |
| `45m` | minutes |
| `2h` | hours |
| `7d` | days |
| `perm` | permanent |

The reason is optional but is stored, logged and shown to the player on their next join
attempt. Write one.

### `!ow gm` and `!ow ungm`

A session-only Game Master grant. It is **never written to disk** and is cleared when the
player disconnects — including a rejoin during the same server session.

`!ow ungm` restores the editor access the player had *before* the grant, rather than forcing
it off. That distinction matters: a normal player with Arma Vision keeps Arma Vision after
you take their temporary Game Master away.

Read the Game Master section of [README.md](README.md) before handing this out. It is more
power than every other command here combined, and nothing done with it appears in the
Overwatch log.

### Spectate

```
!ow spectate Bravo
!ow unspectate
```

Attaches your camera to that player and follows them anywhere on the map. **Escape** also
ends it and returns you to your character.

Two constraints:

- **It requires Game Master.** The entity streaming that lets you see a player across the
  map is a property of having the editor open. If your tier is below `gmTier` you are
  refused, with that as the stated reason. Spectate does not grant itself the editor — that
  would be a back door to full Game Master behind a button labelled "Spectate".
- **The target is not notified.** This is deliberate and the server log is the only record.
  See the covert-spectate section in [README.md](README.md); it is a community policy
  question you should answer before enabling this.

There is no next/previous cycling yet. Reopen the menu to change target.

---

## Owner — tier 3

Everything above, plus:

| Command | Aliases | What it does |
|---|---|---|
| `!ow grant <player> <tier>` | `promote`, `setrank` | sets someone's tier (1–3) |
| `!ow revoke <player>` | `demote` | removes staff status entirely |

`!ow grant` writes to `Overwatch_Admins.json` immediately and reports whether the new tier
also receives Game Master automatically, given the current `gmTier`:

```
Bravo is now Admin (tier 2). They also get Game Master automatically (gmTier 2).
```

or, on a server configured for Owners only:

```
Bravo is now Admin (tier 2). NOTE: gmTier is 3 (Owner), so tier 2 does NOT get Game Master automatically.
```

That second line exists because a silent correct refusal and a silent bug look identical
from outside.

`!ow revoke` **cannot demote an Owner.** See [Configuration.md](Configuration.md) for why,
and for what to do instead.

---

## The admin menu

**Numpad minus**, or `/ow menu`.

A live player list with a row per player and action buttons alongside. Buttons are hidden by
tier, so a Moderator sees Heal and Info and nothing else.

Every button runs the same command as typing it — same permission check, same tier rule,
same log entry. There is no second execution path, which is why the menu cannot do anything
chat cannot.

The Spectate button toggles: it reads **Spectate** when idle and **Stop Spectate** while
active.

**There are no confirmation dialogs.** Kill, Kick, Ban and Revoke fire on a single click,
and the Ban button is **permanent**. A stray click writes a real permanent ban record.
`!ow unban` lifts it, but the ban happens instantly. This is the top item on the fix list.

---

## Command count

20 commands, registered at startup:

```
[Overwatch] Command router ready — 20 commands registered. v0.7.3
```

8 Moderator, 10 Admin, 2 Owner. If that number is not 20 in your log, you are running a
different build than this document describes.
