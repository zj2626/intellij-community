已按要求先做 macOS 安装包基线构建，结果如下：

- 已补齐仓库 README 要求的 Android 依赖模块。
- 构建命令：`./installers.cmd -Dintellij.build.target.os=current`
- Bazel 已完成大部分目标分析（约 2,567 个包、9,192 个配置目标）。
- 最终失败在下载 macOS SDK：JetBrains 缓存地址返回 `401 Unauthorized`，缺少 `CLTools_macOSNMOS_SDK.pkg`，因此当前环境没有产出可安装 DMG/ZIP。
- 这属于构建环境/SDK 下载权限问题，不是源码编译错误。后续需要在已安装 Xcode Command Line Tools 或可访问该 SDK 缓存的环境重试。当前源码可进入完整构建分析阶段，但尚未完成可安装包证明。

## 功能清单、作用和影响范围

| 功能 | 作用 | 主要范围 | 对精简版建议 |
|---|---|---|---|
| 平台启动与产品配置 | 启动 IDE、加载模块/插件、用户配置、诊断 | `platform/boot`、`bootstrap`、`starter`、`build` | 保留，属于基础闭包 |
| 项目/模块模型 | 打开项目、模块、SDK、工作区和持久化 | `platform/projectModel`、`workspace`、`configuration-store` | 保留 |
| 编辑器 | 文本编辑、缓冲、折叠、导航、编辑器 UI | `platform/editor`、`editor-ui-*`、`lang-*` | 保留 |
| Java 解析与 PSI | Java 语法树、语义模型、代码结构 | `java/java-psi-*`、`java/java-impl` | 保留，核心目标 |
| Java 索引与分析 | 建索引、查找符号、检查、补全、导航 | `java/java-indexing-*`、`java/java-analysis-*`、`platform/indexing-*` | 保留，核心目标 |
| Java 编译与运行 | Javac、运行配置、类路径、构建输出 | `java/compiler`、`java/execution`、`java/java-runtime` | 保留，核心目标 |
| Maven | POM 编辑/补全、依赖解析、项目导入、Maven 生命周期 | `plugins/maven`、Maven server modules | 保留，核心目标 |
| Gradle | Gradle 导入、同步、任务和 Tooling API | `plugins/gradle`、`plugins/gradle-maven` | 第一版移出默认包，后续按需恢复 |
| Git | 提交、分支、差异、历史、冲突解决 | `plugins/git4idea` | 保留，核心目标 |
| GitHub/GitLab/SVN/Hg | 远程仓库、平台集成和其他 VCS | `plugins/github`、`gitlab`、`svn4idea`、`hg4idea` | 默认移除；Git 基础不受影响 |
| Spring 支持 | Spring 专用配置、Bean 导航、注入、Spring Boot 智能分析 | 主要属于 JetBrains Ultimate 的 Spring 插件能力 | 社区源码不能直接保证完整 Spring 支持；先保留 Java + Maven + XML/Properties/YAML + 注解分析，再单独评估 Spring 插件许可/来源 |
| 搜索 | 全局搜索、文件搜索、符号/类/文本查找、Search Everywhere | `platform/find`、`platform/search`、索引模块 | 保留，核心目标；移除 ML 排名以降内存 |
| Terminal | IDE 内置终端和 Shell 支持 | `plugins/terminal`、`java/terminal` | 建议保留为轻量实用功能 |
| 调试器 | Java 调试、断点、变量和调用栈 | `java/debugger`、`platform/xdebugger-*` | 建议保留为可选或第二阶段默认功能 |
| JUnit/TestNG | 测试运行、测试树、测试配置 | `plugins/junit`、`testng`、`platform/testRunner` | 保留 JUnit；TestNG 后置为可选 |
| Groovy/Kotlin | 语言支持及其 Gradle/脚本集成 | `plugins/groovy`、`plugins/kotlin` | 默认移除；会明显减少插件和索引负担 |
| XML/JSON/YAML/Properties | 配置文件编辑、校验和补全 | `xml`、`json`、`plugins/yaml`、`plugins/properties` | 保留；Spring/Maven 项目实际需要 |
| Markdown/Mermaid | 文档和图表编辑 | `plugins/markdown`、`mermaid` | 默认移除，可按需安装 |
| Android/Compose/JavaFX | Android、Compose、JavaFX 开发 | `android`、相关插件和外部模块 | 移除，避免大依赖闭包 |
| Notebook/Jupyter/Python | Python、Notebook、数据科学能力 | `python`、`jupyter`、`notebooks` | 移除 |
| AI/ML/Grazie | 模型、智能排名、语法/写作辅助 | `completion-ml-*`、`search-everywhere-ml`、`grazie` 等 | 移除，优先降低常驻内存 |
| 远程开发/协作 | Gateway、远程后端、协作会话 | `remote-control`、`fleet`、`collaboration-tools` 等 | 移除 |
| 插件市场/设置同步 | 安装插件、同步配置 | 平台插件管理、`settings-sync` | 保留基础插件管理；设置同步可后置 |
| Maven Helper | Maven 依赖树、冲突分析等增强操作 | 当前社区仓库未发现 `maven-helper` 源码模块或插件 ID | 不建议第一阶段内置；应先确认插件许可证、兼容版本和是否能从 Marketplace 离线/在线安装 |

## 针对你的目标的推荐保留集合

第一版建议只做以下默认闭包：

1. 平台核心、项目模型、编辑器、基础设置和插件管理。
2. Java 解析、索引、检查、补全、重构、编译和运行。
3. Maven（含 Maven Server、POM/XML/Properties/YAML 支持）。
4. Git 基础能力。
5. 全文/文件/符号/类搜索；关闭 Search Everywhere ML 排名。
6. JUnit、基础调试器和 Terminal。
7. Spring 项目的可用基础：Java + Maven + XML + Properties + YAML + 注解；完整 Spring 专用导航需要另行确认 Ultimate 插件授权或合法兼容插件。

第一版明确移除：Kotlin、Groovy、Gradle、Android、Compose、JavaFX、Python、Notebook、AI/ML、Grazie、远程开发、GitHub/GitLab、SVN/Hg、Markdown/Mermaid、额外 Keymap 和性能测试插件。

## 分阶段工作

### 阶段 0：基线和构建环境

- 在可访问 macOS SDK 的机器安装 Xcode Command Line Tools，重试完整 installers 构建。
- 记录原版启动时间、空项目 RSS、Java 中型项目首次索引峰值、稳定 RSS、编译峰值。
- 保存原版安装包和校验信息，作为裁剪前基线。

### 阶段 1：Lite 产品变体

- 新增独立 `LiteIdeaProperties` 和 `//build:idea_community_lite`，不修改原 `idea_community`。
- 用产品布局插件白名单构建：平台核心、Java、Maven、Git、搜索、XML/JSON/YAML/Properties、JUnit、Terminal。
- 新产品使用独立产品 ID、配置目录和输出目录，保证可回滚。

### 阶段 2：依赖闭包和功能回归

- 通过 Bazel/JPS 依赖图验证每个保留功能的运行时闭包。
- 验证 Maven 导入、依赖下载、POM 编辑、Java 索引/补全/检查/重构、编译运行、Git 操作、搜索、JUnit 和调试。
- 验证 Spring Boot Maven 项目：项目导入、依赖解析、Java/XML/Properties/YAML 编辑；把完整 Spring 专用能力列为单独验收项。

### 阶段 3：内存优化

- Lite 初始使用 `-Xms128m/-Xmx1024m`，保留用户可覆盖配置；大型项目用实测决定是否提高到 1536m。
- 移除 ML/AI 模型和重量级插件的默认加载。
- 检查服务是否按需初始化，比较索引缓存、插件加载和后台任务的 RSS。

### 阶段 4：Maven Helper 评估

- 确认 Maven Helper 的正式插件坐标、许可证、IDE 版本兼容性和是否允许随发行版再分发。
- 若不能合法内置，则提供安装说明或做一个只覆盖依赖树/冲突分析的轻量替代插件。
- 只有通过兼容性和许可验证后，才加入 Lite 默认集合。

### 阶段 5：安装包和发布验收

- 在具备 SDK 的 macOS 环境构建可安装 DMG/ZIP。
- 安装后验证启动、升级、配置隔离、插件安装和卸载。
- 对比基线报告：安装包体积、启动时间、空闲 RSS、索引峰值、编译峰值和功能通过率。
