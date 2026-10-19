# StationEdit：命令、菜单、快捷键与工具栏

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../commands-and-shortcuts.md)

## 参考源与适用范围

参考 [CommandManager.java](../../src/com/lushprojects/circuitjs1/client/CommandManager.java) 的 menuPerformed、[UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 onPreviewNativeEvent、[Toolbar.java](../../src/com/lushprojects/circuitjs1/client/Toolbar.java) 和 [ShortcutsDialog.java](../../src/com/lushprojects/circuitjs1/client/ShortcutsDialog.java)。命令注册表、统一可用性判断与实例搜索为新增设计。

## 统一命令入口

```ts
type Command = {
  id: string;
  canExecute(context: EditorContext): boolean;
  execute(context: EditorContext): void | Promise<void>;
};
```

菜单、工具栏、快捷键和未来外部调用共享命令 ID。execute 内仍检查上下文，不能只依靠按钮灰显。命令把修改意图交给文档事务模块，不自己修改多个数据副本。

## 命令与工具的区别

删除、撤销、保存是执行后结束的命令。选择轨道工具只是切换后续指针行为，不创建文档历史。命令内部可以启动工具，但两者生命周期分开。

## 输入规则

1. 模态窗口优先，其次属性输入框，再到画布。
2. 输入框内保留文本复制粘贴与删除；Esc 可先结束当前输入或交互。
3. 活动拖动会话中拒绝其他文档修改，切换工具先取消预览。
4. 保存等异步命令显示进行中状态，重复触发不并发覆盖文件。
5. 快捷键设置按命令 ID 保存，检测同一上下文内的冲突。

## 设备搜索

将“搜索可放置设备类型”和“按编号查找已存在设备”分开。实例搜索结果包含稳定 ID，选择结果后设置选区并把设备移入视口；改名不改变目标身份。

## 验收

菜单和快捷键对同一命令产生相同文档结果；只读时所有入口都拒绝编辑；文本框 Delete 不删除设备；异步保存失败可重试且不会产生多次覆盖。
