# Multi-sequence alignment using the Quantum Approximate Optimization Algorithm -- 论文总结

---

## 目录

- [1. 核心结论与创新点](#1-核心结论与创新点)
- [2. 内容总结](#2-内容总结)
  - [2.1 对QAOA在理论方面的创新](#21-对qaoa在理论方面的创新)
  - [2.2 对QAOA工程实现技术的实践](#22-对qaoa工程实现技术的实践)
  - [2.3 帮助QAOA落地的技术创新](#23-帮助qaoa落地的技术创新)
  - [2.4 对QAOA实际应用的扩展与探索](#24-对qaoa实际应用的扩展与探索)
  - [2.5 QAOA对其他领域的启发式贡献](#25-qaoa对其他领域的启发式贡献)
- [3. 面临的困难、论文的不足之处和后续的改进方向](#3-面临的困难论文的不足之处和后续的改进方向)
- [4. 真机性能](#4-真机性能)

---

## 1. 核心结论与创新点

**来源：Abstract、Section I (Introduction)、Section V (Conclusion)**

- **首次将QAOA应用于多序列比对（MSA）问题**：论文首次提出了MSA的二进制编码方案，并设计了对应的软约束代价函数，将其映射为哈密顿量，从而能够在QAOA框架下处理该生物信息学中的核心组合优化问题。（Section I, Abstract）

- **证明了可行解空间在希尔伯特空间中的占比存在上界**：论文从理论上分析并给出了MSA问题的可行状态数与完整希尔伯特空间大小之比的上界，结果表明该比值随参考序列长度L以超指数方式衰减、随序列数量N以指数方式衰减。具体上界公式为：（Section IV.B, Appendix A）
  $$
  \frac{|S|}{|H|} \leq \frac{1}{L!} e^{-[\ln(2)L - \ln(L)][L+N-1]}, \quad \forall N, L \in \mathbb{N}_{\geq 2} \tag{28}
  $$

- **模拟结果验证了QAOA对MSA问题的可行性**：在经典量子模拟器中，随着QAOA层数p的增加（p < 5），理想MSA解（全局最优）对应的态成为采样概率最高的态。（Section IV.B.1, Fig. 5 & 6）

- **真机实验揭示了噪声限制**：在Rigetti Aspen-M-3真实超导量子计算机上运行的实验表明，当前噪声水平下无法从输出数据中区分可行解与其他态，说明实现有效MSA-QAOA需要进一步的电路编译优化或更紧凑的ansatz设计。（Abstract, Section IV.B.1, Fig. 7）

---

## 2. 内容总结

### 2.1 对QAOA在理论方面的创新

**来源：Section III (Encodings), Section IV (Variational Quantum Algorithms)**

**（1）MSA的One-Hot Column编码方案**

论文提出了一种One-Hot Column编码方案，将N条序列的MSA问题编码为二进制矩阵。对于每条序列的第n个字符，使用一个长度为L（最长序列的长度）的二进制向量表示该字符在矩阵中所在的列。总体所需的量子比特数为：（Section III.A）

$$
n = L \sum_{i=1}^{N} l_i, \quad \text{其中 } L \equiv \max\{l_i\}_{i=1}^{N} \tag{5}
$$

量子比特数随最大序列长度多项式扩展：\(n = O(NL^2)\)。

**（2）硬约束的数学表述**

编码方案引入三类硬约束（Section III.B.1）：

- 每个序列的每个字母必须恰好位于L个可用列中的某一列：
  $$
  \sum_{i=1}^{L} x_{s,n,i} = 1, \quad \forall s, n \tag{6}
  $$

- 每行的每一列最多被一个字母占据：
  $$
  \sum_{n=1}^{l_s} x_{s,n,i} \leq 1, \quad \forall s, i \tag{7}
  $$

- 字母顺序约束——每一个字母必须在更靠后的字母之前出现：
  $$
  \sum_{s=1}^{N} x_{s,n',i} x_{s,n,i'} = 0, \quad \forall n < n', \forall i < i' \tag{8}
  $$

**（3）从硬约束到软约束的转换**

论文参考了文献[27]的方法，将三类硬约束分别转换为带惩罚因子\(p_1, p_2, p_3\)的二次惩罚项：（Section III.B.3）

$$
\sum_i x_{s,n,i} = 1 \implies p_1 \sum_s \sum_n \left( \sum_i x_{s,n,i} - 1 \right)^2 \tag{10}
$$

$$
\sum_n x_{s,n,i} \leq 1 \implies p_2 \sum_s \sum_i \sum_{n<n'} x_{s,n,i} x_{s,n',i} \tag{11}
$$

$$
\sum_s x_{s,n',i} x_{s,n,i'} = 0 \implies p_3 \sum_s \sum_{n<n'} \sum_{i<i'} x_{s,n',i} x_{s,n,i'} \tag{12}
$$

**（4）完整的QUBO代价函数**

结合SP评分方案（Sum-of-Pairs）与上述软约束，最终构建的代价函数为：（Section III.B.2 & III.B.3）

$$
C(x) = \sum_{s<s'} \sum_{n,n'} \sum_{i} \omega_{s,n,s',n'} x_{s,n,i} x_{s',n',i} + p_1 \sum_s \sum_n \left( \sum_i x_{s,n,i} - 1 \right)^2 + p_2 \sum_s \sum_i \sum_{n<n'} x_{s,n,i} x_{s,n',i} + p_3 \sum_s \sum_{n<n'} \sum_{i<i'} x_{s,n',i} x_{s,n,i'} \tag{13}
$$

其中\(\omega_{s,n,s',n'}\)由预计算矩阵定义（基于SP评分方案），评分规则为：相同字母为-1，不同字母为+1，涉及空位的比较为0 (eq. 3)。

**（5）QUBO到Ising模型的映射**

通过变量变换\(x_{s,n,i} \to \frac{1 - \hat{\sigma}_{s,n,i}^z}{2}\) (eq. 16)，将QUBO模型映射为Ising哈密顿量：（Section III.C, Appendix B）

$$
C(s) = \frac{1}{4}s^T Q s - \frac{1}{4}\left(\mathbf{1}_n^T Q^T + \mathbf{1}_n^T Q + 2h^T\right)s + \frac{1}{4}\mathbf{1}_n^T Q \mathbf{1}_n + \frac{1}{2}\mathbf{1}_n^T h + d
$$

这直接给出了量子电路中单比特RZ门和两比特ZZ门的旋转角度——矩阵Q的非对角项代表量子比特间的耦合强度，对角项连同线性项确定单比特旋转角度。（Section III.C）

**（6）可行解空间占比的理论分析**

论文证明了MSA可行解数量在高维搜索空间中所占比例极低，上界随L呈超指数衰减、随N呈指数衰减。（Section IV.B, Appendix A, eq. 28）

$$
F \equiv \frac{|S|}{|H|} = \frac{\prod_{i=1}^{N} \binom{L}{l_i}}{2^{L \sum_{i=1}^{N} l_i}} \tag{A7}
$$

在一个实例中，10条待比序列对一条50字符长的目标序列进行比对，每条序列插入7个空位，可能的比对数量约为\(10^{79}\)，与已知宇宙中的原子数相当 (eq. 4)。这一分析从根本上解释了为何搜索完整希尔伯特空间效率极低。

---

### 2.2 对QAOA工程实现技术的实践

**来源：Section IV, Appendix C**

**（1）QAOA标准ansatz在MSA上的实现流程**

论文给出了完整的工程实现路径（Section IV.B）：

- 初始态制备：将输入制备为均匀叠加态：
  $$
  |\psi_1\rangle = \hat{H}^{\otimes n}|0\rangle^{\otimes n} = \frac{1}{\sqrt{2^n}} \sum_{x \in \{0,1\}^n} |x\rangle \tag{22}
  $$

- 问题酉算子：\(\hat{U}_P(\gamma) = e^{-i\gamma \hat{H}_P}\)，其中\(\hat{H}_P\)由Pauli-Z算子的张量积构成 (eq. 20)

- 混合酉算子：\(\hat{U}_M(\beta) = e^{-i\beta \hat{H}_M}\)，由标准单比特X旋转生成：
  $$
  e^{-i\beta \hat{H}_M} = \left( R_x(\beta/2) \right)^{\otimes n} \tag{21}
  $$

- 试态构造（p层重复）：
  $$
  |\psi(\beta, \gamma)\rangle = \left( e^{-i\gamma_1 \hat{H}_P} e^{-i\beta_1 \hat{H}_M} \right) \cdots \left( e^{-i\gamma_p \hat{H}_P} e^{-i\beta_p \hat{H}_M} \right) |\psi_1\rangle \tag{23}
  $$

**（2）Toy Model的具体构造**

论文构建了一个2序列的本征示例：S = (AG, G)，使用6个量子比特编码（每个字符在每个可能的列位置有一个二进制变量[2序列 x (序列1最多2字符 + 序列2最多1字符) x 最大长度2 = 6]）。（Section IV.B.1, eq. 29-30）

对应p=1的完整QAOA电路在附录C的Fig. 8中展示。

**（3）参数优化与软件实现**

- 经典优化器：采用COBYLA（Constrained Optimization by Linear Approximation）算法，由SciPy实现[40]
- 量子模拟框架：Qiskit
- 惩罚因子设置：\(p_1 = 10, p_2 = p_3 = 1\)

**（4）模拟结果**（Section IV.B.1, Fig. 5 & 6）

- p=1时：采样概率最高的态为\(|110001\rangle\)，该态并不可行
- p=2时：\(|011010\rangle, |101001\rangle\)为最可能态，全局最优\(|100101\rangle\)已出现在前十中
- p=3时：全局最优\(|100101\rangle\)的排名继续提升
- p=4和p=5时：\(|100101\rangle\)（对应理想MSA——AG/(空格)G对齐）成为**最可能的采样态**
- 概率趋势：全局最优态的采样概率随p单调递增

这表明**在理想无噪声条件下，QAOA确实有能力找到MSA的全局最优解**。

---

### 2.3 帮助QAOA落地的技术创新

**来源：Section IV.B**

**（1）软约束转换方法论**

论文系统性地展示了如何将组合优化问题中的硬约束转换为可被QAOA直接处理的软约束二次惩罚项。这一转换参考了Glover等人的QUBO教程[27]，给出了三种典型约束类型（等式约束、不等式约束、乘积为零约束）对应的惩罚项构造范式 (eq. 10-12)。

**（2）预计算权重矩阵策略**

为使得编码方案能够处理不同的字母（A, C, T, G），论文采用了预处理权重矩阵\(\Omega\)的策略，将所有序列对之间的字符级评分预先计算并存储，再在代价函数中通过\(x_{s,n,i} x_{s',n',i}\)的乘积项实现对SP评分方案的等价实现 (eq. 9)。（Section III.B.2）

**（3）约束保持混合算子（Constraint-Preserving Mixers）的设想**

论文明确指出了标准Pauli-X混合算子的局限性——它会在整个希尔伯特空间中进行搜索，而可行解仅占极小部分。论文提出应探索约束保持混合算子来限制搜索空间到可行子空间S，这与Hadfield等人[35]及Fuchs等人[36]的QAOA变体方法一致。（Section IV.B）

---

### 2.4 对QAOA实际应用的扩展与探索

**来源：Section I, Section II, Section V**

**（1）QAOA在生物信息学领域的新应用场景**

论文首次将QAOA框架应用于生物信息学中的一个核心问题——多序列比对（MSA）。MSA广泛应用于系统发育推断、功能分析和结构预测等领域[12,13]，论文的探索表明QAOA在生物信息学领域具有潜在的应用价值。（Section II.A, Section I）

**（2）全局比对（Global Alignment）建模**

论文聚焦于全局比对问题，即在考虑完整序列的前提下进行比对，这与局部比对（Local Alignment）形成对照，扩展了量子优化在生物序列分析中的应用范围。（Section II.A）

**（3）基于Sum-of-Pairs评分的QUBO建模**

论文以Sum-of-Pairs (SP)评分方案作为代价函数的核心，成功将其转化为QUBO格式。这一建模方法具有通用性——原则上可以替换为其他评分方案，只需修改预计算权重矩阵\(\Omega\)即可。（Section II.C, Section III.B.2）

---

### 2.5 QAOA对其他领域的启发式贡献

**来源：Section IV.B**

标准QAOA的Pauli-X混合器会在整个\(2^n\)维希尔伯特空间中搜索，但当问题的可行解空间S远小于H时（如MSA中\(\frac{|S|}{|H|}\)随问题规模指数级衰减），大量量子资源被浪费在评估不可行态上。论文通过严格的理论上界 (eq. 28) 量化了这一矛盾，为**更广泛类型的约束组合优化问题**提供了重要的方法论启示：对于具有大量约束的优化问题，在设计QAOA ansatz时应优先考虑**约束保持混合算子**而非依赖软约束惩罚项。这一观察对任何试图利用QAOA解决具有严格约束的NP-hard问题（如调度问题、装箱问题等）均具有借鉴意义。（Section IV.B）

---

## 3. 面临的困难、论文的不足之处和后续的改进方向

**来源：Section IV.B, Section V (Conclusion)**

| 序号 | 困难/不足 | 详细说明 | 来源 |
|------|-----------|----------|------|
| 1 | **编码冗余度高** | One-Hot Column编码存在严重冗余——可行解仅占完整希尔伯特空间中指数衰减的部分。可行状态占比上界随L超指数衰减、随N指数衰减 (eq. 28)。 | Section IV.B, Appendix A |
| 2 | **真机噪声过大** | 在Rigetti Aspen-M-3超导量子计算机上运行的实验结果显示，噪声导致输出分布高度随机，无法区分可行解与其他态。这表明当前NISQ设备的噪声水平尚不适合此种MSA-QAOA算法。 | Section IV.B.1 (Fig. 7), Section V |
| 3 | **软约束引入超参数负担** | 软约束转换引入了额外的惩罚因子\(p_1, p_2, p_3\)，以及层数p等超参数，这些都需要在电路执行前进行预优化，增加了算法的实用复杂性。 | Section V |
| 4 | **经典优化子程序不确定** | QAOA中的经典优化步骤依赖COBYLA算法，但该算法的选择并非唯一的，且其收敛性取决于问题的具体特性。论文未对不同优化器进行对比研究。 | Section V |
| 5 | **问题规模限制** | 研究仅在一个极小规模的toy model（2条序列，6个量子比特）上进行了模拟和真机实验。对于实际规模的MSA问题（例如10条以上序列、序列长度>100字符），所需量子比特数和电路深度将急剧增长。 | Section IV.B.1 |
| 6 | **电路编译策略未优化** | 论文指出所选电路编译方法和量子比特拓扑布局对性能有显著影响，但并未对不同编译策略进行比较实验。 | Section IV.B.1, Section V |
| 7 | **未探索约束保持混合算子** | 论文虽然指出了约束保持混合算子的必要性，但并未在研究中实现或测试任何具体的约束保持混合器设计。 | Section IV.B, Section V |
| 8 | **评分方案的局限性** | 采用的SP评分方案（相同字母-1，不同字母+1，含空位0）是最简化的版本，生物信息学实际应用中使用更复杂的评分矩阵（如PAM、BLOSUM等）。论文的One-Hot编码本身也无法区分不同字母。 | Section III.B.2 |

**后续改进方向（Section V）：**

1. **设计约束保持混合算子**（Constraint-Preserving Mixers）：使得QAOA的混合层只在可行子空间S内进行搜索，从而提高算法效率。
2. **改进编码方案**：寻找更紧凑的编码方式以减少量子比特需求和冗余。
3. **优化电路编译策略**：针对特定量子硬件拓扑设计专用的量子比特映射和门分解方案。
4. **超参数优化自动化**：开发更系统化的预处理优化策略以确定惩罚因子、层数p和优化器选择。
5. **门分解效率研究**：探索约束保持算子能否在基于门的量子计算机上被高效分解和实现。

---

## 4. 真机性能

**来源：Section IV.B.1 (Fig. 7), Abstract**

论文在 **Rigetti Aspen-M-3**（79量子比特超导量子计算机）上运行了toy model的QAOA电路（2序列MSA，6量子比特，p=1至p=5，每次5000 shots）。

### 真机实验结果

| 指标 | 结果 |
|------|------|
| **设备** | Rigetti Aspen-M-3 (79-qubit superconducting) |
| **问题规模** | 2条序列（"AG"和"G"），6量子比特 |
| **QAOA层数** | p = 1, 2, 3, 4, 5 |
| **每层采样数** | 5000 shots |
| **惩罚因子** | \(p_1=10, p_2=p_3=1\) |
| **全局最优态** | \(\vert 100101\rangle\) |

### 核心发现

- **噪声主导输出了无意义的概率分布**：真机结果显示的输出概率分布（Fig. 7）表现为高度的随机性，与模拟结果（Fig. 5）形成鲜明对比。在模拟中，随着p从1增加到5，可行解的采样概率稳步提升，且在p>=4时全局最优态\(|100101\rangle\)成为最可能的态——但在真机中，这一趋势完全消失。

- **无法区分可行解与不可行解**：论文明确指出，在当前噪声水平下，**无法从量子硬件输出数据中将可行解与其他态区分开来**。"the level of noise in current devices is still a formidable challenge"（当前设备的噪声水平仍然是这类MSA-QAOA算法的巨大挑战）。(Abstract)

- **与模拟结果的对比**：模拟中全局最优态的最高采样概率约~0.18（p=5时），而在真机数据中，该态甚至无法可靠地出现在概率分布的top榜单中。

- **论文的一个说明**：Fig. 7中的态排序是按照Fig. 6的顺序排列的（以便与模拟结果对比），而不是按照采样概率排序。因此，真机运行中最可能的态可能并未显示在Fig. 7的直方图中，这进一步印证了结果的高度随机性。

### 结论

论文的真机实验实际上是一个**负面结果**（negative result）——它诚实地报告了在当前NISQ设备上MSA-QAOA尚无法获得有意义的性能。作者指出，需要进一步研究：(1) 改进电路编译策略；(2) 设计更紧凑的ansatz（如约束保持混合器）；(3) 等待下一代噪声水平更低、电路深度更大的量子硬件，才有可能使QAOA在MSA问题上展现出可与经典方法竞争的性能。（Section V）

---

*论文信息：arXiv:2308.12103v2 [quant-ph], 2025年4月24日。作者：Sebastian Yde Madsen, Frederik Kofoed Marqversen, Stig Elkjær Rasmussen, Nikolaj Thomas Zinner，来自奥尔胡斯大学物理与天文系及Kvantify ApS。*
