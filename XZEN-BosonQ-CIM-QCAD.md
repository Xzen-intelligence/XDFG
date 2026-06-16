# XZEN-BosonQ-CIM 量子计算辅助设计工具

## 概述

将三维Bose-Hubbard量子自组织问题的完整研究流程——从经典仿真到CIM真机验证——封装为可复用的量子计算辅助设计(QCAD)工具。

**命名**: XZEN-BosonQ-CIM-QCAD

## 架构

```
┌─────────────────────────────────────────────────────────┐
│          XZEN-BosonQ-CIM-QCAD 工具链                      │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │ Step 1      │  │ Step 2       │  │ Step 3        │  │
│  │ 问题建模    │→ │ 参数扫描      │→ │ CIM映射与验证 │  │
│  │ (BH→QUBO)  │  │ (Bose-       │  │ (QUBO→Ising  │  │
│  │            │  │  Hubbard     │  │ →CIM求解     │  │
│  │            │  │ 含时演化)    │  │ →噪声匹配)   │  │
│  └─────────────┘  └──────────────┘  └───────────────┘  │
│         │                │                │             │
│         └────────────────┼────────────────┘             │
│                          ▼                              │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Step 4: 结果分析与参数反推                       │   │
│  │ 噪声-有序度曲线 → 最优噪声参数 → 实验建议       │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## 安装

```bash
# 依赖
pip install numpy scipy matplotlib scikit-learn
# 玻色量子SDK (Python 3.10)
pip install kaiwu==1.3.1
```

## 使用流程

### Step 1: Bose-Hubbard → QUBO

```python
from xzen_boson_qcad import BH3DToQUBO

converter = BH3DToQUBO(lattice_size=(8,8,8), filling=0.125)
Q_matrix, info = converter.build(n_modes=5)
# info: {n_sites, n_particles, J, U, modes, ...}
```

### Step 2: 全精度参数扫描

```python
from xzen_boson_qcad import FullPrecisionScan

scan = FullPrecisionScan(lattice_size=(8,8,8), n_particles=64)
results = scan.run(n_shots=10000, n_workers=4)
# 输出: 秩序度分布、涡旋动力学、结构多样性
```

### Step 3: CIM映射与验证

```python
from xzen_boson_qcad import CIMValidator

validator = CIMValidator()
# 8bit精度调整 → Ising → CIM提交
cim_result = validator.submit(Q_matrix, task_name='bh3d_custom')
# 噪声映射
noise_result = validator.noise_scan(Q_matrix, noise_levels=[0, 0.05, 0.1])
```

### Step 4: 最优参数反推

```python
from xzen_boson_qcad import ParameterInversion

inverter = ParameterInversion(cim_noise_data, sim_data)
optimal_noise = inverter.find_best_match()
# 返回: 最优噪声级别、置信区间、实验建议
```

## CIM真机映射协议 (已验证)

```
输入:  Bose-Hubbard 参数 (J, U, filling, lattice)
步骤:  BH能泛函 → 模式分解 → QUBO矩阵
       → adjust_qubo_matrix_precision(8bit)
       → qubo_matrix_to_ising_matrix
       → CIMOptimizer.solve()
       → 自旋解 → 结构指纹
输出:  有序结构集合 (N个解)
```

### 正确格式管道

```python
# 已验证通过 (65自旋、126自旋均已成功)
Q = build_qubo()                          # NxN
Q_adj = adjust_qubo_matrix_precision(Q)   # 8bit整数
ising_mat, _ = qubo_matrix_to_ising_matrix(Q_adj)  # (N+1)x(N+1)
optimizer = CIMOptimizer(task_name=name, wait=True, interval=2)
solutions = optimizer.solve(ising_mat)    # 自旋解
```

## 噪声-有序度校准曲线 (XZEN实验基准)

| 噪声级别 | CIM有序度均值 | CIM有序度标准差 | 结构多样性 |
|---------|-------------|---------------|-----------|
| 0.00    | 0.8279      | 0.2616        | 31.6%     |
| 0.05    | 0.6490      | 0.4036        | 55.6%     |
| 0.10    | 0.6735      | 0.3576        | 42.9%     |
| 0.20    | 0.9127      | 0.1197        | 35.7%     |

**最优噪声参数**: nl* ≈ 0.015 (内插)
**匹配目标**: C3 as=2.0 仿真的 0.7739 ± 0.1229

## 文件结构

```
xzen_boson_qcad/
├── __init__.py
├── bh3d_qubo.py           # BH→QUBO映射
├── simulation_engine.py   # Bose-Hubbard含时演化
├── cim_adapter.py         # CIM真机适配器 (格式管道)
├── noise_calibrator.py    # 噪声校准与参数反推
├── analyzer.py            # 结构聚类/动力学分析
├── results/               # 实验数据
│   ├── xzen_10000_shots/       # 全速方案
│   ├── xzen_8000_shots_full/   # 全精度方案
│   └── xzen_cim_validation/    # CIM真机验证
└── docs/
    └── xzen_qcad_protocol.md   # 完整协议文档
```

## 设计原则 (XZEN原则)

1. **最小干预** — 不给系统预设目标函数，让量子系统自主演化
2. **动态观测** — 无偏见物理量（关联函数、布拉格峰、结构因子）
3. **量化流程** — 每一步可复现、可审计、可优化
4. **真机验证** — 经典仿真必由量子硬件交叉验证

## 版本

v1.0.0 — XZEN-BosonQ-CIM-QCAD
首次验证: 2026-06-10
