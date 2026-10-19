# StationEdit：仿真循环、矩阵与运行状态

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../simulation-and-runtime.md)

## 参考源与适用范围

借鉴 [SimulationManager.java](../../src/com/lushprojects/circuitjs1/client/SimulationManager.java) 的分析、时间推进与 stop，[UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的运行/暂停。下面的离散运行状态与事件流是新设计，不移植电路矩阵求解。

## 文档与运行状态分开

文档记录设备、轨道连接和配置；RuntimeState 记录演示时的区段占用、道岔位置及信号显示。状态变化触发重绘，不修改设计文件和文档撤销栈。

进入演示前验证文档，并固定 documentRevision。第一版演示期间禁止修改设计；退出演示后丢弃运行态或另存场景。

## 确定性事件推进

```ts
type RuntimeEvent = {
  sequence: number;
  time: number;
  targetId: string;
  type: string;
  payload: unknown;
};
```

运行模块按 (time, sequence) 处理输入，生成新状态与派生事件。明确时间单位、同一时刻顺序及暂停/单步语义，避免依赖渲染帧率决定业务结果。

第一版采用手工状态和明确事件即可；列车移动、占用判断和业务条件是独立后续需求。

## 视图绑定

渲染器通过设备 ID 读取状态，状态缺失显示未知；颜色之外同时使用符号或文字。外部实时数据经适配与校验后进入运行态，不直接改设计几何。

## 验收

同初始状态和事件流产生同结果；暂停期间时间不自动推进；不同帧率不改变事件顺序；演示失败保留设计文档；加载其他文档清除旧运行态和订阅。
