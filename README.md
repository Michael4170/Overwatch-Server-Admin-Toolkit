<p align="center">
  <img src="Overwatch_Banner.png" width="500" alt="Overwatch Banner">
</p>

## Short description  (179 / 200)

In-game admin tools for Reforger servers. Tiered UID permissions, hidden chat commands, an admin menu with a live player list, right-click player actions, and every action logged.

---

## Long description  (1958 / 2000)

**OVERWATCH - Server Admin Toolkit**

In-game admin tools for Arma Reforger dedicated servers. Add it to your mod list and it works - no dependencies, no gameplay changes.

Permissions are enforced server-side against identity UIDs, so a modified client cannot grant itself anything.

FEATURES

- Three tiers - Moderator, Admin, Owner. An admin can only act on someone below them, so your Admins cannot kick each other.
- Admin menu - /ow menu opens a live player list. Select a player to see their UID, tier and status, then heal, teleport, kick or ban with a click. Roster and ban panels alongside.
- Hidden commands - /ow ... never appears in chat, so admins are not announcing every action.
- Player list integration - right-click any player for the same actions. Actions above your rank are hidden.
- Moderation - timed or permanent bans, enforced on reconnect, and unbans that take effect immediately.
- Full audit log - acting admin's UID, the exact command, and the reason behind every refusal.
- Fail-safe - a malformed permission file grants nobody anything, a malformed ban file bans nobody. Neither can lock you out of your own server.

SETUP

1. Add the mod to your server's mod list.
2. Start the server once - Overwatch_Admins.json is generated in your server profile.
3. Add your IdentityId from the server log with tier 3, then restart. Add every admin after that in-game with /ow grant.

Overwatch installs by overriding GameMode_Base.et, so any game mode inheriting from it picks the toolkit up with no setup. If another mod overrides that same prefab, only one will apply.

KNOWN LIMITATIONS

The menu's Kill, Kick and Ban fire on a single click - no confirmation step yet, and the menu's Ban is permanent. No keybind yet; open with /ow menu. Banned players see Reforger's generic kick message rather than the ban reason.

Full command list, configuration and troubleshooting:
https://github.com/Michael4170/Overwatch-Server-Admin-Toolkit

Suggestions and bug reports welcome via https://discord.gg/SsM7r8b7ae or the GitHub
page.
