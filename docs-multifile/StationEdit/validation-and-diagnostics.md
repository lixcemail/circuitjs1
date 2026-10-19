# StationEdit：约束校验与错误定位

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../validation-and-diagnostics.md)

## 参考源与适用范围

参考 [CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 validate、creationFailed，[SimulationManager.java](../../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的 validateCircuit、stop、makePostDrawList，[FindPathInfo.java](../../src/com/lushprojects/circuitjs1/client/FindPathInfo.java) 的条件路径检查。以下诊断结构与站场约束是新增设计。

## 统一诊断结果

```ts
type Diagnostic = {
  code: string;
  severity: "error" | "warning";
  elementIds: string[];
  field?: string;
  messageKey: string;
};
```

解析失败、编辑拒绝、文档警告和程序异常分开表达。规则返回诊断，不直接弹窗口或修改设备；UI 负责列表、高亮、定位与字段提示。

## 校验层次

| 层次 | 示例 | 失败处理 |
| --- | --- | --- |
| 文件结构 | 缺失类型、非法数值、不支持版本 | 拒绝加载，保留原文档 |
| 身份与引用 | ID 重复、引用设备不存在、端口不存在 | 拒绝候选提交 |
| 几何 | 零长度轨道、无效尺寸、相连端口不重合 | 拒绝当前编辑 |
| 业务配置 | 编号重复、信号关联方向无效 | 按规则定义拒绝或警告 |
| 完整性提示 | 允许作为边界的未连接端口 | 提示待确认，不能一律判错 |

## 规则执行

提交时检查受影响对象及其引用邻居，保存或显式检查时运行全量校验。第一版规模小可统一全量校验，确认正确性后再做增量优化。

异步结果附带文档 revision；过期结果不覆盖当前诊断。点击诊断根据 ID 定位对象，设备已删除时安全忽略并刷新结果。

## 验收

同一文档的诊断稳定可复现；失败候选不写历史；恢复断开的引用后对应错误消失；未连接边界端口能与意外断开区分；文案翻译不改变规则代码。
