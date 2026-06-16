XYZEN Boson QIM Toolchain
版权声明：© 2026 XYZEN Intelligence. 本工具链基于 MIT 开源协议发布，嵌入 ORCID: ]，知识共享许可：CC BY-NC-ND 4.0
1. 项目概述
核心优势
完整可落地的量子计算工具链，覆盖从理论建模到硬件真机验证全流程：三维玻色 - 哈伯德建模→QUBO 转换→参数扫描→QIM 真机提交→噪声校准→反推结果动力学分析。非零反脚本，工程规范与实验数据量化落地。
2. 环境依赖与安装
Python 版本：3.9.x
虚拟环境：建议使用 venv 或 conda
依赖安装：pip install numpy==1.23.5 scipy==1.10.1 qiskit==1.0.2
3. 极简快速上手
python
运行
from xyzzen.boson import BH3DToQUBO, QIMSubmitter

# 1. 三维玻色-哈伯德模型转QUBO
bh_qubo = BH3DToQUBO(lattice_size=(8,8,8), filling=0.125)
qubo_matrix = bh_qubo.generate()

# 2. QIM真机提交与结果获取
submitter = QIMSubmitter(hardware="CPQC550")
result = submitter.run(qubo_matrix, shots=1024)
print("真机运行结果：", result)
4. 完整分步流程
硬件协议与实验基准
真机验证日期：2026 年 6 月 10 日
硬件型号：CPQC550
噪声校准：自带噪声有序度校准基准表，支持最优噪声差值结果反推
仿真方案：区分全速 / 全精度两套方案，附带真机 8x8x8 晶格实测数据
文件结构
plaintext
xyzzen-boson-qim/
├── src/                # 源代码
├── experiments/        # 实验数据
├── docs/               # 协议文档
├── requirements.txt    # 依赖配置
└── README.md           # 使用说明
设计原则：Zen4 准则
最小干预：减少对量子硬件原生流程的修改
动态五篇观测：实时跟踪计算过程关键节点
全流程可审计：每一步操作留痕，支持结果回溯
仿真 + 量子硬件交叉验证：确保结论可靠性
5. 版本说明
当前版本：v1.0.0
Latest 标签对应：v1.0.0（首个真机验证稳定版本）
后续迭代：同步更新噪声校准基准与晶格拓展功能
6. 数据文件说明
大容量仿真原始数据文件（100MB-392MB）未随代码仓库上传，可通过配套数据集仓库 / 云盘下载，仓库仅保留校准配置与真机小型结果。
7. 开源合规说明
本工具链为原创自研项目，完整三维 BH→QIM 映射架构、噪声参数反推校准流程均为独立设计。遵循 MIT 开源协议，任何人可自由修改、商用、分发，仅需保留顶部版权声明与仓库 LICENSE 文件，禁止移除 ORCID / 双 C 认证、XYZEN 原创标识等基础署名信息。
GitHub 话题标签：quantum-computing, boson-hubbard, qim-simulator, qubo, quantum-annealing, many-body-physics, quantum-hardware
