# working with Gradle for Android Development

**This is a summary version of the official [User Guide][1]**  

[TOC]

## Introduction | 介绍
Gradle is an advanced build system as well as an advanced build toolkit allowing to create custom build logic through plugins.  

adb
记忆病

### Goals of Gradle | 目标
- Make it easy to reuse code and resources;  

- Make it easy to create several variants of an application,  either for [multi-apk][5] distribution or for different flavors of an application; (Such as : pad , lite , free , abc  full , main etc.）  
	简化为一个项目创建多个变种版本的工作，包括支持[multi-apk][5]打包，和维护一个应用的多个不同风味的发行版（比如：pad，lite，free，full，main等）  
- Make it easy to create several variants of an application,  either for [multi-apk][5] distribution or for different flavors of an application; (Such as : pad , lite , free , full , main etc.）  

- Make it easy to configure, extend and customize the build process  

- Good IDE integration  

### Why Gradle? | 何得何能？
- [Domain Specific Language (DSL)][2] to describe and manipulate the build logic  

- Build files are [Groovy][3] based and allow mixing of declarative elements through the DSL and using code to manipulate the DSL elements to provide custom logic.  
	编译脚本adb文件基于 [Groovy][3]，并且允许采用DSL和代码混合使用的方式来操控DSL元素，从而自定义构建过程  

- Built-in dependency management through [Maven][4] and/or Ivy  

- Very flexible. Allows using best practices but doesn’t force its own way of doing things.  

- Plugins can expose their own DSL and their own API for build files to use.  

- Good Tooling API allowing IDE integration  

## Requirements | 系统要求
- Gradle 1.10 or 1.11 or 1.12 with the plugin 0.11.1
- SDK with Build Tools 19.0.0. Some features may require a more recent version.

## Basic Project | 基础工程介绍
A Gradle project describes its build in a file called build.gradle located in the root folder of the project.  

### Simple build files | 简易构建脚本
The most simple Android project has the following build.gradle;   
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

#### apply plugin
The **plugin** provides everything to build and test a project. for android project, You should only apply the android plugin. Applying the java plugin as well will result in a build error.  

- `apply plugin: 'java'`  
	packaged with Gradle; This applies the Java plugin, provides everything to build and test Java applications.  
- `apply plugin: 'android'`  
	packaged with Gradle since v0.11.1; Like the Java plugin, provides everything to build and test android applications.  

#### buildscript
`buildscript { ... }` configures the code driving the build.    
In this case, this declares that it uses the [Maven Central repository][6], and that there is a classpath dependency on a [Maven artifact][7]. This artifact is the library that contains the Android plugin for Gradle in version 0.11.1
Note: This only affects the code running the build, not the project. The project itself needs to declare its own repositories and dependencies. This will be covered later.   

#### android
`android { ... }` configures all the parameters for the android build. This is the entry point for the Android DSL.
By default, only the compilation target, and the version of the build-tools are needed. This is done with the `compileSdkVersion` and `buildtoolsVersion` properties.  
The compilation target is the same as the target property in the project.properties file of the old build system. This new property can either be assigned a int (the api level) or a string with the same value as the previous target property.  

**Note:** You will also need a local.properties file to set the location of the SDK in the same way that the existing SDK requires, using the `sdk.dir` property.  
Alternatively, you can set an environment variable called `ANDROID_HOME`. There is no differences between the two methods, you can use the one you prefer.  

### Project Structure | 工程结构
The basic build files above expect a default folder structure. Gradle follows the concept of convention over configuration, providing sensible default option values when possible.  

The basic project starts with two components called “source sets”. The main source code and the test code. These live respectively in:  

- `src/main/`
- `src/androidTest/`

Inside each of these folders exists folder for each source components.  
For both the Java and Android plugin, the location of the Java source code and the Java resources:  

- `java/`
- `resources/`

For the Android plugin, extra files and folders specific to Android:  
 
- `AndroidManifest.xml`
- `res/`
- `assets/`
- `aidl/`
- `rs/`
- `jni/`
- `jniLibs/`

**Note:** `src/androidTest/AndroidManifest.xml` is not needed as it is created automatically.  

#### Configuring the Structure | 配置工程目录
When the default project structure isn’t adequate, it is possible to configure it. According to the Gradle documentation, reconfiguring the `sourceSets` for a Java project can be done with the following:  
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

To replace the default source folders, you will want to use `srcDirs` instead, which takes an array of path. This also shows a different way of using the objects involved:  
```groovy
sourceSets {
    main.java.srcDirs = ['src/java']
    main.resources.srcDirs = ['src/resources']
}
```
For more information, see the Gradle documentation on the Java plugin [here][8].

The Android plugin uses a similar syntaxes, but because it uses its own `sourceSets`, this is done within the `android` object.    
Here’s an example, using the old project structure for the main code and remapping the `androidTest` `sourceSet` to the tests folder:  
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

**Note:** `setRoot()` moves the whole `sourceSet` (and its sub folders) to a new folder. This moves `src/androidTest/*` to `tests/* `  

This is Android specific and will not work on Java `sourceSets`.  

The ‘migrated’ sample shows this.    

### Build Tasks | 构建任务

#### General Tasks | 常规任务
Applying a plugin to the build file automatically creates a set of build tasks to run. Both the Java plugin and the Android plugin do this.  
The convention for tasks is the following:  

- **assemble**  
	The task to assemble the output(s) of the project  
	
- **check**  
	The task to run all the checks.  
	
- **build**  
	This task does both `assemble` and `check`  
    
- **clean**  
	This task cleans the output of the project  
    
The tasks `assemble`, `check` and `build` don’t actually do anything. They are anchor tasks for the plugins to add actual tasks that do the work.  

This allows you to always call the same task(s) no matter what the type of project is, or what plugins are applied.  
For instance, applying the `findbugs` plugin will create a new task and make `check` depend on it, making it be called whenever the `check` task is called  

list tasks from the command line：  

- `gradle tasks`  
	get the high level task running.  

- `gradle tasks --all`  
	full list and seeing dependencies between the tasks run.  

Note: Gradle automatically monitor the declared inputs and outputs of a task.  
Running the `build` twice without change will make Gradle report all tasks as `UP-TO-DATE`, meaning no work was required. This allows tasks to properly depend on each other without requiring unneeded build operations.  

#### Java project tasks | java工程任务
The Java plugin creates mainly two tasks, that are dependencies of the main anchor tasks:  

- assemble
	- jar  
		This task creates the output.  
    
- check  
    - test  
    	This task runs the tests.  
    
The `jar` task itself will depend directly and indirectly on other tasks: `classes` for instance will compile the Java code.  
The tests are compiled with `testClasses`, but it is rarely useful to call this as `test` depends on it (as well as `classes`).  

In general, you will probably only ever call `assemble` or `check`, and ignore the other tasks.   

You can see the full set of tasks and their descriptions for the Java plugin [here][8].  

#### Android tasks | Android任务
The Android plugin use the same convention to stay compatible with other plugins, and adds an additional anchor task:  

- **assemble**  
	The task to assemble the output(s) of the project  
	
- **check**  
	The task to run all the checks.  
	
- **connectedCheck**  
	Runs checks that requires a connected device or emulator. they will run on all connected devices in parallel.  
	
- **deviceCheck**  
	Runs checks using APIs to connect to remote devices. This is used on CI servers.  
	
- **build**  
	This task does both assemble and check  
	
- **clean**  
	This task cleans the output of the project  

The new anchor tasks are necessary in order to be able to run regular checks without needing a connected device.  
Note that `build` does not depend on `deviceCheck`, or `connectedCheck`.  

An Android project has at least two outputs: a debug APK and a release APK. Each of these has its own anchor task to facilitate building them separately:  

- assemble
	- assembleDebug
	- assembleRelease

They both depend on other tasks that execute the multiple steps needed to build an APK. The `assemble` task depends on both, so calling it will build both APKs.  

**Tip:** Gradle support camel case shortcuts for task names on the command line. For instance: `gradle aR` is the same as typing `gradle assembleRelease` as long as no other task match `aR`.  

The check anchor tasks have their own dependencies:  

- check
	- lint
- connectedCheck
	- connectedAndroidTest
	- connectedUiAutomatorTest (not implemented yet)
- deviceCheck
	- This depends on tasks created when other plugins implement test extension points.  

Finally, the plugin creates install/uninstall tasks for all build types (`debug`, `release`, `test`), as long as they can be installed (which requires signing).  

### Basic Build Customization | 自定义构建过程基础
The Android plugin provides a broad DSL to customize most things directly from the build system.  

#### Manifest entries | Manifest选项
Through the DSL it is possible to configure the following manifest entries:  

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

Previous versions of the Android Plugin used *packageName* to configure the manifest 'packageName' attribute.  
Starting in 0.11.0, you should use *applicationId* in the build.gradle to configure the manifest 'packageName' entry.  
This was disambiguated to reduce confusion between the application's packageName (which is its ID) and java packages.  

The power of describing it in the build file is that it can be dynamic.  
For instance, one could be reading the version name from a file somewhere or using some custom logic:  

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

If a property is not set through the DSL, some default value will be used. Here’s a table of how this is processed.  

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
```groovy
if (android.defaultConfig.testInstrumentationRunner == null) {
    // assign a better default...
}
```

If the value remains null, then it is replaced at build time by the actual default from column 3, but the DSL element does not contain this default value so you can't query against it.  
This is to prevent parsing the manifest of the application unless it’s really needed.  

#### Build Types | 构建类型
By default, the Android plugin automatically sets up the project to build both a debug and a release version of the application.  
These differ mostly around the ability to debug the application on a secure (non dev) devices, and how the APK is signed.  

The debug version is signed with a key/certificate that is created automatically with a known name/password (to prevent required prompt during the build). The release is not signed during the build, this needs to happen after.  

This configuration is done through an object called a `BuildType`. By default, 2 instances are created, a `debug` and a `release` one.  

The Android plugin allows customizing those two instances as well as creating other Build Types. This is done with the `buildTypes` DSL container:  

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

- Configures the default `debug` `Build` Type:  
	- set its package to be `<app appliationId>.debug` to be able to install both debug and release apk on the same device  
- Creates a new `BuildType` called `jnidebug` and configure it to be a copy of the `debug` build type.  
- Keep configuring the jnidebug, by enabling debug build of the JNI component, and add a different package suffix.  
	
Creating new Build Types is as easy as using a new element under the `buildTypes` container, either to call `initWith()` or to configure it with a closure.  

The possible properties and their default values are:  

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
	`src/<buildtypename>/`
This means the `Build Type` names cannot be `main` or `androidTest` (this is enforced by the plugin), and that they have to be unique to each other.  


Like any other source sets, the location of the build type source set can be relocated:  

```groovy
android {
    sourceSets.jnidebug.setRoot('foo/jnidebug')
}
```
Additionally, for each Build Type, a new `assemble<BuildTypeName>` task is created.  

The `assembleDebug` and `assembleRelease` tasks have already been mentioned, and this is where they come from. When the `debug` and `release` Build Types are pre-created, their tasks are automatically created as well.  

The `build.gradle` snippet above would then also generate an `assembleJnidebug` task, and `assemble` would be made to depend on it the same way it depends on the `assembleDebug` and `assembleRelease` tasks.  

**Tip:** remember that you can type `gradle aJ` to run the `assembleJnidebug` task.  

Possible use case:  

- Permissions in debug mode only, but not in release mode  
- Custom implementation for debugging  
- Different resources for debug mode (for instance when a resource value is tied to the signing certificate).  

The code/resources of the BuildType are used in the following way:  

- The manifest is merged into the app manifest  
- The code acts as just another source folder  
- The resources are overlayed over the main resources, replacing existing values.  

#### Signing Configurations | 配置签名
Signing an application requires the following:  

- A keystore
- A keystore password
- A key alias name
- A key password
- The store type

The location, as well as the key name, both passwords and store type form together a Signing Configuration (type `SigningConfig`)  

By default, there is a debug configuration that is setup to use a debug keystore, with a known password and a default key with a known password.  
The debug keystore is located in `$HOME/.android/debug.keystore`, and is created if not present.  

It is possible to create other configurations or customize the default built-in one. This is done through the `signingConfigs` DSL container:  

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

It also creates a new Signing Config and a new Build Type that uses the new configuration.  

**Note:** Only debug keystores located in the default location will be automatically created. Changing the location of the debug keystore will not create it on-demand. Creating a SigningConfig with a different name that uses the default debug keystore location will create it automatically. In other words, it’s tied to the location of the keystore, not the name of the configuration.  

**Note:** Location of keystores are usually relative to the root of the project, but could be absolute paths, thought it is not recommended (except for the debug one since it is automatically created).  

**Note:**  If you are checking these files into version control, you may not want the password in the file. The following Stack Overflow [post][10] shows ways to read the values from the console, or from environment variables.
We'll update this guide with more detailed information later.


#### Running ProGuard | 执行混淆
ProGuard is supported through the Gradle plugin for ProGuard version 4.10. The ProGuard plugin is applied automatically, and the tasks are created automatically if the Build Type is configured to run ProGuard through the `minifyEnabled` property.  

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

There are 2 default rules files  

- proguard-android.txt
- proguard-android-optimize.txt

They are located in the SDK. Using `getDefaultProguardFile()` will return the full path to the files. They are identical except for enabling optimizations.  

#### Shrinking Resources | 压缩资源
You can also remove unused resources, automatically, at build time. For more information, see the [Resource Shrinking][11] document.  

## Dependencies, Android Libraries and Multi-project setup | 依赖关系，Android库和Multi-project配置
Gradle projects can have dependencies on other components. These components can be external binary packages, or other Gradle projects.  

### Dependencies on binary packages | 依赖于二进制包
#### Local packages | 本地包
To configure a dependency on an external library jar, you need to add a dependency on the `compile` configuration.  

```groovy
dependencies {
    compile files('libs/foo.jar')
}

android {
    ...
}
```

**Note:** the `dependencies` DSL element is part of the standard Gradle API and does not belong inside the `android` element.  

The compile configuration is used to compile the main application. Everything in it is added to the compilation classpath and also packaged in the final APK.  
There are other possible configurations to add dependencies to:  

- `compile`: main application
- `androidTestCompile`: test application
- `debugCompile`: debug Build Type
- `releaseCompile`: release Build Type.

Because it’s not possible to build an APK that does not have an associated Build Type, the APK is always configured with two (or more) configurations: `compile` and `<buildtype>Compile`.  
Creating a new Build Type automatically creates a new configuration based on its name.  

This can be useful if the debug version needs to use a custom library (to report crashes for instance), while the release doesn’t, or if they rely on different versions of the same library.  

#### Remote artifacts | 远端神器
Gradle supports pulling artifacts from Maven and Ivy repositories.  

First the repository must be added to the list, and then the dependency must be declared in a way that Maven or Ivy declare their artifacts.  

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

For more information about setting up dependencies, read the Gradle user guide [here][13], and DSL documentation [here][14].  

#### Multi project setup | 多工程配置
Gradle projects can also depend on other gradle projects by using a multi-project setup.  
A multi-project setup usually works by having all the projects as sub folders of a given root project.  
For instance, given to following structure:  

```
MyProject/
 + app/
 + libraries/
    + lib1/
    + lib2/
```

We can identify 3 projects. Gradle will reference them with the following name:  

```
:app
:libraries:lib1
:libraries:lib2
```

Each projects will have its own build.gradle declaring how it gets built.  
Additionally, there will be a file called settings.gradle at the root declaring the projects.  
This gives the following structure:  

```
MyProject/
 | settings.gradle
 + app/
    | build.gradle
 + libraries/
    + lib1/
       | build.gradle
    + lib2/
       | build.gradle

```

The content of settings.gradle is very simple，we can defines which folder is actually a Gradle project like this :  
```groovy
include ':app', ':libraries:lib1', ':libraries:lib2'
```
The `:app` project is likely to depend on the libraries, and this is done by declaring the following dependencies:  

```groovy
dependencies {
    compile project(':libraries:lib1')
}
```

More general information about multi-project setup [here][15].  

### Library projects | 库工程
In the above multi-project setup, `:libraries:lib1` and `:libraries:lib2` can be Java projects, and the `:app` Android project will use their jar output.  
However, if you want to share code that accesses Android APIs or uses Android-style resources, these libraries cannot be regular Java project, they have to be Android Library Projects.  

#### Creating a Library Project | 创建库工程
A Library project is very similar to a regular Android project with a few differences.  
Since building libraries is different than building applications, a different plugin is used. Internally both plugins share most of the same code and they are both provided by the same `com.android.tools.build.gradle` jar.  

```groovy
buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:0.5.6'
    }
}

apply plugin: 'android-library'

android {
    compileSdkVersion 15
}
```

This creates a library project that uses API 15 to compile. SourceSets, and dependencies are handled the same as they are in an application project and can be customized the same way.  

#### Differences between a Project and a Library Project | 普通工程和库工程的区别
A Library project's main output is an `.aar` package (which stands for Android archive). It is a combination of compile code (as a jar file and/or native .so files) and resources (manifest, res, assets).  
A library project can also generate a test apk to test the library independently from an application.  

The same anchor tasks are used for this (`assembleDebug`, `assembleRelease`) so there’s no difference in commands to build such a project.  

For the rest, libraries behave the same as application projects. They have build types and product flavors, and can potentially generate more than one version of the aar.  
Note that most of the configuration of the Build Type do not apply to library projects. However you can use the custom sourceSet to change the content of the library depending on whether it’s used by a project or being tested.  

#### Referencing a Library | 引用库
Referencing a library is done the same way any other project is referenced:  

```groovy
dependencies {
    compile project(':libraries:lib1')
    compile project(':libraries:lib2')
}
```

**Note:** if you have more than one library, then the order will be important. This is similar to the old build system where the order of the dependencies in the `project.properties` file was important.  

#### Library Publication | 发布库
By default a library only publishes its release variant. This variant will be used by all projects referencing the library, no matter which variant they build themselves. This is a temporary limitation due to Gradle limitations that we are working towards removing.  

You can control which variant gets published with  

```groovy
android {
    defaultPublishConfig "debug"
}
```

Note that this publishing configuration name references the full variant name. `Release` and `debug` are only applicable when there are no flavors. If you wanted to change the default published variant while using flavors, you would write:  

```groovy
android {
    defaultPublishConfig "flavor1Debug"
}
```
It is also possible to publish all variants of a library. We are planning to allow this while using a normal project-to-project dependency (like shown above), but this is not possible right now due to limitations in Gradle (we are working toward fixing those as well).  
Publishing of all variants are not enabled by default. To enable them:  

```groovy
android {
    publishNonDefault true
}
```

It is important to realize that publishing multiple variants means publishing multiple aar files, instead of a single aar containing multiple variants. Each aar packaging contains a single variant.  
Publishing an variant means making this aar available as an output artifact of the Gradle project. This can then be used either when publishing to a maven repository, or when another project creates a dependency on the library project.  

Gradle has a concept of default artifact. This is the one that is used when writing:  

```groovy
compile project(':libraries:lib2')
```

To create a dependency on another published artifact, you need to specify which one to use:  

```groovy
dependencies {
    flavor1Compile project(path: ':lib1', configuration: 'flavor1Release')
    flavor2Compile project(path: ':lib1', configuration: 'flavor2Release')
}
```

**Important:** Note that the published configuration is a full variant, including the build type, and needs to be referenced as such.  
**Important:** When enabling publishing of non default, the Maven publishing plugin will publish these additional variants as extra packages (with classifier). This means that this is not really compatible with publishing to a maven repository. You should either publish a single variant to a repository OR enable all config publishing for inter-project dependencies.  

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

There are two main use cases:  

1. Different versions of the same application. For instance, a free/demo version vs the “pro” paid application.  
2. Same application packaged differently for multi-apk in Google Play Store. See [Multiple APK Support][12] for more information.  
3. A combination of 1. and 2. 

The goal was to be able to generate these different APKs from the same project, as opposed to using a single Library Projects and 2+ Application Projects.  

### Product flavors | 产品风味
A product flavor defines a customized version of the application build by the project. A single project can have different flavors which change the generated application.  

This new concept is designed to help when the differences are very minimum. If the answer to “Is this the same application?” is yes, then this is probably the way to go over Library Projects.  

Product flavors are declared using a `productFlavors` DSL container:  

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

### Build Type + Product Flavor = Build Variant | 构建类型 + 产品风味 = 变种版本
As we have seen before, each Build Type generates a new APK.  

Product Flavors do the same: the output of the project becomes all possible combinations of Build Types and, if applicable, Product Flavors.  

Each (Build Type, Product Flavor) combination is called a Build Variant.  

For instance, with the default `debug` and `release` Build Types, the above example generates four Build Variants:  

- Flavor1 - debug
- Flavor1 - release
- Flavor2 - debug
- Flavor2 - release

Projects with no flavors still have Build Variants, but the single `default` flavor/config is used, nameless, making the list of variants similar to the list of Build Types.  

### Product Flavor Configuration | 配置产品风味
Each flavors is configured with a closure:  

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

`defaultConfig` provides the base configuration for all flavors and each flavor can override any value. In the example above, the configurations end up being:  

- flavor1
	- packageName: com.example.flavor1
	- minSdkVersion: 8
	- versionCode: 20
- flavor2
	- packageName: com.example.flavor2
	- minSdkVersion: 14
	- versionCode: 10

Usually, the Build Type configuration is an overlay over the other configuration. For instance, the Build Type's `packageNameSuffix` is appended to the Product Flavor's packageName.  

There are cases where a setting is settable on both the Build Type and the Product Flavor. In this case, it’s is on a case by case basis.  

For instance, signingConfig is one of these properties.  
This enables either having all release packages share the same `SigningConfig`, by setting `android.buildTypes.release.signingConfig`, or have each release package use their own `SigningConfig` by setting each `android.productFlavors.*.signingConfig` objects separately.  

### Sourcesets and Dependencies | 代码集与依赖关系
Similar to Build Types, Product Flavors also contribute code and resources through their own sourceSets.  

The above example creates four sourceSets:  

- android.sourceSets.flavor1  
	Location src/flavor1/  
- android.sourceSets.flavor2  
	Location src/flavor2/  
- android.sourceSets.androidTestFlavor1  
	Location src/androidTestFlavor1/  
- android.sourceSets.androidTestFlavor2  
	Location src/androidTestFlavor2/  

Those sourceSets are used to build the APK, alongside `android.sourceSets.main` and the Build Type sourceSet.  

The following rules are used when dealing with all the sourcesets used to build a single APK:  

- All source code (`src/*/java`) are used together as multiple folders generating a single output.
- Manifests are all merged together into a single manifest. This allows Product Flavors to have different components and/or permissions, similarly to Build Types.
- All resources (Android res and assets) are used using overlay priority where the Build Type overrides the Product Flavor, which overrides the main sourceSet.
- Each Build Variant generates its own R class (or other generated source code) from the resources. Nothing is shared between variants.

Finally, like Build Types, Product Flavors can have their own dependencies. For instance, if the flavors are used to generate a ads-based app and a paid app, one of the flavors could have a dependency on an Ads SDK, while the other does not.  

```groovy
dependencies {
    flavor1Compile "..."
}
```
In this particular case, the file `src/flavor1/AndroidManifest.xml` would probably need to include the internet permission.  

Additional sourcesets are also created for each variants:  

- android.sourceSets.flavor1Debug  
	Location src/flavor1Debug/  
- android.sourceSets.flavor1Release  
	Location src/flavor1Release/  
- android.sourceSets.flavor2Debug  
	Location src/flavor2Debug/  
- android.sourceSets.flavor2Release  
	Location src/flavor2Release/  

These have higher priority than the build type sourcesets, and allow customization at the variant level.  

### Building and Tasks | 构建与任务
We previously saw that each Build Type creates its own `assemble<name>` task, but that Build Variants are a combination of Build Type and Product Flavor.  

When Product Flavors are used, more assemble-type tasks are created. These are:  

1. `assemble<Variant Name>`  
	allows directly building a single variant. For instance `assembleFlavor1Debug`.  
2. `assemble<Build Type Name>`  
	allows building all APKs for a given Build Type. For instance `assembleDebug` will build both `Flavor1Debug` and `Flavor2Debug` variants.  
3. `assemble<Product Flavor Name>`  
	allows building all APKs for a given flavor. For instance `assembleFlavor1` will build both `Flavor1Debug` and `Flavor1Release` variants.  

The task `assemble` will build all possible variants.  

### Testing | 测试
// TODO:

### Multi-flavor variants | 多种风味的变种版本
In some case, one may want to create several versions of the same apps based on more than one criteria.  
For instance, multi-apk support in Google Play supports 4 different filters. Creating different APKs split on each filter requires being able to use more than one dimension of Product Flavors.  

Consider the example of a game that has a demo and a paid version and wants to use the ABI filter in the multi-apk support. With 3 ABIs and two versions of the application, 6 APKs needs to be generated (not counting the variants introduced by the different Build Types).  
However, the code of the paid version is the same for all three ABIs, so creating simply 6 flavors is not the way to go.  
Instead, there are two dimensions of flavors, and variants should automatically build all possible combinations.  

This feature is implemented using Flavor Dimensions. Flavors are assigned to a specific dimension  
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

From the following dimensioned Product Flavors [freeapp, paidapp] and [x86, arm, mips] and the [debug, release] Build Types, the following build variants will be created:  

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

Each variant is configured by several Product Flavor objects:  

- android.defaultConfig
- One from the abi dimension
- One from the version dimension

The order of the dimension drives which flavor override the other, which is important for resources when a value in a flavor replaces a value defined in a lower priority flavor.  
The flavor dimension is defined with higher priority first. So in this case:  

	`abi > version > defaultConfig`

Multi-flavors projects also have additional sourcesets, similar to the variant sourcesets but without the build type:  

- android.sourceSets.x86Freeapp
	Location src/x86Freeapp/
- android.sourceSets.armPaidapp
	Location src/armPaidapp/
- etc...

These allow customization at the flavor-combination level. They have higher priority than the basic flavor sourcesets, but lower priority than the build type sourcesets.  

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
[15]: https://docs.gradle.org/current/userguide/multi_project_builds.html                               "Multi-project Builds"
