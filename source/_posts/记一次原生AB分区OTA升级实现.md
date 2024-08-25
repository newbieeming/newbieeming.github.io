---
title: 记一次原生AB分区OTA升级实现
date: 2024-07-12 14:36
categories:
  - Android
tags:
  - OTA	
  - UpdateEngine
---
> 系统需要实现软件ota功能

#### 具体代码实现

```java
UpdateEngine mUpdateEngine = new UpdateEngine();
UpdateParser.ParsedUpdate mParsedUpdate;
try {
    mParsedUpdate = UpdateParser.parse(new File(Environment.getDataDirectory(), "ota_package/update.zip"));
    Log.d(TAG,"mParsedUpdate = " + mParsedUpdate);
} catch (IOException e) {
    throw new RuntimeException(e);
}
if (!mParsedUpdate.isValid()){
    return;
}
mUpdateEngine.bind(new UpdateEngineCallback() {
    @Override
    public void onStatusUpdate(int status, float percent) {
        Log.d(TAG,"onStatusUpdate status = " + status + ",percent = " + percent);
    }

    @Override
    public void onPayloadApplicationComplete(int errorCode) {
        Log.d(TAG,"onPayloadApplicationComplete errorCode = " + errorCode);
    }
});
mUpdateEngine.applyPayload(mParsedUpdate.mUrl, mParsedUpdate.mOffset, mParsedUpdate.mSize, mParsedUpdate.mProps);
```

#### 升级流程

onStatusUpdate回调status变化值，状态码参考[UpdateEngine](http://aospxref.com/android-13.0.0_r3/xref/frameworks/base/core/java/android/os/UpdateEngine.java) UpdateEngine#UpdateStatusConstants、UpdateEngine#ErrorCodeConstants 

> IDLE(0)-UPDATE_AVAILABLE(2)-11-DOWNLOADING (3)-VERIFYING(4)-FINALIZING(5)-UPDATED_NEED_REBOOT(6)

**UpdateStatusConstants**

```java
/**
184       * Status codes for update engine. Values must agree with the ones in
185       * {@code system/update_engine/client_library/include/update_engine/update_status.h}.
186       */
187      public static final class UpdateStatusConstants {
188          /**
189           * Update status code: update engine is in idle state.
190           */
191          public static final int IDLE = 0;
192  
193          /**
194           * Update status code: update engine is checking for update.
195           */
196          public static final int CHECKING_FOR_UPDATE = 1;
197  
198          /**
199           * Update status code: an update is available.
200           */
201          public static final int UPDATE_AVAILABLE = 2;
202  
203          /**
204           * Update status code: update engine is downloading an update.
205           */
206          public static final int DOWNLOADING = 3;
207  
208          /**
209           * Update status code: update engine is verifying an update.
210           */
211          public static final int VERIFYING = 4;
212  
213          /**
214           * Update status code: update engine is finalizing an update.
215           */
216          public static final int FINALIZING = 5;
217  
218          /**
219           * Update status code: an update has been applied and is pending for
220           * reboot.
221           */
222          public static final int UPDATED_NEED_REBOOT = 6;
223  
224          /**
225           * Update status code: update engine is reporting an error event.
226           */
227          public static final int REPORTING_ERROR_EVENT = 7;
228  
229          /**
230           * Update status code: update engine is attempting to rollback an
231           * update.
232           */
233          public static final int ATTEMPTING_ROLLBACK = 8;
234  
235          /**
236           * Update status code: update engine is in disabled state.
237           */
238          public static final int DISABLED = 9;
239      }
```

**ErrorCodeConstants** 

```java
/**
59       * Error codes from update engine upon finishing a call to
60       * {@link applyPayload}. Values will be passed via the callback function
61       * {@link UpdateEngineCallback#onPayloadApplicationComplete}. Values must
62       * agree with the ones in {@code system/update_engine/common/error_code.h}.
63       */
64      public static final class ErrorCodeConstants {
65          /**
66           * Error code: a request finished successfully.
67           */
68          public static final int SUCCESS = 0;
69          /**
70           * Error code: a request failed due to a generic error.
71           */
72          public static final int ERROR = 1;
73          /**
74           * Error code: an update failed to apply due to filesystem copier
75           * error.
76           */
77          public static final int FILESYSTEM_COPIER_ERROR = 4;
78          /**
79           * Error code: an update failed to apply due to an error in running
80           * post-install hooks.
81           */
82          public static final int POST_INSTALL_RUNNER_ERROR = 5;
83          /**
84           * Error code: an update failed to apply due to a mismatching payload.
85           *
86           * <p>For example, the given payload uses a feature that's not
87           * supported by the current update engine.
88           */
89          public static final int PAYLOAD_MISMATCHED_TYPE_ERROR = 6;
90          /**
91           * Error code: an update failed to apply due to an error in opening
92           * devices.
93           */
94          public static final int INSTALL_DEVICE_OPEN_ERROR = 7;
95          /**
96           * Error code: an update failed to apply due to an error in opening
97           * kernel device.
98           */
99          public static final int KERNEL_DEVICE_OPEN_ERROR = 8;
100          /**
101           * Error code: an update failed to apply due to an error in fetching
102           * the payload.
103           *
104           * <p>For example, this could be a result of bad network connection
105           * when streaming an update.
106           */
107          public static final int DOWNLOAD_TRANSFER_ERROR = 9;
108          /**
109           * Error code: an update failed to apply due to a mismatch in payload
110           * hash.
111           *
112           * <p>Update engine does validity checks for the given payload and its
113           * metadata.
114           */
115          public static final int PAYLOAD_HASH_MISMATCH_ERROR = 10;
116  
117          /**
118           * Error code: an update failed to apply due to a mismatch in payload
119           * size.
120           */
121          public static final int PAYLOAD_SIZE_MISMATCH_ERROR = 11;
122  
123          /**
124           * Error code: an update failed to apply due to failing to verify
125           * payload signatures.
126           */
127          public static final int DOWNLOAD_PAYLOAD_VERIFICATION_ERROR = 12;
128  
129          /**
130           * Error code: an update failed to apply due to a downgrade in payload
131           * timestamp.
132           *
133           * <p>The timestamp of a build is encoded into the payload, which will
134           * be enforced during install to prevent downgrading a device.
135           */
136          public static final int PAYLOAD_TIMESTAMP_ERROR = 51;
137  
138          /**
139           * Error code: an update has been applied successfully but the new slot
140           * hasn't been set to active.
141           *
142           * <p>It indicates a successful finish of calling {@link #applyPayload} with
143           * {@code SWITCH_SLOT_ON_REBOOT=0}. See {@link #applyPayload}.
144           */
145          public static final int UPDATED_BUT_NOT_ACTIVE = 52;
146  
147          /**
148           * Error code: there is not enough space on the device to apply the update. User should
149           * be prompted to free up space and re-try the update.
150           *
151           * <p>See {@link UpdateEngine#allocateSpace}.
152           */
153          public static final int NOT_ENOUGH_SPACE = 60;
154  
155          /**
156           * Error code: the device is corrupted and no further updates may be applied.
157           *
158           * <p>See {@link UpdateEngine#cleanupAppliedPayload}.
159           */
160          public static final int DEVICE_CORRUPTED = 61;
161      }
```

#### !!! 注意事项：
[错误码汇总](https://blog.csdn.net/xiaowang_lj/article/details/133387249)
adb remount后会导致失败会通过UpdateEngineCallback#onPayloadApplicationComplete(int errorCode)返回errorCode = 7

remount的解决方式：

```shell
adb root
adb enable-verity
adb reboot
```

#### 参考代码

**UpdateEngine**、**UpdateEngineCallback**可叫系统提供sdk支持

**UpdateParser** 参考android-13.0.0_r3源码[UpdateParser](http://aospxref.com/android-13.0.0_r3/xref/packages/apps/Car/SystemUpdater/src/com/android/car/systemupdater/UpdateParser.java)

```java
import android.util.Log;
import java.io.BufferedReader;
import java.io.File;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.Locale;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;


/** Parse an A/B update zip file. */
public class UpdateParser {

    private static final String TAG = "ROTAUpdateManager";
    private static final String PAYLOAD_BIN_FILE = "payload.bin";
    private static final String PAYLOAD_PROPERTIES = "payload_properties.txt";
    private static final String FILE_URL_PREFIX = "file://";
    private static final int ZIP_FILE_HEADER = 30;

    private UpdateParser() {
    }


    private long getOffset(ZipEntry zipEntry, String str) {
        return (zipEntry.getMethod() != 0 ? 16 : 0) + 30 + str.length() + zipEntry.getCompressedSize() + (zipEntry.getExtra() != null ? zipEntry.getExtra().length : 0);
    }

    /**
     * Parse a zip file containing a system update and return a non null ParsedUpdate.
     */
    public static ParsedUpdate parse(File file) throws IOException {
        long payloadOffset = 0;
        long payloadSize = 0;
        boolean payloadFound = false;
        String[] props = null;

        try (ZipFile zipFile = new ZipFile(file)) {
            Enumeration<? extends ZipEntry> entries = zipFile.entries();
            while (entries.hasMoreElements()) {
                ZipEntry entry = entries.nextElement();
                long fileSize = entry.getCompressedSize();
                if (!payloadFound) {
                    payloadOffset += ZIP_FILE_HEADER + entry.getName().length();
                    if (entry.getExtra() != null) {
                        payloadOffset += entry.getExtra().length;
                    }
                }

                if (entry.isDirectory()) {
                    continue;
                } else if (entry.getName().equals(PAYLOAD_BIN_FILE)) {
                    payloadSize = fileSize;
                    payloadFound = true;
                } else if (entry.getName().equals(PAYLOAD_PROPERTIES)) {
                    try (BufferedReader buffer = new BufferedReader(
                            new InputStreamReader(zipFile.getInputStream(entry)))) {
                        props = buffer.lines().toArray(String[]::new);
                    }
                }
                if (!payloadFound) {
                    payloadOffset += fileSize;
                }

                //if (Log.isLoggable(TAG, Log.DEBUG)) {
                    Log.d(TAG, String.format("Entry %s", entry.getName()));
                //}
            }
        }
        return new ParsedUpdate(file, payloadOffset, payloadSize, props);
    }

    /** Information parsed from an update file. */
    public static class ParsedUpdate {
        public final String mUrl;
        public final long mOffset;
        public final long mSize;
        public final String[] mProps;

        ParsedUpdate(File file, long offset, long size, String[] props) {
            mUrl = FILE_URL_PREFIX + file.getAbsolutePath();
            mOffset = offset;
            mSize = size;
            mProps = props;
        }

        /** Verify the update information is correct. */
        public boolean isValid() {
            return mOffset >= 0 && mSize > 0 && mProps != null;
        }

        @Override
        public String toString() {
            return String.format(Locale.getDefault(),
                    "ParsedUpdate: URL=%s, offset=%d, size=%s, props=%s",
                    mUrl, mOffset, mSize, Arrays.toString(mProps));
        }
    }
}
```
