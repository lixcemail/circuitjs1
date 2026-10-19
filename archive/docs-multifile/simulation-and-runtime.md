# CircuitJS：仿真循环、矩阵与运行状态

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/simulation-and-runtime.md)

## 源码入口

[CirSim.java](../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 timer，[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 updateCircuit/setSimRunning，[SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的 analyzeCircuit、preStampCircuit、stampCircuit、runCircuit、lu_factor/lu_solve，[CircuitMatrix.java](../src/com/lushprojects/circuitjs1/client/CircuitMatrix.java) 与 [matrix/SparseLU.java](../src/com/lushprojects/circuitjs1/client/matrix/SparseLU.java)。

## 从电路到方程

```text
图元及端点 → 网络分析 → 节点/电压源未知量
    → 元件 stamp 写矩阵和右端项 → 求解 → 更新节点电压和电流
```

stampResistor 等方法把元件电气关系累积到矩阵。CircuitMatrix 保存矩阵、右端项、节点状态与分解相关数据；网络闭包分析可形成多个矩阵。

线性矩阵可以提前分解，多个时间步重复求解；非线性元件在迭代中更新项，并按收敛情况重新分解。求解分派支持稠密 LU 和稀疏 LU，AUTO 根据矩阵规模选择；当前仍有稠密矩阵数据及转为 CSC 的路径，不能理解成全流程始终稀疏。

## 时间推进

runCircuit 在一次界面刷新中推进多个时间步，调用 startIteration、doStep、stepFinished，并更新测量和时间步钩子。自适应步长在收敛困难时减小，成功后逐步增大到最大步长。

计算循环有帧耗时预算，避免一次推进长期占用界面线程。仿真时间 t 与真实墙钟时间不是同一个概念。

## 调度和错误

运行时 GWT Timer 周期请求更新；暂停时取消周期更新，仍允许 repaint。矩阵奇异、非有限值或收敛失败等通过 stop 停止并提示。

## 对站场图的价值

编辑器本身不需要电路方程求解器。可以借鉴静态模型、派生分析和运行状态的区分，以及有预算的时间推进。若仅实现编辑器，可先不引入仿真模块。
