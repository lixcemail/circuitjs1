# CircuitJS：图元与编辑器实现

本文记录当前仓库已有软件如何实现图元抽象及其周边编辑能力，用于源码学习。站场图编辑器的设计方案单独维护在 [StationEdit 开发指南](StationEdit/station-editor-element-design.md)。

本文中的行为来自当前源码。示意代码只用于解释调用关系，精确实现以链接文件为准。

## 1. 技术与启动结构

项目使用 Java 编写主体，经 GWT 编译为浏览器 JavaScript，使用 Canvas 绘图。`app/` 提供 Electron 桌面外壳。

启动链路：

```text
circuitjs1.onModuleLoad()
  → loadLocale()
  → loadSimulator()
  → new CirSim()
  → CirSim.init()
  → updateCircuit()
```

入口是 [circuitjs1.java](../src/com/lushprojects/circuitjs1/client/circuitjs1.java)，GWT 入口声明位于 [circuitjs1.gwt.xml](../src/com/lushprojects/circuitjs1/circuitjs1.gwt.xml)。

[CirSim](../src/com/lushprojects/circuitjs1/client/CirSim.java) 是应用协调者，持有图元集合 `elmList`、视图变换 `transform`、各管理模块，以及分析和未保存等状态。部分模块在 UIManager 初始化过程中创建，不能把结构图理解为所有对象都直接在 CirSim 中实例化。

```text
CirSim
  ├── elmList: Vector<CircuitElm>
  ├── UIManager：界面、画布、刷新
  ├── MouseManager：指针交互
  ├── CommandManager：菜单和编辑操作分发
  ├── UndoManager：历史和本地恢复
  ├── CircuitLoader：加载入口
  └── SimulationManager：电路网络分析和计算
```

一次编辑通常沿以下路径流动：

```text
输入事件 → MouseManager 或 CommandManager → CircuitElm 几何/属性变化
  → 分析标记、恢复数据、历史及未保存状态更新
  → UIManager 调度刷新 → 必要的网络分析/仿真 → 图元 draw
```

文件加载从 CircuitLoader/XMLDeserializer 进入，JavaScript 接口和 Electron 文件桥接也使用当前应用的模型。共享状态主要通过 CirSim 和对象引用协作；跟踪调用链时，应同时记录各方法修改了哪些对象。

## 2. 图元统一基类 CircuitElm

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

## 3. 图元通过哪些方法协作

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

## 4. 以电阻为例理解继承

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

## 5. 图元怎样创建

[ElementFactory.java](../src/com/lushprojects/circuitjs1/client/ElementFactory.java) 提供按类名和参数创建元件的入口，分别服务于新建和从数据恢复等场景。

[ElementFactoryGenerator.java](../src/com/lushprojects/circuitjs1/rebind/ElementFactoryGenerator.java) 在 GWT 编译阶段查找 CircuitElm 子类，根据构造函数生成创建分支；CirSim 使用 `GWT.create(ElementFactory.class)` 获得实现。

这是编译期生成的工厂机制，不能据此认为添加一个运行时脚本就能动态注册元件。新增元件还需要检查菜单、类型映射、序列化标识与快捷键等入口。

### 元件族与扩展入口

理解单个电阻后，可以按类别继续阅读，而不必逐个浏览所有 Elm：

| 类别 | 代表源码 | 学习重点 |
| --- | --- | --- |
| 被动与动态元件 | [CapacitorElm.java](../src/com/lushprojects/circuitjs1/client/CapacitorElm.java)、[InductorElm.java](../src/com/lushprojects/circuitjs1/client/InductorElm.java) | 时间相关状态与电气贡献 |
| 非线性模型 | [Diode.java](../src/com/lushprojects/circuitjs1/client/Diode.java)、[DiodeModel.java](../src/com/lushprojects/circuitjs1/client/DiodeModel.java)、[MosfetModel.java](../src/com/lushprojects/circuitjs1/client/MosfetModel.java) | 实例与模型参数、数值近似和迭代 |
| 数字逻辑与芯片 | [GateElm.java](../src/com/lushprojects/circuitjs1/client/GateElm.java)、[ChipElm.java](../src/com/lushprojects/circuitjs1/client/ChipElm.java)、[DFlipFlopElm.java](../src/com/lushprojects/circuitjs1/client/DFlipFlopElm.java) | 类别公共行为、引脚、位序与状态更新 |
| 总线 | [WireElm.java](../src/com/lushprojects/circuitjs1/client/WireElm.java)、[BusSplitterElm.java](../src/com/lushprojects/circuitjs1/client/BusSplitterElm.java) | 位宽、端口 z 与位连接 |
| 复合模型与自定义逻辑 | [CompositeElm.java](../src/com/lushprojects/circuitjs1/client/CompositeElm.java)、[CustomLogicElm.java](../src/com/lushprojects/circuitjs1/client/CustomLogicElm.java) | 内部元件组合、模型与规则 |
| 外部数据与音频 | [DataInputElm.java](../src/com/lushprojects/circuitjs1/client/DataInputElm.java)、[AudioInputElm.java](../src/com/lushprojects/circuitjs1/client/AudioInputElm.java)、[AudioOutputElm.java](../src/com/lushprojects/circuitjs1/client/AudioOutputElm.java) | 时序数据、浏览器资源与释放时机 |
| 图形标注 | [TextElm.java](../src/com/lushprojects/circuitjs1/client/TextElm.java)、[BoxElm.java](../src/com/lushprojects/circuitjs1/client/BoxElm.java) | 无电气端口的可编辑图形 |

这些类别共用 CircuitElm，但端口、计算、状态和资源生命周期不同。总线用位宽和 Point.z 等信息区分不同位，不只是把线画粗，也不能直接对应站场中的并行轨道。

AudioInputElm 使用静态 audioFileMap 保留剪切粘贴和撤销重做需要的音频数据。外部资源不一定完整包含在普通元件字段中；研究保存与复制时，应同时检查序列化和资源缓存。

新增类型除了接入工厂，还要检查属性、端口、几何、绘图、存档标识、运行状态与资源释放。具体器件公式仍沿对应源码继续阅读。

## 6. 坐标与鼠标交互

阅读 [MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java)。

它保存当前鼠标模式、临时模式、悬停对象、正在创建的对象和拖动状态。事件根据模式分发，而不是让每个元件分别注册整套鼠标事件。

### 坐标转换

```text
screenX = worldX × transform[0] + transform[4]
screenY = worldY × transform[3] + transform[5]
```

inverseTransformX/Y 用于从鼠标屏幕位置得到图纸坐标；transformX/Y 完成正向转换。setCircuitScale 更新比例，并重新计算平移量，使鼠标位置或画布中心对应的图纸位置保持稳定。转换过程中有整数截断，因此会有取整误差。

### 平移与鼠标中心缩放

MouseManager.dragAll 使用相邻鼠标事件的显示坐标差，直接更新 transform[4/5]，从而平移视图；这条路径不逐个修改元件坐标。

setCircuitScale 先用旧变换取得鼠标位置对应的图纸点，再设置新比例，并计算：

```text
newOffsetX = mouseX - worldX × newScale
newOffsetY = mouseY - worldY × newScale
```

从菜单缩放时使用画布中心作为固定点。zoomCircuit 当前将比例限制在 0.2 至 2.5，并采用在旧比例上增加变化量的方式；它不是本站场方案中示例的乘法缩放。

### 网格吸附

UIManager.setGrid 将网格间距设置为 8 或 16，并生成掩码与舍入参数。CirSim.snapGrid 使用这些参数做整数位运算吸附；CircuitElm.drag 等路径会调用吸附逻辑。

这是针对当前整数坐标及特定网格间距的实现。任意间距网格、小数坐标与负数舍入策略不应直接套用其位运算公式。

### 高分屏与 Canvas 缓冲区

UIManager.setCanvasSize 分别设置 CSS 显示尺寸和 Canvas coordinate space：后者为显示尺寸乘 devicePixelRatio 后取整。例如显示为 800 × 600、像素比为 2 时，绘图缓冲区为 1600 × 1200。

updateCircuit 的图元绘制变换将视图比例及平移量同时乘设备像素比。鼠标交互路径使用事件的画布相对显示坐标，再通过 inverseTransformX/Y 转换，未将鼠标位置先额外乘一次像素比。

### 图元内部几何

CircuitElm.setPoints 根据两端计算长度、方向及垂直方向量，interpPoint 等方法帮助构造符号点。ResistorElm.draw 还使用 Canvas 的局部变换绘制电阻主体。

这些是局部几何计算与绘制变换的实例；当前代码没有统一规定所有元件都采用“锚点＋旋转角＋局部端口”的模型。

更完整的坐标分层、浮点精度、固定显示像素命中容差及道岔旋转约定，见 [站场图方案第 6 节](StationEdit/station-editor-element-design.md#6-几何命中与显示)。这些属于新项目设计。

### 命中

mouseSelect 先考虑已有对象的手柄，再遍历图元：通过 boundingBox 筛选，用 getMouseDistance 比较距离，同时补充端口距离判断。图元默认命中逻辑并非任意轮廓精确检测。

### 拖动

```text
onMouseDown：判断模式、准备操作并在相关路径记录历史
onMouseMove → mouseDragged：按模式执行移动、框选或调整端点
onMouseUp → endDrag：结束拖动、处理创建和连接相关收尾、刷新
```

mouseDragged 中有平移全部、拖行列、拖端点、移动选择集和重新布线等分支。endDrag 对新元件检查零尺寸，并在相关端点处尝试切分导线，再将有效元件加入集合。

这些路径直接修改对象，并在多个位置维护恢复数据和历史；当前实现不是统一的候选文档验证后原子提交模型。

## 7. 绘制与属性面板

[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 updateCircuit 既组织分析与仿真刷新，也组织画面绘制：清空画布，设置结合设备像素比的变换，遍历图元调用 draw，再绘制端点、手柄、高亮及其他界面内容。

[Graphics.java](../src/com/lushprojects/circuitjs1/client/Graphics.java) 封装绘图能力，具体图元也会访问其中的 Canvas 上下文。这里采用遍历绘制方式，不能当作已有完整场景树或空间索引。

部分元件在 draw 中更新包围盒，因此命中数据与绘制顺序存在联系。颜色可表达电压、功率和选择等信息，打印与导出又会调整部分显示设置；当前渲染不是只接收不可变场景的独立函数。

[EditDialog.java](../src/com/lushprojects/circuitjs1/client/EditDialog.java) 配合 EditInfo 组织属性编辑。具体元件按索引提供属性项，面板负责生成相应控件，再通过 setEditValue 将修改交回元件。

## 8. 保存、撤销与复制

### 保存加载

CirSim.dumpCircuit 当前调用 [XMLSerializer.java](../src/com/lushprojects/circuitjs1/client/XMLSerializer.java)，输出文档及各图元的 XML 数据。CircuitElm.dumpXml 保存坐标和 flags，子类继续写入自身属性。

[CircuitLoader.java](../src/com/lushprojects/circuitjs1/client/CircuitLoader.java) 检查文本是否以 `<` 开头：满足时交给 XMLDeserializer，否则走旧文本格式读取逻辑。因此不能把当前存档概括为只有旧式逐行文本。

XMLSerializer 还保存根级选项、图元 dumpXmlState、模型和示波器等配置；XMLDeserializer 根据标签、类型映射及工厂恢复对象。研究文件往返时，应使用包含模型和曲线的样例，不能只验证两个坐标是否保留。

### 撤销重做

[UndoManager.java](../src/com/lushprojects/circuitjs1/client/UndoManager.java) 用两个栈保存序列化文档快照。UndoItem 同时记录比例及平移。撤销时把当前状态放入重做栈，再加载旧快照并恢复视图；重做过程相反。

pushUndo 清空重做栈，并对连续相同文档快照去重。本地恢复使用浏览器 Storage 的 circuitRecovery 项。

具体顺序是先清空重做栈，再判断快照是否重复；不能把 StationEdit 的“无变化操作不影响重做栈”当成当前实现保证。loadUndoItem 使用 RC_NO_CENTER 加载后恢复比例和平移，避免默认居中覆盖历史视图。

快照复用了保存加载流程，代价是序列化、内存占用及恢复时重建对象。持有旧 CircuitElm 引用的悬停或其他临时状态需要随恢复重新绑定。

### 复制粘贴

[CommandManager.java](../src/com/lushprojects/circuitjs1/client/CommandManager.java) 负责菜单分发以及复制、删除、粘贴等操作。复制路径序列化图元，粘贴通过加载路径恢复对象并处理放置。

CommandManager 的名称不代表它采用每个操作一个 execute/undo 对象的设计；撤销机制仍以 UndoManager 的文档快照为核心。

## 9. 几何端点怎样进入电路网络

[SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 负责导线闭包、节点列表等分析。makeNodeList 遍历图元端点，通过点到节点的映射将端点关联到 CircuitNode，并记录 CircuitNodeLink。

这是几何端点参与构建电路网络的机制，还包含总线、子电路与导线等处理；不能仅凭画面交叉就概括所有连接语义。

[WireRouter.java](../src/com/lushprojects/circuitjs1/client/WireRouter.java) 解决导线几何路径：建立障碍网格，先尝试简单路径，失败后进行带方向状态和转弯代价的 A* 搜索，再压缩路径。它与电路网络的连接分析承担不同职责。

## 10. 建议的源码学习顺序

1. 入口与 CirSim：找到模块及图元集合。
2. CircuitElm 与 ResistorElm：理解公共约定和子类差异。
3. MouseManager：追踪从按下到创建一条导线的完整路径。
4. UIManager 与 Graphics：追踪数据怎样绘制到画布。
5. EditDialog：追踪一个阻值属性的修改。
6. XMLSerializer、CircuitLoader、UndoManager：追踪保存与恢复。
7. CommandManager：追踪复制粘贴和删除。
8. SimulationManager、WireRouter：分别理解电路连接与几何布线。

维护本文时应记录实际源码行为与限制。迁移取舍和新功能设计写入 StationEdit 文档，避免把建议误当作现有能力。

## 11. 鼠标交互与工具状态机

主要参考 [MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的模式常量、onMouseDown、mouseDragged、onMouseUp、endDrag、onMouseOut，以及 [UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的键盘事件处理。

### 模式与状态如何表达

当前实现以模式常量、布尔字段和对象引用共同表达交互状态，并非一个独立的显式状态机类。

| 字段 | 含义 |
| --- | --- |
| mouseMode | 用户选择的常驻操作模式 |
| tempMouseMode | 当前手势实际采用的模式，可受修饰键或命中结果影响 |
| mouseDragging | 是否进入按下后的拖动处理路径 |
| mouseElm | 当前指向的图元 |
| dragElm | 正在创建、尚未正式加入图元集合的对象 |
| selectedArea | 框选区域 |
| draggingPost | 当前调整的端点 |

例如用户常驻选择模式，按下时命中可操作手柄，可以临时进入 MODE_DRAG_POST；Alt 或鼠标中键可进入 MODE_DRAG_ALL 平移视图。释放后的 endDrag 将临时模式恢复为常驻模式。

### 按下、移动与释放

onMouseDown 先设置画布焦点、刷新命中结果，再依据鼠标键和修饰键选择临时模式。它还处理只读模式、开关点击、端点命中等特殊情况。相关路径调用 pushUndo，记录拖动起点；添加模式通过 constructElement 创建 dragElm。

onMouseMove 在 mouseDragging 为真时进入 mouseDragged。该方法先把显示坐标转为图纸坐标，再调用 dragElm.drag 或按临时模式分发到平移、框选、移动选择集、拖动端点等行为。选择模式拖动图元时有约 150ms 的延迟判断，用于减少移动设备上点击开关时的误拖。

onMouseUp 执行选择、切分等相关处理后进入 endDrag。endDrag 清理状态，拒绝创建零尺寸图元，处理自动切分导线和添加新对象，并在相关路径请求重新分析、更新恢复数据及历史。

### 预览与已存在对象的区别

新建图元使用 dragElm 表达临时对象，结束时才加入集合；移动已有图元则直接改变图元对象，并在相关路径写本地恢复数据。这两类路径没有统一的隔离预览事务。

因此，学习调用顺序时应同时检查状态字段、图元修改和历史记录发生的位置，不能把所有修改都理解为“松开时才提交”。

### 鼠标离开画布

onMouseOut 在仍处于拖动时调用 endDrag，避免丢失 mouseup 后持续误拖。它采用结束操作的策略，并非恢复原始数据的取消策略；有效的新图元仍可能在此时完成创建。

StationEdit 推荐的 Pointer Events、指针捕获、Esc 取消及窗口失焦回滚流程，属于新设计，见 [站场图工具状态机](StationEdit/station-editor-element-design.md#14-鼠标交互与工具状态机)。

## 12. 渲染调度、失效标记与性能测量

**参考源：** [CirSim.java](../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 timer、FASTTIMER、needAnalyze；[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 repaint、setSimRunning、updateCircuit；[PerfMonitor.java](../src/com/lushprojects/circuitjs1/client/util/PerfMonitor.java)。

运行状态使用约 16ms 周期的 GWT Timer 驱动更新，这只是请求更新的间隔，不保证实际达到固定帧率。暂停时仍可通过 repaint 请求绘制。

repaint 用 needsRepaint 合并重复请求：已有待执行刷新时不重复排队，延迟回调执行 updateCircuit 后复位。needAnalyze 设置分析标记并请求刷新，updateCircuit 再按标记触发网络分析。

这是两种重要技术：合并高频刷新请求，以及用失效标记推迟昂贵计算。当前 updateCircuit 同时组织仿真与绘图，不能视为计算线程和渲染线程已经分离。

PerfMonitor 用 performance.now 等计时来源记录嵌套过程；UIManager 在分析、仿真、绘图等步骤埋点。优化应先定位时间花在哪里，再决定是否使用缓存、裁剪或其他结构。

## 13. 属性描述、表单生成与输入校验

**参考源：** [EditInfo.java](../src/com/lushprojects/circuitjs1/client/EditInfo.java)、[EditDialog.java](../src/com/lushprojects/circuitjs1/client/EditDialog.java)、[ResistorElm.java](../src/com/lushprojects/circuitjs1/client/ResistorElm.java)。

EditDialog 按索引调用 getEditInfo，直到返回 null；根据 EditInfo 中的文本、选项、复选框、按钮等信息生成控件。EditInfo 还包含正数、非负数、错误信息、单位步长等属性。

这是一种以描述信息驱动表单的方式。它减少重复界面代码，但 EditInfo 本身持有 GWT 控件，数据描述与界面实现仍有耦合。具体值的读取、解析和应用路径应结合 EditDialog.apply 与元件 setEditValue 阅读。

对话框的 Apply 可以应用修改；Cancel 按钮调用关闭逻辑，不表示自动撤销此前已通过 Apply 应用的全部变化。不能直接将其理解为“整个打开期间都有隔离草稿”。

Editable 接口也声明在 EditDialog.java 中，除属性读写外还提供标题。parseUnits 解析工程数值，apply 结合 positive、nonNegative 等标记和具体 setter 检查输入；不能据此推断多字段修改已有原子回滚。

[Adjustable.java](../src/com/lushprojects/circuitjs1/client/Adjustable.java) 与 [SliderDialog.java](../src/com/lushprojects/circuitjs1/client/SliderDialog.java) 将元件参数关联到滑块，修改后由仿真和绘制反映结果。阅读时区分显示单位、内部数值和格式化文本；设备编号或端口关系不适合直接套用连续滑块。

## 14. 选择集合、批量变换与剪贴板

**参考源：** [MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的 selectArea、dragSelected、clearSelection；[CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 selectRect、flipX/Y/XY；[CommandManager.java](../src/com/lushprojects/circuitjs1/client/CommandManager.java) 的 prepareFlip、rotateCW/CCW、copyOfSelectedElms、doPaste、doDelete。

选择状态保存在元件对象中；框选构造图纸矩形，再逐个调用 selectRect。移动选择集遍历选中对象。旋转和镜像先准备共同中心，再对各图元变换，90° 旋转通过翻转组合实现。

基类 selectRect 默认判断选框与 boundingBox 是否相交，不要求完整包含；子类还可重写。dragSelected 对非纯图形选择集进行网格吸附，先遍历 allowMove，全部通过后移动，再联动 routedWireMap 中相关导线的共享端点。基类 allowMove 主要排查双端坐标重合，不是通用碰撞检测。

prepareFlip 使用选择对象、或无选择时所有对象的端点范围计算共同中心。具体元件可重写变换；当前坐标约定并不保证任意站场符号在镜像后自动保持正确业务语义。

复制将选择对象序列化，写入 circuitClipboard 本地存储。粘贴使用 RC_RETAIN 加载到现有集合，选中新对象，并根据新旧包围盒和鼠标位置选择偏移。删除路径逆序遍历集合，调用元件清理，并删除不再使用的示波器关联。

值得学习的不是某个固定偏移公式，而是“选区提取 → 序列化 → 恢复对象 → 成组定位 → 更新依赖”的完整过程。铁路设备的稳定 ID 重映射和端口引用清理仍需要另行实现。

## 15. 工具栏、菜单、搜索与键盘分发

**参考源：** [Toolbar.java](../src/com/lushprojects/circuitjs1/client/Toolbar.java)、[Menus.java](../src/com/lushprojects/circuitjs1/client/Menus.java)、[MyCommand.java](../src/com/lushprojects/circuitjs1/client/MyCommand.java)、[SearchDialog.java](../src/com/lushprojects/circuitjs1/client/SearchDialog.java)、[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 onPreviewNativeEvent、saveShortcuts、loadShortcuts。

菜单和工具栏通过命令分发请求操作；搜索使用现有元件标签及映射提供选择入口。工具栏还支持按住图标拖入画布，MouseManager 管理待放置状态，UIManager 全局事件预览跟踪跨出工具栏的手势。

键盘处理同时涉及对话框、临时弹窗、元件快捷键和编辑操作。删除、方向键移动、Esc 切回选择、缩放等行为集中在 UIManager。部分事件通过 repeat 检查避免重复触发，快捷键可序列化到本地存储。

Esc 在不同上下文下含义不同：工具栏拖动可以取消，普通编辑路径则包含切回选择模式等处理。不能假设所有普通图元拖动都已有完整回滚。

快捷键映射既可指向元件类名，也可使用 `cmd:<menu>:<item>` 操作标识，例如 RUNSTOP_SHORTCUT_ACTION，配置界面见 [ShortcutsDialog.java](../src/com/lushprojects/circuitjs1/client/ShortcutsDialog.java)。菜单可用性和实际执行条件分散在相关模块，应一起检查。SearchDialog 搜索的是可放置元件条目，不是图纸上已有设备实例。

## 16. 图结构、连接闭包与约束检查

**参考源：** [SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的 calculateWireClosure、makeNodeList、validateCircuit、findUnconnectedNodes；[FindPathInfo.java](../src/com/lushprojects/circuitjs1/client/FindPathInfo.java)；[CircuitNode.java](../src/com/lushprojects/circuitjs1/client/CircuitNode.java) 与 [CircuitNodeLink.java](../src/com/lushprojects/circuitjs1/client/CircuitNodeLink.java)。

端点与导线关系形成节点映射；节点记录相连的图元及端口索引。FindPathInfo 使用带 visited 的深度优先搜索，按检查类型限制允许经过的元件，用于电路回路或连通性判断。

可迁移的知识点包括：图的邻接表达、带条件遍历、访问标记、闭包和结构校验。电路中某些导线等价节点可以合并，但铁路路径必须保留设备、方向和通路语义，不能把电气闭包当作铁路拓扑模型。

calculateClosures 对电气连接分组，为相应分组构造矩阵。FindPathInfo 按 INDUCT、VOLTAGE、SHORT 等模式限制能经过的元件。analyzeCircuit 处理总线和显示端点并设置 needsStamp，较重的网络及矩阵准备延迟到运行所需路径；并非每次刷新都完整重建求解网络。

约束检查分布在创建时的 creationFailed、编辑时的 allowMove/movePoint、属性解析与 setter，以及网络分析和求解阶段。CircuitElm.validate 默认返回 true，validateCircuit 遍历并在失败时返回 false，不会生成完整诊断列表。makePostDrawList 还生成 badConnectionList 等显示信息；stop 记录原因及相关元件，UIManager 会寻找可高亮的顶层对象，包括内部子电路错误对应的外层对象。

## 17. 自动布线的网格、代价与路径压缩

**参考源：** [WireRouter.java](../src/com/lushprojects/circuitjs1/client/WireRouter.java) 的 initGrid、addObstacle、tryPatternRouting、routeWire、compressPath、placeWire。

模块先把场景转成离散网格，登记障碍及已有导线，再尝试简单模式路径。失败后使用 A* 风格搜索；状态包含行、列和方向，因此到达同一格但朝向不同可以有不同后续代价。代价还考虑转弯与端点逃离方向偏好。

搜索后重建路径，压缩不必要的中间点，再生成导线对象。当前搜索按上下左右移动，不是直接输出带铁路道岔约束的斜线布局。

需要区分障碍网格上的几何搜索与 FindPathInfo 的既有电路网络遍历：两者的输入、节点和允许移动规则不同。

优先队列按 fScore 排序，gScore 记录已知代价，cameFrom 保存前驱；启发式使用曼哈顿距离。相同格点从不同方向进入，后续转弯代价可能不同，因此不能合并成一个仅含位置的状态。越界、目标阻挡或无路时可能返回空结果；这些路径偏好不等于对所有组合代价的全局最优保证。自动布线图元及障碍接入还可阅读 [RoutedWireElm.java](../src/com/lushprojects/circuitjs1/client/RoutedWireElm.java) 和 CircuitElm.addRoutingObstacle。

## 18. 复合元件、模型复用与上下文切换

**参考源：** [CompositeElm.java](../src/com/lushprojects/circuitjs1/client/CompositeElm.java)、[CustomCompositeElm.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeElm.java)、[CustomCompositeModel.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeModel.java)、[CirSim.java](../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 contextStack 及上下文切换方法。

模型保存内部元件定义和外部端口信息，实例通过模型名引用模型；模型区分本地与全局集合，部分内容通过 subcircuit: 前缀写入本地存储。保存复合对象时处理模型定义与引用，避免同一模型重复写出。

编辑子电路需要保存并恢复上层上下文。可学习“定义与实例”“外部端口与内部网络”“上下文栈”三个概念，但它不是普通的绘图分组：复合元件还参与电气分析。

CompositeElm 根据内部节点编号关联子元件，再将外部节点映射进去。CustomCompositeModel 的 globalModelMap/localModelMap 管理定义，extList 保存外部端口，内部定义可存为 XML；dumped 等状态用于避免重复写出。

除 CustomCompositeElm 外，[CustomCompositeChipElm.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeChipElm.java) 展示了以芯片形式使用模型的实例入口，可对照外观与端口如何适配同一模型定义。

pushContext 保存外层文档、历史和视图，并建立局部历史；popContextAndGetChangedModels 恢复外层并返回深层模型变更。[SubcircuitBar.java](../src/com/lushprojects/circuitjs1/client/SubcircuitBar.java) 显示上下文路径。打开新文件需要 resetEditingContext，避免把旧模型的编辑栈带入新文档；模型库读取还兼容历史格式。

## 19. 本地文件、URL 分享与兼容读取

**参考源：** [LoadFile.java](../src/com/lushprojects/circuitjs1/client/LoadFile.java)、[ExportAsLocalFileDialog.java](../src/com/lushprojects/circuitjs1/client/ExportAsLocalFileDialog.java)、[ExportAsUrlDialog.java](../src/com/lushprojects/circuitjs1/client/ExportAsUrlDialog.java)、[CircuitLoader.java](../src/com/lushprojects/circuitjs1/client/CircuitLoader.java)、[circuitjs1.java](../src/com/lushprojects/circuitjs1/client/circuitjs1.java)。

浏览器加载依赖 File/FileReader；LoadFile 检查文件大小后异步读取，再调用加载器，并更新标题、最近文件名及未保存状态。当前大小检查阈值是 1280000 字节，属于该入口的限制，不代表所有导入入口都相同。

应用支持通过启动查询参数传入原始或压缩电路文本，也能按示例名或 URL 加载。XML 与旧文本格式共存，属于文件兼容读取，而不是通用版本迁移框架。

URL 分享涉及 LZString 压缩，短链接还依赖 shortrelay.php 等服务配置，不属于纯静态离线能力；远程文件加载受跨域和网络状态影响。

加载入口可能在解析和验证完成前清空当前电路，不能承诺所有失败都保留原文档。StationEdit 的“暂存区加载、验证后替换”是新增设计。

## 20. 图片、SVG 与打印导出

**参考源：** [ImageExporter.java](../src/com/lushprojects/circuitjs1/client/ImageExporter.java) 的 getCircuitAsCanvas、getCircuitAsSVG、drawCircuitInContext、doPrint；[canvas2svg.js](../war/canvas2svg.js)。

导出使用独立绘图上下文，按电路包围盒加边距计算输出范围；PNG 支持一定倍率的采样，并根据代码中的边长及面积上限缩小倍率，防止分配过大画布。上限是实现策略，不能当作所有浏览器的普遍能力保证。

SVG 路径动态加载 canvas2svg，复用类似 Canvas 的绘图调用。打印路径先生成图像，再建立临时打印内容并调用浏览器打印。

drawCircuitInContext 临时调整视图和部分显示选项，完成后恢复。新项目可学习复用绘制逻辑，但需要明确异常时的恢复及“导出是否携带交互高亮”等策略。

## 21. 仿真主循环与数值计算

**参考源：** [SimulationManager.java](../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的 analyzeCircuit、preStampAndStampCircuit、runCircuit、lu_factor、lu_solve；[CircuitMatrix.java](../src/com/lushprojects/circuitjs1/client/CircuitMatrix.java)；[SparseLU.java](../src/com/lushprojects/circuitjs1/client/matrix/SparseLU.java)；[INTERNALS.md](../INTERNALS.md)。

核心过程是建立电路节点和约束，将元件贡献写入矩阵，求解节点电压及部分电流，再推进时间。线性与非线性元件处理不同，非线性路径需要重复线性化和收敛判断，时间步长也参与控制。

普通求解通过 LU 分解与前后代入完成，代码提供稠密/稀疏求解分发；不应因为理论写成 X=A⁻¹B 就认为每个时间步都先显式构造逆矩阵。

INTERNALS.md 有历史背景和旧类名引用，阅读时要对照当前已迁移到 SimulationManager 的方法。对站场编辑器，可学习计算与显示的协调、失效重算和有界迭代；电气方程无需进入第一版站场编辑逻辑。

### 矩阵准备与时间预算

stampResistor 等方法把元件关系累积到矩阵和右端项。线性矩阵可提前分解并在多个时间步重复使用；非线性元件在迭代中更新项，根据收敛情况重新分解。AUTO 按矩阵规模选择求解器，当前仍保留稠密数据和转为 CSC 的路径，不能理解成全流程始终稀疏。

runCircuit 在一次刷新中推进多个时间步，调用 startIteration、doStep、stepFinished 和时间步钩子。收敛困难时减小步长，成功后逐步增大到最大步长；根据 minFrameRate 对应的帧预算退出计算循环。仿真时间与真实时间不同，矩阵奇异、非有限值或收敛失败会通过 stop 停止并提示。

### 表达式解析与自定义逻辑

参考 [Expr.java](../src/com/lushprojects/circuitjs1/client/Expr.java)、[VCCSElm.java](../src/com/lushprojects/circuitjs1/client/VCCSElm.java)、[CustomLogicModel.java](../src/com/lushprojects/circuitjs1/client/CustomLogicModel.java) 和 [CustomLogicElm.java](../src/com/lushprojects/circuitjs1/client/CustomLogicElm.java)。

```text
文本表达式 → 按运算优先级解析 → 表达式树 → 使用 ExprState 求值
```

ExprParser 解析，Expr.eval 按节点类型递归求值，ExprState 保存输入、时间和先前值。支持算术、比较、逻辑、条件及数学函数等操作；解析与求值分开，避免每个时间步重新解析文本。

CustomLogicModel.parseRules 则解析输入输出模式，CustomLogicElm 引用模型并维护输出及历史值。自定义数字逻辑与连续数值表达式是两套机制；两者都不等于任意 JavaScript 执行，也不能直接承担完整站场或联锁规则。

## 22. 运行状态、波形采样与观测

**参考源：** [Scope.java](../src/com/lushprojects/circuitjs1/client/Scope.java)、[ScopePlot.java](../src/com/lushprojects/circuitjs1/client/ScopePlot.java)、[ScopeTrigger.java](../src/com/lushprojects/circuitjs1/client/ScopeTrigger.java)、[ScopeFFT.java](../src/com/lushprojects/circuitjs1/client/ScopeFFT.java)、[ScopeManager.java](../src/com/lushprojects/circuitjs1/client/ScopeManager.java)。

ScopeManager 组织示波器，ScopePlot 在时间步中采样，并以循环缓冲区存储显示桶的最小/最大值；ScopeTrigger 处理触发，ScopeFFT 处理频域显示。使用位掩码环绕索引的缓冲区要求容量满足实现约定，不能任意改容量而保留相同索引公式。

这部分可学习有限历史、采样与绘制分离及多窗口布局。对于站场，后续设备状态时间线或事件记录可以借鉴有限缓冲，但不需要照搬电气波形和 FFT。

一个显示像素可能覆盖多个仿真步，保存桶内最小/最大值比只保留最后一个值更容易展示尖峰。ScopePlot.reset 处理容量、速度和数据保留；ScopeTrigger 用 ARMED、TRIGGERED、AUTO_RUN 等状态控制窗口，并用 validDataCount 避免显示旧槽位残留。ScopeFFT 构造实部/虚部后调用 FFT 计算幅值，[ScopeSerializer.java](../src/com/lushprojects/circuitjs1/client/ScopeSerializer.java) 保存曲线及显示配置，不等于归档全部历史原始数据。

## 23. 外部接口、嵌入与 WebSocket 桥接

**参考源：** [JSInterface.java](../src/com/lushprojects/circuitjs1/client/JSInterface.java)、[js-interface.html](../war/doc/js-interface.html)、[websocket/README.md](../websocket/README.md)。

JSInterface 在 window.CircuitJS1 上暴露运行控制、时间读取、元件查询、电压读写、电路导入导出和 SVG 请求，并提供加载、更新、分析、时间步与 SVG 完成回调。

SVG 请求包含异步脚本加载与完成回调，不应按同步返回 SVG 字符串使用。WebSocket 目录提供外围桥接示例，以 JSON 命令和 msgid 对应请求及响应，不等于主体内置了多人协作服务。

可学习小型外部接口与事件钩子；StationEdit 的外部修改仍应经过统一文档编辑入口，避免绕过历史和校验。

初始化完成通过 oncircuitjsloaded 通知宿主。getJSElements 为元件附加可访问方法后返回数组，调用者接触的是当前应用对象的桥接结果，不能当作跨加载稳定的数据库记录。iframe 宿主访问还受同源规则约束；示例页面见 [jsinterface.html](../war/jsinterface.html)。

## 24. 国际化、主题与触屏适配

**参考源：** [circuitjs1.java](../src/com/lushprojects/circuitjs1/client/circuitjs1.java) 的 loadLocale；[Locale.java](../src/com/lushprojects/circuitjs1/client/util/Locale.java)；[style.css](../src/com/lushprojects/circuitjs1/public/style.css)；[Color.java](../src/com/lushprojects/circuitjs1/client/Color.java)；[MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的 doTouchHandlers；[Toolbar.java](../src/com/lushprojects/circuitjs1/client/Toolbar.java) 的触屏处理。

语言选择从参数、本地存储及浏览器语言取得，加载文本映射；Locale.LS 查询翻译，缺失时返回原文。主题同时涉及 CSS 和 Canvas 颜色，不能只改页面样式就期望所有图元同步变化。

触屏路径将触摸事件适配为鼠标事件，双指距离变化用于缩放，并提供长按等处理。它不是基于 Pointer Events 的统一输入实现。文字输入、焦点可见性和画布内容的可访问替代仍需作为新项目需求明确设计。

### 语言、示例和启动参数的加载

语言优先级为 lang URL 参数、本地 language 设置、浏览器语言。英文直接使用空映射，其他语言请求 locale 文件；HTTP 非成功响应回落为空映射，但当前网络 onError 只记录日志，不能概括为所有失败都会继续启动。Locale.LS 还处理末尾 `~` 翻译语境标记；LSHTML 使用可信 HTML，仅适用于受控资源。

[Menus.java](../src/com/lushprojects/circuitjs1/client/Menus.java) 的 getSetupList/processSetupList/readSetupFile 从模块目录加载 setuplist.txt，解析分类和示例，再读入电路。[QueryParameters.java](../src/com/lushprojects/circuitjs1/client/QueryParameters.java) 解析启动选项，CirSim 据此决定文件、颜色、只读或运行状态；解析器较简单，不能视为通用健壮查询解析器。RequestBuilder 回调与应用初始化、默认示例之间有顺序关系，排查启动问题时需同时核对资源路径及回调链。

UIManager 根据窗口、菜单、侧边栏和工具栏计算画布尺寸，调整可能触发居中。触摸操作还需检查默认滚动、合成鼠标事件与拖动状态；原生控件的键盘能力不会自动让 Canvas 元件成为可聚焦设备树。

## 25. 自动恢复、错误定位与测试现状

**参考源：** [UndoManager.java](../src/com/lushprojects/circuitjs1/client/UndoManager.java)、[CirSim.java](../src/com/lushprojects/circuitjs1/client/CirSim.java) 的定时器异常处理；[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的异常提示；[TestManager.java](../src/com/lushprojects/circuitjs1/client/TestManager.java)、[TestCreator.java](../src/com/lushprojects/circuitjs1/client/TestCreator.java)、[test.sh](../test.sh)。

异常路径会记录控制台信息，部分绘制与计算过程有错误指示和停止机制；本地恢复快照用于重新加载电路。它不构成数据库事务或持久化成功保证。

当前 TestManager 的活动方法多为空实现，较完整的测试逻辑位于注释块中；TestCreator 的功能也被禁用且主体注释。不要因文件名或 enabled 常量就将这些模块视为可直接执行的完整回归框架。

test.sh 的实际用途是构建站点并启动本地 HTTP 服务器，不会自动断言鼠标行为、拓扑正确性或截图结果。构建通过和功能验证通过需要分别记录。

### 偏好、恢复与保存状态

UIManager 的 getOptionFromStorage/setOptionInStorage、saveShortcuts/loadShortcuts 管理偏好，CustomCompositeModel 和 ScopeSerializer 还保存用户模型及示波器默认配置。它们与 circuitRecovery 共用浏览器存储，用不同 key 或前缀区分，并不是统一带版本的工程数据库。

恢复快照经 readRecovery/doRecover 交给加载器，不等于每个工程都有独立备份历史。localStorage 属于当前 origin，可能不可用、超额、被清理或被其他窗口共享；已有支持检查不代表完整的配额、冲突和崩溃一致性协议。unsavedChanges、savedFlag 也由具体加载和编辑路径维护，应沿实际写文件调用确认保存结果。

调试时用最小电路重复创建、拖动、保存、撤销和加载，在事件入口、几何方法和加载器设断点，记录触发步骤及期望/实际数据。可结合浏览器控制台、可用的源码映射和 PerfMonitor 定位；文档检查通过不表示运行行为已验证。

## 26. 构建、静态部署与桌面文件桥接

**参考源：** [build.gradle](../build.gradle)、[README.md](../README.md)、[main.js](../app/main.js)、[preload.js](../app/preload.js)、[package.json](../app/package.json)。

本仓库 build.gradle 配置 GWT 编译及 makeSite 静态资源组装。构建产物位于 site，页面入口加载生成的脚本。依赖与 Gradle 版本以仓库配置和实际构建结果为准，本文不承诺其他版本组合的兼容性。

当前配置为 GWT 2.8.2、Java 8 源码级别，[wrapper](../gradle/wrapper/gradle-wrapper.properties) 固定 Gradle 8.7。这些是仓库配置事实，不是对兼容性或新项目选型的推荐。构建与预览入口为：

```bash
./gradlew compileGwt makeSite
python3 -m http.server 8000 --directory site
```

随后访问 circuitjs.html。compileGwt 生成 build/gwt/out/circuitjs1，makeSite 合并 war 静态资源和新产物到 site，并排除 war 中旧 circuitjs1 目录，避免旧产物覆盖新编译结果。首次运行需下载依赖，应分别排查 JDK、网络和预览端口问题。

[build.yml](../.github/workflows/build.yml) 在 master/dev 的推送及 PR 上编译构建；[deploy-pages.yml](../.github/workflows/deploy-pages.yml) 在 dev 推送时构建、上传 site 并部署 Pages。存在流程配置不表示某次部署已经成功，结果需查看运行记录。

Electron 主进程建立窗口并处理文件对话框等 IPC，preload 暴露读写文件及窗口操作桥接；应用也支持启动文件与多窗口。当前窗口配置包含 contextIsolation:false，属于现有配置事实，不应自动成为新项目模板。

preload 的 fs.writeFile 是异步操作，其现有桥接没有向上层返回明确的写入完成 Promise。因此“调用保存”与“磁盘写入成功”不是同一时刻，StationEdit 应单独定义成功确认协议。

主进程还处理单实例文件转交和系统打开文件事件，启动文本通过窗口变量交给应用；macOS 菜单使用原生编辑角色保留文本输入快捷键。renderer.js 主要是占位，主要业务仍在应用中。当前还设置 sandbox:false，preload 直接向 window 挂桥接函数，阅读时应与新应用的平台隔离方案区分。

## 27. 离线资源与缓存更新

**参考源：** [circuitjs.html](../war/circuitjs.html) 的 serviceWorker 注册；[service-worker.js](../war/service-worker.js)、[manifest.json](../war/manifest.json)、[update-service-worker.sh](../war/update-service-worker.sh)。

页面有 Service Worker 注册路径；脚本定义缓存名、预缓存资源、命中优先的请求处理和旧缓存清理。资源列表包含部署路径假设，makeSite 的复制规则也需要单独核对，不能据此承诺任意子目录部署后自动离线可用。

离线缓存保存的是应用资源，UndoManager 恢复保存的是用户电路，两类数据用途不同。新版本脚本与旧缓存混用、资源漏拷贝和首次离线打开都需要实际验证。

缓存模板见 [service-worker.orig](../war/service-worker.orig)：install 预缓存，fetch 优先命中缓存并追加成功请求，activate 清理其他版本；部分 URL 固定为 /circuit/。update-service-worker.sh 生成 service-worker.new，并要求人工更新版本和替换，不会因修改源码自动刷新清单。

makeSite 使用静态 include 列表，没有复制全部 JS/JSON 文件。应核对最终产物中的字体、语言、示例、图标、canvas2svg、manifest 和离线脚本；不能仅因 war 中存在这些文件就断言 site 可完整离线运行。

## 28. 技术迁移对照

| 技术 | 可学习内容 | 站场图需要补充的能力 |
| --- | --- | --- |
| 元件继承与工厂 | 公共行为、具体类型创建 | 稳定设备身份、铁路设备属性 |
| 坐标与 Canvas | 变换、高分屏、局部几何 | 浮点约定、道岔端口一致几何 |
| 鼠标模式 | 事件分发、拖动生命周期 | 隔离预览、取消、指针捕获 |
| 编辑和历史 | 选择、复制、快照恢复 | 原子关联修改、ID 重映射 |
| 电路图分析 | 邻接、闭包、条件 DFS | 方向、内部通路、区段与业务约束 |
| 自动布线 | 网格、障碍、方向代价 | 站场允许的线形、固定端口和不可达解释 |
| 复合模型 | 定义与实例、上下文 | 普通分组、模板版本与独立实例策略 |
| 数据和导出 | 兼容读取、PNG/SVG/打印 | 暂存验证、保存确认、确定性输出 |
| 外部接口 | 命令入口和事件回调 | 修订控制、输入校验和订阅清理 |
| 运行与观测 | 采样、时间步和有限历史 | 编辑/运行数据分离、事件重放 |
| 工程设施 | 构建、桌面桥接、缓存 | 可重复验证、部署资源一致性 |

本文覆盖与学习二维编辑器相关的主要技术链路，并对电气计算给出独立入口；不逐个展开所有电子元件的器件物理模型。需要实现某个具体器件时，再沿对应 Elm 的 stamp、doStep 和状态更新继续阅读。
