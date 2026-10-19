# StationEdit：图元模型与统一接口

本文是站场图编辑器的建议设计，尚未在仓库中实现。TypeScript 风格示例用于说明结构，不限定技术选型。各节注明参考源码与新增设计。

[文档导航](../README.md) · [对应 CircuitJS 实现](../element-abstraction.md)

## 统一的是能力，不是形状

> **参考源与适用范围：** 参考 [CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 getPostCount、getPost、getNumHandles、drawHandles 和 [GraphicElm.java](../../src/com/lushprojects/circuitjs1/client/GraphicElm.java) 的 getPostCount。借鉴统一能力与子类差异；将锚点、手柄、端口分别建模是本站场方案的改进。

编辑器需要统一处理绘制、命中检测、选择、移动和保存，但不应要求所有图元拥有相同几何结构。

| 图元 | 持久化的几何输入 | 可编辑手柄 | 连接或关联 |
| --- | --- | --- | --- |
| 轨道线段 | 起点、终点 | 两个端点 | start、end 端口 |
| 简单道岔 | 锚点、朝向、左右开向、尺寸参数 | 移动、朝向控制 | common、normal、reverse 端口 |
| 信号机 | 位置、朝向 | 移动、朝向控制 | 关联轨道及面向方向 |
| 文字 | 位置、文本、字号 | 位置；需要时增加文本框手柄 | 无拓扑连接 |

三个概念必须分别表达：

- **锚点**：放置和移动图元的几何基准。
- **手柄**：用户可以拖动的编辑控制点。
- **端口**：业务连接的稳定标识及其位置。

手柄和端口可能在画面上重合，但职责不同。道岔三个端口不意味着可以独立拖动三个端口；文字可以有手柄，但没有轨道端口。

## 先确定数据归谁所有

> **参考源与适用范围：** 参考 [CirSim.java](../../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 elmList、transform 等状态、[CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 几何字段与 selected。本节按文档、派生几何、会话、运行状态重新划分归属，是建议设计，现有代码并未完全这样分离。

建议采用“文档持有数据，图元逻辑解释数据”的方式。相比前期把所有能力放进基类，这种组织便于保存、撤销和独立验证。

| 数据 | 所有者 | 是否进入工程文件 |
| --- | --- | --- |
| ID、名称、类型、位置、设备属性 | 文档中的图元记录 | 是 |
| 端口之间的连接、信号机关联 | 文档 | 是 |
| 包围盒、符号轮廓、端口位置 | 由图元数据计算的几何结果 | 通常否 |
| 当前选择、悬停、拖动预览 | 编辑会话 | 否 |
| 缩放、平移、网格设置 | 视图状态 | 可作为可选视图配置保存 |
| 占用、信号显示等运行状态 | 后续运行状态模块 | 默认不写入静态设计数据 |

不要把包围盒与真实几何作为两套独立可修改的数据。几何改变后，应重新计算派生结果或使缓存失效。

## 稳定身份与最小模型

> **参考源与适用范围：** 几何与端点建模参考 [CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 setPoints、getPost。稳定设备 ID、端口引用、Connection、轨道区段区分与信号关联结构均为本站场方案新增设计；这些类型不是从 CircuitJS 直接移植。

设备名称可修改，不能作为内部身份。文档引用应使用稳定 ID；端口 ID 在所属图元内部稳定。

```ts
type Point = { x: number; y: number };
type ElementId = string;

type TrackElement = {
  id: ElementId;
  kind: "track";
  name: string;
  start: Point;
  end: Point;
};

type TurnoutElement = {
  id: ElementId;
  kind: "turnout";
  name: string;
  anchor: Point;
  rotation: number; // 本示例约定弧度；UI 可以显示角度
  handedness: "left" | "right";
  length: number;
  branchOffset: number;
};

type SignalElement = {
  id: ElementId;
  kind: "signal";
  name: string;
  position: Point;
  rotation: number;
  attachment?: {
    trackId: ElementId;
    t: number; // 沿轨道 start → end 的归一化位置，范围 [0, 1]
    facing: "towardStart" | "towardEnd";
  };
};

type TextElement = {
  id: ElementId;
  kind: "text";
  position: Point;
  text: string;
  fontSize: number;
};

type DiagramElement =
  | TrackElement | TurnoutElement | SignalElement | TextElement;

type PortRef = { elementId: ElementId; portId: string };
type Connection = { id: string; a: PortRef; b: PortRef };

type StationDocument = {
  schemaVersion: 1;
  id: string;
  elements: DiagramElement[];
  connections: Connection[];
};
```

这里的 track 表示可绘制线段，尚未定义业务上的轨道区段。后续如果一个区段包含多条线段，应增加区段记录引用线段，避免让线段承担所有业务含义。

信号机的几何位置与 attachment 是独立数据。第一版可以约定轨道变形时保留 t，信号机符号仍由用户布置；若要自动随轨道移动，应另行明确偏移、方向与更新规则。

## 图元逻辑的公共接口

> **参考源与适用范围：** 参考 [CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 draw、move、movePoint、getMouseDistance、[ResistorElm.java](../../src/com/lushprojects/circuitjs1/client/ResistorElm.java) 的 具体方法重写。ElementBehavior、纯计算及返回新记录的约定为建议设计；CircuitJS 当前主要使用可变对象与继承。

第一版可以用一个类型注册表集中查找图元逻辑，不必同时建设复杂继承树与插件系统。

```ts
interface ElementBehavior<E extends DiagramElement> {
  geometry(element: E): ElementGeometry;
  draw(element: E, geometry: ElementGeometry, context: RenderContext): void;
  hitTest(element: E, point: Point, tolerance: number): HitResult | null;
  translate(element: E, delta: Point): E;
  editHandle(element: E, handleId: string, target: Point): E;
}
```

`ElementGeometry` 提供包围盒、手柄以及零个或多个端口；`RenderContext` 提供绘图能力和必要的显示信息，不传入整个应用对象。以上接口中的辅助类型应随实际功能定义，不必预先扩展为通用图形框架。

接口还必须说明行为约定：

- geometry、hitTest、draw 不修改文档；translate、editHandle 返回新记录。
- 坐标使用图纸坐标，数值必须有限；角度单位在模型内统一。
- 几何计算不依赖 Canvas，可在不启动界面的情况下验证。
- 非法编辑返回明确的失败结果或错误，不能静默留下部分修改。
- 文档模块负责更新关联图元，单个图元逻辑只计算自身变化。

如果实际使用 Java，可以采用抽象类加具体图元类实现相同约定。关键是调用者只需理解少量操作，几何细节集中在对应图元实现中。

## 设计自查

> **参考源与适用范围：** 本节为评估本站场图元抽象的设计问题；“绝缘节”是扩展场景，未在本文中实现。公共约定的源码对照为 [CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 图元公共方法。

新增一种“绝缘节”图元时，检查以下问题：

1. 是否主要增加数据、几何、画法、属性与类型注册？
2. 现有选择、移动、删除、撤销是否仍然可用？
3. 是否被迫填写无意义的第二端点或电气字段？
4. 如果它需要切分轨道，能否将切分与引用更新作为一次事务提交？
5. 保存加载能否保留它的身份与关联？

公共抽象应隐藏重复的编辑复杂度，同时允许设备表达真实差异。出现实际差异时再扩展接口，避免为了可能出现的设备提前搭建复杂框架。
