# DropSweeper

<p align="center">
  <img src="assets/封面.png" width="60%" alt="DropSweeper Cover">
</p>

> An intelligent item-drop sweeper for Minecraft servers.
> 
> With the friendly "Sweeper Maid" (扫地姬) by your side, say goodbye to piles of cobblestone, sand, and arrows. ฅ^•ﻌ•^ฅ
> 
> 📖 [中文文档](README.md) | English

[![Minecraft](https://img.shields.io/badge/Minecraft-26.1.2-green.svg)](https://www.minecraft.net/)
[![Fabric](https://img.shields.io/badge/Loader-Fabric-blueviolet.svg)](https://fabricmc.net/)
[![Java](https://img.shields.io/badge/JDK-25-orange.svg)](https://www.azul.com/downloads/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](#license)

---

## What is this?

`DropSweeper` is a Fabric server-side mod that automatically collects dropped items on the ground into **dustbins** (virtual large chests with 54 slots each), giving you:

- 🧹 **Auto sweep** — runs on a configurable interval (default: every 10 minutes)
- 📦 **Multiple dustbins** — defaults to 1, configurable up to N; FIFO eviction across bins
- 🚫 **White/Black lists** — whitelist items are never touched; blacklist items are removed but not stored
- 🎯 **Extra entity cleanup** — arrows, experience orbs, etc.
- ⚠️ **Overload warning** — alerts OP when a chunk has too many items piling up
- 💬 **Friendly announcements** — countdown before sweep (ActionBar/Chat), clickable dustbin links on completion
- 🔧 **Hot-reload config** — no server restart needed
- 🔐 **Granular permissions** — control who can open dustbins vs. who can trigger sweeps

---

## Installation

1. Install [Fabric Loader 0.19.3+](https://fabricmc.net/) and [Fabric API 0.151.0+](https://modrinth.com/mod/fabric-api)
2. Drop `dropsweeper-1.x.x.jar` into your server's `mods/` directory
3. Start the server once — it will auto-generate `config/dropsweeper.json`
4. Restart the server. Done ✨

> The client is **loaded but has no actual functionality** (`environment: "*"`, client entrypoint is a stub).
> Players on multiplayer servers don't need to install it; it works in single-player out of the box.

---

## Commands

All commands are prefixed with `/dropsweeper`.

### Sweep

| Command | Permission | Description |
|---|---|---|
| `/dropsweeper items` | OP | Sweep current dimension |
| `/dropsweeper items all` | OP | Sweep all dimensions |
| `/dropsweeper items <world_id>` | OP | Sweep a specific dimension (Tab-completable) |

### Open dustbin

| Command | Permission | Description |
|---|---|---|
| `/dropsweeper dustbin` | Everyone | Open dustbin 0 |
| `/dropsweeper dustbin <index>` | Everyone | Open dustbin by index |

> After a sweep completes, the result message lists all non-empty dustbins as **clickable links**.

### White/Black list management

```
/dropsweeper whitelist add <item_id>     # Add to whitelist (never swept)
/dropsweeper whitelist remove <item_id>  # Remove from whitelist
/dropsweeper whitelist list              # Show whitelist

/dropsweeper blacklist add <item_id>     # Add to blacklist (swept but not stored)
/dropsweeper blacklist remove <item_id>  # Remove from blacklist
/dropsweeper blacklist list              # Show blacklist
```

> Wildcard `modid:*` is supported, e.g. `minecraft:*` matches all vanilla items.

### Config

```
/dropsweeper config reload    # Hot-reload config/dropsweeper.json
```

---

## Configuration

Location: `config/dropsweeper.json` (auto-generated on first run).

### Common fields

```json
{
  "sweepIntervalSeconds": 600,           // Auto-sweep interval in seconds (0 = disabled)
  "dustbinCount": 1,                     // Number of dustbins (54 slots each)
  "itemWhitelist": [                     // Whitelist: never swept
    "minecraft:nether_star",
    "minecraft:heavy_core"
  ],
  "itemBlacklist": [                     // Blacklist: swept but not stored
    "minecraft:cobblestone",
    "minecraft:sand",
    "minecraft:gravel"
  ],
  "extraEntityTypes": [                  // Extra entities to clean up
    "minecraft:arrow",
    "minecraft:spectral_arrow",
    "minecraft:experience_orb"
  ],
  "itemOverloadThreshold": 640,          // Per-chunk item count to trigger warning (0 = disabled)
  "permissionLevelDustbin": 0,           // Min. permission to open dustbin (0 = any player)
  "permissionLevelClean": 2,             // Min. permission to manually sweep
  "announcementChannel": {               // Announcement channels
    "long": "actionbar",                 // Long countdown (default 120/60/30s)
    "short": "chat",                     // Short countdown (≤3s)
    "complete": "chat"                   // Sweep completion
  }
}
```

### Permission levels

| Level | MC 26.1.2 equivalent |
|---|---|
| 0 | Anyone (no permission required) |
| 1 | Game master |
| 2 | Moderator |
| 3 | Admin |
| 4 | Owner |

---

## Announcement channels

- **ActionBar** — small text above the hotbar, auto-fades after 60 ticks (3 seconds), **doesn't spam**
- **Chat** — bold message in chat, **doesn't auto-fade**

Default: long countdown uses ActionBar (unobtrusive), short countdown and completion use Chat (visible).

---

## How much can dustbins hold?

- Each dustbin has 54 slots (6 rows × 9 columns, equivalent to a large chest)
- Default 1 → 54 slots
- Set `dustbinCount: 8` → 8 × 54 = 432 slots
- When full, the **oldest** item is replaced by the new one (FIFO)

---

## Data storage

- Dustbin contents are stored in the world save: `world/data/dimensions/minecraft/overworld/data/dropsweeper/dustbin.dat`
- Auto-saved, no manual action needed
- **Decreasing** the number of dustbins will truncate the extras and may discard items in non-empty dustbins
- **Increasing** the number preserves existing data

---

## FAQ

**Q: Where is the dustbin? How do I open it?**  
A: Run `/dropsweeper dustbin` (any player by default), or click the links in the sweep-completion message.

**Q: How do I add custom items to the white/black list?**  
A: Use `/dropsweeper whitelist add <item_id>` (e.g. `minecraft:diamond`). You can also edit `config/dropsweeper.json` directly, then run `/dropsweeper config reload`.

**Q: Do I have to restart the server after changing the config?**  
A: No. Run `/dropsweeper config reload` to apply changes immediately.

**Q: Do clients need to install the mod?**  
A: On a multiplayer server, **no** — all sweep/command/storage logic runs server-side. Clients that install it just get a stub entrypoint.  
For single-player or LAN play, you can install it on the client as well without affecting gameplay.

**Q: Will it conflict with other sweep mods?**  
A: Yes. If two mods try to clean the same drops, you may get duplicate removals. Use one or the other.

---

## License

MIT License — free to use, modify, and redistribute, including for commercial purposes, provided the copyright notice is preserved.  
See [LICENSE](LICENSE) for details.
