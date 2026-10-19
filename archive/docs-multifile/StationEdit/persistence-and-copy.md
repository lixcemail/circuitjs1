# StationEdit：保存、复制与删除

本文是站场图编辑器的建议设计，尚未在仓库中实现。TypeScript 风格示例用于说明结构，不限定技术选型。各节注明参考源码与新增设计。

[文档导航](../README.md) · [对应 CircuitJS 实现](../persistence-and-copy.md)

工程文件的数据结构见 [图元模型](element-abstraction.md#稳定身份与最小模型)。复制和删除产生的文档修改按 [编辑事务](edit-transactions.md) 提交。

## 保存、复制和删除

> **参考源与适用范围：** 参考 [XMLSerializer.java](../../src/com/lushprojects/circuitjs1/client/XMLSerializer.java) 的 dumpCircuit、[CircuitLoader.java](../../src/com/lushprojects/circuitjs1/client/CircuitLoader.java) 的 readCircuit、[CommandManager.java](../../src/com/lushprojects/circuitjs1/client/CommandManager.java) 的 copyOfSelectedElms、doPaste、doDelete。版本校验、稳定 ID 重映射、轨道删除后的信号关联清理和加载失败保留文档属于本站场方案。

### 保存加载

工程文件保存 schemaVersion、图元和连接，派生几何加载后重新计算。加载步骤建议为：

1. 解析文件并检查格式版本。
2. 检查字段类型、有限数值、ID 唯一性和设备属性范围。
3. 检查端口引用、信号关联和连接约束。
4. 完成后整体替换当前文档。

加载失败保留原文档，并报告可定位的原因。未知类型或不支持的版本应明确报错，避免静默丢失设备。

### 复制粘贴

- 为副本生成新的图元 ID 和连接 ID。
- 复制选区内部的连接，并通过旧 ID → 新 ID 映射更新引用。
- 默认不复制指向选区外设备的连接。
- 信号关联指向选区内轨道时重新映射；指向选区外轨道时，第一版清除关联并给出提示。
- 保持副本的相对位置，并作为一次事务提交。

### 删除

- 删除图元时，同时删除引用其端口的连接。
- 删除轨道时，清除相关信号机的 attachment，保留信号机符号并提示待重新关联。
- 整组删除及关联清理必须可以一次撤销。

这些规则应集中在文档编辑模块，不能只在键盘 Delete 的处理函数中实现，否则右键删除或其他入口容易遗漏。

## 验收

- 保存加载后图元 ID、属性、几何和连接语义等价。
- 损坏文件和不支持版本不会替换当前文档。
- 复制选择集后内部引用指向副本，外部引用按约定清理。
- 删除及引用清理一次撤销即可完整恢复。
- 同一文件通过 UI 与外部接口导入，得到相同校验结果。
