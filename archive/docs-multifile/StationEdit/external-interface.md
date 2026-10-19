# StationEdit：JavaScript 接口、嵌入与外部通信

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../external-interface.md)

## 参考源与适用范围

参考 [JSInterface.java](../../src/com/lushprojects/circuitjs1/client/JSInterface.java) 的 setupJSInterface 与回调，[CircuitElm.java](../../src/com/lushprojects/circuitjs1/client/CircuitElm.java) 的 addJSMethods，以及 [WebSocket 示例](../../websocket/README.md)。稳定 ID、版本协议和事务控制属于新设计。

## 最小接口

```ts
interface StationEditorHost {
  exportDocument(): Promise<unknown>;
  importDocument(data: unknown): Promise<ImportResult>;
  execute(commandId: string, payload: unknown): Promise<CommandResult>;
  selectElements(ids: string[]): void;
}
```

输入按结构验证，返回副本或只读数据，不暴露可变的文档引用。初始化完成后再接收命令，实例销毁后取消订阅和未完成请求。

## 事件与版本

事件包含实例 ID、documentId、revision、requestId 和类型。导出、保存等异步操作用 Promise 或明确结果事件表示完成；回调抛错不影响文档内部事务。

跨 iframe 可用 postMessage，核对来源窗口与明确的 origin，验证消息结构和允许的命令。这里只描述设计约定，当前仓库没有这套站场协议。

## 实时数据与协作

外部设备状态进入 runtime，不绕过校验修改设计。第一版多窗口采用版本检查与冲突提示即可；若要多人协作，应单独定义并发模型、重放与冲突处理，WebSocket 连接本身不解决冲突。

## 验收

拒绝未知命令和无效 ID；只读状态无法经外部入口编辑；重复请求不会重复创建设备；销毁后无回调写入；外部状态更新不污染工程文件历史。
