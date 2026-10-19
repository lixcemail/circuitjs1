# CircuitJS：JavaScript 接口、嵌入与外部通信

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/external-interface.md)

## 源码入口

[JSInterface.java](../src/com/lushprojects/circuitjs1/client/JSInterface.java) 的 setupJSInterface、getJSElements 及各 hook；[CircuitElm.java](../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 addJSMethods；[接口说明](../war/doc/js-interface.html)、[示例页面](../war/jsinterface.html)；[WebSocket 示例](../websocket/README.md)。

## 暴露接口

JSInterface 通过 GWT 的原生 JavaScript 互操作代码将导入导出、运行控制、时间、外部电压和元件访问等入口挂到 window.CircuitJS1。加载完成后调用 oncircuitjsloaded 通知宿主。

getJSElements 为元件附加 JavaScript 可访问方法，再返回数组。调用者接触的是当前应用对象相关的桥接结果，不是持久稳定的跨版本数据库记录。

## 生命周期钩子

onupdate、onanalyze、ontimestep 将刷新、分析与时间步事件暴露出去。SVG API 可能先异步加载脚本，再通过 onsvgrendered 返回内容，不能假定调用立即同步返回 SVG 字符串。

## 嵌入和通信

iframe 可嵌入应用；宿主如何访问其对象受浏览器同源规则约束。websocket/ 提供外部控制演示，但 JavaScript 接口本身不等于已经具备多人协同编辑和冲突合并。

## 迁移价值

可借鉴初始化通知、明确操作入口和异步完成回调。站场设备应使用稳定 ID，对外修改应经过命令和事务，不公开任意改写内部数组的能力。
