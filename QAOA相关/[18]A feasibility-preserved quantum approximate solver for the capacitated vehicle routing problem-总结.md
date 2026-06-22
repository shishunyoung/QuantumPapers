# [18] A Feasibility-Preserved Quantum Approximate Solver for the Capacitated Vehicle Routing Problem -- 论文总结

## 目录

- [1. 核心结论与创新点](#1-核心结论与创新点)
- [2. 内容总结](#2-内容总结)
  - [2.1 对QAOA在理论方面的创新](#21-对qaoa在理论方面的创新)
  - [2.2 对QAOA工程实现技术的实践](#22-对qaoa工程实现技术的实践)
  - [2.3 帮助QAOA落地的技术创新](#23-帮助qaoa落地的技术创新)
  - [2.4 对QAOA实际应用的扩展与探索](#24-对qaoa实际应用的扩展与探索)
- [3. 面临的困难、论文的不足之处和后续的改进方向](#3-面临的困难论文的不足之处和后续的改进方向)
- [4. 真机性能](#4-真机性能)

---

## 1. 核心结论与创新点

本文提出了一种面向容量约束车辆路径问题（CVRP）的可行性保持的量子近似求解器（Section I, Abstract）。核心创新包括：

1. **提出了一种新的 O(N^2) 二进制变量编码方案**（Section IV-A）：将 CVRP 的解编码为两部分——用置换矩阵 x 表示客户访问顺序，用无约束二进制串 y 表示是否在相邻时间步之间返回 depot。该编码天然满足 depot 约束（所有路径起点和终点均为 depot），且不需要预定义车辆数量 K。

2. **通过约束保持的混合操作实现可行性保持**（Section IV-E）：利用 Permutation Matrix Encoding 专用的 Partialized Mixer（来自 [6]）和 Grover Mixer（来自 [15]），将搜索空间严格限制在可行子空间内，而非沿用传统的惩罚项方法。

3. **条件解码机制绕过车辆容量约束**（Section IV-A, Algorithm 1）：将车辆容量约束从优化目标中移除，在解码阶段通过条件判断 （y_t = 1 或容量不足） 来完成子路径的划分。

4. **性能显著提升**（Section V-B）：与传统 QAOA + QUBO 编码相比，所提方法在可行性比率 （feasibility ratio） 、最优性比率 （optimality ratio） 和最优性差距 （optimality gap） 三个指标上均实现了数量级的提升。例如，P1 实例的最优性比率从 QUBO 的 0 提升到 0.597，可行性比率从 1.77e-5 提升到 1（Table I）。

---

## 2. 内容总结

### 2.1 对QAOA在理论方面的创新

**（1）将 AOA 框架应用于 CVRP 的理论构造（Section II, IV-A）**

论文系统阐述了 QAOA 的变体——Quantum Alternating Operator Ansatz （AOA） 的理论框架（Section II）。传统 QAOA 对于约束优化问题将约束转化为惩罚项加入目标函数，形成 QUBO/HOBO 形式（Equation (2)），导致能量景观存在大量局部极小值和贫瘠高原（barren plateaus），可行解采样概率极低。AOA 则要求搜索空间被限制在可行集 F 内（Equation (3)），混合操作 UM 满足 ⟨x'|HM|x⟩ = 0, x ∉ F ⊕ x' ∉ F（Equation (5)），初始态为所有可行态的等幅叠加（Equation (6)）。

**（2）新的 CVRP 二进制编码理论（Section IV-A）**

论文提出将 CVRP 的解表示为：
- 置换矩阵 x ∈ {0, 1}^{N×N}，满足每行每列恰好一个 1，可行空间 Fx 由 Equation (8a)-(8c) 定义；
- 无约束二进制串 y := y2y3...yN ∈ {0, 1}^{N-1}，yt = 1 表示在 t-1 和 t 时刻之间访问 depot。

该编码将 CVRP 重新概念化为：规划一辆可多次在 depot 补充容量的车辆的最短路径，客户按置换顺序依次服务。解码过程通过 Algorithm 1 实现条件性子路径划分。

**（3）条件编码的理论形式化（Section IV-C）**

由于直接推导 cost Hamiltonian 存在困难，论文引入 ancilla 比特 a 来表示每个时间步的 depot 访问条件（Equation (13)），将成本函数重写为（Equation (14a)-(14c)），使其能够被转换为 cost Hamiltonian HC，满足 ⟨x,a|HC|x,a⟩ = C(x,a)。

**（4）两类约束保持混合器的理论分析（Section IV-E）**

- **Partialized Mixer**（Equation (23)）：基于 [6] 提出的针对置换矩阵编码的混合器，通过行间交换保持置换矩阵的约束，实现为 Ring Mixer 形式 Hring（Equation (22)），对 x 寄存器保持约束，对 y 寄存器使用标准 X-mixer。
- **Grover Mixer**（Equation (24)）：基于 [15] 的 |F⟩⟨F| 形式，U_M^(GM)(β) = US(I - (1-e^{-iβ})|0⟩⟨0|^{⊗n})US^†，将搜索空间限制在 F，且具有相同目标值的态具有相同振幅。

### 2.2 对QAOA工程实现技术的实践

**（1）初始态制备电路（Section IV-B）**

- 置换矩阵 x：采用 [15] 的 Ux 操作（Equation (10)），逐寄存器构建等幅叠加，需要 O(N^3) 规模的电路。
- 决策向量 y：使用 Hadamard 门制备等幅叠加 |Fy⟩（Equation (11)）。
- 总制备操作 US = Ux ⊗ H^{⊗(N-1)}（Equation (12)）。

**（2）条件编码操作 UE 的电路实现（Section IV-D, Figure 4, Algorithm 2）**

引入四个 ancilla 寄存器：
- d：⌈log_2(Q + max(q) + 1)⌉ 个 qubit，追踪子路径中的已满足需求
- c：N 个 qubit，追踪已被服务的客户
- r：N(N-3) 个 qubit，用于恢复 c 寄存器
- a：N-1 个 qubit，存储 depot 访问条件

每个时间步包含四个操作过程：
1. **记录已满足需求**：通过 N 个 xt,i 控制的 [+qi] 门将当前客户需求累加到 d 寄存器（Equation (16)-(17)），每个 [+qi] 可分解为至多 O(log_2 Q) 个 [+1] 操作，每个 [+1] 需要 O(n) 个 Toffoli/CNOT/Pauli-X 门，总计 O(N(log_2 Q)^2) 门。d + qi 的范围限于 [min(q), Q + max(q)]。
2. **编码条件**：当 yt = 1 时通过 CNOT 翻转 at；当 d > Q 且 yt = 0 时通过 [>Q] 操作翻转 at（Equation (18), Figure 5）。[>Q] 分解为至多 K 个多控 Pauli-X 门，需要 O((log_2 Q)^2) 门。
3. **恢复寄存器并记录已服务客户**：当 at = |1⟩（表示新子路径开始）时，使用 ci 和 at 控制的 [-qi] 操作（Equation (19)）减去之前子路径的需求，将 d 恢复到仅包含当前需求。c 寄存器通过 Toffoli 门和辅助寄存器 rt 恢复到 |0⟩。恢复 d 需 O(N(log_2 Q)^2) 门，恢复 c 需 O(N) 门。
4. 最后时间步不需要步骤 3 中的恢复操作。

**（3）相位分离操作（Section IV-C）**

相位分离操作为 UP(γ) := UE^† e^{-iγ HC} UE（Equation (15)），其中 e^{-iγ HC} 可分解为 O(N^2) 个 CNOT 和 Z 轴旋转门。整个电路结构如 Figure 6 所示。

**（4）资源评估（Section IV-F）**

- 总 qubit 数：2N^2 - ⌈log_2(Q + max(q) + 1)⌉ - 2（Equation (25)）
- 量子寄存器共 6 组：x, y, a, d, c, r
- 单层（depth-p）相位分离：O(N^2(log_2 Q)^2) 个基本门
- 单层混合操作：O(N^3) 个基本门
- 总电路规模：O(p N^2(N + (log_2 Q)^2)) 个基本门

### 2.3 帮助QAOA落地的技术创新

**（1）可行性保持策略（贯穿 Section IV）**

论文的核心工程贡献在于将 AOA 应用于 CVRP 的完整流水线：
- 通过编码设计使 depot 约束天然满足；
- 通过约束保持混合器确保客户访问约束（每客户恰好访问一次）；
- 通过条件解码（Algorithm 1）在解读取阶段处理车辆容量约束，避免将其编码为惩罚项。

这种方法论具有泛化价值：对于类似的约束组合优化问题（如 Knapsack Problem），可以通过问题重新编码，将部分约束转移到解码阶段，而使量子部分仅处理能自然保持的约束（Section VI）。

**（2）与传统 QAOA+QUBO 的实证对比（Section V-B, Figure 10, Table I）**

实验表明，传统 QAOA+QUBO 方法在 CVRP 上的可行性比率极低（P1: 1.77e-5, P2: 1.78e-5, P3s: 5.76e-4），最优性比率几乎为 0。而所提方法的可行性比率始终为 1（100% 可行），最优性比率和最优性差距均有数量级的改善。此外，QUBO 编码需要预定义车辆数 K = ⌈∑q_i / Q⌉，可能导致搜索空间无法覆盖最优解（如 P1 实例）。

**（3）深度与性能的关系验证（Section V-B, Figure 9）**

在 48 个随机生成的 3 客户问题（P3s）上验证了 depth-1 至 depth-9 电路，随深度增加最优性差距趋近于 0，最优性比率持续上升。

**（4）状态向量模拟器实验环境（Section V-A）**

使用 Qiskit 的 statevector simulator 进行无噪模拟，采用 COBYLA 优化器调参，使用三个评估指标：optimality gap α（Equation (26)）、optimality ratio r_opt（Equation (27)）、feasibility ratio r_feas（Equation (28)）。

### 2.4 对QAOA实际应用的扩展与探索

论文将 AOA 框架的应用范围从已有的无约束问题（Max-Cut）、排列问题（TSP）扩展到 CVRP（Section I, VI）。作为 NPO 难题，CVRP 在交通和物流领域具有广泛实际应用。论文指出所提出的编码和条件处理方法"具有扩展到其他约束组合优化问题的潜力，例如 Knapsack Problems，因为它们具有类似的问题结构"（Section VI）。此外，论文不预设车辆数量 K，相比已有 QUBO 编码方案（需预设 K）具有更大的搜索空间覆盖能力（Section III, IV-A）。

---

## 3. 面临的困难、论文的不足之处和后续的改进方向

**（1）大量多控 Toffoli (MCT) 门（Section VI）**

论文明确指出所设计的电路虽然门数多项式级别，但包含大量 MCT 门。MCT 门对拓扑量子设备不友好，需要额外的 SWAP 门来适配硬件拓扑。

**（2）对量子噪声的脆弱性（Section VI）**

实际量子设备中的比特翻转噪声（bit-flip noise）很容易破坏混合器维持的可行空间。因此，该方法是否适合在近期 NISQ 设备上运行尚存疑问。

**（3）仿真规模受限（Section V-A）**

受限于仿真环境，只能构建简单的问题实例（4 客户 P1/P2，3 客户 P3s），难以在更大规模问题上验证方法有效性。

**（4）深度限制下的次优解（Section V-B）**

在 P2 实例中，depth-1 电路的最优性比率仅为 0.241，虽然最优编码概率最高，但低于次优峰值的概率之和（因为最优编码数为 14，而次优编码数为 23）。需要 depth-2 电路才能达到 0.43 的最优性比率。

**（5）后续改进方向（Section VI）**

- 进一步优化电路设计以减少 MCT 门数量
- 增强对量子噪声的鲁棒性，以实现在 NISQ 设备上的实际部署

---

## 4. 真机性能

**本文未在真实量子计算机上进行实验。** 所有实验均基于 Qiskit 的 statevector simulator（Section V-A），即在无噪理想条件下进行量子电路模拟。论文未提供任何真实量子硬件上的运行数据、错误率或保真度指标。

论文在 Section VI 的讨论中明确表示，该方法是否适合在近期 NISQ 设备上运行尚存疑问，真机部署需要进一步优化和噪声鲁棒性改进。
