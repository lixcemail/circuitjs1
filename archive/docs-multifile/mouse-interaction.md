# CircuitJS：鼠标交互与工具状态

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/mouse-interaction.md)

## 鼠标交互与工具状态机

主要参考 [MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的模式常量、onMouseDown、mouseDragged、onMouseUp、endDrag、onMouseOut，以及 [UIManager.java](../src/com/lushprojects/circuitjs1/client/UIManager.java) 的键盘事件处理。

### 模式与状态如何表达

当前实现以模式常量、布尔字段和对象引用共同表达交互状态，并非一个独立的显式状态机类。

| 字段 | 含义 |
| --- | --- |
| mouseMode | 用户选择的常驻操作模式 |
| tempMouseMode | 当前手势实际采用的模式，可受修饰键或命中结果影响 |
| mouseDragging | 是否进入按下后的拖动处理路径 |
| mouseElm | 当前指向的图元 |
| dragElm | 正在创建、尚未正式加入图元集合的对象 |
| selectedArea | 框选区域 |
| draggingPost | 当前调整的端点 |

例如用户常驻选择模式，按下时命中可操作手柄，可以临时进入 MODE_DRAG_POST；Alt 或鼠标中键可进入 MODE_DRAG_ALL 平移视图。释放后的 endDrag 将临时模式恢复为常驻模式。

### 按下、移动与释放

onMouseDown 先设置画布焦点、刷新命中结果，再依据鼠标键和修饰键选择临时模式。它还处理只读模式、开关点击、端点命中等特殊情况。相关路径调用 pushUndo，记录拖动起点；添加模式通过 constructElement 创建 dragElm。

onMouseMove 在 mouseDragging 为真时进入 mouseDragged。该方法先把显示坐标转为图纸坐标，再调用 dragElm.drag 或按临时模式分发到平移、框选、移动选择集、拖动端点等行为。选择模式拖动图元时有约 150ms 的延迟判断，用于减少移动设备上点击开关时的误拖。

onMouseUp 执行选择、切分等相关处理后进入 endDrag。endDrag 清理状态，拒绝创建零尺寸图元，处理自动切分导线和添加新对象，并在相关路径请求重新分析、更新恢复数据及历史。

### 预览与已存在对象的区别

新建图元使用 dragElm 表达临时对象，结束时才加入集合；移动已有图元则直接改变图元对象，并在相关路径写本地恢复数据。这两类路径没有统一的隔离预览事务。

因此，学习调用顺序时应同时检查状态字段、图元修改和历史记录发生的位置，不能把所有修改都理解为“松开时才提交”。

### 鼠标离开画布

onMouseOut 在仍处于拖动时调用 endDrag，避免丢失 mouseup 后持续误拖。它采用结束操作的策略，并非恢复原始数据的取消策略；有效的新图元仍可能在此时完成创建。

StationEdit 推荐的 Pointer Events、指针捕获、Esc 取消及窗口失焦回滚流程，属于新设计，见 [站场图工具状态机](StationEdit/mouse-interaction.md)。

### 命中


mouseSelect 先考虑已有对象的手柄，再遍历图元：通过 boundingBox 筛选，用 getMouseDistance 比较距离，同时补充端口距离判断。图元默认命中逻辑并非任意轮廓精确检测。

### 拖动

```text
onMouseDown：判断模式、准备操作并在相关路径记录历史
onMouseMove → mouseDragged：按模式执行移动、框选或调整端点
onMouseUp → endDrag：结束拖动、处理创建和连接相关收尾、刷新
```

mouseDragged 中有平移全部、拖行列、拖端点、移动选择集和重新布线等分支。endDrag 对新元件检查零尺寸，并在相关端点处尝试切分导线，再将有效元件加入集合。

这些路径直接修改对象，并在多个位置维护恢复数据和历史；当前实现不是统一的候选文档验证后原子提交模型。
