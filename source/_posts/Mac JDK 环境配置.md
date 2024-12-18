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
vim ~/.zshrc
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
source ~/.zshrc
```
4. 查看是否配置成功
```shell
java -version 
```

5. 已配置的环境变量（备份）
```shell
# cat ~/.zshrc
#jdk path
JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-8.jdk/Contents/Home
# android sdk path
SDK_HOME=/Users/xm/Library/Android/sdk
# custome script
SCRIPT_HOME=/Users/xm/Documents/Script
# git path
GIT_HOME=/opt/homebrew/Cellar/git/2.46.0
# maven path
M2_HOME=/Users/xm/Library/env/apache-maven-3.9.9
# jadx path
JADX_HOME=/Users/xm/Library/env/jadx-1.4.7
# classpath
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.
# echo $PATH 如果重复，重启终端
PATH=$PATH:$JAVA_HOME/bin:$SDK_HOME/platform-tools:$SDK_HOME/build-tools/35.0.0:$GIT_HOME/bin:$M2_HOME/bin:$JADX_HOME/bin:$SCRIPT_HOME:$TXZ_GIT_ROOT
# python3 alias
alias python="/opt/homebrew/Cellar/python@3.12/3.12.5/Frameworks/Python.framework/Versions/3.12/bin/python3"
alias pip="/opt/homebrew/Cellar/python@3.12/3.12.5/Frameworks/Python.framework/Versions/3.12/bin/pip3"
export JAVA_HOME
export PATH
export CLASSPATH
export M2_HOME
# fnm环境配置
eval "$(fnm env --use-on-cd)"
# fnm 阿里镜像
export FNM_NODE_DIST_MIRROR=https://mirrors.aliyun.com/nodejs-release/
```