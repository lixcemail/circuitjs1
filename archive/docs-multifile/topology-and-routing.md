# CircuitJS：连接拓扑与几何布线

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/topology-and-connections.md)

## 几何端点怎样进入电路网络

[SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 负责导线闭包、节点列表等分析。makeNodeList 遍历图元端点，通过点到节点的映射将端点关联到 CircuitNode，并记录 CircuitNodeLink。

这是几何端点参与构建电路网络的机制，还包含总线、子电路与导线等处理；不能仅凭画面交叉就概括所有连接语义。

[WireRouter.java](../src/com/lushprojects/circuitjs1/client/WireRouter.java) 解决导线几何路径：建立障碍网格，先尝试简单路径，失败后进行带方向状态和转弯代价的 A* 搜索，再压缩路径。它与电路网络的连接分析承担不同职责。

## 图结构的数据

CircuitNode 记录节点，CircuitNodeLink 记录元件的哪一个端点关联到该节点。SimulationManager 在点映射、导线等价、总线检测和子图元处理中构造计算需要的网络。

图纸上的端口提供几何位置，分析网络才得到电气节点关系。导线、标签与总线等机制会改变简单的“同位置即一个节点”理解。

## 连通与条件路径

calculateClosures 对电气连接进行分组，构造分组对应的矩阵。FindPathInfo 使用 visited 数组避免重复访问，并按 INDUCT、VOLTAGE、SHORT 等模式限制可经过的元件类型。

这是一个重要思路：路径能否经过某个设备由问题的规则决定，不能只遍历一张无条件邻接表。但电路回路规则不是站场进路规则。

## 分析触发

编辑路径调用 needAnalyze 标记网络变化。analyzeCircuit 先进行总线和显示端点相关处理，并设置 needsStamp；较重的网络与矩阵准备在运行所需路径完成。不能认为每次画面刷新都会无条件完整重建求解网络。

## 专题入口

- 节点与矩阵怎样配合，见 [仿真循环](simulation-and-runtime.md)。
- A*、障碍和拐点压缩，见 [自动布线](automatic-routing.md)。
- 多层图元网络，见 [组合模型](composites-and-libraries.md)。
