<p align="center">
  <img src="Overwatch_Banner.png" width="500" alt="Overwatch Banner">
</p>

## Short description  (179 / 200)

In-game admin tools for Reforger servers. Tiered UID permissions, hidden chat commands, an admin menu with a live player list, right-click player actions, and every action logged.

---

## Long description  (1995 / 2000)

**OVERWATCH - Server Admin Toolkit**

In-game admin tools for Arma Reforger dedicated servers. Add it to your mod list and it works - no dependencies, no gameplay changes.

Permissions are enforced server-side against identity UIDs, so a modified client cannot grant itself anything.

FEATURES

- Three tiers - Moderator, Admin, Owner. An admin can only act on someone below them, so your Admins cannot kick each other.
- Admin menu - numpad minus or /ow menu opens a live player list. Select a player to see their UID, tier and status, then heal, teleport, kick or ban with a click. Owners get tier management too.
- Hidden commands - /ow ... never appears in chat, so admins are not announcing every action.
- Player list integration - right-click any player for the same actions. Actions above your rank are hidden.
- Moderation - timed or permanent bans, enforced on reconnect, and unbans that take effect immediately.
- Optional Game Master access, granted automatically to a tier you choose, or off entirely.
- Full audit log - acting admin's UID, the exact command, and the reason behind every refusal.
- Fail-safe - a malformed permission file grants nobody anything; a bad ban file bans nobody.

SETUP

1. Add the mod to your server's mod list.
2. Start the server once - Overwatch_Admins.json is generated in your server profile.
3. Add your IdentityId from the server log with tier 3, then restart. Add admins after that with /ow grant.

Overwatch installs by overriding GameMode_Base.et, so any game mode inheriting from it picks it up with no setup. If another mod overrides that same prefab, only one will apply.

KNOWN LIMITATIONS

The menu's Kill, Kick and Ban fire on a single click with no confirmation yet, and Ban is permanent. The keybind is inactive on the deploy screen; use /ow menu there. Game Master actions are not recorded in the Overwatch audit log.

Full command list, configuration and troubleshooting:
https://github.com/Michael4170/Overwatch-Server-Admin-Toolkit

Suggestions and bug reports welcome via https://discord.gg/SsM7r8b7ae or the GitHub
page.
