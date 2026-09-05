<p align="center">
  <img src="Overwatch_Banner.png" width="500" alt="Overwatch Banner">
</p>

## Short description

Tiered admin tooling for Arma Reforger dedicated servers. Permissions are keyed to Bohemia identity UIDs and checked entirely server-side, so a modified client cannot exceed the tier its UID holds.

---

## Long description

**OVERWATCH - Server Admin Toolkit**

FEATURES

- Three tiers — Moderator, Admin, Owner — with 20 chat commands filtered to what you can actually use.
- In-game admin menu on a keybind, with a live player list and click-to-action buttons. Same permission checks as chat, no second code path.
- Spectate — attach your camera to any player anywhere on the map and follow them.
- Game Master integration — automatic for qualifying tiers, plus session-only grants with !ow gm.
- Persistent bans — duration-based or permanent, survive restarts, revocable in game.
- Full server-side logging — every action records the actor, the target and both UIDs.

COMMANDS

Moderator: help, admins, heal, players, playerinfo, broadcast, bans, menu
Admin: kill, goto, bring, spectate, unspectate, kick, ban, unban, gm, ungm
Owner: grant, revoke

Type !ow help in game for the list filtered to your tier. Both !ow and /ow work; !ow messages are never shown to other players.

SETUP

Add the mod to your server. Nothing to wire up — Overwatch extends the base game mode prefab, so its components attach on load. Start the server once, and it writes a template config and logs the exact path. Add your UID as tier 3, restart, done.

Overwatch fails closed: a missing or malformed config grants nobody anything, and says so in the log.

TWO THINGS TO KNOW

Game Master is more power than every command here combined, and nothing done with it passes through Overwatch's log. gmTier controls who gets it automatically — default 2 (Admin), 3 for Owners only, 0 to disable.

Spectate does not notify the target. It is covert by design, because an admin checking for cheating cannot announce it first. The server log is the only record. Decide your disclosure policy before enabling it.

Full command list, configuration and troubleshooting:
https://github.com/Michael4170/Overwatch-Server-Admin-Toolkit

Suggestions and bug reports welcome via https://discord.gg/SsM7r8b7ae or the GitHub
page.
