---
title: 记一次WIFI 密码错误判断实现
date: 2024-07-09 15:22
categories:
  - Android
tags:
  - WIFI
---

需求： wifi密码错误、弹出提示

在网上找了半天大部分都是以下方式实现的

```java
public class NetworkStateReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (action.equals(WifiManager.SUPPLICANT_CONNECTION_CHANGE_ACTION)) {
            SupplicantState supState = intent.getParcelableExtra(WifiManager.EXTRA_NEW_STATE);
            if (supState == SupplicantState.DISCONNECTED) {
                int error = intent.getIntExtra(WifiManager.EXTRA_SUPPLICANT_ERROR, -1);
                if (error == WifiManager.ERROR_AUTHENTICATING) {
                    Toast.makeText(context, "WiFi密码错误", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }
}
```

经测试验证非密码错误也会走到里面

参考博客[Android Q 学习WiFi AP的禁用](https://blog.csdn.net/sinat_20059415/article/details/103534598 )后找了WifiConfiguration源码查看 [WifiConfiguration](http://aospxref.com/android-13.0.0_r3/xref/packages/modules/Wifi/framework/java/android/net/wifi/WifiConfiguration.java) **Status**

```java
    /** Possible status of a network configuration. */
472      public static class Status {
473          private Status() { }
474  
475          /** this is the network we are currently connected to */
476          public static final int CURRENT = 0;
477          /** supplicant will not attempt to use this network */
478          public static final int DISABLED = 1;
479          /** supplicant will consider this network available for association */
480          public static final int ENABLED = 2;
481  
482          public static final String[] strings = { "current", "disabled", "enabled" };
483      }
484  
```

解决方式：**在收到对应广播后判断下status**

```kotlin
val wifi = WiFiUtil.getInstance(wifiManager).everConnected(wifiName)
if (wifi.status != WifiConfiguration.Status.DISABLED) {
	// 如果WifiConfiguration.Status.DISABLED说明没连接上过
    LogUtil.d("everConnected.status != WifiConfiguration.Status.DISABLED")
    return
}
```

工具方法

```java
public WifiConfiguration everConnected(String ssid) {
    List<WifiConfiguration> existingConfigs = mWifiManager.getConfiguredNetworks();
    if (existingConfigs == null || existingConfigs.isEmpty()) {
        return null;
    }
    ssid = "\"" + ssid + "\"";
    for (WifiConfiguration existingConfig : existingConfigs) {
        if (existingConfig.SSID.equals(ssid)) {
            return existingConfig;
        }
    }
    return null;
}
```
