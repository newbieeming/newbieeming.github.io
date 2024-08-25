---
title: Gradle失败解决方式
date: 2023-12-31 22:14
categories:
  - Gradle
---

### 解决网络等问题导致Android Studio 下载 Gradle失败

**腾讯镜像**：https://mirrors.cloud.tencent.com/gradle/
**阿里镜像**：https://mirrors.aliyun.com/gradle/
选择以上镜像，下载项目对应版本 gradle.zip

```shell
# 切换到gradle 目录
cd /Users/用户名/.gradle/wrapper/dists
```
如下图所示
![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211714407.png)
找到对应版本，进入最内层目录将以上下载的 gradle.zip复制并解压到当前目录，如下图所示
![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211715702.png)
 Android Studio 同步后如下图所示
![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211715434.png)