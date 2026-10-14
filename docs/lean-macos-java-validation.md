# Lean macOS Java 验收记录

## 构建环境

- 目标系统：macOS Apple Silicon。
- 构建目标：`//build:lean_java_i_build_target`。
- SDK 来源：本机 Xcode SDK archive。
- SDK 地址：`http://127.0.0.1:8765/MacOSX.sdk.with-private.tar.gz`。
- 下载配置：`/tmp/lean-bazel-downloader.cfg`。

## 构建结果

构建命令：

```bash
./bazel.cmd run //build:lean_java_i_build_target \
  -- --target-os=mac \
  --downloader_config=/tmp/lean-bazel-downloader.cfg
```

macOS DMG 已生成：

```text
out/idea-lean/artifacts/ideaIC-263.SNAPSHOT-aarch64.dmg
```

当前 SDK 地址只适用于本机验证。构建前必须启动本地 HTTP 服务：

```bash
python3 -m http.server 8765 --directory /tmp/lean-macos-sdk
```

## 已验证功能

- Java Run 和 Debug。
- Maven 导入、运行和日志控制台。
- Spring Boot 主类启动。
- `mvn spring-boot:run`。
- 内嵌 Tomcat 日志。
- Java Debug。
- JUnit。
- Git。
- Git 状态、提交、日志和分支。
- macOS 安装包启动。
- 插件市场和自动更新入口。
- Gradle 不在产品范围内。

## 当前基线

- Apple Silicon DMG：约 343 MB（359,956,893 bytes）。
- 发行目录插件数量：26。
- Gradle 插件已完全移除。
- 本轮移除 Eclipse、Shell、TestNG、Grid、HTML 工具和 Java 国际化插件。
- Java 字节码查看器保留。
- Lombok 插件使用仓库内置的当前版本。
- 统计上报和反馈入口：通过 Lean VM 参数关闭。
- 统计采集：通过 `idea.disable.collect.statistics=true` 关闭。
- IDE 更新检查：通过 `ide.no.platform.update=true` 关闭。
- OTLP、Jaeger 和 OpenTelemetry 导出：通过启动参数禁用。Telemetry API 和核心实现仍保留。
- Windows 安装包、ZIP 和签名步骤：跳过。
- 当前没有可用的同版本未精简 DMG，因此未计算体积和启动时间差值。

## 已移除的插件

- Terminal。
- JCEF。
- WebP。
- Subversion。
- Mercurial。
- Gradle。
- Groovy。
- Kotlin。
- Markdown 和 Mermaid。
- GitHub 和 GitLab 集成。
- Grazie、MCP 和功能培训。

## 保留的底层模块

以下模块仍由核心平台依赖保留：

- `intellij.platform.feedback`。
- `intellij.platform.statistics`。
- `intellij.platform.statistics.config`。
- `intellij.platform.statistics.uploader`。
- WSL 和 WinP 相关运行时模块。

这些模块没有作为用户插件提供。直接删除它们会影响 Git、Gradle、执行框架或平台接口。

## 当前限制

- 本机 SDK archive 没有提交到仓库。
- `MODULE.bazel` 使用本机回环地址。
- 其他机器需要准备相同的 archive 和本地 HTTP 服务。
- 当前没有执行 Git commit 或 push。
- RD、Remote 和协作 API 仍保留，因为核心执行框架、Git 和平台服务直接依赖它们。
- `platform-images` 中的 JCEF 可选 JAR 仍会打包，但没有 JCEF 插件入口，也不会加载 JCEF 功能。完全删除该 JAR 需要建立 Images 专用模块变体。

- JSP、Java EE/Jakarta EE、外置 Tomcat、应用服务器和 Java Remote Servers 不在产品范围内。
- Spring Boot 使用的内嵌 Tomcat 必须继续工作。
