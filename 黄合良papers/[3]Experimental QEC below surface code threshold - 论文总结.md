# [表面码下阈值] Experimental Quantum Error Correction below the Surface Code Threshold via All-Microwave Leakage Suppression — 论文总结

> **文献信息**
> - **标题**：Experimental Quantum Error Correction below the Surface Code Threshold via All-Microwave Leakage Suppression
> - **作者**：Tan He, Weiping Lin, Rui Wang, Yuan Li (共同一作), ... He-Liang Huang, ... Fusheng Chen, Xiaobo Zhu, Jian-Wei Pan (通讯)
> - **机构**：中科大 / 上海量子科学研究中心 / 合肥国家实验室 / 量子CTek / 河南省量子信息与密码学重点实验室 / 西电 / 济南量子院
> - **发表**：*Physical Review Letters* **135**, 260601 (2025)
> - **标注**：**Editors' Suggestion** | **Featured in Physics**
> - **DOI**：10.1103/rqkg-dw31
> - **日期**：Received 2025-10-09 / Published 2025-12-22
> - **硬件**：**祖冲之 3.2**（107-qubit 超导处理器），使用 97 qubits

---

## 目录

- [一、创新性贡献](#一创新性贡献)
- [二、泄漏错误：表面码的头号公敌](#二泄漏错误表面码的头号公敌)
- [三、全微波泄漏抑制架构](#三全微波泄漏抑制架构)
- [四、实验结果](#四实验结果)
- [五、与系列中纠错码工作的关系](#五与系列中纠错码工作的关系)
- [六、总体评价](#六总体评价)

---

## 一、创新性贡献

1. **首次在超导量子处理器上实现表面码逻辑错误率的跨阈值（below-threshold）指数抑制**。在 distance-7 表面码上测得逻辑错误抑制因子 $\Lambda = 1.40(6)$（$\Lambda > 1$ 标志着逻辑错误随码距增大而指数衰减），终结了此前所有表面码实验 $\Lambda < 1$（码距越大反而越差）的"above-threshold"状态。**PRL Editors' Suggestion + Featured in Physics**。

2. **提出并验证了全微波（all-microwave）泄漏抑制架构**——将数据 qubit 的**泄漏减少单元（LRU）**与 ancilla qubit 的**无条件复位（RST）**无缝嵌入表面码周期电路，不增加任何时间或空间开销。在 40 个 QEC 周期后，泄漏布居被压制至 $6.4(5) \times 10^{-4}$——相比无缓解基线改善 **72 倍**。

3. **提出针对大规模量子处理器的高效频率分配与校准协议**：将 LRU/RST 驱动频率作为新的约束，与单比特门、双比特门、读出的频率联合优化以避免冲突；将 RST 协议解耦为 $|f0\rangle \to |e1\rangle \to |e0\rangle$ 和 $|e0\rangle \to |g1\rangle \to |g0\rangle$ 两个顺序阶段——降低波形复杂度，实现系统规模的可扩展性。

4. **在祖冲之 3.2 处理器上完成 distance-7（97 qubits）的完整实验**，逻辑错误率 $\epsilon_{L,d=7} = 0.783(3)\%$，是**目前全球表面码实验中最大的码距和最低的逻辑错误率**。

---

## 二、泄漏错误：表面码的头号公敌

### 2.1 什么是泄漏？

Transmon qubit 是弱非谐振荡器——计算子空间（$|0\rangle_g, |1\rangle_e$）之上存在更高的 $|2\rangle_f$ 能级。门操作（特别是 CZ 门和测量）可能将量子信息**泄漏**到 $|f\rangle$ 态，导致：

- **空间扩散**：泄漏的 qubit 通过耦合器将错误传播给近邻 qubits
- **时间累积**：泄漏的布居不随 QEC 周期衰减，逐周期累加
- **破坏 QEC 的基本假设**：表面码的错误模型假设物理错误是**独立的**——泄漏产生长程关联错误，直接破坏这一假设

### 2.2 为什么传统方法不足

此前有两种泄漏处理策略：

| 策略 | 原理 | 缺点 |
|------|------|------|
| **dc-基方案**（Google [9–12]） | 基带磁通脉冲将 qubit 频率移至与损耗读出谐振腔共振 → SWAP 耗散 | 要求谐振腔频率低于 qubit → 限制处理器架构设计；易引发读出诱导跃迁 |
| **全微波方案**（本文） | 参数磁通调制激活特定边带跃迁 → 返回计算子空间 | 此前仅在小规模系统上验证，尚不清楚在大规模 QEC 架构中的串扰和频率拥挤问题 |

---

## 三、全微波泄漏抑制架构

### 3.1 物理机制（Fig. 1b–c）

利用**参数磁通调制**（parametric flux modulation）激活两个边带跃迁：

**f-LRU**（Leakage Reduction Unit，数据 qubit + ancilla qubit 均用）：
$$|f0\rangle \xrightarrow{\omega_d} |e1\rangle \xrightarrow{\kappa_r} |e0\rangle$$

- 单步过程：$|f\rangle$ 泄漏布居 → 微波驱动耦合到 $|e\rangle$ + 1 光子 → 光子通过谐振腔的快速衰减（$\kappa_r$，~15 ns ringdown）耗散到环境 → 净效果 $|f0\rangle \to |e0\rangle$

**e-RST**（仅 ancilla qubit 用）：
$$|e0\rangle \xrightarrow{\omega_d} |g1\rangle \xrightarrow{\kappa_r} |g0\rangle$$

- ancilla qubit 的两步级联：$|f\rangle$ →（f-LRU）→ $|e\rangle$ →（e-RST）→ $|g\rangle$ → 完全复位到基态

### 3.2 电路集成（Fig. 1d）

关键创新：**零时间开销、零空间开销**的集成方式。

| qubit 类型 | 操作 | 时序 |
|------|------|------|
| **Ancilla qubit** | 测量(M) + RST (f-LRU + e-RST) | 测量后立即执行 RST：44 ns f-LRU + 44 ns e-RST = **96 ns**（含 padding） |
| **Data qubit** | LRU + 动态解耦(DD) | LRU 与 ancilla 的 M+RST **并发执行**：496 ns f-LRU + 48 ns padding = **544 ns**——恰好等于 ancilla 的 M+RST 总时间 |

- 数据 qubit 的 LRU **不破坏编码信息**（仅作用于 $|f\rangle$ 态，不触及 $|g\rangle, |e\rangle$ 计算子空间）
- LRU 引入的计算误差 $\leq 0.01\%$（与 DD-only 在统计上不可区分，Fig. 2d）

### 3.3 可扩展校准协议

**挑战**：97 qubits 并行操作时的**频率拥挤**和**微波串扰**。

**解决方案**：
1. 将 LRU/RST 驱动频率作为新的约束维度，与单比特门、CZ 门、读出的频率**联合优化**以避免碰撞
2. **两阶段校准**（Fig. 2a–b）：
   - **粗扫**：驱动幅度-频率平面扫描 → 拟合 ac Stark 频移（抛物线）
   - **精扫**：沿拟合曲线扫描幅度 → 找到最小残余 $|f\rangle$ 布居
3. 对于长脉冲（数据 LRU 496 ns），残余布居**单调衰减**（非振荡）——改用目标泄漏降低率（95%）作为停止准则
4. RST 协议解耦为 $|f\rangle \to |e\rangle$ 和 $|e\rangle \to |g\rangle$ 两个独立阶段 → 降低波形复杂度

---

## 四、实验结果

### 4.1 硬件性能

| 指标 | 数值 |
|------|:---:|
| **处理器** | 祖冲之 3.2 (107-qubit 芯片, 频率可调 transmon + 可调耦合器) |
| **使用 qubits** | **97**（49 数据 + 48 ancilla, 完整 distance-7 表面码） |
| **单比特门误差** | **0.089%** |
| **双比特门（CZ）误差** | **0.543%** |
| **读出误差** | **0.95%** |
| **样品重复周期** | **100 μs**（RST 加速初始化） |

### 4.2 组件级基准（Fig. 2c–d）

- **并行 RB 式序列**（所有 97 qubits 同时）：每周期注入 ~5% 泄漏，随后执行 RST（ancilla）或 LRU（data）
- ancilla 的 RST 将 $|e\rangle$ 残余压制至 $< 7 \times 10^{-3}$，$|f\rangle$ 至 $< 2 \times 10^{-4}$
- 数据 qubit 的 LRU 将 $|f\rangle$ 残余压制至 $< 2 \times 10^{-3}$
- **LRU 引入的计算误差**：LRU+DD = 1.28(2)% vs DD-only = 1.27(2)%——**统计上不可区分**（Fig. 2d 内插图）

### 4.3 全系统泄漏抑制（Fig. 3）

distance-7 表面码运行 40 个 QEC 周期：

| 配置 | 40 周期后平均泄漏布居 | vs 无缓解 |
|------|:---:|:---:|
| 无缓解（Neither） | $4.64(4) \times 10^{-2}$ | 1× |
| 仅 LRU | — | 部分抑制（仅 data qubit） |
| 仅 RST | — | 部分抑制（仅 ancilla qubit） |
| **全缓解（Both）** | **$6.4(5) \times 10^{-4}$** | **72× 改善** |

- 全缓解下泄漏布居达到**稳态**（不再增长）
- 空间泄漏热点（spatial leakage hotspots）被完全消除（Fig. 3b）

### 4.4 逻辑性能：跨阈值 ⭐（Fig. 4）

| 码距 $d$ | 逻辑错误率 $\epsilon_L$ | 衰减趋势 |
|:---:|:---:|:---:|
| 3 | **1.532(2)%** | — |
| 5 | **1.012(2)%** | ↓ |
| 7 | **0.783(3)%** | ↓↓ |

**核心结果**：用 $\epsilon_L \propto \Lambda^{-(d+1)/2}$ 拟合：

$$\boxed{\Lambda = 1.40(6) > 1}$$

- $\Lambda > 1$ 意味着每增大一步码距，逻辑错误率按照 $\Lambda^{-(d+1)/2}$ **指数衰减**
- 模拟值 $\Lambda_{\text{sim}} = 1.56(3)$——实验值略低，归因于残留串扰和非局域噪声
- **无全泄漏缓解时** $\Lambda < 1$（码距越大越差，逻辑错误率反而放大）

**错误预算**（Fig. 4d）：泄漏不再是主导错误源——当前瓶颈是 **CZ 门保真度**。

---

## 五、与系列中纠错码工作的关系

```
[35] 祖冲之 2.1, d=3 (PRL 2022)
  │  首次重复纠错, Λ<1（above-threshold）
  │  物理错误率：1Q=0.092%, CZ=0.878%, RO=5.39%
  │
[9]  绷带式超稳缺陷适配 (npj QI 2025)
  │  处理制造缺陷, 保持码距
  │
[13] 祖冲之 3.0, 105 qubits (PRL 2025)
  │  1Q=0.097%, CZ=0.375%, RO=0.87%
  │
[3]  祖冲之 3.2, d=7 (PRL 2025) 【本文】
      Λ=1.40(6), 首次 below-threshold ⭐
      1Q=0.089%, CZ=0.543%, RO=0.95%
      Editors' Suggestion + Featured in Physics
```

**演进逻辑**：
- [35]：证明了 "可以做纠错"（原理验证）
- [13]：提升了硬件基础（qubit 数 + 门保真度）
- **[3]**：解决了 "为什么还是不能指数抑制"——**泄漏是隐藏杀手**。全微波 LRU+RST 将泄漏压制 72× 后，$\Lambda$ 从 <1 跳变到 1.40

---

## 六、总体评价

### 6.1 论文定位

这是**祖冲之系列量子纠错路线的登顶之作**（PRL 2025, Editors' Suggestion + Featured in Physics），也是**全球第二个报道表面码跨阈值实验**的论文（与 Google Quantum AI [Nature 2025] 的 dc-基泄漏抑制方案形成两种互补的技术路线）。He-Liang Huang 位列作者名单。

### 6.2 与 Google 方案的对比

| 维度 | Google [Nature 2025] | 本文（祖冲之 3.2） |
|------|------|------|
| 泄漏抑制方案 | dc-基（磁通脉冲 + 谐振腔耗散） | **全微波**（参数磁通调制 + 边带跃迁） |
| 架构约束 | 谐振腔频率必须低于 qubit | **无约束** |
| 频率复用能力 | 有限 | **天然支持**（→ 减少低温布线复杂度） |
| 空间开销 | 需专用泄漏移除 qubits | **零**（与 M+RST 并发） |
| 时间开销 | 额外 | **零** |
| 最大码距 | 未公开（估计 d=7） | **d=7** |
| 逻辑错误抑制因子 | ~1.4 | **1.40(6)** |

### 6.3 一句话总结

> 祖冲之 3.2 以距离-7 表面码首次实现逻辑错误率随码距增大而指数衰减（$\Lambda = 1.40 > 1$），其关键突破是全微波泄漏抑制架构——以零时空开销将泄漏布居压制 72 倍，使泄漏从主导错误源降为次要因素。这篇 PRL Editors' Suggestion 代表表面码纠错从"能做但做不好"正式跨入"越大越好"的容错范式。

---

> 生成日期：2026-06-17
