# Local Player Avatar Usage Guide  
# 本地玩家头像使用指南  
**Source SDK 2013 (VGUI 2)**
# Not Steam Api
# by Source App
---

## Overview | 概述

**English**

This document explains how to use **local player avatars based on player names** in a Source SDK 2013–based game or mod.

When configured correctly, the avatar system will:

- Display a custom local avatar when a matching avatar exists  
- Automatically fall back to the default avatar when no matching avatar is found  

**No source code modification is required** to use this feature.

---

**中文**

本文档说明了在基于 **Source SDK 2013** 的游戏或 Mod 中，如何使用**基于玩家名称的本地头像系统**。

正确配置后，头像系统将具备以下行为：

- 当存在匹配的本地头像时，显示自定义头像  
- 当不存在匹配头像时，自动回退到默认头像  

**使用该功能无需修改任何源代码。**

---

## How It Works | 工作原理

**English**

At runtime, the avatar system follows this logic:

1. Read the player's display name  
2. Search for a local avatar using the same name  
3. Load and display the avatar if found  
4. Otherwise, display the default avatar  

---

**中文**

在运行时，头像系统按照以下流程工作：

1. 读取玩家显示名称  
2. 使用该名称查找对应的本地头像  
3. 若找到头像，则加载并显示  
4. 若未找到，则显示默认头像  

---

## Directory Structure | 目录结构

**English**

Local avatar files must be placed in the following directory:
materials/ └── vgui/ └── avatar/ ├── .vmt └── .vtf
---

---

**中文**

本地头像文件必须放置在以下目录结构中：
materials/ └── vgui/ └── avatar/ ├── <玩家名>.vmt └── <玩家名>.vtf
---

---

## Example | 示例

**English**

For a player named `zzh`:
materials/vgui/avatar/zzh.vmt materials/vgui/avatar/zzh.vtf
---

---

**中文**

当玩家名为 `zzh` 时：
materials/vgui/avatar/zzh.vmt materials/vgui/avatar/zzh.vtf
---

---

## Naming Rules | 命名规则

### Player Name Matching | 玩家名匹配规则

**English**

- Avatar file names **must exactly match** the player name  
- Matching is **case-sensitive** on Linux and Android  

| Player Name | Valid Avatar Files |
|------------|--------------------|
| zzh | zzh.vmt, zzh.vtf |
| ZZH | ZZH.vmt, ZZH.vtf |
| player_01 | player_01.vmt, player_01.vtf |

---

**中文**

- 头像文件名**必须与玩家名完全一致**  
- 在 Linux 和 Android 平台上，**大小写敏感**  

| 玩家名 | 有效头像文件 |
|------|-------------|
| zzh | zzh.vmt, zzh.vtf |
| ZZH | ZZH.vmt, ZZH.vtf |
| player_01 | player_01.vmt, player_01.vtf |

---

### Unsupported Cases | 不支持的情况

**English**

- Partial matches (e.g. `zzh_1` vs `zzh`)  
- Incorrect letter casing on case-sensitive platforms  
- Additional or altered file extensions  

---

**中文**

以下情况将无法匹配头像：

- 部分匹配（如 `zzh_1` 与 `zzh`）  
- 在大小写敏感平台上的大小写不一致  
- 文件名包含额外或错误的扩展名  

---

## Material Requirements | 材质要求

**English**

Local avatars must use **VGUI-compatible materials**.

### Recommended Material Type

- `UnlitGeneric`

### Recommended Material Settings

- Base texture path:
vgui/avatar/
---

- Alpha channel supported  
- No mipmaps required  

---

**中文**

本地头像必须使用**兼容 VGUI 的材质**。

### 推荐材质类型

- `UnlitGeneric`

### 推荐材质设置

- 基础纹理路径：
vgui/avatar/<玩家名>
---

- 支持 Alpha 通道  
- 不需要 mipmaps  

---

## Texture Requirements | 纹理要求

**English**

- Format: **VTF**
- Recommended sizes:
- 32 × 32
- 64 × 64
- 128 × 128
- Power-of-two dimensions recommended  
- Transparent backgrounds supported  

---

**中文**

- 格式：**VTF**
- 推荐尺寸：
- 32 × 32
- 64 × 64
- 128 × 128
- 推荐使用 2 的幂尺寸  
- 支持透明背景  

---

## Reloading Avatars | 重新加载头像

**English**

After adding or modifying avatar files, reload materials in-game:
mat_reloadallmaterials
---

---

**中文**

在添加或修改头像文件后，可在游戏内执行以下命令重新加载材质：
mat_reloadallmaterials
---

---

## Verifying Loaded Avatars | 验证头像是否加载

**English**

To verify loaded avatars:
mat_texture_list | findstr vgui/avatar
---

Expected output example:
vgui/avatar/zzh
---

---

**中文**

使用以下命令确认头像是否已加载：
mat_texture_list | findstr vgui/avatar
---

预期输出示例：
vgui/avatar/zzh
---

---

## Fallback Behavior | 回退行为

**English**

If any of the following occur:

- No matching avatar file exists  
- The material file is invalid  
- The texture fails to load  

The system will **automatically display the default avatar**.

No additional configuration is required.

---

**中文**

在以下任意情况下：

- 不存在匹配的头像文件  
- 材质文件无效  
- 纹理加载失败  

系统将**自动显示默认头像**，无需额外配置。

---

## Platform Notes | 平台说明

### Linux / Android

**English**

- File systems are case-sensitive  
- File names must match player names exactly  
- `zzh.vtf` and `Zzh.vtf` are different files  

**中文**

- 文件系统大小写敏感  
- 文件名必须与玩家名完全一致  
- `zzh.vtf` 与 `Zzh.vtf` 被视为不同文件  

---

### Windows

**English**

- File system is usually case-insensitive  
- Case mismatches may appear to work but are **not portable**  

**中文**

- 文件系统通常大小写不敏感  
- 大小写错误在本地可能可用，但**不具备可移植性**  

---

## Recommended Use Cases | 推荐使用场景

**English**

- Offline or LAN games  
- Mods without Steam dependency  
- Bot or AI player avatars  
- Role-based or class-based avatar systems  

---

**中文**

- 单机或局域网游戏  
- 无 Steam 依赖的 Mod  
- Bot 或 AI 玩家头像  
- 职业 / 角色区分头像系统  

---

## Summary | 总结

**English**

To use local player avatars:

1. Name avatar files after the player name  
2. Place them under `materials/vgui/avatar/`  
3. Use valid VGUI-compatible materials  
4. Reload materials if needed  

The avatar system handles loading and fallback automatically.

---

**中文**

使用本地玩家头像的步骤如下：

1. 使用玩家名命名头像文件  
2. 放置于 `materials/vgui/avatar/` 目录  
3. 使用合法的 VGUI 兼容材质  
4. 必要时重新加载材质  

