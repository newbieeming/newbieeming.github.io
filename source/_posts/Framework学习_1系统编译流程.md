---
title: Framework学习_1系统编译流程
date: 2026-02-28 20:48
categories:
  - linux
  - aosp
---



## `AOSP` 构建系统解析: `AndroidProducts.mk` 的识别与加载机制

在 Android 构建系统中，`AndroidProducts.mk` 是定义设备配置的核心文件。本文将详细解析其如何通过 **环境初始化** 和 **宏函数解析** 被系统识别并加载。

------

### 1. 构建环境初始化阶段

**执行 `envsetup.sh` 脚本**

当运行以下命令时：

```bash
source build/envsetup.sh
```

构建系统会执行以下操作：

1. **扫描关键目录**
   递归搜索 `device/`、`vendor/` 等目录下的所有子目录，寻找并执行其中的 `vendorsetup.sh` 文件。

2. **解析 `AndroidProducts.mk` 文件**
   通过以下核心宏函数完成文件定位和变量提取：

   | 宏函数名称                     | 功能描述                                                     |
   | ------------------------------ | ------------------------------------------------------------ |
   | `_find-android-products-files` | 递归搜索 `device/`、`vendor/` 和 `build/target/product/` 目录，定位所有 `AndroidProducts.mk` 文件 |
   | `get-all-product-makefiles`    | 读取每个 `AndroidProducts.mk` 中的 `PRODUCT_MAKEFILES` 变量，收集所有产品配置文件路径 |
   | `get-product-makefiles`        | 根据用户选择的 `TARGET_PRODUCT`，从 `PRODUCT_MAKEFILES` 中筛选对应的产品配置文件 |

------

### 2. 构建配置选择阶段

**执行 `lunch` 命令**

当运行以下命令时：

```bash
lunch aosp_marlin-userdebug
```

构建系统会执行以下关键步骤：

1. **解析产品组合列表**
   从 `AndroidProducts.mk` 中读取 `COMMON_LUNCH_CHOICES` 变量，该变量定义了所有可用的产品组合，例如：

   ```makefile
   # <product_name>-<build_variant>
   COMMON_LUNCH_CHOICES := \
       aosp_marlin-userdebug \
       aosp_sailfish-eng
   ```

2. **拆分产品名称**
   将用户输入的产品名称（如 `aosp_marlin-userdebug`）拆分为：

   - `TARGET_PRODUCT`：设备标识（`aosp_marlin`）
   - `TARGET_BUILD_VARIANT`：构建类型（`userdebug`）

3. **定位产品配置文件**
   根据 `TARGET_PRODUCT`，在 `AndroidProducts.mk` 中找到对应的 `PRODUCT_MAKEFILES` 变量，该变量指向实际的产品配置文件路径，例如：

   ```makefile
   PRODUCT_MAKEFILES := \
       $(LOCAL_DIR)/aosp_marlin.mk
   ```

### 总结

`AOSP` 构建系统通过以下机制实现 `AndroidProducts.mk` 的动态加载：

1. **环境初始化阶段**：通过 `envsetup.sh` 扫描目录并解析宏函数
2. **配置选择阶段**：通过 `lunch` 命令解析产品组合并定位具体配置

这种设计使得 Android 构建系统能够灵活支持多设备、多版本的构建需求。
