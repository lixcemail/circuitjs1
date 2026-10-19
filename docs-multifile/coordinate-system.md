# CircuitJS：坐标系统

本文记录当前仓库的实际实现，用于源码学习。

[文档导航](README.md) · [对应站场图方案](StationEdit/coordinate-system.md)

## 坐标系统

阅读 [MouseManager.java](../src/com/lushprojects/circuitjs1/client/MouseManager.java)。

### 坐标转换

```text
screenX = worldX × transform[0] + transform[4]
screenY = worldY × transform[3] + transform[5]
```

inverseTransformX/Y 用于从鼠标屏幕位置得到图纸坐标；transformX/Y 完成正向转换。setCircuitScale 更新比例，并重新计算平移量，使鼠标位置或画布中心对应的图纸位置保持稳定。转换过程中有整数截断，因此会有取整误差。

### 平移与鼠标中心缩放

MouseManager.dragAll 使用相邻鼠标事件的显示坐标差，直接更新 transform[4/5]，从而平移视图；这条路径不逐个修改元件坐标。

setCircuitScale 先用旧变换取得鼠标位置对应的图纸点，再设置新比例，并计算：

```text
newOffsetX = mouseX - worldX × newScale
newOffsetY = mouseY - worldY × newScale
```

从菜单缩放时使用画布中心作为固定点。zoomCircuit 当前将比例限制在 0.2 至 2.5，并采用在旧比例上增加变化量的方式；它不是本站场方案中示例的乘法缩放。

### 网格吸附

UIManager.setGrid 将网格间距设置为 8 或 16，并生成掩码与舍入参数。CirSim.snapGrid 使用这些参数做整数位运算吸附；CircuitElm.drag 等路径会调用吸附逻辑。

这是针对当前整数坐标及特定网格间距的实现。任意间距网格、小数坐标与负数舍入策略不应直接套用其位运算公式。

### 高分屏与 Canvas 缓冲区

UIManager.setCanvasSize 分别设置 CSS 显示尺寸和 Canvas coordinate space：后者为显示尺寸乘 devicePixelRatio 后取整。例如显示为 800 × 600、像素比为 2 时，绘图缓冲区为 1600 × 1200。

updateCircuit 的图元绘制变换将视图比例及平移量同时乘设备像素比。鼠标交互路径使用事件的画布相对显示坐标，再通过 inverseTransformX/Y 转换，未将鼠标位置先额外乘一次像素比。

### 图元内部几何

CircuitElm.setPoints 根据两端计算长度、方向及垂直方向量，interpPoint 等方法帮助构造符号点。ResistorElm.draw 还使用 Canvas 的局部变换绘制电阻主体。

这些是局部几何计算与绘制变换的实例；当前代码没有统一规定所有元件都采用“锚点＋旋转角＋局部端口”的模型。

更完整的坐标分层、浮点精度、固定显示像素命中容差及道岔旋转约定，见 [站场图坐标方案](StationEdit/coordinate-system.md)。这些属于新项目设计。
