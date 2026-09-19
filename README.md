<p align="center">
  <img src="Overwatch_Banner.png" width="500" alt="Overwatch Banner">
</p>

**OVERWATCH - Server Admin Toolkit**

Tiered admin tooling for Arma Reforger dedicated servers. Permissions are keyed to Bohemia identity UIDs and checked entirely server-side, so a client cannot exceed the tier they hold.

FEATURES

- Three tiers — Moderator, Admin, Owner — 20 chat commands, filtered to what you can use.
- In-game admin menu on a keybind — live player list, click-to-action. Same permission checks as chat, no second path.
- Spectate — attach your camera to any player on the map and follow them.
- Game Master — automatic for qualifying tiers, plus session-only grants with !ow gm.
- Persistent bans — duration-based or permanent, survive restarts, revocable in game.
- Restart countdown — an on-screen timer before your scheduled restarts. It warns only; your host still does the restarting.
- Message of the day — a rules panel on the welcome screen, your text and your banner.
- Full server-side logging — every action records the actor, the target and both UIDs.

COMMANDS

Moderator: help, admins, heal, players, playerinfo, broadcast, bans, menu
Admin: kill, goto, bring, spectate, unspectate, kick, ban, unban, gm, ungm
Owner: grant, revoke

Type !ow help in game for your tier's list. !ow and /ow both work; !ow is hidden from other players.

SETUP

Add the mod. Nothing to wire up — Overwatch extends the base game mode prefab, so components attach on load. First start writes template configs and logs their paths. Add your UID as tier 3, restart, done.

Overwatch fails closed: a bad config grants nobody anything and says so. Countdown and MOTD ship off.

TWO THINGS TO KNOW

Game Master is more power than every command here combined, and nothing done with it reaches Overwatch's log. gmTier sets who gets it — default 2 (Admin), 3 for Owners only, 0 to disable.

Spectate does not notify the target — covert by design, because an admin checking for cheating cannot announce it first. The log is the only record; set your disclosure policy before enabling it.

Full docs, configuration and troubleshooting:
https://github.com/Michael4170/Overwatch-Server-Admin-Toolkit

Suggestions and bug reports: https://discord.gg/SsM7r8b7ae or the GitHub page.
