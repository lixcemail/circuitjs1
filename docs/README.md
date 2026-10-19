# 学习与开发文档

这里记录 CircuitJS 的实现，以及借鉴这些实现开发铁路信号站场图编辑器的方案。

## 文档导航

| 文档 | 回答的问题 | 主要内容 |
| --- | --- | --- |
| [CircuitJS：图元与编辑器实现](circuitjs-element-implementation.md) | 现有软件是怎么实现的？ | 28 节：编辑器核心、电路网络与计算、外围接口、性能、测试现状和工程交付 |
| [StationEdit：技术实现与开发指南](StationEdit/station-editor-element-design.md) | 站场图编辑器要怎么实现？ | 32 节：模型、绘制、交互、拓扑、文件、恢复、扩展能力和分阶段验收 |

CircuitJS 文档描述当前仓库的实际代码；StationEdit 文档属于建议设计，其中的接口和类型尚未在项目中实现。StationEdit 各节注明参考源码、关键方法及新增设计，便于对照学习。

两份正文覆盖从数据和绘图到交付的主要技术链路。电子器件的具体物理模型通过源码入口继续阅读；铁路制图和联锁领域规则需根据实际需求另行确定。

正文按连续阅读组织，同一主题的实现细节集中在原有章节；设备族、表达式与规则、资源加载等扩展通过章节内的小节阅读，无需切换多份主题文件。

## 建议阅读顺序

1. 阅读 CircuitJS 文档第 1 至 5 节，理解应用结构与图元抽象。
2. 阅读 StationEdit 文档第 1 至 5 节，明确第一版范围、模型和接口。
3. 对照两份文档的第 6 节，学习坐标、缩放、吸附、命中和高分屏处理。
4. 对照 [CircuitJS 第 11 节](circuitjs-element-implementation.md#11-鼠标交互与工具状态机) 和 [StationEdit 第 14 节](StationEdit/station-editor-element-design.md#14-鼠标交互与工具状态机)，学习工具、状态、预览、提交与取消。
5. 学习 CircuitJS 的绘制、属性、保存与撤销，再阅读 StationEdit 的连接、事务和保存方案。
6. 按下方技术点索引学习拓扑、几何布线、模板、导出、运行状态和外部接口。
7. 按 StationEdit 第 11、32 节分阶段实现，使用第 12、14、30 节及各节末尾的条件验收。

建议用“一个道岔、三条轨道、一个信号机、一段文字”作为贯穿开发的样例。

## 技术点索引

表中链接分别指向两份文档的对应章节。站场方案在相关章节中继续链接实际参考源码。

| 技术点 | CircuitJS 现有实现 | StationEdit 实现方案 |
| --- | --- | --- |
| 应用启动与职责组织 | [第 1 节](circuitjs-element-implementation.md#1-技术与启动结构) | [第 10 节](StationEdit/station-editor-element-design.md#10-建议的代码组织) |
| 图元抽象与创建 | [第 2–5 节入口](circuitjs-element-implementation.md#2-图元统一基类-circuitelm) | [第 2–5 节入口](StationEdit/station-editor-element-design.md#2-统一的是能力不是形状) |
| 设备族与新类型接入 | [第 5 节：元件族](circuitjs-element-implementation.md#元件族与扩展入口) | [第 13 节：设备族](StationEdit/station-editor-element-design.md#设备族与新增类型的接入) |
| 坐标、高分屏、命中与吸附 | [第 6 节](circuitjs-element-implementation.md#6-坐标与鼠标交互) | [第 6 节](StationEdit/station-editor-element-design.md#6-几何命中与显示) |
| 工具状态机与取消 | [第 11 节](circuitjs-element-implementation.md#11-鼠标交互与工具状态机) | [第 14 节](StationEdit/station-editor-element-design.md#14-鼠标交互与工具状态机) |
| 渲染、重绘和失效标记 | [第 12 节](circuitjs-element-implementation.md#12-渲染调度失效标记与性能测量) | [第 15 节](StationEdit/station-editor-element-design.md#15-渲染管线图层与重绘调度) |
| 几何算法与布局约束 | [第 3、6 节入口](circuitjs-element-implementation.md#3-图元通过哪些方法协作) | [第 16 节](StationEdit/station-editor-element-design.md#16-几何算法与布局约束) |
| 属性面板与输入校验 | [第 13 节](circuitjs-element-implementation.md#13-属性描述表单生成与输入校验) | [第 17 节](StationEdit/station-editor-element-design.md#17-属性面板与描述信息) |
| 选择、复制删除和批量变换 | [第 14 节](circuitjs-element-implementation.md#14-选择集合批量变换与剪贴板) | [第 9、21 节入口](StationEdit/station-editor-element-design.md#21-批量编辑分组与模板) |
| 菜单、搜索与快捷键 | [第 15 节](circuitjs-element-implementation.md#15-工具栏菜单搜索与键盘分发) | [第 28 节](StationEdit/station-editor-element-design.md#28-快捷键触屏主题与可访问性) |
| 显式连接与拓扑索引 | [第 16 节](circuitjs-element-implementation.md#16-图结构连接闭包与约束检查) | [第 7、18 节入口](StationEdit/station-editor-element-design.md#18-拓扑索引轨道区段与一致性) |
| 条件路径查询 | [第 16 节](circuitjs-element-implementation.md#16-图结构连接闭包与约束检查) | [第 19 节](StationEdit/station-editor-element-design.md#19-条件路径查询与候选路径高亮) |
| 几何自动布线 | [第 17 节](circuitjs-element-implementation.md#17-自动布线的网格代价与路径压缩) | [第 20 节](StationEdit/station-editor-element-design.md#20-几何自动布线与辅助对齐) |
| 复合模型、分组与模板 | [第 18 节](circuitjs-element-implementation.md#18-复合元件模型复用与上下文切换) | [第 21 节](StationEdit/station-editor-element-design.md#21-批量编辑分组与模板) |
| 文件、版本和保存成功确认 | [第 19、26 节入口](circuitjs-element-implementation.md#19-本地文件url-分享与兼容读取) | [第 22 节](StationEdit/station-editor-element-design.md#22-文档版本保存确认与迁移) |
| 图片、SVG、打印及数据导出 | [第 20 节](circuitjs-element-implementation.md#20-图片svg-与打印导出) | [第 23 节](StationEdit/station-editor-element-design.md#23-导出图片svg打印与业务数据) |
| 撤销、事务和修订控制 | [第 8 节](circuitjs-element-implementation.md#8-保存撤销与复制) | [第 8、24 节入口](StationEdit/station-editor-element-design.md#24-命令入口历史与修订控制) |
| 计算主循环与数值求解 | [第 21 节](circuitjs-element-implementation.md#21-仿真主循环与数值计算) | [第 25 节：迁移范围](StationEdit/station-editor-element-design.md#25-运行状态事件时间线与回放) |
| 表达式与自定义规则 | [第 21 节：解析与逻辑](circuitjs-element-implementation.md#表达式解析与自定义逻辑) | [第 29 节：受限规则](StationEdit/station-editor-element-design.md#受限表达式与自定义规则) |
| 状态采样、时间线和回放 | [第 22 节](circuitjs-element-implementation.md#22-运行状态波形采样与观测) | [第 25 节](StationEdit/station-editor-element-design.md#25-运行状态事件时间线与回放) |
| 性能、缓存和后台计算 | [第 12 节](circuitjs-element-implementation.md#12-渲染调度失效标记与性能测量) | [第 26 节](StationEdit/station-editor-element-design.md#26-性能空间索引与后台计算) |
| 外部接口和平台适配 | [第 23 节](circuitjs-element-implementation.md#23-外部接口嵌入与-websocket-桥接) | [第 27 节](StationEdit/station-editor-element-design.md#27-外部接口与平台适配) |
| 国际化、主题和触屏 | [第 24 节](circuitjs-element-implementation.md#24-国际化主题与触屏适配) | [第 28 节](StationEdit/station-editor-element-design.md#28-快捷键触屏主题与可访问性) |
| 启动资源与示例加载 | [第 24 节：加载链路](circuitjs-element-implementation.md#语言示例和启动参数的加载) | [第 28 节：启动与竞态](StationEdit/station-editor-element-design.md#资源语言与示例加载) |
| 恢复、诊断与错误处理 | [第 25 节](circuitjs-element-implementation.md#25-自动恢复错误定位与测试现状) | [第 29 节](StationEdit/station-editor-element-design.md#29-自动恢复诊断与可观测性) |
| 验证与测试 | [第 25 节：实际状态](circuitjs-element-implementation.md#25-自动恢复错误定位与测试现状) | [第 30 节](StationEdit/station-editor-element-design.md#30-自动验证样例与测试分层) |
| 构建、部署与桌面 | [第 26 节](circuitjs-element-implementation.md#26-构建静态部署与桌面文件桥接) | [第 31 节](StationEdit/station-editor-element-design.md#31-构建部署桌面与离线) |
| 离线资源与缓存 | [第 27 节](circuitjs-element-implementation.md#27-离线资源与缓存更新) | [第 31 节](StationEdit/station-editor-element-design.md#31-构建部署桌面与离线) |
| 迁移取舍与开发计划 | [第 28 节](circuitjs-element-implementation.md#28-技术迁移对照) | [第 32 节](StationEdit/station-editor-element-design.md#32-实现顺序与完成定义) |

## 维护约定

- 现有软件的行为、源码入口和限制，维护在 CircuitJS 实现文档。
- 站场图的模型、实现取舍和验收要求，维护在 `StationEdit/` 开发指南。
- 新增技术主题时同时说明现有实现与站场图方案；方案中的参考源应指向实际文件和方法。
- 区分“已有代码”“借鉴思路”和“新增设计”，避免把建议方案描述成已实现功能。
- 本 README 维护导航与阅读顺序，具体技术内容集中在两份正文中。
