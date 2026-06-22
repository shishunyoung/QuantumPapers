# [27] Implementing the Quantum Approximate Optimization Algorithms for QUBO problems Across Quantum Hardware Platforms: Performance Analysis, Challenges, and Strategies -- 论文总结

## 目录

1. [核心结论与创新点](#核心结论与创新点)
2. [内容总结](#内容总结)
   - [对QAOA在理论方面的创新](#对qaoa在理论方面的创新)
   - [对QAOA工程实现技术的实践](#对qaoa工程实现技术的实践)
   - [帮助QAOA落地的技术创新](#帮助qaoa落地的技术创新)
   - [对QAOA实际应用的扩展与探索](#对qaoa实际应用的扩展与探索)
   - [QAOA对其他领域的启发式贡献](#qaoa对其他领域的启发式贡献)
3. [面临的困难、论文的不足之处和后续的改进方向](#面临的困难论文的不足之处和后续的改进方向)
4. [真机性能](#真机性能)

---

## 核心结论与创新点

1. **ADAPT-QAOA 在困难问题上显著优于标准 QAOA**（Section 4.1, Section 5）：当特征选择问题的 trade-off 参数 $\alpha = 0.6$（问题更困难）时，ADAPT-QAOA 在平均近似比和时间-to-solution 两个指标上均显著优于标准 QAOA。具体而言，15 层 ADAPT-QAOA 的表现即优于 30 层标准 QAOA。而在 $\alpha = 0.2$（较简单问题）下，两者性能相近。

2. **首次将量子硬件校准数据与真实拓扑结构纳入 QAOA 可扩展性分析**（Section 4.3, Section 4.4）：论文利用 IBM Brisbane（超导）和 Quantinuum H2（离子阱）的真实设备校准数据（门持续时间、门错误率和测量错误率），结合设备实际拓扑，对标准 QAOA 的实际运行时间和总错误概率进行了理论估算，系统性揭示了不同硬件平台之间的性能权衡。

3. **在金融特征选择问题上展开了 QAOA 的实用性研究**（Section 1, Section 2）：论文以金融领域的特征选择问题（feature selection）作为 QUBO 建模的支柱，给出了从 QUBO 建模到 Ising 哈密顿量映射的完整流程（包括公式 \tag{2}, \tag{3}, \tag{4}），为 QAOA 在金融领域的潜在应用提供了方法论参考。

4. **提出超导与离子阱硬件的性能互补性结论**（Section 4.3, Section 4.4, Section 5）：超导量子计算机（IBM Brisbane）因门操作速度快（纳秒级）在时间-to-solution 上占优；而离子阱设备（Quantinuum H2）因门错误率和测量错误率更低，在总错误概率上表现更佳。两者形成互补格局。

5. **量化了量子比特拓扑连通性对 QAOA 性能的影响**（Section 4.3）：论文比较了 heavy-hex、正方形晶格和全连接（all-to-all）三种拓扑，发现在同一硬件平台下，拓扑连通度越高（越接近全连接），所需 SWAP 门越少，时间-to-solution 越短，总错误概率越低，为硬件设计方向提供了数据支撑。

---

## 内容总结

### 对QAOA在理论方面的创新

**(a) 标准 QAOA 的迭代实现方式**（Section 3.1）

论文采用逐层添加的迭代方式实现标准 QAOA——每添加一层 QAOA 层后，优化所有变分参数，优化后的参数作为下一层的初始值。这种方式类似于 ADAPT-QAOA 的逐层构造策略 [11]，与一次性设置全层架构的方案不同。公式 \tag{7} 给出了 Hadamard 初始化（等权叠加态）：

$$
|\psi_0\rangle = \left[\hat{H}|0\rangle\right]^{\otimes n} = \left(\frac{|0\rangle + |1\rangle}{\sqrt{2}}\right)^{\otimes n}. \tag{7}
$$

标准 QAOA 的代价层与混合层定义见公式 \tag{5}，其中混合哈密顿量固定为所有量子比特上的 Pauli-X 旋转：

$$
\hat{H}_M = \sum_{i=1}^{n} \hat{X}_i. \tag{6}
$$

**(b) ADAPT-QAOA 中混合器池的完整设计**（Section 3.2）

论文构建了一个三部分混合器池：
- $P_{XY}$：全量子比特的 X-混合器和 Y-混合器（2 个）
- $P_{\text{single}}$：单量子比特混合器集合 $\{\hat{X}_i, \hat{Y}_i\}$（$2n$ 个）
- $P_{\text{two}}$：双量子比特混合器 $\{B_iC_j \mid B,C \in \{\hat{X},\hat{Y},\hat{Z}\}\}$（共 $9n(n-1)/2$ 个）

混合器池总规模为 $2 + 2n + \frac{9n(n-1)}{2}$，量级为 $O(n^2)$。

混合器选择依据最大梯度准则，梯度计算公式为：

$$
g_l = \left| -i \left\langle \psi^{(k-1)} \left| e^{i\hat{H}_C\gamma_0} \left[\hat{H}_C, \hat{A}_l\right] e^{-i\hat{H}_C\gamma_0} \right| \psi^{(k-1)} \right\rangle \right|, \tag{11}
$$

其中 $\gamma_0 \approx 0.01$ 为小量。从混合器池中选取梯度最大的混合器作为当前层的混合哈密顿量。

**(c) 特征选择问题的 QUBO 到 Ising 哈密顿量映射**（Section 2, Section 3）

特征选择问题的目标函数为：

$$
f_{\text{FS}}(\bar{x}) = -(1-\alpha)\sum_i Q_{ii}x_i + \alpha\sum_{i<j} Q_{ij}x_i x_j, \tag{2}
$$

其中 $\alpha \in [0,1]$ 控制线性项与二次项之间的权衡。通过 $x_i \to \frac{1-\hat{Z}_i}{2}$ 的代换，得到代价哈密顿量：

$$
\hat{H}_C = -(1-\alpha)\sum_i Q_{ii}\frac{1-\hat{Z}_i}{2} + \alpha\sum_{i<j} Q_{ij}\frac{1-\hat{Z}_i}{2}\frac{1-\hat{Z}_j}{2}. \tag{4}
$$

这一映射流程完整覆盖了从应用问题到量子算法可执行形式的全过程。

### 对QAOA工程实现技术的实践

**(a) 实验设置的具体工程参数**（Section 4）

论文采用 Qiskit 理想量子模拟器，每优化迭代 104 shots，使用 SciPy 的 Powell 方法进行经典优化，每层最大优化迭代次数限制为 1500 次。对每种问题规模，使用 10 个不同随机种子生成 QUBO 矩阵数据，所有实验中始终保持相同种子集合，保证实验的可重复性。

**(b) 近似比与时间-to-solution 双指标评估体系**（Section 3.1, Section 4.1）

- 近似比定义：
$$
r_k = \frac{C_k}{C_{\text{exact}}} \tag{10}
$$
其中 $C_k$ 是第 $k$ 层结束时的期望值，$C_{\text{exact}}$ 通过经典求解器 Gurobi OptiMods 获得。
- 时间-to-solution 考虑算法完整运行时间，包括混合器选择时间（ADAPT-QAOA）和量子电路模拟时间。

**(c) 门层次分组与电路深度分析**（Section 4.3）

在估算真实硬件的运行时间时，论文将量子电路中的门按照"同一量子比特上至多一个门"的原则分组为门层（gate layers），确保同层门可并行执行。单层执行时间估算公式为：

$$
T_{\text{layer}} = (d_1 t_1 + d_2 t_2) N S, \tag{13}
$$

其中 $d_1$、$d_2$ 分别为单/双量子比特门层数，$t_1$、$t_2$ 为对应门持续时间，$N$ 为优化迭代数，$S$ 为每迭代的 shots 数。

**(d) Z 门的虚拟实现优化**（Section 4.3）

论文指出，在超导（IBM）和离子阱（Quantinuum）平台上，Z 门均可通过改变控制电子学的参考框架（reference frame）来实现——即改变后续所有作用于该量子比特上的门的相位，从而 Z 门"实际上不消耗时间且不引入额外错误"。这一工程细节在估算中被纳入考虑。

### 帮助QAOA落地的技术创新

**(a) 基于真实校准数据的硬件性能预测方法**（Section 4.3, Section 4.4）

论文提出了一套从实际设备校准数据出发、计算 QAOA 在真实硬件上理论性能的方法论：
- 从 IBM Quantum 获取 ibm_brisbane 的快照校准数据（2025 年 8 月 26 日）
- 从 Quantinuum 官方文档 [18, 20] 获取 H2 设备的门持续时间和错误率数据
- 结合 QAOA 电路中的门计数和电路深度，按公式 \tag{13} 估算运行时间，按公式 \tag{14} 估算总错误概率

**(b) 总错误概率的估算模型**（Section 4.4）

总错误概率公式为：

$$
E_{\text{tot}} = 1 - \left[(1 - e_1)^{N_1} (1 - e_2)^{N_2} (1 - e_m)^{N_m}\right], \tag{14}
$$

其中 $e_1$、$e_2$、$e_m$ 分别为单量子比特门、双量子比特门和测量的错误概率，$N_1$、$N_2$、$N_M$ 为电路中对应操作的计数。论文使用 1 层标准 QAOA 估算错误（而非 30 层），因为未考虑错误缓解方法，30 层会放大量级但结论趋势一致，1 层的估算结果应视为错误的上界。

**(c) 不同硬件拓扑的理论扩展分析**（Section 4.3）

除了真实设备拓扑（ibm_brisbane 的 heavy-hex，Quantinuum H2 的全连接），论文还分析了 ibm_brisbane 在正方形晶格和全连接两种理论拓扑下的性能，揭示了拓扑连通度对 QAOA 性能的重要性：更密集的拓扑意味着更少的 SWAP 操作，从而更短的运行时间和更低的错误概率。

### 对QAOA实际应用的扩展与探索

**(a) 金融特征选择问题作为QAOA应用场景**（Section 1, Section 2）

论文明确指出，在金融领域的贷款风险评估中，特征选择问题具有直接应用价值——从大量候选变量中选出最能预测违约风险的子集。该问题被建模为 QUBO，并通过量子算法求解。这一场景的选择具有实际业务动机（OP Financial Group 的参与也佐证了这一点）。

**(b) 不同问题规模和硬度的 QAOA 性能差异分析**（Section 4.1）

论文系统比较了 6、10、14 个特征（qubits）在 $\alpha = 0.2$ 和 $\alpha = 0.6$ 下的性能：
- $\alpha = 0.2$（偏重线性项，问题较简单）：QAOA 与 ADAPT-QAOA 的近似比相近
- $\alpha = 0.6$（偏重二次项，问题更组合化、更困难）：ADAPT-QAOA 显著优于 QAOA
- 在每种问题规模和两个 $\alpha$ 值下，ADAPT-QAOA 至少有一个实例收敛至与精确解非常接近的近似比

**(c) 量子算法与经典求解器的对比**（Section 4.2）

论文使用经典 QUBO 求解器 Gurobi OptiMods 对同样的问题进行求解，最优性间隙（optimality gap）设置为 $10^{-4}$：

$$
\text{gap} = \frac{|z_{\text{bound}} - z_{\text{best}}|}{|z_{\text{best}}|}. \tag{12}
$$

结果表明，经典求解器在时间-to-solution 的可扩展性方面远优于当前的标准 QAOA 和 ADAPT-QAOA。这一发现为未来研究方向提供了重要参照——需要寻找使经典求解器更困难的 QUBO 问题模式（如加入惩罚项），同时这类问题仍需在量子算法能力范围内可解。

**(d) 超导与离子阱硬件的适用性对比**（Section 4.3, Section 4.4）

两种硬件平台的性能呈现明确的互补特征：
- **超导**（ibm_brisbane）：门持续时间在纳秒量级（$t_1 = 0.06\ \mu\text{s}$, $t_2 = 0.66\ \mu\text{s}$），时间-to-solution 更短；但门错误率较高（$e_1 = 3.3 \times 10^{-4}$, $e_2 = 1.2 \times 10^{-2}$）
- **离子阱**（Quantinuum H2）：门持续时间在微秒量级（$t_1 = 63\ \mu\text{s}$, $t_2 = 308\ \mu\text{s}$），时间-to-solution 更长；但门错误率更低（$e_1 = 3.0 \times 10^{-5}$, $e_2 = 1.0 \times 10^{-3}$）

这为不同应用场景的硬件选择提供了决策依据。

### QAOA对其他领域的启发式贡献

本论文的研究框架（即同时评估近似比和时间-to-solution、结合真实硬件校准数据估算算法实际性能）可以推广到其他变分量子算法在其他应用场景（如材料科学中的分子基态能量计算、物流中的车辆路径问题等）的评估中。论文为量子算法的实用性评估提供了一套可复用的方法论模板（Section 4, Section 5）。

---

## 面临的困难、论文的不足之处和后续的改进方向

### 论文指出的困难

1. **ADAPT-QAOA 的混合器选择时间开销随问题规模快速增长**（Section 4.1）：混合器池规模为 $O(n^2)$，每层都需遍历计算梯度并选取最优混合器，导致随问题规模的增大，ADAPT-QAOA 的时间-to-solution 显著高于标准 QAOA。这种开销在规模更大时可能更为严重。

2. **当前量子算法在时间-to-solution 方面仍不及经典求解器**（Section 4.2）：Gurobi OptiMods 在求解所考虑的特征选择问题时展现了更好的时间可扩展性，量子算法在实用性上仍存在差距。

3. **离子阱设备门速度慢导致的运行时间劣势**（Section 4.3）：Quantinuum H2 因需要额外的量子比特输运时间（intrazone/interzone shift），门持续时间比超导设备高 1~2 个数量级，导致时间-to-solution 在各类问题规模下均为最长。此外，H2 目前仅有 56 个量子比特，在超导设备大拓扑面前难以靠增加量子比特数量弥补时间差距。

4. **拓扑连通性不足导致额外 SWAP 开销**（Section 4.3）：heavy-hex 拓扑平均连接度仅 2.3，需大量额外 SWAP 门，增大了总门数、总时间和总错误概率。

### 论文的不足之处

1. **错误估算未纳入错误缓解方法的影响**（Section 4.4）：论文承认，由于未考虑 zero-noise extrapolation [24] 和 probabilistic error cancellation [25] 等错误缓解技术，所估算的总错误概率应被视为上界。这意味着实验结论在实际部署中（通常会启用错误缓解）可能会更乐观。

2. **均基于理想量子模拟器，未在真实硬件上执行实验**（Section 4）：所有 QAOA 和 ADAPT-QAOA 的算法性能测试均在 Qiskit 理想模拟器上完成，真实硬件部分仅涉及基于校准数据的理论估算（时间、错误概率），并无真机运行的实际实验数据。

3. **问题规模有限**（Section 4.1）：考虑的最大问题规模仅为 14 个量子比特（14 个特征），对于评估算法的实际可扩展性而言范围较窄。

4. **特征选择问题仅使用随机生成的 Q 矩阵**（Section 2）：虽以金融领域场景为动机，但实验中使用的 QUBO 矩阵均为随机生成，未使用真实的金融数据，与实际金融应用之间仍存在差距。

5. **仅测试了无约束的简单特征选择形式**（Section 2, Section 5）：论文指出实验只涵盖最简单的特征选择场景，未加入约束条件（如惩罚项），而真实金融应用往往需要各类约束。

### 后续改进方向（Section 5）

1. **探索加入惩罚项（constraints）的 QUBO 目标函数**，使得问题对经典求解器更具挑战性，同时仍保持在量子算法的可解范围内。
2. **探索更高效的量子算法**（如 QAOA 的其他变体），并评估其对不同类型门错误和量子比特错误的敏感性（susceptibility）。
3. **进一步提升量子硬件的量子比特连通度和门/测量错误率**，以增强量子计算的整体保真度。

---

## 真机性能

> 注意：本论文并未在真实量子计算机上运行 QAOA 或 ADAPT-QAOA 的实际实验。所有"真机性能"数据均为基于真实设备校准数据（calibration data）的理论估算值，而非实际运行结果。

### 表 1：真机校准数据（Section 4.3, Table 1）

| 参数 | ibm_brisbane (超导) | Quantinuum H2 (离子阱) |
|---|---|---|
| 单量子比特门持续时间 $t_1$ | 0.06 μs | 63 μs |
| 双量子比特门持续时间 $t_2$ | 0.66 μs | 308 μs |
| 单量子比特门错误率 $e_1$ | $3.3 \times 10^{-4}$ | $3.0 \times 10^{-5}$ |
| 双量子比特门错误率 $e_2$ | $1.2 \times 10^{-2}$ | $1.0 \times 10^{-3}$ |
| 测量错误率 $e_m$ | $3.7 \times 10^{-2}$ | $1.0 \times 10^{-3}$ |

数据来源：ibm_brisbane 数据来自 IBM Quantum 服务 2025 年 8 月 26 日快照 [19]；Quantinuum H2 数据来自参考文献 [18, 20]。

### 估算的时间-to-solution（Section 4.3, Fig. 6）

使用 30 层标准 QAOA、$\alpha = 0.2$，基于公式 \tag{13} 估算，不同硬件/拓扑配置下的性能排序（从快到慢）：

1. ibm_brisbane 理论全连接拓扑 —— 最快
2. ibm_brisbane 理论正方形晶格拓扑
3. ibm_brisbane 真实 heavy-hex 拓扑
4. Quantinuum H2 真实全连接拓扑 —— 最慢

原因：Quantinuum H2 的门持续时间（微秒级）比 ibm_brisbane（纳秒级）高 1~2 个数量级；且 heavy-hex 拓扑因连通度最低需要最多的 SWAP 门。随着问题规模增大，不同拓扑间的运行时间差距进一步扩大。

### 估算的总错误概率（Section 4.4, Fig. 7）

使用 1 层标准 QAOA 求解 6 特征的问题，基于公式 \tag{14} 估算：

| 设备 | 拓扑 | 估算总错误概率 |
|---|---|---|
| ibm_brisbane | heavy-hex | 约 50-60%（Fig. 7 中最高） |
| ibm_brisbane | 正方形晶格 (理论) | 居中 |
| ibm_brisbane | 全连接 (理论) | 居中偏低 |
| Quantinuum H2 | 全连接 | 约 10-15%（Fig. 7 中最低） |

Quantinuum H2 因其显著更低的门错误率和测量错误率，提供了最低的估算总错误概率。论文强调，这些估算值应视为上界，因为实际使用中通常会采用错误缓解方法。此估算使用 1 层而非 30 层，原因在于论文未纳入错误缓解的考量，30 层会放大错误但结论趋势一致。

### 真机性能的核心结论

超导量子计算机在速度上占优（适合需要快速求解的场景），离子阱量子计算机在精度上占优（适合对解质量要求高的场景），两者在当前的含噪声中等规模量子（NISQ）时代形成互补。

---

**论文信息**：Teemu Pihkakoski, Aravind Plathanam Babu, Pauli Taipale, Petri Liimatta, Matti Silveri. "Implementing the Quantum Approximate Optimization Algorithms for QUBO problems Across Quantum Hardware Platforms: Performance Analysis, Challenges, and Strategies." arXiv:2510.12336v1 [quant-ph], 14 Oct 2025.
