# [19] Quantum Optimization Methods for the Generalized Traveling Salesman Problem -- 论文总结

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

本文提出了针对广义旅行商问题（Generalized Traveling Salesman Problem, GTSP）的量子优化基线方案，主要贡献包括（Section I, pp.1--2）：

1. **提出了一种新的GTSP QUBO编码方案**（称为Q-GTSP），具有专门针对产出可行解的二次惩罚项，可直接用于量子退火。（贡献 i, Section IV-A）
2. **提出了基于门的约束QAOA管道**（称为C-QAOA），采用保持one-hot约束的XY-mixer，将逐步汉明权重约束在理想电路模型中，其余约束通过惩罚项在后处理中显式追踪。（贡献 i, Section IV-B）
3. **扩展了NN2C预处理方法**以处理GTSPLIB中的非对称实例，将实例规模缩减至NISQ设备可处理的范围。（贡献 iii, Section IV-C）
4. **在GTSPLIB基准数据集上全面评估**：在小型实例（3--5节点）上，两种量子方法均可频繁找到最优解或近似最优解，与经典state-of-the-art求解器（GLNS/Gurobi）性能可比；但实例规模超过~15节点和4个簇后，有效解不再可靠出现。（Section VI, VIII）
5. **量子退火方法（Q-GTSP）在可行解率上始终优于C-QAOA**，这很可能是由于本工作中QAOA仅使用了p=1的极浅层深度所致。（Section VI-D）

---

## 内容总结

### 对QAOA在理论方面的创新

- **约束混合算子的应用**（Section IV-B）：传统QAOA使用横场混合算子 $U_M(\beta) = e^{-i\beta H_M}$，其中 $H_M = -\sum_{i=1}^n \sigma_i^x$（见公式(5)）。本文采用了约束保持混合算子（constraint-preserving mixer），具体为XY-mixer：

  $$H_M = \sum_t \sum_{(u,v) \in E_t} (X_u X_v + Y_u Y_v) \tag{11}$$

  其中 $E_t$ 表示分区 $P_t$ 内的边，该混合算子将量子演化限制在可行子空间，确保逐步one-hot约束在理想电路模型中得到满足。（Section II-D-3）

- **QUBO编码的代价函数设计**（Section IV-A-1）：目标函数使用一步one-hot编码（$N \cdot K$ 个二元决策变量），代价函数为

  $$E_{\text{cost}}(x) = \sum_{c=0}^{K-1} \sum_{i=0}^{N-1} \sum_{j=0}^{N-1} w_{ij} x_{c,i} x_{(c+1) \bmod K, j} \tag{6}$$

  约束包括：每步恰好访问一个节点的惩罚项 $p_0(x)$（公式(7)）、每个簇恰好一个节点的惩罚项 $p_1(x)$（公式(8)），以及不存在边转移的惩罚项 $p_2(x)$（公式(9)）。

- **惩罚权重的严格上界定义**（Section IV-A-2）：通过将所有边权重按非递增排序 $w_1 \ge w_2 \ge \dots \ge w_{|E|}$，定义

  $$\lambda := \sum_{k=1}^{K} w_k + 1 \tag{10}$$

  作为任何有效解代价的平凡上界，确保最便宜的无效解在能量景观中一定比最贵的有效解代价更高（Section IV-A-2）。

### 对QAOA工程实现技术的实践

- **参数优化采用网格搜索而非梯度优化**（Section V-A-b）：为追求NISQ友好的方案，在 $\gamma \in [0.05, \pi]$ 和 $\beta \in [0.05, \pi/2]$ 范围内使用 $10 \times 10$ 网格搜索，每次电路评估使用1500 shots，batch size为1。这避免了在量子硬件上计算精确梯度的高昂开销。
- **初始态制备的简化**（Section IV-B）：不是从所有可行态的均匀叠加 $|+\rangle^{\otimes n}$ 开始，而是从单个随机选择的可行one-hot计算基态初始化，以最小化硬件需求。
- **只使用单层QAOA（$p = 1$）**（Section IV-B）：为最小化电路深度，采用了极浅层设计。p=1被论文明确标识为限制大实例求解质量的关键因素（Section VI-D, VI-C）。
- **XY-mixer使用低深度环拓扑**（Section IV-B）：在每个分区内应用XY-mixer时采用low-depth ring topology，以最小化所需电路深度。
- **后处理中的可行性解码**（Section I）：在C-QAOA中，仅one-hot约束（$p_0$）由XY-mixer保证，其余两个约束（$p_1$ 和 $p_2$）仍通过惩罚项处理，可行性在后处理中显式追踪。
- **D-Wave量子退火使用默认参数**（Section V-A-a）：除`num_reads`设为1500以确保充分采样外，其余全部使用D-Wave默认参数。Minor embedding由`EmbeddingComposite`自动处理。

### 帮助QAOA落地的技术创新

- **集群子集采样策略（Cluster Subset Sampling）**（Section V-D）：从大型GTSPLIB实例中通过单阶段集群采样随机选取整个集群，直到近似达到目标节点数，从而生成量子硬件可处理的小型GTSP子实例。该方法保留了GTSP的集群结构特征（每簇节点数、集群间成对连接等）。
- **扩展的NN2C预处理算法**（Section IV-C）：原始NN2C方法[26]仅适用于对称GTSP实例。本文扩展版本对每个集群显式保留至少一个最佳入口节点（best entry node, 最小化来自其他集群的入边距离）和一个最佳出口节点（best exit node, 最小化到其他集群的出边距离）。在对称平局时，使用相反方向的平均代价作为次级选择标准。该预处理可将多达100节点的原始实例缩减至20节点以内（Table IV），使原本无法在量子硬件上运行的GTSPLIB实例得以求解。
- **将GTSPLIB实例缩减至NISQ可处理规模**（Section V-C, V-D）：由于量子求解器最多只能处理25个节点，论文生成了四组实验数据：Subsample Small（3--5节点, Table I）、Subsample Medium（3--20节点, Table II）、Preprocess Small（原始14--22节点缩减至3--5节点, Table III）、Preprocess Medium（原始14--100节点缩减至3--20节点, Table IV）。

### 对QAOA实际应用的扩展与探索

- **首次将QAOA端到端地应用于GTSP**（Section I, VIII）：尽管QAOA/TSP的研究已有相当积累[9]--[11]，但针对GTSP的量子专属公式和基准测试在文献中几乎是空白。本文填补了这一缺口。
- **GTSP在工业场景的建模能力阐述**（Section I）：GTSP广泛应用于柔性制造调度[2]、自动化存取系统[3]、仓库订单拣选（含多库存位置）[4]、机器人工具路径导航与运动规划（如焊接、钻孔、切割等子任务的执行变体选择）[5], [6]等场景。本文的量子基线方案为这些工业问题向量子计算迁移提供了起点。
- **量子退火与门模型两个范式的直接对比**（Section VI, VII）：在同一GTSPLIB基准上对Q-GTSP（D-Wave量子退火）和C-QAOA（基于Qiskit Aer模拟器，目标部署在IBM Heron处理器系列上）进行了系统比较，分析了两种范式的可行解率、近似比、采样稳定性和扩展性差异。
- **与经典最优求解器的验证对比**（Section V-E）：使用GLNS（自适应大邻域搜索算法）结合Gurobi作为经典基线，在测试规模下GLNS总能找到全局最优解。这为量子求解器的性能提供了可靠的参照基准。

### QAOA对其他领域的启发式贡献

- **约束混合算子的可行性追踪方法**（Section IV-B, I）：本文明确区分了"在理想电路模型中被混合算子保证的约束"与"仅通过惩罚项在后处理中追踪的约束"，这种分层约束处理策略（部分约束硬件级保证，部分约束后处理级检查）可以为其他复杂组合优化问题的量子算法设计提供参考。
- **系统化的NISQ实用性评估框架**（Section V, VI）：通过四组不同来源/规模的实验数据集（Subsample + Preprocess, Small + Medium），系统地评估了量子求解器在可行性、近似比、采样稳定性、嵌入能力等多个维度上的表现，形成了一个可复现的量子优化基准评估框架。（贡献 ii）
- **实例规模与可行解率的退化关系**（Section VI-D, VII）：揭示了核心瓶颈不在可行解的质量，而在于给定采样预算内观察到可行巡回的概率。这一发现提示未来量子求解器应优先解决不可行性问题，而非优化可行解的质量。

---

## 面临的困难、论文的不足之处和后续的改进方向

### 面临的困难与论文不足之处

1. **可行解率急剧下降**（Section VI, VII）：当实例规模增长到5--7节点以上时，C-QAOA的可行解率急剧下降，在13--15节点时降至1%以下；Q-GTSP在15节点以上也出现类似的急剧退化。Table V显示C-QAOA在10节点实例上已报告"Invalid tour"，而Q-GTSP在18节点以上也出现"Invalid tour"或嵌入失败。
2. **QAOA仅使用p=1层**（Section VI-C, VI-D）：论文明确指出这是限制大实例求解质量的一个关键因素。p=1的浅层QAOA与量子退火的标准退火时间形成对比，后者是Q-GTSP在可行解率上表现更优的可能原因。
3. **C-QAOA在真实量子硬件上未完成完整评估**（Section V-B）：由于IBM量子硬件上可用的QPU时间不足以进行包含参数优化的完整实验评估，QAOA实验仅在Qiskit Aer模拟器上运行。论文在IBM QPU上只进行了初步测试确认了算法的硬件兼容性，完整QPU评估留待未来工作。
4. **预处理可能导致最优解丢失**（Section IV-C, Section VI-E）：论文明确指出，NN2C扩展预处理不能保证原始GTSPLIB实例的最优巡回在缩减实例中得以保留。此外，C-QAOA不仅未从预处理中获益，甚至可能受到负面影响——Preprocess Small数据集中5节点的实例全部超时失败，而Subsample Small中同等维度的实例偶尔能成功完成（Section VI-E, Table VIII vs Table VII）。
5. **QUBO惩罚项方法不可扩展**（Section VII）：论文直接指出，"通过惩罚项在QUBO方案中处理约束的选项在实践中不可扩展"（the option to address constraints via penalty terms in a QUBO-based approach does not scale in practice）。两个量子求解器在超过~15节点后都未能可靠找到可行解。
6. **采样预算不足**（Section VI-C）：1500次采样虽然足以覆盖极小型问题的完整搜索空间，但在问题规模增大时很快不足以覆盖搜索空间，导致可行解发现概率极低。
7. **嵌入失败**（Section VII, Table VI）：在Preprocess Medium数据集上，Q-GTSP遇到嵌入失败（"Could not embed"），当所需逻辑量子比特数增长至多达400时，远超当前D-Wave Advantage2硬件容量。
8. **XY-mixer仅处理一个约束**（Section IV-B）：C-QAOA的XY-mixer仅确保逐步one-hot约束（$p_0$）的满足，而集群约束（$p_1$）和不存在边约束（$p_2$）仍需通过惩罚项处理，因为没有已知的短电路深度混合算子结构可以同时处理这三个约束的组合。

### 后续改进方向

论文在Section VIII中明确提出了以下未来工作方向：

1. **通过适当的约束处理提升可扩展性**（proper constraint handling）：基于本文的发现，未来量子求解器应将不可行性视为首要优先级问题。基于门的方案（如QAOA中的约束混合算子）被认为最有希望从根本上解决这一瓶颈。
2. **增强的预处理技术**（enhanced preprocessing techniques）：开发更好的实例缩减方法，以更有效地利用有限量子资源。
3. **更高效的编码方案**（more efficient encodings）：探索可减少量子比特使用量的替代编码方案。
4. **利用GTSP集群结构的问题特定嵌入**（problem-specific embeddings）：设计与GTSP集群结构特征相适应的专用嵌入方案。
5. **增加退火时间与QAOA层深度的实验**（Section VI-A）：特别应研究量子求解器在增加退火时间（对Q-GTSP）和增加QAOA层数p（对C-QAOA）条件下的time-to-solution扩展性能。
6. **完整QPU评估**（Section V-B）：在真实IBM Heron系列量子处理器上进行完整的QAOA实验评估。

---

## 真机性能

### 量子退火（Q-GTSP）-- D-Wave Advantage2 QPU

- **硬件平台**：D-Wave Systems的Advantage2 QPU，采用Zephyr拓扑，拥有超过4,400个超导量子比特，具有增强的连接性[30]。（Section V-B）
- **参数设置**：`num_reads = 1500`，其余参数使用D-Wave默认值。Minor embedding由`EmbeddingComposite`自动处理。（Section V-A-a）

**Subsample Small（3--5节点）数据集表现**（Section VI-B, Figure 2）：
- 一致找到最优解。
- 可行解率范围：28.7%（16eil76, 4节点）至91.6%（6fri26, 4节点）。
- 实例6fri26_nodes_4 达到91.6%可行解率；16eil76_nodes_4仅28.7%。
- Figure 7(a)显示Q-GTSP在所有10个Subsample Small实例上的best-shot近似比均达到最优或接近最优。

**Subsample Medium（3--20节点）数据集表现**（Section VI-B, Figure 3, Table V）：
- 实例5ulysses22_nodes_3（3节点）：22.7%可行解率。
- 9p43_nodes_5（5节点）：50.9%可行解率。
- 10att48_nodes_7（7节点）：45.5%可行解率。
- 10hk48_nodes_10（10节点）：19.1%可行解率。
- 14st70_nodes_11（11节点）：23.3%可行解率。
- 11ft53_nodes_13（13节点）：仅0.1%可行解率，均值甚至低于随机猜测均值。
- 20kroD100_nodes_15（15节点）：0.2%可行解率。
- 20gr96_nodes_16（16节点）：无效巡回（Invalid tour）。
- 12brazil58_nodes_18（18节点）：无效巡回（Invalid tour）。
- 20rd100_nodes_20（20节点）：无效巡回（Invalid tour）。

**Preprocess Small（3--5节点）数据集表现**（Section VI-E, Figure 6, Table VIII）：
- 可行解率极高：3burma14达98.1%，4br17达92.5%，4gr17达90.6%，4ulysses16达87.1%，5gr21达70.0%，5gr24达65.5%，5ulysses22达67.8%。
- 可行解率范围为~65%至98%，明显高于相同节点数的Subsample Small实例。

**Preprocess Medium（3--20节点）数据集表现**（Table VI）：
- 3burma14（3节点）：95.4%可行解率。
- 6bayg29（6节点）：26.3%可行解率。
- 7ftv33（12节点）：0.1%可行解率。
- 8ftv38（13节点）、14st70（14节点）、10ftv47（15节点）、9ftv44（15节点）："Invalid tour"。
- 16pr76（16节点）、12ftv55（20节点）、20kroA100（20节点）："Could not embed"（嵌入失败），对应所需逻辑量子比特数为256、240、400。

### 门模型QAOA（C-QAOA）-- IBM Qiskit Aer模拟器

- **运行平台**：虽目标部署于IBM Heron系列处理器（约156量子比特，heavy-hex晶格）[31]，但完整评估在Aer模拟器上完成，原因是在IBM真实QPU上的机时不足。（Section V-B）
- **参数设置**：$p=1$，$10 \times 10$网格搜索，$\gamma \in [0.05, \pi]$，$\beta \in [0.05, \pi/2]$，1500 shots/次评估，batch size=1，每实例超时300秒。（Section V-A-b）
- **初始态**：单个随机可行one-hot计算基态。（Section IV-B）

**Subsample Small（3--5节点）表现**（Section VI-C, Figure 4, Table VII）：
- 可行解率范围：4.6%（20gr96, 5节点）至51.3%（9ftv44, 5节点）。
- 12ftv55_nodes_3（3节点）：36.9%可行解率。
- 6fri26_nodes_3（3节点）：35.8%；6fri26_nodes_4（4节点）：7.7%。
- 4ulysses16_nodes_4（4节点）：18.0%。
- 5ulysses22_nodes_4（4节点）：10.6%。
- 9ftv44_nodes_5（5节点）：51.3%。
- 20gr96_nodes_5（5节点）：4.6%。
- 9p43_nodes_5（5节点）：15.3%。
- 16pr76_nodes_3（3节点）和16eil76_nodes_4（4节点）："Invalid tour"。
- 可采样接近最优的巡回，但分布通常较宽且部分呈双峰分布——部分概率质量集中在最优解附近，另一部分却低于随机基线。

**Subsample Medium（3--20节点）表现**（Section VI-C, Figure 5, Table V）：
- 5ulysses22_nodes_3（3节点）：7.4%可行解率。
- 9p43_nodes_5（5节点）：12.7%可行解率。
- 10att48_nodes_7（7节点）：21.8%可行解率。
- 10hk48_nodes_10（10节点）："Invalid tour"。
- 14st70_nodes_11（11节点）："Invalid tour"。
- 11ft53_nodes_13（13节点）：7.3%可行解率。
- 20kroD100_nodes_15（15节点）：9.2%可行解率。
- 20gr96_nodes_16（16节点）："Invalid tour"。
- 12brazil58_nodes_18（18节点）：0.2%可行解率。
- 20rd100_nodes_20（20节点）：0.1%可行解率。

**Preprocess Small（3--5节点）表现**（Table VIII）：
- 3burma14（3节点）：22.5%可行解率。
- 4br17（4节点）：2.6%；4gr17（4节点）：2.9%；4ulysses16（4节点）：1.9%。
- 5gr21、5gr24、5ulysses22（全部5节点）：**全部超时**（Timeout）。

**Preprocess Medium（3--20节点）表现**（Table VI, Figure 8）：
- 3burma14（3节点）：22.7%可行解率。
- 6bayg29（6节点）：0.5%可行解率。
- 7ftv33（12节点）：0.3%可行解率。
- 8ftv38（13节点）及以上全部实例："Invalid tour"。

### 对比总结

- 在所有四个数据集的相同实例上，**Q-GTSP（量子退火）的可行解率始终高于C-QAOA**。论文将QAOA的较低表现主要归因于p=1的极浅层深度（Section VI-D）。
- Q-GTSP在Preprocess Small上达到最高可行解率98.1%（3burma14），C-QAOA的最高可行解率仅为51.3%（9ftv44, Subsample Small）。
- 两种方法在超过约15节点后均无法可靠找到可行解。
- 量子退火在Preprocess Medium上遇到嵌入失败（~256--400逻辑量子比特时超出Advantage2容量），而C-QAOA在Preprocess Small上遇到超时（所有5节点实例）。两者受不同瓶颈限制。
