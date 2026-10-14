# Lean macOS Java 产品实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个只面向 macOS 的精简 Java IDE 产品，保留 Maven/Java Run、Debug、Spring Boot 内嵌 Tomcat 日志和日志控制台，并移除 Gradle、外置 Tomcat、应用服务器、JSP 与 Java EE/Jakarta EE。

**Architecture:** 新增独立的 Lean 产品属性类和构建目标，不修改现有 Community 产品的默认插件列表。产品使用明确的插件白名单和模块内容描述符；仅从发行内容中移除功能，不删除共享源代码目录。在线依赖、Maven 仓库访问和本地凭据存储继续保留。

**Tech Stack:** Kotlin、IntelliJ Product DSL、Bazel、JPS 模块模型、macOS 产品构建。

---

## 文件和职责

- **Create:** `build/src/org/jetbrains/intellij/build/LeanJavaIdeaProperties.kt`：定义 Lean 产品属性、插件白名单、macOS 定制器和产品内容描述符。只有在专用模块集合可用后创建此文件。
- **Modify:** `platform/build-scripts/src/org/jetbrains/intellij/build/productLayout/CommunityModuleSets.kt`、`CoreModuleSets.kt` 和相关产品片段：提供不包含在线服务、Compose、Onboarding 和 Terminal 内容的 Lean 模块集合。不能使用通用 `ideCommon()`、`essential()` 或 `coreLang()` 后再尝试排除模块，因为当前 DSL 没有可靠的模块排除 API。
- **Create:** `build/src/LeanJavaIdeaInstallersBuildTarget.kt`：定义 Lean macOS 构建入口，并跳过跨平台包、源码包和非 macOS 安装包。
- **Modify:** `build/BUILD.bazel`：新增 Lean 构建目标，并复用现有 `:build` 运行时依赖。
- **Modify:** `docs/lean-macos-java-plan.md` 和 `docs/lean-macos-java-plan.zh-CN.md`：同步已确认的产品范围。
- **Test:** 产品布局验证、插件模型工具、macOS 构建和人工冒烟测试记录。

## 产品边界

保留 Java、Maven、Git、JUnit、Lombok、XML、JSON、YAML、properties、Java/Maven Run、Debug、Spring Boot 内嵌 Tomcat、本地插件管理、在线插件市场、IDE 更新检查、插件自动更新、通用网络能力和本地凭据存储。

移除 `intellij.terminal`、Gradle、JSP、Java EE/Jakarta EE、外置 Tomcat、应用服务器、Java Remote Servers、Settings Sync、Collaboration Tools、Feedback、Statistics uploader、GitHub/GitLab 集成、Subversion、Mercurial、Grazie、AI/ML、MCP Server、Kotlin、Groovy、Android、JavaFX、JCEF、Markdown、Mermaid、TOML、TextMate、性能测试、功能培训和其他低频插件。统计 API 如果仍被基础平台引用，只保留 API 或空实现。不要删除共享源代码目录。

### Task 1: 确认现有产品布局和构建入口

**Files:** Read `build/src/org/jetbrains/intellij/build/IdeaCommunityProperties.kt`, `build/src/OpenSourceCommunityInstallersBuildTarget.kt`, `platform/build-scripts/src/org/jetbrains/intellij/build/productLayout/CommunityModuleSets.kt`, `platform/build-scripts/src/org/jetbrains/intellij/build/productLayout/CommunityProductFragments.kt`, and `platform/build-scripts/src/org/jetbrains/intellij/build/BaseIdeaProperties.kt`.

- [ ] **Step 1:** 运行 `rg -n 'IDEA_BUNDLED_PLUGINS|productLayout|intellijCommunityBaseFragment|ideCommon\\(|settingsSync\\(|collaborationTools|statistics|feedback' build/src platform/build-scripts/src/org/jetbrains/intellij/build`，记录默认插件、内容片段及在线服务模块来源。
- [ ] **Step 2:** 运行 `rg -n 'OpenSourceCommunityInstallersBuildTarget|BuildTarget|build/src/.*BuildTarget' build platform/build-scripts --glob 'BUILD*' --glob '*.bzl' --glob '*.kt'`，确认构建目标注册方式。
- [ ] **Step 3:** 使用产品模型工具建立候选依赖边界，确认 Java/Maven Run Console、Debugger、Gradle Import、Git Core、Credential Store、Plugin Manager、Update Checker 保留，Terminal、Settings Sync、Collaboration、Feedback、Statistics Uploader、GitHub/GitLab、AI/ML/MCP 移除。记录 `ideCommon()`、`essential()` 和 `coreLang()` 的隐式模块，因为它们会带入设置同步、协作、反馈和统计模块。

### Task 2: 创建 Lean 专用模块集合

**Files:** Modify `platform/build-scripts/src/org/jetbrains/intellij/build/productLayout/CommunityModuleSets.kt`, `CoreModuleSets.kt`, `CommunityProductFragments.kt`.

- [ ] **Step 1:** 为 Lean 产品增加可组合的模块集合参数，显式跳过 Settings Sync、Collaboration Tools、Feedback、Statistics uploader、Compose、Onboarding 和 Terminal welcome 内容。
- [ ] **Step 2:** 保留 Java、Maven、、执行控制台、调试器、Git、凭据、插件管理和更新检查模块。
- [ ] **Step 3:** 添加 Product DSL 测试，断言 Lean 集合不包含移除模块，并包含 Run/Debug、和更新模块。
- [ ] **Step 4:** 运行 `./tests.cmd --module intellij.platform.buildScripts.productDsl.tests --test "org.jetbrains.intellij.build.productLayout.*"`。预期测试通过。

### Task 3: 创建 Lean 产品属性类

**Files:** Create `build/src/org/jetbrains/intellij/build/LeanJavaIdeaProperties.kt`.

- [ ] **Step 1:** 创建继承 `IdeaCommunityProperties` 的公开类，设置 `buildCrossPlatformDistribution = false`、`buildSourcesArchive = false` 和 `productLayout.buildAllCompatiblePlugins = false`，复用现有 macOS 定制器。
- [ ] **Step 2:** 定义插件白名单，保留 Java、Maven、、Git、JUnit、Lombok、properties、EditorConfig、JSON、YAML、XML、VCS、搜索、结构视图、最近文件、问题视图和测试运行器；排除已确认的 Terminal、在线服务、语言插件和低频插件；保留插件市场及更新模块。
- [ ] **Step 3:** 定义 Lean 产品内容描述符。不要直接使用会隐式带入移除项的完整 `ideCommon()`、`essential()` 或 `coreLang()` 集合。复制并组合所需的最小模块集合，保留 Java、Maven、、执行框架、调试器、Git、凭据、插件管理和更新模块；排除 Settings Sync、协作、反馈、统计上传和 Terminal。
- [ ] **Step 4:** 运行 `rg -n 'intellij\\.platform\\.statistics|statistics\\.uploader|Statistics' platform plugins java --glob '*.xml' --glob '*.kt' --glob '*.java'`，只移除统计上传实现，不删除仍被核心平台引用的 API。

### Task 4: 创建 macOS Lean 构建目标

**Files:** Create `build/src/LeanJavaIdeaInstallersBuildTarget.kt`; modify the BUILD file identified in Task 1.

- [ ] **Step 1:** 复用 `OpenSourceCommunityInstallersBuildTarget` 的构建生命周期，但使用 `LeanJavaIdeaProperties` 创建构建上下文。跳过 Windows、Linux、跨平台分发、源码归档、macOS 签名和公证、兼容插件归档。
- [ ] **Step 2:** 在 `build/BUILD.bazel` 增加 `java_binary(name = "lean_java_i_build_target", main_class = "LeanJavaIdeaInstallersBuildTarget", runtime_deps = [":build"])`，并复制现有 `i_build_target` 的 `data`、`add_opens` 和 Bazel targets JSON 配置。不要手工修改 `.idea/modules.xml`。若新增或修改 `.iml`，运行 `bun build/jps-module.mjs register <path-to-iml> --fix-iml-eof` 和 `./build/jpsModelToBazelCommunityOnly.cmd`。
- [ ] **Step 3:** 运行 `bazel build //build:lean_java_i_build_target`，预期构建脚本编译通过。

### Task 5: 生成产品模型并修正依赖闭包

**Files:** Modify `build/src/org/jetbrains/intellij/build/LeanJavaIdeaProperties.kt`; inspect generated product model and dependency report.

- [ ] **Step 1:** 运行 `./bazel run //platform/buildScripts:plugin-model-tool`，预期产品布局生成成功。
- [ ] **Step 2:** 确认发行内容不包含 `intellij.terminal`、`intellij.settingsSync.core`、`intellij.libraries.jgit`、协作模块、`intellij.platform.feedback`、统计上传、`intellij.vcs.github`、`intellij.vcs.gitlab`、`intellij.grazie` 和 `intellij.mcpserver.plugin`。确认这些模块没有被 `ideCommon()`、`essential()` 或 `coreLang()` 隐式带入。
- [ ] **Step 3:** 确认 Java、Maven、执行控制台、调试器、Git Core、凭据、插件管理、更新检查和 Maven 网络依赖完整。
- [ ] **Step 4:** 每次只修正一个依赖边界并重复模型生成，直到保留项完整且移除项没有被隐式带回。不要删除源代码目录。

### Task 6: 验证布局和构建产物

**Files:** Test the Lean product layout report and macOS build output.

- [ ] **Step 1:** 运行 `./bazel run //platform/buildScripts:plugin-model-tool`，预期无未解析插件依赖。
- [ ] **Step 2:** 使用 `//build:lean_java_i_build_target` 和 `-Dintellij.build.target.os=mac` 构建 macOS 产品，预期只生成 macOS 安装包。
- [ ] **Step 3:** 检查发行目录：移除项不存在；Java、Maven、、Git、调试器、插件市场和更新检查存在。

### Task 7: 执行核心功能冒烟验证

**Files:** Test records for Maven, Gradle, Java, Git, credentials and plugin management.

- [ ] **Step 1:** Maven/Java：验证导入、依赖下载、Java 主类 Run、Debug 断点、Run Console 标准输出和错误输出、JUnit。
- [ ] **Step 2:** Gradle：验证项目识别、同步、依赖解析、Java 索引、补全和导航；不验证 Gradle 任务运行、Gradle 工具窗口或 Gradle 专用 Run/Debug。
- [ ] **Step 3:** Git/凭据：验证状态、差异、提交、日志、分支、fetch、pull、push，以及 Maven 私服凭据读取；确认 GitHub/GitLab UI 不存在。
- [ ] **Step 4:** 确认插件管理、在线插件市场、IDE 更新检查和插件自动更新仍可用。
- [ ] **Step 5:** 确认 Settings Sync、Code With Me、Feedback、Statistics upload、GitHub/GitLab UI、Grazie、AI 和 MCP 入口不存在。

### Task 8: 记录性能基线和完成检查

**Files:** Create `docs/lean-macos-java-validation.md`.

- [ ] **Step 1:** 记录精简前后的 macOS 包体积、冷启动、热启动、常驻内存、Java 索引时间、Maven 导入时间和 时间。
- [ ] **Step 2:** 运行 `git diff --check`。只有修改 `.iml`、`BUILD.bazel` 或 `.idea/` 时，才运行 `./build/jpsModelToBazelCommunityOnly.cmd`。
- [ ] **Step 3:** 运行受影响模块的 `./tests.cmd --module <affected-module> --test <fully.qualified.test.name>`。产品布局没有专用测试时，以布局验证和 macOS 构建作为门槛。
- [ ] **Step 4:** 如果 ijproxy 或 JetBrains MCP 可用，使用 `lint_files` 检查新增和修改的 Kotlin 文件。
- [ ] **Step 5:** 运行 `git status --short` 和 `git diff --stat`，确认只包含 Lean 产品、计划文档和验证记录。未经用户分别确认，不执行 `git commit` 或 `git push`。
