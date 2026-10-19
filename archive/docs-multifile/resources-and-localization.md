# CircuitJS：资源加载、示例与国际化

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/resources-and-localization.md)

## 源码入口

[circuitjs1.java](../src/com/lushprojects/circuitjs1/client/circuitjs1.java) 的 loadLocale/processLocale/loadSimulator，[util/Locale.java](../src/com/lushprojects/circuitjs1/client/util/Locale.java) 的 LS/LSHTML，[Menus.java](../src/com/lushprojects/circuitjs1/client/Menus.java) 的 getSetupList/processSetupList/readSetupFile，[QueryParameters.java](../src/com/lushprojects/circuitjs1/client/QueryParameters.java)。

## 启动语言

语言优先从 lang URL 参数读取，再尝试本地 language 设置和浏览器语言。英文直接使用空映射；其他语言请求 locale 文件，解析后创建应用。HTTP 非成功响应可回落为空映射；网络 onError 等路径应逐一检查，不能把所有失败都描述成已实现自动回退。

Locale.LS 查找字符串，缺失时返回原文，并处理用于区分翻译语境的末尾标记。LSHTML 使用可信 HTML 形式输出翻译，适用于受控资源，不应套用于任意用户输入。

## 示例资源

Menus 从模块目录加载 setuplist.txt，解析分类和电路条目，再通过加载器读入选定电路。资源与应用编译产物一起部署；文件名、路径和模块基址需要匹配。

## 配置与异步

QueryParameters 解析 URL 中的启动选项，CirSim 决定启动文件、颜色、只读或运行状态等。现有解析器较简单，不能当作对任意畸形查询参数都健壮的通用解析器。

加载器通过 GWT RequestBuilder 异步读取文本。界面启动、默认示例和外部指定文件之间存在顺序关系，阅读时要检查回调触发哪一步。
