# Agora 服务端录制

本示例代码使用 Agora Recording SDK for Linux 实现语音或视频通话的服务端录制。

## 简介

Agora Recording SDK for Linux 集成在你的 Linux 服务器而非 App 上。

 <div><img src="../../readme_pics/recording_linux_cn.png" height="60%" width="60%"></div>

录制某频道内的音视频信息相当于将一个特殊的观众加入该频道。该观众获取频道内的音视频信息，将获取到的信息转码并储存在 Linux 服务器上。
因此，你必须：

-   将录制 SDK 集成在你的 Linux 服务器上；
-   在录制 SDK 中和进行音视频通话的其他声网 SDK 中使用同一个 APP ID 。
-   指定希望录制的频道。

## 环境配置
### 硬件配置

<table>
  <tr>
    <th>硬件</th>
    <th>要求</th>
  </tr>
  <tr>
    <td>服务器</td>
    <td>物理或虚拟</td>
  </tr>
  <tr>
  	<td>系统</td>
  	<td>Ubuntu Linux 14.04+ LTS 64-bit 或 CentOS 7+ x64</td>
   </tr>
  <tr>
    <td>网络</td>
    <td>这台 Linux 服务器要接入公网，有公网 IP</td>
  </tr>
  <tr>
    <td>网络接入带宽</td>
    <td>根据需要同时录制的频道数量和频道内情况确定所需带宽。以下数据可供参考：录制一个分辨率为 640*480 的画面需要的带宽约为 500kbps；录制一个有两个人的频道则需 1Mbps；同时录制 100 个这样的频道，需要带宽为 100Mbps。</td>
  </tr>
  <tr>
    <td>域名解析</td>
    <td>服务器允许访问 qos.agoralab.co，否则 SDK 无法上报必要的统计数据。</td>
  </tr>
 </table>

参考硬件配置：

<table>
  <tr>
    <th>产品</th>
    <th>描述</th>
    <th>数量</th>
  </tr>
  <tr>
    <td>SUPERMICRO SYS-6017R-TDF</td>
    <td>1U rack-mounted SYS-6017R-TDF (Intel® Xeon® E5-2600 Series Processor)</td>
    <td>1</td>
  </tr>
  <tr>
    <td>机箱</td>
    <td>1U Rackmountable (440-W high-efficiency redundant power supply w/ PMBus)</td>
    <td>1</td>
  </tr>
  <tr>
    <td>处理器</td>
    <td>Intel Xeon E5-2620V2 2.1 G, L3:15M, 6C (P4X-DPE52620V2-SR1AN)</td>
    <td>2</td>
  </tr>
  <tr>
    <td>内存</td>
    <td>MEM-DR380L-HL06-ER16 (8-GB DDR3-1600 2Rx8 1.35-V ECC REG RoHS)</td>
    <td>1</td>
  </tr>
  <tr>
    <td>硬盘</td>
    <td>250-G 3.5 SATA Enterprise (HDD-T0250-WD2503ABYZ)</td>
    <td>2</td>
  </tr>
</table>

假设每个频道内有两个人进行视频通话（通信模式），分辨率是 640*480 ，帧率为 15fps ，单流码率为 500kbps ：
实测在参考硬件配置下， 12 核 24 线程的 CPU 满载并发 100 个频道，此时：

  -   每个频道占用约 25M 内存；总共占用约 2.5G 内存。内存占用率约为 31%；
  -   每个频道写入磁盘的速度约为 60kB/s ；总写入速度约为 6.0MB/s ，远低于磁盘的最大写入速度；
  -   每个频道下行网络流量约为 500kbps * 2 = 1Mbps ，总下行流量约为 100 Mbps ，上行流量可以忽略不计。

### SDK 兼容性

录制 SDK 支持：

  -   纯 Native 端录制；
  -   纯 Web 端录制；
  -   Web 与 Native 互通时录制。

录制 SDK 与以下 SDK 兼容:

  -   Agora Native SDK v1.7.0+.
  -   Agora Web SDK v1.12.0+ .

**Note:** 

      一旦有人在频道中使用了不兼容的 SDK ，则整个频道都无法录制。

## 示例代码运行步骤

### 步骤 1: 准备环境
1.  获取正在进行通信或直播的频道名和使用的 APP ID。
2.  下载 [Agora recording SDK 及示例代码](https://docs.agora.io/en/2.3.1/download).
 
 <div><img src="../../readme_pics/linux_structure.png" height="50%" width="50%"></div>
 
  <table>
    <tbody>
      <tr>
        <td><strong>文件夹</strong></td>
        <td><strong>描述</strong></td>
      </tr>
      <tr>
        <td>bin</td>
        <td>AgoraCoreService 所在的目录</td>
      </tr>
      <tr>
        <td>include</td>
        <td>
          <ul>
            <li>base: libs 所依赖的一些基础的头文件</li>
            <li>IAgoraLinuxSdkCommon.h: 公共的基础结构体和枚举值</li>
            <li>IAgoraRecordingEngine.h: 录制引擎的接口类和配置信息</li>
          </ul>
        </td>
      </tr>
      <tr>
        <td>libs</td>
        <td>录制的依赖库</td>
      </tr>
      <tr>
        <td>samples</td>
        <td>示例代码
          <ul>
            <li>agorasdk: 对录制 C++ 接口的实现以及回调的处理示例</li>
            <li>base: 公共的示例代码</li>
            <li>cpp: C++ 示例代码
              <ul>
                <li>release/bin/recorder: 可运行的父程序</li>
              </ul>
            </li>
            <li>java: java 示例代码
              <ul>
                <li>native: Native code</li>
                <li>native/jni: JNI 代理</li>
                <li>src: java 代码</li>
                <li>src/io/agora/recording/RecordingEventHandler.java: 回调接口类</li>
                <li>src/io/agora/recording/RecordingSDK.java: 录制接口类</li>
              </ul>
            </li>
          </ul>
        </td>
      </tr>
      <tr>
        <td>Tools</td>
        <td>转码工具</td>
      </tr>
    </tbody>
  </table>

3.  打开 TCP 端口：1080、8000
4.  打开 UDP 端口：双向 1080、4000-4030、8000、9700、25000 和 所有的录制进程所使用的单向下行端口

    **Note:** 

      -   录制一个频道的内容需要开启一个对应的录制进程；单个录制进程需要使用 4 个单向下行端口。进程（包括各个录制进程和系统进程）之间不得有端口冲突。

      	- Agora 建议您指定录制进程使用端口的范围。您可以为多个录制进程统一配置较大的端口范围（Agora 建议 40000 ~ 41000 或更大）。此时，录制 SDK 会在指定范围内为每个录制进程分配端口，并避免端口的冲突。要设置端口范围，您需要配置参数 lowUdpPort 和 highUdpPort；
        - 如果不指定参数 lowUdpPort 和 highUdpPort ，录制进程所使用的端口为随机端口，会有端口冲突的风险。

	  -   查看 UDP 端口时可以使用 iptables -L 命令

5.  将域名 .agora.io、vocs.agora.io、qoslbs.agora.io、qos.agora.io 设为白名单；
6.  安装编译器: gcc 4.4+ ；
7.  如果你希望使用 java 编写录制的程序，请确保您已配置好 JDK 环境。

### 步骤 2: 编译代码示例

在 *samples/cpp* 的目录下，进行编译：

1. 进行环境预设置，执行如下命令。其中 *jni_path* 请填入 jni.h 文件绝对路径，如：/usr/java8u161/jdk1.8.0_161/include/

	```
	source build.sh pre_set jni_path
	```

	**Note:** 
	
	要获取 jni.h 文件的本地绝对路径，运行 ```locate jni.h```。

2. 执行编译脚本：

	```
	build.sh build
	```

	编译完成后，同目录下会生成一个名为 *bin* 的文件夹。此文件夹中包含名为 *RecordingSample* 的可执行程序。
	
3. 开始录制

	```
	./recorder_local --appId APP_ID --uid 0 --channel Channel_Name --appliteDir ../../bin
	```



	**Note:** 
	
	-   请用你在音视频通话中使用的 APP ID 替换命令中的 *APP_ID* ；
	-   请用希望录制的频道名替换命令中的 *Channel_Name* 。

	此命令包含了以下信息：
	
	-   –appId APP\_ID 指定了使用的 APP ID；
	-   –uid 0 由 SDK 为录制分配 uid；
	-   –channel Channel\_Name 指明了录制的频道名；
	-   –appliteDir ../../bin 指明了 AgoraCoreService 所在的目录。

	开始录制后，你可以在 *samples/java* 目录下找到以 *日期\_时间戳* 命名的录制文件夹。其中， *时间戳* 是录制开始的时间。

要查看详细的 API 调用参考，请前往 [Recording API](https://docs.agora.io/en/2.3.1/addons/Recording/API%20Reference/recording_java). 你也可以执行 *java RecordingSample* 命令, 查看相关用法。

## 编写示例代码的步骤

示例代码的核心部分是 `main.java`。

### 加载库

``` {.sourceCode .java}
import io.agora.recording.common.*;
import io.agora.recording.common.Common.*;
import java.lang.InterruptedException;
import java.io.FileWriter;
import java.io.IOException;
import java.io.File; 
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.OutputStream; 
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Vector;
import java.util.HashMap;
import java.util.Map;
import java.nio.ByteBuffer;
import java.nio.channels.WritableByteChannel;
import java.nio.channels.Channels;
```

### 定义 AgoraJavaRecording 类

``` {.sourceCode .java}
class AgoraJavaRecording{
...
}
```

### 定义 main 方法

``` {.sourceCode .java}
public static void main(String[] args){
...
}
```

### 定义变量

#### 定义用于 Agora Engine 的变量

  -------------------------------
  变量                 用途
  -------------------- ----------
  `uid`                用户 ID

  `appId`              App ID

  `channelKey`         频道密钥

  `name`               频道名

  `channelProfile`     频道设置
  -------------------------------

``` {.sourceCode .java}
int uid = 0;
String appId = "";
String channelKey = "";
String name = "";
int channelProfile = 0;
```

#### 定义用于录制设置的变量

定义加密模式和密钥。

``` {.sourceCode .java}
String decryptionMode = "";
String secret = "";
```

定义视频合图的分辨率和空闲时间（单位：秒）。

``` {.sourceCode .java}
String mixResolution = "360,640,15,500";
int idleLimitSec=5*60;//300s
```

定义路径。

  ---------------------------------------------------
  变量                    描述
  ----------------------- ---------------------------
  `applitePath`           AgoraCoreService 所在目录

  `recordFileRootDir`     录制文件所在目录

  `cfgFilePath`           配置文件所在目录
  ---------------------------------------------------

``` {.sourceCode .java}
String applitePath = "";
String recordFileRootDir = "";
String cfgFilePath = "";
```

定义最低和最高的 UDP 端口.

``` {.sourceCode .java}
int lowUdpPort = 0;//40000;
int highUdpPort = 0;//40004;
```

定义音视频合图变量。

``` {.sourceCode .java}
boolean isAudioOnly=false;
boolean isVideoOnly=false;
boolean isMixingEnabled=false;
boolean mixedVideoAudio=false;
```

定义音视频流格式。

``` {.sourceCode .java}
int getAudioFrame = AUDIO_FORMAT_TYPE.AUDIO_FORMAT_DEFAULT_TYPE.ordinal();
int getVideoFrame = VIDEO_FORMAT_TYPE.VIDEO_FORMAT_DEFAULT_TYPE.ordinal();
int streamType = REMOTE_VIDEO_STREAM_TYPE.REMOTE_VIDEO_STREAM_HIGH.ordinal();
```

定义截图时间间隔和截图模式。

``` {.sourceCode .java}
int captureInterval = 5;
int triggerMode = 0;
```

定义视频参数。

``` {.sourceCode .java}
int width = 0;
int height = 0;
int fps = 0;
int kbps = 0;
int count = 0;
```

### 检查并读取命令行参数

``` {.sourceCode .java}
if(args.length % 2 !=0){
  System.out.println("command line parameters error, should be '--key value' format!");
  return;
}
String key = "";
String value = "";
Map<String,String> map = new HashMap<String,String>();
if(0 < args.length ){
  for(int i = 0; i<args.length-1; i++){
    key = args[i];
    value = args[i+1];
    map.put(key, value);
  }
}
Object Appid = map.get("--appId");
Object Uid = map.get("--uid");
Object Channel = map.get("--channel");
Object AppliteDir = map.get("--appliteDir");
Object ChannelKey = map.get("--channelKey");
Object ChannelProfile = map.get("--channelProfile");
Object IsAudioOnly = map.get("--isAudioOnly");
Object IsVideoOnly = map.get("--isVideoOnly");
Object IsMixingEnabled = map.get("--isMixingEnabled");
Object MixResolution = map.get("--mixResolution");
Object MixedVideoAudio = map.get("--mixedVideoAudio");
Object DecryptionMode = map.get("--decryptionMode");
Object Secret = map.get("--secret");
Object Idle = map.get("--idle");
Object RecordFileRootDir = map.get("--recordFileRootDir");
Object LowUdpPort = map.get("--lowUdpPort");
Object HighUdpPort = map.get("--highUdpPort");
Object GetAudioFrame = map.get("--getAudioFrame");
Object GetVideoFrame = map.get("--getVideoFrame");
Object CaptureInterval = map.get("--captureInterval");
Object CfgFilePath = map.get("--cfgFilePath");
Object StreamType = map.get("--streamType");
Object TriggerMode = map.get("--triggerMode");
Object ProxyServer = map.get("--proxyServer");
```

### 传入命令行参数

``` {.sourceCode .java}
appId = String.valueOf(Appid);
uid = Integer.parseInt(String.valueOf(Uid));
appId = String.valueOf(Appid);
name = String.valueOf(Channel);
applitePath = String.valueOf(AppliteDir);

if(ChannelKey != null) channelKey = String.valueOf(ChannelKey);
if(ChannelProfile != null) channelProfile = Integer.parseInt(String.valueOf(ChannelProfile));
if(DecryptionMode != null) decryptionMode = String.valueOf(DecryptionMode);
if(Secret != null) secret = String.valueOf(Secret);
if(MixResolution != null) mixResolution = String.valueOf(MixResolution);
if(Idle != null) idleLimitSec = Integer.parseInt(String.valueOf(Idle));
if(RecordFileRootDir != null) recordFileRootDir = String.valueOf(RecordFileRootDir);
if(CfgFilePath != null) cfgFilePath = String.valueOf(CfgFilePath);
if(LowUdpPort != null) lowUdpPort = Integer.parseInt(String.valueOf(LowUdpPort));
if(HighUdpPort != null) highUdpPort = Integer.parseInt(String.valueOf(HighUdpPort));
if(IsAudioOnly != null &&(Integer.parseInt(String.valueOf(IsAudioOnly)) == 1)) isAudioOnly = true;
if(IsVideoOnly != null &&(Integer.parseInt(String.valueOf(IsVideoOnly)) == 1)) isVideoOnly = true;
if(IsMixingEnabled != null &&(Integer.parseInt(String.valueOf(IsMixingEnabled))==1)) isMixingEnabled = true;
if(MixedVideoAudio != null &&(Integer.parseInt(String.valueOf(MixedVideoAudio)) == 1)) mixedVideoAudio = true;
if(GetAudioFrame != null) getAudioFrame = Integer.parseInt(String.valueOf(GetAudioFrame));
if(GetVideoFrame != null) getVideoFrame = Integer.parseInt(String.valueOf(GetVideoFrame));
if(StreamType != null) streamType = Integer.parseInt(String.valueOf(StreamType));
if(CaptureInterval != null) captureInterval = Integer.parseInt(String.valueOf(CaptureInterval));
if(TriggerMode != null) triggerMode = Integer.parseInt(String.valueOf(TriggerMode));
if(ProxyServer != null) proxyServer = String.valueOf(ProxyServer);
```

### 实例化录制类和参数类

``` {.sourceCode .java}
AgoraJavaRecording ars = new AgoraJavaRecording();
RecordingConfig config= new RecordingConfig();
config.channelProfile = CHANNEL_PROFILE_TYPE.values()[channelProfile];
config.idleLimitSec = idleLimitSec;
config.isVideoOnly = isVideoOnly;
config.isAudioOnly = isAudioOnly;
config.isMixingEnabled = isMixingEnabled;
config.mixResolution = mixResolution;
config.mixedVideoAudio = mixedVideoAudio;
config.appliteDir = applitePath;
config.recordFileRootDir = recordFileRootDir;
config.cfgFilePath = cfgFilePath;
config.secret = secret;
config.decryptionMode = decryptionMode;
config.lowUdpPort = lowUdpPort;
config.highUdpPort = highUdpPort;
config.captureInterval = captureInterval;
config.decodeAudio = AUDIO_FORMAT_TYPE.values()[getAudioFrame];
config.decodeVideo = VIDEO_FORMAT_TYPE.values()[getVideoFrame];
config.streamType = REMOTE_VIDEO_STREAM_TYPE.values()[streamType];
config.triggerMode = triggerMode;
config.proxyServer = proxyServer;
```

### 传入合图参数

``` {.sourceCode .java}
ars.isMixMode = isMixingEnabled; 
ars.profile_type = CHANNEL_PROFILE_TYPE.values()[channelProfile];
if(isMixingEnabled && !isAudioOnly) {
  String[] sourceStrArray=mixResolution.split(",");
  if(sourceStrArray.length != 4) {
       System.out.println("Illegal resolution:"+mixResolution);
        return;
    }
    ars.width = Integer.valueOf(sourceStrArray[0]).intValue();
    ars.height = Integer.valueOf(sourceStrArray[1]).intValue();
    ars.fps = Integer.valueOf(sourceStrArray[2]).intValue();
    ars.kbps = Integer.valueOf(sourceStrArray[3]).intValue();
}
```

### 创建并加入频道

``` {.sourceCode .java}
ars.createChannel(appId, channelKey,name,uid,config);
```

## 其他
- 在 [开发者中心](https://docs.agora.io/en/) 查看详细文档;
- [报告示例代码中的错误](https://dashboard.agora.io)。

## 许可证
此示例代码采用 MIT 许可。 [查看详细条款](LICENSE.md)。