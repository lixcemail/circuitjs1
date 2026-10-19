# StationEdit：文件入口、图片导出与打印

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../import-export-and-print.md)

## 参考源与适用范围

参考 [LoadFile.java](../../src/com/lushprojects/circuitjs1/client/LoadFile.java) 的 FileReader 入口、[ImageExporter.java](../../src/com/lushprojects/circuitjs1/client/ImageExporter.java) 的 getCircuitAsCanvas/getCircuitAsSVG/drawCircuitInContext、[ExportAsUrlDialog.java](../../src/com/lushprojects/circuitjs1/client/ExportAsUrlDialog.java)。独立导出快照、统一解析入口与站场打印布局是新设计。

## 工程导入

文本、文件、模板和外部调用共用 parse → migrate → validate → replace/merge 入口。替换前保留当前文档，异步解析带 requestId 防止后到的旧请求覆盖新选择。

合并导入必须重映射 ID 和内部引用；替换导入按新文档处理历史与运行状态。文件大小和对象数量设上限，失败提供原因，不静默跳过未知设备。

## 图片与 SVG 导出

以不可变文档快照和独立视图参数渲染，不修改编辑器当前视图。导出选项明确范围（全图/选区）、背景、比例、字体和是否包含运行态；默认排除手柄、悬停和预览。

输出像素宽高及面积设预算，超大图允许降低分辨率或改用矢量输出。SVG 需验证文本、虚线、符号和变换是否被后端支持；不能假定所有 Canvas 调用都能无损转换。

## 打印

第一版先做全图适配单页，标明比例和页边距。后续再支持纸型、分页、重叠边界、图框及版本信息。图纸示意单位不自动成为实际工程比例。

## 验收

导出前后文档、选择、视图不变；取消保存不清除未保存标记；空图不报非法尺寸；极大图不创建无限画布；文件往返保留编辑语义，图片导出不被误作工程备份。
