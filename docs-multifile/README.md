# 学习与开发文档

本系列按技术点对照学习 CircuitJS，并指导铁路信号站场图编辑器开发。当前覆盖 27 个主题，每个主题分别维护现有实现与 StationEdit 建议方案。

- `docs/`：基于当前仓库源码，解释实际流程、代码入口和限制。
- `docs/StationEdit/`：给出站场设计、接口、失败处理和验收条件，并注明参考源码。方案尚未在仓库实现。

## 覆盖范围

覆盖编辑内核、设备扩展、网络分析、运行观测、持久化和工程化。电子器件按类别介绍共同机制与源码入口，不逐个展开全部器件公式。站场业务规则需要按实际需求继续细化；技术设计不等于已具备完整联锁功能。

示例代码用于说明接口与数据结构，不是已经可以运行的新应用。文档检查通过也不表示运行行为或部署已验证。

## 技术主题导航

### 基础结构与图形内核

| 技术点 | CircuitJS：已有实现 | StationEdit：建议方案 | 内容 |
| --- | --- | --- | --- |
| 应用结构与路线 | [源码实现](architecture.md) | [开发方案](StationEdit/architecture-and-roadmap.md) | 模块组织、数据流、实施阶段 |
| 图元统一抽象 | [源码实现](element-abstraction.md) | [开发方案](StationEdit/element-abstraction.md) | 公共接口、数据归属、稳定 ID |
| 坐标系统 | [源码实现](coordinate-system.md) | [开发方案](StationEdit/coordinate-system.md) | 局部/图纸/显示坐标、缩放、吸附、高分屏 |
| 绘制与样式 | [源码实现](rendering-and-properties.md) | [开发方案](StationEdit/rendering-and-properties.md) | Canvas、绘制次序、状态叠加、导出上下文 |
| 设备类别与扩展 | [源码实现](element-families.md) | [开发方案](StationEdit/element-families.md) | 数字逻辑、总线、外部数据及站场设备接入 |

### 交互与编辑闭环

| 技术点 | CircuitJS：已有实现 | StationEdit：建议方案 | 内容 |
| --- | --- | --- | --- |
| 鼠标与工具状态机 | [源码实现](mouse-interaction.md) | [开发方案](StationEdit/mouse-interaction.md) | 命中、预览、提交、取消与指针捕获 |
| 选择与变换 | [源码实现](selection-and-transforms.md) | [开发方案](StationEdit/selection-and-transforms.md) | 框选、整组移动、旋转、镜像与联动 |
| 命令与快捷键 | [源码实现](commands-and-shortcuts.md) | [开发方案](StationEdit/commands-and-shortcuts.md) | 菜单、工具栏、焦点、只读和搜索 |
| 属性编辑 | [源码实现](property-editing.md) | [开发方案](StationEdit/property-editing.md) | 属性描述、单位、草稿与批量修改 |
| 编辑历史与事务 | [源码实现](edit-history.md) | [开发方案](StationEdit/edit-transactions.md) | 快照、原子提交、撤销重做 |
| 校验与诊断 | [源码实现](validation-and-diagnostics.md) | [开发方案](StationEdit/validation-and-diagnostics.md) | 结构、几何、业务约束与错误定位 |
| 触摸与可访问操作 | [源码实现](touch-and-accessibility.md) | [开发方案](StationEdit/touch-and-accessibility.md) | 响应式、键盘、焦点及多指交互 |

### 拓扑、复用与业务扩展

| 技术点 | CircuitJS：已有实现 | StationEdit：建议方案 | 内容 |
| --- | --- | --- | --- |
| 连接拓扑 | [源码实现](topology-and-routing.md) | [开发方案](StationEdit/topology-and-connections.md) | 端口网络、条件通路、路径查询 |
| 自动布线 | [源码实现](automatic-routing.md) | [开发方案](StationEdit/automatic-routing.md) | 障碍网格、A*、方向代价、拐点压缩 |
| 组合与模板 | [源码实现](composites-and-libraries.md) | [开发方案](StationEdit/composites-and-libraries.md) | 模型/实例、端口暴露、模板版本 |
| 表达式与规则 | [源码实现](expressions-and-rules.md) | [开发方案](StationEdit/expressions-and-rules.md) | 解析、求值、配置规则与执行约束 |

### 持久化与输出

| 技术点 | CircuitJS：已有实现 | StationEdit：建议方案 | 内容 |
| --- | --- | --- | --- |
| 工程文件与批量编辑 | [源码实现](persistence-and-copy.md) | [开发方案](StationEdit/persistence-and-copy.md) | XML/旧格式、版本、复制和删除引用 |
| 设置与自动恢复 | [源码实现](preferences-and-recovery.md) | [开发方案](StationEdit/preferences-and-recovery.md) | 本地设置、草稿、revision 和保存状态 |
| 导入、导出与打印 | [源码实现](import-export-and-print.md) | [开发方案](StationEdit/import-export-and-print.md) | 文件入口、PNG、SVG、打印和输出范围 |

### 运行、观测与性能

| 技术点 | CircuitJS：已有实现 | StationEdit：建议方案 | 内容 |
| --- | --- | --- | --- |
| 仿真与运行状态 | [源码实现](simulation-and-runtime.md) | [开发方案](StationEdit/simulation-and-runtime.md) | 矩阵、时间推进、静态设计与运行态 |
| 采样与时间线 | [源码实现](monitoring-and-timeline.md) | [开发方案](StationEdit/monitoring-and-timeline.md) | 环形缓冲、触发、FFT、事件记录与回放 |
| 性能与调度 | [源码实现](performance-and-scheduling.md) | [开发方案](StationEdit/performance-and-scheduling.md) | 刷新合并、预算、缓存、异步与测量 |

### 平台与工程化

| 技术点 | CircuitJS：已有实现 | StationEdit：建议方案 | 内容 |
| --- | --- | --- | --- |
| 外部接口 | [源码实现](external-interface.md) | [开发方案](StationEdit/external-interface.md) | JS 桥接、异步结果、嵌入与通信 |
| 资源与国际化 | [源码实现](resources-and-localization.md) | [开发方案](StationEdit/resources-and-localization.md) | 语言、示例、启动参数与加载竞态 |
| 桌面与离线 | [源码实现](desktop-and-offline.md) | [开发方案](StationEdit/desktop-and-offline.md) | Electron、文件桥接、缓存和更新 |
| 测试与调试 | [源码实现](testing-and-debugging.md) | [开发方案](StationEdit/testing-and-debugging.md) | 当前测试现状、测试分层与回归样本 |
| 构建与部署 | [源码实现](build-and-deployment.md) | [开发方案](StationEdit/build-and-deployment.md) | GWT、静态资源、CI、打包与发布 |

## 建议阅读与实施顺序

1. **建立模型**：应用结构 → 图元抽象 → 坐标 → 绘制 → 设备扩展。
2. **完成操作闭环**：鼠标状态机 → 选择与变换 → 命令 → 属性 → 编辑事务 → 校验。
3. **保留成果**：工程文件 → 自动恢复 → 导入导出，同时补核心测试。
4. **维护站场关系**：连接拓扑 → 模板；有需要再做自动布线和自定义规则。
5. **增加展示与集成**：运行演示 → 时间线 → 外部接口 → 性能优化。
6. **交付使用**：国际化、触摸与键盘、构建、桌面与离线按目标平台实施。

先用“一个道岔、三条轨道、一个信号机、一段文字”贯穿开发。第一版范围和验收见 [StationEdit 开发路线](StationEdit/architecture-and-roadmap.md)，不要求第一版同时实现所有扩展。

## 维护约定

- 每个技术点单独维护，新增主题同步更新此导航和两侧对照链接。
- 源码说明记录实际能力和限制；StationEdit 区分借鉴思路与新增设计。
- 站场方案保留可点击的参考文件及关键方法，不把文件名当作功能已完成的证据。
- 公共类型和规则在所属主题维护，其他主题链接引用，避免重复定义。
- 跨主题使用文件链接，不依赖其他文件的章节编号。
- README 维护范围、导航和阅读顺序；具体技术内容写在主题正文。
