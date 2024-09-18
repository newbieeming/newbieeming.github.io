---
title: 记一次android-10.0.0_r41编译
date: 2024-09-15 03:09
categories:
  - Android
  - AOSP
tags:
  - Linux
  - Git
  - Repo
---

> ubuntu版本: 18.04.6 

参考资料: 

+ [AOSP](https://source.android.google.cn/?hl=zh-cn)
+ [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)
+ [【阳光沙滩】Android Open Source Project入门课程](https://www.bilibili.com/video/BV1y84y1k7be)
+ [【AOSP】手把手教你编译和调试AOSP源码](https://blog.csdn.net/qq_34330286/article/details/137440618)
+ [Android10.0编译 make api-stubs-docs-update-current-api问题](https://blog.csdn.net/amosstan/article/details/122077634)

#### 1.安装必需的软件包 [AOSP 9.0 或更高版本](https://source.android.google.cn/docs/setup/start/requirements?hl=zh-cn)

```sh
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev libc6-dev-i386 x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
```

#### 2.安装必需的软件 [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/)

```sh
sudo apt-get update
```

```sh
# 安装 repo 工具
mkdir ~/bin
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o ~/bin/repo
chmod a+x ~/bin/repo
# 配置环境变量
vim ~/.bashrc
# 添加一下内容
PATH=$PATH:~/bin
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
# 保存退出执行以下命令
source ~/.bashrc
```

#### 3.初始化并且同步代码 [AOSP 开始开发](https://source.android.google.cn/docs/setup/start?hl=zh-cn)

```sh
cd ~
mkdir aosp
cd ~/aosp
# 初始化repo，指定源码分支为 `android-10.0.0_r41`
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r41
# 配置git名称和邮箱
git config --global user.name xx
git config --global user.email xx@xx.xx
# 同步源代码,耗时较久
repo sync -j4
```

#### 4.构建代码

```shell
# 查看处理器架构,我这边是x86_64
xm@ubuntu:~$ arch
x86_64

cd ~/aosp
source build/envsetup.sh
# 输入lunch aosp_x86_64-eng(aosp_$arch-eng) 或直接输入lunch 键入数字选择，如下输入26
xm@ubuntu:~/aosp$ lunch

You're building on Linux

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_blueline-userdebug
     4. aosp_bonito-userdebug
     5. aosp_car_arm-userdebug
     6. aosp_car_arm64-userdebug
     7. aosp_car_x86-userdebug
     8. aosp_car_x86_64-userdebug
     9. aosp_cf_arm64_phone-userdebug
     10. aosp_cf_x86_64_phone-userdebug
     11. aosp_cf_x86_auto-userdebug
     12. aosp_cf_x86_phone-userdebug
     13. aosp_cf_x86_tv-userdebug
     14. aosp_coral-userdebug
     15. aosp_coral_car-userdebug
     16. aosp_crosshatch-userdebug
     17. aosp_crosshatch_car-userdebug
     18. aosp_flame-userdebug
     19. aosp_marlin-userdebug
     20. aosp_sailfish-userdebug
     21. aosp_sargo-userdebug
     22. aosp_taimen-userdebug
     23. aosp_walleye-userdebug
     24. aosp_walleye_test-userdebug
     25. aosp_x86-eng
     26. aosp_x86_64-eng
     27. beagle_x15-userdebug
     28. car_x86_64-userdebug
     29. fuchsia_arm64-eng
     30. fuchsia_x86_64-eng
     31. hikey-userdebug
     32. hikey64_only-userdebug
     33. hikey960-userdebug
     34. hikey960_tv-userdebug
     35. hikey_tv-userdebug
     36. m_e_arm-userdebug
     37. mini_emulator_arm64-userdebug
     38. mini_emulator_x86-userdebug
     39. mini_emulator_x86_64-userdebug
     40. poplar-eng
     41. poplar-user
     42. poplar-userdebug
     43. qemu_trusty_arm64-userdebug
     44. uml-userdebug

Which would you like? [aosp_arm-eng] 26

#使用make -j<number> 命令进行指定线程数编译，也可以使用 m 命令自动选择最大线程数，我这个虚拟机分配了4处理器4核心
make -j12
```

#### 5.虚拟机运行系统（需要开启硬件加速）

> 打开虚拟机中的硬件加速功能 虚拟机->设置->处理器->虚拟化引擎->虚拟化Intel VT-X/EPT或AMD-V/RVI (需要关机后设置)
>
> [在 Linux 上配置虚拟机加速](https://developer.android.com/studio/run/emulator-acceleration?hl=zh-cn#vm-linux)
>
> [阳光沙滩 KVM设置](https://www.sunofbeach.net/a/1244130485534273536)

```shell
# 编译完成后通过emulator在模拟器中启动
cd ~/aosp
emulator
# 找不到emulator命令再重复执行以下命令后执行 1. sourch build/envsetup.sh  2. lunch
```

### 编译遇到的问题

#### 1.编译使用了m 命令自动选择最大线程数

```sh
xm@ubuntu:~/aosp$ m

FAILED: //frameworks/base:api-stubs-docs Metalava Check API [common]
Outputs: out/soong/.intermediates/frameworks/base/api-stubs-docs/android_common/check_last_released_api.timestamp out/soong/.intermediates/frameworks/base/api-stubs-docs/android_common/last_released_baseline.txt
Error: exited with code: 1
******************************
You have tried to change the API from what has been previously approved.

To make these errors go away, you have two choices:
   1. You can add '@hide' javadoc comments to the methods, etc. listed in the
      errors above.

   2. You can update current.txt by executing the following command:
         make test-api-stubs-docs-update-current-api

      To submit the revised current.txt to the main Android repository,
      you will need approval.
******************************
```

#### 1.处理方式：降低编译线程，调整交换分区，参考[Android10.0编译 make api-stubs-docs-update-current-api问题](https://blog.csdn.net/amosstan/article/details/122077634)

```sh
make -j12
```

#### 2. print "'%s' cannot be converted to int" % (line[2])
> File "device/generic/goldfish/tools/mk_combined_img.py", line 48
    print "'%s' cannot be converted to int" % (line[2])
```sh
# python 版本不对
sudo apt install python
```
#### 3.emulator打开失败

```text
emulator 
emulator: WARNING: Couldn't find crash service executable /home/xm/aosp/prebuilts/android-emulator/linux-x86_64/emulator64-crash-service

emulator: WARNING: system partition size adjusted to match image file (3083 MB > 800 MB)

Warning: could not connect to display  ((null):0, (null))
INFO: QtLogger.cpp:66: Warning: could not connect to display  ((null):0, (null))


INFO: QtLogger.cpp:66: Info: Could not load the Qt platform plugin "xcb" in "/home/xm/aosp/prebuilts/android-emulator/linux-x86_64/lib64/qt/plugins" even though it was found. ((null):0, (null))


Fatal: This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb.
 ((null):0, (null))
INFO: QtLogger.cpp:66: Fatal: This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb.
 ((null):0, (null))


已放弃 (核心已转储)
```

尝试增大partition size(依然错误)

```text
emulator partition-size 4096
emulator: WARNING: Couldn't find crash service executable /home/xm/aosp/prebuilts/android-emulator/linux-x86_64/emulator64-crash-service

emulator: WARNING: userdata partition is resized from 2048 M to 4096 M

ERROR: resizing partition failed to launch /home/xm/aosp/prebuilts/android-emulator/linux-x86_64/bin64/resize2fs
Warning: could not connect to display  ((null):0, (null))
INFO: QtLogger.cpp:66: Warning: could not connect to display  ((null):0, (null))


INFO: QtLogger.cpp:66: Info: Could not load the Qt platform plugin "xcb" in "/home/xm/aosp/prebuilts/android-emulator/linux-x86_64/lib64/qt/plugins" even though it was found. ((null):0, (null))


Fatal: This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb.
 ((null):0, (null))
INFO: QtLogger.cpp:66: Fatal: This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: xcb.
 ((null):0, (null))


已放弃 (核心已转储)
```

#### 3.处理方式：打开虚拟机终端执行，以上是在ssh工具执行导致