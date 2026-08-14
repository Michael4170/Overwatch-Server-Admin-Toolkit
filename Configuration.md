# Overwatch — Setup and Configuration

Built for Arma Reforger **1.8**. Server-side: no separate client download beyond the
normal mod sync.

---

## 1. Install

Add the mod to your server's mod list.

## 2. Add the components

Open your GameMode prefab and add all three:

- `OW_PermissionManagerComponent`
- `OW_CommandRouterComponent`
- `OW_BanManagerComponent`

Miss one and commands will report which manager is unavailable.

## 3. First run

Start the server once. `Overwatch_Admins.json` is generated in your server profile
directory with a placeholder entry.

Find your Identity UID in the server log when you connect:

```
### Updating player: PlayerId=1, Name=YourName, IdentityId=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

The `IdentityId` value is what goes in the config. Replace the placeholder, set tier 3,
and restart.

Confirm it worked — the log should show:

```
[Overwatch] Loaded 1 admin entries from config.
[Overwatch] Local tier received: 3 (Owner).
```

---

## Admin config — `$profile:Overwatch_Admins.json`

```json
{
    "version": 1,
    "admins": {
        "bbe7b313-580b-4350-9709-18583139e0f7": { "tier": 3, "name": "Michael" },
        "a1c2e4f6-1234-5678-9abc-def012345678": { "tier": 1, "name": "SomeModerator" }
    }
}
```

| Tier | Role | Gets |
|---|---|---|
| 1 | Moderator | Informational and non-destructive commands |
| 2 | Admin | Everything above, plus destructive and moderation commands |
| 3 | Owner | Everything |

`name` is a human-readable label only. All checks are against the UID.

The file is read on server start. Editing it while the server runs has no effect until
restart.

---

## Ban list — `$profile:Overwatch_Bans.json`

Created when the first ban is issued. Expired bans are pruned on start.

```json
{
    "version": 1,
    "bans": {
        "a1c2e4f6-1234-5678-9abc-def012345678": {
            "name": "Griefer123",
            "reason": "Team killing",
            "bannedBy": "Michael",
            "bannedByUid": "bbe7b313-580b-4350-9709-18583139e0f7",
            "bannedAt": 1753660800,
            "expiresAt": 1754265600
        }
    }
}
```

`expiresAt` of `0` means permanent. Timestamps are Unix seconds (UTC).

Editing this file by hand works, but the server only reads it on start.

---

## Config overrides

Two vanilla configs need overriding. Use **right-click → Override in [your addon]** in
the Resource Browser — this preserves the GUID, which is how the engine identifies a
replacement. Duplicating instead creates an unrelated file the engine will ignore.

Overridden files show a puzzle icon in the Resource Browser.

### `Configs/System/chimeraMenus.conf` — the admin menu

Add one Menu Preset entry:

| Field | Value |
|---|---|
| Name | `OW_AdminMenu` |
| Layout | `UI/layouts/OW_AdminMenu.layout` (drag it in, do not type it) |
| Action Context | `MenuContext` |
| Class | `OW_AdminMenu` |

The name must match the `modded enum ChimeraMenuPreset` value in `OW_MenuPresets.c`.
If it does not, the log shows `Menu preset 'OW_AdminMenu' not found!` at startup.

### `Configs/System/Actions/PlayerListActions.conf` — player list actions

Add an entry per action. Classes:

- `OW_HealPlayerListAction`
- `OW_KillPlayerListAction`
- `OW_GotoPlayerListAction`
- `OW_BringPlayerListAction`
- `OW_KickPlayerListAction`
- `OW_BanPlayerListAction`

Set **Action Name** on each — this is what admins see, e.g. `Overwatch: Heal`.

`OW_KickPlayerListAction` has a configurable **reason**.

`OW_BanPlayerListAction` has a configurable **duration** and **reason**. A context menu
cannot prompt for a duration, so register one entry per duration you want:

| Duration | Suggested name |
|---|---|
| `1h` | Overwatch: Ban 1h |
| `24h` | Overwatch: Ban 24h |
| `7d` | Overwatch: Ban 7d |
| `perm` | Overwatch: Ban permanently |

Make sure the name and the duration agree — nothing enforces it, and an entry labelled
"Ban 1h" that actually bans for a week is a nasty surprise.

Consider whether "Ban permanently" belongs in a context menu at all. There is no
confirmation step, and it is one misclick from the entry above it.

---

## Failure modes

**A malformed `Overwatch_Admins.json`** grants nobody anything, and logs an error. This
is deliberate — a corrupt config must never silently grant access.

**A malformed `Overwatch_Bans.json`** bans nobody, and logs an error. Also deliberate,
in the opposite direction — a corrupt file must never lock your players out.

Both directions mean the same thing: a broken config never wrongly punishes anyone.

---

## Troubleshooting

**Commands do nothing.** Check the log for `[Overwatch] Command router ready`. If it is
absent, the components are not on the GameMode prefab.

**"You don't have permission for that."** Your UID is not in the admin config, or is at
too low a tier. Check the `Local tier received` line in the log.

**Player list actions do not appear.** Check `Local tier received` shows your real tier.
Note that actions which cannot target yourself are hidden on your own row — with one
player connected, only Heal shows. That is expected.

**Menu does not open.** Look for `Menu preset 'OW_AdminMenu' not found!` at startup —
that means the `chimeraMenus.conf` override is not being read, or the entry name does
not match the enum.

**Vanilla admins are not Overwatch admins.** Overwatch's permission system is entirely
separate from Reforger's. A vanilla server admin not listed in `Overwatch_Admins.json`
is tier 0 to Overwatch and can be actioned by Overwatch admins.

---

## Compatibility

Pure server-side components plus two config overrides. Safe to add or remove without
affecting saves.

The `PlayerListActions.conf` override may conflict with another mod that overrides the
same file. If both are loaded, one will win — merge the entries by hand if needed.

