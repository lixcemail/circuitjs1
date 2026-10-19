# StationEdit：采样、波形与时间线

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../monitoring-and-timeline.md)

## 参考源与适用范围

参考 [ScopePlot.java](../../src/com/lushprojects/circuitjs1/client/ScopePlot.java) 的 timeStep/reset、[ScopeTrigger.java](../../src/com/lushprojects/circuitjs1/client/ScopeTrigger.java) 的状态和有效数据处理，[ScopeSerializer.java](../../src/com/lushprojects/circuitjs1/client/ScopeSerializer.java) 的配置保存。离散事件时间线与回放为新设计。

## 记录什么

第一版记录设备 ID、事件类型、事件时间、序号、旧值、新值和来源。用序号区分同一时间事件，展示时间和内部排序时间分别处理。

事件日志独立于设计文档历史：撤销移动设备不能撤销已经接收的运行事件；清空监视数据也不删除站场设备。

## 缓冲和回放

使用有容量上限的环形缓冲，淘汰旧数据时显示保留范围。长记录另存文件。回放从确定的初始状态和版本开始；需要快速跳转时定期保存运行快照，再从最近快照重放事件。

对事件 ID 去重；外部乱序事件按协议处理，不能无提示重写历史。回放模式暂停实时输入或隔离实时通道，避免混合两种时间。

## 显示与导出

时间线支持按设备和事件类型过滤，选中事件定位设备；设备已删除时仍能查看历史标签快照。连续测量数据可借鉴 min/max 分桶，但占用等离散状态使用区间或状态阶梯。

导出记录包含文档 ID、设计版本、时间单位和场景版本；不把设备名称作为唯一引用。

## 验收

同一时间多事件顺序稳定；缓冲回绕不显示旧槽位；暂停不会持续追加演示事件；跳转回放与从头重放得到相同状态；保存曲线配置不误称保存完整历史。
