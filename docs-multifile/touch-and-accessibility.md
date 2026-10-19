# CircuitJS：触摸、响应式与可访问操作

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/touch-and-accessibility.md)

## 源码入口

[MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的 doTouchHandlers、longPress、twoFingerTouch 与鼠标事件接口；[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 setCanvasSize 和移动界面判断；[style.css](../src/com/lushprojects/circuitjs1/public/style.css)；[Dialog.java](../src/com/lushprojects/circuitjs1/client/Dialog.java) 与 [Toolbar.java](../src/com/lushprojects/circuitjs1/client/Toolbar.java)。

## 触摸适配思路

MouseManager 通过原生 JavaScript 绑定触摸处理，将部分触摸手势适配到现有鼠标交互。长按和双指入口提供移动端相关行为，因此鼠标事件并不是唯一输入来源。

这与统一使用 Pointer Events 的方案不同。阅读时需要同时检查触摸默认行为、页面滚动、合成鼠标事件和现有拖动状态。

## 响应式布局

UIManager 根据窗口、菜单、侧边栏及工具栏计算画布显示尺寸，结合 CSS 对移动界面进行判断。画布变化可能触发居中等行为，不能只把 CSS 宽高变化视为纯视觉调整。

## 可访问性现状的判断

原生按钮、菜单与文本控件有助于键盘操作，但 Canvas 中的电路符号不会自动成为可聚焦的设备树。当前源码不能作为已完成站场无障碍体验的证据。

## 学习重点

区分设备符号的视觉尺寸与触摸目标范围；检查焦点、对话框关闭及快捷键是否与文本输入冲突。触摸场景还要避免轻点被误判成拖动。
