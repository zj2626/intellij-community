# Lean macOS Java IDE Plan

## Goal

Build a lean macOS IDE for Java and Spring projects.

Keep project indexing, Maven import, Java compilation, service startup, debugging, Git, project management, search, and JAR search.

## Main Approach

Do not delete source directories first.

Create a new product configuration. Build a dependency closure for the required functions. Remove unused plugins and platform modules from the product. Delete unused source directories only after the product is stable.

This approach reduces merge problems and avoids breaking shared platform modules.

## Product Scope

Keep:

- Core IDE platform.
- Editor, completion, inspections, navigation, run, and debug.
- Project model and indexing.
- Search Everywhere, Find in Files, class search, and library class search.
- Maven and Java support.
- Git and project management.
- Git version control only. Do not bundle Subversion or Mercurial.
- Do not bundle Gradle.
- Keep Tomcat and application server support.
- XML, properties, JSON, YAML, JUnit, and Lombok.
- macOS only.

Spring note:

- The open-source repository does not contain the Spring-specific plugin.
- Maven import, Java compilation, and Spring Boot startup work without that plugin.
- Spring Bean navigation and Spring inspections require the commercial plugin. Do not distribute that plugin with an open-source build.

## Trimming Levels

### Level 1: Bundled Plugins

Start with a plugin whitelist.

Keep:

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

Remove:

- Kotlin.
- Groovy.
- Gradle.
- Android.
- JavaFX.
- JCEF.
- Terminal.
- Markdown.
- Mermaid.
- TOML.
- TextMate.
- GitHub and GitLab.
- Subversion and Mercurial.
- Grazie.
- AI and ML features.
- Onboarding and feature trainer.
- Performance testing.
- Compose IDE plugin.
- Alternative keymaps.
- Sh, Eclipse, WebP, and other low-use plugins.

This level has the lowest risk.

### Level 2: Platform Modules

Reduce the platform content after the plugin whitelist works.

- Use `essential()` instead of the larger `ideCommon()`.
- Add debugger, VCS, Java, Maven, XML, and search modules explicitly.
- Remove Compose.
- Remove spellchecker.
- Remove settings sync.
- Remove collaboration tools.
- Remove RD and JetBrains Client dependencies when the dependency closure allows it.
- Keep the Java plugin dependency closure intact.

Compose is about 80 MB of JARs plus the Skiko native runtime.

### Level 3: Source Tree

Only delete source directories after the Lean build passes the smoke tests.

Use the product dependency report to identify unused modules. Do not delete directories by directory name alone.

## Code Changes

Create `LeanJavaIdeaProperties` as a subclass of `IdeaCommunityProperties`.

Set:

- A bundled-plugin whitelist.
- A lean product content descriptor.
- `buildCrossPlatformDistribution = false`.
- `buildSourcesArchive = false`.
- `buildAllCompatiblePlugins = false`.
- A macOS distribution customizer.

The current community plugin list starts in `build/src/org/jetbrains/intellij/build/IdeaCommunityProperties.kt:62`.

The current default bundled-plugin list starts in `platform/build-scripts/src/org/jetbrains/intellij/build/BaseIdeaProperties.kt:14`.

After changing the product layout, run:

```bash
./bazel run //platform/buildScripts:plugin-model-tool
```

## macOS Build

Set `targetOs` to macOS. On Apple hardware, set `targetArch` to the current architecture.

Skip:

- Windows installers.
- Linux installers.
- Cross-platform distribution.
- Source archive.
- macOS signing and notarization for local builds.
- Non-bundled plugin publishing.

Use:

```bash
-Dintellij.build.target.os=mac
```

The current installer build target is `build/src/OpenSourceCommunityInstallersBuildTarget.kt`.

## Startup and Memory Controls

Remove:

- Embedded browser.
- AI and ML services.
- Collaboration services.
- Settings sync.
- Onboarding.
- Plugin auto-update prompts.

Indexing controls:

- Keep Java and project indexes.
- Keep Maven dependency JAR indexing.
- Turn the full Maven repository index on only when needed.
- Exclude `target/`, `build/`, generated directories, and large local resources.

Memory tuning:

Measure cold startup, warm startup, resident memory, index time, and app size first. Then tune:

- `-Xmx`.
- `-XX:MaxMetaspaceSize`.
- `-XX:ReservedCodeCacheSize`.

Do not reduce memory before collecting the baseline.

## Validation

Build:

- Product layout validation passes.
- macOS package builds successfully.

Java and Maven:

- Open a Maven project.
- Java resolution, completion, navigation, refactoring, and debugging work.
- Maven import, dependency download, Run, Debug, and Maven tooling work.

Spring:

- Spring Boot main class starts.
- Debugger stops on a breakpoint.
- Spring-specific navigation is available only with the commercial Spring plugin.

JAR search:

- `Go to Class` finds classes from dependency JARs.
- `Find Usages` works with decompiled library classes.

Git:

- Status, diff, commit, log, and branches work.

Performance:

- Record app size.
- Record cold startup time.
- Record warm startup time.
- Record resident memory.
- Record indexing time.
- Compare all values with the untrimmed build.

## Recommended First Delivery

Deliver Level 1 first:

1. macOS-only build.
2. Plugin whitelist.
3. Removal of JCEF, AI, ML, multiple languages, and multiple build systems.
4. Performance baseline and smoke tests.

Move to Level 2 only after the Level 1 build is stable.
