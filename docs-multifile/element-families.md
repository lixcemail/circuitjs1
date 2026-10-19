# CircuitJS：设备类型、数字逻辑与外部数据扩展

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/element-families.md)

## 源码分类

| 类型 | 代表源码 | 学习重点 |
| --- | --- | --- |
| 被动与动态元件 | [ResistorElm.java](../src/com/lushprojects/circuitjs1/client/ResistorElm.java)、[CapacitorElm.java](../src/com/lushprojects/circuitjs1/client/CapacitorElm.java)、[InductorElm.java](../src/com/lushprojects/circuitjs1/client/InductorElm.java) | 公共图元接口、时间相关状态与电气贡献 |
| 非线性模型 | [Diode.java](../src/com/lushprojects/circuitjs1/client/Diode.java)、[DiodeModel.java](../src/com/lushprojects/circuitjs1/client/DiodeModel.java)、[MosfetModel.java](../src/com/lushprojects/circuitjs1/client/MosfetModel.java) | 实例与模型参数、数值近似和迭代 |
| 数字与芯片 | [GateElm.java](../src/com/lushprojects/circuitjs1/client/GateElm.java)、[ChipElm.java](../src/com/lushprojects/circuitjs1/client/ChipElm.java)、[DFlipFlopElm.java](../src/com/lushprojects/circuitjs1/client/DFlipFlopElm.java) | 引脚、位序、输入输出、状态更新 |
| 总线 | [WireElm.java](../src/com/lushprojects/circuitjs1/client/WireElm.java)、[BusSplitterElm.java](../src/com/lushprojects/circuitjs1/client/BusSplitterElm.java) | 位宽、端口 z 与位连接 |
| 复合模型 | [CompositeElm.java](../src/com/lushprojects/circuitjs1/client/CompositeElm.java)、[CustomLogicElm.java](../src/com/lushprojects/circuitjs1/client/CustomLogicElm.java) | 子图元组合与自定义逻辑 |
| 外部数据 | [DataInputElm.java](../src/com/lushprojects/circuitjs1/client/DataInputElm.java)、[AudioInputElm.java](../src/com/lushprojects/circuitjs1/client/AudioInputElm.java)、[AudioOutputElm.java](../src/com/lushprojects/circuitjs1/client/AudioOutputElm.java) | 时序数据、浏览器音频与资源生命周期 |
| 图形标注 | [GraphicElm.java](../src/com/lushprojects/circuitjs1/client/GraphicElm.java)、[TextElm.java](../src/com/lushprojects/circuitjs1/client/TextElm.java)、[BoxElm.java](../src/com/lushprojects/circuitjs1/client/BoxElm.java) | 无电气端口的可编辑图形 |

## 同一抽象中的差异

不同类别共用 CircuitElm，但计算方式、端口、状态和资源不相同。ChipElm 统一部分芯片外形和引脚配置，GateElm 统一部分逻辑门行为，具体类重写差异。

WireElm 对总线以位宽及 Point.z 等信息区分不同位，不只是把线画粗。站场的多条并行轨道不等于电路总线，不能直接复制位宽语义。

AudioInputElm 持有数据与采样信息，并以静态缓存支持剪切粘贴、撤销等过程的数据保留。外部资源不一定天然完整包含在普通元件字段中，理解保存时必须检查实际序列化及资源缓存。

## 新增类别的检查清单

检查构造工厂、菜单、属性、端口、几何、绘图、存档标识、运行状态与资源释放。不要仅新增 draw 就认为完整支持一种新元件。

本系列以模块机制为覆盖范围，不逐个重述全部电子器件公式。元件族的具体物理模型可沿表中源码继续研究。
