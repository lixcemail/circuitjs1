# CircuitJS：应用结构与源码学习路线

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/architecture-and-roadmap.md)

## 技术与启动结构

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


## 建议的源码学习顺序

1. 入口与 CirSim：找到模块及图元集合。
2. CircuitElm 与 ResistorElm：理解公共约定和子类差异。
3. MouseManager：追踪从按下到创建一条导线的完整路径。
4. UIManager 与 Graphics：追踪数据怎样绘制到画布。
5. EditDialog：追踪一个阻值属性的修改。
6. XMLSerializer、CircuitLoader、UndoManager：追踪保存与恢复。
7. CommandManager：追踪复制粘贴和删除。
8. SimulationManager、WireRouter：分别理解电路连接与几何布线。

现有源码行为与限制维护在 docs/ 对应主题，站场图方案维护在 StationEdit/ 对应主题。文档导航见 [README](README.md)。

## 一次编辑怎样流经模块

```text
输入事件 → MouseManager 或 CommandManager
  → CircuitElm 几何/属性变化
  → needAnalyze、恢复数据、历史及未保存状态更新
  → UIManager 调度 updateCircuit
  → 必要的网络分析/仿真 → 图元 draw
```

文件加载从 CircuitLoader/XMLDeserializer 进入，也会创建图元、恢复模型并触发后续刷新。JavaScript 接口和 Electron 文件桥接是其他入口，不是独立复制的一套电路模型。

当前共享状态主要通过 CirSim 和对象引用协作；这有利于跟踪完整调用链，但也意味着渲染、历史与仿真并非完全隔离。学习时应记录某方法修改哪些对象，而不仅是它调用了谁。

## 技术覆盖范围

本系列覆盖编辑内核、领域扩展、运行与观测、持久化、外部平台和工程化。电子器件按 [元件族](element-families.md) 介绍共性及阅读入口，具体物理公式以对应源码继续学习。导航见 [README](README.md)。
