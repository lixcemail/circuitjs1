# CircuitJS：属性描述、面板与批量编辑

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/property-editing.md)

## 源码入口

[EditDialog.java](../src/com/lushprojects/circuitjs1/client/EditDialog.java) 同时声明 Editable 接口，并实现属性对话框；[EditInfo.java](../src/com/lushprojects/circuitjs1/client/EditInfo.java) 描述编辑项；[ResistorElm.java](../src/com/lushprojects/circuitjs1/client/ResistorElm.java) 展示 getEditInfo 和 setEditValue；[Adjustable.java](../src/com/lushprojects/circuitjs1/client/Adjustable.java) 与 [SliderDialog.java](../src/com/lushprojects/circuitjs1/client/SliderDialog.java) 处理可调参数。

## 描述属性再生成界面

Editable 通过 getEditInfo(index) 按序提供属性，通过 setEditValue(index, info) 接收结果，并提供对话框标题。EditInfo 包含数值、文本、选项、复选框、颜色、正数限制、错误等信息，也直接持有部分 UI 控件。

因此这里是属性描述与 GWT 控件结合的实现，不是纯粹与界面无关的 schema。

## 输入到数值

EditDialog.parseUnits 支持工程数值输入。apply 读取输入、解析数值、检查 positive/nonNegative 等标记，调用图元 setter，并显示属性项报告的错误。具体范围及纠正还可能在元件 setter 中处理。

不能从这条流程推断多个字段或多个对象的修改具有统一原子回滚；分析时应查看具体 setter 是否已经产生副作用。

## 滑动调参

Adjustable 将图元参数与滑块设置关联。滑块改变的是参数，随后的仿真和绘制反映变化。它适合学习参数控制，但站场图的设备编号和端口关系不适合直接作为连续滑块值处理。

## 阅读练习

追踪电阻阻值从 getEditInfo 到输入解析，再到 setEditValue、保存和重新加载。区分字段显示单位、内部数值与格式化文本。
