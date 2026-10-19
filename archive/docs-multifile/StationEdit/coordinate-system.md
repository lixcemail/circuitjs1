# StationEdit：坐标系统、几何与命中

本文是站场图编辑器的建议设计，尚未在仓库中实现。TypeScript 风格示例用于说明结构，不限定技术选型。各节注明参考源码与新增设计。

[文档导航](../README.md) · [对应 CircuitJS 实现](../coordinate-system.md)

本文的 Point 与图元数据结构见 [图元模型与统一接口](element-abstraction.md)。坐标模块为 [鼠标工具](mouse-interaction.md) 提供位置转换，为 [连接维护](topology-and-connections.md) 提供端口几何。

## 几何、命中与显示

> **参考源与适用范围：** 参考 [MouseManager.java](../../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的 inverseTransformX/Y、transformX/Y、mouseSelect、setCircuitScale、[UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 updateCircuit。固定屏幕命中容差、统一派生几何与道岔局部坐标是本方案的具体约定，不代表现有代码已全部实现。

推荐顺序：图元数据 → 几何结果 → 绘制与命中。包围盒、端口和符号轮廓应来自一致的几何定义。

### 四种坐标与单位

| 坐标 | 用途 | 示例 |
| --- | --- | --- |
| 图元局部坐标 local | 定义图元自身符号和端口 | 道岔分支端相对于锚点的位置 |
| 图纸坐标 world | 保存站场布局、执行几何与连接检查 | 道岔锚点位于 (1000, 300) |
| 画布显示坐标 screen | 鼠标交互与显示尺寸，单位为 CSS 像素 | 鼠标在画布 (500, 200) |
| 画布像素坐标 device | Canvas 绘图缓冲区 | dpr=2 时对应 (1000, 400) |

本文 screen 特指画布相对坐标，不是操作系统桌面坐标。图纸采用自定义示意单位，不默认代表实际米数；需要里程或实际长度时另设业务数据。

图纸和局部坐标统一采用 X 向右、Y 向下，角度存弧度，正角度视觉上为顺时针。内部保留浮点数；只在明确的吸附或输出步骤舍入，避免反复转换累计误差。

### 视图变换与反变换

```ts
type Viewport = { scale: number; offsetX: number; offsetY: number };

function worldToScreen(p: Point, v: Viewport): Point {
  return { x: p.x * v.scale + v.offsetX,
           y: p.y * v.scale + v.offsetY };
}

function screenToWorld(p: Point, v: Viewport): Point {
  return { x: (p.x - v.offsetX) / v.scale,
           y: (p.y - v.offsetY) / v.scale };
}
```

接口要求 scale 有限且大于零，偏移量和点坐标也必须有限。第一版只支持视图均匀缩放和平移，不增加整张图纸的旋转或非均匀缩放。

```text
屏幕坐标 = 图纸坐标 × scale + translation
图纸坐标 = (屏幕坐标 - translation) / scale
```

图元位置保存图纸坐标，缩放和平移修改视图。Canvas 的设备像素比在渲染层处理；鼠标的 CSS 像素坐标不应再被重复乘设备像素比。

例如图纸点 (100, 200)、scale=2、偏移 (50, 30)，显示结果为 (250, 430)。反向转换应在浮点误差范围内得到原点。

### 浏览器事件到图纸坐标

```ts
const rect = canvas.getBoundingClientRect();
const screenPoint = {
  x: event.clientX - rect.left,
  y: event.clientY - rect.top,
};
const worldPoint = screenToWorld(screenPoint, viewport);
```

本例要求 Canvas 本身无 border、padding 和额外 CSS 缩放或旋转；其 CSS 内容尺寸与视图使用的显示尺寸一致。布局需要边框时可放在外层容器。若允许 CSS 变换，应扩展事件转换，不能直接套用减去 rect.left/top 的公式。

已提供画布相对坐标的事件适配器不再减一次偏移。建议在事件入口集中转换，工具仅接收约定好的 screen/world 数据。

### 平移、拖动与鼠标中心缩放

平移使用显示坐标差更新 offset，不修改图元数据。图元拖动使用图纸坐标差；缩放比例为 2 时，鼠标移动 20 个显示像素只对应 10 个图纸单位。

拖动过程中第一版固定视图变换，拖动预览始终由起始图纸点和当前位置计算；若允许拖动期间自动平移或缩放，需要额外调整拖动基准。

鼠标中心缩放的操作顺序：

```ts
const fixedWorld = screenToWorld(mousePoint, viewport);
const nextScale = Math.max(minScale,
  Math.min(maxScale, viewport.scale * zoomFactor));

viewport = {
  scale: nextScale,
  offsetX: mousePoint.x - fixedWorld.x * nextScale,
  offsetY: mousePoint.y - fixedWorld.y * nextScale,
};
```

zoomFactor 必须有限且大于零；缩放上下限为产品配置。滚轮增量应在事件适配层归一化，不直接把任意浏览器原始增量作为缩放倍率。

### 高分屏与显示尺寸

参考 [UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 setCanvasSize、devicePixelRatio 与 updateCircuit。

```ts
canvas.style.width = `${cssWidth}px`;
canvas.style.height = `${cssHeight}px`;
canvas.width = Math.round(cssWidth * dpr);
canvas.height = Math.round(cssHeight * dpr);

ctx.setTransform(
  dpr * viewport.scale, 0, 0, dpr * viewport.scale,
  dpr * viewport.offsetX, dpr * viewport.offsetY
);
```

尺寸应为有效正数，零尺寸布局暂缓绘制。Canvas 修改 width/height 会重置上下文状态，需重新设置变换、线型等；仅在尺寸或 dpr 改变时更新缓冲区。高分屏迁移或浏览器缩放后也要检查 dpr。

800 × 600 的显示尺寸在 dpr=2 时使用 1600 × 1200 的缓冲区。dpr 只影响渲染，文档坐标与鼠标 CSS 坐标不随它改变。

先设置合适的变换再清理画面：可用单位变换清空整个缓冲区，再设置图纸变换。绘制固定大小手柄或提示时，可改用仅含 dpr 的变换，以显示坐标绘制。

### 网格与端口吸附

源码参考：[UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的 setGrid、[CirSim.java](../../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 snapGrid。下面是支持任意正间距的建议公式，未直接沿用源码位运算：

```ts
function snapToGrid(value: number, gridSize: number): number {
  return Math.round(value / gridSize) * gridSize;
}
```

gridSize 必须有限且大于零。Math.round 在负数半格处向正无穷方向取整；如果希望半格远离零，应替换为明确的对称舍入策略并验证。

处理顺序建议为：鼠标转图纸坐标 → 搜索可连接端口 → 无可用目标时吸附网格 → 预览 → 提交连接或几何修改。端口吸附预览不等于已经建立业务连接。

移动选择集时，吸附统一锚点或总位移，然后对所有对象应用相同位移，避免逐个吸附破坏相对位置。

### 命中流程

1. 将鼠标转换成图纸坐标。
2. 用扩大了容差的包围盒筛选候选图元。
3. 对候选进行点到线段、符号轮廓或文字区域的精确检测。
4. 返回图元 ID、命中部位、手柄或端口 ID、距离。
5. 按当前工具的优先级决定结果。

容差先用屏幕像素定义，再计算 `图纸容差 = 屏幕容差 / scale`，让缩放后的点击难度保持稳定。当前选中图元的手柄可优先于主体；重叠对象还需明确绘制层级及选取顺序。

例如显示容差固定为 6 CSS 像素，scale 为 0.5、1、2 时，图纸容差分别为 12、6、3。手柄大小也可按显示像素固定。此处不再乘 dpr。

第一版可遍历所有图元。出现实测性能问题后，再增加空间索引和视口裁剪。

### 道岔几何

在局部坐标系中定义公共端、直向端和分支端，然后统一旋转、平移到图纸坐标。端口位置、符号绘制和命中轮廓使用同一组几何结果。

不要在绘制函数、命中函数和端口函数中分别写三套道岔尺寸公式。

例如局部公共端为 (-40, 0)、直向端为 (40, 0)、分支端为 (40, -20)。绕锚点旋转 θ 后：

```text
worldX = anchorX + localX × cosθ - localY × sinθ
worldY = anchorY + localX × sinθ + localY × cosθ
```

反向转换时令 dx=worldX-anchorX、dy=worldY-anchorY：

```text
localX =  dx × cosθ + dy × sinθ
localY = -dx × sinθ + dy × cosθ
```

命中点可以先转回局部坐标再检测。局部包围盒旋转后的图纸轴对齐包围盒，应由全部角点变换后取最小/最大值，不能只平移原矩形。

### 坐标模块接口与验收

将 worldToScreen、screenToWorld、localToWorld、worldToLocal、snapToGrid 放入 geometry，避免工具和图元各写转换公式。

- 正向再反向转换，应在规定误差范围内恢复原点。
- 平移和缩放不修改图元坐标，也不产生文档编辑历史。
- 鼠标中心缩放前后，固定图纸点对应的显示位置不变。
- 道岔旋转 90°、180° 后，绘制、端口和命中位置一致。
- dpr 从 1 改为 2，显示位置与点击结果不变，缓冲区尺寸改变。
- 不同缩放比例下命中与手柄显示尺寸保持稳定。
- 验证负坐标、半格吸附、极限缩放及非零页面布局偏移。
