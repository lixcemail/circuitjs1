# CircuitJS：采样、波形与时间线

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/monitoring-and-timeline.md)

## 源码入口

[ScopeManager.java](../src/com/lushprojects/circuitjs1/client/ScopeManager.java) 管理示波器；[Scope.java](../src/com/lushprojects/circuitjs1/client/Scope.java) 组织显示；[ScopePlot.java](../src/com/lushprojects/circuitjs1/client/ScopePlot.java) 保存单条曲线；[ScopeTrigger.java](../src/com/lushprojects/circuitjs1/client/ScopeTrigger.java) 管理触发；[ScopeFFT.java](../src/com/lushprojects/circuitjs1/client/ScopeFFT.java) 配合 [FFT.java](../src/com/lushprojects/circuitjs1/client/FFT.java) 处理频谱；[ScopeSerializer.java](../src/com/lushprojects/circuitjs1/client/ScopeSerializer.java) 保存示波器配置。

## 仿真采样与显示分开

ScopePlot.timeStep 从元件取得测量值。每个显示时间桶维护最小值与最大值，使用环形数组和 ptr 推进。一个像素代表多个仿真步时，保留极值比仅保留最后一个值更容易显示尖峰。

环形索引的位与运算依赖容量约定。扩展到任意容量不能直接复制掩码公式。reset 处理容量、速度和已有数据保留。

## 触发机制

ScopeTrigger 维护 ARMED、TRIGGERED、AUTO_RUN 等状态，结合触发模式、边沿和阈值决定显示窗口。validDataCount 避免绘制尚未刷新、仍残留在环形缓冲中的旧数据。

## 频谱与保存

ScopeFFT 从缓冲构造实部和虚部，调用 FFT 并计算幅值，支持相关频谱显示选项。ScopeSerializer 管理曲线和显示配置；不要把保存示波器设置理解成无限量历史原始数据归档。

## 可借鉴的技术

采样与绘图解耦、环形缓冲、有效区间、触发状态和配置持久化适用于运行监视。站场离散事件更适合事件时间线，不必将所有状态都转换成连续模拟波形。
