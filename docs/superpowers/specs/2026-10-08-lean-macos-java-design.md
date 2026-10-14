# Lean macOS Java 产品设计

## 目标

创建一个只面向 macOS 的精简版 Java IDE 产品。产品保留 Maven、Java 和 Spring Boot 内嵌服务器的核心开发流程，并移除 IDE Terminal、Gradle 和外置应用服务器支持。

## 范围

### 保留功能

- Java 核心插件。
- Maven 导入、依赖下载、工具窗口、Run 和 Debug。
- Java 项目 Run 和 Debug。
- Run/Debug Console，包括标准输出和错误输出。
- Git 状态、差异、提交、日志和分支。
- 项目模型、索引、搜索、JUnit、Lombok、XML、JSON、YAML 和 properties。
- JAR 中的类搜索和反编译库类的 Find Usages。
- macOS 安装包。
- 本地 IDE 设置保存和首次启动设置导入。
- 本地插件管理和在线插件市场入口。
- IDE 和插件自动更新检查。

### 不在范围内

- IDE Terminal 工具窗口。
- 设置同步、设置同步历史和云端设置恢复。
- Code With Me 及其他实时协作功能。
- 使用统计上传、反馈和用户调查。
- GitHub、GitLab 账号集成及 Pull/Merge Request 工具。
- Subversion 和 Mercurial 版本控制插件。
- Relax NG、XML Emmet、JSP、Java EE/Jakarta EE Web、Tomcat 外置服务器和 Java Remote Servers。
- Grazie、AI/ML 和 MCP Server 在线智能功能。
- 在 IDE Terminal 中执行 Shell、Maven、Gradle 或服务命令。
- Gradle 项目导入、同步、依赖解析、Gradle 工具窗口、Gradle 任务运行和 Gradle 专用 Run/Debug。
- Kotlin、Groovy、Android 和 JavaFX。
- JCEF、Markdown、Mermaid、TOML 和 TextMate。
- GitHub、GitLab、Grazie、AI/ML、Onboarding、功能培训和性能测试插件。
- 非 macOS 安装包。
- 删除共享平台源代码目录。

用户通过 macOS 外部 Terminal 执行命令行操作。
用户使用本地设置保存和首次启动设置导入。产品不提供云端设置同步。

## 架构方案

新增独立的 `LeanJavaIdeaProperties`，继承 `IdeaCommunityProperties`。新配置只影响 Lean 产品，不修改 IntelliJ IDEA Community 的默认发行配置。

产品配置包含：

- macOS 目标系统。
- 内置插件白名单。
- `buildCrossPlatformDistribution = false`。
- `buildSourcesArchive = false`。
- `buildAllCompatiblePlugins = false`。
- macOS 分发定制器。

实现先调整产品插件白名单，不删除源代码目录。产品依赖报告稳定后，再判断是否可以减少平台模块。

## 插件和模块策略

### Maven、Java 和运行框架

保留 Java、Maven、项目模型、索引、执行框架、调试器、测试运行器和 Run/Debug Console 所需的模块。

Run/Debug Console 必须继续显示 Java 和 Maven 进程的标准输出与错误输出。移除 Terminal 不得影响 IDE Run 或 Debug。

### Gradle

Lean 产品不包含 Gradle 插件。Gradle 项目导入、同步、依赖解析、工具窗口、任务运行和 Gradle 专用 Run/Debug 都不在产品范围内。

Spring Boot 通过 Maven 或 Java 主类启动。Spring Boot 使用的内嵌 Tomcat 日志和调试功能必须保留。

### Terminal

从产品插件白名单移除 `intellij.terminal`，但不删除 Terminal 源代码目录。

先通过依赖报告识别运行控制台仍需的底层模块，再保留最小依赖集合。产品应移除 Terminal 工具窗口、Shell 会话、PTY、JediTerm 和 Shell 补全模块。如果某个底层 Terminal API 被 Run/Debug Console 使用，则只保留该 API 所需的最小模块。

### 设置同步和协作工具

一级阶段直接从 Lean 产品布局中移除设置同步和协作工具模块，不删除对应源代码目录。

设置同步模块包括：

- `intellij.settingsSync.core`
- `intellij.libraries.jgit`

协作工具模块包括：

- `intellij.platform.collaborationTools`
- `intellij.platform.collaborationTools.auth`
- `intellij.platform.collaborationTools.auth.base`
- `intellij.platform.collaborationTools.shared`

移除设置同步后，用户不能通过 JetBrains 账户同步或恢复 IDE 设置，但本地设置保存和首次启动设置导入继续可用。移除协作工具后，Code With Me、协作邀请、协作会话和协作认证不可用。移除 Subversion 和 Mercurial 后，只保留 Git 版本控制。Git 的提交、分支、日志和远程操作不受影响。

### 在线服务和集成

一级阶段同时移除以下在线服务模块或插件：

- `intellij.platform.feedback`。
- `intellij.platform.statistics.uploader`。
- `intellij.vcs.github`。
- `intellij.vcs.gitlab`。
- `intellij.grazie`。
- AI/ML 插件和 `intellij.mcpserver.plugin`。

统计 API 如果仍被基础平台引用，则保留最小 API 或空实现，只移除统计上传实现。移除 GitHub 和 GitLab 集成不影响 Git 的 clone、fetch、pull、push、commit、branch 和 log。移除 AI、Grazie 和 MCP 功能不影响普通 Java 补全、检查、导航和重构。

保留插件管理、在线插件市场入口、IDE 更新检查和插件自动更新检查。保留通用网络能力和本地凭据存储，以支持 Maven 依赖下载、Git 远程仓库和私服认证。

## 分阶段交付

### 阶段 1：产品布局

- 创建 Lean 产品配置。
- 生成插件模型和产品依赖报告。
- 确认 Terminal 不在产品插件列表中。
- 确认 Java、Maven、Git 和调试器依赖完整。
- 确认设置同步和协作工具不在产品模块列表中。
- 确认反馈、统计上传、GitHub、GitLab、Grazie、AI/ML 和 MCP 功能不在产品模块列表中。
- 确认插件市场、插件管理和自动更新检查仍可用。
- 不删除源代码目录。

### 阶段 2：构建

- 通过产品布局验证。
- 确认插件依赖完整。
- 构建 macOS 安装包。
- 确认产品不包含 Terminal 插件。
- - 确认本地设置保存和首次启动设置导入仍可用。

### 阶段 3：功能验证

Maven 和 Java：

- 导入 Maven 项目。
- 下载依赖。
- 启动 Java 主类。
- 查看标准输出和错误输出。
- 使用 Debug 断点。
- 运行 JUnit。

其他核心功能：

- 验证 Git 状态、差异、提交、日志和分支。
- 验证 JAR 中的类搜索。
- 验证反编译库类的 Find Usages。
- 在 macOS 外部 Terminal 执行 Maven 和服务命令。
- 确认设置同步和协作功能入口不存在。
- 确认反馈、统计上传、GitHub/GitLab 集成和 AI/MCP 入口不存在。
- 确认插件市场和自动更新检查入口存在。

### 阶段 4：性能基线

记录精简前后的：

- 安装包大小。
- 冷启动时间。
- 热启动时间。
- 常驻内存。
- Java 项目索引时间。
- Maven 导入时间。

## 风险和应对

### 运行控制台依赖 Terminal API

平台运行和输出代码可能引用 Terminal API。直接删除 Terminal 源代码会造成编译或产品布局失败。

应对措施：先移除产品插件，再根据依赖报告保留运行控制台所需的最小底层模块。

### 现有社区产品受影响

修改默认产品配置可能影响标准 Community 构建。

应对措施：使用独立的 `LeanJavaIdeaProperties`，不修改现有社区产品的默认插件列表。

## 验收标准

产品布局验证通过，macOS 安装包构建成功。产品不包含 IDE Terminal 和 Gradle。JSP、Java EE/Jakarta EE、外置 Tomcat、应用服务器和 Java Remote Servers 保留。Maven 普通 Java 项目、Spring Boot 主类、`mvn spring-boot:run`、内嵌 Tomcat 日志、Java Debug、Maven 导入和依赖解析、JUnit、Git、设置同步、协作、反馈、统计上传、GitHub/GitLab 集成和 AI/MCP 功能按范围验证。未删除共享源代码目录。

## 后续计划

设计通过后，使用 Superpowers 的 writing-plans 流程生成逐任务实施计划。实施阶段再确定具体文件、模块、构建命令和测试命令。
