# Memory and Startup Modules

This document lists the module groups that affect IntelliJ IDEA memory and startup time. It separates build-time module sets from runtime plugins and external processes.

## How To Read This Document

IntelliJ builds the product from modules, module sets, and plugins.

- A **module** is a compile and packaging unit.
- A **module set** bundles related modules.
- A **plugin** is a runtime unit and owns its class loader.
- An **external process** runs outside the main IDEA JVM.

Module names change over time. Use the product dependency closure as the source of truth before removing a module.

## Impact Legend

| Level | Meaning |
| --- | --- |
| High | Strong startup or resident-memory impact. |
| Medium | Impact appears in common workflows or background work. |
| Low | Impact appears only after the feature starts. |

## Core Platform

| Module or set | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| `intellij.platform.core` | Basic services, disposable objects, extensions, and lifecycle control. | High. Starts with the product. | Keep. |
| `intellij.platform.core.impl` | Core service implementation and extension points. | High. Starts with the product. | Keep. |
| `intellij.platform.util.*` | Utilities for UI, text, coroutines, progress, storage, and serialization. | High. Many subsystems use them. | Keep. |
| `intellij.platform.concurrency` | Thread pools, coroutines, and background jobs. | High. Startup services use it. | Keep. |
| `intellij.platform.projectModel` | Projects, modules, roots, facets, libraries, and content roots. | High. Project opening and indexing use it. | Keep. |
| `intellij.platform.projectModel.impl` | Project model implementation and root detection. | High. Project import and indexing use it. | Keep. |
| `intellij.platform.workspace.*` | Workspace model and JPS model storage. | High. Large projects increase memory. | Keep. |
| `intellij.platform.indexing` | Index infrastructure and file indexing APIs. | High. CPU and memory depend on index size. | Keep. |
| `intellij.platform.indexing.impl` | Index implementation and background indexing. | High. Large indexes use memory and disk. | Keep. |
| `intellij.platform.analysis.*` | Base analysis and inspection services. | Medium to high. Code insight uses it. | Keep. |
| `intellij.platform.syntax.*` | New syntax-tree infrastructure. | Medium. Language support uses it. | Keep if Java/XML use it. |
| `intellij.platform.settings` | Settings model and persistence. | Medium. Startup reads user settings. | Keep. |
| `intellij.platform.pluginSystem.parser.impl` | Plugin descriptor parsing. | Medium. Startup loads plugin descriptors. | Keep. |
| `intellij.platform.icons.*` | Icon API and icon rendering. | Medium. UI startup loads icons. | Keep. |
| `intellij.platform.diagnostic.telemetry` | Telemetry API and OpenTelemetry types. | Medium. Starts with core platform. | Keep API if required. |
| `intellij.platform.diagnostic.telemetry.impl` | Telemetry export implementation. | Medium. Can send background data. | Trim for local builds. |
| `intellij.platform.diagnostic.telemetry.exporters` | OTLP and Jaeger exporters. | Medium. Adds libraries and background work. | Trim for local builds. |
| `intellij.platform.runtime.product` | Runtime module repository and product model. | Medium. Startup resolves modules. | Keep. |
| `intellij.platform.ide.core` | IDE core model and application services. | High. Starts with the product. | Keep. |

## Basic IDE Modules

| Module or set | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| `intellij.platform.ide` | Application and project UI shell, actions, tool windows, and menus. | High. Starts with the product. | Keep. |
| `intellij.platform.ide.impl` | IDE service implementations. | High. Starts with the product. | Keep. |
| `intellij.platform.lang.impl` | Language-aware IDE services. | High. Editor and navigation use it. | Keep. |
| `intellij.platform.editor` | Editor lifecycle and document editing. | High. Starts with an open project. | Keep. |
| `intellij.platform.editor.ui` | Swing editor UI. | High. Starts when a file opens. | Keep. |
| `intellij.platform.editor.backend` | Editor backend services. | Medium. Supports the editor. | Keep. |
| `intellij.platform.editor.frontend` | Editor frontend for split products. | Medium. Only split UI needs it. | Keep for monolith. |
| `intellij.platform.searchEverywhere` | Search Everywhere model and contributors. | Medium. Popup startup is lazy. | Keep. |
| `intellij.platform.searchEverywhere.backend` | Search backend and provider work. | Medium. Search can run in background. | Keep. |
| `intellij.platform.searchEverywhere.frontend` | Search Everywhere UI. | Medium. Popup startup is lazy. | Keep. |
| `intellij.platform.find` | Find in Files and usage-search infrastructure. | High. Text and usage indexes use it. | Keep. |
| `intellij.platform.find.backend` | Find work execution. | Medium. Large projects use background threads. | Keep. |
| `intellij.platform.completion.*` | Completion engine and UI. | Medium to high. Runs after indexing. | Keep. |
| `intellij.platform.inline.completion` | Inline completion service. | Medium. Can run in background. | Keep only if wanted. |
| `intellij.platform.execution.impl.*` | Run configurations and process execution. | Medium. Starts with run actions and consoles. | Keep. |
| `intellij.platform.externalSystem.impl` | External build-system integration base. | High. Maven import uses it. | Keep. |
| `intellij.platform.externalSystem.dependencyUpdater` | Dependency updates for external systems. | Medium. Can start background jobs. | Keep with Maven. |
| `intellij.platform.pluginManager.*` | Plugin discovery and installation. | Medium. Startup reads plugins. | Keep. |
| `intellij.platform.ide.updateChecker` | IDE update checking. | Low. Usually background. | Trim for controlled builds. |
| `intellij.platform.credentialStore.*` | Password and credential storage. | Low. Starts with services. | Keep only if Git or plugins need it. |
| `intellij.platform.builtInServer.impl` | Built-in HTTP server. | Low. Opens a local service. | Keep only if required. |
| `intellij.platform.managed.cache.*` | Managed caches. | Medium. Large projects use memory. | Keep. |
| `intellij.platform.progress.backend` | Background progress and tasks. | Medium. Startup and imports use it. | Keep. |

## Language and Editing Sets

| Module or set | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| `corePlatform()` | Platform without IDE or language UI. | High. It is the base. | Keep. |
| `coreIde()` | Platform plus basic IDE UI. | High. Most IDE products need it. | Keep. |
| `coreLang()` | Platform plus language support. | High. Editor and navigation need it. | Keep. |
| `essentialMinimal()` | Editor, search, completion, backend/frontend split. | High. Starts with a project. | Keep. |
| `essential()` | Find, completion, execution, plugin manager, external systems. | High. Most IDE products need it. | Keep. |
| `intellij.platform.xml.*` | XML parser, PSI, DOM, analysis, and UI. | Medium to high. POM and configs use it. | Keep. |
| `intellij.relaxng` | RELAX NG support. | Low. Starts with XML use. | Trim if not needed. |
| `intellij.xml.emmet.*` | Emmet abbreviations for markup. | Low. Starts with HTML/XML editing. | Trim for Java-only builds. |
| `intellij.jsp.base` | Shared JSP API. | Low. Starts with JSP files. | Trim unless JSP is needed. |
| `lsp()` | Language Server Protocol support. | Medium. Can start language servers. | Trim unless using LSP. |
| `intellij.platform.duplicates.analysis` | Duplicate-code analysis. | Low. Starts when invoked. | Trim for lean builds. |
| `platformTestFrameworksCore()` | IDE test framework modules. | High for tests. Not needed in the IDE runtime. | Exclude from product. |
| `platformTestFrameworksJunit5()` | JUnit 5 IDE test fixtures. | High for tests. Not needed in the IDE runtime. | Exclude from product. |

## Debug and Version Control

| Module or set | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| `intellij.platform.debugger` | Debugger API. | Medium. Starts with debug sessions. | Keep for Java development. |
| `intellij.platform.debugger.impl` | Debugger implementation. | Medium to high. Debug UI and engine use it. | Keep. |
| `intellij.platform.debugger.impl.ui` | Debugger views and controls. | Medium. Starts with debug UI. | Keep. |
| `intellij.platform.debugger.impl.backend` | Debugger backend evaluation and process control. | Medium. Starts with debug sessions. | Keep. |
| `intellij.platform.vcs` | Version-control API and project integration. | Medium. Starts with VCS changes. | Keep. |
| `intellij.platform.vcs.core` | File and change-model support. | Medium. Starts with a Git project. | Keep. |
| `intellij.platform.vcs.impl` | VCS UI, change lists, log, and shelves. | Medium to high. Background refresh uses it. | Keep. |
| `intellij.platform.vcs.dvcs.impl.shared` | Distributed VCS support. | Medium. Git uses it. | Keep. |
| `intellij.libraries.jgit` | Pure-Java Git implementation. | Medium. Used by settings sync or special tasks. | Trim unless required. |

## UI and Remote Sets

| Module or set | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| `compose()` | Compose, Skiko, Coil, Jewel, and Markdown bridge. | High. Adds large JARs and native code. | Trim if the classic UI is enough. |
| `intellij.platform.compose.swing` | Compose-to-Swing integration. | High. Starts with Compose UI. | Trim if no Compose UI. |
| `intellij.platform.jewel.*` | Compose-based IDE UI components. | High. Starts with Compose UI. | Trim if no Compose UI. |
| `rdCommon()` | Remote development and split IDE protocol. | Medium to high. Adds RPC models. | Trim if no remote IDE. |
| `intellij.rd.client.*` | Remote client services. | Medium. Remote development uses it. | Trim if no remote IDE. |
| `intellij.platform.frontend` | Frontend shell for split IDEs. | Medium. Monolith still packages it. | Review dependency closure. |
| `intellij.platform.monolith` | Monolithic IDE assembly. | Medium. Allows one-process operation. | Keep. |
| `intellij.platform.split.protocol` | Split frontend/backend protocol. | Medium. Remote development uses it. | Trim if no remote IDE. |
| `intellij.platform.collaborationTools.*` | Collaboration and shared sessions. | Medium. Adds services and auth. | Trim. |
| `intellij.platform.settings.local` | Local settings backend. | Medium. Starts with settings. | Keep. |
| `settingsSync()` | Settings synchronization and JGit. | Medium. Can run background sync. | Trim. |

## Bundled Plugins

### Keep For Lean Java IDE

| Plugin | Main function | Memory and startup effect |
| --- | --- | --- |
| `intellij.java.plugin` | Java PSI, inspections, navigation, refactoring, compiler integration, Java run/debug. | High. Core for Java projects. |
| `intellij.java.ide.customization` | Java IDE customization and presentation. | Low to medium. Starts with product. |
| `intellij.maven.plugin` | Maven import, POM model, dependency resolution, Maven tool window, Maven run. | High. Starts with Maven projects. |
| `intellij.vcs.git` | Git status, diff, commit, branches, history, and fetch/push. | Medium. Starts with Git projects. |
| `intellij.java.decompiler` | Decompiler for class files and dependency JARs. | Medium. Supports JAR navigation. |
| `intellij.junit` | JUnit test detection and run/debug integration. | Medium. Starts with Java tests. |
| `intellij.lombok.plugin` | Lombok support and generated-member visibility. | Medium. Starts with Lombok projects. |
| `intellij.properties` | Properties-file editing and resource references. | Low. Starts with properties files. |
| `intellij.editorconfig.plugin` | EditorConfig support. | Low. Starts with config files. |
| `intellij.json` | JSON parser, editor, schema, and navigation. | Medium. Starts with JSON configs. |
| `intellij.yaml` | YAML parser, editor, and navigation. | Medium. Starts with YAML configs. |
| `intellij.xml.plugin` | XML editing and validation. | Medium. Starts with XML and POM files. |
| `intellij.platform.vcs.plugin` | VCS UI and project integration. | Medium. Starts with VCS projects. |
| `intellij.platform.searchEverywhere` | Search Everywhere. | Medium. Starts with first search. |

### Commonly Trimmed

| Plugin | Main function | Memory and startup effect |
| --- | --- | --- |
| Kotlin plugin | Kotlin language support and compiler integration. | High. Adds language, daemon, and libraries. |
| Gradle plugins | Gradle import, sync, run, and tooling. | High. Starts daemon and sync tasks. |
| Groovy plugins | Groovy support and scripting. | High. Adds language and runtime. |
| Android plugins | Android project model and tools. | High. Adds large model and build support. |
| JavaFX plugin | JavaFX project support. | Medium. Starts with JavaFX projects. |
| Terminal plugin | Embedded terminal and PTY. | Medium. Starts with terminal tool window. |
| Markdown plugin | Markdown editing and preview. | Low to medium. Preview may start JCEF. |
| Mermaid plugin | Mermaid diagram rendering. | Medium. Preview uses a browser runtime. |
| JCEF plugin | Embedded Chromium browser. | High. Adds native browser and helper processes. |
| AI and ML plugins | AI assistance, ML ranking, and smart features. | Medium to high. Adds models and background work. |
| Grazie | Writing and grammar support. | Low to medium. Starts with text features. |
| Features trainer | Onboarding lessons. | Low. Starts with onboarding. |
| GitHub/GitLab | Remote repository and review services. | Medium. Adds network and VCS services. |
| Coverage | Coverage collection and visualization. | Medium. Runs instrumentation agents. |
| TestNG | TestNG test support. | Low to medium. Starts with TestNG projects. |
| Eclipse integration | Eclipse project import. | Low. Starts with Eclipse projects. |
| Shell support | Shell script editing. | Low. Starts with shell files. |
| TOML | TOML editing. | Low. Starts with TOML files. |
| TextMate | Generic TextMate grammars. | Low. Starts with matching files. |
| WebP | WebP image support. | Low. Starts with images. |
| Alternative keymaps | Eclipse, Visual Studio, and NetBeans keymaps. | Low. Adds startup descriptors. |
| Compose IDE plugin | Jetpack Compose support. | Medium. Starts with Compose files. |
| Performance testing | Startup and performance tests. | Low. Not useful for normal users. |
| MCP server plugin | Model Context Protocol services. | Medium. Adds network service. |

## External Processes And Large Runtimes

| Runtime | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| JetBrains Runtime | Java runtime for IDEA. | High. Sets JVM baseline. | Use the correct macOS runtime. |
| JCEF | Chromium embedded browser. | High. Adds helper processes and native memory. | Remove unless required. |
| Maven server | Dependency resolution and Maven model import. | High during import. | Keep for Maven projects. |
| Maven repository index | Full artifact search. | High if persistent. | Turn on only when needed. |
| Gradle daemon | Gradle model import and tasks. | High. Separate JVM. | Remove Gradle support. |
| Kotlin daemon | Kotlin compilation and script support. | High. Separate JVM. | Remove Kotlin support. |
| Java compiler | Incremental Java compilation and JPS build. | High during build. | Keep. |
| Git executable | Local Git operations. | Medium. Runs during VCS refresh. | Keep. |
| LSP server | External language services. | Medium. Separate process. | Remove unless required. |
| AI services | Remote or local AI helpers. | Medium. Network or background work. | Remove. |

## Background Work

| Work | Main function | Memory and startup effect | Lean decision |
| --- | --- | --- | --- |
| Project indexing | Builds symbol, usage, file, and framework indexes. | High CPU and memory. | Keep; reduce scope. |
| Framework detection | Detects Spring, Maven, and other frameworks. | Medium CPU. | Keep Maven and Java. |
| Maven sync | Imports POMs and dependencies. | High CPU, memory, and I/O. | Keep; run on demand. |
| VCS refresh | Reads Git status and changes. | Medium CPU and I/O. | Keep. |
| Plugin update scan | Reads and checks plugins. | Low to medium. | Disable automatic checks. |
| Telemetry export | Sends diagnostics. | Low to medium. | Remove for local builds. |
| Settings sync | Uploads settings. | Low to medium. | Remove. |
| Search indexing | Updates Find in Files data. | Medium CPU and memory. | Keep. |
| Git log indexing | Builds commit history views. | Medium CPU and memory. | Keep only if needed. |

## Lean Product Matrix

| Goal | Keep | Remove |
| --- | --- | --- |
| Java editing | Java plugin, core language support, editor, completion. | Kotlin, Groovy, Android, unsupported languages. |
| Maven project | Maven plugin, external system support, Java compiler. | Gradle, Kotlin daemon, other build systems. |
| Service startup | Run configuration support, Java execution. | Unneeded application servers and remote services. |
| Debugging | Java debugger and JUnit runner. | Script debugger, remote debugger, TestNG if not needed. |
| JAR search | Java indexes, decompiler, Maven dependency roots. | Full Maven Central index by default. |
| Git | VCS core, Git plugin, credential store. | GitHub, GitLab, task servers. |
| Search | Search Everywhere, Find in Files, class index. | AI search, extra contributors, large repository index. |
| macOS package | macOS runtime and native code. | Windows, Linux, cross-platform artifacts. |

## Measurement Checklist

Measure each item before and after a trim:

- `.app` size.
- Cold startup time.
- Warm startup time.
- Resident memory of the main process.
- Resident memory of helper processes.
- Index time and index disk size.
- Maven sync time and Maven server memory.
- Time to first editor input.
- Time to first Search Everywhere result.

Compare one change at a time. Use the same project, operating system, JVM options, cache state, and hardware.
