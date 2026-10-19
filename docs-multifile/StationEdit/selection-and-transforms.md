# StationEdit：选择集与几何变换

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../selection-and-transforms.md)

## 参考源与适用范围

借鉴 [MouseManager.java](../../src/com/lushprojects/circuitjs1/client/MouseManager.java) 的选择集移动与 selectArea，以及 [CommandManager.java](../../src/com/lushprojects/circuitjs1/client/CommandManager.java) 的镜像和旋转。以下独立选择集、变换计划和业务约束是新增设计。

## 数据与接口

会话持有 selectedIds: Set<ElementId>，文档不保存选中状态。删除或加载文档后清理不存在的 ID。

```ts
type TransformIntent = {
  ids: string[];
  operation: "translate" | "rotate90" | "mirrorX";
  pivot: Point;
  delta?: Point;
};
```

几何计算生成候选图元，文档编辑模块维护相关连接并验证，最后一次提交。不允许一半设备已移动、另一半失败。

## 操作规则

- 点击选择、Shift 切换选择、框选完全包含，采用与 CircuitJS 默认相交框选不同的明确规则。
- 图元整体锁定时可以被选中和查看，变换命令则拒绝包含它的候选并提示原因。
- 移动使用一个公共位移，统一吸附，保持相对布局。
- 旋转使用共同枢轴，四次 90° 旋转应恢复原几何，允许数值误差。
- 镜像改变道岔开向及符号方向，但不按端口数组位置重命名 common/normal/reverse；业务端口语义由设备实现维护。
- 相连设备同在选择集时，连接只维护一次；选区外相连轨道按连接联动规则更新。

## 变换流程

冻结基线 → 生成所有目标图元 → 更新关联端点和信号关联 → 检查几何与引用 → 预览或提交。图元的平移、旋转、镜像可作为内部行为，不要求所有类型支持所有变换。

## 验收

混合选择轨道、道岔、信号机和文字，移动后相对位置不变；拒绝操作不改变文档；镜像两次恢复；一次撤销恢复整个选择集和选区外被联动的轨道。
