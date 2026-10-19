# StationEdit：触摸、响应式与可访问操作

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../touch-and-accessibility.md)

## 参考源与适用范围

参考 [MouseManager.java](../../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的触摸桥接、[UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的布局调整、[Dialog.java](../../src/com/lushprojects/circuitjs1/client/Dialog.java) 和 [样式](../../src/com/lushprojects/circuitjs1/public/style.css)。统一 Pointer Events、键盘设备列表与状态反馈是新增设计。

## 第一版输入范围

先保证桌面鼠标与键盘，触摸加入后复用工具状态机。拖动阈值、命中容差及手柄大小按指针类型配置，图纸几何不变。

单指编辑、双指平移缩放采用明确规则。多指进入时先取消单指编辑预览，再进入视图手势；处理 pointercancel，不把浏览器滚动打断误当作成功提交。

## 响应式

窄屏把属性面板改为抽屉，仍保留可见的当前工具与取消入口。尺寸变化调整视图，不擅自改变设备位置；活动手势遇布局变化可统一取消，避免基线失效。

## 键盘与可访问性

提供可聚焦设备列表，与画布选择同步；支持按编号查找、方向键移动、属性编辑、删除与撤销。危险或不可用动作通过命令统一判定。

选中、错误和运行状态不能只靠颜色表达，增加轮廓、图标或文字。对话框管理焦点，关闭后返回触发位置。设备列表及属性控件有明确标签，工具栏标识当前工具。

## 验收

只用键盘可创建样例中的设备并修改属性；放大文字后面板仍可操作；触摸取消不提交；视口变化不移动图元；不同运行状态在无颜色情况下仍可区分。
