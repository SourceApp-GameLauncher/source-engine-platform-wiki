# 🎮 Touch 阻尼系统说明与调优指南
### Source / Android 触控视角平滑参数完整手册

本页系统性梳理 **触控输入 → 视角旋转** 链路中所有与：

- 阻尼（Damping）
- 平滑（Smoothing）
- 惯性（Inertia）
- 滤波（Filter）

相关的 ConVar，并解释：

- 数学原理
- 实际手感影响
- 调参逻辑
- 推荐预设

目标：

> 让你能“可预测地”调手感，而不是盲猜数值。

---

## 1. 总览架构

当前存在 **两套平滑系统 + 一层滤波**：

```
原始输入
   ↓
Look Damping（线性 + 幂曲线 + 惯性）
   ↓
ApplyTouchDamping（自适应 EMA）
   ↓
touch_filter（帧间滤波，可选）
   ↓
最终视角
```

### 分工

| 模块 | 职责 |
|--------|--------|
| touch_look_* | 主阻尼 + 惯性 |
| touch_damping | 自适应平滑（新方案） |
| touch_filter | 最终滤波 |
| touch_sensitivity | 纯灵敏度倍率 |

---

## 2. 控制项速查表

| 指令 | 默认 | 类型 | 影响 |
|--------|--------|--------|--------|
| touch_look_damping | 0.1 | 线性阻尼 | 输入削弱 |
| touch_look_damping_power | 1.1 | 幂曲线 | 小动作灵敏度 |
| touch_look_inertia | 0.2 | 惯性 | 拖尾感 |
| touch_filter | 0 | 滤波 | 抗抖但增加延迟 |
| touch_precise_amount | 0.5 | 倍率 | 精准模式 |
| touch_sensitivity | 1.0 | 灵敏度 | 总速度 |
| touch_damping | 0.3 | 自适应阻尼 | 动态平滑 |

---

## 3. 核心算法解析

### 3.1 原始 Look 阻尼

```cpp
float damped = raw * (1 - damping);

if (dampingPower != 1.0f)
    damped = copysignf(powf(fabsf(damped), dampingPower), damped);

remaining += damped;

frame = remaining * inertia * (frameTime / 0.016f);

remaining -= frame;
```

### 数学含义

分三步：

① 线性削弱
```
input × (1 - damping)
```

② 幂曲线变形
```
|x|^power
```

③ 惯性释放（动量积分）
```
remaining → 慢慢释放到 frame
```

### 体感总结

| 参数 | 手感变化 |
|--------|------------|
| damping ↑ | 更稳但迟钝 |
| power ↑ | 小动作更细腻 |
| inertia ↓ | 更拖泥带水 |
| inertia ↑ | 更跟手 |

---

## 3.2 新的 ApplyTouchDamping（推荐）

```cpp
float velocity = fabs(raw - prev);
float alpha = clamp(0.1f + velocity * (1 - damping), 0.1f, 1.0f);
return prev + alpha * (raw - prev);
```

### 原理

这是 **EMA（指数移动平均）自适应版**：

- 小速度 → 小 alpha → 强平滑
- 大速度 → 大 alpha → 快速跟手

### 优点

相比旧系统：

- 不影响最大速度
- 不耦合灵敏度
- 更自然
- 延迟更低

实战体验明显更好。

👉 强烈推荐主要依赖 `touch_damping` 调节。

---

## 4. 每个参数怎么调（实战解释）

### touch_look_damping
主阻尼强度。

- 太低 → 抖动明显
- 太高 → 像拖泥

建议：
```
0.05 – 0.15
```

---

### touch_look_damping_power
微调小幅动作精度。

- >1：微动更精细（推荐）
- <1：微动放大（不推荐）

建议：
```
1.05 – 1.2
```

---

### touch_look_inertia
惯性拖尾感。

- 小 → 电影感拖尾
- 大 → 电竞风跟手

建议：
```
0.15 – 0.35
```

---

### touch_filter
简单帧平均：

```
(prev + current) / 2
```

副作用：
- + 平滑
- + 延迟

建议：
- 高帧率设备：关
- 低端机抖动：开

---

### touch_sensitivity（新）

只负责：

> 速度倍率

完全不影响阻尼。

这是 **唯一应该改灵敏度的地方**。

---

### touch_damping（新，核心推荐）

自适应阻尼强度：

- 0 = 无平滑
- 1 = 强平滑

建议：
```
0.2 – 0.4（甜点区）
```

实战：

- 0.25 → 电竞
- 0.3 → 平衡
- 0.4 → 稳定电影感

---

## 5. 标准调参流程（强烈推荐）

严格按顺序：

```
① touch_filter 0
② 固定 touch_sensitivity
③ 调 touch_damping（主手感）
④ 微调 look_damping / inertia
⑤ 最后再考虑 touch_filter
```

⚠️ 不要同时改多个，否则无法判断因果。

---

## 6. 预设配置

### ⚡ 低延迟 / 竞技

```
touch_sensitivity 1.0
touch_damping 0.2
touch_look_damping 0.03
touch_look_inertia 0.4
touch_filter 0
```

---

### ⭐ 推荐平衡（默认建议）

```
touch_sensitivity 1.0
touch_damping 0.3
touch_look_damping 0.08
touch_look_damping_power 1.1
touch_look_inertia 0.25
touch_filter 0
```

---

### 🎬 电影感顺滑

```
touch_sensitivity 0.9
touch_damping 0.4
touch_look_damping 0.2
touch_look_inertia 0.2
touch_filter 1
```

---

## 7. 常见问题

### Q：感觉“又慢又粘”
→ damping 太高 或 filter 开着

---

### Q：抖动严重
→ damping 太低 或 帧率不稳

---

### Q：不同帧率手感不同
→ inertia 依赖 frameTime，需适当调高

---

### Q：灵敏度变低了
→ 别动 damping，改 sensitivity

---

## 8. 最终建议（经验总结）

实战经验：

- 新系统 > 旧系统
- 优先调 `touch_damping`
- 旧 look_* 只做细节微调
- touch_filter 最后再用

一句话：

> sensitivity 决定“多快”，damping 决定“多稳”。

两者职责分离，手感才可控。
