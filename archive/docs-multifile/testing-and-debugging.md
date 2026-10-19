# CircuitJS：测试分层、调试与回归验证

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/testing-and-debugging.md)

## 源码与脚本入口

[Test 脚本](../test.sh)、[构建工作流](../.github/workflows/build.yml)、[TestManager.java](../src/com/lushprojects/circuitjs1/client/TestManager.java)、[TestCreator.java](../src/com/lushprojects/circuitjs1/client/TestCreator.java)、[util/PerfMonitor.java](../src/com/lushprojects/circuitjs1/client/util/PerfMonitor.java)，以及 CirSim/UIManager 中的异常与调试输出。

## 当前测试能力的实际状态

- test.sh 查找 Gradle，运行 compileGwt/makeSite，再启动 Python 静态服务器并尝试打开浏览器。它不是自动断言编辑行为的测试集。
- build.yml 执行 GWT 编译和站点构建。
- TestManager 当前生效部分主要是空方法及默认返回值，大块录制与回归实现处于注释中。enabled 字段存在不表示整套回归功能已启用。
- TestCreator 的 enabled 为 false，创建测试夹具的主体也在注释中。

因此不能在学习文档中声称仓库已经自动覆盖鼠标交互、文件恢复或数值回归。

## 调试入口

浏览器控制台和源码映射适合追踪 GWT 编译后的调用。CirSim 的 debugger/console 与刷新异常指示可辅助定位；PerfMonitor 区分分析、计算和绘图耗时。

## 有效的阅读练习

用一个最小电路重复创建、拖动、保存、撤销和加载；在入口事件、图元几何方法与加载器设置断点。记录触发步骤、期望与实际文档，避免只截图判断状态。

本技术文档的检查与运行应用测试不同：文档链接通过不代表编辑行为已验证。
