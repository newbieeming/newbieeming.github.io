---
title: AudioRecord 学习
date: 2023-06-23 16:38
categories:
  - Android
tags:
  - AudioRecord
  - 音频
---

## 音频基础

> 声音是一种由物体振动引发的物理现象，如小提琴的弦声等。物体的振动使其四周空气的压强产生变化，这种忽强忽弱变化以波的形式向四周传播，当被人耳所接收时我们就听见了声音。

### 波形

> 声音是由物体的振动产生，这种振动引起了周围空气压强的振荡，我们称这种振荡的函数表现形式为波形

![](https://cdn.jsdelivr.net/gh/xmbest/image/img/202407211716600.png)

### 频率

> 声音的频率的周期的倒数，它表示的是声音在1s内的周期数，单位时赫兹(Hz，1000Hz表示每秒振动1000次

| 声波   | 范围（单位Hz） |
| ------ | -------------- |
| 次声   | 0~20           |
| 人耳   | 20~20K         |
| 超声   | 20K~1G         |
| 特超声 | 1G~10T         |

采样定理：**两倍频率去采集**

麦克风采集模拟世界**20**KHz -> AD -> 计算机采样的2倍的**20**KHz（48KHz、**44.1**KHz）->**DA**转换->喇叭播放

### 采样精度(深度)

> 每个样本点的大小，常用8bit、16bit、24bit

### 通道数

> 单声道、双声道、3、4、6

### 帧

> 音频的概念没那么清晰，视频编码格式可以简单认为一帧就是编码后的一张图像

**帧长**：(1)可以指每帧采样数播放的时间,mp3 48k,1152个采样点，每帧为2ms; aac则是每帧是1024个采样点。 攒够一帧的数据才送去做编码(2) 也可以指压缩后每的数据长度。

每持续时间(秒) = 每采样点数 / 采样频率 (HZ)

### 存储方式

**交错模式**：数据以连续帧的方式进行存放，首先记录帧1的左声道样本和右声道样本再记录下一帧

|  L   |  R   |  L   |  R   |  L   |  R   | ...  |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |

**非交错模式**：记录一个周期内所有帧的左声道样本再记录所有右声道样本

|  L   |  L   |  L   | ...  |  R   |  R   |  R   |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |

## Android音频

### AudioRecord

> 在Android中，使用android.media.AudioRecord可实现灵活的录音配置，录取的是原始的pcm数据

```java
/**
     * Class constructor.
     * Though some invalid parameters will result in an {@link IllegalArgumentException} exception,
     * other errors do not.  Thus you should call {@link #getState()} immediately after construction
     * to confirm that the object is usable.
     * @param audioSource the recording source.
     *   See {@link MediaRecorder.AudioSource} for the recording source definitions.
     * @param sampleRateInHz the sample rate expressed in Hertz. 44100Hz is currently the only
     *   rate that is guaranteed to work on all devices, but other rates such as 22050,
     *   16000, and 11025 may work on some devices.
     *   {@link AudioFormat#SAMPLE_RATE_UNSPECIFIED} means to use a route-dependent value
     *   which is usually the sample rate of the source.
     *   {@link #getSampleRate()} can be used to retrieve the actual sample rate chosen.
     * @param channelConfig describes the configuration of the audio channels.
     *   See {@link AudioFormat#CHANNEL_IN_MONO} and
     *   {@link AudioFormat#CHANNEL_IN_STEREO}.  {@link AudioFormat#CHANNEL_IN_MONO} is guaranteed
     *   to work on all devices.
     * @param audioFormat the format in which the audio data is to be returned.
     *   See {@link AudioFormat#ENCODING_PCM_8BIT}, {@link AudioFormat#ENCODING_PCM_16BIT},
     *   and {@link AudioFormat#ENCODING_PCM_FLOAT}.
     * @param bufferSizeInBytes the total size (in bytes) of the buffer where audio data is written
     *   to during the recording. New audio data can be read from this buffer in smaller chunks
     *   than this size. See {@link #getMinBufferSize(int, int, int)} to determine the minimum
     *   required buffer size for the successful creation of an AudioRecord instance. Using values
     *   smaller than getMinBufferSize() will result in an initialization failure.
     * @throws java.lang.IllegalArgumentException
     */
public AudioRecord(int audioSource, int sampleRateInHz, int channelConfig, int audioFormat, int bufferSizeInBytes)
```

**audioSource**: 音频录制的音源

```java
package android.media;
public class MediaRecorder implements AudioRouting,
                                      AudioRecordingMonitor,
                                      AudioRecordingMonitorClient,
                                      MicrophoneDirection{
   /**
   		* 省略
   */                                   
   public final class AudioSource {
        private AudioSource() {}
        //默认音频源
        public static final int DEFAULT = 0;
       	//设定录音来源为主麦克风
        public static final int MIC = 1;
       	//上行声音
        public static final int VOICE_UPLINK = 2;
       	//下行声音
        public static final int VOICE_DOWNLINK = 3;
        //设定录音来源为语音拨出的语音与对方说话的声音
        public static final int VOICE_CALL = 4;
       	//设定录音来源于同方向的相机麦克风相同，若相机无内置相机或无法识别，则使用预设的麦克风
        public static final int CAMCORDER = 5;
       	//语音识别
        public static final int VOICE_RECOGNITION = 6;
       	//摄像头旁边的麦克风
        public static final int VOICE_COMMUNICATION = 7;
        public static final int REMOTE_SUBMIX = 8;
        public static final int UNPROCESSED = 9;
        public static final int VOICE_PERFORMANCE = 10;
        public static final int ECHO_REFERENCE = 1997;
        public static final int RADIO_TUNER = 1998;
        public static final int HOTWORD = 1999;
        public static final int ULTRASOUND = 2000;
   }   
    /**
   		* 省略
   */                               
}
```

**sampleRateInHz**：音频采样率，即可每秒中采集多少个音频数据

> 44100Hz是目前唯一保证在所有设备上工作的采样率，但其他速率如22050、16000和11025可能在某些设备上无法工作

**channelConfig**: 录音通道，单通道为 AudioFormat.CHANNEL_IN_MONO，双通道为AudioFormat.CHANNEL_IN_STEREO

```java
    public static final int CHANNEL_IN_LEFT = 0x4;
    public static final int CHANNEL_IN_RIGHT = 0x8;
    public static final int CHANNEL_IN_FRONT = 0x10;
    public static final int CHANNEL_IN_BACK = 0x20;
	public static final int CHANNEL_IN_MONO = CHANNEL_IN_FRONT;
    public static final int CHANNEL_IN_STEREO = (CHANNEL_IN_LEFT | CHANNEL_IN_RIGHT);
```

**audioFormat**: 音频每个采样点的位数，即音频的精度，通道选用AudioFormat.ENCODING_PCM_16BIT即pcm 16位即可

```java
public final class AudioFormat implements Parcelable {

    //---------------------------------------------------------
    // Constants
    //--------------------
    /** Invalid audio data format */
    public static final int ENCODING_INVALID = 0;
    /** Default audio data format */
    public static final int ENCODING_DEFAULT = 1;

    // These values must be kept in sync with core/jni/android_media_AudioFormat.h
    // Also sync av/services/audiopolicy/managerdefault/ConfigParsingUtils.h
    /** Audio data format: PCM 16 bit per sample. Guaranteed to be supported by devices. */
    public static final int ENCODING_PCM_16BIT = 2;
    /** Audio data format: PCM 8 bit per sample. Not guaranteed to be supported by devices. */
    public static final int ENCODING_PCM_8BIT = 3;
    /** Audio data format: single-precision floating-point per sample */
    public static final int ENCODING_PCM_FLOAT = 4;
     /**
   		* 省略
   */      
}
```

**bufferSizeInBytes**: 音频数据写入缓冲区的总数，通过 AudioRecord.getMinBufferSize 获取最小的缓冲区。

```java
static public int getMinBufferSize(int sampleRateInHz, int channelConfig, int audioFormat)
```