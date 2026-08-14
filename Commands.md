# Overwatch — Command Reference

Every command works two ways:

| Form | Visible to others? |
|---|---|
| `/ow <command>` | **No** — consumed by the chat panel, never sent as a message |
| `!ow <command>` | Yes — appears in chat like any other message |

Both reach the same server-side router, with the same permission checks and the same
audit log entry. Use `/ow` unless you have a reason not to.

Responses appear as an on-screen hint panel, visible only to the admin who ran the
command.

---

## Targeting a player

Any command that takes a player accepts:

| Form | Example |
|---|---|
| `me` or `self` | `/ow heal me` |
| Player id | `/ow heal 3` |
| Exact name, case-insensitive | `/ow heal [AAO] Michael.M` |
| Any unique fragment of a name | `/ow heal mich` |

If a fragment matches more than one player the command refuses and lists the matches
rather than guessing. Use `/ow players` to see ids.

---

## Moderator (tier 1+)

| Command | Aliases | Description |
|---|---|---|
| `/ow help` | `commands` | List commands available at your tier |
| `/ow admins` | | List configured admins and tiers |
| `/ow players` | `who`, `list` | List connected players with their ids |
| `/ow playerinfo [player]` | `info`, `pi` | UID, tier, faction and state for one player |
| `/ow heal [player]` | | Fully heal a player. No argument heals you. Reports before/after health |
| `/ow broadcast [message]` | `bc`, `announce` | Announce to all players, signed with your name |
| `/ow bans` | `banlist` | List active bans |
| `/ow menu` | `gui`, `admin` | Open the admin menu |

## Admin (tier 2+)

| Command | Aliases | Description |
|---|---|---|
| `/ow kill [player]` | | Kill a player's character. Requires an explicit target |
| `/ow goto [player]` | `tp`, `tpto` | Teleport yourself to a player |
| `/ow bring [player]` | `summon` | Teleport a player to you |
| `/ow kick [player] [reason]` | | Disconnect a player. No reconnect block |
| `/ow ban [player] [duration] [reason]` | | Ban for a duration, or `perm` |
| `/ow unban [uid or name]` | | Lift a ban |

### Ban durations

`30s`, `45m`, `2h`, `7d`, or `perm`.

The duration is **required**. There is no way to issue a permanent ban by forgetting an
argument — `perm` has to be typed.

---

## Player list actions

Open the player list, right-click a player, and Overwatch actions appear alongside the
vanilla ones. Each click runs the equivalent command, so permission checks and audit
logging are identical to typing it.

| Action | Tier | Notes |
|---|---|---|
| Overwatch: Heal | Moderator | Can target yourself |
| Overwatch: Kill | Admin | |
| Overwatch: Teleport to player | Admin | You → them |
| Overwatch: Bring player here | Admin | Them → you |
| Overwatch: Kick | Admin | Reason set in config |
| Overwatch: Ban … | Admin | Duration set per config entry |

Actions you do not have the tier for are hidden. Actions that cannot target yourself
are hidden on your own row — so with only one player connected, only Heal appears.
That is expected.

Ban durations are fixed per config entry, because a context menu cannot prompt for one.
Several entries can be registered — "Ban 1h", "Ban 24h", "Ban permanently".

---

## Rules

**Tier hierarchy.** You can only kick, ban or kill someone at a strictly lower tier than
your own. Ordinary players are tier 0, so any admin can action them. An Admin cannot ban
another Admin — only an Owner can. `heal` is the exception and has no tier check on the
target.

**Overwatch tiers are separate from Reforger's admin system.** A vanilla server admin
who is not in `Overwatch_Admins.json` is tier 0 to Overwatch, and can be actioned by
Overwatch admins.

**Everything is logged.** Every dispatch, denial, unknown command, kick, ban and ban
enforcement is written to the server log prefixed with `[Overwatch]`, including the
acting admin's UID and the verbatim command.

---

## Known limitations

- Banned players see Reforger's generic "kicked" message, not the ban reason. The ban
  itself works correctly; only the notification is generic. Use `/ow bans` for details.
- `!ow` commands are visible in chat to other players. Use `/ow` to avoid this.
- There is no keybind for the admin menu. Use `/ow menu`.
