# [预测代理] Demonstration of Efficient Predictive Surrogates for Large-Scale Quantum Processors — 论文总结

> **文献信息**
> - **标题**：Demonstration of efficient predictive surrogates for large-scale quantum processors
> - **作者**：Wei-You Liao, Yuxuan Du, Xinbiao Wang (共同一作), Tian-Ci Tian, Yong Luo, Bo Du, Dacheng Tao, He-Liang Huang (通讯)
> - **机构**：河南省量子信息与密码学重点实验室 / 南洋理工大学 / 武汉大学
> - **发表**：*Nature Communications* (2026) 17:4731
> - **DOI**：https://doi.org/10.1038/s41467-026-72506-5
> - **日期**：Received 2025-09-27 / Accepted 2026-04-17

---

## 目录

- [一、创新性贡献](#一创新性贡献)
- [二、问题建模](#二问题建模)
- [三、预测代理 $h_{cs}$：经典影子预测器](#三预测代理-hcs经典影子预测器)
- [四、预测代理 $h_{qs}$：量子模拟代理](#四预测代理-hqs量子模拟代理)
- [五、实验验证](#五实验验证)
- [六、目前的不足](#六目前的不足)
- [七、未来改进方向](#七未来改进方向)
- [八、总体评价](#八总体评价)

---

## 一、创新性贡献

1. **提出预测代理（Predictive Surrogate）的概念框架**——用经典学习模型模拟量子处理器的**均值行为** $f(\tilde{\rho}(\mathbf{x}), O) = \text{Tr}(\tilde{\rho}(\mathbf{x}) O)$，且提供**可证明的计算效率保证**（provable guarantees）。这与纯启发式的深度学习代理（无理论保证）和理想量子电路的经典代理（不适用于含噪场景）有本质区别。

2. **提出两种互补的预测代理**：
   - $h_{cs}$（classical shadow predictor）：适用于**独立可调 $R_Z$ 门 + 多可观测量**场景，支持任意 $K$-局域有界可观测量
   - $h_{qs}$（quantum simulation surrogate）：适用于**关联参数 + 固定可观测量**场景，支持任意输入分布

3. **在 42-qubit 可编程超导量子处理器上完成实验验证**（芯片为 6×11 阵列，66 qubits 总计），证明：
   - VQE 预训练：仅用**0.023% 的测量资源**达到优于原始 VQE 的基态能量精度
   - 标度实验：$h_{cs}$ 扩展至 20 qubits（1D TFIM），$h_{qs}$ 扩展至 42 qubits（2D TFIM），性能与量子比特数**几乎无关**
   - Floquet 对称保护拓扑（FSPT）相识别：20 × 79 个代理成功定位相变临界点 $\delta^* \approx 0.172$

4. **两个核心定理的证明**：
   - Theorem 1：样本复杂度 $n$ 不显式依赖于量子比特数 $N$，在 Pauli 噪声下多项式标度
   - Theorem 2：当 Pauli 噪声参数满足 $q(1+R) < 1/e$ 时，$h_{qs}$ 可高效实现

---

## 二、问题建模

### 2.1 量子电路与噪声模型

考虑由 Clifford 门与 $R_Z$ 旋转门组成的量子电路（式 (1)）：

$$U(\mathbf{x}) = \prod_{l=1}^{d} \left(R_Z(x_l) V_l\right)$$

- $\mathbf{x} \in [-\pi, \pi]^d$：$d$ 维可调参数（$R_Z$ 旋转角）
- $V_l$：Clifford 操作，由 $\{H, S, \text{CNOT}\}$ 门构成
- $R_Z(x_l) = e^{-i x_l \sigma_z / 2}$：绕 $z$ 轴旋转 $x_l$ 的单量子比特门

**噪声模型**（式 (1)）：经 Pauli 旋转（twirling）后，含噪电路信道为
$$\tilde{U}(\mathbf{x}) = \bigcirc_{l=1}^{d} \tilde{R}_Z(x_l) \circ \tilde{V}_l$$

其中 $\tilde{R}_Z(x_l) = \mathcal{N}_P \circ R_Z(x_l)$，$\tilde{V}_l = \mathcal{M} \circ V_l$。
- $\mathcal{N}_P(\rho) = (1 - \Sigma_p)\rho + p_X X\rho X + p_Y Y\rho Y + p_Z Z\rho Z$：单量子比特 Pauli 噪声信道（$p_X, p_Y, p_Z$ 为噪声参数，$\Sigma_p = p_X + p_Y + p_Z$）
- $\mathcal{M}$：多量子比特 Pauli 信道

量子处理器的**均值行为**由下式刻画（式 (2)）：
$$f(\tilde{\rho}(\mathbf{x}), O) = \text{Tr}(\tilde{\rho}(\mathbf{x}) O)$$

- $\tilde{\rho}(\mathbf{x})$：初态 $\rho_0$ 经含噪电路 $\tilde{U}(\mathbf{x})$ 演化后的量子态
- $O$：可观测量（observable）

### 2.2 预测代理的目标

给定仅**有限次**访问量子处理器收集的训练数据，构建经典模型 $h(\mathbf{x}, O)$，使其在新输入 $\mathbf{x}'$ 上的预测误差满足（式 (3)）：

$$R(h) = \mathbb{E}_{\mathbf{x} \sim \mathcal{D}}\left|h(\mathbf{x}, O) - f(\tilde{\rho}(\mathbf{x}), O)\right|^2 \leq \epsilon$$

- $\mathcal{D}$：输入数据分布
- $\epsilon$：目标预测误差

代理 $h$ 被称为**计算高效**的，若其训练时间关于量子比特数 $N$ 和输入维度 $d$ 均为多项式，且以高概率满足 $R(h) \leq \epsilon$。

---

## 三、预测代理 $h_{cs}$：经典影子预测器

### 3.1 适用场景

- $R_Z$ 门参数**独立可调**
- 支持**多个** $K$-局域（$K$-local）有界可观测量 $O$（$\|O\|_\infty \leq B$）
- 输入 $\mathbf{x}$ 从 $[-\pi, \pi]^d$ 上**均匀分布**采样

### 3.2 实现步骤

**步骤 1（数据收集）**：对 $n$ 个不同输入 $\mathbf{x}^{(i)} \sim \text{Unif}[-\pi, \pi]^d$，在量子处理器上各执行 $T$ 次快照（snapshot）的 Pauli 基经典影子测量 [55]，得到 $\tilde{\rho}_T(\mathbf{x}^{(i)})$。训练集 $\mathcal{T}_{cs} = \{(\mathbf{x}^{(i)}, \tilde{\rho}_T(\mathbf{x}^{(i)}))\}_{i=1}^{n}$。

- $\tilde{\rho}_T(\mathbf{x}^{(i)})$：从 $T$ 次 Pauli 测量快照重构的经典影子表示（$\mathcal{O}(T \cdot 3^K)$ 存储）

**步骤 2（模型构建）**：对新的 $\mathbf{x}'$，预测的经典影子为（式 (4)）：

$$\hat{\sigma}_n(\mathbf{x}') = \frac{1}{n} \sum_{i=1}^{n} \kappa_\Lambda(\mathbf{x}', \mathbf{x}^{(i)}) \cdot \tilde{\rho}_T(\mathbf{x}^{(i)})$$

核函数 $\kappa_\Lambda$ 为**截断三角函数单项式核**（truncated trigonometric monomial kernel）：

$$\kappa_\Lambda(\mathbf{x}, \mathbf{x}^{(i)}) = \sum_{\omega, \|\omega\|_0 \leq \Lambda} 2^{\|\omega\|_0} \Phi_\omega(\mathbf{x}) \Phi_\omega(\mathbf{x}^{(i)})$$

- $\Phi_\omega(\mathbf{x}) = \prod_{i=1}^{d} \left[ \mathbf{1}_{\omega_i=0} + \mathbf{1}_{\omega_i=1} \cos(x_i) + \mathbf{1}_{\omega_i=-1} \sin(x_i) \right]$：三角函数单项式基
- $\omega \in \{0, \pm 1\}^d$：频率向量，$\|\omega\|_0$ 为非零分量个数（稀疏度）
- $\Lambda$：截断阈值（频率稀疏度上界）
- $\mathcal{C}(\Lambda) = \{\omega \mid \omega \in \{0, \pm 1\}^d, \|\omega\|_0 \leq \Lambda\}$：截断频率集

对可观测量 $O$ 的预测（式 (5)）：
$$h_{cs}(\mathbf{x}', O) = \frac{1}{n} \sum_{i=1}^{n} \kappa_\Lambda(\mathbf{x}', \mathbf{x}^{(i)}) \cdot \text{Tr}(\tilde{\rho}_T(\mathbf{x}^{(i)}) O)$$

### 3.3 Theorem 1（计算效率保证）

**设定**：$\mathbf{x} \sim \text{Unif}[-\pi, \pi]^d$，Pauli 噪声 $\mathcal{N}_P(p_X, p_Y, p_Z)$，记 $p = \min\{p_X, p_Y\}$，$\mathbb{E}_\mathbf{x} \|\nabla_\mathbf{x} \text{Tr}(\tilde{\rho}(\mathbf{x}) O)\|_2^2 \leq C$。

当 $\Lambda = 4C/\epsilon$，训练样本数
$$n = \tilde{\Omega}\left( \min\left\{\frac{4C}{\epsilon}, \frac{1}{2(p + p_Z)}\right\} \cdot \log \frac{2B}{\sqrt{\epsilon}} \cdot \frac{2^{9K}}{\epsilon} \right)$$

时，以概率 $\geq 1 - \delta$ 有 $R(h_{cs}) \leq \epsilon$，且**仅需 $T = 1$ 次快照/样本**。

**关键洞察**：
- $n$ 不显式依赖于量子比特数 $N$——仅通过梯度范数上界 $C$ 隐含
- **更高的 Pauli 噪声反而降低样本需求**——噪声增大 → $1/(2(p+p_Z))$ 减小 → $n$ 减小。这与噪声诱导的贫瘠高原（barren plateau）效应一致 [58]：强噪声使景观变平坦，代理只需少量样本即可拟合
- $C$ 通常随 $N$ 和电路深度**减小**（经验观察 [44]）——意味着更大系统反而**更易**学习

> 📌 收集的经典影子 $\{\tilde{\rho}_T(\mathbf{x}^{(i)})\}$ **隐式编码了量子处理器的噪声特征**——代理无需显式表征 Pauli 噪声参数即可准确模拟含噪行为。这是区别于经典模拟器（需昂贵的噪声建模）的关键优势。

---

## 四、预测代理 $h_{qs}$：量子模拟代理

### 4.1 适用场景

- $R_Z$ 门参数**存在关联**（如由物理哈密顿量决定）
- **固定**可观测量 $O$
- 输入 $\mathbf{x}$ 从任意分布采样，范围有界：$[-R, R]^d$

### 4.2 实现步骤

**步骤 1（数据收集）**：收集 $\mathcal{T}_{qs} = \{(\mathbf{x}^{(i)}, y^{(i)})\}_{i=1}^{n}$，其中 $y^{(i)}$ 为 $T$ 次 shots 估计的均值，满足 $\max_i |y^{(i)} - \text{Tr}(\tilde{\rho}(\mathbf{x}^{(i)}) O)| \leq \epsilon_l$。

**步骤 2（模型构建）**：代理为截断三角函数单项式特征的线性模型（式 (6)）：

$$h_{qs}(\mathbf{x}, \hat{\mathbf{w}}) = \langle \Phi_{\mathcal{C}(\Lambda)}(\mathbf{x}), \hat{\mathbf{w}} \rangle$$

- $\Phi_{\mathcal{C}(\Lambda)} = [\Phi_\omega(\mathbf{x})]_{\omega \in \mathcal{C}(\Lambda)}$：$|\mathcal{C}(\Lambda)|$ 维特征向量
- $\hat{\mathbf{w}}$：通过**岭回归（Ridge Regression）**求解：
  $$\min_{\mathbf{w}} \frac{1}{n} \sum_{i=1}^{n} (y^{(i)} - h_{qs}(\mathbf{x}^{(i)}, \mathbf{w}))^2 + \lambda \|\mathbf{w}\|^2$$
- $\lambda$：正则化参数

### 4.3 Theorem 2（计算效率保证）

**设定**：$q = 1 - 2(p + p_Z)$，$\epsilon = 16 B^2 (d e q (1+R) / \Lambda)^{2\Lambda}$，$\epsilon_l \leq \sqrt{\epsilon}/4$。

当 $q(1+R) < 1/e$ 且 $\Lambda > d e q (1+R)$ 时，训练样本数
$$n = \frac{1}{q(1+R)} \left( \frac{4 d e q (1+R)}{\log(1/\delta)} \right)^9$$

以概率 $\geq 1 - \delta$ 有 $R(h_{qs}) \leq \epsilon$。

**关键差异 vs $h_{cs}$**：
- 额外条件 $q(1+R) < 1/e$：要求噪声不能太低（$q$ 较小）或输入范围不能太大（$R$ 较小）——这是处理关联参数付出的代价
- 若 $q > \mathcal{O}(1/d)$（低噪声），需将 $R$ 进一步缩小（见 SI D 中的替代方案）

---

## 五、实验验证

### 5.1 实验平台

- **42 个活跃量子比特**（从 66-qubit 可编程超导处理器中选取，6×11 阵列，频率可调 transmon + 可调耦合器）
- 中位 $T_1 = 32.3\ \mu\text{s}$；单比特门中位错误率 0.11%；CZ 门中位错误率 1.09%
- 经典端：NVIDIA A100 GPU（80GB HBM2）+ JAX v0.4.30 + FP64

### 5.2 $h_{cs}$ 预训练 VQE（1D TFIM）

**哈密顿量**：$H_{\text{TFIM}} = -J \sum_i Z_i Z_{i+1} - h \sum_i X_i$

**Ansatz**（Trotter 分解，$L=1$ 层）：
$$U(\mathbf{x}) = \prod_{i=1}^{N} R_{X_i}(x_{1,i}) \prod_{i=1}^{N-1} R_{ZZ_{i,i+1}}(x_{1,N+i})$$

输入维度 $d = L(2N-1)$。

**性能度量**（归一化偏差）：
$$R(\mathbf{x}) = \frac{|f(\tilde{\rho}(\mathbf{x}), H_{\text{TFIM}}) - E_0(H_{\text{TFIM}})|}{E_{\max}(H_{\text{TFIM}}) - E_0(H_{\text{TFIM}})}$$

| 实验 | 规模 | 结果 |
|------|------|------|
| $N=6, J=-0.1, h=-0.5$（Fig. 2a–c） | $n=2000, T=10, \Lambda=2$ | $R$ 从初始 0.48 → 预训练后 **0.09**（vs 原始 VQE 的 0.21） |
| 多哈密顿量（$J \in \{-0.5, \ldots, 0.5\}$，Fig. 2e） | $n=2000$ | 对所有 $J$ 一致改善，$n=1500$ 后性能饱和 |
| 测量资源对比 | — | 仅用原始 VQE 的 **0.023%** 测量量 |
| 微调（fine-tuning，Fig. 2c） | Post-train | $R$ 从 0.09 → 0.07（边际改善，说明预训练已接近最优） |

**标度实验**（Fig. 3a）：$h_{cs}$ 在 $N \in \{8, 12, 16, 20\}$（1D TFIM）上，固定 $n=1024$——预训练后 $R$ 与 $N$ **几乎无关**。

### 5.3 $h_{qs}$ 预训练 VQE（2D TFIM）

**标度实验**（Fig. 3b）：$h_{qs}$ 在 $N \in \{18, 24, 30, 36, 42\}$（2D 正方晶格 TFIM）上，固定 $n=160$——预训练后 $R$ 同样与 $N$ 几乎无关。

### 5.4 FSPT 相识别（20-qubit 时间周期哈密顿量）

**哈密顿量**（式 (8)）：
$$H_{\text{tp}}(t) = \begin{cases} H_{\text{tp},1} = (\frac{\pi}{2} - \delta) \sum_i X_i, & 0 \leq t < T_1 \\ H_{\text{tp},2} = -\sum_i J_i Z_{i-1} X_i Z_{i+1}, & T_1 \leq t < 2T_1 \end{cases}$$

- $P_i \in \{X, Y, Z\}$：第 $i$ 个量子比特上的 Pauli 算符
- $\delta$：驱动扰动参数（控制相变）
- $J_i$：无序耦合参数
- $T_1, 2T_1$：驱动周期

**方法**：构建 $20 \times 79$ 个预测代理 $\{h_{qs}^{(i,t)}\}$，每个对应一个量子比特 $i$ 和一个驱动时间 $kT_1$，各用 $n=800, T=10,000$ 训练。

**结果**（Fig. 4）：
- $\delta = 0.01$（FSPT 相）：边界自旋（Q1, Q20）的局域磁化 $\langle Z_i \rangle$ 呈现稳定的**次谐波振荡**（subharmonic oscillation，$\omega/\omega_0 = 0.5$），体自旋迅速衰减
- $\delta = 0.8$（热化相）：所有自旋的磁化均在几个周期内衰减至零
- 傅里叶谱（Fig. 4b）：FSPT 相边界自旋锁定在 $\omega/\omega_0 = 0.5$，热化相无次谐波峰
- 相变临界点（Fig. 4c）：次谐波峰高的方差在 **$\delta^* \approx 0.172$** 处最大——与量子处理器上的小规模验证一致 [50]

---

## 六、目前的不足

### 6.1 仅支持 Clifford + $R_Z$ 电路结构

两个代理的三角函数单项式基展开依赖于 $U(\mathbf{x})$ 的 Clifford + $R_Z$ 结构。对于包含任意旋转角（如 $R_X, R_Y$ 混合）或非 Clifford 门的电路，定理的保证不再成立。论文在 Remark 中提到可扩展到任意 Pauli 旋转门，但未提供实验验证。

### 6.2 $h_{cs}$ 仅支持局域可观测量

Theorem 1 要求 $O$ 为 $K$-局域（$K$-local）且有界（$\|O\|_\infty \leq B$）。对于全局可观测量（如哈密顿量的全和形式 $H = \sum_i Z_i Z_{i+1}$ 本身是局域的加权和，这不构成限制），但真正全局的算符（如投影到某特定本征态）无法直接支持。

### 6.3 $h_{qs}$ 的噪声条件限制

Theorem 2 要求 $q(1+R) < 1/e$——即噪声不能太低、或参数范围不能太大。对于高保真度量子处理器（$p, p_Z \to 0 \Rightarrow q \to 1$），$h_{qs}$ 的效率保证退化，需缩小 $R$（见 SI D 的替代方案）或采用其他代理类型。

### 6.4 仅模拟均值行为，不支持分布采样

代理输出标量均值 $\text{Tr}(\tilde{\rho}(\mathbf{x}) O)$，而非从 $\tilde{\rho}(\mathbf{x})$ 采样的比特串分布。论文在 Discussion 中明确指出，扩展到分布级代理是重要的未来方向——因为许多具有量子优势的算法（Shor、Grover、随机电路采样）依赖采样而非均值估计。

### 6.5 三角函数单项式基的维度诅咒

特征空间 $|\mathcal{C}(\Lambda)| = \sum_{w=0}^{\Lambda} \binom{d}{w} 2^w = \mathcal{O}(d^\Lambda)$——随输入维度 $d$ 多项式增长，但指数依赖于截断阈值 $\Lambda$。对于需要大 $\Lambda$ 的高频信号（如深层电路的快速振荡均值函数），特征维度可能爆炸。实验中 $\Lambda = 2$（$h_{cs}$）和 $\Lambda = 25$（$h_{qs}$），后者在 $d$ 大时可能有挑战。

### 6.6 FSPT 实验中仅测试了边界相的次谐波签名

相图通过 $\langle Z_1 \rangle$ 的次谐波峰高方差构建——这是一种**单一观测量的代理**。全相图（包括关联函数、纠缠熵等更丰富的序参量）需要大量代理扩展。

---

## 七、未来改进方向

### 7.1 扩展电路类型支持

- 任意 Pauli 旋转门（$R_X, R_Y, R_Z$ 混合）的代理构造与实验验证
- 包含对称性的量子电路（如 UCCSD ansatz [65]）——利用对称性进一步降低样本复杂度
- 含中电路测量和经典反馈的动态电路

### 7.2 非线性函数代理

将代理从 $\text{Tr}(\tilde{\rho} O)$ 扩展到：
- 纠缠熵（Rényi-2）、稳定子熵
- 态纯度 $\text{Tr}(\rho^2)$
- 保真度 $\langle \psi_{\text{target}} | \rho | \psi_{\text{target}} \rangle$

### 7.3 分布级代理

用生成模型（如 VAE、扩散模型）学习 $\tilde{\rho}(\mathbf{x})$ 的输出比特串分布——使代理可应用于随机电路采样等场景。

### 7.4 深度学习代理的理论化

将 Theorem 1–2 的理论保证框架扩展到深度神经网络代理，解释为什么 [78–80] 中的深度学习方法在经验上成功。

### 7.5 主动学习与自适应采样

当前 $\mathbf{x}$ 从固定分布采样。能否用贝叶斯优化或不确定性量化选择**最有信息量的 $\mathbf{x}$** 来标注，以进一步减少量子处理器访问次数？

---

## 八、总体评价

### 8.1 论文定位

这是**同系列文献中发表刊物级别最高的工作**（*Nature Communications* 2026），也是唯一在 **42-qubit 真实超导量子处理器**上完成大规模验证的工作（同组 He-Liang Huang 也是 CompShadow [1] 的通讯作者）。它将经典机器学习（三角函数单项式核回归）与量子处理器均值行为模拟严格结合，给出了**可证明效率 + 硬件实验**双重验证——这在量子计算与 AI 交叉领域是罕见的高标准。

### 8.2 与同系列文献的对比

| 维度 | [1] CompShadow | [2] BP-OSD | **本文 [Liao et al.]** |
|------|:--:|:--:|:--:|
| 发表刊物 | arXiv | EPJ ST | **Nat. Commun. 2026** |
| 方法论原创性 | ★★★★★ | ★★★ | ★★★★★ |
| 理论保证 | ★★★★ | ★★ | ★★★★★ |
| 实验规模 | 模拟器 | 仿真 ([[882,48]]) | **真实 42-qubit 芯片** |
| 真实量子硬件 | ✗ | ✗ | **✓（超导 66-qubit 芯片）** |
| 量子资源节省 | 场景特定 | 常数因子 | **~4300×（0.023% 测量量）** |
| 可扩展性证明 | 仅仿真 | 仅两码型 | **20–42 qubits 实验标度** |

### 8.3 一句话总结

> 本文用三角函数单项式核回归这一经典 ML 技术，在 42-qubit 真实超导量子处理器上证明了：只需经典影子测量的一次快照 + 多项式样本，即可构建一个无需再访问量子硬件的预测代理——在 VQE 预训练上仅用 0.023% 的测量资源就超越了原始 VQE 的性能。这是目前量子-AI 交叉领域最令人信服的"经典替量子"示范之一。

---

> 生成日期：2026-06-16
