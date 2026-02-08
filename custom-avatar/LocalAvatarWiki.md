# Local Player Avatar 使用指南
### Source SDK 2013 (VGUI2)
### 本地头像方案（非 Steam API / 纯本地资源加载）

---

## 1. 简介

本文档说明如何在 **Source SDK 2013 / VGUI2** 项目中使用：

> ✅ 基于玩家名称自动匹配的本地头像系统

该方案：

- ❌ 不依赖 Steam API
- ❌ 无需修改引擎或源码
- ✅ 纯资源驱动
- ✅ 自动回退默认头像
- 仅仅只是本地（请输入文本）

适用于：

- 单机游戏
- 局域网游戏
- Mod 项目
- Android / Linux 端口
- Bot / AI 角色头像

---

## 2. 工作机制（核心逻辑）

运行时流程：

```
读取玩家名
   ↓
按同名搜索本地材质
   ↓
找到 → 加载头像
未找到 → 使用默认头像
```

等价伪代码：

```cpp
if (Exists("materials/vgui/avatar/<playername>.vmt"))
    LoadAvatar(playername);
else
    LoadDefaultAvatar();
```

⚠️ 完全基于文件名匹配。

---

## 3. 目录结构（必须严格一致）

所有头像资源必须放置：

```
materials/
└── vgui/
    └── avatar/
        ├── <playername>.vmt
        └── <playername>.vtf
```

示例：

```
materials/vgui/avatar/zzh.vmt
materials/vgui/avatar/zzh.vtf
```

---

## 4. 命名规则（最容易出错）

### 必须满足

- 文件名 = 玩家名
- 完全一致
- 无空格
- 无额外后缀

### 示例

| 玩家名 | 正确文件 |
|-------|----------|
| zzh | zzh.vmt / zzh.vtf |
| ZZH | ZZH.vmt / ZZH.vtf |
| player_01 | player_01.vmt / player_01.vtf |

---

### ❌ 不支持情况

| 错误类型 | 示例 |
|---------|-------|
| 部分匹配 | zzh_1 |
| 大小写错误 | Zzh |
| 多余后缀 | zzh_avatar.vtf |
| 错扩展名 | .png / .jpg |

---

## 5. 材质（Material）要求

头像必须使用 **VGUI 兼容材质**。

### 推荐类型

```
UnlitGeneric
```

### 推荐 vmt 模板

```vmt
"UnlitGeneric"
{
    "$basetexture" "vgui/avatar/zzh"
    "$translucent" "1"
    "$vertexcolor" "1"
    "$vertexalpha" "1"
}
```

### 说明

| 参数 | 作用 |
|-------|-------|
| $translucent | 支持透明 |
| $vertexalpha | UI Alpha |
| $vertexcolor | UI 着色 |

⚠️ 不需要：
- mipmaps
- lightmap
- bumpmap

---

## 6. 纹理（VTF）要求

### 格式

```
VTF
```

### 推荐尺寸

- 32 × 32
- 64 × 64
- 128 × 128

必须：

- 2 的幂尺寸（Power-of-two）
- 支持 Alpha
- 建议 RGBA8888

---

## 7. 热重载（调试必备）

修改资源后无需重启游戏：

```
mat_reloadallmaterials
```

---

## 8. 验证是否加载成功

查看是否进入材质缓存：

```
mat_texture_list | findstr vgui/avatar
```

期望输出：

```
vgui/avatar/zzh
```

---

## 9. 自动回退机制

当出现：

- 文件不存在
- vmt 无效
- vtf 加载失败

系统会：

> 自动显示默认头像

无需任何额外代码处理。

---

## 10. 平台差异（非常关键）

### Linux / Android

- 文件系统大小写敏感
- 必须完全匹配

```
zzh.vtf ≠ Zzh.vtf
```

### Windows

- 默认大小写不敏感
- 可能“看起来能用”
- ❌ 但不可移植

建议始终严格区分大小写。

---

## 11. 推荐使用场景

适合：

- 离线模式
- 非 Steam 项目
- Bot / AI
- 职业头像
- 角色身份区分
- Android 移植版本

不适合：

- 在线 Steam 头像同步需求
- 云端动态头像

---

## 12. 快速使用步骤（TL;DR）

```
① 制作 vtf
② 写 vmt（UnlitGeneric）
③ 文件名改为玩家名
④ 放入 materials/vgui/avatar/
⑤ 游戏内执行 mat_reloadallmaterials
```

完成。

---

## 13. 常见坑位总结（经验向）

最常见失败原因：

- ❌ 大小写不一致（Linux/Android 100%踩坑）
- ❌ vmt 路径写错
- ❌ 忘记打包进 VPK
- ❌ 非 2^n 尺寸
- ❌ 使用 png/jpg 而非 vtf

建议：

> 先用 64x64 RGBA8888 测试成功，再优化尺寸。

---

## 14. 结论

该头像系统：

- 资源驱动
- 零代码侵入
- 高性能
- 易维护
- 跨平台

对于 Source SDK Mod / Android 移植是最稳妥方案，直接使用即可。
