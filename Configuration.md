# Overwatch — Configuration

Everything needed to get the toolkit running on a server, in the order you'll need it.

---

## 1. Installation

**There is nothing to attach.** Overwatch overrides the vanilla game mode prefab
`Prefabs/MP/Modes/GameMode_Base.et` and adds its three components there, so any game mode
inheriting from `GameMode_Base` — Conflict, Campaign, Game Master and the great majority
of custom modes — picks the toolkit up automatically. Add the mod to your server's mod
list and start it.

The components it attaches:

| Component | Provides |
|---|---|
| `OW_PermissionManagerComponent` | Tiers, UID resolution, `grant` / `revoke` |
| `OW_CommandRouterComponent` | Command parsing, the permission gate, logging |
| `OW_BanManagerComponent` | Ban list, enforcement on connect, `kick` |

Confirm on startup:

```
[Overwatch] Command router ready — 16 commands registered.
[Overwatch] Loaded 2 admin entries from config.
```

> The router's `OnGameModeStart` runs **before** the permission manager's, so there is a
> brief window at startup where commands are refused. The permission manager re-pushes
> tiers to everyone already connected once it finishes loading, so this corrects itself.

### When the override doesn't apply

Two situations need attention, and both look identical from in-game — **no `[Overwatch]`
lines at all in the server log**:

**Another mod also overrides `GameMode_Base.et`.** Only one override of a given prefab
applies, so whichever loses simply isn't there. Check your mod list for anything else
touching the base game mode. Load order decides it.

**Your game mode doesn't inherit from `GameMode_Base`.** A mode built on its own root
prefab won't see the override. Attach the three components above to that game mode's
prefab by hand and everything else in this guide applies unchanged.

If you are running the ban commands, `OW_BanManagerComponent` must be present or
`ban` / `unban` / `bans` / `kick` reply
`Ban manager not found — add the OW_BanManagerComponent to your GameMode prefab`.

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
| key | The player's **identity UID**. The only thing used for authentication. |
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
> error. If you see it on your connect line, sign in and reconnect to get a real UID.

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
host. The asymmetry is deliberate: an Owner may freely *create* peers but not remove them,
so a single compromised Owner account cannot strip everyone else and take the server. If
your community would rather Owners could demote each other, remove the `TIER_OWNER` check
in `RevokeTier`.

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

Both are dropped on load with a log line, and `SaveBans` will not write either. A `LOCAL_`
ban still works **in memory** so the ban flow stays testable in Workbench — it just does
not survive the session, and the command reply says so.

---

## 5. Failure behaviour

The two config files fail in **opposite** directions, deliberately:

| File | On corruption | Result |
|---|---|---|
| `Overwatch_Admins.json` | Fails **closed** | Nobody has admin |
| `Overwatch_Bans.json` | Fails **open** | Nobody is banned |

In both cases the server keeps running and nobody is wrongly punished. Both log loudly.

---

## 6. Game Master access

> **This is ON by default.** With no configuration at all, every **Admin (tier 2) and
> Owner (tier 3) is automatically granted Game Master.** You do not add anything to turn
> it on — you add `gmTier` to restrict or disable it.

Overwatch hands the vanilla editor (Game Master) to admins automatically. The `gmTier`
field in `Overwatch_Admins.json` controls the threshold, and **is absent from a
freshly-generated config**, in which case the default of `2` applies:

| `gmTier` | Effect |
|---|---|
| *(field absent)* | **Admin and above** — the default |
| `0` | Off. Overwatch never touches editor access. |
| `1` | Moderator and above |
| `2` | Admin and above (same as absent) |
| `3` | Owner only |

To change it, add the field at the **top level**, alongside `version` — not inside
`admins`:

```json
{
    "version": 1,
    "gmTier": 3,
    "admins": { ... }
}
```

Anything outside `0-3` disables the feature and logs an error rather than guessing.

The field appears in your config on its own the first time `!ow grant` or `!ow revoke`
rewrites the file. That changes nothing — it just makes the current setting visible.

**Check the startup log rather than the file** to know what is actually in force. An
absent field and `"gmTier": 2` produce the identical line.

Confirm on startup:

```
[Overwatch] Game Master auto-grant ENABLED for tier 2 (Admin) and above.
[Overwatch] GAME MASTER | granted to <name> (UID ...) | tier 3 (Owner) meets gmTier 2
```

### Two things to weigh before leaving this on

**Game Master is more power than every Overwatch command combined** — spawn anything,
delete anything, teleport anyone, edit any entity. It collapses the tier model: an Admin who
cannot `!ow kick` an equal-tier Admin can do considerably worse to them from the editor.

**Editor actions are not in the Overwatch audit log.** Nothing done in Game Master passes
through the command router, so the most powerful capability on your server is the only one
with no record. Set `gmTier` to `3`, or `0`, if that trade is wrong for you.

### It grants, it never revokes

`!ow revoke` does **not** remove Game Master. Other systems — the scenario, the server
config's own admin list, another mod — may have granted editor access for their own reasons,
and silently stripping it would be a baffling bug with no visible cause. A demoted admin
keeps GM until they reconnect. Kick them after revoking if that matters.

### Timing

The grant is attempted on connect, on audit success, on the post-load config sweep, and
after a successful `!ow grant`. The per-player editor manager is not guaranteed to exist at
any single one of those, so a missing one is treated as "not yet" rather than a failure.

---

## 7. The admin menu keybind

Bound to **numpad minus** by default, and rebindable in the game's own keybind settings like
any other action.

It is declared in `Configs/System/chimeraInputCommon.conf`, an override of the vanilla input
config, as a single action inside `CharacterGeneralContext`:

```
ActionManager {
 Contexts {
  ActionContext CharacterGeneralContext {
   Actions {
    Action OW_OpenAdminMenu {
     InputSource InputSourceValue "{...}" { Input "keyboard:KC_SUBTRACT" }
    }
   }
  }
 }
}
```

The key does not open the menu directly — it submits `!ow menu`, so it takes the same
router, the same server-side tier check and the same audit entry as typing it.

**It only works while you control a character.** `CharacterGeneralContext` is not active on
the deploy or loading screen, so the key does nothing there; `/ow menu` still works. To
change that, move the action to a broader context such as `IngameContext` — but test menu
stacking first.

An action declared at the config's **root** `Actions` level instead of inside a context will
never fire. It belongs to no context, and `ActionManager` only fires actions whose context is
currently active. This costs nothing and produces no error, so it is easy to miss.

---

## 8. Admin menu defaults

The menu's action buttons carry fixed arguments, since a button cannot prompt for a
duration or a reason. They are constants at the top of `OW_AdminMenu`:

| Constant | Default | Effect |
|---|---|---|
| `BAN_BUTTON_DURATION` | `perm` | Duration applied by the Ban button |
| `BAN_BUTTON_LABEL` | `Ban (Perm)` | The button's caption |
| `BAN_BUTTON_REASON` | `Banned from admin menu` | Reason stored in the record |
| `KICK_BUTTON_REASON` | `Kicked from admin menu` | Reason logged for a menu kick |

> **There is currently no confirmation step**, so the Ban button writes a permanent record
> on a single click. If that is not what your admins should have one click away, set
> `BAN_BUTTON_DURATION` to something like `1h` and leave long bans to the typed command,
> where the admin has to spell out both duration and reason.

Changing the label and the duration together matters — the caption is its own constant, so
setting the duration alone leaves a button that says one thing and does another.

---

## 9. Player list actions (optional)

The `OW_*PlayerListAction` classes are registered in
`Configs/System/Actions/PlayerListActions.conf`, which ships with the mod — right-clicking
a player in the vanilla player list gives Overwatch entries with no setup.

| Action class | Tier | Notes |
|---|---|---|
| `OW_HealPlayerListAction` | 1 | Can target yourself |
| `OW_KillPlayerListAction` | 2 | |
| `OW_GotoPlayerListAction` | 2 | Teleport yourself to them |
| `OW_BringPlayerListAction` | 2 | Teleport them to you |
| `OW_KickPlayerListAction` | 2 | `m_sReason` attribute |
| `OW_BanPlayerListAction` | 2 | `m_sDuration` + `m_sReason` attributes |

To change durations or reasons, or to add entries, override that config. The ban action
carries a **fixed** duration per entry — register several ("Ban 1h", "Ban 7d") rather than
making one entry do everything. There is no confirmation prompt on this route and the
framework provides no way to add one, so consider registering only short durations here
and leaving permanent bans to the typed command or the menu.

> **Testing solo is misleading.** `CanBeShown` runs client-side and hides any action that
> cannot target yourself. With one player connected every row in the player list is you,
> so the menu correctly shows Heal alone. Verify these with a second player.

---

## 10. Troubleshooting

### No `[Overwatch]` lines in the log at all

The game mode override isn't applying. Either another mod is also overriding
`GameMode_Base.et`, or your game mode doesn't inherit from it — see
[§1 When the override doesn't apply](#when-the-override-doesnt-apply).

### "You don't have permission for that"

The caller always sees the same vague refusal, because the specific reason would tell an
ordinary player about the shape of your admin roster. **The server log has the real
reason** on a `DENY REASON` line:

| Log says | Meaning | Fix |
|---|---|---|
| `resolved to an EMPTY identity UID` | Not signed in to the backend | Sign in, reconnect |
| `admin config is NOT LOADED` | File missing or malformed | Check the parse error above it |
| `is not in the admin list (N entries loaded)` | Config loaded, UID isn't in it | Check for a typo'd UID |
| `is tier X but this command requires tier Y` | Working as intended | Raise their tier |

### `/ow` isn't recognised

Look for `Chat command registered: '/ow ...'` in the log — it should appear around the
time you connect, without anyone typing first. `!ow` works regardless, so if `!ow` works
and `/ow` doesn't, the problem is the chat-command registration rather than the toolkit.

### The menu opens but the player list is empty

Look for `'PlayerListRoot' not found — the player list will be empty`. The menu treats
every widget as optional, so a missing or misnamed one is logged and skipped rather than
being fatal. The same applies to each `Act*` button and to `TitleText` / `TierBadge`.

### The keybind does nothing

Look for `Admin menu keybind listener attached` on connect, then `Admin menu keybind pressed`
when you press it. Attached but never pressed means the action is not declared, or its
context is not active — see §7. Note it is inactive on the deploy screen by design.

### An admin entry seems ignored

`$profile:` resolves to **different directories** for a Workbench session and a standalone
or dedicated server. A config that works in one can be entirely absent in the other, which
reads as "no permission for any command". The path is printed in the startup log line.

### `LOCAL_1` in the admin config

Allowed, and intended for Workbench testing — it grants to whoever holds that player id,
which is always you in a solo session. It logs a warning on load. **Remove it before
shipping a config to a real server.** The `LOCAL_` fallback is compiled out of non-dev
builds entirely (`#ifdef WORKBENCH`).
