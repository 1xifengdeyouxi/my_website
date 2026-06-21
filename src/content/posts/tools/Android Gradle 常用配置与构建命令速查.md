---
title: Android Gradle 常用配置与构建命令速查
published: 2026-06-22
description: 这篇文章整理 Android Gradle 的基础概念、项目结构、依赖管理、构建变体、签名配置、多渠道打包、混淆、性能优化和常用构建命令。
image: "http://img.wmt9038.xyz/file/test/1781018256642_SNAFFUR-动漫少女.png"
tags: [gradle, Android]
category: 工具
draft: false
---

## 前言

Android 项目离不开 Gradle。无论是日常编译、安装 APK、刷新依赖，还是配置多渠道、签名、混淆、构建变体和多模块工程，最终都会落到 Gradle 配置和命令上。

这篇文章把 Android Gradle 常用知识整理成一份速查手册。它不追求把 Gradle 所有细节讲完，而是围绕实际开发中最常遇到的问题：项目怎么配、依赖怎么管、变体怎么构建、包怎么打、构建慢怎么优化，以及常见命令应该怎么用。

## 常用构建命令

日常开发最常用的命令通常围绕清理、构建、安装、刷新依赖和离线构建展开。

```bash title="常用 Gradle 构建命令"
# 清理构建产物
./gradlew clean

# 构建 Debug 包
./gradlew assembleDebug

# 构建指定变体
./gradlew assembleGoogleRelease

# 构建并安装到设备
./gradlew installDebug

# 查看所有任务
./gradlew tasks --all

# 查看项目属性
./gradlew properties

# 查看依赖树
./gradlew app:dependencies

# 查看指定配置的依赖树
./gradlew app:dependencies --configuration debugRuntimeClasspath

# 刷新依赖
./gradlew build --refresh-dependencies

# 离线模式构建
./gradlew assembleDebug --offline

# 跳过测试
./gradlew assembleDebug -x test

# 生成构建耗时报告
./gradlew assembleDebug --profile
```

如果项目里有很多业务变体，命令名通常会带上渠道、环境、构建类型等信息。下面这些命令可以作为模板，实际使用时把 Variant 名替换成项目中的真实名称。

```bash title="业务 Variant 构建示例"
./gradlew clean assembleNetdiskDebug
./gradlew assembleNetdiskFastDebug --offline
./gradlew App:assembleNetdiskDebug
./gradlew App:assembleNetdiskFastDebugSign
./gradlew clean assembleNetdiskProguardNoLog
```

安装 APK 时常见写法：

```bash title="安装 APK"
adb install -r App/build/outputs/apk/netdisk/debug/BaiduNetDisk-debug.apk
adb install -r App/build/outputs/apk/netdisk/fastDebug/BaiduNetDisk-fastDebug.apk
```

> [!TIP] 小建议
> 项目里的 Variant 名往往非常长，建议把常用构建命令整理成脚本或文档，避免每次都靠记忆手敲。

## Gradle 基础概念

Gradle 是一个基于 JVM 的自动化构建工具。它结合了 Ant 的灵活性和 Maven 的依赖管理能力，在 Android 项目中主要负责：

- 编译源码；
- 处理资源；
- 管理依赖；
- 生成不同构建变体；
- 执行测试；
- 生成 APK / AAB；
- 处理签名、混淆、资源压缩等任务。

Gradle 的核心特点包括：

- 支持 Groovy DSL 和 Kotlin DSL；
- 依赖管理能力强；
- 支持增量构建；
- 支持并行构建；
- 插件生态丰富；
- Android Gradle Plugin 对 Android 构建做了专门封装。

## Android Gradle Plugin

Android Gradle Plugin，简称 AGP，是 Google 提供的 Gradle 插件，专门用于构建 Android 应用和 Android Library。

AGP、Gradle 和 JDK 之间存在版本对应关系。使用时要注意版本兼容，否则很容易遇到构建失败、插件不兼容或 JDK 版本不匹配的问题。

| AGP 版本 | Gradle 版本 | JDK 版本 |
| :--- | :--- | :--- |
| 8.7.x | 8.9 | 17 |
| 8.6.x | 8.8 | 17 |
| 8.5.x | 8.7 | 17 |
| 8.4.x | 8.6 | 17 |
| 8.3.x | 8.4 | 17 |

升级 AGP 时，不要只改插件版本。最好同步检查：

- Gradle Wrapper 版本；
- JDK 版本；
- Kotlin 插件版本；
- Android Studio 版本；
- 第三方 Gradle 插件兼容性。

## 三类常见配置文件

Android 项目里最常见的 Gradle 配置文件有三类：

| 文件 | 作用 | 适用场景 |
| :--- | :--- | :--- |
| `settings.gradle` | 声明项目名称和子模块 | 多模块项目 |
| 根目录 `build.gradle` | 配置项目级插件、仓库、全局变量 | 整个工程 |
| 模块 `build.gradle` | 配置具体模块的 Android 构建逻辑 | App 或 Library 模块 |

示例：

```groovy title="settings.gradle"
rootProject.name = 'MyProject'
include ':app'
include ':lib_module'
include ':feature_module'
```

Kotlin DSL 写法：

```kotlin title="settings.gradle.kts"
rootProject.name = "MyProject"
include(":app")
include(":lib_module")
```

## 标准 Android 项目结构

一个标准 Android 项目通常长这样：

```text
MyProject/
├── settings.gradle
├── build.gradle
├── gradle.properties
├── app/
│   ├── build.gradle
│   └── src/
│       ├── main/
│       │   ├── java/
│       │   ├── kotlin/
│       │   └── res/
│       ├── debug/
│       └── release/
└── lib_module/
    ├── build.gradle
    └── src/
```

其中 `src/main` 是主源码集，`src/debug` 和 `src/release` 可以放不同构建类型专属的资源或代码。

## 项目级 build.gradle

项目级 `build.gradle` 常用于声明插件版本和全局配置。

```groovy title="build.gradle"
plugins {
    id 'com.android.application' version '8.2.0' apply false
    id 'com.android.library' version '8.2.0' apply false
    id 'org.jetbrains.kotlin.android' version '1.9.20' apply false
}

tasks.register('clean', Delete) {
    delete rootProject.buildDir
}

ext {
    compileSdkVersion = 34
    minSdkVersion = 24
    targetSdkVersion = 34
    versionCode = 1
    versionName = "1.0.0"
}
```

`apply false` 表示插件版本在项目级统一声明，但不直接应用到根项目。具体模块需要时再应用。

## 模块级 build.gradle

模块级 `build.gradle` 才是真正配置 App 或 Library 构建逻辑的地方。

```groovy title="app/build.gradle" showLineNumbers
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {
    namespace 'com.example.myapp'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.myapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        debug {
            applicationIdSuffix ".debug"
            debuggable true
        }

        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = '17'
    }
}
```

这里最关键的是 `android {}` 块，它决定了 SDK 版本、包名、构建类型、签名、源码集、变体等配置。

## 依赖管理

### 常见依赖声明

```groovy title="dependencies.gradle"
dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'

    implementation group: 'androidx.core', name: 'core-ktx', version: '1.12.0'

    def okHttpVersion = '4.12.0'
    implementation "com.squareup.okhttp3:okhttp:$okHttpVersion"
}
```

不推荐使用动态版本：

```groovy title="不推荐"
implementation 'com.squareup.okhttp3:okhttp:4.+'
```

动态版本会让构建结果不稳定。今天构建和明天构建拿到的依赖版本可能不同，排查问题会变得很痛苦。

### 依赖配置类型

| 配置 | 说明 | 是否传递给依赖方 |
| :--- | :--- | :--- |
| `implementation` | 编译和运行时依赖 | 不暴露实现细节 |
| `api` | 公开 API 依赖 | 会传递给依赖方 |
| `compileOnly` | 仅编译时需要 | 不传递 |
| `runtimeOnly` | 仅运行时需要 | 不传递 |
| `testImplementation` | 单元测试依赖 | 不传递 |
| `androidTestImplementation` | Android 测试依赖 | 不传递 |

一般业务模块优先使用 `implementation`。只有当某个库的类型会暴露给外部模块使用时，才考虑 `api`。

### Version Catalog

现在更推荐使用版本目录统一管理依赖版本。

```toml title="gradle/libs.versions.toml"
[versions]
agp = "8.2.0"
kotlin = "1.9.20"
coreKtx = "1.12.0"
appcompat = "1.6.1"
material = "1.11.0"

[libraries]
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "coreKtx" }
androidx-appcompat = { group = "androidx.appcompat", name = "appcompat", version.ref = "appcompat" }
google-material = { group = "com.google.android.material", name = "material", version.ref = "material" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
```

模块中使用：

```groovy title="build.gradle"
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

dependencies {
    implementation libs.androidx.core.ktx
    implementation libs.androidx.appcompat
    implementation libs.google.material
}
```

### 本地依赖

```groovy title="local-dependencies.gradle"
dependencies {
    implementation files('libs/my-library.aar')
    implementation fileTree(dir: 'libs', include: ['*.aar'])
    implementation project(':lib_module')
}
```

### 排除传递依赖

```groovy title="exclude.gradle"
dependencies {
    implementation('com.example:library:1.0.0') {
        exclude group: 'com.android.support', module: 'support-v4'
        exclude module: 'support-v4'
    }
}
```

## 构建变体

Android 构建变体由 `Build Type` 和 `Product Flavor` 组合而来。

```text
Build Type × Product Flavor = Build Variant
```

### Build Type

```groovy title="build-types.gradle"
android {
    buildTypes {
        debug {
            debuggable true
            minifyEnabled false
            applicationIdSuffix ".debug"
            versionNameSuffix "-debug"
        }

        release {
            debuggable false
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

### Product Flavor

```groovy title="product-flavors.gradle"
android {
    flavorDimensions "channel", "env"

    productFlavors {
        google {
            dimension "channel"
            manifestPlaceholders = [CHANNEL: "google"]
        }

        huawei {
            dimension "channel"
            manifestPlaceholders = [CHANNEL: "huawei"]
        }

        develop {
            dimension "env"
            applicationIdSuffix ".dev"
            buildConfigField "String", "API_URL", '"https://dev.api.com"'
        }

        production {
            dimension "env"
            buildConfigField "String", "API_URL", '"https://api.com"'
        }
    }
}
```

组合后会生成类似：

```text
googleDevelopDebug
googleProductionDebug
googleDevelopRelease
googleProductionRelease
```

## Source Sets

Source Sets 可以让不同渠道、不同构建类型使用不同源码或资源。

```groovy title="source-sets.gradle"
android {
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src/main/java']
            res.srcDirs = ['src/main/res']
        }

        google {
            java.srcDirs = ['src/google/java']
            res.srcDirs = ['src/google/res']
        }

        googleDebug {
            java.srcDirs = ['src/googleDebug/java']
        }
    }
}
```

这个能力在多渠道包、定制化 UI、不同环境配置中很常用。

## 签名配置

### 基础签名配置

```groovy title="signing-configs.gradle"
android {
    signingConfigs {
        debug {
            storeFile file('debug.keystore')
            storePassword 'android'
            keyAlias 'androiddebugkey'
            keyPassword 'android'
        }

        release {
            storeFile file('release.keystore')
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias System.getenv("KEY_ALIAS")
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }

    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }

        release {
            signingConfig signingConfigs.release
        }
    }
}
```

### 推荐使用环境变量

```groovy title="signing-env.gradle"
android {
    signingConfigs {
        release {
            storeFile file(System.getenv("KEYSTORE_FILE") ?: 'release.keystore')
            storePassword System.getenv("KEYSTORE_PASSWORD")
            keyAlias System.getenv("KEY_ALIAS")
            keyPassword System.getenv("KEY_PASSWORD")
        }
    }
}
```

> [!IMPORTANT] 重要
> 签名密码、KeyAlias、私钥路径等敏感信息不要写死在仓库中。能用环境变量就用环境变量。

## 多渠道打包

### 使用 Product Flavor

```groovy title="channel-flavor.gradle"
android {
    flavorDimensions "channel"

    productFlavors {
        google {
            dimension "channel"
            manifestPlaceholders = [CHANNEL: "google"]
        }

        huawei {
            dimension "channel"
            manifestPlaceholders = [CHANNEL: "huawei"]
        }

        xiaomi {
            dimension "channel"
            manifestPlaceholders = [CHANNEL: "xiaomi"]
        }
    }
}
```

### Manifest Placeholder

```xml title="AndroidManifest.xml"
<manifest>
    <application>
        <meta-data
            android:name="CHANNEL"
            android:value="${CHANNEL}" />
    </application>
</manifest>
```

### 批量打包任务

```groovy title="assemble-all-release.gradle"
tasks.register('assembleAllRelease') {
    dependsOn tasks.matching { it.name ==~ /assemble.*Release/ }
}
```

## 代码混淆与 R8

Release 包通常需要开启混淆和资源压缩。

```groovy title="minify.gradle"
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

常见 ProGuard 规则：

```proguard title="proguard-rules.pro"
# 保持实体类
-keep class com.example.myapp.entity.** { *; }

# 保持 Parcelable
-keep class * implements android.os.Parcelable { *; }
-keepclassmembers class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# 保持 Serializable
-keep class * implements java.io.Serializable { *; }

# 保持枚举
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 常见第三方库
-keep class okhttp3.** { *; }
-keep class retrofit2.** { *; }
-keep class com.google.gson.** { *; }
-keep class com.bumptech.glide.** { *; }
```

混淆后排查线上问题时，`mapping.txt` 很重要，发布流程里应该妥善保存。

## 自定义任务

Gradle 的一个强大能力是自定义任务。

```groovy title="custom-task.gradle"
tasks.register('printVersion') {
    doLast {
        println "Version: ${android.defaultConfig.versionName}"
        println "Version Code: ${android.defaultConfig.versionCode}"
    }
}

tasks.register('buildAndPrint') {
    dependsOn 'assembleRelease', 'printVersion'
    doLast {
        println "Build complete!"
    }
}
```

适合把一些重复命令固化成任务，例如生成版本号、拷贝产物、上传包、生成报告等。

## 动态生成版本号

```groovy title="version.gradle"
android {
    defaultConfig {
        versionCode generateVersionCode()
        versionName generateVersionName()
    }
}

def generateVersionCode() {
    def versionPropsFile = file('version.properties')
    def versionProps = new Properties()

    if (versionPropsFile.canRead()) {
        versionProps.load(new FileInputStream(versionPropsFile))
    }

    def versionCode = versionProps['VERSION_CODE'].toInteger()
    versionProps['VERSION_CODE'] = (++versionCode).toString()
    versionProps.store(versionPropsFile.newWriter(), null)

    return versionCode
}

def generateVersionName() {
    return "1.0.${new Date().format('yyyyMMddHHmm')}"
}
```

> [!WARNING] 注意
> 动态修改文件会影响构建可重复性。团队项目里最好明确版本号生成规则，避免本地构建产生意外文件变更。

## 多模块项目配置

多模块项目通常会在 `settings.gradle` 中声明模块：

```groovy title="settings.gradle"
rootProject.name = 'MyProject'
include ':app'
include ':core'
include ':feature-home'
include ':feature-user'
include ':data'
include ':domain'
```

模块之间通过 `project()` 依赖：

```groovy title="app/build.gradle"
dependencies {
    implementation project(':core')
    implementation project(':data')
    implementation project(':domain')
}
```

常见分层方式：

- `app`：壳工程和入口；
- `core`：基础能力；
- `feature-*`：业务功能模块；
- `data`：数据层；
- `domain`：领域层。

## BuildConfig 使用

`BuildConfig` 适合放构建期常量。

```groovy title="build-config.gradle"
android {
    defaultConfig {
        buildConfigField "String", "API_URL", '"https://api.example.com"'
        buildConfigField "boolean", "ENABLE_LOG", "true"
        buildConfigField "int", "MAX_RETRY", "3"
    }

    productFlavors {
        develop {
            buildConfigField "String", "API_URL", '"https://dev.api.com"'
        }

        production {
            buildConfigField "String", "API_URL", '"https://api.com"'
        }
    }
}
```

Kotlin 中使用：

```kotlin title="ApiService.kt"
class ApiService {
    private val apiUrl = BuildConfig.API_URL
    private val enableLog = BuildConfig.ENABLE_LOG
}
```

> [!CAUTION] 不要放密钥
> `BuildConfig` 里的内容会被打进 APK，不能当成真正安全的密钥存储。

## NDK 与 CMake 配置

```groovy title="ndk.gradle"
android {
    defaultConfig {
        ndk {
            abiFilters 'armeabi-v7a', 'arm64-v8a', 'x86', 'x86_64'
        }

        externalNativeBuild {
            cmake {
                cppFlags "-std=c++17"
                arguments "-DANDROID_STL=c++_shared"
            }
        }
    }

    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.22.1"
        }
    }
}
```

NDK 项目要特别注意 ABI 数量。支持的 ABI 越多，包体积和构建时间通常也越大。

## 依赖冲突处理

```groovy title="resolution-strategy.gradle"
configurations.all {
    resolutionStrategy {
        force 'com.google.guava:guava:31.1-android'
        failOnVersionConflict()
        cacheDynamicVersionsFor 10 * 60, 'seconds'
    }
}
```

也可以在单个依赖里排除冲突依赖：

```groovy title="exclude-conflict.gradle"
dependencies {
    implementation('com.example:library:1.0.0') {
        exclude group: 'com.google.guava', module: 'guava'
    }
}
```

## 构建性能优化

### gradle.properties

```properties title="gradle.properties"
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.configureondemand=true
org.gradle.daemon=true
org.gradle.workers.max=4
org.gradle.jvmargs=-Xmx4096m -XX:MaxMetaspaceSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

常见优化思路：

- 开启 Gradle Daemon；
- 开启构建缓存；
- 合理增加 JVM 内存；
- 开启并行构建；
- 避免动态依赖版本；
- 减少无用模块参与构建；
- 使用离线模式加速本地重复构建。

### Kotlin 编译选项

```groovy title="kotlin-options.gradle"
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = '17'
        freeCompilerArgs += [
            '-opt-in=kotlin.RequiresOptIn',
            '-Xjvm-default=all'
        ]
    }
}
```

## 常见问题

### 依赖下载失败

可以检查网络、代理、仓库配置。如果需要配置国内镜像：

```groovy title="repositories.gradle"
repositories {
    maven { url 'https://maven.aliyun.com/repository/google' }
    maven { url 'https://maven.aliyun.com/repository/central' }
    maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
    google()
    mavenCentral()
}
```

### 构建内存不足

```properties title="gradle.properties"
org.gradle.jvmargs=-Xmx4096m -XX:MaxMetaspaceSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

如果仍然 OOM，可以继续分析：

- 是否有过多模块参与构建；
- 是否同时开启了太多 worker；
- 是否有生成代码或资源处理任务过重；
- 是否需要升级 Gradle/AGP。

### 如何查看构建耗时

```bash title="build-profile.sh"
./gradlew assembleDebug --profile
```

如果项目允许，也可以使用：

```bash title="build-scan.sh"
./gradlew assembleDebug --scan
```

### 如何跳过测试

```bash title="skip-test.sh"
./gradlew assembleDebug -x test
```

## 总结

Android Gradle 的核心可以拆成几条主线：

1. 项目结构：`settings.gradle` 管模块，根 `build.gradle` 管全局，模块 `build.gradle` 管具体构建。
2. 依赖管理：优先使用 `implementation` 和 Version Catalog，避免动态版本。
3. 构建变体：`Build Type × Product Flavor` 决定最终 Variant。
4. 签名与混淆：敏感信息不要写死，Release 包要关注 R8、资源压缩和 mapping 文件。
5. 性能优化：缓存、并行、Daemon、JVM 参数和依赖解析都会影响构建速度。
6. 日常命令：熟悉 `assemble`、`install`、`dependencies`、`tasks`、`--offline`、`--refresh-dependencies`，能省很多排查时间。

Gradle 一开始看起来很复杂，但它本质上是在描述“项目如何被构建”。只要把配置层级、依赖关系和构建变体理清楚，再复杂的 Android 工程也能慢慢拆开。
