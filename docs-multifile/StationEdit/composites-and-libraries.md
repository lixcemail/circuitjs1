# StationEdit：组合图元、模板与模型库

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../composites-and-libraries.md)

## 参考源与适用范围

参考 [CompositeElm.java](../../src/com/lushprojects/circuitjs1/client/CompositeElm.java)、[CustomCompositeModel.java](../../src/com/lushprojects/circuitjs1/client/CustomCompositeModel.java) 的模型与端口定义，[CirSim.java](../../src/com/lushprojects/circuitjs1/client/CirSim.java) 的 pushContext/popContext。以下分组、模板版本和实例展开规则为新增设计。

## 区分三种能力

| 能力 | 保存内容 | 修改影响 |
| --- | --- | --- |
| 临时选择集 | 会话中的 ID 集合 | 当前操作 |
| 持久分组 | 分组 ID、子对象 ID | 组内布局 |
| 模板 | 可复制的设备及连接定义 | 新实例或显式升级的实例 |

第一版先支持选择集和复制；需要复用“道岔加信号机”等配置时增加模板快照，不自动引入无限嵌套编辑。

## 模板实例化

模板存局部坐标、内部连接、显式暴露端口及版本。放置时统一变换位置，为所有设备和连接分配新 ID，重映射内部引用，验证后一次提交。未暴露的内部端口不接收外部自动吸附。

模板中的设备编号作为占位值，实例化时生成或要求补齐，不能造成全站场编号重复。

## 版本与修改

模板升级不静默改写已有站场。文档可携带所用定义快照，避免换电脑后缺库无法打开。若采用关联实例，升级必须生成候选变更并展示端口变化及引用修复计划。

## 验收

同模板放置两次无重复 ID；外部只连接暴露端口；模板库缺失时仍可打开携带定义的文档；取消放置无副作用；分组删除不会保留悬空引用。
