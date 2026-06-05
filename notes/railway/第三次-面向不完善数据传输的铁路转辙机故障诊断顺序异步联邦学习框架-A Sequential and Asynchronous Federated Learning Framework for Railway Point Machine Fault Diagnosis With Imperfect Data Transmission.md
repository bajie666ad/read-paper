# 【IEEE TII'2024】面向不完善数据传输的铁路转辙机故障诊断顺序异步联邦学习框架 / A Sequential and Asynchronous Federated Learning Framework for Railway Point Machine Fault Diagnosis With Imperfect Data Transmission

## Metadata

- Title: A Sequential and Asynchronous Federated Learning Framework for Railway Point Machine Fault Diagnosis With Imperfect Data Transmission
- 中文题名: 面向不完善数据传输的铁路转辙机故障诊断顺序异步联邦学习框架
- Authors: Tao Wen, Xiaohan Chen, Dingcheng Zhang, Clive Roberts, Baigen Cai
- Venue / Year: IEEE Transactions on Industrial Informatics, 20(6), 2024
- Note Name: 第三次-面向不完善数据传输的铁路转辙机故障诊断顺序异步联邦学习框架-A Sequential and Asynchronous Federated Learning Framework for Railway Point Machine Fault Diagnosis With Imperfect Data Transmission
- Paper: [PDF](../../papers/railway/第三次-面向不完善数据传输的铁路转辙机故障诊断顺序异步联邦学习框架-A Sequential and Asynchronous Federated Learning Framework for Railway Point Machine Fault Diagnosis With Imperfect Data Transmission.pdf); DOI `10.1109/TII.2024.3378800`
- Code: Not reported in the paper
- Dataset / Artifact: ZDJ9 railway point machine vibration dataset, 16 conditions, 960 samples; data/code not reported as public
- Scope / Subfield: 铁路转辙机故障诊断；联邦学习；异步聚合；通信延迟与丢包鲁棒性
- Tags: `Railway Point Machine`, `Fault Diagnosis`, `Federated Learning`, `Asynchronous FL`, `Sequential Kalman Filtering`, `Packet Loss`, `Communication Delay`, `DS-FCN`
- Status: DONE

## TL;DR

这篇论文针对铁路转辙机分布式故障诊断中的隐私、通信成本、传输延迟和丢包问题，提出 SAFL（Sequential and Asynchronous Federated Learning）。它用参数量更小的 DS-FCN 作为全局模型，减少通信压力；再用时间周期机制和 Sequential Kalman Filtering（SKF）按参数到达顺序进行异步聚合，避免同步 FL 等所有客户端导致的等待，并降低丢包对全局模型的影响。实验在 ZDJ9 转辙机 16 类振动数据上验证，DS-FCN 以更少参数接近 DS-WCNN 准确率；SKF 聚合在不同训练策略、通信延迟和丢包率下表现更好。

## 毒舌评论

这篇文章很“工程现场感”：它没有假装无线传输永远可靠，而是把延迟和丢包摆上台面，这是铁路资产联邦学习里很实际的问题。但它的短板也明显：所谓真实部署场景主要还是仿真实验，五个客户端、随机延迟、随机丢包，和真实铁路通信链路的突发性、相关性、带宽竞争、边缘算力差异还有距离。DS-FCN 参数从 38832 降到 28848 是有意义的，但谈不上压倒性轻量；真正支撑论文的是 SKF 聚合思路，而不是网络结构本身。

## Research Question

- 研究对象：ZDJ9 铁路转辙机振动信号故障诊断。
- 小领域范围：面向铁路分布式资产的联邦学习故障诊断。
- 具体问题：在数据不能集中上传、客户端数据有限、通信存在延迟和丢包时，如何训练一个高准确率的转辙机故障诊断模型。
- 为什么重要：铁路转辙机分散部署，集中数据训练存在隐私、商业竞争和通信成本问题；同步 FL 在延迟和丢包下会等待过久或聚合不完整，影响准确率和效率。
- 论文边界：实验使用模拟客户端和人工设置的延迟/丢包率；未覆盖恶意客户端、非 IID 程度变化、真实网络链路日志、在线概念漂移。

## Motivation and Basic Idea

- Motivation：传统集中式深度学习要汇集数据，不适合铁路资产分布式与隐私约束；现有同步 FL 依赖所有客户端上传，遇到延迟和丢包会拖慢训练、损害精度。
- Basic idea：把 RPM 作为客户端，在本地训练；服务器不等所有客户端，而是在一个时间周期内按参数到达顺序用 SKF 连续聚合；同时用 DS-FCN 减少上传参数。
- 这个 idea 如何回应 motivation：DS-FCN 缩小参数量，降低拥塞和丢包可能性；异步时间周期避免长期等待；SKF 用 Kalman gain 动态调节每个到达参数对全局模型的影响。
- 作者给出的证据：DS-FCN 参数量 28848，低于 DS-WCNN 的 38832；SKF 策略在 AlexNet、DS-WCNN、DS-FCN 上分别达到 90.13%、95.96%、95.37% 准确率；在不同丢包率下 SKF-based FL 均取得最高准确率。
- 我的判断：这篇对“转辙机多站点/多设备协同诊断”很有启发，特别适合把每台转辙机或每个站场当作 client 的场景。

## Background

- 背景：RPM 负责执行道岔转换，常在露天环境中工作，受温度、湿度、润滑、异物和机械磨耗影响，容易出现机械/电气故障。
- 问题：信号处理方法依赖专家经验；集中式深度学习要聚合数据；同步 FL 面对延迟和丢包会等待、丢轮次或聚合质量下降。
- Gap：已有铁路资产 FL 诊断多假设通信较理想，或者只考虑延迟，不充分处理丢包；RPM 故障诊断中的 FL 研究仍少。

## Threat Model / Assumptions (Optional)

- 参与者能力：每个 RPM/边缘设备作为客户端，保留本地数据，只上传模型参数；服务器负责聚合全局模型。
- 数据假设：所有客户端任务类别相同，属于 horizontal FL；单个设备数据不足以训练强模型；数据不能直接共享。
- 通信假设：存在信息延迟和包丢失；丢包通过客户端每轮是否上传参数模拟；延迟用均匀随机等待时间模拟。
- 不覆盖的情况：恶意客户端投毒、隐私攻击、加密通信开销、强 non-IID 数据分布、真实通信链路的突发丢包。

## Method

- 核心思路：SAFL = DS-FCN 轻量全局模型 + 异步时间周期 + SKF 顺序参数聚合。
- 核心思路是怎么想到的：同步 FL 要等所有客户端，而铁路资产分散、无线通信不稳定；如果把到达的客户端参数视为 sequential measurements，就可以用 Kalman filtering 思路逐个融合。
- 从 motivation 到 method 的逻辑链：
  1. 每个转辙机本地采集振动信号，本地训练诊断模型。
  2. 数据不上传，只上传模型参数。
  3. 模型参数越多，通信越重，越可能拥塞/丢包，因此先设计 DS-FCN 减少参数。
  4. 服务器按时间周期运行，一到参数就聚合，不等全部客户端。
  5. 把第 k 轮第 i 个客户端参数视为第 i 个 measurement。
  6. 用 SKF 递推更新全局参数估计和协方差。
  7. 当收到所有客户端或到达时间阈值，结束本轮并进入下一轮。
- 关键设计取舍：异步机制牺牲了“每轮所有客户端都参与”的整齐性，换来延迟环境下更少等待；SKF 比简单平均复杂，但能处理到达顺序和缺失更新。
- 为什么不是更直接 / 更简单的方案：
  - 单机训练数据少，容易欠拟合/过拟合。
  - 集中式训练要集中数据，不满足隐私和通信成本约束。
  - FedAvg 只做统计平均，不能很好处理延迟/丢包。
  - 同步 KF/FedSGD 仍依赖完整接收或固定轮次，面对包丢失不够灵活。
- 系统流程或算法步骤：
  1. 服务器初始化 DS-FCN 结构和参数。
  2. 将模型结构/参数发送给客户端。
  3. 客户端加载参数，在本地振动数据上训练。
  4. 客户端将模型参数上传给服务器，上传可能延迟或丢失。
  5. 服务器在 SAFL 周期内按参数到达顺序执行 SKF 聚合。
  6. 收到全部客户端或到达等待阈值后结束本轮。
  7. 新一轮用最新全局参数继续训练，客户端用最新模型诊断故障。
- 关键定义 / 公式 / 不变量：
  - 模型参数 `x=[W^T,b^T]^T`，把权重和偏置统一作为待估状态。
  - KF/SKF 中客户端上传参数 `z_i(k)` 被视作 measurement。
  - Kalman gain 动态调整每个 measurement 对全局参数估计的影响。
  - 最终全局参数为本周期最后一次顺序聚合后的 `x_hat^(N)(k|k)`。
- 实现细节：DS-FCN 使用双分支、宽第一层卷积核、FCN block；去除中间 pooling，减少 dense layer，仅保留 softmax 输出层。

## Evaluation

- 数据集：
  - ZDJ9 RPM 振动信号。
  - 16 种工况：正/反向动作下的正常、欠驱动、合理输出、不同过驱动、表示杆断裂、负载丢失、解锁失败等。
  - 每种工况 60 条，共 960 条。
  - 每次动作约 8 s，最大 10 s；裁剪/零填充为 10 s，即 5120 点。
  - 训练/测试比例 8:2；标签 one-hot。
  - 五个客户端模拟；结果为 30 次独立实验平均。
- Global model 实验：
  - 对比 SVM、RF、LeNet、AlexNet、RNN、LSTM、DS-WCNN、DS-FCN。
  - DS-FCN 准确率接近 DS-WCNN，但参数量从 38832 降到 28848。
  - 作者选择 DS-FCN 作为 FL 全局模型，因为参数少有助于降低通信成本和拥塞/丢包风险。
- Training strategy 实验：
  - 比较单设备训练、集中式训练、FedAvg、KF-based、SKF-based 等策略。
  - 单设备训练因数据少表现最差；集中式训练收敛最快。
  - SKF-based 在 AlexNet、DS-WCNN、DS-FCN 上分别达到 90.13%、95.96%、95.37%。
  - SKF 优于 KF，原因是它把每个客户端参数作为单独 measurement 顺序聚合，而不是简单一轮估计。
- Framework 实验：
  - 延迟实验：客户端等待时间在 0 到 Limit 秒均匀分布，Limit 取 10、20、40、60、80、120 s；聚合每个数据包约 15 s。
  - 结论：通信延迟越大，SKF-based 异步框架越有额外时间优势，因为它能边等边聚合；同步 KF 等收到全部参数后才开始聚合。
  - 丢包实验：丢包率设为 0、0.2、0.4、0.6。
  - 结论：SKF-based FL 在所有丢包率下诊断准确率最高。

## Key Artifacts

- 关键图：
  - Fig. 1：同步 FL 在延迟和丢包下的流程，展示等待问题。
  - Fig. 2：DS-FCN 结构，说明双分支多尺度特征提取。
  - Fig. 3：FCN block，说明去 pooling 和减少 dense 参数。
  - Fig. 5：SAFL 框架，展示 RPM 客户端、本地训练、服务器顺序聚合。
  - Fig. 7：反向动作下不同工况振动波形，说明故障在振动信号中有差异。
  - Fig. 9/10：不同训练策略准确率和训练曲线。
  - Fig. 11：延迟场景额外聚合时间。
  - Fig. 12：丢包率下诊断准确率。
- 关键表：
  - Table I：ZDJ9 RPM 的 16 种工况。
  - Table II：不同方法准确率，支撑 DS-FCN 的有效性。
- 关键公式 / 定义 / 算法：
  - Eq. (1)-(7)：KF 参数聚合形式。
  - Eq. (8)-(16)：SKF 顺序聚合和 SAFL 周期内全局参数更新。
  - 时间周期终止条件：收到所有客户端或达到预设等待阈值。
- 这些证据分别支撑哪些结论：DS-FCN 支撑轻量模型；SKF 公式支撑顺序异步聚合；延迟/丢包实验支撑不完善传输下的鲁棒性。

## Findings

- 发现 1：在 RPM 振动诊断中，DS-FCN 能在接近 DS-WCNN 准确率的同时减少约四分之一参数。
- 发现 2：单客户端训练不够好，说明分布式 RPM 数据协同确实有必要。
- 发现 3：SKF 聚合比 KF/FedAvg 更适合延迟和丢包，因为它能把到达顺序和缺失更新纳入递推估计。

## Strengths

- 论文最有说服力的地方：问题设定很贴近铁路现场通信：客户端分散、无线传输、延迟、丢包，而不是只在理想 FL 环境里比较准确率。
- 方法、数据、实验或问题设定的优势：同时从模型参数量、聚合策略、时间周期机制三个层面处理通信问题；实验覆盖模型、策略、框架三类验证。
- 相比已有工作的有效推进：把异步 FL 和 SKF 聚合引入 RPM 故障诊断，明确考虑 imperfect data transmission。

## Limitations

- 威胁模型、假设或适用范围的限制：没有考虑恶意客户端、模型投毒、隐私泄露、加密开销；客户端数据分布差异也没有被系统分析。
- 数据集、baseline、metric、ablation 或复现性的不足：数据/代码未公开；延迟和丢包是随机模拟，不是真实铁路通信链路；Table II 的具体数值在文本抽取中不完整，笔记只保留论文正文明确给出的关键数值。
- 在真实铁路场景中可能失效的条件：某些站点长期离线、客户端严重 non-IID、故障类别不一致、新故障未见过、网络丢包呈突发相关而非独立随机。

## My Takeaways

- 对小样本道岔/转辙机故障诊断的启发：如果未来你有多台转辙机或多个站场数据，SAFL 是一个很自然的协同学习方向。
- 可复用的方法：DS-FCN 作为轻量振动诊断模型；SKF 聚合作为丢包/延迟鲁棒的 federated aggregation；时间周期机制用于在线训练。
- 可能的后续问题：把 SAFL 和小样本/半监督结合；把客户端 non-IID、类不平衡、罕见故障引入实验；加入通信链路真实日志。

## Related Papers

- 前置阅读：Federated Averaging、asynchronous federated learning、Kalman filtering aggregation、RPM vibration fault diagnosis。
- 后续阅读：non-IID FL、robust FL under packet loss、privacy-preserving FL、federated semi-supervised fault diagnosis。
- 可对比论文：基于 DS-WCNN 的 ZDJ9 RPM 低采样振动诊断；SecureBoost 高铁转向架 FL；旋转机械动态验证 FL。
- 最接近的相关工作：已有 asynchronous FL fault diagnosis 方法考虑不同到达顺序，但未考虑 packet loss；本文加入时间周期和 SKF。
- 关键差异：本文不是只做模型准确率，而是把通信不完美作为核心约束来设计聚合机制。

## Open Questions

- SAFL 在强 non-IID 客户端数据下还能保持优势吗？
- 如果某些客户端长期不上传，SKF 的协方差和 Kalman gain 如何避免偏置？
- 能否把丢包模型从独立随机改成真实通信链路的突发丢包？
- DS-FCN 是否适合红外/电流等非振动模态？
- 能否把 SAFL 与半监督伪标签结合，解决每个站点标注样本不足？
