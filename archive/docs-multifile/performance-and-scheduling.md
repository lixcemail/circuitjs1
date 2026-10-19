# CircuitJS：性能、刷新调度与缓存

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/performance-and-scheduling.md)

## 源码入口

[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 repaint/updateCircuit，[CirSim.java](../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 timer/FASTTIMER，[SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的 runCircuit，[util/PerfMonitor.java](../src/com/lushprojects/circuitjs1/client/util/PerfMonitor.java) 的 startContext/stopContext/buildString。

## 刷新合并

repaint 用 needsRepaint 避免重复排队，通过 Scheduler 延迟执行一次 updateCircuit。运行时 Timer 周期刷新；暂停时通过按需 repaint 保持编辑反馈。

updateCircuit 中先处理需要的分析与计算，再绘制。图元遍历绘制和命中候选遍历随对象数增长，并非已存在完整的空间索引或局部重绘框架。

## 计算预算

runCircuit 在一帧内执行多个步，但根据最小帧率对应的时间预算退出。线性矩阵提前分解、元件数组 elmArr 等路径减少重复计算和调用开销。

## 测量

PerfMonitor 用 performance.now 等计时，组织嵌套上下文；UIManager 给分析、计算、绘图等阶段添加测量。它有助于知道耗时在哪，不能代替真实设备与真实文档上的性能验证。

## 迁移时的判断

渲染次数、每次绘制成本、几何分析、快照序列化和存储写入是不同瓶颈。优化前应单独计时，不把界面卡顿一律归因于 Canvas。
