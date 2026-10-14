# 精简版 macOS Java IDE 计划

## 目标

构建一个面向 Java 和 Spring 项目的精简版 macOS IDE。

保留项目索引、Maven 导入、Java 编译、服务启动、调试、Git、项目管理、搜索和 JAR 搜索。

## 主要方案

先不要删除源代码目录。

创建新的产品配置。为所需功能建立依赖闭包。从产品中移除未使用的插件和平台模块。只有产品稳定后，才删除未使用的源代码目录。

这个方案可以减少合并冲突，也可以避免破坏共享平台模块。

## 产品范围

保留：

- IDE 核心平台。
- 编辑器、补全、检查、导航、运行和调试。
- 项目模型和索引。
- Search Everywhere、文件内查找、类搜索和库类搜索。
- Maven 和 Java 支持。
- Git 和项目管理。
- 仅保留 Git 版本控制，不内置 Subversion 和 Mercurial。
- 不内置 Gradle。
- XML、properties、JSON、YAML、JUnit 和 Lombok。
- 仅支持 macOS。

Spring 说明：

- 开源仓库不包含 Spring 专用插件。
- 不使用该插件也可以完成 Maven 导入、Java 编译和 Spring Boot 启动。
- Spring Bean 导航和 Spring 检查需要商业插件。开源构建不能分发该插件。

## 精简级别

### 级别 1：内置插件

从插件白名单开始。

保留：

- `JavaPluginLayout.MAIN_MODULE_NAME`
- `intellij.maven.plugin`
- `intellij.vcs.git`
- `intellij.junit`
- `intellij.lombok.plugin`
- `intellij.properties`
- `intellij.editorconfig.plugin`
- `intellij.json`
- `intellij.yaml`
- `intellij.xml.plugin`
- `intellij.platform.vcs.plugin`
- `intellij.platform.searchEverywhere`
- `intellij.platform.structureView.plugin`
- `intellij.platform.recentFiles.plugin`
- `intellij.platform.problemView.plugin`
- `intellij.platform.testRunner.plugin`

移除：

- Kotlin。
- Groovy。
- Gradle。
- Android。
- JavaFX。
- JCEF。
- Terminal。
- Markdown。
- Mermaid。
- TOML。
- TextMate。
- GitHub 和 GitLab。
- Subversion 和 Mercurial。
- Grazie。
- AI 和 ML 功能。
- Onboarding 和功能培训。
- 性能测试。
- Compose IDE 插件。
- 其他键位映射。
- Sh、Eclipse、WebP 和其他低频插件。

此级别的风险最低。

### 级别 2：平台模块

插件白名单稳定后，再减少平台内容。

- 使用 `essential()`，而不是更大的 `ideCommon()`。
- 显式添加调试器、VCS、Java、Maven、XML 和搜索模块。
- 移除 Compose。
- 移除拼写检查器。
- 移除设置同步。
- 移除协作工具。
- 在依赖闭包允许时，移除 RD 和 JetBrains Client 依赖。
- 保持 Java 插件的依赖闭包完整。

Compose 的 JAR 文件和 Skiko 原生运行时约占 80 MB。

### 级别 3：源代码树

只有精简版构建通过冒烟测试后，才删除源代码目录。

使用产品依赖报告识别未使用模块。不要只按目录名称删除目录。

## 代码修改

创建 `LeanJavaIdeaProperties`，使其继承 `IdeaCommunityProperties`。

设置：

- 内置插件白名单。
- 精简版产品内容描述符。
- `buildCrossPlatformDistribution = false`。
- `buildSourcesArchive = false`。
- `buildAllCompatiblePlugins = false`。
- macOS 分发定制器。

当前社区插件列表从 `build/src/org/jetbrains/intellij/build/IdeaCommunityProperties.kt:62` 开始。

当前默认内置插件列表从 `platform/build-scripts/src/org/jetbrains/intellij/build/BaseIdeaProperties.kt:14` 开始。

修改产品布局后，执行：

```bash
./bazel run //platform/buildScripts:plugin-model-tool
```

## macOS 构建

将 `targetOs` 设置为 macOS。在 Apple 硬件上，将 `targetArch` 设置为当前架构。

跳过：

- Windows 安装包。
- Linux 安装包。
- 跨平台分发包。
- 源代码压缩包。
- 本地构建的 macOS 签名和公证。
- 非内置插件发布。

使用：

```bash
-Dintellij.build.target.os=mac
```

当前安装包构建目标是 `build/src/OpenSourceCommunityInstallersBuildTarget.kt`。

## 启动和内存控制

移除：

- 内置浏览器。
- AI 和 ML 服务。
- 协作服务。
- 设置同步。
- Onboarding。
- 插件自动更新提示。

索引控制：

- 保留 Java 和项目索引。
- 保留 Maven 依赖 JAR 索引。
- 仅在需要时开启完整 Maven 仓库索引。
- 排除 `target/`、`build/`、生成目录和大型本地资源。

内存调优：

先测量冷启动、热启动、常驻内存、索引时间和应用体积。然后再调整：

- `-Xmx`。
- `-XX:MaxMetaspaceSize`。
- `-XX:ReservedCodeCacheSize`。

收集基线前，不要降低内存配置。

## 验证

构建：

- 产品布局验证通过。
- macOS 安装包构建成功。

Java 和 Maven：

- 打开 Maven 项目。
- Java 解析、补全、导航、重构和调试正常。
- Maven 导入、依赖下载、运行、调试和 Maven 工具正常。

Spring：

- Spring Boot 主类可以启动。
- 调试器可以在断点处停止。
- Spring 专用导航仅在安装商业 Spring 插件后可用。

JAR 搜索：

- `Go to Class` 可以查找依赖 JAR 中的类。
- `Find Usages` 可以处理反编译的库类。

Git：

- 状态、差异、提交、日志和分支功能正常。

性能：

- 记录应用体积。
- 记录冷启动时间。
- 记录热启动时间。
- 记录常驻内存。
- 记录索引时间。
- 将所有数据与未精简构建进行比较。

## 首次交付建议

先交付级别 1：

1. 仅支持 macOS 的构建。
2. 插件白名单。
3. 移除 JCEF、AI、ML、多种语言和多种构建系统。
4. 性能基线和冒烟测试。

级别 1 构建稳定后，再进入级别 2。
