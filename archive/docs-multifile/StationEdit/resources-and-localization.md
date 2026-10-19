# StationEdit：资源加载、示例与国际化

本文是建议设计，尚未在仓库中实现。参考源及适用范围见下文。

[主题导航](../README.md) · [对照文档](../resources-and-localization.md)

## 参考源与适用范围

参考 [circuitjs1.java](../../src/com/lushprojects/circuitjs1/client/circuitjs1.java) 的语言加载、[util/Locale.java](../../src/com/lushprojects/circuitjs1/client/util/Locale.java) 的映射、[Menus.java](../../src/com/lushprojects/circuitjs1/client/Menus.java) 的示例索引与 [QueryParameters.java](../../src/com/lushprojects/circuitjs1/client/QueryParameters.java)。以下显式启动流程、请求竞态处理与翻译 key 为新增设计。

## 启动流程

加载用户设置 → 加载必要字体和语言 → 初始化编辑器 → 打开指定文档或空白文档。语言和可选示例失败不阻止空白编辑，必要文件失败给出可重试状态。

使用 loading/ready/error 状态和 requestId。连续选择两个示例时，仅最后一次请求可替换文档；正在编辑的文档不被延迟默认示例覆盖。

## 国际化与单位

采用稳定消息 key，参数与翻译文本分开。设备 ID、端口 ID、文件字段名和枚举值不翻译；设备自定义名称原样保留。数值存储使用统一单位与格式，界面按语言格式化。

用户文字以文本方式显示，不能借用可信 HTML 翻译接口插入任意标记。切换语言后更新布局和文字测量缓存。

## 示例与帮助

示例清单记录 ID、标题 key、文件路径、格式版本和说明。示例经过相同解析校验流程，不是绕过规则的特殊入口；打开前处理未保存修改。

## 验收

缺少翻译显示可读回退；语言失败仍可编辑；示例晚返回不覆盖新文档；中文设备名往返保存无损；改变语言不改变业务 ID 和数值。
