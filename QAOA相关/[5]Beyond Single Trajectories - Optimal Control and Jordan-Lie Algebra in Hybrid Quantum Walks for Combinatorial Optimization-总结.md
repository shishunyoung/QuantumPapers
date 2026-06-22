# [5] Beyond Single Trajectories: Optimal Control and Jordan-Lie Algebra in Hybrid Quantum Walks for Combinatorial Optimization -- 总结

**论文信息**: Tianen Chen, Yun Shang (中国科学院数学与系统科学研究院), arXiv:2604.25760v1, 2026年4月28日

---

## 目录 (Table of Contents)

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

本文通过将混合量子行走（Hybrid Quantum Walk, HQW）引入QAOA范式，提出了超越单一路径演化的量子优化框架。其核心贡献可概括为三个层面：

**理论拓展（Section III-IV, 第3-5页）**：
- QAOA被严格证明为HQW框架的一个特例 -- 当coin算子固定为Pauli-X门时，HQW退化为标准QAOA（式(14)）。所有通过修改QAOA哈密顿量得到的变体（如multi-angle QAOA）也同样可视为对应HQW的特例。
- 利用庞特里亚金极小值原理（Pontryagin's Minimum Principle, PMP）导出了最优coin算子的解析形式 -- 最优coin在每个时刻应为以瞬时"灵敏度向量" `(ΦX, ΦY, ΦZ)` 为轴的旋转操作（式(25)-(27)），这通常不等于Pauli-X门，从而证明QAOA在HQW框架内并非最优。

**代数结构分析（Section V-VI, 第6-8页）**：
- HQW生成严格更大的Jordan-Lie代数 `g_H = su(2) ⊗ L_Q ⊕ I_c ⊗ K_Q`（式(29)和定理1），其维数满足 `dim(g_H) > 4 dim(g_Q)`，而QAOA的DLA仅为该结构的Lie子代数 `g_Q = ⟨iH_c, iH_b⟩_Lie`（式(28)）。
- 揭示了Jordan积负性 `N_min = min λ(H_c ∘ H_b)`（式(33)）与HQW性能优势之间的强相关性：相关系数 r = 0.9605, p = 7.15×10⁻¹¹（FIG. 5），为路径叠加ansatz何时优于传统方法提供了定量判据。

**数值验证（Section VII, 第8-12页）**：
- 在50个8顶点Max-Cut图实例上，HQW在49/50个实例中平均性能优于QAOA，获胜率98%，平均相对改进11.63%（18-23边图）至28.26%（24-28边图）（Table I, Table II）。
- 组件消融实验证实路径叠加（而非简单的参数重排）是性能优势的根本来源（FIG. 12-13）。

---

## 2. 内容总结

### 2.1 对QAOA在理论方面的创新

**QAOA作为HQW特例的严格证明（Section III）**：

论文将QAOA的两个核心哈密顿量 `H_c` 和 `H_b` 以条件演化形式插入HQW框架：`|1⟩⟨1|⊗H_c` 和 `|0⟩⟨0|⊗H_b`。HQW的单步算子为：

$$ W(\gamma, \beta) = e^{-i|1\rangle\langle 1|\otimes H_c \gamma} e^{-i|0\rangle\langle 0|\otimes H_b \beta}(C \otimes I) \tag{11}$$

当coin算子取为Pauli-X门且每个演化算子重复作用一次时，末态明确收敛为QAOA的ansatz：

$$ |0\rangle \otimes e^{-iH_b\beta_p}e^{-iH_c\gamma_p} \cdots e^{-iH_b\beta_1}e^{-iH_c\gamma_1}|s\rangle \tag{14}$$

这意味着QAOA仅是HQW中所有coin操作均为Pauli-X的特殊情况，其仅对应两条哈密顿量的一种交错演化路径，无法产生不同路径的叠加。

**路径叠加机理（Section III, 式(13)）**：

一般HQW的末态为所有演化路径的相干叠加：

$$ |\psi_f\rangle = \sum_{v \in \mathbb{F}_2^p} \alpha_v |v_p\rangle \otimes \prod_{k=1}^{p} (v_k U_c(\gamma_k) + (1-v_k)U_b(\beta_k))|s\rangle \tag{13}$$

其中系数 `α_v = ⟨v_p|C_p|v_{p-1}⟩···⟨v_1|C_1|0⟩` 由coin算子序列精确控制。这使HQW能够通过coin算子在每次电路层内相干地叠加多条哈密顿驱动的演化路径。

**最优coin算子的理论推导（Section IV）**：

利用庞特里亚金极小值原理，论文构建了包含三个动力学历程的时变哈密顿量：

$$ H(t) = y_1(t)|1\rangle\langle 1| \otimes H_c + y_2(t)|0\rangle\langle 0| \otimes H_b + y_3(t)(n_x(t)X + n_y(t)Y + n_z(t)Z) \otimes I \tag{17-18}$$

其中控制参数 `u(t) ∈ {0, 1/2, 1}` 分别对应coin操作、`H_b` 演化和 `H_c` 演化三个阶段。这也是对"量子绝热演化含中间哈密顿量"这一更大类别的离散化 -- QAOA对应标准的量子绝热演化离散化，而HQW对应带有中间哈密顿量的扩展绝热演化离散化（Section IV, 第4-5页）。

通过构造Lagrange函数并求解极值条件，得到最优coin参数：

$$ n_x(t) = \frac{\Phi_{X\otimes I}(t)}{\sqrt{\Phi_X^2 + \Phi_Y^2 + \Phi_Z^2}}, \quad n_y(t) = \frac{\Phi_{Y\otimes I}(t)}{\sqrt{\Phi_X^2 + \Phi_Y^2 + \Phi_Z^2}}, \quad n_z(t) = \frac{\Phi_{Z\otimes I}(t)}{\sqrt{\Phi_X^2 + \Phi_Y^2 + \Phi_Z^2}} \tag{25-27}$$

其中 `Φ_A(t) = i⟨k(t)|A|ψ(t)⟩ + c.c.`，这表明最优coin旋转轴必须与coin空间中的瞬时"灵敏度向量"对齐。而Pauli-X门作为固定算子，其作用方向不随瞬时状态调整，因此一般不满足此最优条件。

**Jordan-Lie代数分析（Section V）**：

QAOA的DLA仅由问题哈密顿量和混合哈密顿量的Lie闭包生成：

$$ g_Q = \langle iH_c, iH_b \rangle_{Lie} \subseteq \mathfrak{u}(N) \tag{28}$$

而HQW的DLA具有更丰富的结构：

$$ g_H = \mathfrak{su}(2) \otimes L_Q \oplus I_c \otimes K_Q \tag{29}$$

其中 `L_Q` 是由 `{iH_c, iH_b, iI_p}` 通过交换子 `[·,·]` 和Jordan积 `i{·,·}` 生成的Jordan-Lie代数（Lemma 1, Appendix A），`K_Q = [L_Q, L_Q] ⊕ span{iH_c, iH_b}`。DLA维数的严格不等式为：

$$ \dim(g_H) > 3\dim(g_Q) + \dim(g_Q) = 4\dim(g_Q) \tag{30}$$

若coin的引入不影响位置空间的动力学演化，应有 `dim(g_H) = 4\dim(g_Q)`。不等式（30）说明coin通过生成Jordan积丰富了位置空间的演化模式，这为HQW的性能优势提供了严格的代数基础。

**Jordan积负性与路径叠加（Section VI）**：

Jordan积定义为：

$$ H_c \circ H_b = \frac{1}{2}(H_c H_b + H_b H_c) \tag{32}$$

Jordan积负性定义为：

$$ N_{\min} = \min \lambda(\tilde{H}_c \circ \tilde{H}_b) \in [-1, 0) \tag{33}$$

通过对单层QAOA的Taylor展开分析（式(35)），论文证明Jordan积 `H_c ∘ H_b` 在二阶出现，且当它存在负特征值时，任何参数 `(β, γ)` 的选择都无法消解其引入的约束。由于QAOA的DLA仅由交换子生成，Baker-Campbell-Hausdorff公式（式(37)）表明反对易子（进而Jordan积）不在Lie闭包中，QAOA流形 `M_Q = exp(g_Q)` 无法沿Jordan方向演化。

HQW通过引入coin空间，使得路径叠加态 `|Ψ⟩ = α|0⟩⊗e^{-iH_bβ}|ψ⟩ + β|1⟩⊗e^{-iH_cγ}|ψ⟩`（式(38)）成为可能。当与条件演化结合时，coin参数的导数生成正比于 `i{H_c, H_b}` 的项，这恰好提供了QAOA流形中缺失的横向方向。因此，HQW的演化流形切空间分解为 `T_U M_H = T_U M'_Q ⊕ T_U`（式(31)），额外的横向方向使HQW能够逃离局部极小值并在更高维的搜索空间中向更优解导航。

几何解释如FIG. 4所示：当 `N_min ≈ 0` 时，QAOA流形 `M_Q` 接近平坦；当 `N_min ≪ 0` 时，HQW流形 `M_H` 高度弯曲，由Jordan积生成的横向方向（红色箭头）使得HQW能够导航整个流形。

### 2.2 对QAOA工程实现技术的实践

**HQW电路实现（Appendix B, Section 2）**：

论文给出了HQW在PennyLane框架中的具体电路实现方案：
- 使用 n+1 个量子比特（n个系统比特代表图顶点 + 1个coin比特控制演化）
- 所有系统比特初始化为 `H|0⟩`（Hadamard门）
- 每步执行：(1) 对coin比特施加U3门，`U3(θ, ϕ, δ)`（式(B5)）；(2) 根据coin比特状态施加条件演化 -- 若coin为 `|1⟩` 则对系统施加 `exp(-iθ_{s,0}H_c)`，若为 `|0⟩` 则施加 `exp(-iθ_{s,1}H_b)`
- 条件演化通过PennyLane的`ctrl`操作结合`ApproxTimeEvolution`实现
- 为公平比较，设QAOA层数为p，HQW步数为2p，确保两种电路施加相同总数的哈密顿演化操作（Section VII, Appendix B）

**优化协议（Section VII, Appendix B）**：
- 优化器：Adam，学习率0.1
- 参数初始化：均匀随机采样自 `[0, 2π]`
- 梯度计算：利用参数偏移法则（parameter-shift rule）或自动微分精确计算梯度
- 100次随机初始化，每次运行300步优化
- 软件栈：PennyLane v0.34.0 + NumPy v1.24.3，模拟器为`default.qubit`

**PMP理论与数值优化的衔接（Section IV末, 第5-6页）**：

虽然PMP给出了最优coin旋转轴的闭式解，但该解依赖于瞬时状态和伴随态，难以直接在真实量子硬件上编译。然而，HQW的目标函数 `⟨I⊗H_c⟩` 对coin参数光滑可微，梯度可通过参数偏移法则或自动微分精确计算，因此基于梯度的优化器（如Adam）可以自动调整coin参数。PMP在此扮演理论保证的角色：它揭示了固定coin（如Pauli-X）一般不最优，并为HQW的参数化结构提供了原理性理解，从而架起了最优控制理论与实际变分量子算法之间的桥梁。

### 2.3 帮助QAOA落地的技术创新

**算法选择判据（Section VI-E, 第8页）**：

论文提出了一个实用的算法选择标准：在执行变分量子算法之前，可先从问题哈密顿量计算 `N_min`。若 `|N_min|` 超过某一阈值（例如 > 0.1），则HQW这类能实现路径叠加的ansatz很可能优于QAOA这类单路径方法。这为工程实践中的算法选型提供了定量指导。

**设计原理（Section V末和Section VI-E, 第6页和第8页）**：

论文将Jordan-Lie代数分析提升为通用的量子优化器设计原则：为提升性能，应有意识地构造能够激活问题完整Jordan-Lie代数结构（而非仅其Lie代数分量）的ansatz。这一原则可指导未来量子优化算法的工程设计。

**组合优化问题的图编码（Appendix B, Section 1）**：

对于Max-Cut，采用标准哈密顿量：
$$ H_c = \sum_{(i,j)\in E} \frac{I - Z_i Z_j}{2}, \quad H_b = \sum_i X_i \tag{B1}$$

对于Maximum Independent Set，编码为：
$$ H_c = -\sum_i n_i + \lambda \sum_{(i,j)\in E} n_i n_j, \quad n_i = \frac{1}{2}(I - Z_i) \tag{B2}$$

论文进一步指出，可将问题映射到n维超立方体 `Q_n`，其中顶点x处的自环权重为 `N(x)`（目标函数值），所有自环标记为1，其他边标记为0，作为HQW的底层图结构（Appendix B, 式(B3)）。这一清晰的图结构为未来在物理平台上实验实现HQW解决组合优化问题奠定了理论基础。

### 2.4 对QAOA实际应用的扩展与探索

**Max-Cut问题的基准测试（Section VII-A, 第9-10页）**：

在50个8顶点、18-23边的连通图上，HQW与QAOA的对比（Table I）：
- HQW平均 `1-r` 值：0.154200 vs QAOA：0.175399
- HQW在49/50实例中平均性能更优，获胜率98%
- 平均相对改进：11.63%，中位数相对改进：11.93%

在50个8顶点、24-28边的图上（Table II）：
- HQW平均 `1-r` 值：0.074771 vs QAOA：0.093015
- HQW在所有50个实例中均优于QAOA，获胜率100%
- 平均相对改进：28.26%，中位数相对改进：20.80%，最高可达约80%
- 观察到：图边密度越高，HQW相对QAOA的性能改进越显著，说明HQW更擅长处理高边密度图；在完全图K8上HQW展示了最大的性能优势

在最优性能比较中（Table III, 50个8顶点随机图，20-28边）：
- HQW在68%的实例中（34/50）优于QAOA，在其余32%（16个实例）中持平
- 平均相对改进：0.86%
- 所有数据点在对角线上或之上（FIG. 18），HQW从未劣于QAOA

**Maximum Independent Set问题的测试（Section VII-B, 第10-12页）**：

在8顶点MIS实例上（FIG. 13），四条曲线对比表明：性能排序为 HQW > QAOA > Variant 1/Variant 2，HQW在所有深度p上收敛更快、达到更低能量、基态占据概率更高，且具有最小的标准偏差。

**组件消融实验（Section VII-B, 第10-12页）**：

设计了四种算法的对照实验：
1. Classic QAOA：标准QAOA
2. HQW：混合量子行走
3. QAOA Variant 1：取HQW奇步的 `γ_k` 和偶步的 `β_k` 构造QAOA电路
4. QAOA Variant 2：取HQW偶步的 `γ_k` 和奇步的 `β_k` 构造QAOA电路

结果表明：HQW的性能优势不能通过简单的QAOA参数重排来复现，支撑路径叠加的coin控制结构才是性能优势的关键来源。两个变体的表现显著差于HQW，证明"参数移植"无法替代结构创新。

**可扩展性测试（Section VII-D, 第12页）**：

- 12顶点、30边Max-Cut实例（FIG. 16）：HQW收敛更快，在低深度即可达到更低能量
- 10顶点MIS实例（FIG. 17）：HQW在所有方法中表现最好，收敛更快且最终能量更低
- 两种情况下HQW均保持最小的跨初始化标准差，表明其鲁棒性随问题规模增大而保持

### 2.5 QAOA对其他领域的启发式贡献

**资源论视角（Section VI-E, 第8页）**：

论文提出Jordan积负性可以被解释为一种量子资源（quantum resource for optimization）。HQW框架通过其路径叠加能力有效利用这一资源，而QAOA则未能做到。这将量子测量不相容性（measurement incompatibility）这一基础量子特性与量子优化中的实际计算优势联系起来，为量子资源理论在优化算法设计中的应用开辟了新方向。

**代数设计原则（Section V末, 第6-7页）**：

论文从Jordan-Lie代数的角度提出的"激活完整代数结构"的设计原则，超越了具体算法范畴，可推广为通用的变分量子电路设计方法论：在构造ansatz时，不应仅考虑哈密顿量的Lie代数结构，而应有意识地引入机制来利用其完整的Jordan-Lie代数结构（包括Jordan积分量）。

---

## 3. 面临的困难、论文的不足之处和后续的改进方向

**直接实现最优coin算子的困难（Section IV末和Section VIII, 第5-6页, 第12-13页）**：

论文明确指出：庞特里亚金极小值原理导出的最优coin旋转轴依赖于瞬时状态和伴随态的闭式解，"难以直接在真实量子硬件上编译"（difficult to compile directly on real quantum hardware）。当前只能依赖Adam等数值优化器在参数空间中迭代搜索来逼近最优条件，这意味着实际实现中不一定能真正达到理论最优。不过，论文指出这并不妨碍高效数值优化获得接近最优的coin参数，PMP在此充当理论指导而非直接编译工具。

**问题规模限制（Section VII-D和Section VI-E, 第12页, 第8页）**：

数值实验的图规模限于8-12顶点，尚未在更大规模问题上验证HQW的优势。论文明确将"扩展到更大系统"列为未来工作的重要方向。

**缺乏严格的理论界（Section VI-E, 第8页）**：

论文指出"建立将 `N_min` 与性能优势下界联系起来的严格理论证明仍是一个开放挑战"（A rigorous theoretical proof linking N_min to a lower bound on the performance advantage remains an open challenge）。目前的结论主要基于数值实验中的强相关性（r = 0.9605），尚未从理论上严格证明Jordan积负性的大小能够给出性能改进的定量下界。

**噪声影响未评估（Section VI-E, 第8页）**：

论文未在噪声模型下评估HQW的性能，明确提出问题："NISQ设备中的噪声和退相干如何影响这一优势的可实现性仍是一个问题"（how noise and decoherence in NISQ devices might affect the realizability of this advantage）。由于HQW使用额外的coin量子比特和条件演化操作（受控门），相比标准QAOA有更深的电路和更多的两比特门，噪声环境下的实际表现可能受到影响。

**高维推广（Section VI-E, 第8页）**：

论文指出将框架推广至多于两个哈密顿量的情形涉及高阶Jordan代数，"可能揭示更丰富的结构"（may reveal even richer structure），当前工作仅针对两个哈密顿量 `H_c` 和 `H_b` 的情况。

**最优性能改进幅度有限（Section VII-A, Table III, 第10-11页）**：

在最优性能（best-performance）比较中，HQW的平均相对改进仅为0.86%，尽管从未劣于QAOA，但绝对改进幅度较温和。论文指出这30个图中有17个改进接近于零（在±0.1%以内），只有3个图达到了5%-20%的中等改进。这表明在小规模实例上，两类算法的最优可达性能可能较为接近。

**appendix图表数据的解读需谨慎（Section VII-A末, 第16-17页）**：

15个图在直方图中显示负相对改进（FIG. 19），论文解释这是由于两个算法的绝对差异极小（小于10⁻⁶）时，数值波动导致了微小的负值，这些实例在容差标准下仍被分类为"性能持平"。这表明在最优性能层面上两类算法的区分度有时非常微弱。

---

## 4. 真机性能

**本论文未在真实量子计算机上开展实验。** 所有数值实验均在经典模拟器上完成（PennyLane的`default.qubit`模拟器，运行于NumPy后端，详见Section VII和Appendix B）。论文使用的是无噪声的理想模拟环境（statevector simulation），不涉及任何量子硬件。

论文所报告的"性能数据"均为理想模拟环境下的结果，包括：
- 50个8顶点Max-Cut图的平均性能和最优性能统计（Table I-III）
- 组件消融实验中的收敛曲线和基态概率（FIG. 12-13）
- 12顶点和10顶点图的可扩展性测试（FIG. 16-17）

论文未涉及量子纠错、错误缓解或任何噪声适应策略，也未讨论在真实NISQ设备上的部署方案。关于真实硬件的实现，论文仅在Appendix B的图编码部分提及"为未来在物理平台上的实验实现奠定了理论基础"，并在Section VI-E中将噪声影响列为待解决的开放问题。
