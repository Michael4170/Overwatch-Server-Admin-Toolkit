# Overwatch — Configuration

Everything needed to get the toolkit running on a server, in the order you'll need it.

---

## 1. Add the components

All three are `SCR_BaseGameModeComponent` subclasses and belong on your **GameMode
prefab**, not on a world entity.

| Component | Required | Provides |
|---|---|---|
| `OW_PermissionManagerComponent` | Yes | Tiers, UID resolution, `grant` / `revoke` |
| `OW_CommandRouterComponent` | Yes | Command parsing, the permission gate, logging |
| `OW_BanManagerComponent` | For bans | Ban list, enforcement on connect, `kick` |

Without the ban manager, `ban` / `unban` / `bans` / `kick` reply
`Ban manager not found — add the OW_BanManagerComponent to your GameMode prefab`.

> The router's `OnGameModeStart` runs **before** the permission manager's, so there is a
> brief window at startup where commands are refused. The permission manager re-pushes
> tiers to everyone already connected once it finishes loading, so this corrects itself.

Confirm on startup:

```
[Overwatch] Command router ready — 16 commands registered.
[Overwatch] Loaded 2 admin entries from config.
```

No `[Overwatch]` lines at all means the components are not on the prefab.

---

## 2. Admin config — `$profile:Overwatch_Admins.json`

Written automatically on first run if absent. The template grants **nobody** anything:

```json
{
    "version": 1,
    "admins": {
        "PASTE_YOUR_IDENTITY_UID_HERE": { "tier": 3, "name": "REPLACE_WITH_YOUR_NAME" }
    }
}
```

| Field | Meaning |
|---|---|
| key | The player's **identity UID**. This is the only thing used for authentication. |
| `tier` | `1` Moderator, `2` Admin, `3` Owner |
| `name` | A label for your readability. **Never** used for auth. |

### Finding your identity UID

Join the server and look for your connect line in the log:

```
### Updating player: PlayerId=1, Name=YourName, IdentityId=bbe7b313-580b-4350-9709-18583139e0f7
```

That `IdentityId` is the key to use.

> **Never use `00000000-0000-0000-0000-000000000000`.**
> That is what the backend returns when nobody is signed in, so it belongs to *every*
> unauthenticated session rather than to one player — putting it in the config would hand
> that tier to anyone who joins signed-out. The toolkit rejects it on load and logs an
> error. If you see it in your connect line, sign in and reconnect to get a real UID.

### A filled-in example

```json
{
    "version": 1,
    "admins": {
        "bbe7b313-580b-4350-9709-18583139e0f7": { "tier": 3, "name": "Michael" },
        "a1b2c3d4-1111-2222-3333-444455556666": { "tier": 2, "name": "Someone Else" },
        "9f8e7d6c-5555-4444-3333-222211110000": { "tier": 1, "name": "Trial Mod" }
    }
}
```

Restart after editing. Once one Owner exists, add further admins in-game with
`!ow grant` instead of editing the file — the file is rewritten automatically.

---

## 3. Tiers and the two hierarchy rules

| Tier | Name | Can do |
|---|---|---|
| 0 | Player | Nothing. Every command refused. |
| 1 | Moderator | Look, heal, announce, open the menu |
| 2 | Admin | Everything above, plus kick, ban, kill, teleport |
| 3 | Owner | Everything, plus `grant` and `revoke` |

Permission checks are `>=` the command's minimum tier, with two rules layered on top:

**You may only act on someone at a strictly lower tier.**
An Admin (2) cannot kick or ban another Admin — only an Owner (3) can. Ordinary players
are tier 0, so any admin can action them. Self-targeting is blocked for destructive
commands.

**Owners cannot revoke or demote other Owners.**
That requires editing `Overwatch_Admins.json` by hand, which needs file access to the
host. The asymmetry is deliberate: an Owner may freely *create* peers but not remove
them, so a single compromised Owner account cannot strip everyone else and take the
server. If your community would rather Owners could demote each other, remove the
`TIER_OWNER` check in `RevokeTier`.

---

## 4. Ban list — `$profile:Overwatch_Bans.json`

Created on first ban. Keyed by identity UID, enforced on connect via
`OnPlayerAuditSuccess` with `OnPlayerConnected` as a backup. Expired records are pruned
automatically when the file loads.

```json
{
    "version": 1,
    "bans": {
        "a1b2c3d4-1111-2222-3333-444455556666": {
            "name": "Someone Else",
            "reason": "Team killing",
            "bannedBy": "Michael",
            "bannedByUid": "bbe7b313-580b-4350-9709-18583139e0f7",
            "bannedAt": 1755000000,
            "expiresAt": 1755086400
        }
    }
}
```

`expiresAt` of `0` means permanent. Times are Unix seconds.

**Banned players are kicked with no engine-level reconnect timeout, on purpose.** Passing
the ban duration to the engine would also arm *its* reconnect block, which no script can
clear — `!ow unban` would then appear to do nothing until the original sentence expired.
The Overwatch list is the single revocable source of truth, and enforcement re-runs on
every connection attempt, so a still-banned player is kicked again regardless.

The trade-off: a banned player briefly connects before being kicked, rather than being
refused at the network layer.

### Keys the ban file will never accept

| Key | Why it is rejected |
|---|---|
| `00000000-0000-...` | Shared by every signed-out session — would kick all of them |
| `LOCAL_<playerId>` | Workbench dev key; player ids are recycled between sessions |

Both are dropped on load with a log line, and `SaveBans` will not write either. A
`LOCAL_` ban still works **in memory** so the ban flow stays testable in Workbench — it
just does not survive the session, and the command reply says so.

---

## 5. Failure behaviour

The two config files fail in **opposite** directions, deliberately:

| File | On corruption | Result |
|---|---|---|
| `Overwatch_Admins.json` | Fails **closed** | Nobody has admin |
| `Overwatch_Bans.json` | Fails **open** | Nobody is banned |

In both cases the server keeps running and nobody is wrongly punished. Both log loudly.

---

## 6. Player list actions (optional)

Register the `OW_*PlayerListAction` classes in an override of `PlayerListActions.conf` to
get Overwatch entries when right-clicking a player in the vanilla player list.

| Action class | Tier | Notes |
|---|---|---|
| `OW_HealPlayerListAction` | 1 | Can target yourself |
| `OW_KillPlayerListAction` | 2 | |
| `OW_GotoPlayerListAction` | 2 | Teleport yourself to them |
| `OW_BringPlayerListAction` | 2 | Teleport them to you |
| `OW_KickPlayerListAction` | 2 | `m_sReason` attribute |
| `OW_BanPlayerListAction` | 2 | `m_sDuration` + `m_sReason` attributes |

The ban action carries a **fixed** duration per entry, since a context menu cannot prompt
for one. Register several — "Ban 1h", "Ban 7d" — rather than making one entry do
everything. There is no confirmation prompt and a misclick is expensive, so consider
registering only short durations here and leaving permanent bans to the typed command.

> **Testing solo is misleading.** `CanBeShown` runs client-side and hides any action that
> cannot target yourself. With one player connected every row in the player list is you,
> so the menu correctly shows Heal alone. Verify these with a second player.

---

## 7. Troubleshooting

### "You don't have permission for that"

The caller always sees the same vague refusal, because the specific reason would tell an
ordinary player about the shape of your admin roster. **The server log has the real
reason** on a `DENY REASON` line:

| Log says | Meaning | Fix |
|---|---|---|
| `resolved to an EMPTY identity UID` | Not signed in to the backend | Sign in, reconnect |
| `admin config is NOT LOADED` | File missing/malformed, or component missing | Check the parse error above it |
| `is not in the admin list (N entries loaded)` | Config loaded, UID isn't in it | Check for a typo'd UID |
| `is tier X but this command requires tier Y` | Working as intended | Raise their tier |

### `/ow` isn't recognised

Look for `Chat command registered: '/ow ...'` in the log — it should appear around the
time you connect, without anyone typing first. `!ow` works regardless, so if `!ow` works
and `/ow` doesn't, the problem is the chat-command registration rather than the toolkit.

### An admin entry seems ignored

`$profile:` resolves to **different directories** for a Workbench session and a
standalone or dedicated server. A config that works in one can be entirely absent in the
other, which reads as "no permission for any command". The path is printed in the
startup log line.

### `LOCAL_1` in the admin config

Allowed, and intended for Workbench testing — it grants to whoever holds that player id,
which is always you in a solo session. It logs a warning on load. **Remove it before
shipping a config to a real server.** The `LOCAL_` fallback is compiled out of non-dev
builds entirely (`#ifdef WORKBENCH`).
