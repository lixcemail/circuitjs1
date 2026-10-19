# CircuitJS：保存加载与复制粘贴

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/persistence-and-copy.md)

## 保存加载


CirSim.dumpCircuit 当前调用 [XMLSerializer.java](../src/com/lushprojects/circuitjs1/client/XMLSerializer.java)，输出文档及各图元的 XML 数据。CircuitElm.dumpXml 保存坐标和 flags，子类继续写入自身属性。

[CircuitLoader.java](../src/com/lushprojects/circuitjs1/client/CircuitLoader.java) 检查文本是否以 `<` 开头：满足时交给 XMLDeserializer，否则走旧文本格式读取逻辑。因此不能把当前存档概括为只有旧式逐行文本。


## 复制粘贴


[CommandManager.java](../src/com/lushprojects/circuitjs1/client/CommandManager.java) 负责菜单分发以及复制、删除、粘贴等操作。复制路径序列化图元，粘贴通过加载路径恢复对象并处理放置。

CommandManager 的名称不代表它采用每个操作一个 execute/undo 对象的设计；撤销机制仍以 UndoManager 的文档快照为核心。

## XML 结构与类型分发

XMLSerializer 保存根级选项，遍历图元调用 dumpXml 与 dumpXmlState，并保存模型、曲线及其他必要配置。XMLDeserializer 按标签识别元件、模型和示波器，结合类型映射与构造工厂恢复对象。

旧 dump 文本入口保留兼容用途。保存和读取的格式能力不完全等同：支持读取旧文本，不表示默认写出仍是旧文本。

## 粘贴的数据范围

copyOfSelectedElms 创建片段文档并序列化目标元件；doPaste 使用保留已有文档等读取选项添加对象，再处理选择及位置。具体图元的模型与外部数据还可能依赖额外序列化和缓存，见 [组合模型](composites-and-libraries.md) 与 [设备类型](element-families.md)。

电路主要依靠几何与网络分析重新关联，不能将其片段粘贴等同于站场稳定设备 ID 的引用重映射。

## 异常与兼容的实际界限

readCircuit 在非保留读取时会清理当前文档，再进入解析；现有文件读取不应被描述为“所有格式解析失败都自动保留原文档”的事务式加载器。

研究兼容性时应准备旧文本、当前 XML、带模型和曲线的文件，并比较解析后的语义。外部文件入口与展示导出见 [导入导出](import-export-and-print.md)。
