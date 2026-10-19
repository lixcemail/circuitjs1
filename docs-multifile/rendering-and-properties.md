# CircuitJS：画布渲染、图层与样式

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/rendering-and-properties.md)

## 源码入口

[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 updateCircuit，[Graphics.java](../src/com/lushprojects/circuitjs1/client/Graphics.java) 的绘图封装，[CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的颜色和符号辅助方法，[ImageExporter.java](../src/com/lushprojects/circuitjs1/client/ImageExporter.java) 的上下文绘制。

## 一帧怎样生成

```text
检查尺寸及待分析状态 → 需要时推进仿真 → 清空画布
  → 设置视图和 dpr 变换 → 遍历图元 draw
  → 绘制端点、手柄及高亮 → 绘制其他界面内容
```

updateCircuit 同时承担仿真调度和绘制组织，不是只接收不可变场景快照的独立渲染函数。它设置打印或普通背景，并按显示选项影响元件颜色。

## 图元绘制

Graphics 封装线段、文字和图形操作，具体元件仍可能直接访问 Canvas context。ResistorElm 等在局部 save/transform/restore 中绘制符号。包围盒也可能在 draw 中更新，因此读源码要注意命中数据与绘制顺序的联系。

当前主要遍历 elmList 绘制，不是独立层树。某些对象还会在后续高亮阶段重画，以保证提示在上层。

## 状态与样式

颜色可以表示电压、功率、选择等信息，CircuitElm 提供公共配色。元件状态决定符号外观及电流动画。打印与导出临时改变部分配置，与交互渲染有共享状态。

## 属性面板独立阅读

Editable、EditInfo、属性解析和应用流程见 [属性编辑](property-editing.md)。渲染器处理属性的显示结果，属性面板负责编辑数据，不应把两者的职责混在同一绘图函数中。

## 可学习与限制

可学习图纸变换、Canvas 状态保存和辅助标记。没有完整空间索引、脏区域渲染或无状态导出架构的前提下，不应宣称这些机制已存在。
