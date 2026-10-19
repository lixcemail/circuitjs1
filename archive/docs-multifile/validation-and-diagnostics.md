# CircuitJS：约束校验与错误定位

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/validation-and-diagnostics.md)

## 源码入口

[CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 validate 默认返回 true；[SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的 validateCircuit 遍历元件并在失败时返回 false，stop 保存停止原因与相关元件；[FindPathInfo.java](../src/com/lushprojects/circuitjs1/client/FindPathInfo.java) 按电气条件进行深度优先路径检查。

## 不同层次的检查

- 创建阶段：creationFailed 检查零尺寸等创建失败条件。
- 编辑阶段：allowMove、movePoint 等限制部分操作。
- 属性阶段：EditDialog 解析与属性 setter 检查输入。
- 网络及计算阶段：元件验证、电气回路检查、奇异矩阵及收敛失败等。

validateCircuit 的 bool 返回方式用于当前分析流程，并不是包含全部问题的诊断列表。makePostDrawList 还会生成端点及 badConnectionList 等显示信息。

## 错误怎样显示

SimulationManager.stop 关联 stopMessage 和图元；UIManager.updateCircuit 查找适合高亮的顶层对象，包括子电路内失败对象的外层显示。刷新异常会记录控制台信息并显示异常指示。

## 可学习的部分

校验不能只依靠绘图结果，应结合对象属性、网络和计算行为。错误需要关联对象，才能从文字提示定位回图纸。电路中的回路规则不能直接当作铁路业务规则使用。
