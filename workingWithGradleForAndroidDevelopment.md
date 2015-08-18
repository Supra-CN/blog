# working with Gradle for Android Development

**This is a summary version of the official [User Guide][1]**  
**这是一个官方[用户指南][1]的精简版本**  

[TOC]

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
The Android plugin provides a broad DSL to customize most things directly from the build system.  
Android插件提供了广范的DSL元素，用于直接定制构建系统中的大部分事物。  

#### Manifest entries | Manifest选项
Through the DSL it is possible to configure the following manifest entries:  
通过DSL组件，可以配置以下manifest选项  

- minSdkVersion
- targetSdkVersion
- versionCode
- versionName
- applicationId (the effective packageName -- see [ApplicationId versus PackageName][9] for more information)
- Package Name for the test application
- Instrumentation test runner

Example:

```groovy
android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"

    defaultConfig {
        versionCode 12
        versionName "2.0"
        minSdkVersion 16
        targetSdkVersion 16
    }
}
```

The `defaultConfig` element inside the `android` element is where all this configuration is defined.  
这些配置选项都在`android`元素中包含的`defaultConfig`元素中定义。  

Previous versions of the Android Plugin used *packageName* to configure the manifest 'packageName' attribute.  
Starting in 0.11.0, you should use *applicationId* in the build.gradle to configure the manifest 'packageName' entry.  
This was disambiguated to reduce confusion between the application's packageName (which is its ID) and java packages.  
先前的Android插件使用*packageName*去配置manifest中的'packageName'属性。  
但是在Gragle v0.11.0之后需要使用*applicationId*来配置manifest中的'packageName'属性。  
这是为了消除apk包名和java报名之间的歧义。  

The power of describing it in the build file is that it can be dynamic.  
For instance, one could be reading the version name from a file somewhere or using some custom logic:  
在构建脚本中配置这些属性的一个强大之处在于，他们可以是动态配置的；  
如下所示，他可以给版本名赋予一个定义在任意文件中的值，或者直接通过一段自定义的逻辑代码来生成值：  

```groovy
def computeVersionName() {
    ...
}

android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"

    defaultConfig {
        versionCode 12
        versionName computeVersionName()
        minSdkVersion 16
        targetSdkVersion 16
    }
}
```

**Note:** Do not use function names that could conflict with existing getters in the given scope. For instance instance `defaultConfig { ...}` calling `getVersionName()` will automatically use the getter of `defaultConfig.getVersionName()` instead of the custom method.  
**注意：** 方法名不要和给定作用范围内已有的getter名字冲突，举个例子，在`defaultConfig { ...}`的实例中调用`getVersionName()`就会自动掉`defaultConfig.getVersionName()`的getter方法，而不是自定义方法。  

If a property is not set through the DSL, some default value will be used. Here’s a table of how this is processed.  
如果某一属性没有通过DSL的方式明确声明，那么他们会按下列表格中的某认值处理。  

 Property Name             | Default value in DSL object | Default value
---------------------------|-----------------------------|---------------
 versionCode               | -1                          | value from manifest if present
 versionName               | null                        | value from manifest if present
 minSdkVersion             | -1                          | value from manifest if present
 targetSdkVersion          | -1                          | value from manifest if present
 applicationId             | null                        | value from manifest if present
 testApplicationId         | null                        | applicationId + “.test”
 testInstrumentationRunner | null                        | android.test.InstrumentationTestRunner
 signingConfig             | null                        | null
 proguardFile              | N/A (set only)              | N/A (set only)
 proguardFiles             | N/A (set only)              | N/A (set only)

The value of the 2nd column is important if you use custom logic in the build script that queries these properties. For instance, you could write:  
当需要在构建脚本中定制构建逻辑时，其中第二列的值是非常重要的，类似如下的写法：  
```groovy
if (android.defaultConfig.testInstrumentationRunner == null) {
    // assign a better default...
}
```

If the value remains null, then it is replaced at build time by the actual default from column 3, but the DSL element does not contain this default value so you can't query against it.  
This is to prevent parsing the manifest of the application unless it’s really needed.  
对于值为null的属性，它将在编译时才真正被赋上第三列中的默认值，所以不能从DSL元素中查询出他们的值。  
这是为了防止在不必要的情况下解析manifest。  

#### Build Types | 构建类型
By default, the Android plugin automatically sets up the project to build both a debug and a release version of the application.  
These differ mostly around the ability to debug the application on a secure (non dev) devices, and how the APK is signed.  
默认情况下，android插件会自动为工程配置两个构建类型，一个debug版，一个release版。  
两种构建类型的区别在于debug版可以在安全设备（非开发设备）上调试，另外他们的签名方式也不同。  

The debug version is signed with a key/certificate that is created automatically with a known name/password (to prevent required prompt during the build). The release is not signed during the build, this needs to happen after.  
debug版的会自动生成一个免密码的key用来签名（这是为了防止构建过程中出现输入密码的提示框）。release版在构建时不会被签名，他需要在稍后再签名。  

This configuration is done through an object called a `BuildType`. By default, 2 instances are created, a `debug` and a `release` one.  
这是通过一个叫`BuildType`的东西配置的，默认情况下会有两个实例被创建，一个`debug`和一个`release` 。  

The Android plugin allows customizing those two instances as well as creating other Build Types. This is done with the `buildTypes` DSL container:  
android插件允许自定义这两个构建类型，并且可以创建其他构建类型，这是在`buildTypes`这个DSL容器中完成的：  

```groovy
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }

        jnidebug.initWith(buildTypes.debug)
        jnidebug {
            packageNameSuffix ".jnidebug"
            jniDebuggable true
        }
    }
}
```
The above snippet achieves the following:  
上面的代码片段实现了以下内容：  

- Configures the default `debug` `Build` Type:  
	配置默认的`debug`构建类型：
	- set its package to be `<app appliationId>.debug` to be able to install both debug and release apk on the same device  
		将其包名设置为`<app appliationId>.debug`，以便在一个设备上同时安装正式版和测试版  
- Creates a new `BuildType` called `jnidebug` and configure it to be a copy of the `debug` build type.  
	创建了一个名为`jnidebug`的构建类型，并将其初始化为`debug`的副本。  
- Keep configuring the jnidebug, by enabling debug build of the JNI component, and add a different package suffix.  
	继续配置`jnidebug`，使调试JNI组件有效，并且为定义了一个不同的apk后缀。  
	
Creating new Build Types is as easy as using a new element under the `buildTypes` container, either to call `initWith()` or to configure it with a closure.  
创建一个新的构建类型就同在`buildTypes`容器下使用一个新的元素一样简单，直接调`initWith()`方法，或者直接在大括号里重新配置一个。  

The possible properties and their default values are:  
下面是一些可能用到的属性和他们的默认值：  

 Property name	        | Default values for debug     | Default values for release / other
 -----------------------|------------------------------|------------------------------------
 debuggable	            | true	                       | false
 jniDebuggable	        | false	                       | false
 renderscriptDebuggable | false	                       | false
 renderscriptOptimLevel	| 3	                           | 3
 applicationIdSuffix	| null	                       | null
 versionNameSuffix	    | null	                       | null
 signingConfig	        | android.signingConfigs.debug | null
 zipAlignEnabled	    | false	                       | true
 minifyEnabled	        | false	                       | false
 proguardFile	        | N/A (set only)	           | N/A (set only)
 proguardFiles	        | N/A (set only)	           | N/A (set only)

In addition to these properties, `Build Types` can contribute to the build with code and resources.  
For each `Build Type`, a new matching sourceSet is created, with a default location of  
除了上边这些属性之外，`Build Types`支持创建专有其的源码和资源。  
对于每个`Build Type`都会创建一个新的代码集，其默认位置在：  
	`src/<buildtypename>/`
This means the `Build Type` names cannot be `main` or `androidTest` (this is enforced by the plugin), and that they have to be unique to each other.  
这意味着`Build Type`的名字不能是`main`或`androidTest`（这是有android插件强制决定的），并且要保持唯一。  


Like any other source sets, the location of the build type source set can be relocated:  
同其他代码集一样，构建类型的代码集目录位置也可以被自定义。  

```groovy
android {
    sourceSets.jnidebug.setRoot('foo/jnidebug')
}
```
Additionally, for each Build Type, a new `assemble<BuildTypeName>` task is created.  
此外对于每种构建类型，都会相应的创建一个名为`assemble<BuildTypeName>`任务。  

The `assembleDebug` and `assembleRelease` tasks have already been mentioned, and this is where they come from. When the `debug` and `release` Build Types are pre-created, their tasks are automatically created as well.  
上面提到的`assembleDebug`和`assembleRelease`任务是从哪儿来的呢，他是在`debug`和`release`编译类型即将创建是自动生成的。  

The `build.gradle` snippet above would then also generate an `assembleJnidebug` task, and `assemble` would be made to depend on it the same way it depends on the `assembleDebug` and `assembleRelease` tasks.  
上述`build.gradle`中的片段将会生成一个`assembleJnidebug`任务，`assemble`会以`assembleDebug`和`assembleRelease` 相同的方式依赖于他。  

**Tip:** remember that you can type `gradle aJ` to run the `assembleJnidebug` task.  
**小贴士：** 别忘了，可以用`gradle aJ`来调用`assembleJnidebug` 任务。  

Possible use case:  
可能是应用场景：  

- Permissions in debug mode only, but not in release mode  
	发行版中没有，仅在调试模式下存在的权限  
- Custom implementation for debugging  
	为了调试而自定义的逻辑实现  
- Different resources for debug mode (for instance when a resource value is tied to the signing certificate).  
	调试模式下的特有资源（比如绑定到签名证书的资源）

The code/resources of the BuildType are used in the following way:  
构建类型的源码/资源将以下列方式使用：  

- The manifest is merged into the app manifest  
	manifest合并到app manifest  
- The code acts as just another source folder  
	只是存在与另一个代码目录的代码  
- The resources are overlayed over the main resources, replacing existing values.  
	一些凌驾于主要资源的资源，覆盖一些原先的值  

#### Signing Configurations | 配置签名
Signing an application requires the following:  
对一个应用签名的要求如下：  

- A keystore
- A keystore password
- A key alias name
- A key password
- The store type

The location, as well as the key name, both passwords and store type form together a Signing Configuration (type `SigningConfig`)  
签名的配置（`SigningConfig`）包括签名文件目录位置以及名称，两个密码  

By default, there is a debug configuration that is setup to use a debug keystore, with a known password and a default key with a known password.  
The debug keystore is located in `$HOME/.android/debug.keystore`, and is created if not present.  
默认的，debug配置会自动设置一个已知name和密码的keystore。  
他会在自动创建在`$HOME/.android/debug.keystore`目录中。

It is possible to create other configurations or customize the default built-in one. This is done through the `signingConfigs` DSL container:  
可以通过`signingConfigs`这个DSL容器配置或自定义默认的debug key：  

```groovy
android {
    signingConfigs {
        debug {
            storeFile file("debug.keystore")
        }

        myConfig {
            storeFile file("other.keystore")
            storePassword "android"
            keyAlias "androiddebugkey"
            keyPassword "android"
        }
    }

    buildTypes {
        foo {
            debuggable true
            jniDebuggable true
            signingConfig signingConfigs.myConfig
        }
    }
}
```
The above snippet changes the location of the debug keystore to be at the root of the project. This automatically impacts any Build Types that are set to using it, in this case the debug Build Type.  
上面的代码将keystore的位置定义为工程的跟目录，这将自动影响所有使用他的构建类型，上例中他将影响debug构建类型。  

It also creates a new Signing Config and a new Build Type that uses the new configuration.  
另外还创建了一个新的签名配置，并且将他应用到了一个新的构建类型上。  

**Note:** Only debug keystores located in the default location will be automatically created. Changing the location of the debug keystore will not create it on-demand. Creating a SigningConfig with a different name that uses the default debug keystore location will create it automatically. In other words, it’s tied to the location of the keystore, not the name of the configuration.  
**注意：** 只有debug keystore位于默认位置时他才会被自动创建，改变位置之后就不会自动创建了，只是创建另外一个签名配置兵使用另外的名字的话还是会自动创建的，换言之，他关心的只是keystore的所在位置，而不是配置和名字。  

**Note:** Location of keystores are usually relative to the root of the project, but could be absolute paths, thought it is not recommended (except for the debug one since it is automatically created).  
**注意：** keystores的位置通常是工程的根目录，也可以是一个绝对路径，但是不推荐这么做（除非是用于调试且自动创建的）。  

**Note:**  If you are checking these files into version control, you may not want the password in the file. The following Stack Overflow [post][10] shows ways to read the values from the console, or from environment variables.
We'll update this guide with more detailed information later.

**注意：** 如果要把这些文件加入版本控制管理，有可能不希望密码被明文写在文件中，那么Stack Overflow中的这篇[帖子][10]指明了如何从命令行或环境变量读取密码值的方法。  
我会在稍后更新这部分内容。  

#### Running ProGuard | 执行混淆
ProGuard is supported through the Gradle plugin for ProGuard version 4.10. The ProGuard plugin is applied automatically, and the tasks are created automatically if the Build Type is configured to run ProGuard through the `minifyEnabled` property.  
ProGuard v4.10 一插件的形式被支持。已经自动应用了混淆插件，并且在构建类型中通过设置`minifyEnabled`属性，自动创建任务并开启  

```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }

    productFlavors {
        flavor1 {
        }
        flavor2 {
            proguardFile 'some-other-rules.txt'
        }
    }
}
```
Variants use all the rules files declared in their build type, and product flavors.  
产品变种会应用通过在所有规则文件中声明的构建类型和产品风味中的配置。  

There are 2 default rules files  
以下是两个规则文件：

- proguard-android.txt
- proguard-android-optimize.txt

They are located in the SDK. Using `getDefaultProguardFile()` will return the full path to the files. They are identical except for enabling optimizations.  
他俩位于SDK中，可以通过`getDefaultProguardFile()`方法得到他俩的完整位置，他俩除了是否应用优化之外完全相同。  

#### Shrinking Resources | 压缩资源
You can also remove unused resources, automatically, at build time. For more information, see the [Resource Shrinking][11] document.  
可以在编译时自动删除没有使用的资源，详情参见文档[压缩资源][11]

## Dependencies, Android Libraries and Multi-project setup | 依赖关系，Android库和Multi-project配置
Gradle projects can have dependencies on other components. These components can be external binary packages, or other Gradle projects.  
Gradle工程有可能是依赖于其他组件的，这些组件可以是一个外部的二进制包，或者是另外一个Gradle工程  

### Dependencies on binary packages | 依赖于二进制包
#### Local packages | 本地包
To configure a dependency on an external library jar, you need to add a dependency on the `compile` configuration.  
可以通过配置`compile`属性来配置一个外部jar包。

```groovy
dependencies {
    compile files('libs/foo.jar')
}

android {
    ...
}
```

**Note:** the `dependencies` DSL element is part of the standard Gradle API and does not belong inside the `android` element.  
**注意：** `dependencies`DSL元素是Gradle标准API的一部分，而并非属于`android`元素。  

The compile configuration is used to compile the main application. Everything in it is added to the compilation classpath and also packaged in the final APK.  
There are other possible configurations to add dependencies to:  
该编译配置将会用在编译主应用时，他定义的所有东西都会加入编译的classpath，并且会被打进最终的包里。  
其他添加依赖关系的方式有如下几种：

- `compile`: main application
- `androidTestCompile`: test application
- `debugCompile`: debug Build Type
- `releaseCompile`: release Build Type.

Because it’s not possible to build an APK that does not have an associated Build Type, the APK is always configured with two (or more) configurations: `compile` and `<buildtype>Compile`.  
Creating a new Build Type automatically creates a new configuration based on its name.  
以为依赖库本身并不能构建出一个APK文件来，所以他们不支持划分编译类型，APK的编译类型永远只有这两大类： `compile` and `<buildtype>Compile`。  
创建一个新的构建类型将会基于构建类型名称自动的生成一个新的构建配置。  

This can be useful if the debug version needs to use a custom library (to report crashes for instance), while the release doesn’t, or if they rely on different versions of the same library.  
这在下边的情况下是很有用的，如调试版本需要用到一个特殊的库（比如崩溃报告），而发行版并不需要，或者他们依赖于同一个库的不同版本。  

#### Remote artifacts | 远端神器
Gradle supports pulling artifacts from Maven and Ivy repositories.  
Gradle支持采用Maven和Ivy自动拉去远端神器（就是指具体的依赖包）。

First the repository must be added to the list, and then the dependency must be declared in a way that Maven or Ivy declare their artifacts.  
首先，依赖包的库必须在他们的可用列表里，然后，需要按照Maven或Ivy的方式把他们声明出来。  

```groovy
repositories {
    mavenCentral()
}


dependencies {
    compile 'com.google.guava:guava:11.0.2'
}

android {
    ...
}
```

**Note:** `mavenCentral()` is a shortcut to specifying the URL of the repository. Gradle supports both remote and local repositories.  
**Note:** Gradle will follow all dependencies transitively. This means that if a dependency has dependencies of its own, those are pulled in as well.  
**注意：** `mavenCentral()`是指定中心仓库URL的快捷方式,Gradle同时支持远端和本地仓库。  
**注意：** Gradle遵循依赖关系的传递性，这意味着他可以管理依赖的依赖。  

For more information about setting up dependencies, read the Gradle user guide [here][13], and DSL documentation [here][14].  
更多依赖关系配置的咨询详见[Gradle用户指南][13]和[DSL文档][14]

#### Multi project setup | 多工程配置
// TODO:

### Library projects | 库工程
// TODO:

#### Creating a Library Project | 创建库工程
// TODO:

#### Differences between a Project and a Library Project | 普通工程和库工程的区别
// TODO:

#### Referencing a Library | 引用库
// TODO:

#### Library Publication | 发布库
// TODO:

## Testing | 测试
// TODO:

### Unit Testing | 单元测试
// TODO:

### Basics and Configuration | 基础和配置
// TODO:

### Running tests | 执行测试
// TODO:

### Testing Android Libraries | 测试Android库
// TODO:

### Test reports | 测试报告
// TODO:

#### Single projects | 单个工程
// TODO:

#### Multi-projects reports | 多个工程
// TODO:

### Lint support | Lint支持
// TODO:

## Build Variants | 构建变种版本
One goal of the new build system is to enable creating different versions of the same application.  
新构建系统的目标之一是具有为同一个应用构建多个不同的变种版本的能力。  

There are two main use cases:  
下面是几个重要的应用场景：  

1. Different versions of the same application. For instance, a free/demo version vs the “pro” paid application.  
	同一个应用的不同版本，例如一个免费/试用版本和一个付费专业版应用。  
2. Same application packaged differently for multi-apk in Google Play Store. See [Multiple APK Support][12] for more information.  
	同一个应用，需要打多个不同的包，支持Google Play商店的multi-apk功能，详情参见[Multiple APK Support][12]。  
3. A combination of 1. and 2. 
	1和2综合起来的情况。  

The goal was to be able to generate these different APKs from the same project, as opposed to using a single Library Projects and 2+ Application Projects.  
目标是使用同一个工程具备生成不同版本APK的能力，而非采用一个库工程外加几个普通应用工程的方式。  

### Product flavors | 产品风味
A product flavor defines a customized version of the application build by the project. A single project can have different flavors which change the generated application.  
产品风味定义了工程如何构建出一个定制版本，一个单独的工程可以生成多种不同版本的应用程序。  

This new concept is designed to help when the differences are very minimum. If the answer to “Is this the same application?” is yes, then this is probably the way to go over Library Projects.  
这个概念是为了帮助那些存在微小差异的产品需求而设计的，如果要问他们还算是同一个应用程序吗，那么答案是肯定的，大家可能因此而走过依赖库工程的老路。  

Product flavors are declared using a `productFlavors` DSL container:  
通过`productFlavors`DSL容器来声明产品风味  

```groovy
android {
    ....

    productFlavors {
        flavor1 {
            ...
        }

        flavor2 {
            ...
        }
    }
}
```
This creates two flavors, called `flavor1` and `flavor2`.  
**Note:** The name of the flavors cannot collide with existing Build Type names, or with the `androidTest` sourceSet.  
上面创建了两个产品风味，分别是`flavor1`和`flavor2`。  
**注意：** 产品风味名不能和构建类型以及`androidTest`代码集重名。

### Build Type + Product Flavor = Build Variant | 构建类型 + 产品风味 = 变种版本
As we have seen before, each Build Type generates a new APK.  
正如前所示，每个构建类型生成一个新的APK。  

Product Flavors do the same: the output of the project becomes all possible combinations of Build Types and, if applicable, Product Flavors.  
产品风味也一样：如果适用的话，工程将产生构建类型和产品风味的所有可能的组合。  

Each (Build Type, Product Flavor) combination is called a Build Variant.  
每个（构建类型，产品风味）的组合被称为一个构建变种。  

For instance, with the default `debug` and `release` Build Types, the above example generates four Build Variants:  
如，算上默认的`debug`和`release`两种构建类型和上面例子中的产品风味，将生成四个构建变种：  

- Flavor1 - debug
- Flavor1 - release
- Flavor2 - debug
- Flavor2 - release

Projects with no flavors still have Build Variants, but the single `default` flavor/config is used, nameless, making the list of variants similar to the list of Build Types.  
如果工程没有配置产品风味，他依然存在构建变种，只不过用了一个`default`的产品风味/配置，省略了名字，导致变种列表跟构建类型列表一致。  

### Product Flavor Configuration | 配置产品风味
Each flavors is configured with a closure:  
每个风味都配置在一个大括号中：  

```groovy
android {
    ...

    defaultConfig {
        minSdkVersion 8
        versionCode 10
    }

    productFlavors {
        flavor1 {
            packageName "com.example.flavor1"
            versionCode 20
        }

        flavor2 {
            packageName "com.example.flavor2"
            minSdkVersion 14
        }
    }
}
```
Note that the `android.productFlavors.*` objects are of type `ProductFlavor` which is the same type as the `android.defaultConfig` object. This means they share the same properties.  
注意`android.productFlavors.*`对象为`ProductFlavor`类型，与`android.defaultConfig`相同，这就意味着，他们内部有相同的配置项目。  

`defaultConfig` provides the base configuration for all flavors and each flavor can override any value. In the example above, the configurations end up being:  
`defaultConfig`提供了所以风味的默认配置，每个风味都可以覆盖重写任何配置选项，如下所示：  

- flavor1
	- packageName: com.example.flavor1
	- minSdkVersion: 8
	- versionCode: 20
- flavor2
	- packageName: com.example.flavor2
	- minSdkVersion: 14
	- versionCode: 10

Usually, the Build Type configuration is an overlay over the other configuration. For instance, the Build Type's `packageNameSuffix` is appended to the Product Flavor's packageName.  
通常构建类型中的设置优先级较高，会重写其他配置项。例如，构建类型的`packageNameSuffix`会加在产品风味的报名之后。  

There are cases where a setting is settable on both the Build Type and the Product Flavor. In this case, it’s is on a case by case basis.  
下列设置都可以在构建类型和产品风味中生效。个别需要视情况下而定，  

For instance, signingConfig is one of these properties.  
This enables either having all release packages share the same `SigningConfig`, by setting `android.buildTypes.release.signingConfig`, or have each release package use their own `SigningConfig` by setting each `android.productFlavors.*.signingConfig` objects separately.  
例如，签名设置就是其中之一。  
这使得签名既可以让所有发行版都共享`android.buildTypes.release.signingConfig`中相同的`SigningConfig`，又可以通过`android.productFlavors.*.signingConfig`为个别产品风味的发行版配置特有的`SigningConfig`。  

### Sourcesets and Dependencies | 代码集与依赖关系
Similar to Build Types, Product Flavors also contribute code and resources through their own sourceSets.  
类似与构建类型，产品风味也可以在他们自己的源码集中提价代码。  

The above example creates four sourceSets:  
上面的例子产生了四个代码集：  

- android.sourceSets.flavor1  
	Location src/flavor1/  
- android.sourceSets.flavor2  
	Location src/flavor2/  
- android.sourceSets.androidTestFlavor1  
	Location src/androidTestFlavor1/  
- android.sourceSets.androidTestFlavor2  
	Location src/androidTestFlavor2/  

Those sourceSets are used to build the APK, alongside `android.sourceSets.main` and the Build Type sourceSet.  
这些代码集可以佐以主代码集`android.sourceSets.main`和构建类型代码集来构建APK。  

The following rules are used when dealing with all the sourcesets used to build a single APK:  
用于构建APK的所有代码集的处理规则如下：  

- All source code (`src/*/java`) are used together as multiple folders generating a single output.
	在多个目录下的所有代码（`src/*/java`）一同被用于产生一个构建。
- Manifests are all merged together into a single manifest. This allows Product Flavors to have different components and/or permissions, similarly to Build Types.
	类似与构建类型，所有需要的manifest文件被合并成一个文件，这使得不同风味的产品可以有不同的manifest文件包含不同的组件和权限。
- All resources (Android res and assets) are used using overlay priority where the Build Type overrides the Product Flavor, which overrides the main sourceSet.
	一切需要的资源（Android资源和assets）按后面的优先级规则重写覆盖，构建类型代码覆盖产品风味代码覆盖主代码。
- Each Build Variant generates its own R class (or other generated source code) from the resources. Nothing is shared between variants.
	每个变种都会从资源文件中生成他自己的独立的R文件（或其他自动生成的文件），没有任何东西是共享的。

Finally, like Build Types, Product Flavors can have their own dependencies. For instance, if the flavors are used to generate a ads-based app and a paid app, one of the flavors could have a dependency on an Ads SDK, while the other does not.  
最后，类似与构建类型，产品风味可以有自己的依赖关系，例如，产品风味被用与生成一个有广告的和一个付费的app，那么其中一个可能会依赖一个广告SDK，而另一个则不会。  

```groovy
dependencies {
    flavor1Compile "..."
}
```
In this particular case, the file `src/flavor1/AndroidManifest.xml` would probably need to include the internet permission.  
这种情况下，`src/flavor1/AndroidManifest.xml`可能需要声明一个需要网络的权限。  

Additional sourcesets are also created for each variants:  
每个变种将会创建额外的代码集：  

- android.sourceSets.flavor1Debug  
	Location src/flavor1Debug/  
- android.sourceSets.flavor1Release  
	Location src/flavor1Release/  
- android.sourceSets.flavor2Debug  
	Location src/flavor2Debug/  
- android.sourceSets.flavor2Release  
	Location src/flavor2Release/  

These have higher priority than the build type sourcesets, and allow customization at the variant level.  
这些代码集的优先级高于构建类型代码集，并且在变种岑面上允许做一些自定义的工作。  

### Building and Tasks | 构建与任务
We previously saw that each Build Type creates its own `assemble<name>` task, but that Build Variants are a combination of Build Type and Product Flavor.  
之前我们看到根据每个构建类型会创建名外`assemble<name>`的任务，而构建变种则需要组合构建类型和产品风味。  

When Product Flavors are used, more assemble-type tasks are created. These are:  
当引用了产品风味后，更多的构建任务会被创建，如下：  

1. `assemble<Variant Name>`  
	allows directly building a single variant. For instance `assembleFlavor1Debug`.  
	允许直接构建一个变种，例如`assembleFlavor1Debug`。  
2. `assemble<Build Type Name>`  
	allows building all APKs for a given Build Type. For instance `assembleDebug` will build both `Flavor1Debug` and `Flavor2Debug` variants.  
	允许构建给定类型的所有APK，例如`assembleDebug`会生成`Flavor1Debug`和`Flavor2Debug`两个变种。  
3. `assemble<Product Flavor Name>`  
	allows building all APKs for a given flavor. For instance `assembleFlavor1` will build both `Flavor1Debug` and `Flavor1Release` variants.  
	允许构建给定风味的APK，例如`assembleFlavor1`会生成`Flavor1Debug`和`Flavor1Release`两个变种。  

The task `assemble` will build all possible variants.  
`assemble`任务会构建所有可能的产品变种。  

### Testing | 测试
// TODO:

### Multi-flavor variants | 多种风味的变种版本
In some case, one may want to create several versions of the same apps based on more than one criteria.  
For instance, multi-apk support in Google Play supports 4 different filters. Creating different APKs split on each filter requires being able to use more than one dimension of Product Flavors.  
某些情况下，可能会需要使用相同的产品为不同环境构建不同APK。  
例如在Google Play中多apk支持4中不同的过滤项。为每个不同的选项分别生成不同的APK，可以避免在一个APK中存在多份尺度资源的情况。  

Consider the example of a game that has a demo and a paid version and wants to use the ABI filter in the multi-apk support. With 3 ABIs and two versions of the application, 6 APKs needs to be generated (not counting the variants introduced by the different Build Types).  
However, the code of the paid version is the same for all three ABIs, so creating simply 6 flavors is not the way to go.  
Instead, there are two dimensions of flavors, and variants should automatically build all possible combinations.  
试想一下，例如一个游戏，需要分试玩版和免费版，又希望采用multi-apk支持ABI过滤出不同的安装包，那么这个应用就需要生成3个ABI和两个风味中共6个APK（这还不算构建类型）。  
然而付费版本的几个ABI版本代码完全一样，所以简单的创建6个产品风味没有意义。  
取而代之的是Gradle可以自动将所有可能的尺度组合都构建出来。  

This feature is implemented using Flavor Dimensions. Flavors are assigned to a specific dimension  
下面是使用风味尺度的例子，将风味应用于尺度  
```groovy
android {
    ...

    flavorDimensions "abi", "version"

    productFlavors {
        freeapp {
            flavorDimension "version"
            ...
        }

        x86 {
            flavorDimension "abi"
            ...
        }
    }
}
```

The `android.flavorDimensions` array defines the possible dimensions, as well as the order. Each defined Product Flavor is assigned to a dimension.  
`android.flavorDimensions`定义了一列可能的尺度以及其顺序，每个产品口味都会按尺度分配。  

From the following dimensioned Product Flavors [freeapp, paidapp] and [x86, arm, mips] and the [debug, release] Build Types, the following build variants will be created:  
根据后面的口味尺度[freeapp, paidapp]和[x86, arm, mips]和[debug, release]编译类型，可以为一个风味构建出以下不同的尺度版本。  

- x86-freeapp-debug
- x86-freeapp-release
- arm-freeapp-debug
- arm-freeapp-release
- mips-freeapp-debug
- mips-freeapp-release
- x86-paidapp-debug
- x86-paidapp-release
- arm-paidapp-debug
- arm-paidapp-release
- mips-paidapp-debug
- mips-paidapp-release

The order of the dimension as defined by `android.flavorDimensions` is very important.  
`android.flavorDimensions`中定义的顺序是非常重要的。  

Each variant is configured by several Product Flavor objects:  
每个变种均配置为几个产品风味对象：  

- android.defaultConfig
- One from the abi dimension
- One from the version dimension

The order of the dimension drives which flavor override the other, which is important for resources when a value in a flavor replaces a value defined in a lower priority flavor.  
The flavor dimension is defined with higher priority first. So in this case:  
尺度的顺序决定了覆盖关系，这对资源文件来说的很重要的，高优先级风味版本中的值会覆盖地低先级版本的。  
定义越靠前的优先级越高，在上面的例子中优先级关系为：

	`abi > version > defaultConfig`

Multi-flavors projects also have additional sourcesets, similar to the variant sourcesets but without the build type:  
多风味的工程可以有附加的代码集，类似于变种代码集，但是没有构建类型：  

- android.sourceSets.x86Freeapp
	Location src/x86Freeapp/
- android.sourceSets.armPaidapp
	Location src/armPaidapp/
- etc...

These allow customization at the flavor-combination level. They have higher priority than the basic flavor sourcesets, but lower priority than the build type sourcesets.  
这允许了定制风味组合的优先级层次。他们的优先级高于基本风味的代码集，却低于构建类型的代码集。  

## Advanced Build Customization | 高级自定义构建
// TODO:

### Build options | 构建选项
// TODO:

#### Java Compilation options | java编译选项
// TODO:

#### aapt options | aapt选项
// TODO:

#### dex options | dex选项
// TODO:

### Manipulating tasks | 操控任务
// TODO:

### BuildType and Product Flavor property reference | 构建类型和产品风味的参数参考
// TODO:

### Using sourceCompatibility 1.7 | 使源码兼容java 1.7
// TODO:






* * *


[1]: http://tools.android.com/tech-docs/new-build-system/user-guide                                     "Gradle Plugin User Guide"
[2]: https://en.wikipedia.org/wiki/Domain-specific_language                                             "wiki of DSL"
[3]: http://www.groovy-lang.org/                                                                        "groovy"
[4]: https://maven.apache.org/                                                                          "maven"
[5]: http://developer.android.com/google/play/publishing/multiple-apks.html                             "Maven Repository Centre"
[6]: http://maven.apache.org/repository/index.html                                                      "The Central Repository"
[7]: http://maven.apache.org/ref/3.2.5/maven-artifact/                                                  "Maven artifact"
[8]: https://docs.gradle.org/current/userguide/java_plugin.html                                         "The Java Plugin"
[9]: http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename                   "ApplicationId versus PackageName"
[10]: http://stackoverflow.com/questions/18328730/how-to-create-a-release-signed-apk-file-using-gradle  "How to create a release signed apk file using Gradle?"
[11]: http://tools.android.com/tech-docs/new-build-system/resource-shrinking                            "Resource Shrinking"
[12]: http://developer.android.com/google/play/publishing/multiple-apks.html                            "Multiple APK Support"
[13]: http://gradle.org/docs/current/userguide/artifact_dependencies_tutorial.html                      "Dependency Management Basics"
[14]: http://gradle.org/docs/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html            "DependencyHandler"