Touch 阻尼说明与调优

本文档汇总并解释代码中与“阻尼 / 平滑 / 惯性”有关的所有控制变量（ConVar），说明它们的数学意义、对触控/视角手感的实际影响、如何调节与调参建议，适合作为 Wiki 的简易教程。


---

目录

1. 概览


2. 与阻尼相关的控制项清单（含默认值）


3. 算法与公式（逐步说明）


4. 每个指令的作用、效果与调节建议


5. 快速调参（预设）与测试方法


6. 常见问题与排查（包括重复定义的链接错误）


7. 小结




---

1) 概览

代码中存在两套/两个层面的“阻尼/平滑”实现：

原始的 Look 阻尼 / 惯性（代码里的 touch_look_* 组）：这是模块内用于“视角（look）”的主阻尼实现。它先按系数缩减当前输入、可选应用幂曲线（shape），把结果累积到一个“剩余量（momentum）”里，然后每帧按惯性比例释放一部分。效果是带惯性的平滑移动。

我们新增的 ApplyTouchDamping（touch_damping + touch_sensitivity）：一个基于自适应指数/EMA 思路的平滑器，目的是把灵敏度与阻尼参数拆分开，方便直观调节：灵敏度只调速率，阻尼只调平滑度（不直接缩减灵敏度）。


此外还有一个滤波开关 touch_filter，它在输出上再做一次加权平均（frame 与上一帧的加权），相当于一个小的低通滤波。


---

2) 与阻尼相关的控制项（ConVar）

下面列出代码中与阻尼/平滑/惯性相关的 ConVar：

touch_look_damping — 默认 0.1
描述：look damping factor (0.0 - 0.95)

touch_look_damping_power — 默认 1.1
描述：look damping power curve (1.0 - 2.0)

touch_look_inertia — 默认 0.2
描述：look inertia amount (0.1 - 1.0)

touch_filter — 默认 0（布尔）
描述：启用额外的帧间加权平均滤波（输出平滑）

touch_precise_amount — 默认 0.5
描述：按下「精确瞄准」按钮时的灵敏度乘子（间接影响体验）

touch_sensitivity — 默认 1.0（新增）
描述：基础灵敏度（建议与阻尼分离调整）

touch_damping — 默认 0.3（新增）
描述：新实现的阻尼强度 (0 = none, 1 = max)，用于 ApplyTouchDamping 的自适应平滑


> 代码中还存在 touch_pitch / touch_yaw（基础比例）与 touch_forwardzone / touch_sidezone（死区）等设置，它们并非直接“阻尼”，但会显著影响手感，调参时应一并考虑。




---

3) 算法与公式（逐步说明）

A. 原始 Look 阻尼（代码摘录 / 伪代码）

float damping = touch_look_damping.GetFloat();
float dampingPower = touch_look_damping_power.GetFloat();
float inertia = touch_look_inertia.GetFloat();

// 假设 rawYaw / rawPitch 是本帧直接映射后的输入
float dampedYaw  = rawYaw * (1.0f - damping);
float dampedPitch= rawPitch * (1.0f - damping);

// 可选的幂曲线，用于对小幅输入做非线性缩放
if (dampingPower != 1.0f) {
    dampedYaw   = copysignf(powf(fabsf(dampedYaw), dampingPower), dampedYaw);
    dampedPitch = copysignf(powf(fabsf(dampedPitch), dampingPower), dampedPitch);
}

// 把当前帧的贡献累积为 "剩余量"
m_flRemainingYaw += dampedYaw;
m_flRemainingPitch += dampedPitch;

// 每帧按惯性释放一部分（并按帧时间归一）
float frameYaw   = m_flRemainingYaw  * inertia * (flFrameTime / 0.016f);
float framePitch = m_flRemainingPitch * inertia * (flFrameTime / 0.016f);

// 从剩余量中减去已经释放的那部分
m_flRemainingYaw  -= frameYaw;
m_flRemainingPitch -= framePitch;

// 可选额外滤波：
if (touch_filter) {
    outYaw = frameYaw * 0.7 + prevYaw * 0.3;
} else {
    outYaw = frameYaw;
}

含义总结：

touch_look_damping 以线性方式 削弱本帧输入（乘 (1 - damping)），数值越大本帧贡献越小（越“柔软”）。

touch_look_damping_power 是一个形状控制：当 >1 时，对小幅输入进一步压缩（更容易抑制抖动）；<1 则放大小输入（更灵敏）。

touch_look_inertia 决定每帧从“剩余量”里释放多少：值越大，剩余量被更快消耗，响应更直接；值越小，动量释放更慢，产生更长的惯性/漂移感。

touch_filter 在输出上再做一次简单的低通，加权 frame 与上一帧，额外平滑但又增加延迟。


B. 新的自适应阻尼（ApplyTouchDamping，伪代码）

float damping = touch_damping.GetFloat();
if (damping <= 0) return raw; // 不阻尼

float velocity = fabs(raw - prev);
float adaptiveAlpha = clamp(0.1f + velocity * (1.0f - damping), 0.1f, 1.0f);
return prev + adaptiveAlpha * (raw - prev);

含义总结：

这是一个基于上一帧值的指数/递归平滑（类似 EMA），用 alpha 控制“跟随速度”。

adaptiveAlpha 会根据 velocity（原输入和上一输出的差值）放大：

小速度 → adaptiveAlpha 接近 0.1（更强的平滑）

大速度 → adaptiveAlpha 接近 1.0（跟随原始输入，减少延迟）


touch_damping 在这里决定对速度的抑制强度；阻尼越大，(1.0 - damping) 越小，同样速度下 adaptiveAlpha 更小 → 更强平滑（更大延迟）。


> 这种实现把灵敏度（touch_sensitivity）与平滑（touch_damping）分离：你可以先用 touch_sensitivity 控制 input->world 的比例，再用 touch_damping 平滑运动轨迹。




---

4) 指令逐项说明（作用、效果、如何使用）

下面按指令给出更实践的说明与具体示例。

touch_look_damping  — 默认 0.1

作用：按比例削弱本帧视角输入（raw * (1 - damping)）。

效果：增大值会让视角移动更“软”、更平滑，但会显著增加感觉上的迟滞（特别是配合较大 inertia 时）。

使用：控制台输入 touch_look_damping 0.08（实时生效）。

建议范围：0.0（无）— 0.4（可用）；>0.5 会非常迟钝。


touch_look_damping_power — 默认 1.1

作用：对已缩减的输入应用幂函数 sign(pow(|x|, p))，用来控制小幅/大幅输入的非线性响应。

效果：

>1.0：对小幅输入压缩（更稳），对较大输入相对保留；配合 damping 可抑制抖动。

=1.0：无非线性变化（线性）。

<1.0：放大小输入（更灵敏，可能放大抖动）。


使用：touch_look_damping_power 1.2。

建议范围：1.0 — 1.3。


touch_look_inertia — 默认 0.2

作用：每帧按 frame = remaining * inertia * (frameTime/0.016) 释放剩余量。

效果：

值越高：剩余量被快速消耗 → 响应更直接、惯性更短。

值越低：剩余量消耗慢 → 更明显的漂移/惯性。


使用：touch_look_inertia 0.3。

建议范围：0.1 — 0.6，高帧率时可以适当减小 base 值。


touch_filter — 默认 0（false）

作用：启用额外的帧间加权平均（out = frame*0.7 + prev*0.3）。

效果：轻微平滑输出，抑制高频抖动，但会额外增加约 1 帧的延迟感。

使用：touch_filter 1 to enable。

建议：当输入噪声明显（触摸板/手抖）且不追求极低延迟时可启用。


touch_precise_amount — 默认 0.5

作用：在按下“精确瞄准”按钮时对灵敏度做乘子（sensitivity *= touch_precise_amount）。

效果：配合阻尼可使精确瞄准更稳，但不要把它当作主要阻尼手段。

使用：touch_precise_amount 0.4。

建议：0.2—0.6 之间常见。


touch_sensitivity（新增） — 默认 1.0

作用：基础灵敏度乘子。建议把“灵敏度”与“阻尼”分离，先用此调整整体速率。

效果：直接放大/缩小输入幅度，不改变平滑策略。

使用：touch_sensitivity 0.9。

建议：视玩家习惯设定；调试阻尼时先把灵敏度固定在合适值。


touch_damping（新增） — 默认 0.3

作用：配合 ApplyTouchDamping 的自适应阻尼强度（0 = 无，1 = 最强）。

效果：以速度为依据进行平滑：小动作时平滑强（抑制抖动），大动作时快速响应。

使用：touch_damping 0.25。

建议范围：0.0（完全原始） — 0.6（中等平滑）；对大多数触控面板 0.2—0.4 很实用。


5) 快速调参（预设）与测试流程

基本建议的调参流程（快速迭代）

1. 先把 touch_filter 0 关掉，避免混淆多个平滑器的效果。


2. 固定 touch_sensitivity 为一个你熟悉的基准（比如 1.0 或者根据 touch_pitch/touch_yaw 设好基础值）。


3. 测试“原始感”（滑/放开快速移动）：把 touch_look_damping 0.0，touch_look_inertia 0.1，touch_look_damping_power 1.0，感受延迟与抖动。


4. 慢慢增加 touch_look_damping（例如每次 +0.05），观察抖动/延迟的权衡。


5. 根据需要微调 touch_look_inertia（更高 → 更直接；更低 → 更飘）。


6. 若要“只在小动作时抑制抖动但不影响大动作”，尝试启用 ApplyTouchDamping（touch_damping + touch_sensitivity）或把 touch_look_damping_power 提升到 ~1.1。


7. 最终启用 touch_filter 1 作为细调（如果需要更平滑输出）并再次微调 damping/inertia。



预设示例（控制台命令）

Raw / Twitch（低延迟）


// 低阻尼、低惯性、关闭滤波
touch_look_damping 0.0
touch_look_damping_power 1.0
touch_look_inertia 0.1
touch_filter 0

Mild Smooth（推荐，游戏手感平衡）


touch_look_damping 0.08
touch_look_damping_power 1.08
touch_look_inertia 0.2
touch_filter 1

Cinematic / Very Smooth（更平滑但更迟钝）


touch_look_damping 0.3
touch_look_damping_power 1.2
touch_look_inertia 0.35
touch_filter 1

Precision Aim（靠精确按钮配合）


touch_precise_amount 0.35
// 保持较低 damping，使用精确按钮减速而非提高阻尼
touch_look_damping 0.05
touch_look_inertia 0.15


---

6) 常见问题与排查

1) 编译时报 duplicate symbol: touch_sensitivity/链接重复定义

原因：该 ConVar 在多个 .cpp 文件里被定义（而不是在 touch.h 中 extern 声明并只在一个 .cpp 定义）。

解决：

在 touch.h 中加入 extern ConVar touch_sensitivity; extern ConVar touch_damping; 声明。

只在 touch.cpp（或某个单一实现文件）中写 ConVar touch_sensitivity("touch_sensitivity", "1.0", ...); 的定义。



2) 调大阻尼后感觉灵敏度变低（不想要的副作用）

说明：原始 touch_look_damping 的实现是把本帧输入乘以 (1 - damping)，因此阻尼与灵敏度耦合。

建议：采用 touch_sensitivity + touch_damping(ApplyTouchDamping) 的方案：先用 touch_sensitivity 调整幅度，再用 ApplyTouchDamping 做平滑（不会直接缩减灵敏度）。


3) 高帧率 / 低帧率下手感不一致

算法中包含 flFrameTime / 0.016f 的缩放，已尝试按帧时间归一。但仍建议做跨机测试并适当在高帧率下微调 touch_look_inertia。



---

7) 小结（快速要点）

touch_look_damping / touch_look_damping_power / touch_look_inertia 构成了“原始阻尼 + 幂形状 + 惯性”组合：一起决定“软/硬/漂移/延迟”的权衡。

touch_filter 是输出层的轻量滤波器。

建议把灵敏度(touch_sensitivity)与平滑(touch_damping 或 touch_look_damping)分离，这样调参直观且不会互相干扰。

调参流程：固定灵敏度 → 关闭滤波 → 从 0 开始逐步增加阻尼 → 适配 inertia → 最后启用 filter 做微调。



---

如果你希望，我可以：

把上面的预设直接写成 touch.cfg 样例文件；

或者把代码里原有 touch_look_* 的调用替换为使用 ApplyTouchDamping（我会同时修复 extern / 重复定义问题），并生成一个可直接编译的 patch 文件。


需要哪个后续操作，告诉我就行。



我已经把完整的 Markdown 教程 放到侧边的文档（Touch 阻尼说明与调优.md）。里面包含：指令清单、数学公式、每项指令的作用与效果、调参建议、预设命令与常见问题排查（包含你之前遇到的重复定义链接错误的解决方法）。

如果你要我继续：

我可以把几个实用预设生成 touch.cfg 样例，或者

直接帮你在代码里把 touch_look_* 调用切换到 ApplyTouchDamping（并修复 extern 重定义问题），生成一个 patch / 直接修改文件并保存。


你想先要哪一个？

