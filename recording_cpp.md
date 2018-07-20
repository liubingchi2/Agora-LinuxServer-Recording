# Basic: Enabling Recording {#basic-enabling-recording}

In this quickstart, you will learn how to use the Agora Recording SDK to enable recording.

## Introduction {#introduction .section}

The Agora Recording SDK for Linux \(Recording SDK\) supports:

-   Recording communication and live broadcast content

-   Recording the voice and video of all users in a channel

-   Recording the voice and video of all users in multiple channels simultaneously

-   Recording the voice of all users in a channel or in multiple channels simultaneously

-   Recording an encrypted channel if the application has integrated Agora built-in encryption


The Agora Recording SDK for Linux is integrated on your Linux server instead of your app:

 ![](recording_linux_en.png) 

To record the content of a channel, a ‘special audience’ joins the channel, gets the content and stores the content on a Linux server. You must:

-   Implement the recording SDK on your Linux server;

-   Use the same App ID in the recording SDK and in other Agora SDKs implementing voice or video communication. For detailed information about App ID, see [Getting an App ID](../Agora%20Platform/key_native.md#);

-   Specify the channel to record.


Hardware Requirements
 :   The following table lists the hardware requirements:

 :   | **Hardware**

 | **Requirements**

 |
| Server

 | Physical or virtual:

 -   Ubuntu Linux 14.04+ LTS 64-bit

-   CentOS 7+ x64


 |
| Network

 | The Linux server needs Internet access

 |
| Internet Bandwidth

 | Decide the Internet bandwidth based on the number of channels being recorded simultaneously. Refer to the following data:

 -   When the resolution of the recorded scene is 640\*360, the bandwidth is 500kbps;

-   To record a channel with two users, you need a bandwidth of 1 Mbps;

-   For 100 channels, you need a bandwidth of 100Mbps.


 |
| DNS

 | The server needs access to qos.agoralab.co, or the SDK may fail to report necessary data.

 |

:   Agora recommends the following hardware configurations:

 :   | **Product**

 | **Description**

 | **Number**

 |
| SUPERMICRO SYS-6017R-TDF

 | 1U rack-mounted SYS-6017R-TDF Intel® Xeon® E5-2600 Series Processor

 | 1

 |
| Case

 | 1U Rackmountable \(440-W high-efficiency redundant power supply w/ PMBus\)

 | 1

 |
| Processor

 | Intel Xeon E5-2620V2 2.1 G, L3:15M, 6C \(P4X-DPE52620V2-SR1AN\)

 | 2

 |
| Memory

 | MEM-DR380L-HL06-ER16 \(8-GB DDR3-1600 2Rx8 1.35-V ECC REG RoHS\)

 | 1

 |
| Hard Disk

 | 250-G 3.5 SATA Enterprise \(HDD-T0250-WD2503ABYZ\)

 | 2

 |

:   Assuming two users are in a channel in a video call \(communication mode\), with the resolution of 640\*360, frame rate of 15 fps and bitrate of one video stream of 500kbps:

 :   The CPU is fully loaded and 100 channels are recorded simultaneously:

 :   -   Each channel writes to the disk at a speed of 60 kB/s. The total write-in speed is 6.0 MB/s, which is much lower than the maximum write-in speed of the disk;

-   Each channel uses 25 MB of memory. Thus, 2.5 GB of memory, which is 31% of the total memory, is taken;

-   The downstream Internet flow for each channel is 500 kbps \* 2 = 1 Mbps. The total downstream flow is 100 Mbps. The upstream flow is neglected.


Compatibility with the Agora SDKs
 :   The recording SDK supports:

 :   -   recording the communication that uses the native SDK;

-   recording the communication that uses the web SDK;

-   recording the communication that uses both the native SDK and the web SDK;


:   The recording SDK is compatible with the following Agora SDKs:

 :   | **SDK**

 | **Description**

 |
| The Agora Native SDK

 | The Recording SDK is compatible with the Agora Native SDK v1.7.0+ for all platforms. If any user in the channel uses Agora SDK v1.6, the whole channel cannot record anything

 |
| The Agora Web SDK

 | The Recording SDK is compatible with the Agora Web SDK v1.12.0+

 |

## Step 1: Setting up the environment {#step-1-setting-up-the-environment .section}

1.  Get the channel name and App ID of the communication that you want to record.

2.  [Download](https://docs.agora.io/en/2.3.1/download)

     ![](linux_structure.png) 


| **Folder**

 | **Description**

 |
| bin

 | The directory where AgoraCoreService is located

 |
| include

 | -   base: Required header files for developing the recording application

-   IAgoraLinuxSdkCommon.h: Public structure and enumeration

-   IAgoraRecordingEngine.h: Interface of the recording engine and its config information


 |
| libs

 | Required libraries for developing the recording application

 |
| samples

 | Sample code

 -   agorasdk: Demo that implements the C++ interface and callbacks

-   base: Public sample code

-   cpp: C++ sample code

    -   release/bin/recorder: Parent process that can be run

-   java: Java sample code

    -   native: Native code

    -   native/jni: JNI delegate

    -   src: java code

    -   src/io/agora/recording/RecordingEventHandler.java: Callback interface class

    -   src/io/agora/recording/RecordingSDK.java: Recording interface class


 |
| Tools

 | Transcoding tools

 |

1.  Open the TCP ports 1080 and 8000.

2.  Open the UDP ports:

    -   Duplex ports: 1080, 4000-4030, 8000, 9700 and 25000;

    -   Simplex downstream ports used by the recording processes.


**Note:** 

-   Use the command line *iptables -L* to check the UDP port.

-   To record the content in channels, you need one recording process for each of the channels. One recording process requires four simplex downstream ports. There must be no port conflict among the processes, including the system processes and all the recording processes.

    -   Agora recommends that you specify the range of ports used by the recording processes. Configure a large range for all recording processes \(Agora recommends 40000 ~ 41000 or larger\). If so, the Recording SDK assigns ports to each recording process within the specified range and avoids port conflicts automatically. To set the port range, you need to configure the parameters *lowUdpPort* and *highUdpPort*;

    -   If the parameters, *lowUdpPort* and *highUdpPort*, are not specified, the ports used by the recording processes are at random, which may cause port conflicts.


1.  Set whitelist domains: *.agora.io*, *vocs.agora.io*, *qoslbs.agora.io*, and *qos.agora.io* .

2.  Ensure that your compiler is gcc 4.4+.


## Step 2: Compiling the Sample Code {#step-2-compiling-the-sample-code .section}

To compile the sample code under the *samples/cpp* directory, run the following command:

```
make
```

After the compilation, a *record\_local* application is generated in the directory.

## Step 3: Starting Recording {#recording-commandline .section}

Start recording.

Under the *samples/cpp* directory, run the following command:

```
./recorder_local --appId APP_ID --uid 0 --channel Channel_Name --appliteDir ../../bin
```

**Note:** 

-   Replace *APP\_ID* with the App ID used in communication;

-   Replace *Channel\_Name* with the channel name of the channel to record.


The command specifies the following information:

-   –appId APP\_ID specifies the App ID used in communication to record;

-   –uid 0 allows the SDK to automatically assign a uid for recording;

-   –channel Channel\_Name specifies the name of the channel to record;

-   –appliteDir ../../bin specifies the directory of AgoraCoreService.


After you start recording, you can find folders with a name convention of *date\_timestamp* under the directory of *samples/cpp*.

You can also develop C++ programs to implement recording apart from using command line.

For the detailed API reference, see [Recording API](../API%20Reference/recording_cpp.md). You can also run *./record\_local* command for details.

## Step 4: Other information {#step-4-other-information .section}

After the recording is complete, use the transcoding script to merge the recorded file. For details, see [Using the Transcoding Script](recording_voice_video.md#).

If an error or a warning occurs during the recording, see [Error Reported Callback \(onError\)](../API%20Reference/recording_cpp.md#) and [Warning Reported Callback \(onWarning\)](../API%20Reference/recording_cpp.md#).

