# CircuitJS：表达式解析与自定义规则

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/expressions-and-rules.md)

## 源码入口

[Expr.java](../src/com/lushprojects/circuitjs1/client/Expr.java) 包含 Expr、ExprState 与表达式解析逻辑；[VCCSElm.java](../src/com/lushprojects/circuitjs1/client/VCCSElm.java) 等使用自定义函数；[CustomLogicModel.java](../src/com/lushprojects/circuitjs1/client/CustomLogicModel.java) 的 parseRules 解析逻辑规则；[CustomLogicElm.java](../src/com/lushprojects/circuitjs1/client/CustomLogicElm.java) 根据模型执行逻辑。

## 表达式技术

解析器按运算层次处理表达式，生成由运算类型、常数和子表达式组成的树。Expr.eval 根据类型递归求值，ExprState 保存输入、时间和先前值等状态。

```text
文本表达式 → 分词/解析 → 表达式树 → 在给定状态下求值
```

求值支持算术、比较、逻辑、条件和数学函数等操作。解析与求值分离，避免每个时间步重新解释全部文本。

## 自定义逻辑

CustomLogicModel 从文本定义解析输入输出模式，CustomLogicElm 引用模型并维护输出及历史值。模型本身还负责编辑和序列化。

自定义逻辑与连续数值表达式是不同机制，不能只看到“规则”二字就视为同一个语言。

## 可学习与限制

可学习运算优先级、树结构、求值上下文和规则错误提示。源码中的函数集合及数值行为服务于电路，不能直接表示完整站场或联锁规则，也不等同于任意 JavaScript 执行。
