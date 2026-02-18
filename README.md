# 🎮 Source Engine Platform Wiki
### Source Engine 跨平台项目文档

> 维护者：真纪会java（MakiJava） · ZZH

---

## 1. 项目概述

本项目基于 **Source Engine (TF2 2018 泄露版本代码)** 进行移植与跨平台适配，目标：

- 在 Linux / Android (aarch64) 上运行 Source 系列游戏
- 统一构建流程
- 提供轻量级启动器与资源兼容层
- 支持 Mod / 社区内容扩展

### 引擎背景

**Source Engine** 由 Valve 开发：

- 2004：Half-Life: Source
- 2004：Counter-Strike: Source
- 2004：Half-Life 2

特性：

- 无明确大版本号
- 采用增量迭代式更新
- 同一引擎分支高度兼容

---

## ⚠️ 法律与使用声明（必须阅读）

> ⚠️ 本仓库仅供学习 / 研究 / 个人用途

- 源码基于 **TF2 2018 泄露版本**
- ❌ 严禁商业用途
- ❌ 不得用于盈利发行
- ❌ 不提供官方支持

使用风险自行承担。

---

## 2. 构建系统

本项目使用：

```
waf + python3 + clang + Android NDK
```

### 官方文档

https://waf.io/book

### 支持环境

| 平台 | 状态 |
|--------|--------|
| Linux (aarch64) | ✅ 支持 |
| Android (NDK r28c) | ✅ 支持 |
| Windows | ⚠️ 非主要目标 |
| Valve 20th 更新 | ❌ 暂不兼容 |

---

## 3. 发布策略

当前仅发布：

- 📱 Android Release 版本

原因：

- 维护成本可控
- Android 需求最高
- Linux 主要作为开发/调试环境

---

## 4. 支持游戏 / Mod

### 官方游戏

- Half-Life 2
- HL2 Episode One
- HL2 Episode Two
- Portal
- Counter-Strike: Source
- Day of Defeat: Source

### 社区项目 / Mod

- TF2 Classic
- Entropy : Zero 2
- Mapbase
- Rmui

---

## 5. 功能特性

### UI / 输入

- ✅ GamepadUI
- ✅ 手柄完整支持

### 内容系统

- ✅ Fake Steam Workshop（本地创意工坊模拟）
- ✅ PNG 直接加载（无需 VTF 转换，节省流程）

### 跨平台能力

- Android ARM64
- Linux ARM64
- 同一资源包复用

---

## 6. 项目依赖

```
waf
python3
clang 19
Android NDK r28c
stb_image
libpng
```

说明：

- stb_image：快速图片解码
- libpng：PNG 支持
- clang：统一编译链

---

## 7. 启动器（Platform Launcher）

### 信息

- 开发者：zzh の life
- 语言：Kotlin + Dart

### 作用

- 游戏管理
- 资源加载
- 启动参数控制
- 多 Mod 切换
- Android 端适配层

属于：

> 引擎外壳 / 平台层，而非核心引擎代码

---

## 8. 贡献者

- 真纪会java — https://github.com/KonuriMakiJava
- ItzVladik — https://github.com/ItzVladik
- ZZH
- nillerusr — https://github.com/nillerusr

感谢所有参与移植与适配的开发者。

---

## 9. 项目定位总结

这是一个：

- 学习型
- 移植型
- 社区驱动
- 非商业

的 Source Engine 跨平台方案。

目标是：

> 让老引擎在新平台继续可用，而不是重写引擎。

如果你需要：

- 移植
- Mod 适配
- Android 运行
- 轻量 Launcher

本项目就是为此存在。
