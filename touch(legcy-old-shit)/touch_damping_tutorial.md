# Touch 阻尼说明与调优

本文档汇总并解释代码中与“阻尼 / 平滑 / 惯性”有关的所有控制变量（ConVar），说明它们的数学意义、对触控/视角手感的实际影响、如何调节与调参建议，适合作为 Wiki 的简易教程。

---

## 📑 目录

1. 概览  
2. 与阻尼相关的控制项清单（含默认值）  
3. 算法与公式（逐步说明）  
4. 每个指令的作用、效果与调节建议  
5. 快速调参（预设）与测试方法  
6. 常见问题与排查  
7. 小结  

---

## 1) 概览

代码中存在两套“阻尼/平滑”实现：

- **原始的 Look 阻尼 / 惯性（`touch_look_*` 系列）**  
  用于视角移动的主阻尼，包含线性削弱、幂曲线、动量累积和惯性释放。

- **新增的 ApplyTouchDamping（`touch_damping` + `touch_sensitivity`）**  
  基于自适应指数平滑（EMA），将灵敏度与阻尼参数分离，方便直观调节。

- **滤波器 `touch_filter`**  
  在输出层进行一次帧间加权平均，进一步平滑但会增加延迟。

---

## 2) 控制项清单

| 指令                     | 默认值 | 功能描述 |
|--------------------------|--------|----------|
| `touch_look_damping`     | 0.1    | 削弱本帧输入，增大值=更平滑但更迟钝 |
| `touch_look_damping_power` | 1.1  | 幂曲线调整：>1 压缩小输入，<1 放大小输入 |
| `touch_look_inertia`     | 0.2    | 控制惯性释放速率，越大越直接，越小越漂移 |
| `touch_filter`           | 0      | 启用帧间加权滤波（轻度平滑+延迟） |
| `touch_precise_amount`   | 0.5    | 精确瞄准按钮时灵敏度乘子 |
| `touch_sensitivity`（新）| 1.0    | 基础灵敏度，与阻尼独立 |
| `touch_damping`（新）    | 0.3    | 自适应阻尼强度，0=无，1=最强 |

---

## 3) 算法与公式

### 🔹 原始 Look 阻尼（核心伪代码）

```cpp
float damped = raw * (1 - damping);
if (dampingPower != 1.0f)
    damped = copysignf(powf(fabsf(damped), dampingPower), damped);

remaining += damped;
frame = remaining * inertia * (frameTime / 0.016f);
remaining -= frame;
```

- `damping`：削弱输入幅度。  
- `dampingPower`：幂曲线，调节小动作灵敏度。  
- `inertia`：控制惯性释放速度。  

### 🔹 新的 ApplyTouchDamping

```cpp
float velocity = fabs(raw - prev);
float adaptiveAlpha = clamp(0.1f + velocity * (1 - damping), 0.1f, 1.0f);
return prev + adaptiveAlpha * (raw - prev);
```

- 小动作 → 阻尼强（更稳）  
- 大动作 → 跟随快（减少延迟）  

---

## 4) 指令详解与调节建议

### `touch_look_damping`
- **作用**：削弱输入，增强平滑度。  
- **建议范围**：0.0–0.4。  

### `touch_look_damping_power`
- **作用**：调整小幅输入响应。  
- **建议范围**：1.0–1.3。  

### `touch_look_inertia`
- **作用**：控制惯性释放速度。  
- **建议范围**：0.1–0.6。  

### `touch_filter`
- **作用**：帧间滤波，进一步平滑。  
- **建议**：输入噪声明显时开启。  

### `touch_precise_amount`
- **作用**：瞄准时降低灵敏度。  
- **建议范围**：0.2–0.6。  

### `touch_sensitivity`（新）
- **作用**：基础灵敏度，不影响阻尼。  

### `touch_damping`（新）
- **作用**：自适应阻尼，分离灵敏度和平滑度。  
- **建议范围**：0.2–0.4。  

---

## 5) 快速调参与预设

### 🔧 调参流程
1. 关闭滤波（`touch_filter 0`）。  
2. 固定灵敏度（`touch_sensitivity 1.0`）。  
3. 从 `touch_look_damping 0.0` 开始逐步增加。  
4. 根据需要调整 `touch_look_inertia`。  
5. 若要小动作稳，大动作快 → 使用 `touch_damping`。  
6. 最后可再开 `touch_filter` 做微调。  

### 🌟 预设示例

- **低延迟**  
```text
touch_look_damping 0.0
touch_look_inertia 0.1
touch_filter 0
```

- **推荐平衡**  
```text
touch_look_damping 0.08
touch_look_damping_power 1.1
touch_look_inertia 0.2
touch_filter 1
```

- **电影感平滑**  
```text
touch_look_damping 0.3
touch_look_damping_power 1.2
touch_look_inertia 0.35
touch_filter 1
```

---

## 6) 常见问题与排查

- **重复定义错误**：确保 `touch_sensitivity`/`touch_damping` 只在一个 `.cpp` 定义，其他地方用 `extern`。  
- **阻尼影响灵敏度**：使用新参数 `touch_sensitivity + touch_damping` 组合，避免耦合。  
- **帧率不同手感不一致**：检查 `inertia` 设置，按需在高帧率下减小。  

---

## 7) 小结

- `touch_look_*` 系列 → 原始阻尼 + 惯性。  
- `touch_filter` → 额外滤波。  
- 新的 `touch_sensitivity` + `touch_damping` → 分离灵敏度和平滑度，更直观调参。  
- 推荐流程：固定灵敏度 → 增加阻尼 → 调整惯性 → 启用滤波微调。  
