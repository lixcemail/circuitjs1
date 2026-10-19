# CircuitJS：命令、菜单、快捷键与工具栏

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/commands-and-shortcuts.md)

## 源码入口

[CommandManager.java](../src/com/lushprojects/circuitjs1/client/CommandManager.java) 的 menuPerformed 根据菜单名和条目字符串分发；[Menus.java](../src/com/lushprojects/circuitjs1/client/Menus.java) 创建菜单；[MyCommand.java](../src/com/lushprojects/circuitjs1/client/MyCommand.java) 封装菜单回调；[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 onPreviewNativeEvent 处理键盘；[Toolbar.java](../src/com/lushprojects/circuitjs1/client/Toolbar.java) 创建工具入口；[ShortcutsDialog.java](../src/com/lushprojects/circuitjs1/client/ShortcutsDialog.java) 管理快捷键。

## 多种输入如何到同一操作

菜单和部分键盘动作最终调用 menuPerformed。快捷键映射既可以是图元类名，也可以是 `cmd:<menu>:<item>` 形式的操作标识。CirSim 的 RUNSTOP_SHORTCUT_ACTION 是后一种示例。

工具栏可启动放置图元，也支持从图标开始拖动。UIManager 的原生事件预览会跟踪跨画布的工具栏拖动，并对这条路径处理 Esc 和方向修饰键。

## 焦点与上下文

键盘分发先处理对话框、滚动编辑弹窗等场景，再进入画布动作。图元、选区、只读状态会影响菜单可用性。桌面版还涉及文本输入框的原生编辑快捷键，见桌面文档。

因此快捷键不能只是一个全局 keydown 中无条件执行删除。应沿着事件上下文和操作入口一起理解。

## 实现限制

现有分发依赖字符串和分支，菜单是否禁用与操作是否可执行分散在相关模块。CommandManager 不代表每条命令都是可撤销对象。搜索元件的 SearchDialog 搜索可放置的元件条目，不是全站场设备实例搜索。

## 阅读练习

从菜单删除、键盘删除和右键删除三条路径追到 doDelete，检查选择集来源、只读限制及历史记录位置。
