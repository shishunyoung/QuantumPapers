# Multiscale Quantum Approximate Optimization Algorithm (MQAOA) 论文总结

arXiv:2312.06181v1 [quant-ph], 2023年12月11日

作者: Ping Zou (华南师范大学信息光电子科技学院，广东省量子工程与量子材料重点实验室)

---

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

（来源：Abstract、Introduction、Conclusion）

1. **提出了一种全新的QAOA变体 -- 多尺度量子近似优化算法（MQAOA）**，将标准QAOA与实空间重整化群（real-space renormalization group, RG）变换相结合。通过QAOA与实空间RG变换之间的反馈循环（feedback loop），显著提升了算法性能。（来源：Abstract、Section "Multiscale QAOA"）

2. **突破了标准QAOA在低深度下的局域性限制**。标准QAOA在depth-p下，与边⟨j, k⟩相关的期望值仅涉及qubit j、k以及图上与其距离≤p的顶点。MQAOA通过步骤(2)中的短距离连接与步骤(4)中的长距离连接之间的竞争（competition between the short-distance connection in step (2) and the long-distance connection in step (4)），使得低深度QAOA也能建立长距离量子比特之间的关联。（来源：Section "Quantum approximate optimization algorithm" 末尾段及 "Multiscale QAOA"）

3. **即使使用最低深度（depth-1）的QAOA，也能为特定随机生成的问题实例提供精确解**。数值模拟表明，对所有随机采样图集合中的实例，仅需在每个粗粒化步骤中采用depth-1 QAOA即可获得精确解。（来源：Abstract、Section "Multiscale QAOA" 中数值实验部分）

4. **避免了贫瘠高原（barren plateaus）问题**。由于MQAOA仅需使用低深度QAOA，不需要大量参数，因此不受贫瘠高原限制。（来源：Introduction 及 Conclusion）

5. **算法复杂度高效**：若每层有效哈密顿量的QAOA轮数限制为常数K、不同划分方案数限制为常数C，则QAOA的总运行次数约为O(N log₂(CK))。数值实验表明K = 6已足够，因此MQAOA是一个高效的算法。（来源：Section "Computational complexity"）

6. **克服了对称性保护的限制**。通过在多基态情况下选择一个对称性破缺的乘积态，确保了算法不受参考文献[11]中提出的对称性保护（symmetry protection）带来的限制。（来源：Section "Multiscale QAOA" 第4步说明）

---

## 内容总结

### 对QAOA在理论方面的创新

（来源：Abstract、Introduction、"Real-space RG on graphs"、"Multiscale QAOA"各节）

1. **首次将实空间重整化群（RG）变换引入QAOA框架**。论文提出了一种针对定义在图上的量子系统的实空间RG变换方案。传统的实空间RG方法用于格点系统（如DMRG [17,18]用于一维系统、纠缠重整化[20-22]用于高维或标度不变系统），本文将其推广到一般图结构上，并与QAOA结合形成MQAOA。（来源：Introduction 及 "Real-space RG on graphs"）

2. **提出基于最大匹配（maximal matching）的顶点分块与粗粒化方案**。在MQAOA中，首先寻找图G的最大匹配M（边子集中每个顶点仅出现一次），匹配M中每条边连接的两个顶点被分入同一块，通过粗粒化每个块被转化为一个有效顶点（携带一个qubit），图被转化为顶点数大为减少的有效图。（来源：Section "Real-space RG on graphs"）

3. **提出受约束的二维子空间构造方法**。与传统DMRG不同，MQAOA的等距映射（isometry）构造不仅要求最小化截断误差 \|\|ρ(k) - Pρ(k)P\|\|，还额外要求三个算符 σᶻ_i、σᶻ_j 和 σᶻ_iσᶻ_j 在新基底向量下是对角的，以保证粗粒化后得到的有效哈密顿量保持Ising形式（即式(1)的形式）。这一约束条件是MQAOA理论框架的关键创新，确保了QAOA可以在粗粒化后的有效哈密顿量上继续使用。（来源：Section "Real-space RG on graphs" 后半部分）

4. **通过密度矩阵重构实现混合量子-经典粗粒化**。每个块k的约化密度矩阵ρ(k)通过在量子计算机上测量15个局域算符的期望值⟨σᵅ_i σᵝ_j⟩来重建：ρ(k) = Σ_{α,β} ⟨σᵅ_i σᵝ_j⟩ σᵅ_i σᵝ_j，其中α, β = 0, 1, 2, 3，σ⁰为恒等算符，其余为Pauli算符。（来源：Section "Real-space RG on graphs" 中式(3)前后段落）

5. **提出反馈循环架构**。MQAOA的核心工作流程包含六个步骤（详见下文），形成从G⁽⁰⁾到G⁽¹⁾再到G⁽⁰⁾的反馈循环，直至能量收敛。这一迭代架构使得低深度下的短程QAOA与RG变换中的长程信息传递交替进行，实现了"多尺度"效果。（来源：Section "Multiscale QAOA" 步骤1-6）

6. **生成了一系列迭代的哈密顿量序列 {H⁽ⁿ⁾_z}**，其对应图中顶点之间体现了原始图上不同尺度的连接关系，正因如此算法被命名为"多尺度"（multiscale）QAOA。当达到某个特定的n₀时，H⁽ⁿ⁰⁾_z可由量子或经典计算机有效求解。（来源：Section "Multiscale QAOA"）

### 对QAOA工程实现技术的实践

（来源：Section "Real-space RG on graphs"、"Multiscale QAOA"、"Computational complexity"、Supplementary Material）

1. **最大匹配的工程实现**：使用Python的NetworkX包[25]中的贪心算法搜索最大匹配。（来源：Section "Real-space RG on graphs"）

2. **密度矩阵重建的具体操作**：测量15个局域算符⟨σᵅ_i σᵝ_j⟩的期望值（α, β = 0, 1, 2, 3），通过ρ(k) = Σ_{α,β} ⟨σᵅ_i σᵝ_j⟩ σᵅ_i σᵝ_j重建约化密度矩阵。（来源：Section "Real-space RG on graphs"）

3. **基底的两种可能形式及其交替使用**：在构造二维子空间时，基底有两种有效形式：
   - 形式(a): {|0⟩|ψ₀⟩, |1⟩|ψ₁⟩}
   - 形式(b): {|φ₀⟩|0⟩, |φ₁⟩|1⟩}
   
   在MQAOA的循环中，两种形式交替使用（除第一轮中选择截断误差更小的形式）。详细的构造步骤在补充材料中给出，包含5个具体步骤：分块对角化、奇异值分解、线性组合近似解构造、归一化、酉变换还原。（来源：Supplementary Material；Section "Real-space RG on graphs"）

4. **多种最大匹配的尝试策略**：数值实验发现性能比可能依赖于特定的划分方案（partition scheme），因此每个粗粒化步骤中测试多种最大匹配以获取更好解。为了获得不同的匹配，在运行匹配搜索算法前先将图的顶点随机打乱（randomly shuffle）。（来源：Section "Multiscale QAOA" 数值实验讨论段落）

5. **非整数耦合系数的处理**：对于哈密顿量具有非整数耦合系数的情况，需要在搜索最大匹配前排除权重极小的边，因为由这些边连接的qubit对之间几乎无相互作用，在低深度QAOA首轮后该子系统几乎处于最大混合态，将其纳入匹配不会为粗粒化提供有用信息。（来源：Section "Multiscale QAOA" 末尾段）

6. **初始态的制备**：每轮的初始态W|ψ⁽¹⁾⟩是局域态的直积（tensor product of local states），仅需单量子比特门即可高精度制备。根据粗粒化序列中下一级有效哈密顿量的基态，匹配边连接的qubit对被制备为两个基向量之一（均为乘积态），未匹配的qubit制备为|0⟩或|1⟩。（来源：Section "Multiscale QAOA"）

7. **多级粗粒化的终止策略**：通过迭代生成哈密顿量序列{H⁽ⁿ⁾_z}，直到某级n₀的哈密顿量H⁽ⁿ⁰⁾_z可由量子或经典计算机有效求解。数值模拟表明所需RG变换步数约为O(log₂ N)。（来源：Section "Multiscale QAOA" 及 "Computational complexity"）

8. **对称性破缺基态的选择**：当H⁽ⁿ⁰⁾_z由于对称性存在多个基态时，选择一个乘积态（product state），该对称性破缺态（symmetry broken state）确保算法克服对称性保护的限制。（来源：Section "Multiscale QAOA"）

### 帮助QAOA落地的技术创新

（来源：Introduction、Conclusion、"Multiscale QAOA"）

1. **仅需低深度QAOA**：MQAOA只需在每个粗粒化步骤中运行低深度QAOA（depth-1或depth-2），这直接规避了标准QAOA面临的两大制约：(a) 当前NISQ设备有限的相干时间（coherence time）——高深度QAOA的ansatz态制备和测量必须在相干时间内完成；(b) 过多参数带来的贫瘠高原（barren plateaus）问题[15]。（来源：Introduction）

2. **初始态的高精度制备**：由于初始态W|ψ⁽¹⁾⟩是局域量子态的直积（仅需单qubit门），可以在NISQ设备上以高精度制备，降低了对门保真度的要求。（来源：Section "Multiscale QAOA" 末尾段）

3. **适合在NISQ设备上展示量子优势**：MQAOA通过低深度QAOA与经典后处理（RG变换）的结合，在对量子资源需求有限的前提下仍能获得精确解，为在NISQ设备上展示量子计算优势（quantum advantage）提供了可行途径。（来源：Abstract、Conclusion）

4. **经典优化不涉及大量参数**：低深度QAOA意味着需要优化的变分参数（β_k, γ_k, k=1,...,p）数量少，减少了经典优化器的负担。（来源：Introduction 中关于barren plateaus的讨论）

### 对QAOA实际应用的扩展与探索

（来源：Section "Multiscale QAOA" 数值实验部分）

1. **循环图（Cycle Graph / Ring of Disagrees）上的MaxCut问题**：使用depth-1和depth-2 QAOA，经过一步RG粗粒化后，有效哈密顿量由经典算法精确求解。结果表明：
   - QAOA和RG变换在每一轮都改善性能
   - 约6轮即可获得接近精确解的结果
   - depth-2 QAOA收敛更快（depth-1: 约0.0008误差；depth-2: 收敛更快）
   - MQAOA的性能比随轮数收敛到1的速度远快于标准QAOA通过增加深度收敛的速度
   
   已知标准深度-p QAOA对偶数N个顶点的循环图的最优近似比上界为(2p+1)/(2p+2)，仅在p ≥ N/2时才能获得精确解，而MQAOA仅用depth-1或depth-2就在O(log N)轮内收敛到1。（来源：Section "Multiscale QAOA" 循环图部分，图2(a)）

2. **3-正则随机图（Random 3-regular Graphs）**：40个顶点，测试了一级和两级粗粒化。在每个粗粒化步骤中仅采用depth-1 QAOA即可获得精确解，所有实例的近似比均约在6轮内收敛到1。（来源：Section "Multiscale QAOA"，图2(b)）

3. **连通Erdos-Renyi随机图**：20个顶点，边密度ρ = 0.1。一级和两级粗粒化均进行了测试，同样每个步骤仅用depth-1 QAOA，约6轮收敛到1。精确最大能量使用混合整数线性规划（mixed-integer linear programming）[29]计算。（来源：Section "Multiscale QAOA"，图2(c)）

4. **不同图类型上的RG变换效率**：数值模拟表明（图3(a)）：
   - 对于Erdos-Renyi图（尤其是高边密度），每步粗粒化顶点数减少近一半
   - 对于正则图，有效图可能趋向"星形图"（star-shaped graph），顶点数相对较多，但对应的哈密顿量基态仍容易求解
   - RG变换步数约为O(log₂ N)
   - 有效图的平均度在图3(b)中展示

5. **测试了depth-1和depth-2两种低深度配置**，均展示出良好性能。（来源：全文数值实验）

### QAOA对其他领域的启发式贡献

（来源：Introduction、Conclusion）

1. **将实空间重整化群方法从传统格点系统扩展到一般图上的量子系统**。这一推广不仅在QAOA的语境中有意义，也为将RG方法应用于其他基于图的量子-经典混合算法提供了范本。文中提到DMRG [17,18] 已成功用于低维格点量子系统的静态、动力学和热力学量计算[19]，纠缠重整化 [20-22] 扩展到高维格点系统，而本文的方案则推广到一般图结构。（来源：Introduction）

2. **可以与QAOA领域的其他改进算法进行二次结合**（论文提到MQAOA can be merged with other improved algorithms），暗示了该框架具有通用性和可扩展性，不仅适用于标准QAOA，也可作为其他QAOA变体的性能增强模块。（来源：Conclusion）

---

## 面临的困难、论文的不足之处和后续的改进方向

（来源：Section "Multiscale QAOA" 数值实验讨论部分、Conclusion）

1. **深度-1 QAOA可能导致算法陷入次优解（suboptimal solution）**。数值实验发现，对于某些图，当使用depth-1 QAOA时，算法容易卡在局部最优。论文提出了两种改进途径：(a) 增加QAOA深度是一种可行方法；(b) 尝试多种最大匹配划分方案以改善结果。（来源：Section "Multiscale QAOA" 数值实验讨论段）

2. **性能比对划分方案的依赖性**。性能比在很大程度上依赖于特定的划分方案（即选择哪个最大匹配），因此需要在每个粗粒化步骤中测试多种匹配。这增加了算法的经典计算开销。（来源：Section "Multiscale QAOA" 数值实验讨论段）

3. **非整数耦合哈密顿量的额外处理需求**。当哈密顿量具有非整数耦合系数时，需要额外排除权重过小的边后再搜索最大匹配，增加了预处理步骤的复杂度。（来源：Section "Multiscale QAOA" 末尾段）

4. **论文未讨论的问题**：
   - 缺乏对噪声环境下的性能分析和鲁棒性研究（尽管论文声称适用于NISQ设备，但所有数值实验均为无噪声的经典模拟）
   - 缺乏对更多类型组合优化问题的测试（仅测试了MaxCut）
   - 未给出严格的理论收敛性证明，收敛性结论基于数值观察
   - RG变换中基态求解的质量对整体算法性能的影响未做系统的误差传播分析
   - 未讨论粗粒化步骤中测量误差对密度矩阵重建精度的影响

5. **后续研究方向的暗示**（来源：Conclusion）：论文提到MQAOA可与其他改进算法结合（"can be merged with other improved algorithms"），暗示后续工作可探索MQAOA框架与QAOA其他变体（如recursive QAOA、warm-start QAOA等）的融合。

---

## 真机性能

本文为纯理论和数值模拟研究，**未在真实量子计算机上运行实验**。所有性能数据均来自经典计算机上的数值模拟：

- 循环图（cycle graph）上的模拟利用平移对称性（translational symmetry），结果适用于任意偶数N
- 随机正则图（40个顶点）和Erdos-Renyi图（20个顶点）各取100个随机实例取均值
- RG变换效率的数据（图3）取自200个随机生成的图取均值
- 所有精确最大能量（基准值）由混合整数线性规划方法[29]计算
- 经典优化采用单纯形法或基于梯度的优化方法，配合初始猜测参数

尽管论文在摘要和结论中多次强调该算法"适用于NISQ设备展示量子优势"（suitable for NISQ devices to exhibit a quantum advantage），但实际量子硬件上的性能验证仍是一个待填补的空白。
