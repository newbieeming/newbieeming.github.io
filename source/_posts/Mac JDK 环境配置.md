---
title: Mac JDK环境配置
date: 2023-07-21 23:36
categories:
  - Mac
---



### 安装JDK

1. 进入azul下载对应版本、安装

```http
https://www.azul.com/downloads/
```

2. 查看安装路径

```shell
/usr/libexec/java_home -V
# 输出以下内容
Matching Java Virtual Machines (2):
    16.0.2 (arm64) "Azul Systems, Inc." - "Zulu 16.32.15" /Users/xiaoming/Library/Java/JavaVirtualMachines/azul-16.0.2/Contents/Home
    1.8.0_382 (arm64) "Azul Systems, Inc." - "Zulu 8.72.0.17" /Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
/Users/xiaoming/Library/Java/JavaVirtualMachines/azul-16.0.2/Contents/Home
```

3. 配置环境变量

```shell
vim ~/.bash_profile
# 按i键进入输入模式粘贴以下内容
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
PATH=$PATH:$JAVA_HOME/bin/:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export PATH
export CLASSPATH
# 按ESC进入命令模式输入以下内容
:wq
# 生效该配置文件
source ~/.bash_profile
```

4. 发现退出终端再打开时输入常用命令不识别，且输出以下内容

```shell
zsh: command not found: ls
```

```shell
vim ~/.zshrc
# 按i键进入输入模式粘贴以下内容
source ~/.bash_profile
# 按ESC进入命令模式输入以下内容
:wq
# 生效该配置文件
source ~/.bash_profile
```

### 安装Maven

1. 下载maven压缩包解压到指定目录
2. 配置环境变量
3. mvn -v 查看是否成功

```shell
# ~/.bash_profile内容
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
M2_HOME=/Users/xiaoming/environment/apache-maven-3.9.3
PATH=$PATH:$JAVA_HOME/bin/:$M2_HOME/bin:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export JAVA_HOME
export M2_HOME
export PATH
export CLASSPATH
```

```shell
# 当前~/.bash_profile
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
SDK_HOME=/Users/xiaoming/Library/Android/sdk
FLUTTER=/Users/xiaoming/Environment/flutter
M2_HOME=/Users/xiaoming/Environment/apache-maven-3.9.3
GIT=/opt/homebrew/bin/git
PUB_HOSTED_URL="https://pub.flutter-io.cn"
FLUTTER_STORAGE_BASE_URL="https://storage.flutter-io.cn"
PATH=$PATH:$JAVA_HOME/bin/:$FLUTTER/bin/:$SDK_HOME/platform-tools/:$SDK_HOME/build-tools/30.0.3/:$M2_HOME/bin:$GIT/bin:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
export PUB_HOSTED_URL
export FLUTTER_STORAGE_BASE_URL
export JAVA_HOME
export SDK_HOME
export M2_HOME
export GIT
export PATH
export CLASSPATH
```