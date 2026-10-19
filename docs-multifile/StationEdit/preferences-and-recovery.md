# StationEdit：用户设置、自动恢复与保存状态

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../preferences-and-recovery.md)

## 参考源与适用范围

借鉴 [UndoManager.java](../../src/com/lushprojects/circuitjs1/client/UndoManager.java) 的本地恢复、[UIManager.java](../../src/com/lushprojects/circuitjs1/client/UIManager.java) 的偏好与快捷键保存。按文档隔离、revision、写入确认和失败提示为新增设计。

## 分开持久化

- 用户设置：语言、网格显示、快捷键，带设置版本。
- 工程文件：明确保存的设计内容。
- 恢复草稿：文档 ID、时间、revision、应用版本和已提交快照。
- 撤销历史：当前编辑会话，第一版不要求跨重启保留。

小型偏好可用 localStorage；较大恢复数据使用 IndexedDB 等异步存储。不要预先把所有存储都统一成复杂抽象，只定义当前确有两种实现的文件能力接口。

## 保存与恢复流程

提交后更新 revision 并防抖写草稿，草稿只包含已提交内容。显式保存捕获目标 revision；写入成功后更新 savedRevision。如果写入期间又编辑，当前文档仍显示未保存。

打开时比较恢复草稿与已保存版本，让用户明确选择恢复或保留文件。多窗口以 sessionId 和 revision 检测冲突，不用一个全局 key 相互覆盖。

## 失败处理

配额不足、写入失败和不支持本地存储时保留内存文档并提示导出，不谎报已保存。退出时的异步保存不能作为唯一保障，应持续保留最近的已提交草稿。

## 验收

保存被取消不清除脏标记；保存旧 revision 完成后新修改仍未保存；恢复没有半截拖动预览；两窗口不静默覆盖；清理缓存与删除工程文件分别处理。
