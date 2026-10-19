# StationEdit：性能、刷新调度与缓存

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../performance-and-scheduling.md)

## 参考源与适用范围

参考 [UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 repaint、[SimulationManager.java](../../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的帧预算、[util/PerfMonitor.java](../../src/com/lushprojects/circuitjs1/client/util/PerfMonitor.java)。requestAnimationFrame、派生缓存、空间索引和工作线程为建议设计。

## 第一版调度

文档或会话变化只标记需要重绘；通过一个 requestAnimationFrame 合并绘制请求。无交互且无运行状态变化时不持续重绘。指针输入保存最新候选，屏幕刷新显示最近结果。

记录 updateGeometry、hitTest、validate、render、serialize、save 等耗时和文档规模。基准文档至少覆盖小样例、中型站场和密集重叠对象。

## 缓存与失效

以设备 ID、几何版本及符号样式版本作为派生几何缓存依据。位置、角度、尺寸和字体变化使对应缓存失效；平移画布不重算设备本地几何。

全量校验先保证正确性，再按受影响设备与引用邻居做增量校验。命中和绘制确有瓶颈时增加空间索引、视口裁剪及文本测量缓存。

## 异步工作

大型文件解析、路径分析或复杂布局可迁移到 Worker。请求包含 documentRevision 和 requestId；旧结果、已取消结果不得覆盖新文档。返回纯数据，避免工作线程依赖 DOM。

导出单独控制像素数，快照历史控制容量；存储写入防抖不能丢失最后一次已提交版本。

## 验收

在记录的目标设备上报告指针到显示延迟和各阶段耗时，不只报平均 FPS。连续拖动不会堆积无限任务；取消与加载后旧异步结果被丢弃；缓存开启和关闭得到等价几何及诊断。
