# working with Gradle for Android Development

#### This is a summary version of the official [User Guide][1] <br /> 这是一个官方[用户指南][1]的精简版本

## Introduction | 介绍
Gradle is an advanced build system as well as an advanced build toolkit allowing to create custom build logic through plugins.  
Gradle是一套先进的编译体系以及工具包，可以通过各种插件创建自定义编译构建逻辑。

### Goals of Gradle | 目标
- Make it easy to reuse code and resources;  
	简化代码和资源的重用

- Make it easy to create several variants of an application,  either for [multi-apk][5] distribution or for different flavors of an application; (Such as : pad , lite , free , full , main etc.）  
	简化为一个项目创建多个变种版本的工作，包括支持[multi-apk][5]打包，和维护一个应用的多个不同风味的发行版（比如：pad，lite，free，full，main等）

- Make it easy to configure, extend and customize the build process  
	简化自定义和拓展编译构建过程的工作

- Good IDE integration  
	优良的IDE整合支持

### Why Gradle? | 何得何能？
- [Domain Specific Language (DSL)][2] to describe and manipulate the build logic  
	使用[领域特定语言 (DSL)][2]来表述和操控编译构建逻辑

- Build files are [Groovy][3] based and allow mixing of declarative elements through the DSL and using code to manipulate the DSL elements to provide custom logic.  
	编译脚本文件基于 [Groovy][3]，并且允许采用DSL和代码混合使用的方式来操控DSL元素，从而自定义构建过程

- Built-in dependency management through [Maven][4] and/or Ivy  
	可以选择采用[Maven][4]或Ivy来实现内置的工程包依赖管理系统

- Very flexible. Allows using best practices but doesn’t force its own way of doing things.  
	相当灵活，允许采用最佳的实践方案，但不会强制拘束任何实践细节

- Plugins can expose their own DSL and their own API for build files to use.  
	插件可以暴露一部分DSL元素和API一共编译脚本文件使用

- Good Tooling API allowing IDE integration  
	提供了一套优良的工具API以供IDE集成使用

## Requirements | 系统要求
- Gradle 1.10 or 1.11 or 1.12 with the plugin 0.11.1
- SDK with Build Tools 19.0.0. Some features may require a more recent version.

## Basic Project | 基础工程介绍
A Gradle project describes its build in a file called build.gradle located in the root folder of the project.  
Gradle工程在其工程根目录有一个名为build.gradle的文件，用来描述该工程的编译构建过程

### Simple build files | 简易构建脚本
The most simple Android project has the following build.gradle;   
下边是一个最简单的android工程的build.gradle构建脚本
``` groovy
buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:0.11.1'
    }
}

apply plugin: 'android'

android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"
}
```
There are 3 main areas to this Android build file:  
下面是这个android构建脚本的3个基本元素  

#### apply plugin
The **plugin** provides everything to build and test a project. for android project, You should only apply the android plugin. Applying the java plugin as well will result in a build error.  
**插件**为一个工程的构建和测试提供了一切所需，对于android工程，你需要指定应用android插件，如果指定应用java插件的话就会出编译问题；

- `apply plugin: 'java'`  
	packaged with Gradle; This applies the Java plugin, provides everything to build and test Java applications.  
	Gradle内建该插件，适用于构建和测试java工程的一切所需

- `apply plugin: 'android'`  
	packaged with Gradle since v0.11.1; Like the Java plugin, provides everything to build and test android applications.  
	Gradle 从v0.11.1起内建该插件，类似java插件，提供了构建和测试android工程的一切所需

#### buildscript
`buildscript { ... }` configures the code driving the build.    
In this case, this declares that it uses the [Maven Central repository][6], and that there is a classpath dependency on a [Maven artifact][7]. This artifact is the library that contains the Android plugin for Gradle in version 0.11.1
Note: This only affects the code running the build, not the project. The project itself needs to declare its own repositories and dependencies. This will be covered later.   
`buildscript { ... }`用于配置构建脚本如何驱动构建过程。  
上面的例子中，他描述需要使用[Maven Central repository][6]，并且有一个[Maven artifact][7]的classpath依赖关系，这神器是Gradle v0.11.1中包含安卓插件的一个库；  
注意：这些配置仅仅影响Gradle构建系统本身的执行，跟具体的project没有关系，具体的project需要具体定义他自己的repositories和dependencies，下面将会介绍这些内容。  

#### android
`android { ... }` configures all the parameters for the android build. This is the entry point for the Android DSL.
By default, only the compilation target, and the version of the build-tools are needed. This is done with the `compileSdkVersion` and `buildtoolsVersion` properties.  
The compilation target is the same as the target property in the project.properties file of the old build system. This new property can either be assigned a int (the api level) or a string with the same value as the previous target property.  
`android { ... }`用于所有与Android构建相关的参数，这是Android DSL元素的切入点；  
默认情况下只有编译目标和编译版本是必须的，可以通过`compileSdkVersion`和`buildtoolsVersion`为其赋值。  
这里的编译目标等同于先前老编译系统的project.properties文件中`target`属性；  

**Note:** You will also need a local.properties file to set the location of the SDK in the same way that the existing SDK requires, using the `sdk.dir` property.  
Alternatively, you can set an environment variable called `ANDROID_HOME`. There is no differences between the two methods, you can use the one you prefer.  
**注意：** 仍需在local.properties文件中定义`sdk.dir`属性以指明SDK的位置，或者设置一个名为`ANDROID_HOME`的环境变量，两种方法效果等同，酌情使用。

### Project Structure | 工程结构
The basic build files above expect a default folder structure. Gradle follows the concept of convention over configuration, providing sensible default option values when possible.  
基础构建脚本指明了一个默认的工程目录结构，Gradle支持约定优于配置的观点，在可能的情况下提供了合理的默认选项值  

The basic project starts with two components called “source sets”. The main source code and the test code. These live respectively in:  
基础工程有两个称之为“代码集”的组件。一个用来存放主代码，另一个用来存放测试用例；如下所示：  

- `src/main/`
- `src/androidTest/`

Inside each of these folders exists folder for each source components.  
For both the Java and Android plugin, the location of the Java source code and the Java resources:  
目录下是各自对应的代码组件。对于java和android工程来讲，代码包括java源码和资源文件：  

- `java/`
- `resources/`

For the Android plugin, extra files and folders specific to Android:  
余下这些是目录Android工程所特有的：  
 
- `AndroidManifest.xml`
- `res/`
- `assets/`
- `aidl/`
- `rs/`
- `jni/`
- `jniLibs/`

**Note:** `src/androidTest/AndroidManifest.xml` is not needed as it is created automatically.  
**注意：** `src/androidTest/AndroidManifest.xml`无需干预，会自动创建。

#### Configuring the Structure | 配置工程目录
When the default project structure isn’t adequate, it is possible to configure it. According to the Gradle documentation, reconfiguring the `sourceSets` for a Java project can be done with the following:  
如果默认配置不能满足需求，也可以自定义配置选项，根据Gradle的文档，可参照如下方式为java工程重新配置`sourceSets`  
```groovy
sourceSets {
    main {
        java {
            srcDir 'src/java'
        }
        resources {
            srcDir 'src/resources'
        }
    }
}
```
**Note:** `srcDir` will actually add the given folder to the existing list of source folders (this is not mentioned in the Gradle documentation but this is actually the behavior).  
**注意：** 实际上`srcDir`会被添加到已有的源码目录列表中（虽然Gradle文档为提及，但确是如此）  

To replace the default source folders, you will want to use `srcDirs` instead, which takes an array of path. This also shows a different way of using the objects involved:  
如需使用新的源码目录彻底替换原有的`srcDirs`，需要使用一个目录列表，如下所示为另外一种赋值方式：  
```groovy
sourceSets {
    main.java.srcDirs = ['src/java']
    main.resources.srcDirs = ['src/resources']
}
```
For more information, see the Gradle documentation on the Java plugin [here][8].
Gradle java plugin官方文档参见[The Java Plugin][8]  

The Android plugin uses a similar syntaxes, but because it uses its own `sourceSets`, this is done within the `android` object.    
Here’s an example, using the old project structure for the main code and remapping the `androidTest` `sourceSet` to the tests folder:  
对应的，android工程有类似的语法，由于使用使用了他自己的`sourceSets`，所以要定义在`android`组件中。
下面是实现了android老工程结构的一个例子：
```groovy
android {
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        androidTest.setRoot('tests')
    }
}
```
**Note:** because the old structure put all source files (java, aidl, renderscript, and java resources) in the same folder, we need to remap all those new components of the `sourceSet` to the same `src` folder.  
**注意：** 由于老的android工程结构把所有源码文件（包括java, aidl, renderscript, and java resources）都放在一个目录中了，所以我们需要把所有新定义的`sourceSets`统统映射到`src`目录下。

**Note:** `setRoot()` moves the whole `sourceSet` (and its sub folders) to a new folder. This moves `src/androidTest/*` to `tests/* `  
**注意：** `setRoot()`会将整个`sourceSet`重新映射到一个新的目录，包括他的所有子目录，上边例子把`src/androidTest/*`映射到了`tests/* `  

This is Android specific and will not work on Java `sourceSets`.  
该定制脚本尽适用于Android而不适用与java。

The ‘migrated’ sample shows this.    
该脚本可以应用于老工程的迁移。

### Build Tasks | 构建任务

#### General Tasks | 常规任务
Applying a plugin to the build file automatically creates a set of build tasks to run. Both the Java plugin and the Android plugin do this.  
The convention for tasks is the following:  
应用一个插件可以自动化的创建并运行一系列构建任务，java和android工程亦如此；  
任务约定如下所示：

- **assemble**  
	The task to assemble the output(s) of the project  
	编译产出的中间文件的打包组建任务  
	
- **check**  
	The task to run all the checks.  
	执行一切检查任务  
	
- **build**  
	This task does both `assemble` and `check`  
	执行`assemble`和`check` 
    
- **clean**  
	This task cleans the output of the project  
	清除编译产出的文件  
    
The tasks `assemble`, `check` and `build` don’t actually do anything. They are anchor tasks for the plugins to add actual tasks that do the work.  
实际上默认情况下`assemble`, `check` 和 `build`  并没有实际执行，他提供了一个类似回调接口的机制，以供配置执行各种插件的各种任务。

This allows you to always call the same task(s) no matter what the type of project is, or what plugins are applied.  
For instance, applying the `findbugs` plugin will create a new task and make `check` depend on it, making it be called whenever the `check` task is called  
它提供了一个执行全局任务的机会，无论构建任何类型的任何project统统都会执行的任务。  
举个例子，比如我们希望对全局都使用`findbugs`插件，就可以在此配置，执行`check`任务时执行；  

list tasks from the command line：
命令行下查看任务：  

- `gradle tasks`  
	get the high level task running.  
	查看上层运行的tasks。

- `gradle tasks --all`  
	full list and seeing dependencies between the tasks run.  
	查看包括低层依赖在内的所有可执行的task：  

Note: Gradle automatically monitor the declared inputs and outputs of a task.  
Running the `build` twice without change will make Gradle report all tasks as `UP-TO-DATE`, meaning no work was required. This allows tasks to properly depend on each other without requiring unneeded build operations.  
**注意：** Gradle会自动监听某个任务所声明的输入和输出变化。  
连续执行两次`build`操作会提示所有任务都`UP-TO-DATE`，这意味着没有新工作需要处理，说明工程依赖关系是正确的。  

#### Java project tasks | java工程任务
The Java plugin creates mainly two tasks, that are dependencies of the main anchor tasks:  
java插件主要创建两个任务，这依赖于主锚任务  

- assemble
    - jar  
    	This task creates the output.  
    	这个任务生成产出  
    
- check  
    - test  
    	This task runs the tests.  
    	这个任务执行测试  
    
The `jar` task itself will depend directly and indirectly on other tasks: `classes` for instance will compile the Java code.  
The tests are compiled with `testClasses`, but it is rarely useful to call this as `test` depends on it (as well as `classes`).  
`jar`任务本身是直接或间接的依赖于其他任务的：其中`classes` 任务是一个用来编译java源码的实例.    
`testClasses`任务用来编译测试用例，由于`test`任务依赖于他所以很少被直接调用，（`classes`任务也是如此）  

In general, you will probably only ever call `assemble` or `check`, and ignore the other tasks.   
通常，我们只需要直接调用`assemble`或`check`任务就好了，其他任务可以忽略。

You can see the full set of tasks and their descriptions for the Java plugin [here][8].  
完整任务表述参见官方文档[The Java Plugin][8]。   

#### Android tasks | Android任务
The Android plugin use the same convention to stay compatible with other plugins, and adds an additional anchor task:  
android插件兼容并使用同其他插件相同的约定，并且额外新增了几个锚点任务:  

- **assemble**
	The task to assemble the output(s) of the project  
	组装各种产出文件的任务。
	
- **check**
	The task to run all the checks.  
	执行各种检查的任务
	
- **connectedCheck**
	Runs checks that requires a connected device or emulator. they will run on all connected devices in parallel.  
	并行执行检查设备链接的任务，包括模拟器和真机。
	
- **deviceCheck**
	Runs checks using APIs to connect to remote devices. This is used on CI servers.  
	执行使用APIs链接远程设备的检查，用于持续集成。
	
- **build**
	This task does both assemble and check
	执行组装和检查任务。
	
- **clean**
	This task cleans the output of the project
    执行清理产出的任务。

The new anchor tasks are necessary in order to be able to run regular checks without needing a connected device.  
Note that `build` does not depend on `deviceCheck`, or `connectedCheck`.  
新创建的锚点任务必须支持没有可用链接设备的情况。
需要注意的是，`build`任务并不依赖于`deviceCheck`或`connectedCheck`。

An Android project has at least two outputs: a debug APK and a release APK. Each of these has its own anchor task to facilitate building them separately:  
对于Android工程而言，至少会有两个产出：一个排错版APK和一个发行版APK，对于其中的每一个都会有对应的锚点任务以便分别构建：
- assemble
	- assembleDebug
	- assembleRelease

They both depend on other tasks that execute the multiple steps needed to build an APK. The `assemble` task depends on both, so calling it will build both APKs.  
上述两个的构建过程都依赖于多个其他任务步骤的执行，`assemble`任务又依赖于他们两个，所以执行结果会构建产生两个APK文件。

**Tip:** Gradle support camel case shortcuts for task names on the command line. For instance: `gradle aR` is the same as typing `gradle assembleRelease` as long as no other task match `aR`.
**小贴士** Gradle支持驼峰式写法的缩写，举个例子：在没有其他相同匹配的情况下`gradle aR`等同于 `gradle assembleRelease`。

The check anchor tasks have their own dependencies:  
检查锚点任务有他们自己的依赖关系：
- check
	- lint
- connectedCheck
	- connectedAndroidTest
	- connectedUiAutomatorTest (not implemented yet)
- deviceCheck
	- This depends on tasks created when other plugins implement test extension points.  

Finally, the plugin creates install/uninstall tasks for all build types (`debug`, `release`, `test`), as long as they can be installed (which requires signing).  
最终插件会针对所有构建类型 (`debug`, `release`, `test`)创建安装/卸载任务，当然，只有经过签名之后才能被安装。

### Basic Build Customization | 自定义构建过程基础

[1]: http://tools.android.com/tech-docs/new-build-system/user-guide        "Gradle Plugin User Guide"
[2]: https://en.wikipedia.org/wiki/Domain-specific_language                                    "wiki of DSL"
[3]: http://www.groovy-lang.org/                                                                                   "groovy"
[4]: https://maven.apache.org/                                                                                      "maven"
[5]: http://developer.android.com/google/play/publishing/multiple-apks.html      "Maven Repository Centre"
[6]: http://maven.apache.org/repository/index.html                                                   "The Central Repository"
[7]: http://maven.apache.org/ref/3.2.5/maven-artifact/                                              "Maven artifact"
[8]: https://docs.gradle.org/current/userguide/java_plugin.html                               "The Java Plugin"