---
title: Switch 自定义样式
date: 2024-03-27 17:27	
categories:
  - Android
---

最终效果

<img src="https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211709216.png" alt="滑块打开样式" style="zoom: 67%;" />

<img src="https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211710882.png" alt="滑块关闭样式" style="zoom:67%;" />

minHeight，switchMinWidth调整switch开关高度、宽度

android:thumb   开关按钮上原型滑块的样式

android:track  开关按钮下面导轨的样式


```xml
<Switch
        android:layout_width="48dp"
        android:layout_height="24dp"
        android:layout_marginEnd="21dp"
        android:background="@null"
        android:minHeight="24dp"
        android:switchMinWidth="48dp"
        android:thumb="@drawable/selector_switch_thumb"
        android:track="@drawable/selector_switch_track"
        app:layout_constraintBottom_toBottomOf="@+id/tv_br_top"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
```

selector_switch_thumb.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <layer-list>
            <item android:width="18dp" android:height="24dp" android:drawable="@mipmap/ic_track" />
        </layer-list>
    </item>
</selector>
```

selector_switch_track.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_checked="false">
        <layer-list>
            <item android:width="48dp" android:height="24dp" android:drawable="@mipmap/ic_thumb_off" />
        </layer-list>
    </item>
    <item android:state_checked="true">
        <layer-list>
            <item android:width="48dp" android:height="24dp" android:drawable="@mipmap/ic_thumb_on" />
        </layer-list>
    </item>
</selector>
```
对应素材
ic_thumb_off
![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211712944.png)
ic_thumb_on
![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211712605.png)
ic_track
![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211712654.png)