# [误差缓解] Sample-Efficient Quantum Error Mitigation via Classical Learning Surrogates — 论文总结

> **文献信息**
> - **标题**：Sample-efficient quantum error mitigation via classical learning surrogates
> - **作者**：Wei-You Liao, Ge Yan, Yujin Song, Tian-Ci Tian, Wei-Ming Zhu, De-Tao Jiang, Yuxuan Du (通讯), He-Liang Huang (通讯)
> - **机构**：河南省量子信息与密码学重点实验室 / 南洋理工大学
> - **发表**：Research Square 预印本, 2026-02-06
> - **DOI**：https://doi.org/10.21203/rs.3.rs-8659832/v1
> - **备注**：与 Nat. Commun. 2026 预测代理论文为同一团队，方法高度延续

---

## 目录

- [一、创新性贡献](#一创新性贡献)
- [二、背景：ZNE 的测量开销瓶颈](#二背景zne-的测量开销瓶颈)
- [三、S-ZNE 方法](#三s-zne-方法)
  - [3.1 框架概述](#31-框架概述)
  - [3.2 回归型代理（关联参数）](#32-回归型代理关联参数)
  - [3.3 核型代理（独立参数）](#33-核型代理独立参数)
  - [3.4 Theorem 1（性能保证）](#34-theorem-1性能保证)
- [四、实验验证](#四实验验证)
  - [4.1 基态能量估计（100-qubit TFIM 与 Heisenberg 模型）](#41-基态能量估计100-qubit-tfim-与-heisenberg-模型)
  - [4.2 量子计量（100-qubit GHZ Ramsey 干涉）](#42-量子计量100-qubit-ghz-ramsey-干涉)
- [五、目前的不足](#五目前的不足)
- [六、总体评价](#六总体评价)

---

## 一、创新性贡献

1. **提出 S-ZNE（Surrogate-enabled Zero Noise Extrapolation）框架**——将经典学习代理与零噪声外推（ZNE）结合，实现对**一族参数化量子电路**误差缓解的测量开销从线性降为常数。核心创新在于"数据采集与量子执行解耦"：训练代理时一次性采集量子数据，之后所有 ZNE 外推在经典端完成。

2. **提供两种互补的代理实现**：
   - **回归型代理**（ridge regression + 三角函数单项式特征）：用于**关联参数**电路
   - **核型代理**（truncated trigonometric kernel + classical shadows）：用于**独立参数**电路
   这两种实现直接沿用同一团队预测代理论文 [Liao et al., Nat. Commun. 2026] 的 $h_{qs}$ 和 $h_{cs}$ 架构。

3. **Theorem 1 给出 S-ZNE 与常规 ZNE 的误差可比性证明**：在相同测量预算 $M$ 下，S-ZNE 的平均预测误差上界与常规 ZNE 一致——均为 $\zeta^2 + \mathcal{O}(L^2 u B^2 / M)$，其中 $\zeta^2$ 为外推函数选择导致的固有误差。

4. **在 100-qubit 规模上完成数值验证**：基态能量估计（TFIM + Heisenberg 模型）和量子计量（GHZ Ramsey 干涉），S-ZNE 以**恒定**测量开销（vs 常规 ZNE 的线性增长）达到可比的误差缓解精度。

---

## 二、背景：ZNE 的测量开销瓶颈

### 2.1 参数化量子电路

考虑一族由经典参数 $\mathbf{x} \in [0, 2\pi]^d$ 控制的量子电路（式 (1)）：

$$U(\mathbf{x}) = \left(\prod_{j=1}^{d} C_j R_Z(x_j)\right) C_0$$

- $C_j$：第 $j$ 层的 Clifford 门组合（由 $\{H, S, \text{CNOT}\}$ 构成）
- $R_Z(x_j) = e^{-i x_j \sigma_z / 2}$：$Z$ 轴旋转门
- $\rho(\mathbf{x}) = U(\mathbf{x}) \rho_0 U(\mathbf{x})^\dagger$：无噪输出态

含噪输出（式 (2)）：
$$f(\mathbf{x}, O, \lambda) = \text{Tr}\left(\mathcal{N}_\lambda(\rho(\mathbf{x})) O\right)$$

- $\mathcal{N}_\lambda(\cdot)$：Pauli 噪声信道（经 Pauli twirling 转化），$\lambda$ 为噪声水平
- $\lambda = 0$ 对应无噪声极限：$f(\mathbf{x}, O) \equiv f(\mathbf{x}, O, 0)$

### 2.2 传统 ZNE 的代价

ZNE 对每个输入 $\mathbf{x}$ 需要在 $u$ 个放大的噪声水平 $\{\lambda_j\}_{j=1}^{u}$（$\lambda_j < \lambda_{j+1}$，通过幺正折叠 [72] 实现）上各采集 $M$ 次测量，再通过外推函数 $g(\cdot)$ 估计零噪声极限（式 (3)）：

$$g(z_C(\mathbf{x})) \equiv g\left([\hat{f}(\mathbf{x}, O, \lambda_1), \ldots, \hat{f}(\mathbf{x}, O, \lambda_u)]\right)$$

**总测量开销** $=$ **$\#\{\mathbf{x}\} \times u \times M$** ——线性依赖于电路数量。

---

## 三、S-ZNE 方法

### 3.1 框架概述

S-ZNE 的核心思想（Fig. 1）：在 $u$ 个噪声水平上各训练一个经典学习代理 $h(\mathbf{x}, O, \lambda_j)$，训练完成后，对任何新输入 $\mathbf{x}'$ 的 ZNE 估计完全在经典端完成：

$$g(z_S(\mathbf{x}')) = g\left([h(\mathbf{x}', O, \lambda_1), \ldots, h(\mathbf{x}', O, \lambda_u)]\right)$$

三步骤：**数据采集 → 代理建模 → 经典外推**。

### 3.2 回归型代理（关联参数）

适用于参数存在**分组关联**的电路（如 VQE 中同一层的 $R_Z$ 旋转角共享参数）。

对噪声水平 $\lambda_j$，训练集 $\mathcal{T}(\lambda_j) = \{(\mathbf{x}^{(i,j)}, y^{(i,j)})\}_{i=1}^{n_j}$，代理模型为（式 (5)）：

$$h(\mathbf{x}, O, \lambda_j) = \langle \Phi_{\mathcal{C}(\Lambda)}(\mathbf{x}), \mathbf{w}_j \rangle$$

- $\mathcal{C}(\Lambda) = \{\omega \in \{0, \pm 1\}^d \mid \|\omega\|_0 \leq \Lambda\}$：截断频率集，$\Lambda$ 为稀疏度阈值
- $\Phi_{\mathcal{C}(\Lambda)}(\mathbf{x})$：三角函数单项式特征向量

**分组特征映射**（式 (6)）——当参数分 $S$ 组，每组内参数相同：
$$\Phi_\omega(\mathbf{x}) = \prod_{s=1}^{S} \left[ \cos(x_s)^{N_s^+(\omega)} \cdot \sin(x_s)^{N_s^-(\omega)} \right]$$

- $N_s^+(\omega) = \sum_{k=1}^{d_s} \mathbf{1}_{\{\omega_{s,k}=1\}}$：第 $s$ 组中 $\cos$ 项计数
- $N_s^-(\omega) = \sum_{k=1}^{d_s} \mathbf{1}_{\{\omega_{s,k}=-1\}}$：第 $s$ 组中 $\sin$ 项计数

权重 $\mathbf{w}_j$ 通过**岭回归**训练（式 (7)）：
$$\min_{\mathbf{w}_j} \left\{ \frac{1}{n_j} \sum_{i=1}^{n_j} \left(y^{(i,j)} - \langle \Phi_{\mathcal{C}(\Lambda)}(\mathbf{x}^{(i,j)}), \mathbf{w}_j \rangle\right)^2 + \gamma \|\mathbf{w}_j\|_2^2 \right\}$$

- $\gamma > 0$：正则化超参数

### 3.3 核型代理（独立参数）

适用于参数**独立可调**的电路（如每个 $R_Z$ 角独立变化）。

训练集 $\mathcal{T}(\lambda_j) = \{(\mathbf{x}^{(i,j)}, \tilde{\rho}_T(\mathbf{x}^{(i,j)}))\}_{i=1}^{n_j}$ 基于 **Pauli 经典影子** [55]，预测为（式 (8)）：

$$h(\mathbf{x}, O, \lambda_j) = \frac{1}{n_j} \sum_{i=1}^{n_j} \kappa_\Lambda(\mathbf{x}, \mathbf{x}^{(i,j)}) \cdot \text{Tr}\left(\tilde{\rho}_T(\mathbf{x}^{(i,j)}) O\right)$$

截断三角函数核（式 (9)）：
$$\kappa_\Lambda(\mathbf{x}, \mathbf{x}') = \sum_{\omega \in \mathcal{C}(\Lambda)} 2^{\|\omega\|_0} \Phi_\omega(\mathbf{x}) \Phi_\omega(\mathbf{x}')$$

特征函数（式 (10)）：
$$\Phi_\omega(\mathbf{x}) = \prod_{l=1}^{d} \begin{cases} 1 & \omega_l = 0 \\ \cos(x_l) & \omega_l = 1 \\ \sin(x_l) & \omega_l = -1 \end{cases}$$

### 3.4 Theorem 1（性能保证）

**设定**：$\mathbf{x} \in [-R, R]^d \sim \mathcal{D}$，$L$ 为外推函数 $g(\cdot)$ 的 Lipschitz 常数，$\zeta^2$ 为外推的固有误差。

**常规 ZNE**：用 $M$ 次测量估计每个 $\hat{f}(\mathbf{x}, O, \lambda_j)$，以概率 $\geq 1 - 0.05u$：
$$\mathbb{E}_\mathbf{x} |f(\mathbf{x}, O) - g(z_C(\mathbf{x}))|^2 \leq \zeta^2 + \mathcal{O}\!\left(\frac{L^2 u B^2}{M}\right)$$

**S-ZNE**：当 $\Lambda > d e q (1+R)$ 且 $n_j \geq \tilde{\mathcal{O}}(B^2 M (de/\Lambda)^{4\Lambda})$ 时，以概率 $\geq 1 - u\delta$：
$$\mathbb{E}_\mathbf{x} |f(\mathbf{x}, O) - g(z_S(\mathbf{x}))|^2 \leq \zeta^2 + \mathcal{O}\!\left(\frac{L^2 u B^2}{M}\right)$$

**关键含义**：
- S-ZNE 的误差上界与常规 ZNE **形式完全一致**
- $\zeta^2$ 项来自外推函数 $g(\cdot)$ 的选择（如线性外推 vs Richardson 外推）——两种方法共同承担
- 在实际场景中（$\Lambda$ 为小常数时，如强噪声或小 $R$），S-ZNE 所需训练样本 $n_j$ 是**多项式级**的

---

## 四、实验验证

### 4.1 基态能量估计（100-qubit TFIM 与 Heisenberg 模型）

**设置**：
- 哈密顿量变分拟设（Hamiltonian Variational Ansatz），100 qubits
- 噪声：全局退极化（$p_g=0.05$）+ 可选相干噪声（旋转角扰动 $\sim U[-0.01\lambda_j, 0.01\lambda_j]$）
- 外推：线性最小二乘拟合，$u=5$ 噪声水平
- 优化器：Adam，学习率 0.001，1500 轮
- 模拟器：TensorCircuit MPS backend + JAX + NVIDIA A100 GPU

**代理准确度**（Fig. 2a）：
- $n_j = 100$ 时，$\log(1+\text{MSE})$ 已接近平台
- 预测误差对噪声水平 $\lambda_j$ 仅有弱依赖（噪声越大反而略好——与预测代理论文的结论一致）

**误差缓解性能**（Fig. 2b）：
- S-ZNE 残差分布与常规 ZNE **紧密重叠**——均在零附近高度集中
- 两种噪声模型（DP、DP+CO）下均成立

**测量开销**（Fig. 2c）：
| 方法 | 总测量数（1500 轮 VQE） | 比例 |
|------|:---:|:---:|
| S-ZNE | $1 \times 10^9$（恒定） | 1× |
| 常规 ZNE | $3.75 \times 10^{10}$（线性增长） | **37.5×** |

### 4.2 量子计量（100-qubit GHZ Ramsey 干涉）

**设置**：
- 初态：$|\text{GHZ}\rangle_{100} = \frac{1}{\sqrt{2}}(|0\rangle^{\otimes 100} + |1\rangle^{\otimes 100})$
- 编码：$U(x) = R_Z(x)^{\otimes 100}$，$x$ 为待测相位
- 理想信号：$f(x, Z^{\otimes 100}) = \cos(100x)$（Heisenberg 极限标度）
- 噪声：全局退极化 $p_g=0.1$
- $u=5$ 噪声水平，测试 500 个 $x$ 点

**代理训练**（Fig. 3a）：所有噪声水平的代理在 100 轮内 MSE 降至 $2 \times 10^{-4}$ 以下

**相位估计**（Fig. 3b–c）：
| 方法 | MSE | 
|------|:---:|
| 未缓解 | $4.84 \times 10^{-3}$ |
| 常规 ZNE | **$4.83 \times 10^{-4}$** |
| S-ZNE | $6.79 \times 10^{-4}$ |

- S-ZNE 精确恢复了 $\cos(100x)$ 的 $2\pi/100$ 周期性

**测量开销**：
| 方法 | 总 shots |
|------|:---:|
| S-ZNE（训练） | $1 \times 10^7$ |
| 常规 ZNE | $5 \times 10^7$ |
| **节省** | **80%** |

> 📌 效率差距随评估点数量增加而**持续扩大**——S-ZNE 的经典推理几乎零额外成本。

---

## 五、目前的不足

### 5.1 全部实验为经典数值模拟，未在真实量子硬件上验证

预测代理的前一篇（Nat. Commun. 2026）在 **42-qubit 真实超导处理器**上完成了实验。本文（预印本）所有实验结果均来自 TensorCircuit MPS 模拟器——尽管规模达到 100 qubits，但噪声模型（全局退极化 + 均匀相干扰动）是对真实硬件的极大简化，缺少 $T_1/T_2$ 退相干、串扰、时变参数漂移等关键噪声特征。

### 5.2 与预测代理论文的方法论重合度极高

| 组件 | 预测代理 [Nat. Commun. 2026] | 本文 S-ZNE |
|------|------|------|
| 回归型代理 | $h_{qs}$ | 回归型代理（式 (5)–(7)） |
| 核型代理 | $h_{cs}$ | 核型代理（式 (8)–(10)） |
| 三角函数单项式基 | $\Phi_\omega(\mathbf{x})$ | $\Phi_\omega(\mathbf{x})$（完全一致） |
| 理论框架 | Theorem 1 & 2 | Theorem 1 |

**核心增量**在于将代理应用于 **ZNE 的 $u$ 个噪声水平维度**——代理本身的方法论在前一篇已经建立。这使得本文的创新更多在**应用层面**（将代理用于误差缓解）而非**方法层面**。

### 5.3 Theorem 1 仅保证与常规 ZNE 可比，不承诺超越

S-ZNE 的误差上界与常规 ZNE 一致——它不承诺更高的缓解精度，只承诺**相同的精度 + 更低的测量开销**。如果代理的训练样本数 $n_j$ 不足，S-ZNE 的性能将劣于常规 ZNE（虽然定理给出了 $n_j$ 的理论下界，但常数因子可能很大）。

### 5.4 混合 S-ZNE（hybrid S-ZNE）仅提及，未实验

论文 Remark 提到可将代理用于高噪声水平、量子处理器直接测量用于低噪声水平，以平衡精度与效率——但**未提供任何混合方案的实验结果**。

### 5.5 代理训练成本本身不可忽略

S-ZNE 需要 $u$ 个代理各 $n_j$ 个训练样本——每个样本需 $T$ shots（实验中 $T = 10^6$ for VQE, $T = 2 \times 10^4$ for metrology）。总训练成本为 $n_j \times u \times T$：
- VQE：$200 \times 5 \times 10^6 = 10^9$（即 Fig. 2c 的恒定开销）
- 计量：$100 \times 5 \times 2\times10^4 = 10^7$

虽然在大量评估场景下远低于常规 ZNE，但当**评估点极少**时（如仅需一个 $\mathbf{x}$），S-ZNE 的训练开销反而更大。

### 5.6 外推函数 $g(\cdot)$ 的最优选择未被讨论

$\zeta^2$（外推固有误差）是两种方法误差上界的共同项——这意味着 $g(\cdot)$ 的选择（线性/多项式/Richardson/指数外推）对最终精度有决定性影响。论文仅使用了线性外推，未系统比较不同 $g(\cdot)$ 对 S-ZNE 的影响。

---

## 六、总体评价

### 6.1 论文定位

这是**预测代理框架在量子误差缓解上的直接应用**——与 Nat. Commun. 2026 的预测代理论文形成"方法 → 应用"的姐妹篇关系。其核心 message 简洁有力：**既然代理可以预测含噪均值 $f(\mathbf{x}, O, \lambda)$，那在多个 $\lambda$ 上各训一个代理，ZNE 就可以完全经典化**。

### 6.2 与同系列核心论文的对比

| 维度 | 预测代理 [Nat. Commun. 2026] | 本文 S-ZNE [预印本] |
|------|------|------|
| 方法论原创性 | ★★★★★（提出代理概念+两种实现） | ★★★（代理框架的直接应用） |
| 理论保证 | ★★★★★（Theorem 1–2） | ★★★★（Theorem 1） |
| 实验平台 | **42-qubit 真实超导芯片** | TensorCircuit MPS 模拟器 |
| 实验规模 | 20–42 qubits | **100 qubits** |
| 量子资源节省 | ~4300×（vs VQE） | **37.5×（VQE）, 5×（计量）** |
| 发表状态 | Nat. Commun. 2026 | Research Square 预印本 |
| 增量贡献 | 建立代理范式 | 将代理用于 ZNE 的 $u$ 个噪声维度 |

### 6.3 一句话总结

> S-ZNE = 预测代理 × ZNE——在 $u$ 个噪声水平上各训练一个三角函数单项式核回归代理，使 ZNE 从"每输入一次全量子流程"变为"一次训练 + 无限次经典外推"。100-qubit 模拟显示以 37.5× 更少的测量量达到与常规 ZNE 可比的误差缓解精度。方法论增量有限（代理框架此前已建立），但其"恒定开销"的工程价值对大规模参数扫描类量子任务（VQE、量子计量、多体模拟）是变革性的。

---

> 生成日期：2026-06-16
