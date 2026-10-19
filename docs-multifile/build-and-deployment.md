# CircuitJS：构建、资源打包与部署

本文描述当前仓库源码中的实现，示例与限制以所列源码为准。

[主题导航](README.md) · [对照文档](StationEdit/build-and-deployment.md)

## 配置入口

[build.gradle](../build.gradle)、[Gradle wrapper](../gradle/wrapper/gradle-wrapper.properties)、[GWT 模块](../src/com/lushprojects/circuitjs1/circuitjs1.gwt.xml)、[构建工作流](../.github/workflows/build.yml)、[Pages 工作流](../.github/workflows/deploy-pages.yml)。

## 构建链路

```text
Java 源码与 GWT 模块 → compileGwt → build/gwt/out/circuitjs1
war 静态资源 + GWT 新产物 → makeSite → site/
```

当前配置依赖 GWT 2.8.2、Java 8 源码级别，wrapper 固定 Gradle 8.7。这里记录仓库配置，不表示这些是新项目应选择的最新版本。

makeSite 依赖 compileGwt，复制 war 中列出的资源类型，同时排除旧 circuitjs1 输出目录，避免旧文件覆盖新编译产物。

## 本地预览

```bash
./gradlew compileGwt makeSite
python3 -m http.server 8000 --directory site
```

随后访问 circuitjs.html。也可使用 test.sh 一键构建预览。首次构建需要依赖下载；端口、JDK 与网络失败应分别判断。

## 持续集成与部署

build.yml 在指定分支的推送和 PR 上构建；deploy-pages.yml 在 dev 推送时构建站点、上传 Pages artifact 并部署。配置支持不等于每次部署都已经成功，实际状态需看对应运行记录。

## 容易遗漏的资源

语言、示例、字体、图标、canvas2svg 和离线脚本均需要核对最终产物。makeSite 的静态 include 列表不包含所有 JS/JSON 文件；不能因为 war 中存在 manifest 或 service worker 就断言 site 已具备完整离线资源。
