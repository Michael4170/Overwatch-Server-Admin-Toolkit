<p align="center">
  <img src="Overwatch_Banner.png" width="500" alt="Overwatch Banner">
</p>

---

## Short description

Server-side admin toolkit. Tiered UID permissions, hidden chat commands, right-click
actions in the player list, moderation, teleports and broadcasts — every action logged.

---

## Long description

**OVERWATCH — Server Admin Toolkit**

A lightweight, server-side admin toolkit for Arma Reforger dedicated servers. No
dependencies, no gameplay changes — install it on your server and it works for every
player who joins.

**Features**

- **Tiered permissions** — UID-based admin whitelist with Moderator, Admin and Owner
  tiers. Stored in a simple JSON file in your server profile.
- **Hidden commands** — type `/ow …` and the command never appears in chat, so your
  admins aren't announcing every action to the server. `!ow …` also works if you don't
  mind it being visible.
- **Player list integration** — right-click any player in the normal player list and
  heal, kill, teleport, kick or ban them directly. No separate UI to learn, and actions
  you don't have the rank for simply aren't shown.
- **Moderation** — kick and ban with reasons, timed bans with automatic expiry, enforced
  on reconnect.
- **Player management** — heal, kill, teleport to a player, bring a player to you,
  detailed player info.
- **Communication** — server-wide announcements as readable on-screen panels rather than
  chat text lost against the terrain.
- **Full audit log** — every admin action written to the server log with the acting
  admin's UID and the exact command. Failed permission attempts are logged too.
- **Fail-safe by design** — a malformed permission file grants nobody anything; a
  malformed ban file bans nobody. Neither can lock you out of your own server.

**Setup**

1. Add the mod to your server's mod list.
2. Add the three Overwatch components to your GameMode prefab.
3. Start the server once — `Overwatch_Admins.json` is generated in your server profile.
4. Add your Identity UID with tier 3 (Owner), restart, done.

Full documentation, command list and configuration guide:
https://github.com/Michael4170/Overwatch_Server_Admin_Toolkit

**Compatibility**

Built for Arma Reforger 1.8. Server-side components plus two config overrides — works
alongside any game mode or mod set, and safe to add or remove without affecting saves.

**Known limitations**

- Banned players see Reforger's generic "kicked" message rather than the ban reason. The
  ban works correctly; only the notification is generic.
- The admin menu opens with `/ow menu` — there is no keybind yet.

**Roadmap**

Admin menu panels for bans and admin management, in-game admin promotion, Discord
webhook logging, spectate mode, scheduled restart warnings and MOTD on join.

Suggestions and bug reports welcome via https://discord.gg/SsM7r8b7ae or the GitHub
page.

---
