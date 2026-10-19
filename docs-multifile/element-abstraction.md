# CircuitJS：图元统一抽象与创建

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/element-abstraction.md)

## 图元统一基类 CircuitElm

核心文件：[CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java)。

它是抽象类，但很多方法提供默认实现或空实现，具体元件通过继承和重写适应差异。通用代码可以使用 CircuitElm 引用不同元件，调用共同方法。

| 共同信息 | 成员 | 用途 |
| --- | --- | --- |
| 基础几何 | `x/y`、`x2/y2` | 创建起点、拖出终点；简单双端元件的端点 |
| 派生几何 | `dx/dy`、`dn`、`point1/point2`、`lead1/lead2` | 方向、长度、点对象与引线位置 |
| 范围 | `boundingBox` | 绘制范围及命中候选筛选 |
| 编辑状态 | `selected`、`lastHandleGrabbed` | 选择及手柄状态 |
| 电气数据 | `nodes`、`volts`、`current`、`voltSource` | 电路仿真 |
| 公共上下文 | 静态 `app`、`sim`、颜色和字体 | 访问应用及绘图设置 |

这个基类既承载编辑能力，也承载电气能力。它不是只描述形状的数据对象；图元实现可以直接访问应用、绘图上下文和仿真模块。

## 图元通过哪些方法协作

| 能力 | 方法 | 现有实现要点 |
| --- | --- | --- |
| 计算几何 | `setPoints()` | 更新基础点、方向和长度，子类追加符号几何 |
| 绘制 | `draw(Graphics)` | 具体元件重写，绘制符号、文字和端点 |
| 创建时调整 | `drag()`、`dragPlace()` | 吸附、限制方向或按默认长度放置 |
| 整体移动 | `move(dx,dy)` | 更新两端坐标、平移包围盒并调用 setPoints |
| 局部移动 | `movePoint()` | 默认调整两个端点之一，处理方向与零尺寸限制 |
| 连接端点 | `getPostCount()`、`getPost()` | 默认两个端口，可由子类改变 |
| 编辑手柄 | `getNumHandles()`、`drawHandles()` | 默认手柄数量来自端口数量，默认绘制围绕两端坐标展开 |
| 命中距离 | `getMouseDistance()` | 默认双端元件采用线段距离；无端口图元采用中心距离 |
| 属性 | `getEditInfo()`、`setEditValue()` | 按索引提供编辑项并应用值 |
| 保存加载 | `dumpXml()`、`undumpXml()` | 公共坐标、flags 及子类属性 |
| 电气计算 | `preStamp()`、`stamp()`、`doStep()` 等 | 由仿真过程调用 |

默认行为偏向简单双端元件。不能推断任意复杂轮廓都已获得精确命中，或任意多端口元件都能直接使用默认手柄绘制。

## 以电阻为例理解继承

阅读 [ResistorElm.java](../src/com/lushprojects/circuitjs1/client/ResistorElm.java)。

```text
CircuitElm：公共坐标、移动、连接端点、默认行为
  └── ResistorElm：阻值、电阻符号、阻值编辑、电阻计算
```

推荐按以下顺序阅读：

1. 构造函数设置默认阻值或从旧文本参数恢复阻值。
2. setPoints 先调用父类，再计算引线与辅助点。
3. draw 设置包围盒、绘制引线和电阻符号，并按设置绘制数值及端点。
4. getEditInfo 返回阻值编辑项，setEditValue 应用属性。
5. dumpXml 调用父类保存公共数据，再保存阻值；undumpXml 恢复阻值。
6. stamp 和 calculateCurrent 参与电气计算。

显示、属性、保存和仿真行为集中在具体元件类中，便于沿一个元件理解完整生命周期。

[GraphicElm.java](../src/com/lushprojects/circuitjs1/client/GraphicElm.java) 将 getPostCount 重写为零，说明纯图形也复用同一基类，但不提供电气端口。

## 图元怎样创建

[ElementFactory.java](../src/com/lushprojects/circuitjs1/client/ElementFactory.java) 提供按类名和参数创建元件的入口，分别服务于新建和从数据恢复等场景。

[ElementFactoryGenerator.java](../src/com/lushprojects/circuitjs1/rebind/ElementFactoryGenerator.java) 在 GWT 编译阶段查找 CircuitElm 子类，根据构造函数生成创建分支；CirSim 使用 `GWT.create(ElementFactory.class)` 获得实现。

这是编译期生成的工厂机制，不能据此认为添加一个运行时脚本就能动态注册元件。新增元件还需要检查菜单、类型映射、序列化标识与快捷键等入口。
