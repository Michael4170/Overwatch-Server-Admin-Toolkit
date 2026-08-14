<p align="center">
  <img src="Overwatch_Banner.png" width="500" alt="Overwatch Banner">
</p>

## Short description

Server-side admin toolkit. Tiered UID permissions, hidden chat commands, right-click
actions in the player list, moderation, teleports and broadcasts — every action logged.

---

## Long description

**OVERWATCH — Server Admin Toolkit**

A lightweight admin toolkit for Arma Reforger dedicated servers. No mod dependencies and
no gameplay changes — add it to your server, attach three components to your game mode,
and it works for every player who joins.

Admins work from chat or from the player list they already use. Permissions live
server-side, keyed to identity UIDs, and every action is checked and logged on the server.

**Features**

- **Tiered permissions** — UID-based whitelist with Moderator, Admin and Owner tiers,
  stored as simple JSON in your server profile. Admins can only act on someone at a
  strictly lower tier, so your Admins can't kick each other and one rogue Owner can't
  strip the rest.
- **Hidden commands** — type `/ow …` and it never appears in chat, so your admins aren't
  announcing every action to the server. `!ow …` works identically and is suppressed
  before it leaves the client.
- **Player list integration** — right-click any player in the normal player list to heal,
  kill, teleport, kick or ban them. No separate UI to learn, and actions above your rank
  simply aren't shown.
- **Admin menu** — `/ow menu` opens a panel for the things that aren't about one specific
  player: the admin roster, the active ban list, and who's connected.
- **Moderation** — kick and ban with reasons, timed bans with automatic expiry, enforced
  on reconnect. Unbans take effect immediately instead of leaving the player locked out
  until the original sentence runs down.
- **Player management** — full heal, admin kill with no friendly-fire penalty, teleport to
  a player, bring a player to you, and detailed player info including UID and tier.
- **In-game admin management** — promote and demote with `/ow grant` and `/ow revoke`, no
  file editing and no restart. The change reaches the target immediately, without them
  reconnecting.
- **Communication** — server-wide announcements as readable on-screen panels rather than
  chat text lost against the terrain.
- **Full audit log** — every action written to the server log with the acting admin's UID
  and the exact command. Refusals are logged too, with the specific reason they failed.
- **Fail-safe by design** — a malformed permission file grants nobody anything; a
  malformed ban file bans nobody. Neither can lock you out of your own server.

**Commands**

```
/ow help                       what your tier can use
/ow menu                       open the admin menu
/ow players                    connected players and ids
/ow playerinfo <player>        id, UID, tier, faction, alive state
/ow admins                     the admin roster
/ow bans                       active bans
/ow heal <player|me>           full heal
/ow broadcast <message>        announce to everyone
/ow kill <player>              admin kill, no friendly-fire penalty
/ow goto <player>              teleport to a player
/ow bring <player>             teleport a player to you
/ow kick <player> <reason>     disconnect, no reconnect block
/ow ban <player> <30m|7d|perm> <reason>
/ow unban <uid|name>
/ow grant <player> <1|2|3>     set someone's tier
/ow revoke <player>            remove someone's tier
```

Targets accept `me`, a player id, or a full or partial name.

**Setup**

1. Add the mod to your server's mod list.
2. Add the three Overwatch components to your GameMode prefab.
3. Start the server once — `Overwatch_Admins.json` is generated in your server profile.
4. Join, find your `IdentityId` in the server log, add it with tier 3 (Owner), restart.

> If your `IdentityId` reads `00000000-0000-0000-0000-000000000000`, that account isn't
> signed in to the backend. Don't use that value — it belongs to every signed-out session
> rather than to you, and Overwatch rejects it. Sign in and reconnect for a real UID.

Add every admin after the first with `/ow grant` — no more file editing.

Full documentation, command list and configuration guide:
https://github.com/Michael4170/Overwatch-Server-Admin-Toolkit

**Compatibility**

Built for Arma Reforger 1.8. Server-side components plus two config overrides — works
alongside any game mode or mod set, and safe to add or remove without affecting saves.

**Known limitations**

- Banned players see Reforger's generic "kicked" message rather than the ban reason. The
  ban itself works correctly; only the notification is generic.
- The admin menu opens with `/ow menu` — there is no keybind yet.
- `/ow bans` lists 8 entries in chat; read `Overwatch_Bans.json` for the full list.

**Roadmap**

Discord webhook logging, spectate mode, scheduled restart warnings, MOTD on join, and a
timestamped action log file alongside the console output.


Suggestions and bug reports welcome via https://discord.gg/SsM7r8b7ae or the GitHub
page.
