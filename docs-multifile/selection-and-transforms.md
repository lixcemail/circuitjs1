# CircuitJS：选择集与几何变换

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/selection-and-transforms.md)

## 源码入口

- [MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java)：selectArea、dragSelected、dragPost、clearSelection。
- [CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java)：selectRect、move、movePoint、flipX、flipY、flipXY。
- [CommandManager.java](../src/com/lushprojects/circuitjs1/client/CommandManager.java)：prepareFlip、mirrorX/Y、rotateCW/CCW。

## 选择集怎样表达

selected 字段位于每个 CircuitElm 对象中。MouseManager 遍历集合执行框选、清除及全选，并更新相关菜单的可用状态；没有独立的持久化选择文档。

selectArea 用起始点和当前位置构造矩形，再调用各图元 selectRect。基类默认使用矩形与 boundingBox 相交判断，不要求整个图元被包含。具体类可重写，因此不能只凭框的方向推断选取规则。

## 整组移动

1. 根据目标是否已选中决定临时选择。
2. 非纯图形选择集会进行网格吸附。
3. 计算本次增量，先遍历检查 allowMove。
4. 所有检查通过后移动选中对象。
5. 对 routedWireMap 中相关自动布线导线的共享端点追加更新。
6. 请求重新分析，并清理临时选择状态。

普通基类 allowMove 主要排查与已有双端坐标重合，不是通用几何碰撞或铁路约束检查。

## 镜像与旋转

prepareFlip 计算选择对象或无选择时所有对象的两端坐标范围，生成公共中心并保存历史。mirrorX/Y 调用元件翻转，90° 旋转由对角翻转与垂直翻转按不同顺序组合。

元件可重写变换以维护自身几何与方向。这个实现建立在当前元件坐标约定上，不能保证任意站场符号的语义随镜像自动正确变化。

## 学习与验证

重点追踪一组选中电阻和导线移动前后的端点，再比较普通 WireElm 与 RoutedWireElm 的联动。几何对齐、选取范围与电气连接变化应分别观察。
