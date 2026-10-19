# StationEdit：自动布线与路径压缩

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../automatic-routing.md)

## 参考源与适用范围

参考 [WireRouter.java](../../src/com/lushprojects/circuitjs1/client/WireRouter.java) 的 routeWire、compressPath，以及 [RoutedWireElm.java](../../src/com/lushprojects/circuitjs1/client/RoutedWireElm.java)。站场方向限制、业务端口兼容、搜索预算及显式失败类型是新增设计。

## 先限定用途

第一版以人工绘制与端口吸附为主。自动布局只处理明确允许的辅助连线或轨道示意连接；不能把现有四向导线算法直接用于所有道岔斜线。

几何布线与 [连接拓扑](topology-and-connections.md) 分开，路径搜索不得暗中增加业务连接。

## 输入和输出

```ts
type RoutingResult =
  | { ok: true; points: Point[] }
  | { ok: false; reason: "blocked" | "budget" | "invalid" };
```

输入包括图纸端点、出入方向、障碍、网格间距、允许方向与最大搜索预算。输出仅为候选点列，由文档模块验证和提交。

## 实施步骤

1. 在有限范围建立障碍网格，按符号间距膨胀障碍。
2. 对端口附近留出合法出线通道。
3. 先尝试直线和少量拐点模板。
4. 需要时使用带方向的搜索，结合长度和拐点代价。
5. 压缩共线点，重新检查实际符号间距、交叉与连接条件。
6. 显示预览，确认后一次提交。

允许斜线时必须重定邻居、代价与启发式；不要只增加对角邻居却继续无条件使用原启发式。超出预算返回失败，不阻塞指针交互。

## 验收

检查无障碍、绕障碍、不可达、负坐标和端点被包围情况；路径压缩不改变走向；取消不修改文档；几何相交不自动连接；同输入在相同配置下可复现。
