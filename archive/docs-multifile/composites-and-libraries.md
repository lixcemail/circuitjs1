# CircuitJS：组合图元、模板与模型库

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/composites-and-libraries.md)

## 源码入口

[CompositeElm.java](../src/com/lushprojects/circuitjs1/client/CompositeElm.java) 保存内部图元和节点映射；[CustomCompositeModel.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeModel.java) 保存子电路定义、外部端口及模型表；[CustomCompositeElm.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeElm.java)、[CustomCompositeChipElm.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeChipElm.java) 实例化模型；[CirSim.java](../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 pushContext/popContext 管理嵌套编辑；[SubcircuitBar.java](../src/com/lushprojects/circuitjs1/client/SubcircuitBar.java) 提供上下文路径界面。

## 模型与实例

模型保存可复用定义，实例保存该模型在图纸上的使用。CustomCompositeModel 有 globalModelMap 与 localModelMap，外部端口信息存入 extList，子图元定义可存为 XML 文档。

CompositeElm 根据内部节点编号关联子图元端点，并将外部节点映射到内部网络。它不仅把多个符号一起画，还涉及电气行为的组合。

## 编辑上下文

进入模型编辑时，pushContext 保存当前文档快照、历史栈和视图，建立新的局部历史。返回时 popContextAndGetChangedModels 恢复外层文档及历史，并返回更深层修改的模型信息。

上下文隔离解释了为什么打开文件需要 resetEditingContext：不能把旧模型的编辑栈带入新文档。

## 模型库与保存

CustomCompositeModel.loadModelsFromStorage 读取 subcircuit: 前缀的本地数据，也兼容历史模型格式。序列化使用 dumped 等状态避免重复写模型。

## 限制与迁移价值

组合电路的节点映射与站场分组不是同一业务。值得借鉴定义与实例、端口暴露、嵌套编辑上下文；不要因为想同时移动一组设备就先实现完整子电路体系。
