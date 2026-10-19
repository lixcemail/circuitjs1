# CircuitJS：用户设置、自动恢复与保存状态

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/preferences-and-recovery.md)

## 源码入口

[UndoManager.java](../src/com/lushprojects/circuitjs1/client/UndoManager.java) 的 writeRecoveryToStorage/readRecovery/doRecover，[UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 getOptionFromStorage/setOptionInStorage/saveShortcuts/loadShortcuts，[CustomCompositeModel.java](../src/com/lushprojects/circuitjs1/client/CustomCompositeModel.java) 的 loadModelsFromStorage，[ScopeSerializer.java](../src/com/lushprojects/circuitjs1/client/ScopeSerializer.java) 的 saveAsDefault/loadDefaults。

## 不同数据共用浏览器存储

Storage 中既有界面选项和快捷键，也有恢复文档、用户模型及示波器默认配置。数据用各自 key 或前缀区分，并非一个统一带版本的项目数据库。

UndoManager 使用 circuitRecovery 保存当前序列化电路。启动读取 recovery，用户选择恢复后经 loader 加载。它是恢复入口，不等于每个工程文件都有独立备份历史。

## 未保存状态

CirSim 持有 unsavedChanges、savedFlag 等状态，加载与编辑路径分别更新。桌面实际写文件又经过异步桥接。判断“已保存”必须沿具体调用确认，不能仅凭点击菜单推断磁盘写入成功。

## 限制

localStorage 属于当前浏览器 origin，可能不支持、空间不足、被清理或与另一窗口共用。当前若干路径检查存储是否支持，但不能据此断言已有完整配额处理、多窗口冲突恢复和崩溃一致性协议。

## 学习重点

用户偏好、工程文件、撤销栈和恢复草稿生命周期不同。恢复数据的存在不应替代显式导出可移植工程文件。
