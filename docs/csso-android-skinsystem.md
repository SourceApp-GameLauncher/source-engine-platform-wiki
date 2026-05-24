皮肤系统使用指南

本文档基于项目中的皮肤数据库 (cs_skin_database)、库存系统 (cs_inventory) 以及玩家资源 (c_cs_playerresource) 编写，详细说明如何配置和使用自定义皮肤系统。

---

目录

1. 概述
2. 物品类型与 ID 列表
3. skins.txt 配置文件格式
4. VMT 参数详解
5. 示例 skins.txt
6. 常见问题

---

概述

皮肤系统支持以下物品类型：

类型 说明 存储位置
武器皮肤 替换武器外观的涂装 scripts/skins.txt
手套皮肤 替换角色手套外观 scripts/skins.txt
刀具皮肤 替换近战武器外观 scripts/skins.txt
探员皮肤 CT / T 阵营角色模型 硬编码于 cs_inventory.cpp
音乐包 MVP 音乐替换 硬编码于 cs_inventory.cpp

皮肤数据库 (CCSkinDatabase) 负责加载和解析 skins.txt，并动态创建应用了 VMT 参数的材质。

---

物品类型与 ID 列表

1. 武器 ID (CSWeaponID)

定义在 cs_weapon_parse.h 中，常用武器 ID 列表（从代码推断）：

武器名称 别名 / ConVar 后缀
AK-47 ak47
M4A4 m4a4
M4A1-S m4a1s
AWP awp
Desert Eagle deagle
R8 Revolver revolver
Glock-18 glock
USP-S usp
P2000 hkp2000
P250 p250
Five-SeveN fiveseven
Tec-9 tec9
CZ75-Auto cz75a
Dual Berettas dualberettas
MAG-7 mag7
Nova nova
Sawed-Off sawedoff
XM1014 xm1014
M249 m249
Negev negev
MAC-10 mac10
MP9 mp9
MP7 mp7
MP5-SD mp5sd
UMP-45 ump45
P90 p90
PP-Bizon bizon
Galil AR galilar
FAMAS famas
AUG aug
SG 553 sg556
SSG 08 ssg08
SCAR-20 scar20
G3SG1 g3sg1

2. 刀具 ID

武器 ID 别名 槽位索引
WEAPON_KNIFE_CSS knife_css 1
WEAPON_KNIFE_KARAMBIT knife_karambit 2
WEAPON_KNIFE_FLIP knife_flip 3
WEAPON_KNIFE_BAYONET knife_bayonet 4
WEAPON_KNIFE_M9_BAYONET knife_m9_bayonet 5
WEAPON_KNIFE_BUTTERFLY knife_butterfly 6
WEAPON_KNIFE_GUT knife_gut 7
WEAPON_KNIFE_TACTICAL knife_tactical 8
WEAPON_KNIFE_FALCHION knife_falchion 9
WEAPON_KNIFE_SURVIVAL_BOWIE knife_survival_bowie 10
WEAPON_KNIFE_CANIS knife_canis 11
WEAPON_KNIFE_CORD knife_cord 12
WEAPON_KNIFE_GYPSY knife_gypsy_jackknife 13
WEAPON_KNIFE_OUTDOOR knife_outdoor 14
WEAPON_KNIFE_SKELETON knife_skeleton 15
WEAPON_KNIFE_STILETTO knife_stiletto 16
WEAPON_KNIFE_URSUS knife_ursus 17
WEAPON_KNIFE_WIDOWMAKER knife_widowmaker 18
WEAPON_KNIFE_PUSH knife_push 19

3. 手套槽位 ID

手套类型及其对应的槽位索引（0-19），用于 ConVar loadout_slot_gloves_ct / loadout_slot_gloves_t：

索引 手套名称 类名片段
0 默认手套 default
1 血猎手套 bloodhound
2 血猎手套（完美世界） bloodhound perfectworld
3 血猎手套（狂牙大行动） bloodhound brokenfang
4 血猎手套（九头蛇） bloodhound hydra
5 无指手套 fingerless
6 全指手套 fullfinger
7 裹手（皮革） handwrap / leathery
8 硬指节手套 hardknuckle
9 硬指节手套（黑色） hardknuckle black
10 硬指节手套（蓝色） hardknuckle blue
11 摩托手套 motorcycle
12 光滑手套 slick
13 专业手套 specialist
14 运动手套 sporty
15 SAS（旧） sas old
16 FBI（旧） fbi old
17 凤凰战士（旧） phoenix old
18 精英部队（旧） leet old
19 裸手 bare

4. 探员 ID

探员列表硬编码于 cs_inventory.cpp 的 g_AgentIconsCT 和 g_AgentIconsT，槽位 ConVar 为：

· CT: loadout_slot_agent_ct
· T: loadout_slot_agent_t

5. 音乐包 ID

音乐包名称列表在 g_MusicKitNames 中，槽位 ConVar 为 loadout_music。

---

skins.txt 配置文件格式

皮肤配置文件应放置在 scripts/skins.txt，使用 Valve KeyValues 格式。

根结构

```plaintext
"Skins"
{
    "<paintkit_id>"
    {
        // 皮肤定义
    }
    ...
}
```

每个皮肤定义以 PaintKit ID（整数）作为子键名。

皮肤定义字段

字段 类型 必需 说明
name string 是 皮肤显示名称
description string 否 描述文本
weapon string 条件必需* 武器别名（如 ak47），与 glove_class 互斥
glove_class string 条件必需* 手套类名（如 bloodhound），与 weapon 互斥
item_type string 否 物品类型："weapon"、"gloves"、"knife"。默认根据 weapon/glove_class 推断
rarity int 是 稀有度数值：0=消费级, 1=工业级, 2=军规级, 3=受限, 4=保密, 5=隐秘, 6=违禁品
icon string 否 库存图标路径（如 materials/vgui/icons/skins/ak47_asiimov.vtf）
base string 条件必需** 基础材质路径（单材质模式）
material0..materialN sub-key 条件必需** 多材质定义（见下文），与 base 互斥

*：weapon 用于武器/刀具，glove_class 用于手套。
**：必须提供 base 或至少一个 materialX 子键。

多材质定义格式

每个 material0、material1 等是一个子键，内容为：

```plaintext
"material0"
{
    "base"  "models/weapons/customization/paints/custom/ak47_bla"
    // VMT 参数（见下节）
}
```

稀有度数值对照

数值 稀有度 颜色
0 消费级 (Common) 浅蓝灰
1 工业级 (Uncommon) 蓝色
2 军规级 (Rare) 深蓝
3 受限 (Mythical) 紫色
4 保密 (Legendary) 粉紫
5 隐秘 (Ancient) 红色
6 违禁品 (Contraband) 金色

---

VMT 参数详解

在 materialX 块内可以覆盖的 VMT 参数（前缀 $ 可选，建议使用 $）：

纹理参数

参数 说明
$basetexture 基础颜色纹理
$basetexture2 第二基础纹理
$bumpmap 法线贴图
$bumpmap2 第二法线贴图
$detail 细节纹理
$envmap 环境贴图
$phongexponenttexture 高光指数纹理
$phongwarptexture 高光扭曲纹理

光照与着色

参数 类型 说明
$phong int 启用 Phong 光照 (0/1)
$phongboost float Phong 强度
$phongexponent float Phong 指数
$phongalbedotint float 反照率染色强度
$phongalbedoboost int 反照率增强
$phongfresnelranges string Fresnel 范围，如 "[0.1 0.5 1.0]"
$halflambert int 使用 Half-Lambert 光照

环境映射

参数 说明
$envmaptint 环境贴图染色，如 "[1.0 0.9 0.8]"
$envmapsaturation 饱和度
$envmapcontrast 对比度
$envmapfresnel Fresnel 强度
$fresnelreflection Fresnel 反射强度

透明度与混合

参数 说明
$alphatest 启用 Alpha 测试
$alphatestreference Alpha 测试参考值
$translucent 半透明材质
$additive 叠加混合

细节纹理

参数 说明
$detailscale 细节纹理缩放
$detailblendmode 混合模式
$detailblendfactor 混合因子

边缘光

参数 说明
$rimlight 启用边缘光
$rimlightexponent 边缘光指数
$rimlightboost 边缘光强度

自发光

参数 说明
$selfillum 启用自发光
$selfillumtint 自发光染色，如 "[1.0 0.5 0.2]"

颜色调制

参数 说明
$color 主颜色，如 "[1.0 1.0 1.0]"
$color2 次颜色

杂项

参数 说明
$nocull 禁用背面剔除
$nodecal 不接受弹痕
$basemapalphaphongmask 基础贴图 Alpha 通道作为 Phong 遮罩
$normalmapalphaenvmapmask 法线贴图 Alpha 作为环境遮罩
$basealphaenvmapmask 基础贴图 Alpha 作为环境遮罩

自定义参数（预留）

参数 说明
$wear 磨损度 (0-1)
$seed 图案种子
$patternscale 图案缩放
$patternrotate 图案旋转

---

示例 skins.txt

```plaintext
"Skins"
{
    "1001"
    {
        "name"          "AK-47 | 红线"
        "description"   "自定义红色碳纤维"
        "weapon"        "ak47"
        "item_type"     "weapon"
        "rarity"        "4"          // 保密
        "icon"          "materials/vgui/icons/skins/ak47_redline.vtf"
        "base"          "models/weapons/customization/paints/custom/ak47_redline"
        
        // VMT 参数
        "$phong"                "1"
        "$phongboost"           "0.5"
        "$envmap"               "env_cubemap"
        "$envmaptint"           "[0.8 0.2 0.2]"
        "$basetexture"          "models/weapons/customization/paints/custom/ak47_redline"
    }

    "2001"
    {
        "name"          "M9 刺刀 | 多普勒"
        "description"   "多普勒相位 2"
        "weapon"        "knife_m9_bayonet"
        "item_type"     "knife"
        "rarity"        "5"
        "icon"          "materials/vgui/icons/skins/m9_doppler.vtf"
        "base"          "models/weapons/customization/paints/custom/knife_doppler"
        
        "$phong"        "1"
        "$phongboost"   "1.2"
        "$basetexture"  "models/weapons/customization/paints/custom/knife_doppler_phase2"
    }

    "3001"
    {
        "name"          "运动手套 | 潘多拉之盒"
        "description"   ""
        "glove_class"   "sporty"
        "item_type"     "gloves"
        "rarity"        "5"
        "icon"          "materials/vgui/icons/gloves/sporty_pandora.vtf"
        "base"          "models/weapons/customization/gloves/sporty_pandora"
        
        "$basetexture"  "models/weapons/customization/gloves/sporty_pandora_color"
        "$bumpmap"      "models/weapons/customization/gloves/sporty_normal"
        "$phong"        "1"
    }

    // 多材质示例（例如带独立配件的武器）
    "4001"
    {
        "name"          "AWP | 巨龙传说"
        "weapon"        "awp"
        "rarity"        "6"
        "icon"          "materials/vgui/icons/skins/awp_dragonlore.vtf"
        
        "material0"
        {
            "base"          "models/weapons/customization/paints/custom/awp_body"
            "$basetexture"  "models/weapons/customization/paints/custom/awp_dragonlore_body"
        }
        "material1"
        {
            "base"          "models/weapons/customization/paints/custom/awp_scope"
            "$basetexture"  "models/weapons/customization/paints/custom/awp_dragonlore_scope"
        }
    }
}
```

---

常见问题

如何为新武器添加皮肤？

1. 在 skins.txt 中添加新的 PaintKit ID 块。
2. 设置 weapon 字段为正确的武器别名。
3. 指定基础材质路径（base 或 materialX）。
4. 确保纹理文件存在于对应路径。

手套皮肤如何关联到 ConVar？

系统根据 glove_class 自动匹配手套槽位索引（见上表）。保存到配置时，会更新 loadout_gloves_ct_skin<index> 和 loadout_slot_gloves_ct 等 ConVar。

如何调试皮肤材质？

· 控制台命令 mat_reloadmaterial 可重新加载材质。
· 数据库启动时会输出加载日志，检查控制台是否有 [SkinDB] 前缀的错误。

探员和音乐包如何添加？

目前探员和音乐包列表硬编码在 cs_inventory.cpp 的静态数组中。如需添加，需修改源代码并重新编译。

---

文档版本：1.0
最后更新：2026-04-19