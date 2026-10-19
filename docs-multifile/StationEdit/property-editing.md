# StationEdit：属性描述、面板与批量编辑

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../property-editing.md)

## 参考源与适用范围

参考 [EditDialog.java](../../src/com/lushprojects/circuitjs1/client/EditDialog.java) 的 Editable、apply、parseUnits，[EditInfo.java](../../src/com/lushprojects/circuitjs1/client/EditInfo.java) 的字段描述及 [ResistorElm.java](../../src/com/lushprojects/circuitjs1/client/ResistorElm.java) 的属性实现。以下声明式字段、批量 patch 与候选校验是建议设计。

## 字段模型

```ts
type PropertyField = {
  key: string;
  labelKey: string;
  type: "text" | "number" | "enum" | "boolean";
  unit?: string;
  readOnly?: boolean;
};
```

设备属性声明与实际控件分开。表单保存草稿，显示单位与存储单位分别约定，空字符串、缺失值和零不能混为一谈。名称使用稳定字段 key，不用翻译后的标题访问数据。

## 应用流程

编辑草稿 → 解析类型与单位 → 生成候选图元 → 更新端口几何及关联 → 文档校验 → 一次提交。字段错误标明 key 和设备 ID；失败保留草稿供修改，不写入文档。

方向、道岔开向等字段改变几何，必须执行连接维护。编号改变需要唯一性检查，但不更换稳定 ID。

## 批量编辑

多个对象只展示共同且兼容的字段；不同值显示“多个值”。只对用户明确修改的字段生成 patch，不用空值覆盖其余字段。整组验证通过才提交，整体一次撤销。

第一版“应用”提交一次历史，“取消”丢弃未应用草稿，不回滚先前已经应用的修改。此行为需要在 UI 中保持一致。

## 验收

非法数值、重复编号、负字号不会提交；批量修改中任一对象失败则全部不变；改朝向后端口和画面一致；保存加载保留实际数值而非本地化显示字符串。
