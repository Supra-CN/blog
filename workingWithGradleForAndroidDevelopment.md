# working with Gradle for Android Development

#### 这是一个官方[用户指南][1]的精简版本

## Introduction | 介绍
Gradle是一套先进的编译体系以及工具包，可以通过各种插件创建自定义编译构建逻辑。

### Goals of Gradle | 目标

- 简化代码和资源的重用
- 简化为一个项目创建多个变种版本的工作，包括支持[multi-apk][5]打包，和维护一个应用的多个不同风味的发行版（比如：pad，lite，free，full，main等）
- 简化自定义和拓展编译构建过程的工作
- 优良的IDE整合支持

### Why Gradle? | 何得何能？

- 使用[领域特定语言 (DSL)][2]来表述和操控编译构建逻辑
- 编译脚本文件基于 [Groovy][3]，并且允许采用DSL和代码混合使用的方式来操控DSL元素，从而自定义构建过程
- 可以选择采用[Maven][4]或Ivy来实现内置的工程包依赖管理系统
- 相当灵活，允许采用最佳的实践方案，但不会强制拘束任何实践细节
- 插件可以暴露一部分DSL元素和API一共编译脚本文件使用
- 提供了一套优良的工具API以供IDE集成使用

## Requirements | 系统要求

- Gradle 1.10 or 1.11 or 1.12 with the plugin 0.11.1
- SDK with Build Tools 19.0.0. Some features may require a more recent version.

## Basic Project | 基础工程介绍
Gradle工程在其工程根目录有一个名为build.gradle的文件，用来描述该工程的编译构建过程

### Simple build files | 简易构建脚本
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
下面是这个android构建脚本的3个基本元素  

#### apply plugin
**插件**为一个工程的构建和测试提供了一切所需，对于android工程，你需要指定应用android插件，如果指定应用java插件的话就会出编译问题；

- `apply plugin: 'java'`  
	Gradle内建该插件，适用于构建和测试java工程的一切所需
- `apply plugin: 'android'`  
	Gradle 从v0.11.1起内建该插件，类似java插件，提供了构建和测试android工程的一切所需

#### buildscript
`buildscript { ... }`用于配置构建脚本如何驱动构建过程。  
上面的例子中，他描述需要使用[Maven Central repository][6]，并且有一个[Maven artifact][7]的classpath依赖关系，这神器是Gradle v0.11.1中包含安卓插件的一个库；  
注意：这些配置仅仅影响Gradle构建系统本身的执行，跟具体的project没有关系，具体的project需要具体定义他自己的repositories和dependencies，下面将会介绍这些内容。  

#### android
`android { ... }`用于所有与Android构建相关的参数，这是Android DSL元素的切入点；  
默认情况下只有编译目标和编译版本是必须的，可以通过`compileSdkVersion`和`buildtoolsVersion`为其赋值。  
这里的编译目标等同于先前老编译系统的project.properties文件中`target`属性；  

**注意：** 仍需在local.properties文件中定义`sdk.dir`属性以指明SDK的位置，或者设置一个名为`ANDROID_HOME`的环境变量，两种方法效果等同，酌情使用。

### Project Structure | 工程结构
基础构建脚本指明了一个默认的工程目录结构，Gradle支持约定优于配置的观点，在可能的情况下提供了合理的默认选项值  

基础工程有两个称之为“代码集”的组件。一个用来存放主代码，另一个用来存放测试用例；如下所示：

- `src/main/`
- `src/androidTest/`

目录下是各自对应的代码组件。对于java和android工程来讲，代码包括java源码和资源文件：

- `java/`
- `resources/`

余下这些是目录Android工程所特有的：

- `AndroidManifest.xml`
- `res/`
- `assets/`
- `aidl/`
- `rs/`
- `jni/`
- `jniLibs/`

**注意：** `src/androidTest/AndroidManifest.xml`无需干预，会自动创建。

#### Configuring the Structure | 配置工程目录
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
**注意：** 实际上`srcDir`会被添加到已有的源码目录列表中（虽然Gradle文档为提及，但确是如此）  

如需使用新的源码目录彻底替换原有的`srcDirs`，需要使用一个目录列表，如下所示为另外一种赋值方式：  
```groovy
sourceSets {
    main.java.srcDirs = ['src/java']
    main.resources.srcDirs = ['src/resources']
}
```
Gradle java plugin官方文档参见[The Java Plugin][8]  

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
**注意：** 由于老的android工程结构把所有源码文件（包括java, aidl, renderscript, and java resources）都放在一个目录中了，所以我们需要把所有新定义的`sourceSets`统统映射到`src`目录下。

**注意：** `setRoot()`会将整个`sourceSet`重新映射到一个新的目录，包括他的所有子目录，上边例子把`src/androidTest/*`映射到了`tests/* `  

该定制脚本尽适用于Android而不适用与java。

该脚本可以应用于老工程的迁移。

### Build Tasks | 构建任务

#### General Tasks | 常规任务
应用一个插件可以自动化的创建并运行一系列构建任务，java和android工程亦如此；  
任务约定如下所示：

- **assemble**  
	编译产出的中间文件的打包组建任务  
- **check**  
	执行一切检查任务  
- **build**  
	执行`assemble`和`check` 
- **clean**  
	清除编译产出的文件  
    
实际上默认情况下`assemble`, `check` 和 `build`  并没有实际执行，他提供了一个类似回调接口的机制，以供配置执行各种插件的各种任务。

它提供了一个执行全局任务的机会，无论构建任何类型的任何project统统都会执行的任务。  
举个例子，比如我们希望对全局都使用`findbugs`插件，就可以在此配置，执行`check`任务时执行；  

命令行下查看任务：  

- `gradle tasks`  
	查看上层运行的tasks。
- `gradle tasks --all`  
	查看包括低层依赖在内的所有可执行的task：  

**注意：** Gradle会自动监听某个任务所声明的输入和输出变化。  
连续执行两次`build`操作会提示所有任务都`UP-TO-DATE`，这意味着没有新工作需要处理，说明工程依赖关系是正确的。  

#### Java project tasks | java工程任务
java插件主要创建两个任务，这依赖于主锚任务  

- assemble
    - jar  
    	这个任务生成产出  
- check  
    - test  
    	这个任务执行测试  
    
`jar`任务本身是直接或间接的依赖于其他任务的：其中`classes` 任务是一个用来编译java源码的实例.    
`testClasses`任务用来编译测试用例，由于`test`任务依赖于他所以很少被直接调用，（`classes`任务也是如此）  

通常，我们只需要直接调用`assemble`或`check`任务就好了，其他任务可以忽略。  

完整任务表述参见官方文档[The Java Plugin][8]。   

#### Android tasks | Android任务
android插件兼容并使用同其他插件相同的约定，并且额外新增了几个锚点任务:  

- **assemble**
	组装各种产出文件的任务。
- **check**
	执行各种检查的任务
- **connectedCheck**
	并行执行检查设备链接的任务，包括模拟器和真机。
- **deviceCheck**
	执行使用APIs链接远程设备的检查，用于持续集成。
- **build**
	执行组装和检查任务。
- **clean**
    执行清理产出的任务。

Note that `build` does not depend on `deviceCheck`, or `connectedCheck`.  
新创建的锚点任务必须支持没有可用链接设备的情况。
需要注意的是，`build`任务并不依赖于`deviceCheck`或`connectedCheck`。

对于Android工程而言，至少会有两个产出：一个排错版APK和一个发行版APK，对于其中的每一个都会有对应的锚点任务以便分别构建：

- assemble
	- assembleDebug
	- assembleRelease

上述两个的构建过程都依赖于多个其他任务步骤的执行，`assemble`任务又依赖于他们两个，所以执行结果会构建产生两个APK文件。

**小贴士** Gradle支持驼峰式写法的缩写，举个例子：在没有其他相同匹配的情况下`gradle aR`等同于 `gradle assembleRelease`。

检查锚点任务有他们自己的依赖关系：

- check
	- lint
- connectedCheck
	- connectedAndroidTest
	- connectedUiAutomatorTest (not implemented yet)
- deviceCheck
	- This depends on tasks created when other plugins implement test extension points.  

最终插件会针对所有构建类型 (`debug`, `release`, `test`)创建安装/卸载任务，当然，只有经过签名之后才能被安装。

### Basic Build Customization | 自定义构建过程基础
// TODO:

#### Manifest entries | Manifest选项
// TODO:

#### Build Types | 构建类型
// TODO:

#### Signing Configurations | 配置签名
// TODO:

#### Running ProGuard | 执行混淆
// TODO:

#### Shrinking Resources | 压缩资源
// TODO:

## Dependencies, Android Libraries and Multi-project setup | 依赖关系，Android库和Multi-project配置
// TODO:

### Dependencies on binary packages | 依赖于二进制包
// TODO:

#### Local packages | 本地包
// TODO:

#### Remote artifacts | 远端神器
// TODO:

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
// TODO:

### Product flavors | 产品风味
// TODO:

### Build Type + Product Flavor = Build Variant | 构建类型 + 产品风味 = 变种版本
// TODO:

### Product Flavor Configuration | 配置产品风味
// TODO:

### Sourcesets and Dependencies | 代码集与依赖关系
// TODO:

### Building and Tasks | 构建与任务
// TODO:

### Testing | 测试
// TODO:

### Multi-flavor variants | 多种风味的变种版本
// TODO:

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


[1]: http://tools.android.com/tech-docs/new-build-system/user-guide        "Gradle Plugin User Guide"
[2]: https://en.wikipedia.org/wiki/Domain-specific_language                                    "wiki of DSL"
[3]: http://www.groovy-lang.org/                                                                                   "groovy"
[4]: https://maven.apache.org/                                                                                      "maven"
[5]: http://developer.android.com/google/play/publishing/multiple-apks.html      "Maven Repository Centre"
[6]: http://maven.apache.org/repository/index.html                                                   "The Central Repository"
[7]: http://maven.apache.org/ref/3.2.5/maven-artifact/                                              "Maven artifact"
[8]: https://docs.gradle.org/current/userguide/java_plugin.html                               "The Java Plugin"