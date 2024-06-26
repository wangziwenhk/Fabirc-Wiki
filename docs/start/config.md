## 前置
- Java 21（推荐用于 1.21 ）的 JDK （即用于开发 Java 的工具，安装器可参考 [Adoptium](https://adoptium.net/releases.html) ）
  - 专业用户可以从 [Java JDK](http://jdk.java.net/) 获取 JDK ，注意需要手动解压并设置系统变量。
- 任意 IDE（集成开发环境）：例如 [Intellij IDEA](https://www.jetbrains.com/idea/) 或 [Eclipse](https://www.eclipse.org/downloads/) ，也可使用任何可编辑代码的文本编辑器，如 [Visual Studio Code](https://code.visualstudio.com/)
  - 如果对这些不熟悉，推荐使用 Intellij IDEA ，绝大多数开发者都用这个编写模组。

## 配置步骤
你可以使用 [在线 Mod 示例生成器](https://fabricmc.net/develop/template/) 来获取构建 Fabric 所需的大部分资源。

部分地区的用户可能会发现，由于网速原因，构建 Gradle 速度可能比较慢。对于中国内地用户，可参考 [加快 Fabric 模组构建速度](../optimization/speedBuild.md) 。
如果你已经打开代理，但是下载速度仍然比较慢，可以检查一下 IDE 的网络代理设置是否处于关闭状态。
对于 IntelliJ IDEA ，你可以在 File $\rightarrow$ Settings $\rightarrow$ Appearance & Behavior $\rightarrow$ System Settings $\rightarrow$​ HTTP Proxy 找到该设置选项卡，如果勾选的是 "No proxy" ，要切换至 "Auto-detect proxy settings" 

### 手动步骤

<ol markdown>
<li markdown>从 [在线 Mod 示例生成器](https://fabricmc.net/develop/template/) 下载所需的所有文件，并删除 `LICENSE` （许可证）以及 `README.md` （简介）文件，因为他们只适用于模板，而并不适用于你的模组。</li>
<li markdown>编辑 `gradle.properties`（通常情况下，这些已经被生成器自动设置好了）
<ol markdown>
<li markdown>设置 `archives_base_name`  ，这是你的 Mod ID ，你需要确保它是唯一的。</li>
<li markdown>设置 `maven_group` ，这是你的包前缀。如果你不确定用途，请将它设置为 `name.modid`</li>
<li markdown>确保更新 Minecraft、Yarn 映射、加载器和 loom 的版本，你可以在 [FabricMC](https://fabricmc.net/develop/) 查询 </li>
<li markdown>添加你需要在 `build.gradle` 中使用的其他依赖。</li>
</ol>
</li>
<li markdown>将 `build.gradle` 导入到你的 IDE 中（各 IDE 操作各有不同，具体见下）</li>
</ol>




如有需要，可以在 `gradle.properties` 中设置镜像源。镜像源可能有时不可用，或者内容更新滞后，如需使用官方源，可暂时将相关内容注释掉即可。BMCLAPI 镜像缓慢时，可使用官方源，或参考 [校园网联合镜像站](https://mirrors.cernet.edu.cn/list/bmclapi) 。

```properties
loom_libraries_base=https://bmclapi2.bangbang93.com/maven/
loom_resources_base=https://bmclapi2.bangbang93.com/assets/
loom_version_manifests=https://bmclapi2.bangbang93.com/mc/game/version_manifest.json
loom_experimental_versions=https://maven.fabricmc.net/net/minecraft/experimental_versions.json
loom_fabric_repository=https://repository.hanbings.io/proxy/
```

### 导入项目

#### IntelliJ IDEA

如果你使用的是 JetBrains 的 IntelliJ IDEA，请遵循以下步骤（注：中文文本可能会因为 IDEA 或中文插件的版本不同而不同）：

1. 在 IDEA 的主菜单里选择`打开或导入…` (`Import Project`)（如果已经打开了一个项目，选择位于顶端的`文件/打开…`）。
2. 选择项目的 `build.gradle` 文件以导入项目。
3. 在 Gradle 配置完成后，关闭并重新加载项目，否则有些运行配置可能无法正常显示。
4. 如果运行配置还没有出现，试试在 Gradle 页面里重新导入项目。

*可选，但推荐做的一件事*： IDEA 默认使用 Gradle 来构建你的项目，而这在 Fabric 是不必要的，而且会导致构建时间变长以及热交换 (hotswapping) 相关的种种问题 (1) 。以下是让 IDEA 使用默认编译器的步骤：
{ .annotate }

1.  不要运行 `idea` 的 gradle 任务，已知它会破坏开发环境。

<div></div>

1. 在 Gradle 页面里打开 **Gradle 设置 (Gradle Settings)**
2. 将 **使用此工具构建和运行 (Build and run using)** 和 **使用此工具运行测试(Run tests using)** 选项改成 **IntelliJ IDEA** 。


不幸的是，目前还不能给 **使用此工具构建和运行** 和 **使用此工具运行测试** 设置一个全 IDE 内的默认值，所以这些每创建一个新项目都得重复上述步骤。

如果你使用 IntelliJ IDEA，你可以使用 [MinecraftDev 插件](https://plugins.jetbrains.com/plugin/8327) (1) 。该插件支持自动生成 Fabric 项目以及一些与 Mixin 有关的功能，如检查、生成访问器 (accessor) 和影子 (shadow) 字段、复制 Mixin 目标参考 (JVM 描述符) 。你可以在文件 (File) $\rightarrow$ 设置 (Settings) $\rightarrow$  插件(Plugins) 中打开内部插件浏览器，找到并安装这个插件，只需要在搜索框里搜索 **Minecraft**，选择第一个结果安装即可。
{ .annotate }

1. MCDev 插件中的模板会直接使用 loom 的最新不稳定版本，请小心使用

#### Eclipse
如果你使用的是 Eclipse，并且想要生成 IDE 的运行设置，请运行 gradlew eclipse。然后，项目就可以作为普通的（非 Gradle）的 Eclipse 项目导入你的工作区内，方法就是 "文件" $\rightarrow$ "导入…" 菜单，然后 "通用" $\rightarrow$  "已存在的项目导入工作区" 。

#### Visual Studio Code
推荐安装以下插件 (plugins) ，以尽可能提高效率。您只需要安装其一。

- [Language Support for Java(TM) by Red Hat](https://marketplace.visualstudio.com/items?itemName=redhat.java)
- [Debugger for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug)
- [Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)

注意插件 `Language Support for Java(TM) by Red Hat` 需要 Java 11 或更高版本。详情请参考 [JDK Requirements](https://github.com/redhat-developer/vscode-java/wiki/JDK-Requirements)

将项目 (project) 复制到一个文件夹，在 Visual Studio Code 中打开这个文件夹。稍后 IDE 应该导入 Gradle 项目（如果30秒内不开始，打开 build.gradle 文件）。

完成后，游戏引擎的代码编译就可以了，这对模组开发有帮助。

如要带有调试支持地运行游戏，你需要生成运行配置 (run configs) 。这可以通过运行 Gradle vscode 任务来完成。打开终端窗体的一种方式是前往查看 $\rightarrow$ 终端。这将会在打开的项目目录中打开终端面板。然后，运行 `./gradlew vscode` —— 这会自动生成包含运行配置的必需的 launch.json。

最后，要启动游戏，选择左边任务栏的“调试”菜单项。这将会自动建立模组，启动游戏。

如需浏览 Minecraft 资源，你可以使用 Gradle genSources 任务。你可以在终端中运行 `./gradlew genSources` 来完成。

然后，你需要刷新 Java 项目，可以通过按 `Shift + Alt + U` 来完成，build.gradle 文件仍是打开的。

要寻找 Minecraft 类，你可以通过 `Ctrl + P` 打开搜索面板，加上前缀 `#` 来搜索 Minecraft 类。

## 生成 Minecraft 源代码
阅读 Minecraft 源代码是编写模组时的重要一部分。但是，我们不能发布 Minecraft 的源代码，因为这违反了 [Minecraft 的最终用户许可协议 (EULA)](https://zh.minecraft.wiki/w/Minecraft_Wiki:Wiki%E6%9D%A1%E4%BE%8B) 。你需要自己生成 Minecraft 源代码。

要生成 Minecrat 源代码，运行 gradle 任务 `genSources` 。如果你的 IDE 没有嵌入 gradle，在终端内运行以下命令： `./gradlew genSources` 。反编译可能需要一段时间，取决于计算机的能力。你可能需要在运行任务之后刷新 gradle。

如何阅读源代码，可参考 [阅读 Minecraft 源代码](../base/readMcCode.md) 。

## 新手入门

入门可以先尝试 [添加一些物品](../item/addItem.md) 和 [方块](../block/addBlock.md) 。另外，建议了解一下如何 [在不重启 Minecraft 的情况下应用更改](../base/applyChanges.md) ，以便调试。

## 建议

- 虽然 Fabric API 并不是必需的，但其最首要的目标是提供游戏引擎所不提供的跨模组兼容性和接口，所以我们**强烈**推荐多使用 Fabric API。本 wiki 上的许多教程也会默认使用 Fabric API。
- 随着 fabric-loom（我们的Gradle构建插件）的开发和改动，有些时候你可能会遇上一些通过重置 Gradle 缓存才能解决的问题。使用 `gradlew cleanloom` 便能清理缓存，而 `gradlew --stop` 则能帮助你解决一些其他疑难杂症。
- 保持跟进到最新的 Loom 版本（Loom 版本是在 `build.gradle` 中定义的），以及 Fabric Loader 和 Fabric API 的版本（这是在 `build.gradle` 或者 `gradle.properties` 中定义的）。最新的版本可以在 [FabricMC](https://fabricmc.net/develop/) 中找到。即使是旧的 Minecraft 版本，最新的 Loom 和 Fabric Loader 也会支持。
- 保持跟进到最新的 Gradle 版本，这是在 `gradle/wrapper/gradle-wrapper.properties` 中定义的。最新的 Gradle 也可以用于开发旧的 Minecraft 版本。
- 如果你在为旧版本开发 Minecraft，除了修改 `gradle.properties` 外，你还需要修改 `build.gradle` 和 mixin 配置中更改 Java 兼容性版本。
- 如果你有任何问题，尽管问，社区就是用来干这件事的。