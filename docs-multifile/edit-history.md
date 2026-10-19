# CircuitJS：撤销重做与恢复

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/edit-transactions.md)

## 撤销重做


[UndoManager.java](../src/com/lushprojects/circuitjs1/client/UndoManager.java) 用两个栈保存序列化文档快照。UndoItem 同时记录比例及平移。撤销时把当前状态放入重做栈，再加载旧快照并恢复视图；重做过程相反。

pushUndo 清空重做栈，并对连续相同文档快照去重。本地恢复使用浏览器 Storage 的 circuitRecovery 项。

## 快照算法与调用顺序

```text
撤销：当前快照 → redoStack；undoStack 顶部 → 加载
重做：当前快照 → undoStack；redoStack 顶部 → 加载
```

UndoItem 除文档文本还记录视图比例与平移。loadUndoItem 使用 RC_NO_CENTER 读取，再恢复变换，避免加载默认居中破坏原视图。

当前 pushUndo 在相同文档去重前已经清空重做栈；历史调用又分散在鼠标与命令等路径。因此不能将新方案的“无变化操作绝不影响重做栈”当作现有实现保证。

## 快照的收益与代价

快照复用保存加载，不需要为每个图元实现反操作；代价是序列化成本、内存增长以及恢复时对象重新创建。引用旧 CircuitElm 实例的临时 UI 状态可能需要重新绑定。

后续优化可记录差异，但必须先明确原子操作范围。对站场编辑器，移动道岔和联动轨道共同构成一个用户操作。

## 学习练习

验证新建、移动、删除、属性修改各自在何时记录快照，比较撤销后再点击与撤销后再编辑的重做行为。本地恢复的独立生命周期见 [设置与恢复](preferences-and-recovery.md)。
