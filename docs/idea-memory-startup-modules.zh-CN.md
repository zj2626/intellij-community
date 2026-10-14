# 内存与启动相关模块

本文列出影响 IntelliJ IDEA 内存占用和启动速度的模块。它区分构建时的模块集、运行时插件和外部进程。

## 阅读方式

IntelliJ IDEA 由模块、模块集和插件组成。

- **模块** 是编译和打包单元。
- **模块集** 打包一组相关模块。
- **插件** 是运行时单元，并拥有自己的类加载器。
- **外部进程** 运行在 IDEA 主 JVM 之外。

模块名会随版本变化。删除模块前，请先检查产品依赖闭包。

## 影响等级

| 等级 | 含义 |
| --- | --- |
| 高 | 对启动或常驻内存影响明显。 |
| 中 | 在常见流程或后台任务中出现影响。 |
| 低 | 只有功能启动后才出现影响。 |

## 核心平台

| 模块或模块集 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| `intellij.platform.core` | 基础服务、对象生命周期、扩展点和 Disposable。 | 高。产品启动时加载。 | 保留。 |
| `intellij.platform.core.impl` | 核心服务实现和扩展点。 | 高。产品启动时加载。 | 保留。 |
| `intellij.platform.util.*` | UI、文本、协程、进度、存储、序列化等工具。 | 高。很多子系统使用。 | 保留。 |
| `intellij.platform.concurrency` | 线程池、协程和后台任务。 | 高。启动服务使用。 | 保留。 |
| `intellij.platform.projectModel` | 项目、模块、根目录、Facet、库和内容根。 | 高。项目打开和索引用到。 | 保留。 |
| `intellij.platform.projectModel.impl` | 项目模型实现和根目录检测。 | 高。导入和索引用到。 | 保留。 |
| `intellij.platform.workspace.*` | 工作区模型和 JPS 模型存储。 | 高。大项目增加内存。 | 保留。 |
| `intellij.platform.indexing` | 索引基础设施和文件索引 API。 | 高。CPU 和内存取决于索引规模。 | 保留。 |
| `intellij.platform.indexing.impl` | 索引实现和后台索引。 | 高。大索引占用内存和磁盘。 | 保留。 |
| `intellij.platform.analysis.*` | 基础分析和检查服务。 | 中到高。代码分析使用。 | 保留。 |
| `intellij.platform.syntax.*` | 新语法树基础设施。 | 中。语言支持使用。 | Java/XML 需要时保留。 |
| `intellij.platform.settings` | 设置模型和持久化。 | 中。启动时读取用户设置。 | 保留。 |
| `intellij.platform.pluginSystem.parser.impl` | 解析插件描述文件。 | 中。启动时读取插件。 | 保留。 |
| `intellij.platform.icons.*` | 图标 API 和渲染。 | 中。UI 启动时加载图标。 | 保留。 |
| `intellij.platform.diagnostic.telemetry` | Telemetry API 和 OpenTelemetry 类型。 | 中。核心平台启动时加载。 | 必要时保留。 |
| `intellij.platform.diagnostic.telemetry.impl` | Telemetry 导出实现。 | 中。可能发送后台数据。 | 本地构建可移除。 |
| `intellij.platform.diagnostic.telemetry.exporters` | OTLP 和 Jaeger 导出器。 | 中。增加库和后台任务。 | 本地构建可移除。 |
| `intellij.platform.runtime.product` | 运行时模块仓库和产品模型。 | 中。启动时解析模块。 | 保留。 |
| `intellij.platform.ide.core` | IDE 核心模型和应用服务。 | 高。产品启动时加载。 | 保留。 |

## 基础 IDE 模块

| 模块或模块集 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| `intellij.platform.ide` | 应用和项目 UI 外壳、动作、工具窗口、菜单。 | 高。产品启动时加载。 | 保留。 |
| `intellij.platform.ide.impl` | IDE 服务实现。 | 高。产品启动时加载。 | 保留。 |
| `intellij.platform.lang.impl` | 语言相关的 IDE 服务。 | 高。编辑器和导航使用。 | 保留。 |
| `intellij.platform.editor` | 编辑器生命周期和文档编辑。 | 高。打开项目后启动。 | 保留。 |
| `intellij.platform.editor.ui` | Swing 编辑器 UI。 | 高。打开文件时启动。 | 保留。 |
| `intellij.platform.editor.backend` | 编辑器后端服务。 | 中。支持编辑器。 | 保留。 |
| `intellij.platform.editor.frontend` | 分离式产品的前端编辑器。 | 中。只有分离 UI 需要。 | 单体产品可保留。 |
| `intellij.platform.searchEverywhere` | Search Everywhere 模型和 contributor。 | 中。弹窗启动是懒加载。 | 保留。 |
| `intellij.platform.searchEverywhere.backend` | 搜索后端和 provider。 | 中。搜索可能后台运行。 | 保留。 |
| `intellij.platform.searchEverywhere.frontend` | Search Everywhere UI。 | 中。弹窗启动是懒加载。 | 保留。 |
| `intellij.platform.find` | Find in Files 和用法搜索基础设施。 | 高。文本和用法索引用到。 | 保留。 |
| `intellij.platform.find.backend` | 查找任务执行。 | 中。大项目使用后台线程。 | 保留。 |
| `intellij.platform.completion.*` | 补全引擎和 UI。 | 中到高。索引后运行。 | 保留。 |
| `intellij.platform.inline.completion` | 内联补全服务。 | 中。可能后台运行。 | 需要时保留。 |
| `intellij.platform.execution.impl.*` | Run Configuration 和进程执行。 | 中。运行动作和控制台使用。 | 保留。 |
| `intellij.platform.externalSystem.impl` | 外部构建系统集成基础。 | 高。Maven 导入使用。 | 保留。 |
| `intellij.platform.externalSystem.dependencyUpdater` | 外部系统依赖更新。 | 中。可能启动后台任务。 | 与 Maven 一起保留。 |
| `intellij.platform.pluginManager.*` | 插件发现和安装。 | 中。启动时读取插件。 | 保留。 |
| `intellij.platform.ide.updateChecker` | IDE 更新检查。 | 低。通常后台运行。 | 受控构建可移除。 |
| `intellij.platform.credentialStore.*` | 密码和凭据存储。 | 低。服务启动时加载。 | Git 或插件需要时保留。 |
| `intellij.platform.builtInServer.impl` | 内置 HTTP 服务。 | 低。打开本地服务。 | 需要时保留。 |
| `intellij.platform.managed.cache.*` | 托管缓存。 | 中。大项目占用内存。 | 保留。 |
| `intellij.platform.progress.backend` | 后台进度和任务。 | 中。启动和导入使用。 | 保留。 |

## 语言与编辑模块集

| 模块或模块集 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| `corePlatform()` | 没有 IDE 和语言 UI 的平台基础。 | 高。是基础。 | 保留。 |
| `coreIde()` | 平台加基础 IDE UI。 | 高。多数 IDE 产品需要。 | 保留。 |
| `coreLang()` | 平台加语言支持。 | 高。编辑器和导航需要。 | 保留。 |
| `essentialMinimal()` | 编辑器、搜索、补全、前后端分离基础。 | 高。打开项目后启动。 | 保留。 |
| `essential()` | 查找、补全、执行、插件管理、外部系统。 | 高。多数 IDE 产品需要。 | 保留。 |
| `intellij.platform.xml.*` | XML 解析、PSI、DOM、分析和 UI。 | 中到高。POM 和配置文件使用。 | 保留。 |
| `intellij.relaxng` | RELAX NG 支持。 | 低。XML 使用时启动。 | 不需要时移除。 |
| `intellij.xml.emmet.*` | 标记语言的 Emmet 缩写。 | 低。HTML/XML 编辑时启动。 | Java-only 可移除。 |
| `intellij.jsp.base` | 共享 JSP API。 | 低。JSP 文件使用时启动。 | 不需要 JSP 时移除。 |
| `lsp()` | Language Server Protocol 支持。 | 中。可能启动语言服务。 | 不用 LSP 时移除。 |
| `intellij.platform.duplicates.analysis` | 重复代码分析。 | 低。调用时启动。 | 精简版可移除。 |
| `platformTestFrameworksCore()` | IDE 测试框架模块。 | 测试时高。IDE 运行时不需要。 | 不打进产品。 |
| `platformTestFrameworksJunit5()` | JUnit 5 IDE 测试夹具。 | 测试时高。IDE 运行时不需要。 | 不打进产品。 |

## 调试与版本控制

| 模块或模块集 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| `intellij.platform.debugger` | 调试器 API。 | 中。调试会话启动。 | Java 开发保留。 |
| `intellij.platform.debugger.impl` | 调试器实现。 | 中到高。调试 UI 和引擎使用。 | 保留。 |
| `intellij.platform.debugger.impl.ui` | 调试视图和控件。 | 中。调试 UI 启动。 | 保留。 |
| `intellij.platform.debugger.impl.backend` | 调试后端求值和进程控制。 | 中。调试会话启动。 | 保留。 |
| `intellij.platform.vcs` | 版本控制 API 和项目集成。 | 中。VCS 变更时启动。 | 保留。 |
| `intellij.platform.vcs.core` | 文件和变更模型支持。 | 中。Git 项目启动。 | 保留。 |
| `intellij.platform.vcs.impl` | VCS UI、变更列表、日志、Shelve。 | 中到高。后台刷新使用。 | 保留。 |
| `intellij.platform.vcs.dvcs.impl.shared` | 分布式版本控制支持。 | 中。Git 使用。 | 保留。 |
| `intellij.libraries.jgit` | 纯 Java Git 实现。 | 中。设置同步或特殊任务使用。 | 需要时保留。 |

## UI 与远程开发

| 模块或模块集 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| `compose()` | Compose、Skiko、Coil、Jewel 和 Markdown 桥。 | 高。增加大 JAR 和原生代码。 | 经典 UI 够用时可移除。 |
| `intellij.platform.compose.swing` | Compose 到 Swing 的集成。 | 高。Compose UI 启动。 | 没有 Compose UI 时移除。 |
| `intellij.platform.jewel.*` | 基于 Compose 的 IDE UI 组件。 | 高。Compose UI 启动。 | 没有 Compose UI 时移除。 |
| `rdCommon()` | 远程开发和分离 IDE 协议。 | 中到高。增加 RPC 模型。 | 不做远程 IDE 时移除。 |
| `intellij.rd.client.*` | 远程客户端服务。 | 中。远程开发使用。 | 不做远程 IDE 时移除。 |
| `intellij.platform.frontend` | 分离式 IDE 的前端外壳。 | 中。单体产品仍可能打包。 | 检查依赖闭包。 |
| `intellij.platform.monolith` | 单体 IDE 装配。 | 中。支持单进程运行。 | 保留。 |
| `intellij.platform.split.protocol` | 前后端分离协议。 | 中。远程开发使用。 | 不做远程 IDE 时移除。 |
| `intellij.platform.collaborationTools.*` | 协作和共享会话。 | 中。增加服务和认证。 | 可移除。 |
| `intellij.platform.settings.local` | 本地设置后端。 | 中。设置启动。 | 保留。 |
| `settingsSync()` | 设置同步和 JGit。 | 中。可能后台同步。 | 可移除。 |

## 捆绑插件

### Lean Java IDE 保留

| 插件 | 主要功能 | 内存与启动影响 |
| --- | --- | --- |
| `intellij.java.plugin` | Java PSI、检查、导航、重构、编译器集成、Java 运行和调试。 | 高。Java 项目核心。 |
| `intellij.java.ide.customization` | Java IDE 定制和呈现。 | 低到中。产品启动时加载。 |
| `intellij.maven.plugin` | Maven 导入、POM 模型、依赖解析、Maven 工具窗口、Maven 运行。 | 高。Maven 项目启动。 |
| `intellij.vcs.git` | Git 状态、diff、提交、分支、历史、拉取和推送。 | 中。Git 项目启动。 |
| `intellij.java.decompiler` | class 文件和依赖 JAR 反编译。 | 中。支持 JAR 导航。 |
| `intellij.junit` | JUnit 测试检测和运行调试。 | 中。Java 测试使用。 |
| `intellij.lombok.plugin` | Lombok 支持和生成成员可见性。 | 中。Lombok 项目使用。 |
| `intellij.properties` | Properties 文件编辑和资源引用。 | 低。Properties 文件启动。 |
| `intellij.editorconfig.plugin` | EditorConfig 支持。 | 低。配置文件启动。 |
| `intellij.json` | JSON 解析、编辑、Schema 和导航。 | 中。JSON 配置启动。 |
| `intellij.yaml` | YAML 解析、编辑和导航。 | 中。YAML 配置启动。 |
| `intellij.xml.plugin` | XML 编辑和校验。 | 中。XML 和 POM 文件使用。 |
| `intellij.platform.vcs.plugin` | VCS UI 和项目集成。 | 中。VCS 项目启动。 |
| `intellij.platform.searchEverywhere` | Search Everywhere。 | 中。第一次搜索时启动。 |

### 通常可移除

| 插件 | 主要功能 | 内存与启动影响 |
| --- | --- | --- |
| Kotlin 插件 | Kotlin 语言支持和编译器集成。 | 高。增加语言、守护进程和库。 |
| Gradle 插件 | Gradle 导入、同步、运行和工具。 | 高。启动守护进程和同步任务。 |
| Groovy 插件 | Groovy 支持和脚本。 | 高。增加语言和运行时。 |
| Android 插件 | Android 项目模型和工具。 | 高。增加大模型和构建支持。 |
| JavaFX 插件 | JavaFX 项目支持。 | 中。JavaFX 项目使用。 |
| Terminal 插件 | 内嵌终端和 PTY。 | 中。终端工具窗口启动。 |
| Markdown 插件 | Markdown 编辑和预览。 | 低到中。预览可能启动 JCEF。 |
| Mermaid 插件 | Mermaid 图渲染。 | 中。预览使用浏览器运行时。 |
| JCEF 插件 | 内嵌 Chromium 浏览器。 | 高。增加原生浏览器和 helper 进程。 |
| AI 和 ML 插件 | AI 辅助、ML 排序和智能功能。 | 中到高。增加模型和后台任务。 |
| Grazie | 写作和语法支持。 | 低到中。文本功能启动。 |
| Features trainer | 入门引导。 | 低。引导启动。 |
| GitHub/GitLab | 远程仓库和评审服务。 | 中。增加网络和 VCS 服务。 |
| Coverage | 覆盖率收集和展示。 | 中。运行插桩代理。 |
| TestNG | TestNG 测试支持。 | 低到中。TestNG 项目使用。 |
| Eclipse 集成 | Eclipse 项目导入。 | 低。Eclipse 项目使用。 |
| Shell 支持 | Shell 脚本编辑。 | 低。Shell 文件启动。 |
| TOML | TOML 编辑。 | 低。TOML 文件启动。 |
| TextMate | 通用 TextMate 语法。 | 低。匹配文件启动。 |
| WebP | WebP 图片支持。 | 低。图片启动。 |
| 其他 keymap | Eclipse、Visual Studio、NetBeans 键位。 | 低。增加启动描述。 |
| Compose IDE 插件 | Jetpack Compose 支持。 | 中。Compose 文件启动。 |
| Performance testing | 启动和性能测试。 | 低。普通用户不需要。 |
| MCP server 插件 | Model Context Protocol 服务。 | 中。增加网络服务。 |

## 外部进程和大型运行时

| 运行时 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| JetBrains Runtime | IDEA 的 Java 运行时。 | 高。决定 JVM 基线。 | 使用正确的 macOS 运行时。 |
| JCEF | 内嵌 Chromium 浏览器。 | 高。增加 helper 进程和原生内存。 | 不需要时移除。 |
| Maven server | 依赖解析和 Maven 模型导入。 | 导入时高。 | Maven 项目保留。 |
| Maven 仓库索引 | 全量构件搜索。 | 常驻时高。 | 只在需要时启用。 |
| Gradle daemon | Gradle 模型导入和任务。 | 高。独立 JVM。 | 移除 Gradle 支持。 |
| Kotlin daemon | Kotlin 编译和脚本支持。 | 高。独立 JVM。 | 移除 Kotlin 支持。 |
| Java compiler | 增量 Java 编译和 JPS 构建。 | 构建时高。 | 保留。 |
| Git executable | 本地 Git 操作。 | 中。VCS 刷新时运行。 | 保留。 |
| LSP server | 外部语言服务。 | 中。独立进程。 | 不需要时移除。 |
| AI 服务 | 远程或本地 AI 辅助。 | 中。网络或后台工作。 | 移除。 |

## 后台工作

| 工作 | 主要功能 | 内存与启动影响 | 精简建议 |
| --- | --- | --- | --- |
| 项目索引 | 构建符号、用法、文件和框架索引。 | CPU 和内存高。 | 保留；缩小范围。 |
| 框架检测 | 检测 Spring、Maven 等框架。 | CPU 中。 | 保留 Maven 和 Java。 |
| Maven 同步 | 导入 POM 和依赖。 | CPU、内存、I/O 高。 | 保留；按需运行。 |
| VCS 刷新 | 读取 Git 状态和变更。 | CPU 和 I/O 中。 | 保留。 |
| 插件更新扫描 | 读取并检查插件。 | 低到中。 | 关闭自动检查。 |
| Telemetry 导出 | 发送诊断数据。 | 低到中。 | 本地构建移除。 |
| 设置同步 | 上传设置。 | 低到中。 | 移除。 |
| 搜索索引 | 更新 Find in Files 数据。 | CPU 和内存中。 | 保留。 |
| Git 日志索引 | 构建提交历史视图。 | CPU 和内存中。 | 需要时保留。 |

## Lean 产品矩阵

| 目标 | 保留 | 移除 |
| --- | --- | --- |
| Java 编辑 | Java 插件、核心语言支持、编辑器、补全。 | Kotlin、Groovy、Android、其他语言。 |
| Maven 项目 | Maven 插件、外部系统支持、Java 编译器。 | Gradle、Kotlin daemon、其他构建系统。 |
| 服务启动 | Run Configuration 支持、Java 执行。 | 不需要的应用服务器和远程服务。 |
| 调试 | Java 调试器和 JUnit runner。 | 脚本调试器、远程调试器、不需要的 TestNG。 |
| JAR 搜索 | Java 索引、反编译器、Maven 依赖根。 | 默认关闭全量 Maven Central 索引。 |
| Git | VCS 核心、Git 插件、凭据存储。 | GitHub、GitLab、任务服务器。 |
| 搜索 | Search Everywhere、Find in Files、类索引。 | AI 搜索、多余 contributor、大仓库索引。 |
| macOS 包 | macOS 运行时和原生代码。 | Windows、Linux、跨平台产物。 |

## 测量清单

每次裁剪前后测量：

- `.app` 大小。
- 冷启动时间。
- 热启动时间。
- 主进程常驻内存。
- helper 进程常驻内存。
- 索引时间和索引磁盘大小。
- Maven 同步时间和 Maven server 内存。
- 首次编辑输入时间。
- Search Everywhere 首个结果时间。

每次只比较一项变化。硬件、项目、JVM 参数、缓存状态和操作系统必须一致。
