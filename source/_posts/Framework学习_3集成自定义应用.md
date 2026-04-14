---
title: Framework学习_3集成自定义应用
date: 2026-03-02 20:48
categories:
  - linux
  - aosp
---



#### **目录结构说明**

```
/vendor/                        
├── vendor_product.mk           
├── apps/                        # 应用统一管理目录
│   ├── product.mk               # 应用列表声明文件
│   ├── FirstDemo/               # 应用目录
│   │   ├── Android.mk           # Make编译配置（可选）
│   │   ├── Android.bp           # Soong编译配置（可选）
│   │   └── FirstDemo.apk        # 应用二进制文件
```

------

### **1. 配置产品级入口文件**

#### **文件路径**：`/vendor/vendor_product.mk`

> 在目标机型配置中继承此文件（示例为x86_64模拟器配置）

```makefile
makefile# 在目标机型配置中导入（如build/target/product/sdk_phone_x86_64.mk）
$(call inherit-product, vendor/vendor_product.mk)
```

#### **文件内容**：

```makefile
makefile# 继承应用列表配置
$(call inherit-product, vendor/apps/product.mk)
```

------

### **2. 声明应用列表**

#### **文件路径**：`/vendor/apps/product.mk`

> 系统会自动扫描`Android.mk`/`Android.bp`模块，此处仅需声明应用名

```makefile
makefile# 声明需要集成的应用列表
PRODUCT_PACKAGES += \
    FirstDemo 
```

------

### **3. 编译配置（两种方式任选其一）**

#### **方式一：Make（Android.mk）**

#### **文件路径**：`/vendor/apps/FirstDemo/Android.mk`

```makefile
makefile# 1. 定义模块路径（必须）
LOCAL_PATH := $(call my-dir)

# 2. 清理变量（必须）
include $(CLEAR_VARS)

# 3. 基础配置
LOCAL_MODULE        := FirstDemo          # 模块名（必须与PRODUCT_PACKAGES一致）
LOCAL_SRC_FILES     := $(LOCAL_MODULE).apk # APK文件名
LOCAL_MODULE_TAGS   := optional           # 编译所有版本（user/eng/tests）
LOCAL_MODULE_CLASS  := APPS               # 模块类型（应用）
LOCAL_CERTIFICATE   := PRESIGNED          # 保留APK原有签名
LOCAL_VENDOR_MODULE := true               # 安装到/vendor分区
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX) # 自动补全.apk后缀

# 4. 包含编译规则（必须）
include $(BUILD_PREBUILT)
```

#### **方式二：Soong（Android.bp）**

#### **文件路径**：`/vendor/apps/FirstDemo/Android.bp`

```bp
bpandroid_app_import {
    name: "FirstDemo",                   // 模块名（必须与PRODUCT_PACKAGES一致）
    apk: "FirstDemo.apk",                // APK文件名
    presigned: true,                      // 保留APK原有签名
    proprietary: true,                    // 安装到/vendor分区（推荐）
    
    // 可选配置（根据需求启用）：
    // privileged: true,                  // 安装到/system/priv-app（需系统权限）
    // product_specific: true,            // 安装到/product分区
    // system_ext_specific: true		  // 安装到system_ext分区
    // device_specific: true              // 安装到odm分区
    dex_preopt: { enabled: false },    // 禁用DEX预优化（三方APK推荐）
}
```

通过adb命令参考当前apk路径

```shell
~/aosp/out/host/linux-x86/bin$ ./adb shell pm path me.xmbest.firstdemo2
#结果 package:/vendor/app/FirstDemo2/FirstDemo2.apk
```

