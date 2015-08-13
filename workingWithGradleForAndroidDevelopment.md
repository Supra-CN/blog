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
Gradle

- `apply plugin: 'android'`  
packaged with Gradle since v0.11.1; Like the Java plugin, provides everything to build and test android applications.

#### buildscript
`buildscript { ... }` configures the code driving the build.
In this case, this declares that it uses the Maven Central repository, and that there is a classpath dependency on a Maven artifact. This artifact is the library that contains the Android plugin for Gradle in version 0.11.1
Note: This only affects the code running the build, not the project. The project itself needs to declare its own repositories and dependencies. This will be covered later.










[1]: http://tools.android.com/tech-docs/new-build-system/user-guide        "Gradle Plugin User Guide"
[2]: https://en.wikipedia.org/wiki/Domain-specific_language                                    "wiki of DSL"
[3]: http://www.groovy-lang.org/                                                                                   "groovy"
[4]: https://maven.apache.org/                                                                                      "maven"
[5]: http://developer.android.com/google/play/publishing/multiple-apks.html       "Multiple APK Support"
