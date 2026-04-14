---
title: Framework学习_2添加product
date: 2026-03-01 20:48
categories:
  - linux
  - aosp
---



#### 添加Product

> 配置文件，类似应用开发中的build.gradle中的渠道包，用于将系统编译成不同的镜像文件

```shell
# 模拟器相关的produ文件
build/target 
# 实际硬件相关的product文件
device/

aosp_x86_64-eng:
build/target/borad/generic_x86_64/BroadConfig.mk
build/target/product/AndroidProducts.mk
build/target/product/aosp_x86_64.mk
```

在device目录下新增**厂商/机型**目录，如下：

```shell
# 产品配置文件
device/xiaomi/pandora/pandora.mk 
# 定义硬件相关的底层特性、变量，当前源码支持的架构、WiFi、摄像头、内核、启动器
device/xiaomi/pandora/BroadConfig.mk
# 执行 lunch 展示的就是该处定义的
device/xiaomi/pandora/AndroidProducts.mk
```

新增完后 执行**lunch**命令查看列表是否存在并且正确编译

#### 参考文件：

**AndroidProducts.mk**

>  参考build/target/product/AndroidProducts.mk

```makefile
# 指向产品配置文件（如device.mk或自定义.mk文件）
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/pandora.mk \
# 在AndroidProducts.mk中定义，列出所有可用的产品组合（格式为<product_name>-<build_variant>）。
COMMON_LUNCH_CHOICES := \
    pandora-eng \
    pandora-userdebug \
    pandora-user \

```

**pandora.mk** 

> 复制 build/target/product/aosp_x86_64.mk，注释了PRODUCT_ENFORCE_ARTIFACT_PATH_REQUIREMENTS := relaxed上下三行

```makefile
#
# Copyright 2013 The Android Open-Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

PRODUCT_USE_DYNAMIC_PARTITIONS := true

# The system image of aosp_x86_64-userdebug is a GSI for the devices with:
# - x86 64 bits user space
# - 64 bits binder interface
# - system-as-root
# - VNDK enforcement
# - compatible property override enabled

# This is a build configuration for a full-featured build of the
# Open-Source part of the tree. It's geared toward a US-centric
# build quite specifically for the emulator, and might not be
# entirely appropriate to inherit from for on-device configurations.

#
# All components inherited here go to system image
#
$(call inherit-product, $(SRC_TARGET_DIR)/product/core_64_bit.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/generic_system.mk)

# Enable mainline checking for excat this product name
# ifeq (aosp_x86_64,$(TARGET_PRODUCT))
# PRODUCT_ENFORCE_ARTIFACT_PATH_REQUIREMENTS := relaxed
# endif

#
# All components inherited here go to system_ext image
#
$(call inherit-product, $(SRC_TARGET_DIR)/product/handheld_system_ext.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/telephony_system_ext.mk)

#
# All components inherited here go to product image
#
$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_product.mk)

#
# All components inherited here go to vendor image
#
$(call inherit-product-if-exists, device/generic/goldfish/x86_64-vendor.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/emulator_vendor.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/board/generic_x86_64/device.mk)

#
# Special settings for GSI releasing
#
ifeq (aosp_x86_64,$(TARGET_PRODUCT))
$(call inherit-product, $(SRC_TARGET_DIR)/product/gsi_release.mk)
endif


PRODUCT_NAME := pandora
PRODUCT_DEVICE := generic_x86_64
PRODUCT_BRAND := Android
PRODUCT_MODEL := AOSP on x86_64
```

**BroadConfig.mk**

> 位于`device/厂商/设备/`目录下，定义硬件相关的配置（如CPU架构、内核路径等）。
>
> 复制l了build/target/borad/generic_x86_64/BroadConfig.mk

```makefile
# Copyright (C) 2018 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# x86_64 emulator specific definitions
TARGET_CPU_ABI := x86_64
TARGET_ARCH := x86_64
TARGET_ARCH_VARIANT := x86_64

TARGET_2ND_CPU_ABI := x86
TARGET_2ND_ARCH := x86
TARGET_2ND_ARCH_VARIANT := x86_64

include build/make/target/board/BoardConfigGsiCommon.mk

ifdef BUILDING_GSI
include build/make/target/board/BoardConfigGkiCommon.mk

BOARD_KERNEL-5.4_BOOTIMAGE_PARTITION_SIZE := 67108864
BOARD_KERNEL-5.4-ALLSYMS_BOOTIMAGE_PARTITION_SIZE := 67108864
BOARD_KERNEL-5.10_BOOTIMAGE_PARTITION_SIZE := 67108864
BOARD_KERNEL-5.10-ALLSYMS_BOOTIMAGE_PARTITION_SIZE := 67108864

BOARD_USERDATAIMAGE_PARTITION_SIZE := 576716800

BOARD_KERNEL_BINARIES := \
    kernel-5.4 \
    kernel-5.10 \

ifneq (,$(filter userdebug eng,$(TARGET_BUILD_VARIANT)))
BOARD_KERNEL_BINARIES += \
    kernel-5.4-allsyms \
    kernel-5.10-allsyms \

endif

else # BUILDING_GSI
include build/make/target/board/BoardConfigEmuCommon.mk

BOARD_USERDATAIMAGE_PARTITION_SIZE := 576716800

BOARD_SEPOLICY_DIRS += device/generic/goldfish/sepolicy/x86

# Wifi.
BOARD_WLAN_DEVICE           := emulator
BOARD_HOSTAPD_DRIVER        := NL80211
BOARD_WPA_SUPPLICANT_DRIVER := NL80211
BOARD_HOSTAPD_PRIVATE_LIB   := lib_driver_cmd_simulated
BOARD_WPA_SUPPLICANT_PRIVATE_LIB := lib_driver_cmd_simulated
WPA_SUPPLICANT_VERSION      := VER_0_8_X
WIFI_DRIVER_FW_PATH_PARAM   := "/dev/null"
WIFI_DRIVER_FW_PATH_STA     := "/dev/null"
WIFI_DRIVER_FW_PATH_AP      := "/dev/null"

endif # BUILDING_GSI

```

