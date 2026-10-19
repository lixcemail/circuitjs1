# CircuitJS：桌面外壳与离线缓存

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/desktop-and-offline.md)

## 源码入口

[main.js](../app/main.js)、[preload.js](../app/preload.js)、[package.json](../app/package.json)、[页面入口](../war/circuitjs.html)、[service-worker.orig](../war/service-worker.orig)、[缓存清单生成脚本](../war/update-service-worker.sh)。

## Electron 的分工

主进程维护窗口、文件打开事件、单实例转交和原生对话框。preload 暴露 showSaveDialog、saveFile、openFile 等桥接，并通过 fs 读写文件。启动文件文本以窗口变量交给应用。

macOS 使用应用菜单的原生编辑角色保留文本输入快捷键；其他平台有相应窗口与参数处理。renderer.js 主要是占位，不是主要业务所在。

当前 BrowserWindow 设置 sandbox:false、contextIsolation:false，preload 直接往 window 挂函数。这里记录实际方式，不能当作新桌面应用必须复制的隔离设计。

## 离线缓存

页面注册 service worker，模板在 install 预缓存，在 fetch 优先命中缓存、成功请求后追加缓存，activate 清理其他版本。部分路径写为 /circuit/，部署基址影响命中。

update-service-worker.sh 根据编译目录生成 service-worker.new，并要求人工更新版本及替换。不能假定修改源码后缓存清单会自动更新。

## 限制

原生文件保存是异步操作，浏览器缓存与用户工程文件是不同数据。离线能力还依赖实际打包资源、服务环境和首次加载，详见构建文档。
