# Commands

Every command is prefixed `!ow` or `/ow`. Both are identical — `!ow` is typed in normal chat
and the message is swallowed so other players never see it; `/ow` is the engine chat command
and is consumed by the chat panel.

Player arguments accept **partial names**, case-insensitive. `!ow heal bra` matches "Bravo".
An ambiguous match is refused and lists the candidates rather than guessing.

The one exception to the prefix is `/report`, the public command for players — see
[Everyone](#everyone--no-tier-needed).

Every command is checked server-side. The client never decides anything.

---

## The tier rule

**You may only act on someone at a strictly lower tier.**

An Admin cannot kick, ban, kill, teleport or spectate another Admin. Only an Owner can. This
also means destructive commands cannot target yourself.

Informational commands (`players`, `playerinfo`, `admins`, `bans`) are not restricted this
way.

---

## Everyone — no tier needed

| Command | Aliases | What it does |
|---|---|---|
| `/report <message>` | `!ow alert`, `!ow report` | sends an alert to your admins |

`/report` is the only command a player without a tier can use. Type it in the chat box like
`/ow`; it is consumed by the chat panel and never appears in global chat, so other players do
not see it.

```
/report someone is teamkilling at the airfield
```

What happens:

- The message is posted to the **alert** Discord channel, with the reporter's name, which staff
  are in game, and the player count. Keep that channel staff-only.
- Admins in game get an `[Overwatch] REPORT from <name>: ...` hint in their chat panel (this
  can be turned off with `notifyOnlineStaff`).
- The reporter gets a short confirmation. If no staff are in game and no alert webhook is set,
  they are told reports are not available.

Reports are rate-limited so they cannot be used to spam your staff:

- a message that is empty or longer than `alertMaxLength` is refused;
- a player must have been connected for at least `alertMinConnectedSeconds`;
- there is a per-player cooldown and a server-wide cooldown between reports.

Refused reports tell the player why, and every report — accepted or refused — is logged. Reports
are not mirrored into the activity feed (they already go to the alert feed). Settings are in
[Configuration.md](Configuration.md#discord).

Tell your players it exists — put it in the MOTD or the scheduled chat messages.

---

## Moderator — tier 1

| Command | Aliases | What it does |
|---|---|---|
| `!ow help` | `commands` | lists the commands you personally can use |
| `!ow admins` | — | lists staff and their tiers, and notes that Game Master is automatic for every tier |
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

A session-only Game Master grant for a player who has **no tier**. Every tier (Moderator,
Admin, Owner) already gets Game Master automatically, so both commands refuse a staff member
as the target. The grant is **never written to disk** and is cleared when the player
disconnects — including a rejoin during the same server session.

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
  map is a property of having the editor open. Every tier has Game Master automatically, so
  this only refuses you if you have lost your editor access, and the message says so. Spectate
  does not grant itself the editor — that would be a back door to full Game Master behind a
  button labelled "Spectate".
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
| `!ow discordtest [alert\|activity\|both]` | `dtest` | sends a test post to a Discord webhook |

`!ow grant` writes to `Overwatch_Admins.json` immediately and tells you the new tier comes
with Game Master, because it is the largest single capability the toolkit hands out:

```
Bravo is now Admin (tier 2). They also get Game Master automatically (every tier does).
```

Tier 2 (Admin) and above are also given the engine administrator role, so other mods that
check for an admin recognise them. The `adminTier` setting in `Overwatch_Admins.json` controls
that threshold (default 2; see [Configuration.md](Configuration.md)). Moderators get Game
Master only.

`!ow revoke` **cannot demote an Owner.** See [Configuration.md](Configuration.md) for why,
and for what to do instead.

### `!ow discordtest`

```
!ow discordtest activity
```

Posts one test message to a Discord webhook, so you can prove a URL works without restarting
the server. It takes one feed per run — `alert` or `activity` — and defaults to `alert`.
`both` posts to both channels, but you have to ask for it.

The post is asynchronous, so the command can only tell you it was sent. Check the channel,
and the server log for a `DISCORD` line if nothing arrives. It refuses with a reason if
Discord is off or the feed you asked for has no webhook. Setup is in
[Configuration.md](Configuration.md#discord).

When Discord is on, successful admin commands are also mirrored into the activity feed.
Commands that only read — `help`, `admins`, `players`, `playerinfo`, `bans`, `menu` — and
`discordtest` itself are never posted, and neither are refused or failed commands. `/report`
is not mirrored either, since it has its own alert feed.

Activity-feed posts are sent **silently** (Discord's no-notification flag), so staff are not
pinged on every join, leave or command. Alert posts are normal messages. Neither feed ever
mentions anyone.

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

22 commands, registered at startup:

```
[Overwatch] Command router ready — 22 commands registered.
```

1 public, 8 Moderator, 10 Admin, 3 Owner. If that number is not 22 in your log, you are
running a different build than this document describes.
