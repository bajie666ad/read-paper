# 【IEEE Sensors Journal'2025】铁路道岔系统精确建模与高精度状态估计方法 / Accurate Modeling of Railway Turnout Systems and High-Precision State Estimation Methods

## Metadata

- Title: Accurate Modeling of Railway Turnout Systems and High-Precision State Estimation Methods
- 中文题名: 铁路道岔系统精确建模与高精度状态估计方法
- Authors: Junqi Liu, Tao Wen, Xia Fang, Baigen Cai, Clive Roberts
- Venue / Year: IEEE Sensors Journal, 25(8), 2025
- Note Name: 第三次-铁路道岔系统精确建模与高精度状态估计方法-Accurate Modeling of Railway Turnout Systems and High-Precision State Estimation Methods
- Paper: [PDF](../papers/第三次-铁路道岔系统精确建模与高精度状态估计方法-Accurate Modeling of Railway Turnout Systems and High-Precision State Estimation Methods.pdf); DOI `10.1109/JSEN.2025.3545619`
- Code: Not reported in the paper
- Dataset / Artifact: ZD6-A switch machine current data and BG simulation; data/code not reported as public
- Scope / Subfield: 铁路道岔系统机理建模；非线性状态空间；高阶 EKF 状态估计
- Tags: `Railway Turnout System`, `Switch Machine`, `Bond Graph`, `Nonlinear Modeling`, `Gear Backlash`, `Stribeck Friction`, `Extended Kalman Filter`, `State Estimation`
- Status: DONE

## TL;DR

这篇论文面向铁路道岔系统（RTS）机理模型不够精确的问题，用 bond graph（BG）建立了一个包含传感器模型、齿隙（gear backlash）和 Stribeck 非线性摩擦的 ZD6-A 转辙系统非线性模型，并推导出对应状态空间方程。随后作者提出改进 EKF：不仅使用一阶线性化，还引入非线性 Taylor 展开的二阶、三阶统计信息，用于电流和电机角速度状态估计。实验表明，加入非线性因素的模型更贴近真实电流；3oEKF 在正常和异常工况下均比 EKF、UKF、AEKF、AUKF 有更低 RMSE。

## 毒舌评论

这篇文章的价值不是“又做了一个滤波器”，而是把转辙机从黑箱诊断往可解释机理模型推进了一步：电机、电气、减速器、锁闭齿轮、动作杆、尖轨负载这些环节都有物理位置。但它也有典型模型论文的软肋：参数主要来自手册、文献和仿真调参，真实验证主要围绕电流、行程和转换时间，离完整现场泛化还有距离。3oEKF 的精度提升很亮眼，但本质是在作者构造的非线性模型上做仿真/状态估计验证；如果模型参数漂移、测量噪声非高斯、或者现场非线性比论文假设更脏，优势未必能原样保留。

## Research Question

- 研究对象：铁路道岔系统，重点是 ZD6-A 直流电动转辙机驱动的单机牵引道岔系统。
- 小领域范围：模型驱动的道岔系统动态建模与状态估计。
- 具体问题：如何建立能反映齿隙、非线性摩擦等真实非线性因素的 RTS 模型，并在该非线性模型上获得高精度状态估计。
- 为什么重要：道岔是铁路系统薄弱且关键的基础设施，状态估计精度会影响健康监测、故障预警和维护决策。
- 论文边界：主要验证 ZD6-A 单机牵引系统；测量模型为线性；噪声按高斯假设处理；没有展开跨型号、跨环境、长期退化场景验证。

## Motivation and Basic Idea

- Motivation：已有 RTS 模型往往只利用输入输出信号，或者忽略齿隙、摩擦等非线性因素，导致模型与真实系统偏差较大；传统 EKF 又只保留一阶 Taylor 项，非线性较强时估计误差大。
- Basic idea：先用 BG 统一表达 RTS 中电气、机械、传感器等多能域耦合，再把 Stribeck 摩擦和齿隙写进模型；滤波阶段用二阶/三阶 Taylor 统计信息修正 EKF。
- 这个 idea 如何回应 motivation：BG 解决复杂系统结构建模问题；非线性摩擦/齿隙补齐关键物理机制；3oEKF 用高阶统计信息减少一阶线性化误差。
- 作者给出的证据：模型输出行程 163.44 mm，落在 ZD6-A 标定行程 165 +/- 2 mm 范围内；模型转换时间 3.76 s，与标称不超过 3.8 s 一致；正常工况下 3oEKF 电流 RMSE 相比 EKF 降低 33.33%，异常工况下电流 RMSE 降低 28.26%，角速度 RMSE 降低 38.05%。
- 我的判断：这篇对你的方向很有用，因为它把“转辙机小样本故障诊断”前面的机理基座补上了：哪些状态变量值得估计、哪些非线性因素影响电流/速度曲线、模型残差如何可能作为诊断特征。

## Background

- 背景：RTS 承担列车换线功能，异常会造成运营中断甚至安全事故。
- 问题：数据驱动方法依赖历史数据和标签，解释性弱；简单模型驱动方法又不够贴近真实物理系统。
- Gap：已有 RTS 建模较少系统纳入非线性摩擦和齿隙；已有 EKF/UKF 处理非线性时仍存在精度和稳定性折中。

## Threat Model / Assumptions (Optional)

- 攻击者或参与者能力：N/A。
- 主要假设：RTS 可由连续可微非线性状态转移函数描述；系统噪声和测量噪声为相互独立的零均值高斯白噪声；测量方程是线性的。
- 工程假设：ZD6-A 的关键参数可由产品手册、文献和仿真实验确定；齿隙和摩擦是主要非线性因素。
- 不覆盖的情况：非高斯噪声、测量非线性、复杂环境扰动、多机牵引、多型号转辙机、长期磨耗造成的参数漂移。

## Method

- 核心思路：`BG 非线性建模 -> 状态空间方程 -> 3oEKF 高阶统计滤波 -> 正常/异常工况验证`。
- 核心思路是怎么想到的：RTS 是电气-机械-传感器多能域耦合系统，BG 正适合表达功率流与因果约束；而一阶 EKF 对非线性不足，所以引入 Taylor 高阶项。
- 从 motivation 到 method 的逻辑链：
  1. 转辙机电机、电气、减速器、主轴、锁闭齿轮、动作杆和尖轨负载构成多能域系统。
  2. BG 用 effort/flow 变量统一表达电气、旋转机械、平移机械等能量域。
  3. 在电机机械部分和动作杆/尖轨负载部分引入 Stribeck 非线性摩擦。
  4. 用齿隙死区和回弹机制建模 torque disturbance。
  5. 从 BG 因果关系推导电流、电机角速度、动作杆速度等状态方程。
  6. 离散化为非线性状态空间模型。
  7. 对状态转移函数做三阶 Taylor 展开，用 ULS 求二阶、三阶误差统计信息。
  8. 把这些统计信息纳入 EKF 预测、协方差和增益更新。
- 关键设计取舍：3oEKF 比 EKF/UKF 复杂，但作者认为执行时间仍接近 UKF，且精度提升明显；高于三阶的信息收益递减，因此只做到三阶。
- 为什么不是更直接 / 更简单的方案：线性模型无法解释非线性摩擦/齿隙造成的电流偏差；普通 EKF 只保留一阶项，在非线性系统中误差大；UKF 不需要显式导数但可能受参数选择和协方差正定性影响。
- 系统流程或算法步骤：
  1. 建立 ZD6-A 转辙系统 word BG。
  2. 将电机、电气、减速器、主轴、锁闭齿轮、齿条/动作杆、尖轨负载细化为 BG 元件。
  3. 加入 Stribeck 摩擦模型和齿隙扰动力矩。
  4. 由 BG 约束推导非线性状态空间方程。
  5. 用真实电流数据、行程和转换时间验证模型。
  6. 构建 2oEKF/3oEKF，并与 EKF、UKF、AEKF、AUKF 比较。
- 关键定义 / 公式 / 不变量：
  - Stribeck 摩擦：低速静摩擦、Coulomb 摩擦、黏性摩擦和速度阈值共同决定摩擦曲线。
  - 齿隙扰动：通过角位移差 `Delta theta = theta_m - theta_r/k1` 描述传递滞后和回弹。
  - RTS 状态空间：电机电流、电机角速度、动作杆/负载速度等是关键状态变量。
  - 3oEKF：状态预测中包含一阶、二阶、三阶误差项的统计信息。
- 实现细节：BG 仿真使用 20-SIM；滤波实验用 1000 次 Monte Carlo 统计；采样间隔 0.04 s。

## Evaluation

- 实验思路：
  - 模型准确性：用四种 ablation case 比较模型输出电流与真实电流。
  - 状态估计：在正常与异常工况下，对电流和电机角速度估计进行比较。
- 模型准确性 ablation：
  - Case 1：不考虑齿隙和非线性摩擦，最简线性模型。
  - Case 2：只考虑齿隙。
  - Case 3：只考虑非线性摩擦。
  - Case 4：同时考虑齿隙和非线性摩擦。
  - 结论：同时考虑齿隙和非线性摩擦的模型最接近真实电流；非线性摩擦影响更大，齿隙影响较小，原因可能是采集设备齿隙较小。
- 正常工况：
  - 运行时间 94 个采样点，即 3.76 s。
  - 电机角速度稳定在约 253.9 rad/s，即 2425.8 r/min，符合标定转速不低于 2400 r/min。
  - 2oEKF 电流 RMSE 相比 EKF 降低 20.51%；3oEKF 相比 EKF 降低 33.33%。
- 异常工况：
  - 将电机机械部分黏性摩擦系数在第 76 个采样点从 0.03 突变到 0.05。
  - 电流由约 0.3 A 跳到约 0.9 A，角速度由约 253.5 rad/s 降到约 199.5 rad/s。
  - 转换过程持续到第 141 个采样点，即 5.64 s，超过 ZD6-A 标定不超过 3.8 s 的转换时间。
  - 3oEKF 电流 RMSE 相比 EKF 改善 28.26%；角速度 RMSE 相比 EKF 改善 38.05%。

## Key Artifacts

- 关键图：
  - Fig. 1：ZD6-A 转辙机结构示意，明确电机、减速器、主轴、锁闭齿轮、动作杆、尖轨负载的物理链路。
  - Fig. 2：RTS word BG 和完整 BG 模型，是全篇机理建模核心。
  - Fig. 3：BG 仿真与真实电流对比，用于证明模型基本可信。
  - Fig. 4：四种非线性因素 ablation 的电流对比，支撑“非线性因素提高模型精度”。
  - Fig. 5-12：正常/异常工况下电流与角速度估计、MAE/MSE 曲线。
- 关键表：
  - Table I：不同能量域中的 generalized variables。
  - Table II：RTS-BG 模型主要参数。
  - Table III：四种模型 case 的 MAE/RMSE。
  - Table IV-VII：正常/异常工况下不同滤波方法的 RMSE。
  - Table VIII-IX：复杂度和执行时间分析。
- 关键公式 / 定义 / 算法：
  - Eq. (1)-(6)：Stribeck 摩擦和齿隙扰动力矩建模。
  - Eq. (9)、(13)、(16)、(17)：RTS 非线性状态空间方程。
  - Eq. (18)-(20)：离散非线性状态和线性测量模型。
  - Eq. (21)-(39)：三阶 Taylor 统计信息和 3oEKF 滤波更新。
- 这些证据分别支撑哪些结论：BG/状态方程支撑模型可解释性；ablation 支撑非线性因素必要性；RMSE 对比支撑 3oEKF 高精度状态估计。

## Findings

- 发现 1：对 ZD6-A RTS，非线性摩擦比齿隙对电流模型精度影响更明显。
- 发现 2：模型输出的行程和转换时间能够落在设备标定范围内，说明机理模型不是纯数学拟合。
- 发现 3：加入二阶/三阶统计信息后，EKF 对非线性状态估计的 RMSE 明显降低，尤其异常摩擦突变时仍保持优势。

## Strengths

- 论文最有说服力的地方：把 RTS 关键物理结构、非线性因素和状态估计连成闭环，而不是只做黑箱分类。
- 方法、数据、实验或问题设定的优势：BG 适合多能域建模；ablation 能说明非线性因素贡献；正常/异常工况都做了滤波验证。
- 相比已有工作的有效推进：从简化 RTS 模型推进到包含齿隙和非线性摩擦的模型，并把高阶 EKF 用于道岔状态估计。

## Limitations

- 威胁模型、假设或适用范围的限制：滤波依赖高斯噪声、线性测量和较准确模型参数；现场噪声、环境扰动和参数漂移可能削弱效果。
- 数据集、baseline、metric、ablation 或复现性的不足：缺少公开数据/代码；模型参数来源包含手册、文献和仿真实验，参数辨识过程不够透明；实验对象偏单一。
- 在真实铁路场景中可能失效的条件：转辙机型号变化、润滑状态长期退化、尖轨阻力随机波动、传感器漂移、外界温湿度和异物阻塞等复杂因素叠加。

## My Takeaways

- 对小样本道岔故障研究的启发：这篇适合作为“物理先验/数字孪生基座”，可用模型残差、状态估计误差、突变后的电流/角速度响应作为诊断特征。
- 可复用的方法：BG 建模、Stribeck 摩擦、齿隙扰动、3oEKF 状态估计都可以迁移到转辙机健康监测。
- 可能的后续问题：把 3oEKF 估计状态与数据驱动分类模型结合，做 model-informed fault diagnosis；或者用少量标注故障数据校准模型参数。

## Related Papers

- 前置阅读：Bond Graph 建模基础；Kalman filter、EKF、UKF、AEKF/AUKF；ZD6/ZDJ9 转辙机机理。
- 后续阅读：模型残差故障诊断、物理信息神经网络、数字孪生道岔系统、非高斯滤波。
- 可对比论文：数据驱动转辙机电流/振动故障诊断；基于简化转辙机模型的 cumulative residual fault diagnosis。
- 最接近的相关工作：Ladj 等使用 BG 建模 RTS，但未考虑非线性因素；Hamadache/Dutta 类简化转辙机数学模型。
- 关键差异：本文显式纳入齿隙和 Stribeck 摩擦，并把建模结果用于高阶 EKF 状态估计。

## Open Questions

- 能否用实测故障数据验证 3oEKF 估计误差对故障类型的区分能力？
- 模型参数如何在线辨识或自适应更新？
- 3oEKF 在非高斯噪声和非线性测量模型下是否仍稳定？
- 与纯数据驱动模型相比，模型残差特征能否降低标注样本需求？
- 这套 BG 模型如何迁移到 ZDJ9 或多机牵引道岔系统？
