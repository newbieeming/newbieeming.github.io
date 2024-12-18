---
title: android常见操作
date: 2024-12-18 20:48
categories:
  - Android
  - gradle
  - kts
---
### android 清单文件添加标识

1. 清单文件添加对应配置

```xml
 <meta-data android:name="commit_id"
            android:value="${commit_id}"/>
```

2. build.gradle 添加

```kotlin
buildTypes {
        release {
            // ...
            manifestPlaceholders["commit_id"] = getCommitID()
        }
        //...
}
```

### gradle 常用方法

#### kts 版本

```kotlin
// 获取发布时间
fun releaseTime(): String {
    val sdf = SimpleDateFormat("yyMMddHHmmss")
    sdf.timeZone = TimeZone.getTimeZone("GMT+08:00")
    return sdf.format(Date())
}

// 获取用户名
fun getUserName(): String {
    return Runtime.getRuntime().exec("git config user.name").inputStream.bufferedReader().use(
        BufferedReader::readText
    ).trim()
}

// 获取当前分支提交ID
fun getCommitID(): String {
    return Runtime.getRuntime().exec("git rev-parse --short HEAD").inputStream.bufferedReader()
        .use(BufferedReader::readText).trim()
}

// 获取当前分支名
fun getBranch(): String {
    val process = Runtime.getRuntime().exec("git rev-parse --abbrev-ref HEAD")
    val branch = process.inputStream.bufferedReader().use(BufferedReader::readText).trim()
    process.waitFor()
    if (process.exitValue() != 0) {
        println(process.errorStream.bufferedReader().use(BufferedReader::readText))
    }
    return branch
}
```

#### groovy 版本

```groovy

static def releaseTime() {
    return new Date().format("yyMMddHHmm", TimeZone.getTimeZone("GMT+08:00"))
}

static def getUserName(){
    return 'git config user.name'.execute().text.trim()
}

//获取当前分支提交ID
static def getCommitID() {
    return 'git rev-parse --short HEAD'.execute().text.trim()
}

//获取当前分支名
static def getBranch() {
    def branch = ""
    def proc = "git rev-parse --abbrev-ref HEAD".execute()
    proc.in.eachLine { line -> branch = line }
    proc.err.eachLine { line -> println line }
    proc.waitFor()
    branch
}
```

### 修改应用打包名称

#### kts 版本

```groovy
buildTypes {
        release {
						// ...
            applicationVariants.all {
                outputs.all {
                    if (this is com.android.build.gradle.internal.api.ApkVariantOutputImpl) {
                        this.outputFileName =
                            "xxx_v${versionName}_${getCommitID()}_${releaseTime()}.apk"
                    }
                }
            }
        }
				// ... 
}
```

#### groovy 版本

```groovy
buildTypes {
        release {
						// ...
            android.applicationVariants.all { variant ->
                variant.outputs.all {
                    outputFileName = "xxx_v" + versionName + ".apk"
                }
            }
        }
  			// ...
}
```

### 多环境打包

> 简单配置

1. 清单文件添加标识

```xml
<meta-data
            android:name="ENV"
            android:value="${ENV}" />
```

2. build.gradle 配置

```groovy
android{
  // ...
  flavorDimensions += arrayOf("env")
  productFlavors{
        create("dev"){
            dimension = "env"
            manifestPlaceholders["ENV"] = "dev"
        }
        create("test"){
            dimension = "env"
            manifestPlaceholders["ENV"] = "test"
        }
    		create("prop"){
            dimension = "env"
            manifestPlaceholders["ENV"] = "prop"
        }
    }
  // ...
}
```

#### 获取当前环境

##### 代码方式

```kotlin
fun getChannel():String {
    val applicationInfo = App.get().packageManager.getApplicationInfo(
                App.get().packageName,
                PackageManager.GET_META_DATA)
    return applicationInfo.metaData.getString("ENV") ?: ""
}
```

可通过getChannel 来区分环境，例如切换不同服务器地址

