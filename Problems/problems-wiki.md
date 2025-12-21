# 🐞 Problems & Troubleshooting

> **最后更新**：2025-08-14  
> 本文记录了当前 Source Engine 跨平台项目存在的主要技术问题及解决建议。

---

## 📝 问题大纲

1. ⚡ **急需添加** Valve 20th 更新的 **模组管理界面**（`vgui2`）  
2. 🎨 **Valve 20th Shader** 兼容问题（渲染错误）  
3. 🖐️ 重写 **Touch 系统**（使用 PNG 替代 `.vtf` 按键贴图）  
4. 🔊 使用 **FMOD** 声音系统替代现有实现  
5. 🛠 添加 **CMake** 构建系统支持  

---

## 🎨 Shader 兼容性问题（Valve 20th Update）

### 🚨 错误现象

在 `shader.log` 中可以看到多个关键渲染错误，表明 **着色器编译或链接失败**，直接导致：
- 地图贴图消失
- 光照显示异常
- 模型透明/渲染错误

#### 主要报错示例：

CGLMShaderPair::SetProgramPair: Centroid masks differ at link time of vertex shader lightmappedgeneric_vs20 and pixel shader null!

> **说明**：像素着色器（Fragment Shader）缺失或未编译成功，导致整个材质系统渲染失败。

Centroid masks differ at link time of vertex shader vertexlit_and_unlit_generic_bump_vs20 and pixel shader shadow_ps20b!

> **说明**：光照 / 法线贴图相关的 Shader 对无法成功链接。

---

### 📌 问题根源分析

| 可能原因 | 详细说明 |
|----------|----------|
| **1. GLES 3.0 / 移动不兼容语法** | 引擎加载 `#version 300 es` 着色器，但移动平台和移植引擎对其支持不完整，导致编译失败。 |
| **2. Pixel Shader 缺失** | 报错中的 `pixel shader null!` 表示像素着色器文件未加载或损坏。 |
| **3. GPU 限制** | 中低端 / 老款 Android 设备不支持复杂材质管线（如 `bump mapping`、`shadow mapping`）。 |
| **4. Valve 20th 资源结构变化** | 更新后的 Shader/VPK 结构与移植引擎不匹配。 |

---

### ✅ 解决建议

#### 方法 1：切换低配材质设置
在启动参数或配置文件中禁用高开销特性：
```
使用 Android 适配资源
优先使用 2004 原版资源 社区适配的 HL2 资源包
替换 Shader 目录
使用简化版 .glsl Shader 替换高质量版本
从社区获取可在 GLES 平台运行的 Shader 替代方案
```

---

🖐️ Touch 系统问题

现有系统使用 .vtf 按键贴图，不适合移动平台
解决方案：重写 Touch 逻辑，改用 .png 按键贴图以减少解码和渲染成本



---

🔊 声音系统改进

现有声音模块在 Android 上性能/兼容性差
建议：引入 FMOD 作为跨平台音频引擎，提高音质与性能



---

🛠 构建系统优化

目前使用 waf 构建，部分开发者不熟悉
建议：新增 CMake 构建脚本，提升跨平台构建兼容性和 IDE 支持



---

📌 后续计划

[ ] 完成 Valve 20th VGUI2 模组管理界面 适配

[ ] 修复 20th Shader 兼容性问题

[ ] 重写 Touch 系统（PNG 按键贴图）

[ ] 集成 FMOD 音频引擎

[ ] 添加 CMake 构建支持



