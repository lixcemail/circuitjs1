# CircuitJS：自动布线与路径压缩

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/automatic-routing.md)

## 源码入口

[WireRouter.java](../src/com/lushprojects/circuitjs1/client/WireRouter.java)：initGrid、addObstacle、tryPatternRouting、routeWire、compressPath；[RoutedWireElm.java](../src/com/lushprojects/circuitjs1/client/RoutedWireElm.java)：自动布线导线的图元实现；[CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java)：addRoutingObstacle。

## 算法流水线

```text
图元障碍与已有导线 → 网格占用 → 简单路径尝试
    → 必要时 A* → 回溯路径 → 压缩连续同向点
```

routeWire 将端点换算为网格行列，先检查边界和目标可达性，再尝试模式路径。A* 的状态是 (row, column, direction)，不是只有位置；同一格从不同方向进入会影响下一次转弯代价。

优先队列按 fScore 排序，gScore 保存已知代价，cameFrom 保存前驱。启发式使用曼哈顿距离，并结合转弯与端口出线偏好等代价。

## 为什么需要方向状态

两条路径到达同一点，当前方向不同，后面可能分别需要零次或一次转弯。合并为一个位置状态会丢失影响总代价的信息。

当前搜索使用上下左右移动。compressPath 删除连续同向共线的中间点，保留起点、终点与拐点，降低图元保存和绘制复杂度。

## 失败与限制

越界、目标被阻挡和无可行路径会产生空结果等失败路径。源码使用若干路径偏好；不要将其描述为已证明对所有组合代价都找到全局最优解。

这里找到的是导线的几何走向，不是电气网络路径或站场进路。网格大小、障碍范围与搜索空间会影响速度与布局结果。
