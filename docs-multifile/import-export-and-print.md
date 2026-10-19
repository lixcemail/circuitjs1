# CircuitJS：文件入口、图片导出与打印

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/import-export-and-print.md)

## 源码入口

[LoadFile.java](../src/com/lushprojects/circuitjs1/client/LoadFile.java) 的 doLoad/doLoadCallback，[CircuitLoader.java](../src/com/lushprojects/circuitjs1/client/CircuitLoader.java) 的 readCircuit/loadFileFromURL，[ExportAsLocalFileDialog.java](../src/com/lushprojects/circuitjs1/client/ExportAsLocalFileDialog.java)，[ExportAsUrlDialog.java](../src/com/lushprojects/circuitjs1/client/ExportAsUrlDialog.java)，[ImageExporter.java](../src/com/lushprojects/circuitjs1/client/ImageExporter.java)，静态脚本 [canvas2svg.js](../war/canvas2svg.js)。

## 文件与 URL

LoadFile 使用 FileReader 读取本地文本，包含文件大小限制，回调交给加载器并设置标题。CircuitLoader 保留旧文本和 XML 入口；URL 分享使用序列化数据及 LZString 压缩等路径。

短链接依赖 shortrelay.php 等服务配置，不属于离线纯静态能力。远程文件加载还受浏览器跨域和网络状态影响。

## 图像和 SVG

ImageExporter 按电路范围建立导出画布，添加边距，限制目标画布尺寸和面积。drawCircuitInContext 在给定上下文绘制元件及端点；SVG 路径按需加载 canvas2svg.js，再以兼容上下文收集绘图调用。

导出过程临时调整部分视图与显示设置，然后恢复。它不是一个完全无状态的独立渲染器；新项目应留意异常时的恢复。

## 打印

printCanvas 将图像放入临时打印覆盖层并调用浏览器打印，使用打印样式隐藏其他界面，然后移除临时节点。它不是按铁路图纸规格完成的多页排版系统。

## 学习重点

工程文件保留可编辑语义，PNG/SVG 保留展示结果，两者用途不同。导出应处理空图、超大范围、外部字体和桌面/浏览器保存方式的差异。
