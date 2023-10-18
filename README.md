# MCxxx SDK User Manual V1.3

# 1. SDK Architecture introduction

## 1.1 SDK Introduction to publishing directory structure

| Primary directory | Secondary directory |
| --- | --- |
| app\_T8008 | | |-- appfs # Application upgrade package directory| |-- bin # Store compiled temporary executable program| |-- hardware # Hardware related process source code| |-- lib # Store the compiled temporary static library and dynamic library| |-- release # Store boot, kernel, file system, application program and release upgrade program |
| Public | | |-- avsdk # Video socket service for local real-time and historical video acquisition | |-- comm\_app # Common business libraries that can be called by different business processes| |-- comm\_convert # Video export conversion| |-- comm\_middle # General C library encapsulation, open source code interface, etc| |-- config # Equipment parameters| |-- net # Network service| | |-- dvrnet # TTX network lib| | |-- jt\_gps # jt808 network source code，jtgps process| | |-- ttx # ttx api interface，ttxnet process| |-- pubbusiness # Alarm and serial port peripherals| | |-- alarm # Alarm service| | |-- basic # Power management, timing and other basic business Libraries| | |-- interface # interface| | |-- serial # Serial peripherals| |-- pubio # GPIO | |-- recfile # | | |-- aacmng # AAC audio| | |-- asfmng # asf file| | |-- mp4mng # mp4 file| | |-- avimng # avi file| |-- storemng # recording store| | |-- disk\_access # recording access| | |-- disk\_record # recording | |-- tools # tools| | |-- GpioApp # GPIO process for debug| | |-- mkdosfs # format| | |-- sqlite3.8.6 # sqlite lib | |-- upgrade # upgrade| |-- uploadfile # FTP |

## 1.2 Compiler installation

Before compiling the SDK program released by us, you need to install the cross compiler first. Please operate in combination with the compiler installation instructions. Please use the user with root permission to install。

## 1.3 Compilation description

Unzip the SDK package and enter the app\_ T8008 directory, compile and release according to the following commands。

1）Standard product debugging software compilation (the software does not conduct clean operation)

make appbuild

2）Release and compilation of standard product version

make VERSION=T21080401

3) Product specific version compilation

make VERSION=T21080401 USRODM=XXX

Note: USRODM is added as needed. It is not required by default。

## 1.4 How to add one's own process

In the appfs directory, there is an XML file with the name: process XML, for example: add a process in the secondary file for management. The added contents are as follows：

\<task\>

\<name\>myprocess\</name\> /\*the name of process\*/

\<path\>/app/myprocess\</path\> /\*exe directory\*/

\<arg\>\</arg\> /\*para\*/

\<switch\>enable\</switch\> /\ ***enable—start process**** ， ****disable—don't start process** \*/

\<start\_inval\>3\</start\_inval\> /\*start interval\*/

\<feeddog\_inval\>1000\</feeddog\_inval\> /\*timeout for feeddog\*/

\<max\_starttimes\>40\</max\_starttimes\> /\* Maximum startup times after startup \*/

\</task\>

The contents are as follows:：

![](RackMultipart20231018-1-8z8zy5_html_685caa226b8c06f1.png)

MakefilePlease refer to the makefile in sdkdemo to modify it. In app\_XXXX Add a line to the makefile：USER\_LIB += $(LIBDIR)/sdkdemo

# 2. SDK instructions

## 2.1 Key header file description

- StatusMngInterface.h

This file mainly implements the definition of some system state structures, and implements two important function interfaces: setting and obtaining。

- EventMngInterface.h

This file mainly realizes the structure definition used in interprocess communication and the function declaration called by interprocess communication.

- LogMngInterface.h

This file mainly realizes the calling interface of debugging log and user log；

- avsdk\_interface.h

This file is mainly used to define the interface for real-time video preview data acquisition and historical video search playback；

- comm\_dog.h

The definition of dongle interface is mainly used to judge whether the thread inside the process feeds the dog and whether the thread feeds the dog；

-

## 2.2 Interprocess communication

1. Register the events to be received. Refer to the event enumeration type：EVENT\_E，in the header file : EventMngInterface.h.
2. Start event receiving, call the function startunixdomainlisten, and focus on the implementation of the event callback function SDK\_ domain\_ comm\_ proc； Be careful not to do blocking business in the event callback function；
3. The event callback function is implemented by referring to the sample code：sdk\_domain\_comm\_proc
4. Sample code：

void\* SdkStartUnixDomain()

{

long long EventBitmap = 0;

char tmpDomain[128] = {0};

setbit(&EventBitmap, EVENT\_ALARM);

setbit(&EventBitmap, EVENT\_USR\_DEF4);

setbit(&EventBitmap, EVENT\_CAN\_CONTROL);

sprintf(tmpDomain, "/tmp/\_%u\_sdk\_event.listen", getpid());

UnregisterEventReciever((char\*)tmpDomain);

RegisterEventReciever((char\*)tmpDomain, EventBitmap);

return StartUnixDomainListen((char\*)tmpDomain, sdk\_domain\_comm\_proc, NULL);

}

## 2.3 Status acquisition

The status is obtained mainly through the function GetStatus，the specific status to obtain needs to be determined according to enumeration STATUS\_E，header file：StatusMngInterface.h。

Status setting is mainly completed through the function setstatus，Reference structure definition :STATUS\_E。

Note: when setting parameters, you need to obtain them first, then modify them, and then call the setting function to set them to avoid modifying other data contents。

### 2.3.1 GPS data acquisition

### 2.3.2 IO data acquisition

GPSData\_s GPSData = {0};

GetStatus(STATUS\_GPS, (void\*)&GPSData);

### 2.3.3 Get dialing status

WirelessStatus\_t m\_wireless;

GetStatus(STATUS\_WIRELESS, (void\*)& m\_wireless);

### 2.3.4 Get disk status

STATUS\_DISK\_STATUS\_t m\_disk;

GetStatus(STATUS\_DISK\_STATUS, (void\*)& m\_disk);

### 2.3.5 Get peripheral status（IO、ACC、Power,etc）

PeripheraStatus\_t m\_periphera;

GetStatus (STATUS\_PERIPHERA, (void\*)& periphera);


## 2.4 Live video preview

### 2.4.1 Start live preview

Calling function：AVSDK AvSdkStartRealStream(RealAVCmd\_t\* pRealAVCmd)

Reference file：Avsdk\_interface.h

Sample:

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

//Start audio and video coding. This interface will be called when the server requests real-time video and sound monitoring

//You also need to know the audio format parameters

//@Nmediatype: media type, TTX\_ MEDIA\_ TYPE\_ AV or TTX\_ MEDIA\_ TYPE\_ AUDIO

//@Nchannel: the channel number starts from 0. If it is audio, nchannel can also be a microphone channel

//@Nstreamtype: bitstream type, valid when the media type is video

//@Pfnencodercb: Data callback function

//@Pencusr: user defined data of encoding callback function

//@Ppavusr: the encoding object is user-defined data, which is returned to the network library. In stopavenc, this parameter is returned

//The upper layer stores pfnencodercb and pusr, and tells the network layer through this interface when encoding data

//The upper layer transmits the audio and video data to the network library in order

//The network library module does not send audio data, but the client requests audio before sending audio\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

bool CDevNetInterface::StartAVEnc(int nMediaType, int nChannel, int nStreamType, TTXPfnAVEncoderCB pfnEncoderCB, void\* pEncUsr, void\*\* ppAVUsr)

{

TTXFrameCb\_t\* pTTXFrameCb = new TTXFrameCb\_t;

if(pTTXFrameCb == NULL)

{

return false;

}

pTTXFrameCb-\>pfnEncoderCB = pfnEncoderCB;

pTTXFrameCb-\>pEncUsr = pEncUsr;

TTXAVEnc\_t\* pTTXAVEnc = new TTXAVEnc\_t;

if(pTTXAVEnc == NULL)

{

delete pTTXFrameCb;

return false;

}

//RealAVCmd\_t RealAVCmd;

pTTXAVEnc-\>RealAVCmd.chn = nChannel + 1;

if(nMediaType == 0)

{

pTTXAVEnc-\>RealAVCmd.type = 0;

pTTXAVEnc-\>RealAVCmd.streamtype = nStreamType;

//调整预览IPC码流类型使用对应IPC通道类型 2020.09.02 ssx

ProductInfo\_t ProductInfo = {0};

GetStatus(STATUS\_DEV\_TYPE, (void\*)&ProductInfo);

if(nStreamType == 0)

{

RecordSet\_t RecSet = {0};

memset(&RecSet, 0, sizeof(RecSet));

CConfig::Instance()-\>GetConfig(&RecSet);

int audiotype = TTX\_AUDIO\_TYPE\_G726\_40KBPS;

if(RecSet.AudioType == 0)

{

audiotype = PLAY\_A\_TYPE\_PCM\_8K;

}

else if(RecSet.AudioType == 1)

{

audiotype = TTX\_AUDIO\_TYPE\_G711A;

}

if(nChannel \< ProductInfo.VChnCount)//AHD 通道

{

if(RecSet.MainEncType == 0) //h264

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H264);

}

else if(RecSet.MainEncType == 1) //h265

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H265);

}

}

else //IPC 通道

{

pTTXAVEnc-\>RealAVCmd.streamtype = 1; //IPC Preview sub stream 2020.10.15 ssx

if(RecSet.IPCSet[nChannel - ProductInfo.VChnCount].IpcVtype == 0) //h264

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H264);

}

else if(RecSet.IPCSet[nChannel - ProductInfo.VChnCount].IpcVtype == 1) //h265

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H265);

}

}

}

else if(nStreamType == 1)

{

RecordSet\_t RecSet = {0};

memset(&RecSet, 0, sizeof(RecSet));

CConfig::Instance()-\>GetConfig(&RecSet);

int audiotype = TTX\_AUDIO\_TYPE\_G726\_40KBPS;

if(RecSet.AudioType == 0)

{

audiotype = PLAY\_A\_TYPE\_PCM\_8K;

}

else if(RecSet.AudioType == 1)

{

audiotype = TTX\_AUDIO\_TYPE\_G711A;

}

if(nChannel \< ProductInfo.VChnCount)//AHD 通道

{

if(RecSet.SubEncType == 0) //h264

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H264);

}

else if(RecSet.SubEncType == 1) //h265

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H265);

}

}

else //IPC 通道

{

if(RecSet.IPCSet[nChannel - ProductInfo.VChnCount].IpcVtype == 0) //h264

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H264);

}

else if(RecSet.IPCSet[nChannel - ProductInfo.VChnCount].IpcVtype == 1) //h265

{

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H265);

}

}

}

}

else

{

pTTXAVEnc-\>RealAVCmd.type = 2;

pTTXAVEnc-\>RealAVCmd.streamtype = 0;

RecordSet\_t RecSet = {0};

memset(&RecSet, 0, sizeof(RecSet));

CConfig::Instance()-\>GetConfig(&RecSet);

int audiotype = TTX\_AUDIO\_TYPE\_G726\_40KBPS;

if(RecSet.AudioType == 0)

{

audiotype = PLAY\_A\_TYPE\_PCM\_8K;

}

else if(RecSet.AudioType == 1)

{

audiotype = TTX\_AUDIO\_TYPE\_G711A;

}

TTXNET\_SetMediaInfo(audiotype, 1, 8000, 16, TTX\_VIDEO\_CODEC\_H264);

}

**pTTXAVEnc-\>RealAVCmd.port = 5677;**

**strcpy((char\*)pTTXAVEnc-\>RealAVCmd.ip, "127.0.0.1");**

**pTTXAVEnc-\>RealAVCmd.fun = FrameDataCallback;**

**pTTXAVEnc-\>RealAVCmd.args = pTTXFrameCb;**

**printf("----------------StartAVEnc: Type=%d, streamtype=%d, Chn=%d\n", pTTXAVEnc-\>RealAVCmd.type, pTTXAVEnc-\>RealAVCmd.streamtype, pTTXAVEnc-\>RealAVCmd.chn);**

**pTTXAVEnc-\>AvSdkHandle = AvSdkStartRealStream(&(pTTXAVEnc-\>RealAVCmd));**

**\*ppAVUsr = pTTXAVEnc;**

**m\_MediaClientList-\>push\_back(pTTXAVEnc);**

return true;

}

### 2.4.2 Stop live preview

Call function：int AvSdkCloseRealStream(AVSDK pHandle, void\*\* args);

Include file：Avsdk\_interface.h

Sample:

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

//Stop audio and video coding. This interface will be called when the server stops real-time video and sound monitoring

//@Nmediatype: media type, TTX\_ MEDIA\_ TYPE\_ AV or TTX\_ MEDIA\_ TYPE\_ AUDIO

//@Nchannel: channel number starts from 0

//@Nstreamtype: bitstream type

//@Pvusr: the pvusr parameter entered in startavenc

//After calling this interface, the upper layer must stop calling the data callback function\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

bool CDevNetInterface::StopAVEnc(int nMediaType, int nChannel, int nStreamType, void\* pAVUsr)

{

if(pAVUsr == NULL)

{

return false;

}

TTXAVEnc\_t\* pTTXAVEnc = (TTXAVEnc\_t\*)pAVUsr;

std::list\<TTXAVEnc\_t\*\>::iterator it;

TTXAVEnc\_t \*item;

for (it = m\_MediaClientList-\>begin(); it != m\_MediaClientList-\>end();)

{

item = \*it;

if(item-\>AvSdkHandle == pTTXAVEnc-\>AvSdkHandle)

{

it = m\_MediaClientList-\>erase(it);

break;

}

it++;

}

**TTXFrameCb\_t\* pTTXFrameCb = NULL;**

**AvSdkCloseRealStream(pTTXAVEnc-\>AvSdkHandle, (void\*\*)&pTTXFrameCb);**

**if(pTTXFrameCb != NULL)**

**{**

**delete pTTXFrameCb;**

**}**

delete pTTXAVEnc;

return true;

}

## 2.5 History Video Preview

### 2.5.1 Search for file

Call function：int AvSdkSearchFile(SearchFileCmd\_t\* pSearchFileCmd, FileItem\_t\*\* pFileList);

Include file：Avsdk\_interface.h

Sample：

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Function function: new 1078 playback initialization.

Input / output: prequestplay: playback parameters

Simbcd: BDC code

Seq: serial number

Return value: 0 success, - 1 Failure

Instructions for use: None

Created by:

Creation date:

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

int CJTClient::JtRequestPlayBack(jt\_playback\_t \*pRequestPlay, u8 SimBCD[6], u16 Seq)

{

**SearchFileCmd\_t SearchFileCmd;**

**SearchFileCmd.type = 0;**

**SearchFileCmd.chntype = 0;**

**SearchFileCmd.chn = 0x01 \<\< (pRequestPlay-\>chn - 1);**

**SearchFileCmd.start\_time = bcd2time(pRequestPlay-\>start\_time\_bcd);**

**SearchFileCmd.end\_time = bcd2time(pRequestPlay-\>end\_time\_bcd);**

**SearchFileCmd.avtype = pRequestPlay-\>avtype;**

**SearchFileCmd.streamtype = pRequestPlay-\>streamtype;**

**SearchFileCmd.storetype = pRequestPlay-\>storetype;**

**FileItem\_t\* pFileList = NULL;**

**int filecount = AvSdkSearchFile(&SearchFileCmd, &pFileList);**

if(filecount \<= 0)

{

AvSdkFreeSearch(&pFileList);

return -1;

}

media\_playback\_t \*playback = (media\_playback\_t \*)malloc(sizeof(media\_playback\_t));

memset(playback, 0, sizeof(media\_playback\_t));

playback-\>Filecount = filecount;

playback-\>Exportitem = (Jt\_ExportItem \*)malloc(sizeof(Jt\_ExportItem) \* filecount);

if(playback-\>Exportitem == NULL)

{

AvSdkFreeSearch(&pFileList);

return -1;

}

unsigned int DiskType = 0;

ProductInfo\_t tmpProcutInfo;

memset(&tmpProcutInfo, 0, sizeof(ProductInfo\_t));

GetStatus(STATUS\_DEV\_TYPE, (void\*)&tmpProcutInfo);

PartAccess\_t PartAccess;

PartAccess.ProductId = tmpProcutInfo.DevType;

for(int index = 0; index \< filecount; index++)

{

sscanf(pFileList[index].file\_name, "/%04d\_%08x\_%d\_%d\_%d\_%d\_%d.avi", \

&playback-\>Exportitem[index].RecordItem.chn, &playback-\>Exportitem[index].RecordItem.record\_id, &DiskType, &playback-\>Exportitem[index].RecordItem.start\_time, \

&playback-\>Exportitem[index].RecordItem.end\_time, &playback-\>Exportitem[index].RecordItem.record\_type, &playback-\>Exportitem[index].RecordItem.record\_size); \

GetDiskNameByType((DISK\_PORT\_E)DiskType, PartAccess.path);

memcpy(&playback-\>Exportitem[index].PartAccess, &PartAccess, sizeof(PartAccess\_t));

}

AvSdkFreeSearch(&pFileList);

memcpy(&playback-\>RequestPlay, pRequestPlay, sizeof(jt\_playback\_t));

memset(&playback-\>RtpHead, 0, sizeof(rtp\_head\_t));

memcpy(playback-\>RtpHead.simnum, SimBCD, sizeof(playback-\>RtpHead.simnum));

playback-\>RtpHead.vpxcc = 0x81;

playback-\>RtpHead.chn = pRequestPlay-\>chn;

RecordSet\_t RecSet = {0};

memset(&RecSet, 0, sizeof(RecSet));

CConfig::Instance()-\>GetConfig(&RecSet);

if(pRequestPlay-\>streamtype == 0)

{

playback-\>RtpHead.enctype = RecSet.MainEncType;

}

else if(pRequestPlay-\>streamtype == 1)

{

playback-\>RtpHead.enctype = RecSet.SubEncType;

}

//playback-\>RtpHead.audiotype = RecSet.AudioType;

/\*if(RecSet.AudioType == AUDIO\_TYPE\_G711A)

{

pItem-\>pRtpHead-\>audiotype = AUDIO\_TYPE\_G711A;

}

else

{

pItem-\>pRtpHead-\>audiotype = AUDIO\_TYPE\_G726;

}\*/

playback-\>PlayControl.controltype = pRequestPlay-\>playtype;

switch(pRequestPlay-\>playtype)

{

case 0:

playback-\>PlayControl.controltype = JT\_AV\_PLAYBACK\_NORMAL;

playback-\>PlayControl.playspeed = 1;

break;

case 1:

playback-\>PlayControl.controltype = JT\_AV\_PLAYBACK\_FAST;

playback-\>PlayControl.playspeed = pRequestPlay-\>playspeed;

break;

case 3:

playback-\>PlayControl.controltype = JT\_AV\_PLAYBACK\_KFRAME;

playback-\>PlayControl.playspeed = 1;

break;

case 4:

playback-\>PlayControl.controltype = JT\_AV\_PLAYBACK\_ONE;

playback-\>PlayControl.playspeed = 1;

break;

default:

FreePlayBack(playback);

return -1;

}

playback-\>ClientState = SELECT\_SEND;

playback-\>playbacklock = new CLock;

playback-\>jtFrame.FrameList[0] = new std::list\<Jt\_FrameInfoData\_t \*\>;

playback-\>jtFrame.FrameList[1] = new std::list\<Jt\_FrameInfoData\_t \*\>;

playback-\>client = ClientInit();

playback-\>sock = tcp\_connect((char \*)playback-\>RequestPlay.addr, playback-\>RequestPlay.tcp\_port, NULL, 5000, 128\*1024, 128\*1024);

if (playback-\>sock \> 0)

{

ClientStart(playback-\>client, NULL, playback-\>sock, 60000, PlayBackCb, playback);

m\_PlayBackListLock.Lock();

m\_PlayBackList.push\_back(playback);

m\_PlayBackListLock.Unlock();

pthread\_t ThreadId;

pthread\_attr\_t attr;

struct sched\_param param;

pthread\_attr\_init(&attr);

pthread\_attr\_setscope(&attr, PTHREAD\_SCOPE\_SYSTEM);

pthread\_attr\_setdetachstate(&attr, PTHREAD\_CREATE\_DETACHED);

param.sched\_priority = sched\_get\_priority\_max(SCHED\_RR);

pthread\_attr\_setschedpolicy(&attr, SCHED\_RR);

pthread\_attr\_setschedparam(&attr, &param);

if (pthread\_create(&ThreadId, &attr, PsendData, (void \*)playback) == 0)

{

pthread\_attr\_destroy(&attr);

}

}

else

{

FreePlayBack(playback);

}

return 0;

}

### 2.5.1 Play or download files

1） **open file**

AVSDK AvSdkOpenFile(char\* pFileName, int ChnType);

AVSDK AvSdkOpenTimeRange(int Chn, time\_t tmBegin, time\_t tmEnd, int ChnType, int AvType, int StreamType, int StoreType);

2） **Read the media data in the file by frame**

int AvSdkReadFrame(AVSDK pHandle, FrameInfo\_t\* pFrameInfo, void \*\*pData);

int AvSdkReadFrames(AVSDK pHandle, int FrameCount, FrameInfo\_t\* pFrameInfo, unsigned char \*pBuf, int BufLen, int\* pRetCount);

int AvSdkReadNextIFrame(AVSDK pHandle, FrameInfo\_t\* pFrameInfo, void \*\*pData);

int AvSdkReadPrevIFrame(AVSDK pHandle, FrameInfo\_t\* pFrameInfo, void \*\*pData);

int AvSdkFreeFrame(AVSDK pHandle, void \*\*pData);

int AvSdkSeekFrame(AVSDK pHandle, time\_t SeekTime);

3）关闭文件

int AvSdkCloseFile(AVSDK pHandle);

## 2.6 Alarm distribution

### 2.6.1 Alarm sending

Call function：int DistributeEvent(EVENT\_E Event, int EventLen, void\* pEventData)

Include file： EventMngInterface.h

Reference structure：

typedef enum{

EVENT\_PLAYBACK\_CMD = 0,//PlayCmd\_t

EVENT\_VO\_CONTROL,//VoCtlCmd\_t

EVENT\_CONFIG\_UPDATE,//ConfigCmd\_t

EVENT\_WRITE\_DBG\_LOG,//WriteDbgLog\_t + logdata

EVENT\_WRITE\_USR\_LOG,//WriteUserLog\_t + extdata

EVENT\_SET\_CLOCK,//SetClockCmd\_t

EVENT\_OP\_SW\_DOG,//SwDogCmd\_t

EVENT\_OP\_HW\_DOG,//HwDogCmd\_t

EVENT\_OP\_TASK,//TaskCmd\_t

EVENT\_ALARM,//AlarmCmd\_t

EVENT\_POWEROFF,//PowerOffCmd\_t

EVENT\_DISK\_EVENT,//DiskOpCmd\_t

EVENT\_REMOTE,//RemoteCmd\_t

EVENT\_VIDEO\_ALARM, //VideoAlarmCmd\_t

EVENT\_SERIAL\_CONTROL, //SerialCmd\_t

EVENT\_EXT\_IO\_CONTROL, //ExtIoCmd\_t

EVENT\_MAINTAIN, //MaintainCmd\_t

EVENT\_TXT\_TTS, //TxtTtsCmd\_t

EVENT\_IPC\_EVENT, //IPCCmd\_t

EVENT\_PULSE\_SPEED,// EventPulseSpeed\_t,脉冲速度

EVENT\_JT808\_EVENT,//Jt808Event\_t

EVENT\_TOUCH\_PAD,//EventTouchMonitorKeyValue\_t

EVENT\_USER\_DEFINE, //UsrDefineCmd\_t, 该事件不再增加子类型,避免无效耦合度进一步提升2019.01.16

EVENT\_USR\_DEF2, //UsrDefineCmd\_t, JFSK使用2019.01.16

EVENT\_USR\_DEF3, //UsrDefineCmd\_t,

EVENT\_USR\_DEF4, // UsrDefineCmd\_t, qdd use for ai board biaoding,

EVENT\_CAN\_CONTROL,//CanCmd\_t,

EVENT\_PRIMARY\_CENTER,//MasterCenter\_t 主中心一发送的事件

EVENT\_GB19056\_EVENT,//GB19056Event\_t

EVENT\_MISC,//MiscCmd\_t

EVENT\_JT905\_EVENT,

EVENT\_MAX = 63, //

}EVENT\_E;

**Sample**** ：**

MiscCmd\_t MiscCmd;

MiscCmd.Type = MISC\_EVENT\_ACCOPEN;

MiscCmd.Data[0] = accStatus;

DistributeEvent(EVENT\_MISC, sizeof(MiscCmd\_t), (void\*)&MiscCmd);

### 2.6.2 Alarm reception

Call function：StartUnixDomainListen((char\*)tmpDomain, sdk\_domain\_comm\_proc, NULL);

Include file：EventMngInterface.h

Sample： sample code

void\* SdkStartUnixDomain()

{

long long EventBitmap = 0;

char tmpDomain[128] = {0};

setbit(&EventBitmap, EVENT\_ALARM);

setbit(&EventBitmap, EVENT\_USR\_DEF4);

setbit(&EventBitmap, EVENT\_CAN\_CONTROL);

sprintf(tmpDomain, "/tmp/\_%u\_sdk\_event.listen", getpid());

UnregisterEventReciever((char\*)tmpDomain);

RegisterEventReciever((char\*)tmpDomain, EventBitmap);

return StartUnixDomainListen((char\*)tmpDomain, sdk\_domain\_comm\_proc, NULL);

}

int sdk\_domain\_comm\_proc(int fd, struct sockaddr\_un \*addr, socklen\_t \*addrlen, char \*recvbuf, int recvlen, void\* args)

{

//printf("av\_domain\_comm\_proc Recv(%d): %s\n", recvlen, recvbuf);

char Tmpbuf[8192] = {0};

EVENT\_E Event;

int EventLen = 0;

int ret = ParseRecvEvent((unsigned char\*)recvbuf, recvlen, &Event, &EventLen, (void \*)Tmpbuf);

if(ret \< 0)

{

return -1;

}

switch(Event)

{

//报警事件

case EVENT\_ALARM:

{

AlarmCmd\_t\* pAlarmCmd = (AlarmCmd\_t\*)Tmpbuf;

if(EVENT\_ALARM\_IO == pAlarmCmd-\>Type)

{

LedAlarmSt\_t tmpSet = {};

int needsend = 1;

switch(pAlarmCmd-\>subType)

{

case DVR\_IO\_TIGHT\_TURN:

{

if(pAlarmCmd-\>enable)

{

tmpSet.type = KEYCODE\_RIGHT\_ON;

}

else

{

tmpSet.type = KEYCODE\_RIGHT\_OFF;

}

break;

}

case DVR\_IO\_LEFT\_TURN:

{

if(pAlarmCmd-\>enable)

{

tmpSet.type = KEYCODE\_LEFT\_ON;

}

else

{

tmpSet.type = KEYCODE\_LEFT\_OFF;

}

break;

}

case DVR\_IO\_BRAKE:

{

if(pAlarmCmd-\>enable)

{

tmpSet.type = KEYCODE\_BRAKE\_ON;

}

else

{

tmpSet.type = KEYCODE\_BRAKE\_OFF;

}

break;

}

default:

{

needsend = 0;

break;

}

}

if(needsend)

{

g\_eventFb(SDK\_EVENT\_ALARM\_LED, (void\*)&tmpSet, sizeof(tmpSet), g\_eventFbArgs);

}

}

break;

}

case EVENT\_USR\_DEF4:

{

UsrDefineCmd\_t \*pUsrDefineCmd = (UsrDefineCmd\_t\*)Tmpbuf;

if(pUsrDefineCmd-\>Type == EVENT\_RUN\_MODE)

{// 运行模式事件

RunModeSt\_t\* pAlarmCmd = (RunModeSt\_t\*)pUsrDefineCmd-\>Data;

g\_eventFb(SDK\_EVENT\_RUN\_MODE, (void\*)pAlarmCmd, sizeof(RunModeSt\_t), g\_eventFbArgs);

}

else if(pUsrDefineCmd-\>Type == EVENT4\_AVMANAGE\_RUN)

{//

//AvmRunSt\_t \*ptmpCmd = (AvmRunSt\_t \*)pUsrDefineCmd-\>Data;

g\_eventFb(SDK\_EVENT\_AVMANAGE\_RUN, (void\*)pUsrDefineCmd-\>Data, sizeof(AvmRunSt\_t), g\_eventFbArgs);

}

else if(pUsrDefineCmd-\>Type == EVENT4\_MMT\_JT\_UPGRADE)

{

//JtUpgradeSt\_t \*ptmpCmd = (JtUpgradeSt\_t \*)pUsrDefineCmd-\>Data;

g\_eventFb(SDK\_EVENT\_DOWNLOAD\_UPGRADE\_FILE, (void\*)pUsrDefineCmd-\>Data, sizeof(JtUpgradeSt\_t), g\_eventFbArgs);

}

else if(pUsrDefineCmd-\>Type == EVENT4\_SERVER\_TRANS)

{

ServerTransData\_t \*pTmpData = (ServerTransData\_t \*)pUsrDefineCmd-\>Data;

pTmpData-\>pData = &pUsrDefineCmd-\>Data[sizeof(ServerTransData\_t)];// 做一次转换地址

/\*分配内存回收结果\*/

pTmpData-\>resultMaxLen = 2048;

pTmpData-\>pResult = (char \*)malloc(pTmpData-\>resultMaxLen);

if(pTmpData-\>pResult)

{

memset(pTmpData-\>pResult, 0, pTmpData-\>resultMaxLen);

}

else

{

pTmpData-\>resultMaxLen = 0;

printf("-----malloc(%d) error----\n", pTmpData-\>resultMaxLen);

}

int ret = g\_eventFb(SDK\_EVENT\_TRANS\_DATA, (void\*)pTmpData, sizeof(ServerTransData\_t), g\_eventFbArgs);

if(ret \> 0)

{

if(pTmpData-\>pResult)

{

//set file

char \*tmpCbFile="/tmp/.transResponeData";

buffertofile(pTmpData-\>pResult, ret, tmpCbFile);

//释放data

free(pTmpData-\>pResult);

pTmpData-\>pResult = NULL;

pTmpData-\>resultMaxLen = 0;

}

}

}

else if(pUsrDefineCmd-\>Type == EVENT4\_MMT\_AI\_INFO)

{

g\_eventFb(SDK\_EVENT\_AI\_STATUS\_INFO, NULL, 0, g\_eventFbArgs);

}

break;

}

case EVENT\_CAN\_CONTROL:

{

CanCmd\_t \*pCanCmd = (CanCmd\_t\*)Tmpbuf;

if(pCanCmd-\>Cmd == 1)

{//recv

//printf("Hnadle Can[%d] Transpt len=%d data\n",pCanCmd-\>ID,pCanCmd-\>Datalen);

g\_eventFb(SDK\_EVENT\_RECV\_CAN\_DATA, pCanCmd, sizeof(CanCmd\_t), g\_eventFbArgs);

}

break;

}

default:

{

break;

}

}

return 0;

}

## 2.7 Serial peripheral

1）Method 1: the customer operates the serial port by himself；

2）Method 2: serial port data delivery through inter process communication, and alarm distribution process is adopted；

## 2.8 AI alarm receiving and small video acquisition

### 2.8.1 AI alarm registration

int CJTMdvrMng::StartUnixDomain()

{

jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;

long long EventBitmap = 0;

char tmpUnixDomainName[128] = {0};

snprintf(tmpUnixDomainName, sizeof(tmpUnixDomainName) - 1, "/tmp/%s.listen", ProcessName);

setbit(&EventBitmap, EVENT\_MISC);

setbit(&EventBitmap, EVENT\_ALARM);

setbit(&EventBitmap, EVENT\_POWEROFF);

setbit(&EventBitmap, EVENT\_CONFIG\_UPDATE);

setbit(&EventBitmap, EVENT\_DISK\_EVENT);

setbit(&EventBitmap, EVENT\_USER\_DEFINE);

setbit(&EventBitmap, EVENT\_JT808\_EVENT);

setbit(&EventBitmap, EVENT\_SERIAL\_CONTROL);

setbit(&EventBitmap, EVENT\_CAN\_CONTROL);

setbit(&EventBitmap, EVENT\_USR\_DEF2);

setbit(&EventBitmap, EVENT\_PULSE\_SPEED);

setbit(&EventBitmap, EVENT\_USR\_DEF4);

setbit(&EventBitmap, EVENT\_GB19056\_EVENT);

UnregisterEventReciever(tmpUnixDomainName);

RegisterEventReciever(tmpUnixDomainName, EventBitmap);

pHandle-\>pUnixDomain = StartUnixDomainListen(tmpUnixDomainName, CJTMdvrMng::JtGpsInterfaceProc, this);

return 0;

}

### 2.8.2 Alarm event name and distribution event definition

int AimngServer::NldzAlarmUploadFile(NldzAlarm\_t\* item)

{

UsrDefineCmd\_t m\_Cmd = {0};

m\_Cmd.Type = EVENT\_YINCAA\_ALARM\_REPORT;

m\_Cmd.DataLen = sizeof(NldzAlarm\_t);

memcpy(m\_Cmd.Data, (void \*)item, sizeof(NldzAlarm\_t));

DistributeEvent(EVENT\_USER\_DEFINE, sizeof(UsrDefineCmd\_t), (void \*)&m\_Cmd);

WriteDbgLog(2, "m\_Cmd.DataLen = %d AimngAlarmTo808 finish\n", m\_Cmd.DataLen);

return 0;

}

**Alarm event distribution function**

int AimngServer::AimngAlarmTo808(AimngUpload\_t\* item, FileInfor\_t\* pUploadlist)

{

static u16 seq[4] = {0};

int i = 0;

JTBaseSet\_t JtBaseSet;

CConfig \*pConfig = CConfig::Instance();

pConfig-\>GetConfig(&JtBaseSet);

{

NldzAlarm\_t NldzAlarm = {0};

NldzAlarm.type = item-\>aiDownload.type;

if(item-\>aiDownload.type == PER\_ADAS)

{

item-\>aiDownload.Data.AdasDriveAuxiliary.AuxiliaryBasis.flagSta = 0x00;

memcpy(&(NldzAlarm.Data.DriveAuxiliary.AuxiliaryBasis), &(item-\>aiDownload.Data.AdasDriveAuxiliary.AuxiliaryBasis), sizeof(AuxiliaryBasis\_t));

memcpy(NldzAlarm.Data.DriveAuxiliary.alarmIDnum.terminalID, JtBaseSet.TerminalId, 7);

memcpy(NldzAlarm.Data.DriveAuxiliary.alarmIDnum.time, item-\>aiDownload.Data.AdasDriveAuxiliary.AuxiliaryBasis.time, 6);

NldzAlarm.Data.DriveAuxiliary.alarmIDnum.seq = 0;//seq[0]; //qdd 2021.03.03 set 0

NldzAlarm.Data.DriveAuxiliary.alarmIDnum.AttachCount = item-\>aiDownload.Data.AdasDriveAuxiliary.listTotal;

for(i=0; i\<item-\>aiDownload.Data.AdasDriveAuxiliary.listTotal; i++)

{

NldzAlarm.Data.DriveAuxiliary.list[i] = pUploadlist[i];

/\*WriteDbgLog(2, "ADAS %d FileSize = %d, FileType = %d, FileNamelen = %d, FileName = %s, storagePath = %s\n", i, pUploadlist[i].FileSize, pUploadlist[i].FileType, \

pUploadlist[i].FileNamelen, pUploadlist[i].FileName, pUploadlist[i].storagePath);\*/

}

seq[0]++;

}

else if(item-\>aiDownload.type == PER\_DSM)

{

item-\>aiDownload.Data.DsmDriveState.StateBasis.flagSta = 0x00;

memcpy(&(NldzAlarm.Data.DriveState.StateBasis), &(item-\>aiDownload.Data.DsmDriveState.StateBasis), sizeof(StateBasis\_t));

memcpy(NldzAlarm.Data.DriveState.alarmIDnum.terminalID, JtBaseSet.TerminalId, 7);

memcpy(NldzAlarm.Data.DriveState.alarmIDnum.time, item-\>aiDownload.Data.DsmDriveState.StateBasis.time, 6);

NldzAlarm.Data.DriveState.alarmIDnum.seq = 0;//seq[1]; //qdd 2021.03.03 set 0

NldzAlarm.Data.DriveState.alarmIDnum.AttachCount = item-\>aiDownload.Data.DsmDriveState.listTotal;

for(i=0; i\<item-\>aiDownload.Data.DsmDriveState.listTotal; i++)

{

NldzAlarm.Data.DriveState.list[i] = pUploadlist[i];

/\*WriteDbgLog(2, "DSM %d FileSize = %d, FileType = %d, FileNamelen = %d, FileName = %s, storagePath = %s\n", i, pUploadlist[i].FileSize, pUploadlist[i].FileType, \

pUploadlist[i].FileNamelen, pUploadlist[i].FileName, pUploadlist[i].storagePath);\*/

}

seq[1]++;

}

#if defined(HTV400JT\_ZHATU\_2) //HTP9Z项目使用 2021.04.13 ssx

else if(item-\>aiDownload.type == PER\_BSD || item-\>aiDownload.type == PER\_HTPQZ)

#else

else if(item-\>aiDownload.type == PER\_BSD)

#endif

{

item-\>aiDownload.Data.BsdState.alarmState.flagSta = 0x00;

memcpy(&(NldzAlarm.Data.BsdJtState.StateBasis), &(item-\>aiDownload.Data.BsdState.alarmState), sizeof(NldzAlarm.Data.BsdJtState.StateBasis));

memcpy(NldzAlarm.Data.BsdJtState.alarmIDnum.terminalID, JtBaseSet.TerminalId, 7);

memcpy(NldzAlarm.Data.BsdJtState.alarmIDnum.time, item-\>aiDownload.Data.BsdState.alarmState.time, 6);

NldzAlarm.Data.BsdJtState.alarmIDnum.seq = 0;//seq[2]; //qdd 2021.03.03 set 0

NldzAlarm.Data.BsdJtState.alarmIDnum.AttachCount = item-\>aiDownload.Data.BsdState.listTotal;

for(i=0; i\<item-\>aiDownload.Data.BsdState.listTotal; i++)

{

NldzAlarm.Data.BsdJtState.list[i] = pUploadlist[i];

/\*WriteDbgLog(2, "BSD %d FileSize = %d, FileType = %d, FileNamelen = %d, FileName = %s, storagePath = %s\n", i, pUploadlist[i].FileSize, pUploadlist[i].FileType, \

pUploadlist[i].FileNamelen, pUploadlist[i].FileName, pUploadlist[i].storagePath);\*/

}

seq[1]++;

}

else if(item-\>aiDownload.type == PER\_VMS)

{

item-\>aiDownload.Data.VmsState.alarmState.flagSta = 0x00;

memcpy(&(NldzAlarm.Data.VmsState.alarmState), &(item-\>aiDownload.Data.VmsState.alarmState), sizeof(StateBasis\_t));

memcpy(NldzAlarm.Data.VmsState.alarmIDnum.terminalID, JtBaseSet.TerminalId, 7);

memcpy(NldzAlarm.Data.VmsState.alarmIDnum.time, item-\>aiDownload.Data.VmsState.alarmState.time, 6);

NldzAlarm.Data.VmsState.alarmIDnum.seq = 0;//seq[1]; //qdd 2021.03.03 set 0

NldzAlarm.Data.VmsState.alarmIDnum.AttachCount = item-\>aiDownload.Data.VmsState.alarmIDnum.AttachCount;

for(i=0; i\<NldzAlarm.Data.VmsState.alarmIDnum.AttachCount; i++)

{

NldzAlarm.Data.VmsState.list[i] = pUploadlist[i];

/\*WriteDbgLog(2, "VMS %d FileSize = %d, FileType = %d, FileNamelen = %d, FileName = %s, storagePath = %s\n", i, pUploadlist[i].FileSize, pUploadlist[i].FileType, \

pUploadlist[i].FileNamelen, pUploadlist[i].FileName, pUploadlist[i].storagePath);\*/

}

seq[1]++;

}

else

{

return -1;

}

// 上传文件

NldzAlarmUploadFile(&NldzAlarm);

}

return 0;

}

### 2.8.3 Alarm receiving and processing

The process received an excerpt from the event processing section：

case EVENT\_USER\_DEFINE:

{

UsrDefineCmd\_t \*pUsrDefineCmd = (UsrDefineCmd\_t\*)Tmpbuf;

if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_ALARM\_REPORT)

{

pthis-\>EventHandler(EVENT\_NLDZ\_ALARM\_REPORT, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_USR\_808\_CMD\_PACKET)

{

pthis-\>EventHandler(EVENT\_808\_CMD\_PACKET, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_USR\_808\_POS\_EXT)

{

SerialPassCmd\_t\* pSerialPassCmd = (SerialPassCmd\_t\*)pUsrDefineCmd-\>Data;

pthis-\>EventHandler(EVENT\_808\_POS\_EXT, pSerialPassCmd-\>MaxCount, pSerialPassCmd-\>AddId, pSerialPassCmd-\>SerialData, pSerialPassCmd-\>SerialDataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_SEND\_FTP\_CONTROL)

{

JtFtpTransStateCtrl \*ftpFile = (JtFtpTransStateCtrl \*)pUsrDefineCmd-\>Data;

pthis-\>EventHandler(EVENT\_FTP\_RESULT\_PACKET, ftpFile-\>seq, ftpFile-\>result, 0, 0);

}

else if(pUsrDefineCmd-\>Type == EVENT\_ID\_CONNECT2)

{

pthis-\>EventHandler(EVENT\_CONNECT\_CTL\_MSG\_0, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_ID\_CONNECT)

{

pthis-\>EventHandler(EVENT\_CONNECT\_CTL\_MSG\_1, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_TTX)

{

pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_TTX, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_HZSJ)

{

pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_HZSJ, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_HD\_LCD\_LED)

{

Jt808Transpt\_t\* pTmp = (Jt808Transpt\_t\*)pUsrDefineCmd-\>Data;

pthis-\>EventHandler(EVENT\_TRANSIT, pTmp-\>type, 0, (char\*)pTmp-\>Data, pTmp-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_F1)

{

pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_F1, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_QUERY\_REPORT)

{

pthis-\>EventHandler(EVENT\_PERIPHERALS\_QUERY\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_UPGRADE\_RES)

{

pthis-\>EventHandler(EVENT\_PERIPHERALS\_UPGRADE\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_QUERY\_PARAM\_RES)

{

pthis-\>EventHandler(EVENT\_YC\_QUERY\_PARAM\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

else if(pUsrDefineCmd-\>Type == EVENT\_SET\_RUN\_INFO)

{

pthis-\>EventHandler(EVENT\_SET\_RUN\_INFO\_808, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);

}

}

break;

**EventHandler function:**

case EVENT\_NLDZ\_ALARM\_REPORT:

{

NldzAlarm\_t\* pNldzAlarm = (NldzAlarm\_t\*)alarmData;

WriteDbgLog(0, "EVENT\_NLDZ\_ALARM\_REPORT type=0x%x\n", pNldzAlarm-\>type);

if(pNldzAlarm-\>type == PER\_ADAS)

{

//WriteDbgLog(2, "SetExtData ADAS\n");

memset(tmpbuf, 0, sizeof(tmpbuf));

tmplen = m\_client-\>BuildDriveAuxiliaryPacket(&(pNldzAlarm-\>Data.DriveAuxiliary), tmpbuf);

SetExtDataEx(1, PER\_ADAS, tmplen, tmpbuf);

m\_ReportFlag++;

if(pNldzAlarm-\>Data.DriveAuxiliary.alarmIDnum.AttachCount \> 0)

{

m\_client-\>NldzUploadListAdd(pNldzAlarm);

}

}

else if(pNldzAlarm-\>type == PER\_DSM)

{

//WriteDbgLog(2, "SetExtData DSM\n");

memset(tmpbuf, 0, sizeof(tmpbuf));

tmplen = m\_client-\>BuildDriveStatePacket(&(pNldzAlarm-\>Data.DriveState), tmpbuf);

SetExtDataEx(1, PER\_DSM, tmplen, tmpbuf);

m\_ReportFlag++;

if(pNldzAlarm-\>Data.DriveState.alarmIDnum.AttachCount \> 0)

{

m\_client-\>NldzUploadListAdd(pNldzAlarm);

}

}

/\*qdd 2020.10.28 add for bsd\*/

else if(pNldzAlarm-\>type == PER\_BSD)

{

//WriteDbgLog(2, "SetExtData BSD\n");

memset(tmpbuf, 0, sizeof(tmpbuf));

tmplen = m\_client-\>BuildDriveBsdStatePacket(&(pNldzAlarm-\>Data.BsdJtState), tmpbuf);

SetExtDataEx(1, PER\_BSD, tmplen, tmpbuf);

m\_ReportFlag++;

if(pNldzAlarm-\>Data.BsdJtState.alarmIDnum.AttachCount \> 0)

{

m\_client-\>NldzUploadListAdd(pNldzAlarm);

}

}

else if(pNldzAlarm-\>type == PER\_VMS)

{

//WriteDbgLog(2, "SetExtData VMS\n");

memset(tmpbuf, 0, sizeof(tmpbuf));

tmplen = m\_client-\>BuildDriveVmsPacket(&(pNldzAlarm-\>Data.VmsState), tmpbuf);

SetExtDataEx(1, PER\_VMS, tmplen, tmpbuf);

m\_ReportFlag++;

if(pNldzAlarm-\>Data.VmsState.alarmIDnum.AttachCount \> 0)

{

m\_client-\>NldzUploadListAdd(pNldzAlarm);

}

}

}

## 2.9 Video capture

Include file：avsdk\_interface.h

structure：SnapCmd\_t;

typedef struct{

unsigned int chn;////channel num,1~n

unsigned int res;// resolution 0: same as main stream, 1:1080,2:720,3:VGA,4:D1

PictureDataCb fun;

void\* args;

/\*qdd 2020.10.22 add new for multi\*/

unsigned int count;// snap num 1~10

unsigned int interval;//snap inteval, unit: microsecond

}SnapCmd\_t;

Call function：int AvSdkSnapshot(SnapCmd\_t\* pSnapCmd);

## 2.10 Process feed dog

Logic Description:

Each process needs to feed a dog to the process so that the main process can know the operation of the process. The thread of this process class also needs to feed a dog to the main thread of this process, so that the main thread can judge whether it needs to feed a dog to the process process；

Include file：comm\_dog.h

Sample code：Take adding a process, myprocess, as an example

1. The main thread feeds the dog to the process：

int CheckSwDog(time\_t CurTime)

{

static time\_t LastCheckTime = CurTime;

if(abs(CurTime - LastCheckTime) \> 10)

{

LastCheckTime = CurTime;

CCommDog \*pCommDog = CCommDog::Instance();

if(pCommDog-\>CheckDog() \> 0) /\* Judge whether all threads in this process feed dogs normally \*/

{

printf("myprocess Dog TimeOut\n");

//write user log

char logbuf[256] = {0};

snprintf(logbuf, 255, "myprocess Dog TimeOut");

WriteUserLog(LOG\_TASK, LOG\_TASK\_THREAD\_DOG\_ERROR, strlen(logbuf), logbuf);

}

else

{

// Feed the dog to the process

SwDogCmd\_t SwDogCmd;

SwDogCmd.Enable = 2;

strncpy(SwDogCmd.Task, "myprocess", sizeof(SwDogCmd.Task));

DistributeEvent(EVENT\_OP\_SW\_DOG, sizeof(SwDogCmd\_t), (void\*)&SwDogCmd);

}

}

return 0;

}

**The thread in this process feeds the dog. The following is a thread body reference code similar to that**

char DogName[32] = {0};

snprintf(DogName, sizeof(DogName) - 1, "mythread");

CCommDog \*pCommDog = CCommDog::Instance();

int DogId = pCommDog-\>RegisterDog(DogName, 10); /\*10s timeout \*/

while（1）

{

//local business code

//thread feed dog，Every two seconds

time\_t curtime = time(NULL);

if(abs(curtime - timebak) \> 2)

{

timebak = curtime;

pCommDog-\>FeedDog(DogId);

}

}

// Thread exit, logout dog

pCommDog-\>DeregisterDog(DogId);

## 2.11 The customer realizes the 4G dial-up or WiFi connection process

### 2.11.1 Get dialing parameters and WiFi parameters

**1**** ） ****Get dialing parameters**

DialupSet\_t tmpDialSet;

memset(&tmpDialSet, 0, sizeof(tmpDialSet));

CConfig::Instance()-\>GetConfig(&tmpDialSet);

**2**** ） ****Get WiFi parameters**

**WifiSet\_t tmpWifiSet;**

**memset(&tmpWifiSet, 0, sizeof(WifiSet\_t));**

**CConfig::Instance()-\>GetConfig(&tmpWifiSet);**

1. **Control WiFi or 4G module power**

**//wifi power**

**int CNetDevInterface::ControlWifiPower(int OnOrOff)**

**{**

**//peter\_chen add 2017.03.31**

**if(OnOrOff)**

**{**

**return IoControl(IO\_POWER\_WIFI, true);**

**}**

**return IoControl(IO\_POWER\_WIFI, false);**

**}**

**//4G power**

**int CNetDevInterface::PowerSwitch(int isOn)**

**{**

**if(isOn)**

**{**

**return IoControl(IO\_POWER\_3G, true);**

**}**

**return IoControl(IO\_POWER\_3G, false);**

**}**

1.

### 2.11.2 Set and get dial-up status and WiFi connection status

1. The status that needs to be set when dialing fails or dials successfully, so that other processes can get the status or display it

Note: get first and then set, and call two functions **GetStatus**** 和 ****SetStatus.**

int CNetDevInterface::StateCallBack(ggg\_Status p\_info, char\* pATPort, char\* pModemPort)

{

WirelessStatus\_t tmpWirelessStatus;

memset(&tmpWirelessStatus, 0, sizeof(tmpWirelessStatus));

**GetStatus(STATUS\_WIRELESS, (void \*)&tmpWirelessStatus);**

tmpWirelessStatus.Exist = p\_info.m\_module\_exist;

tmpWirelessStatus.enable = p\_info.m\_module\_enable;

tmpWirelessStatus.Sim = p\_info.m\_sim\_exist;

tmpWirelessStatus.Csq = p\_info.m\_sim\_Signal\_Value;

tmpWirelessStatus.Dial = (p\_info.m\_dialup\_Status == DIALUP\_DIAL\_OK)?1:0;

tmpWirelessStatus.WirelessMode = p\_info.m\_net\_type;

tmpWirelessStatus.ModuleType = p\_info.m\_module\_type;

tmpWirelessStatus.SimType = p\_info.SimType;

// printf("StateCallBack:%d\n",tmpWirelessStatus.SimType);

if(strlen(p\_info.m\_IMEI) \> 0 && strcmp(p\_info.m\_IMEI,tmpWirelessStatus.Imei))

{

WriteDbgLog(2,"change imei:(%s)-\>(%s)\n", p\_info.m\_IMEI,tmpWirelessStatus.Imei);

strncpy(tmpWirelessStatus.Imei, p\_info.m\_IMEI, sizeof(tmpWirelessStatus.Imei));

}

else

{

//WriteDbgLog(2,"imei:(%s)-\>(%s)\n", p\_info.m\_IMEI,tmpWirelessStatus.Imei);

}

strncpy(tmpWirelessStatus.Imsi, p\_info.m\_IMSI, sizeof(tmpWirelessStatus.Imsi));

strncpy(tmpWirelessStatus.Iccid, p\_info.m\_sim\_ICCID, sizeof(tmpWirelessStatus.Iccid));

if(tmpWirelessStatus.Dial == 1) /\*peter\_chen 2018.11.28 \*/

{

CWireless::Instance()-\>get\_ppp\_ip(tmpWirelessStatus.DialIp);

}

else

{

strcpy(tmpWirelessStatus.DialIp, "0.0.0.0");

}

strncpy(tmpWirelessStatus.ATPort, pATPort, sizeof(tmpWirelessStatus.ATPort));

strncpy(tmpWirelessStatus.ModemPort, pModemPort, sizeof(tmpWirelessStatus.ModemPort));

WriteDbgLog(0,"StateCallBack Exist:%d ,enable:%d, Dial:%d\n",tmpWirelessStatus.Exist, tmpWirelessStatus.enable,tmpWirelessStatus.Dial);

**SetStatus(STATUS\_WIRELESS, (void \*)&tmpWirelessStatus);**

return 0;

}

1. **The status that needs to be set for WiFi configuration success or failure so that other processes can obtain and display it.**

WifiStatus\_t tmpWifiStatus;

memset(&tmpWifiStatus, 0, sizeof(WifiStatus\_t));

tmpWifiStatus.ModuleStatus = WifiStatus.ModuleStatus;

tmpWifiStatus.ConnStatus = WifiStatus.ConnStatus;

strncpy(tmpWifiStatus.SignalStr, WifiStatus.SignalStr, sizeof(tmpWifiStatus.SignalStr));

strncpy(tmpWifiStatus.LinkSSID, WifiStatus.LinkSSID, sizeof(tmpWifiStatus.LinkSSID));

strncpy(tmpWifiStatus.IpAddr, WifiStatus.IpAddr, sizeof(tmpWifiStatus.IpAddr));

if(tmpWifiStatus.ModuleStatus == 0)

{

if(tmpWifiStatus.ConnStatus != 0)

{

tmpWifiStatus.ConnStatus = 0;

strcpy(tmpWifiStatus.SignalStr, "0");

WifiSignal = 0;

ConnStatus = 0;

}

}

**SetStatus(STATUS\_WIFI, (void \*)&tmpWifiStatus);**

### 2.11.3 implement scheme

**1) Customer does't use netmng process**

Customers need to realize WiFi connection and 4G dial-up. And change the process.xml to disable the netmng process. If the connection fails or succeeds, the above two statuses need to be set so that we can get for other processes；

**2) Customer doesn't use the function of 4G dial-up**

In our dialing parameter settings, turn off 4G dialing, so that our netmng will not dial, and the customer will start a new dialing process by himself。

# appendix 1 Start network thread reference code

**#include "JTClient.h"**

**#include "JTMdvr.h"**

**#include "comm\_dog.h"**

**#include "EventMngInterface.h"**

**#include "NetMngInterface.h"**

**#include "LogMngInterface.h"**

**#include "signal.h"**

**#include "comm\_time.h"**

**CJTClient \*JTgpsClient = NULL;**

**//**** 子程序停止**

**static void \_\_SignalHandler(int signum)**

**{**

**//fprintf(stderr,"[%d]Receive Signal %s\n",getpid(),strsignal(signum));**

**WriteDbgLog(2, "JTGPS [%d]Receive Signal %s\n",getpid(),strsignal(signum));**

**switch( signum )**

**{**

**case SIGCHLD:**

**break;**

**case SIGSEGV:**

**WriteDbgLog(2, "pthread\_self = %ld\n", pthread\_self());**

**break;**

**case SIGINT:**

**case SIGTERM:**

**exit(0);**

**break;**

**case SIGKILL:**

**WriteDbgLog(2, "SIGKILL!\n");**

**break;**

**case SIGABRT:**

**case SIGSTOP:**

**default:**

**break;**

**}**

**}**

**int main(int argc, char \*\*argv)**

**{**

**int RedirectType = InitStdoutRedirect();**

**char ProcessName[64] = {0};**

**NET\_PROTOCOL\_TYPE\_E NetMode;**

**sleep(11);**

**#if !defined(JT\_PROTOCOL)**

**RegisterResUser(0); // not fix**

**#endif**

**int center\_number = 0;//hexj,**** 中心编号， ****1**** 起始**

**JTBaseSet\_t JtBaseSet = { 0 };**

**CConfig::Instance()-\>GetConfig(&JtBaseSet);**

**int server\_Ex\_Total = ARRAY\_SIZE(JtBaseSet.protocolex);//****扩展的中心服务器个数**

**memset(ProcessName, 0, sizeof(ProcessName));**

**NetMode = InitWorkMode(ProcessName,&center\_number,server\_Ex\_Total);**

**int ServerNum = GetNetProtocolInfo(NetMode,center\_number);**

**if(ServerNum \< 0)**

**{**

**WriteDbgLog(2, "%s Not Select\n", ProcessName);**

**TaskCmd\_t TaskCmd = {0};**

**memset(TaskCmd.Task,0,sizeof(TaskCmd.Task));**

**strncpy(TaskCmd.Task, ProcessName, sizeof(TaskCmd.Task));**

**TaskCmd.Op = 0;**

**DistributeEvent(EVENT\_OP\_TASK, sizeof(TaskCmd\_t), (void\*)&TaskCmd);**

**sleep(1);**

**exit(0);**

**}**

**CJTMdvrMng \*pMdvrMng = CJTMdvrMng::Instance();**

**pMdvrMng-\>Start(ProcessName, NetMode,center\_number);**

**pMdvrMng-\>StartUnixDomain();**

**JTgpsClient = new CJTClient;**

**JTgpsClient-\>Start(NetMode,center\_number);**

**CCommDog \*pCommDog = CCommDog::Instance();**

**while ( 1 )**

**{**

**if(pCommDog-\>CheckDog() \> 0)**

**{**

**WriteDbgLog(2, "%s Dog TimeOut\n", ProcessName);**

**}**

**else**

**{**

**SwDogCmd\_t SwDogCmd;**

**SwDogCmd.Enable = 1;**

**strncpy(SwDogCmd.Task, ProcessName, sizeof(SwDogCmd.Task));**

**DistributeEvent(EVENT\_OP\_SW\_DOG, sizeof(SwDogCmd\_t), (void\*)&SwDogCmd);**

**}**

**RedirectType = RefreshStdoutRedirect(RedirectType);**

**sleep(5);**

**}**

**pMdvrMng-\>StopUnixDomain();**

**pMdvrMng-\>Stop();**

**return 0;**

**}**

**#ifndef \_\_JT\_MDVR\_MANAGE\_\_**

**#define \_\_JT\_MDVR\_MANAGE\_\_**

**#include "comm\_thread.h"**

**#include "JTInterfaceApp.h"**

**#include "EventMngInterface.h"**

**#include "PrivStatusStruct.h"**

**class CJTMdvrMng**

**{**

**public:**

**CJTMdvrMng();**

**~CJTMdvrMng();**

**static CJTMdvrMng\* Instance();**

**int Start(char \*process\_name, NET\_PROTOCOL\_TYPE\_E netMode, int center\_number);**

**int Stop();**

**static int JtGpsInterfaceProc(int fd, struct sockaddr\_un \*addr, socklen\_t \*addrlen, char \*recvbuf, int recvlen, void\* args);**

**int StartUnixDomain();**

**int StopUnixDomain();**

**void JTMdvrMngProc(CommonThread \*lt);**

**int InitEventCB(EventCallBack cbFunc, void \*cbArgs);**

**int QuitEventCB(EventCallBack cbFunc, void \*cbArgs);**

**int SetNetMode(NET\_PROTOCOL\_TYPE\_E netMode, int center\_number){this-\>netMode = netMode;this-\>center\_number = center\_number;return 0;}**

**private:**

**static CJTMdvrMng \*m\_pInstance;**

**char ProcessName[64];**

**static NET\_PROTOCOL\_TYPE\_E netMode;**

**static int center\_number;//hexj,**** 中心编号 ****,1**** 开始**

**void\* m\_pPrivate;**

**int EventHandler(int type, int subtype, int enable, char \*pData, int DataLen);**

**int GetGpsProc(u64 &);**

**int UpdateACCState(int &Old\_ACC,int &accAbCnt ,VoltageSet\_t &m\_lowVol,time\_t &paraSysnTime,time\_t curtime);**

**int RegionLineAlm(POS\_ALM\_TYPE type, int Switch, void \*pExtData,void\* args);**

**};**

**#endif //\_\_JT\_MDVR\_MANAGE\_\_**

**#include \<string.h\>**

**#include \<stdlib.h\>**

**#include \<stdio.h\>**

**#include "comm\_dog.h"**

**#include "comm\_unixdomain.h"**

**#include "config.h"**

**#include "GpsMngInterface.h"**

**#include "JTMdvr.h"**

**#include "comm\_misc.h"**

**#include "LogMngInterface.h"**

**#define JT\_GPS\_IF\_DOMAIN "/tmp/jtgpsinterface.listen"**

**struct jt\_mdvr\_mng\_st**

**{**

**CommonThread \*pThread;**

**void\* args;**

**EventCallBack pEventFun;**

**void\* pEventArgs;**

**void\* pUnixDomain;**

**};**

**static void JTMdvrMngThread(CommonThread \*lt, void \*args)**

**{**

**jt\_mdvr\_mng\_st\* pTmp = (jt\_mdvr\_mng\_st\*)args;**

**CJTMdvrMng\* pMdrMng = (CJTMdvrMng\*)pTmp-\>args;**

**pMdrMng-\>JTMdvrMngProc(lt);**

**}**

**CJTMdvrMng\* CJTMdvrMng::m\_pInstance = NULL;**

**NET\_PROTOCOL\_TYPE\_E CJTMdvrMng::netMode = PROTOCOL\_808;**

**int CJTMdvrMng::center\_number = 0;**

**CJTMdvrMng::CJTMdvrMng()**

**{**

**m\_pPrivate = malloc(sizeof(jt\_mdvr\_mng\_st));**

**memset(m\_pPrivate, 0, sizeof(jt\_mdvr\_mng\_st));**

**memset(ProcessName, 0, sizeof(ProcessName));**

**}**

**CJTMdvrMng::~CJTMdvrMng()**

**{**

**if(m\_pPrivate != NULL)**

**{**

**free(m\_pPrivate);**

**m\_pPrivate = NULL;**

**}**

**}**

**CJTMdvrMng\* CJTMdvrMng::Instance()**

**{**

**if(NULL == m\_pInstance)**

**{**

**m\_pInstance = new CJTMdvrMng();**

**}**

**return m\_pInstance;**

**}**

**void CJTMdvrMng::JTMdvrMngProc(CommonThread \*lt)**

**{**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**char DogName[64]={0};**

**sprintf(DogName, "%sMng", ProcessName);**

**CCommDog \*pCommDog = CCommDog::Instance();**

**int DogId = pCommDog-\>RegisterDog(DogName, 120);**

**int lastacc = 0,accAbCnt = 0;**

**time\_t curtime = 0,feedDogTm = 0,paraSysnTime = 0;**

**VoltageSet\_t lowVol = {0};**

**CConfig::Instance()-\>GetConfig(&lowVol);**

**while (ThreadGetStatus(lt))**

**{**

**curtime = time(0);**

**UpdateACCState(lastacc,accAbCnt,lowVol,paraSysnTime,curtime);**

**if(abs(curtime - feedDogTm) \> 5)**

**{**

**pCommDog-\>FeedDog(DogId);**

**feedDogTm = curtime;**

**}**

**usleep(100\*1000);**

**}**

**pCommDog-\>DeregisterDog(DogId);**

**}**

**int CJTMdvrMng::Start(char \*process\_name, NET\_PROTOCOL\_TYPE\_E netMode, int center\_number)**

**{**

**if(m\_pPrivate == NULL)**

**{**

**return -1;**

**}**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**pHandle-\>pThread = ThreadInit();**

**if(pHandle-\>pThread == NULL)**

**{**

**return -1;**

**}**

**pHandle-\>args = this;**

**strcpy(ProcessName, process\_name);**

**this-\>netMode= netMode;**

**this-\>center\_number= center\_number;**

**char threadinfo[64];**

**sprintf(threadinfo, "Start JT MDVR Manage %s", process\_name);**

**int ret = ThreadStart(pHandle-\>pThread, JTMdvrMngThread, (void\*)pHandle, threadinfo);**

**if (ret \< 0)**

**{**

**ThreadUninit(pHandle-\>pThread);**

**return -1;**

**}**

**return 0;**

**}**

**int CJTMdvrMng::Stop()**

**{**

**if(m\_pPrivate == NULL)**

**{**

**return -1;**

**}**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**if (ThreadGetStatus(pHandle-\>pThread))**

**{**

**ThreadStop(pHandle-\>pThread);**

**}**

**ThreadUninit(pHandle-\>pThread);**

**return 0;**

**}**

**int CJTMdvrMng::JtGpsInterfaceProc(int fd, struct sockaddr\_un \*addr, socklen\_t \*addrlen, char \*recvbuf, int recvlen, void\* args)**

**{**

**char Tmpbuf[4096] = {0};**

**EVENT\_E Event;**

**int EventLen = 0;**

**int ret = ParseRecvEvent((unsigned char\*)recvbuf, recvlen, &Event, &EventLen, (void \*)Tmpbuf);**

**if(ret \< 0)**

**{**

**return -1;**

**}**

**CJTMdvrMng \*pthis = (CJTMdvrMng \*)args;**

**switch(Event)**

**{**

**case EVENT\_DISK\_EVENT:**

**{**

**DiskOpCmd\_t\* pDiskStateCmd = (DiskOpCmd\_t\*)Tmpbuf;**

**if(pDiskStateCmd-\>Type == DISK\_OP\_EVENT\_FAT32\_NO\_SPACE)**

**{**

**int enable = (int)pDiskStateCmd-\>DiskType;**

**pthis-\>EventHandler(ALARM\_SPECIAL\_REC,0,enable, NULL, 0);**

**}**

**}**

**break;**

**case EVENT\_ALARM:**

**{**

**AlarmCmd\_t\* pAlarmCmd = (AlarmCmd\_t\*)Tmpbuf;**

**switch(pAlarmCmd-\>Type)**

**{**

**case EVENT\_ALARM\_IO:**

**pthis-\>EventHandler(ALARM\_IO, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_ACCELERATION:**

**if(pAlarmCmd-\>subType == COLLISIONALARM)**

**pthis-\>EventHandler(ALARM\_HIT, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == ROLLOVERALARM)**

**pthis-\>EventHandler(ALARM\_TILT, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == ACCELERATIONALARM)**

**pthis-\>EventHandler(ALARM\_ACCELERATINO, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == SHARPSLOWDOWNALARM)**

**pthis-\>EventHandler(ALARM\_SHARPSLOWDOWN, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == SHARPTURNALARM)**

**pthis-\>EventHandler(ALARM\_SHAARPTURN, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_VIDEO\_LOST:**

**pthis-\>EventHandler(ALARM\_VIDEO\_LOST, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_DISK:**

**// pthis-\>EventHandler(ALARM\_DISK\_ABNORMAL, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**pthis-\>EventHandler(ALARM\_DISK\_ABNORMAL, ((pAlarmCmd-\>subType-1)\<0?0:(pAlarmCmd-\>subType-1)), pAlarmCmd-\>enable, NULL, 0);//****调整磁盘报警上报类型与设备对应 **** 2020.09.21 ssx**

**break;**

**case EVENT\_ALARM\_GNSS\_ERR://GNSS**  **模块发生故障**

**pthis-\>EventHandler(ALARM\_GNSS\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_GNSS\_CUT://GNSS**  **天线未接或被剪断**

**pthis-\>EventHandler(ALARM\_GNSS\_CUT, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_GNSS\_SHORT://GNSS**  **天线短路**

**pthis-\>EventHandler(ALARM\_GNSS\_SHORT, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_MOVEDETECT:**

**break;**

**case EVENT\_ALARM\_OCC\_DETECT:**

**pthis-\>EventHandler(ALARM\_VIDEO\_OD, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_LCD:**

**pthis-\>EventHandler(ALARM\_LCD\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_TTS:**

**pthis-\>EventHandler(ALARM\_TTS\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_CAM:**

**pthis-\>EventHandler(ALARM\_CAMERA\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_IC:**

**pthis-\>EventHandler(ALARM\_IC\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_VSS:**

**pthis-\>EventHandler(ALARM\_VSS\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_OIL:**

**pthis-\>EventHandler(ALARM\_OIL\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_LOSTA:**

**pthis-\>EventHandler(ALARM\_LOST, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_POWER\_LOW:**

**pthis-\>EventHandler(ALARM\_POWER\_LOW, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_OVERSPEED:**

**// break;**

**case EVENT\_ALARM\_REGION:**

**// break;**

**case EVENT\_ALARM\_OFF\_LINE:**

**// break;**

**case EVENT\_ALARM\_DRIVE\_TIME\_ERR:**

**// break;**

**case EVENT\_ALARM\_STOP\_TIMEOUT:**

**// break;**

**case EVENT\_ALARM\_DRIVE\_TIMEOUT:**

**// break;**

**case EVENT\_ALARM\_PRE\_OVERSPEED:**

**{**

**if(pAlarmCmd-\>alarmArgsLength \> 0)**

**{**

**LineFatiAlm\_t \*tmpFatigue=(LineFatiAlm\_t \*)(Tmpbuf + sizeof(AlarmCmd\_t));**

**pthis-\>RegionLineAlm(tmpFatigue-\>AlarmType, tmpFatigue-\>AlarmStatus,(void \*)tmpFatigue-\>data, args);**

**}**

**}**

**break;**

**case EVENT\_ALARM\_FATIGUE:**

**pthis-\>EventHandler(ALARM\_FATIGUE, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_PRE\_FATIGUE:**

**pthis-\>EventHandler(ALARM\_FATIGUE\_PREV, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_VOLTAGE:**

**{**

**if(3 == pAlarmCmd-\>subType)//****掉电报警**

**{**

**pthis-\>EventHandler(ALARM\_POWER\_OFF, 0, pAlarmCmd-\>enable, NULL, 0);**

**}**

**else if(1 == pAlarmCmd-\>subType)//****欠压报警**

**{**

**pthis-\>EventHandler(ALARM\_POWER\_LOW, 0, pAlarmCmd-\>enable, NULL, 0);**

**}**

**}**

**break;**

**#if defined(QD3556V2)**

**case EVENT\_ALARM\_EXTERN:**

**pthis-\>EventHandler(ALARM\_DOOR\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**#endif**

**default:**

**return 0;**

**}**

**}**

**break;**

**case EVENT\_POWEROFF:**

**{**

**PowerOffCmd\_t\* pPowerOffCmd = (PowerOffCmd\_t\*)Tmpbuf;**

**if(pPowerOffCmd-\>Type == 2) //****只检测掉电报警**

**pthis-\>EventHandler(ALARM\_POWER\_OFF, 0, 1, NULL, 0);**

**}**

**break;**

**case EVENT\_USER\_DEFINE:**

**{**

**UsrDefineCmd\_t \*pUsrDefineCmd = (UsrDefineCmd\_t\*)Tmpbuf;**

**if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_ALARM\_REPORT)**

**{**

**pthis-\>EventHandler(EVENT\_NLDZ\_ALARM\_REPORT, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_USR\_808\_CMD\_PACKET)**

**{**

**pthis-\>EventHandler(EVENT\_808\_CMD\_PACKET, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_USR\_808\_POS\_EXT)**

**{**

**SerialPassCmd\_t\* pSerialPassCmd = (SerialPassCmd\_t\*)pUsrDefineCmd-\>Data;**

**pthis-\>EventHandler(EVENT\_808\_POS\_EXT, pSerialPassCmd-\>MaxCount, pSerialPassCmd-\>AddId, pSerialPassCmd-\>SerialData, pSerialPassCmd-\>SerialDataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_SEND\_FTP\_CONTROL)**

**{**

**JtFtpTransStateCtrl \*ftpFile = (JtFtpTransStateCtrl \*)pUsrDefineCmd-\>Data;**

**pthis-\>EventHandler(EVENT\_FTP\_RESULT\_PACKET, ftpFile-\>seq, ftpFile-\>result, 0, 0);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_ID\_CONNECT2)**

**{**

**pthis-\>EventHandler(EVENT\_CONNECT\_CTL\_MSG\_0, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_ID\_CONNECT)**

**{**

**pthis-\>EventHandler(EVENT\_CONNECT\_CTL\_MSG\_1, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_TTX)**

**{**

**pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_TTX, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_HZSJ)**

**{**

**pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_HZSJ, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_HD\_LCD\_LED)**

**{**

**Jt808Transpt\_t\* pTmp = (Jt808Transpt\_t\*)pUsrDefineCmd-\>Data;**

**pthis-\>EventHandler(EVENT\_TRANSIT, pTmp-\>type, 0, (char\*)pTmp-\>Data, pTmp-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_F1)**

**{**

**pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_F1, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_QUERY\_REPORT)**

**{**

**pthis-\>EventHandler(EVENT\_PERIPHERALS\_QUERY\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_UPGRADE\_RES)**

**{**

**pthis-\>EventHandler(EVENT\_PERIPHERALS\_UPGRADE\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_QUERY\_PARAM\_RES)**

**{**

**pthis-\>EventHandler(EVENT\_YC\_QUERY\_PARAM\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_SET\_RUN\_INFO)**

**{**

**pthis-\>EventHandler(EVENT\_SET\_RUN\_INFO\_808, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**}**

**break;**

**case EVENT\_CONFIG\_UPDATE:**

**{**

**ConfigCmd\_t\* pConfigCmd = (ConfigCmd\_t\*)Tmpbuf;**

**if(pConfigCmd-\>Type == NET\_PARA\_SET)**

**{**

**WriteDbgLog(2, "EVENT\_CONFIG\_UPDATE: NET\_PARA\_SET jtgps\n");**

**UsrDefineCmd\_t m\_Cmd;**

**m\_Cmd.Type = EVENT4\_DATA\_SYNC;**

**m\_Cmd.DataLen = 0;**

**DistributeEvent(EVENT\_USR\_DEF4, sizeof(UsrDefineCmd\_t), (void \*)&m\_Cmd);**

**sleep(2);**

**exit(0);**

**}**

**else if(pConfigCmd-\>Type == SPEED\_PARA\_SET)**

**{**

**#if !defined(JT\_PROTOCOL)**

**WriteDbgLog(2, "EVENT\_CONFIG\_UPDATE: SPEED\_PARA\_SET jtgps\n");**

**UsrDefineCmd\_t m\_Cmd;**

**m\_Cmd.Type = EVENT4\_DATA\_SYNC;**

**m\_Cmd.DataLen = 0;**

**DistributeEvent(EVENT\_USR\_DEF4, sizeof(UsrDefineCmd\_t), (void \*)&m\_Cmd);**

**sleep(2);**

**exit(0);**

**#else**

**SpeedSet\_t speedSet = {0};**

**CConfig \*pConfig = CConfig::Instance();**

**pConfig-\>GetConfig(&speedSet);**

**pthis-\>EventHandler(EVENT\_JT\_SPEED\_MODE, 0, 0, (char\*)&speedSet.source, sizeof(speedSet.source));**

**#endif**

**}**

**}**

**break;**

**case EVENT\_JT808\_EVENT:**

**{**

**Jt808Event\_t\* pJt808Event = (Jt808Event\_t\*)Tmpbuf;**

**switch(pJt808Event-\>Type)**

**{**

**case EVENT\_REPORT:**

**{**

**EventReportCmd\_t\* pEventReportCmd = (EventReportCmd\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_REPORT, 0, 0, (char\*)pEventReportCmd, sizeof(EventReportCmd\_t));**

**}**

**break;**

**case EVENT\_QUESTION\_RES:**

**{**

**EventIssuedCmd\_t\* pEventIssuedCmd = (EventIssuedCmd\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_QUESTION\_RES, 0, 0, (char\*)pEventIssuedCmd, sizeof(EventIssuedCmd\_t));**

**}**

**break;**

**case EVENT\_MENU\_RES:**

**{**

**EventMenuCmd\_t\* pEventMenuCmd = (EventMenuCmd\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_MENU\_RES, 0, 0, (char\*)pEventMenuCmd, sizeof(EventMenuCmd\_t));**

**}**

**break;**

**case EVENT\_PASSENGER\_TRAFFIC:**

**{**

**PassengerTraffic\_t\* pPassengerTraffic = (PassengerTraffic\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_PASSENGER\_TRAFFIC,0,0, (char\*)pPassengerTraffic, sizeof(PassengerTraffic\_t));**

**}**

**break;**

**case EVENT\_MEDIA\_DATA:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MEDIA\_DATA, 0, 0, 0, 0);**

**}**

**break;**

**case EVENT\_ELEC\_WAYBILL:**

**{**

**EventElecWaybill\_t\* pEventElecWaybill = (EventElecWaybill\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_ELEC\_WAYBILL,0,0, (char\*)pEventElecWaybill, strlen(pJt808Event-\>data));**

**}**

**break;**

**case EVENT\_MANUALLY\_REG:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MANUALLY\_REG,0,0, 0, 0);**

**}**

**break;**

**case EVENT\_MANUALLY\_AUTH:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MANUALLY\_AUTH,0,0, 0, 0);**

**}**

**break;**

**case EVENT\_MANUALLY\_LOGOUT:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MANUALLY\_LOGOUT,0,0, 0, 0);**

**}**

**break;**

**default:**

**break;**

**}**

**}**

**break;**

**case EVENT\_SERIAL\_CONTROL:**

**{**

**SerialCmd\_t \*pSerialCmd = (SerialCmd\_t\*)Tmpbuf;**

**switch(pSerialCmd-\>FunctionDef)**

**{**

**case COM\_TYPE\_EXPORT\_RECORD\_LOG:**

**/\*if(pSerialCmd-\>Cmd == 1)**

**{**

**pthis-\>EventHandler(EVENT\_JT\_19056\_COLLECT, 0, 0, (char\*)pSerialCmd-\>DataBuf, pSerialCmd-\>Datalen);**

**}\*/**

**break;**

**}**

**}**

**break;**

**case EVENT\_CAN\_CONTROL:**

**{**

**CanCmd\_t \*pCanCmd = (CanCmd\_t\*)Tmpbuf;**

**if(pCanCmd-\>Cmd == 1)**

**{**

**pthis-\>EventHandler(EVENT\_CANBUS, pCanCmd-\>ID, 0,(char\*)pCanCmd-\>DataBuf, pCanCmd-\>Datalen);**

**}**

**}**

**break;**

**case EVENT\_USR\_DEF2:**

**{**

**UsrDefineCmd\_t \*pUsrDefineCmd = (UsrDefineCmd\_t\*)Tmpbuf;**

**if(pUsrDefineCmd-\>Type == EVENT2\_FT\_TO\_808)**

**{**

**pthis-\>EventHandler(EVENT\_FTTRANSPT, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT2\_MCUSET\_JT808)**

**{**

**pthis-\>EventHandler(MCU\_SET\_808\_PARAMETER, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT2\_CLEAN\_AUTH)//HC 20190415**  **清除鉴权**

**{**

**pthis-\>EventHandler(STATE\_AUTH\_CLEAN, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}else if(pUsrDefineCmd-\>Type == EVENT2\_ADC\_VALUE)**

**{**

**pthis-\>EventHandler(EVENT2\_ADC\_VALUE, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**}**

**break;**

**case EVENT\_PULSE\_SPEED:**

**{**

**EventPulseSpeed\_t \*pEventPulseSpeed = (EventPulseSpeed\_t\*)Tmpbuf;**

**pthis-\>EventHandler(EVENT\_JT\_PULSE\_SPEED, 0, 0, (char\*)pEventPulseSpeed, sizeof(EventPulseSpeed\_t));**

**}**

**break;**

**case EVENT\_USR\_DEF4:**

**{**

**UsrDefineCmd\_t \*pUsrDefine = (UsrDefineCmd\_t\*)Tmpbuf;**

**if(pUsrDefine-\>Type==EVENT4\_FillPOS)**

**{**

**pthis-\>EventHandler(EVENT4\_FillPOS, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**else if(pUsrDefine-\>Type==EVENT4\_DATA\_SYNC)**

**{**

**pthis-\>EventHandler(EVENT4\_DATA\_SYNC, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**else if(pUsrDefine-\>Type==EVENT4\_IC\_CARD\_CHANGE)**

**{**

**pthis-\>EventHandler(EVENT4\_IC\_CARD\_CHANGE, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**else if(pUsrDefine-\>Type==EVENT4\_YL\_REPORT)**

**{**

**pthis-\>EventHandler(EVENT4\_YL\_REPORT, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}else if(pUsrDefine-\>Type==EVENT4\_ADAS\_DMS)**

**{**

**pthis-\>EventHandler(EVENT\_JT\_DS\_AI, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**#if defined(FSLX400JT)**

**else if(pUsrDefine-\>Type==EVENT4\_FSLX\_MCU\_808\_CMD)**

**{**

**pthis-\>EventHandler(EVENT\_FSLX\_MCU\_808\_CMD, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**#endif**

**break;**

**}**

**case EVENT\_GB19056\_EVENT:**

**{**

**GB19056Event\_t \*tmpGB19056 = (GB19056Event\_t\*)Tmpbuf;**

**switch(tmpGB19056-\>Type)**

**{**

**/\*case EVENT\_GB19056\_LINE\_FATI\_ALARM:**

**{**

**LineFatiAlm\_t \*tmpFatigue=(LineFatiAlm\_t \*)tmpGB19056-\>data;**

**pthis-\>RegionLineAlm(tmpFatigue-\>AlarmType, tmpFatigue-\>AlarmStatus,(void \*)tmpFatigue-\>data, args);**

**break;**

**}\*/**

**case EVENT\_GB19056\_POWER\_STATE:**

**{**

**EventPowerState\_t\* pEventPowerState = (EventPowerState\_t\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_19056\_POWER\_STATE,0,0, (char\*)pEventPowerState, sizeof(EventPowerState\_t));**

**break;**

**}**

**case EVENT\_GB19056\_COLLECT\_RES\_808:**

**{**

**if(tmpGB19056-\>workmode == 0) //jtgps****暂定中心 ****0**

**pthis-\>EventHandler(EVENT\_JT\_GB19056\_COLLECT\_RES,0,0, (char\*)tmpGB19056-\>data, sizeof(tmpGB19056-\>data));**

**break;**

**}**

**case EVENT\_GB19056\_IC\_CARD:**

**{**

**IcCard\_t\* pIcCard = (IcCard\_t\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_IC\_CARD\_INFOR,0,0, (char\*)pIcCard, sizeof(IcCard\_t));**

**break;**

**}**

**case EVENT\_GB19056\_IC\_CARD\_JIUTONG:**

**{**

**IcCard\_t\_jiutong\* pIcCard = (IcCard\_t\_jiutong\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_IC\_CARD\_INFOR\_JIUTONG, 0, 0, (char\*)pIcCard, sizeof(IcCard\_t\_jiutong));**

**break;**

**}**

**case EVENT\_GB19056\_SWT:**

**{**

**EventSwt\_t\* pEventSwt = (EventSwt\_t\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_19056\_SWT,0,0, (char\*)pEventSwt, sizeof(EventSwt\_t));**

**break;**

**}**

**}**

**}**

**break;**

**case EVENT\_MISC:**

**{**

**MiscCmd\_t\* pMiscCmd = (MiscCmd\_t\*)Tmpbuf;**

**if(netMode == PROTOCOL\_808 || netMode == PROTOCOL\_XIANG\_M1)**

**{**

**if(pMiscCmd-\>Type == MISC\_EVENT\_FILE\_UPLOAD)**

**{**

**AlarmCmd\_t\* pAlarmCmd = (AlarmCmd\_t\*)pMiscCmd-\>Data;**

**if(pAlarmCmd-\>subType == 1)//**  **图片**

**pthis-\>EventHandler(ALARM\_PIC\_AUTO\_UPLOAD, pAlarmCmd-\>subType, pAlarmCmd-\>enable, pMiscCmd-\>Data+ sizeof(AlarmCmd\_t), pAlarmCmd-\>alarmArgsLength);**

**}**

**}**

**}**

**break;**

**default:**

**break;**

**}**

**return 0;**

**}**

**int CJTMdvrMng::RegionLineAlm(POS\_ALM\_TYPE type, int Switch, void \*pExtData,void\* args)**

**{**

**u8 regionData[6] = {0};**

**int regionLen = 0;**

**LineDrvAlm\_t\* pDrvAlm;**

**RegionAlm\_t\* pRegionAlm;**

**CJTMdvrMng \*pthis = (CJTMdvrMng \*)args;**

**switch(type)**

**{**

**case OVER\_SPD\_ALM:**

**if(Switch == 0)**

**{**

**OverSpdAlm\_t\* pSpdAlm = (OverSpdAlm\_t\*)pExtData;**

**switch(pSpdAlm-\>PosType)**

**{**

**case NON\_TYPE:**

**WriteDbgLog(2, "****取消超速，无特定位置****\n");**

**break;**

**case ROUND\_TYPE:**

**WriteDbgLog(2, "****取消超速，圆形区域****: %d\n", pSpdAlm-\>PosId);**

**break;**

**case RECT\_TYPE:**

**WriteDbgLog(2, "****取消超速，矩形区域****: %d\n", pSpdAlm-\>PosId);**

**break;**

**case POLYGON\_TYPE:**

**WriteDbgLog(2, "****取消超速，多边形区域****: %d\n", pSpdAlm-\>PosId);**

**break;**

**case LINE\_TYPE:**

**WriteDbgLog(2, "****取消超速，线路****: %d\n", pSpdAlm-\>PosId);**

**break;**

**default:**

**return -1;**

**}**

**if(pSpdAlm-\>PosType == 0)**

**pthis-\>EventHandler(ALARM\_SPEED, 0, 0, (char \*)(&pSpdAlm-\>PosType), sizeof(pSpdAlm-\>PosType));**

**else**

**pthis-\>EventHandler(ALARM\_SPEED, 0, 0, (char \*)pExtData, sizeof(OverSpdAlm\_t));**

**}**

**else**

**{**

**OverSpdAlm\_t\* pSpdAlm = (OverSpdAlm\_t\*)pExtData;**

**switch(pSpdAlm-\>PosType)**

**{**

**case NON\_TYPE:**

**WriteDbgLog(2, "****超速，无特定位置****\n");**

**break;**

**case ROUND\_TYPE:**

**WriteDbgLog(2, "****超速，圆形区域****: %d\n", pSpdAlm-\>PosId);**

**break;**

**case RECT\_TYPE:**

**WriteDbgLog(2, "****超速，矩形区域****: %d\n", pSpdAlm-\>PosId);**

**break;**

**case POLYGON\_TYPE:**

**WriteDbgLog(2, "****超速，多边形区域****: %d\n", pSpdAlm-\>PosId);**

**break;**

**case LINE\_TYPE:**

**WriteDbgLog(2, "****超速，线路****: %d\n", pSpdAlm-\>PosId);**

**break;**

**default:**

**return -1;**

**}**

**//**  **当无特定位置时 则不需要附加**** ID qqin 2018/3/17**

**if(pSpdAlm-\>PosType == 0)**

**pthis-\>EventHandler(ALARM\_SPEED, 0, 1, (char \*)(&pSpdAlm-\>PosType), sizeof(pSpdAlm-\>PosType));**

**else**

**pthis-\>EventHandler(ALARM\_SPEED, 0, 1, (char \*)pExtData, sizeof(OverSpdAlm\_t));**

**}**

**break;**

**case REGION\_ALM:**

**pRegionAlm = (RegionAlm\_t\*)pExtData;**

**put8(regionData + regionLen, pRegionAlm-\>PosType); regionLen += 1;**

**putbe32(regionData + regionLen, pRegionAlm-\>PosId); regionLen += 4;**

**put8(regionData + regionLen, pRegionAlm-\>Direct); regionLen += 1;**

**switch(pRegionAlm-\>PosType)**

**{**

**case ROUND\_TYPE:**

**if(pRegionAlm-\>Direct == 0)**

**{**

**WriteDbgLog(2, "****进入圆形区域****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_AREA\_IN\_OUT, 0, 1, (char \*)regionData, regionLen);**

**}**

**else**

**{**

**WriteDbgLog(2, "****离开圆形区域****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_AREA\_IN\_OUT, 0, 0, (char \*)regionData, regionLen);**

**}**

**break;**

**case RECT\_TYPE:**

**if(pRegionAlm-\>Direct == 0)**

**{**

**WriteDbgLog(2, "****进入矩形区域****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_AREA\_IN\_OUT, 0, 1, (char \*)regionData, regionLen);**

**}**

**else**

**{**

**WriteDbgLog(2, "****离开矩形区域****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_AREA\_IN\_OUT, 0, 0, (char \*)regionData, regionLen);**

**}**

**break;**

**case POLYGON\_TYPE:**

**if(pRegionAlm-\>Direct == 0)**

**{**

**WriteDbgLog(2, "****进入多边形区域****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_AREA\_IN\_OUT, 0, 1, (char \*)regionData, regionLen);**

**}**

**else**

**{**

**WriteDbgLog(2, "****离开多边形区域****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_AREA\_IN\_OUT, 0, 0, (char \*)regionData, regionLen);**

**}**

**break;**

**case LINE\_TYPE:**

**if(pRegionAlm-\>Direct == 0)**

**{**

**WriteDbgLog(2, "****进入线路****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_LINE\_IN\_OUT, 0, 1, (char \*)regionData, regionLen);**

**}**

**else**

**{**

**WriteDbgLog(2, "****离开线路****: %d\n", pRegionAlm-\>PosId);**

**pthis-\>EventHandler(ALARM\_LINE\_IN\_OUT, 0, 0, (char \*)regionData, regionLen);**

**}**

**default:**

**break;**

**return -1;**

**}**

**break;**

**case PRE\_SKEWING\_ALM:**

**pRegionAlm = (RegionAlm\_t\*)pExtData;**

**if(pRegionAlm-\>Direct == 0)**

**{**

**pthis-\>EventHandler(ALARM\_OFF\_LINE, 0, 0, (char \*)regionData, regionLen);**

**}else**

**{**

**pthis-\>EventHandler(ALARM\_OFF\_LINE, 0, 1, (char \*)regionData, regionLen);**

**}**

**break;**

**case LINE\_DRV\_ALM:**

**pDrvAlm = (LineDrvAlm\_t\*)pExtData;**

**if(pDrvAlm-\>Result == 0)**

**{**

**WriteDbgLog(2, "****路段 ****: %d**  **行驶时间**** : %d **** 不足****\n", pDrvAlm-\>PosId, pDrvAlm-\>DrvTime);**

**}**

**else**

**{**

**WriteDbgLog(2, "****路段 ****: %d**  **行驶时间**** : %d **** 过长****\n", pDrvAlm-\>PosId, pDrvAlm-\>DrvTime);**

**}**

**pthis-\>EventHandler(ALARM\_LACK\_TIME, 0, 1, (char \*)pExtData, sizeof(LineDrvAlm\_t));**

**break;**

**case STOP\_TIMEOUT\_ALM:**

**if(Switch == 0)**

**{**

**WriteDbgLog(2, "****取消超时停车****\n");**

**pthis-\>EventHandler(ALARM\_STOP\_TIMEOUT, 0, 0, NULL, 0);**

**}**

**else**

**{**

**WriteDbgLog(2, "****开始超时停车****\n");**

**pthis-\>EventHandler(ALARM\_STOP\_TIMEOUT, 0, 1, NULL, 0);**

**}**

**break;**

**case DRIVE\_TIMEOUT\_ALM:**

**if(Switch == 0)**

**{**

**WriteDbgLog(2, "****取消当天累计驾驶超时****\n");**

**pthis-\>EventHandler(ALARM\_TOTAL\_DRIVE, 0, 0, NULL, 0);**

**}**

**else**

**{**

**WriteDbgLog(2, "****开始当天累计驾驶超时****\n");**

**pthis-\>EventHandler(ALARM\_TOTAL\_DRIVE, 0, 1, NULL, 0);**

**}**

**break;**

**case PREDICTION\_SPD\_ALM:**

**if(Switch == 0)**

**{**

**WriteDbgLog(2, "****取消超速预警****\n");**

**pthis-\>EventHandler(ALARM\_SPEED\_PREV, 0, 0, NULL, 0);**

**}**

**else**

**{**

**WriteDbgLog(2, "****开始超速预警****\n");**

**pthis-\>EventHandler(ALARM\_SPEED\_PREV, 0, 1, NULL, 0);**

**}**

**break;**

**case PRE\_FATIGUE\_DRIVE\_ALM:**

**if(Switch == 0)**

**{**

**WriteDbgLog(2, "****取消疲劳预警****\n");**

**pthis-\>EventHandler(ALARM\_FATIGUE\_PREV, 0, 0, NULL, 0);**

**}**

**else**

**{**

**WriteDbgLog(2, "****开始疲劳预警****\n");**

**pthis-\>EventHandler(ALARM\_FATIGUE\_PREV, 0, 1, NULL, 0);**

**}**

**break;**

**case FATIGUE\_DRIVE\_ALM:**

**if(Switch == 0)**

**{**

**WriteDbgLog(2, "****取消疲劳驾驶****\n");**

**pthis-\>EventHandler(ALARM\_FATIGUE, 0, 0, NULL, 0);**

**}**

**else**

**{**

**WriteDbgLog(2, "****开始疲劳驾驶****\n");**

**pthis-\>EventHandler(ALARM\_FATIGUE, 0, 1, NULL, 0);**

**}**

**break;**

**default:**

**return -1;**

**}**

**return 0;**

**}**

**int CJTMdvrMng::EventHandler(int type, int subtype, int enable, char \*pData, int DataLen)**

**{**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**if(pHandle-\>pEventFun != NULL)**

**{**

**pHandle-\>pEventFun(type, subtype, enable, pData, DataLen, pHandle-\>pEventArgs);**

**}**

**return 0;**

**}**

**int CJTMdvrMng::StartUnixDomain()**

**{**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**long long EventBitmap = 0;**

**char tmpUnixDomainName[128] = {0};**

**snprintf(tmpUnixDomainName, sizeof(tmpUnixDomainName) - 1, "/tmp/%s.listen", ProcessName);**

**setbit(&EventBitmap, EVENT\_MISC);**

**setbit(&EventBitmap, EVENT\_ALARM);**

**setbit(&EventBitmap, EVENT\_POWEROFF);**

**setbit(&EventBitmap, EVENT\_CONFIG\_UPDATE);**

**setbit(&EventBitmap, EVENT\_DISK\_EVENT);**

**setbit(&EventBitmap, EVENT\_USER\_DEFINE);**

**setbit(&EventBitmap, EVENT\_JT808\_EVENT);**

**setbit(&EventBitmap, EVENT\_SERIAL\_CONTROL);**

**setbit(&EventBitmap, EVENT\_CAN\_CONTROL);**

**setbit(&EventBitmap, EVENT\_USR\_DEF2);**

**setbit(&EventBitmap, EVENT\_PULSE\_SPEED);**

**setbit(&EventBitmap, EVENT\_USR\_DEF4);**

**setbit(&EventBitmap, EVENT\_GB19056\_EVENT);**

**UnregisterEventReciever(tmpUnixDomainName);**

**RegisterEventReciever(tmpUnixDomainName, EventBitmap);**

**pHandle-\>pUnixDomain = StartUnixDomainListen(tmpUnixDomainName, CJTMdvrMng::JtGpsInterfaceProc, this);**

**return 0;**

**}**

**int CJTMdvrMng::StopUnixDomain()**

**{**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**StopUnixDomainListen(pHandle-\>pUnixDomain);**

**pHandle-\>pUnixDomain = NULL;**

**return 0;**

**}**

**int CJTMdvrMng::InitEventCB(EventCallBack cbFunc, void \*cbArgs)**

**{**

**if(m\_pPrivate == NULL)**

**{**

**return -1;**

**}**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**// not fix**

**pHandle-\>pEventFun = cbFunc;**

**pHandle-\>pEventArgs = cbArgs;**

**return 0;**

**}**

**int CJTMdvrMng::QuitEventCB(EventCallBack cbFunc, void \*cbArgs)**

**{**

**if(m\_pPrivate == NULL)**

**{**

**return -1;**

**}**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**// not fix**

**pHandle-\>pEventFun = NULL;**

**pHandle-\>pEventArgs = NULL;**

**return 0;**

**}**

**int CJTMdvrMng::GetGpsProc(u64 &last\_gps\_get)**

**{**

**//if(pGpsReader != NULL)**

**{**

**u64 nowms = msecdiff(0);**

**if(llabs(nowms - last\_gps\_get) \>= 1000)**

**{**

**last\_gps\_get = nowms;**

**GPSData\_s GpsInfo;**

**nmeaGPRMC RMC;**

**memset(&GpsInfo,0,sizeof(GPSData\_s));**

**memset(&RMC, 0, sizeof(nmeaGPRMC));**

**CGpsReader\* pGpsReader = CGpsReader::Instance();**

**if(pGpsReader-\>ReadGps(&GpsInfo, 1) \> 0)**

**{**

**RMC.status = GpsInfo.cGpsStatus;**

**RMC.lat = GpsInfo.dLatitude;**

**RMC.ns = GpsInfo.cDirectionLatitude;**

**RMC.lon = GpsInfo.dLongitude;**

**RMC.ew = GpsInfo.cDirectionLongitude;**

**RMC.speed = ((double)GpsInfo.usSpeed) / 185;**

**if(GpsInfo.cSpeedUnit == 1 )**

**{**

**RMC.speed = ((double)GpsInfo.usSpeed) / 115;**

**}**

**RMC.direction = GpsInfo.usGpsAngle / 100.0;**

**}**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**if(pHandle-\>pEventFun != NULL)**

**{**

**pHandle-\>pEventFun(EVENT\_GPS\_DATA, GpsInfo.cSatelliteCount/\*cls add 2018.05.20\*/, 0, (char\*)&RMC, sizeof(nmeaGPRMC), pHandle-\>pEventArgs);**

**}**

**}**

**}**

**return -1;**

**}**

**int CJTMdvrMng::UpdateACCState(int &Old\_ACC,int &accAbCnt ,VoltageSet\_t &m\_lowVol,time\_t &paraSysnTime,time\_t curtime)**

**{**

**SensorInfo\_t pdata;**

**CJTInterfaceApp \*pJTInterfaceApp = CJTInterfaceApp::Instance(NULL);**

**pJTInterfaceApp-\>GetData(&pdata);**

**if(Old\_ACC != pdata.AccState)**

**{**

**if(++accAbCnt \>= 3)// cls add 2018.12.19**  **防抖**

**{**

**accAbCnt = 0;**

**Old\_ACC = pdata.AccState;**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**if(pHandle-\>pEventFun != NULL)**

**{**

**pHandle-\>pEventFun(ALARM\_ACCOPEN, 0, pdata.AccState, (char \*)&pdata.AccState, 1, pHandle-\>pEventArgs);**

**}**

**}**

**}**

**else**

**{**

**accAbCnt = 0;**

**}**

**//add 20180904**

**/\*\*\*\*\*\*\*\*\***** 判断欠压 ****\*\*\*\*\*\*\*\*\*\*/**

**static int oldunderVoltage = 0;**

**static int count = 0;**

**int nowUnderVoltage = 0;**

**if(abs(curtime - paraSysnTime) \> 30)**

**{**

**CConfig::Instance()-\>GetConfig(&m\_lowVol);**

**paraSysnTime = curtime;**

**}**

**if(access("/tmp/PowerVoltage\_Debug", F\_OK) == 0)**

**{**

**printf("pdata.PowerVoltage = %d %d\n", pdata.PowerVoltage, m\_lowVol.lowalarm.limit);**

**}**

**if ((pdata.PowerVoltage/10) \< m\_lowVol.lowalarm.limit && m\_lowVol.lowalarm.enable)**

**{**

**if(count++ \>= 3)**

**nowUnderVoltage = 1;**

**}**

**else**

**{**

**nowUnderVoltage = 0;**

**count = 0;**

**}**

**#if 0**

**if ((pdata.PowerState/10) != 1) //****掉电解除欠压报警**

**{**

**nowUnderVoltage = 0;**

**}**

**#endif**

**if(oldunderVoltage != nowUnderVoltage)**

**{**

**oldunderVoltage = nowUnderVoltage;**

**jt\_mdvr\_mng\_st\* pHandle = (jt\_mdvr\_mng\_st\*)m\_pPrivate;**

**if(pHandle-\>pEventFun != NULL)**

**{**

**pHandle-\>pEventFun(ALARM\_POWER\_LOW, 0, nowUnderVoltage, NULL, 0, pHandle-\>pEventArgs);**

**}**

**}**

**/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/**

**return 0;**

**}**

# appendix 2 Alarm receiving reference code

**int CJTMdvrMng::JtGpsInterfaceProc(int fd, struct sockaddr\_un \*addr, socklen\_t \*addrlen, char \*recvbuf, int recvlen, void\* args)**

**{**

**char Tmpbuf[4096] = {0};**

**EVENT\_E Event;**

**int EventLen = 0;**

**int ret = ParseRecvEvent((unsigned char\*)recvbuf, recvlen, &Event, &EventLen, (void \*)Tmpbuf);**

**if(ret \< 0)**

**{**

**return -1;**

**}**

**CJTMdvrMng \*pthis = (CJTMdvrMng \*)args;**

**switch(Event)**

**{**

**case EVENT\_DISK\_EVENT:**

**{**

**DiskOpCmd\_t\* pDiskStateCmd = (DiskOpCmd\_t\*)Tmpbuf;**

**if(pDiskStateCmd-\>Type == DISK\_OP\_EVENT\_FAT32\_NO\_SPACE)**

**{**

**int enable = (int)pDiskStateCmd-\>DiskType;**

**pthis-\>EventHandler(ALARM\_SPECIAL\_REC,0,enable, NULL, 0);**

**}**

**}**

**break;**

**case EVENT\_ALARM:**

**{**

**AlarmCmd\_t\* pAlarmCmd = (AlarmCmd\_t\*)Tmpbuf;**

**switch(pAlarmCmd-\>Type)**

**{**

**case EVENT\_ALARM\_IO:**

**pthis-\>EventHandler(ALARM\_IO, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_ACCELERATION:**

**if(pAlarmCmd-\>subType == COLLISIONALARM)**

**pthis-\>EventHandler(ALARM\_HIT, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == ROLLOVERALARM)**

**pthis-\>EventHandler(ALARM\_TILT, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == ACCELERATIONALARM)**

**pthis-\>EventHandler(ALARM\_ACCELERATINO, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == SHARPSLOWDOWNALARM)**

**pthis-\>EventHandler(ALARM\_SHARPSLOWDOWN, 0, pAlarmCmd-\>enable, NULL, 0);**

**else if(pAlarmCmd-\>subType == SHARPTURNALARM)**

**pthis-\>EventHandler(ALARM\_SHAARPTURN, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_VIDEO\_LOST:**

**pthis-\>EventHandler(ALARM\_VIDEO\_LOST, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_DISK:**

**// pthis-\>EventHandler(ALARM\_DISK\_ABNORMAL, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**pthis-\>EventHandler(ALARM\_DISK\_ABNORMAL, ((pAlarmCmd-\>subType-1)\<0?0:(pAlarmCmd-\>subType-1)), pAlarmCmd-\>enable, NULL, 0);//****调整磁盘报警上报类型与设备对应 **** 2020.09.21 ssx**

**break;**

**case EVENT\_ALARM\_GNSS\_ERR://GNSS**  **模块发生故障**

**pthis-\>EventHandler(ALARM\_GNSS\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_GNSS\_CUT://GNSS**  **天线未接或被剪断**

**pthis-\>EventHandler(ALARM\_GNSS\_CUT, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_GNSS\_SHORT://GNSS**  **天线短路**

**pthis-\>EventHandler(ALARM\_GNSS\_SHORT, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_MOVEDETECT:**

**break;**

**case EVENT\_ALARM\_OCC\_DETECT:**

**pthis-\>EventHandler(ALARM\_VIDEO\_OD, pAlarmCmd-\>subType, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_LCD:**

**pthis-\>EventHandler(ALARM\_LCD\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_TTS:**

**pthis-\>EventHandler(ALARM\_TTS\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_CAM:**

**pthis-\>EventHandler(ALARM\_CAMERA\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_IC:**

**pthis-\>EventHandler(ALARM\_IC\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_VSS:**

**pthis-\>EventHandler(ALARM\_VSS\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_OIL:**

**pthis-\>EventHandler(ALARM\_OIL\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_LOSTA:**

**pthis-\>EventHandler(ALARM\_LOST, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_POWER\_LOW:**

**pthis-\>EventHandler(ALARM\_POWER\_LOW, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_OVERSPEED:**

**// break;**

**case EVENT\_ALARM\_REGION:**

**// break;**

**case EVENT\_ALARM\_OFF\_LINE:**

**// break;**

**case EVENT\_ALARM\_DRIVE\_TIME\_ERR:**

**// break;**

**case EVENT\_ALARM\_STOP\_TIMEOUT:**

**// break;**

**case EVENT\_ALARM\_DRIVE\_TIMEOUT:**

**// break;**

**case EVENT\_ALARM\_PRE\_OVERSPEED:**

**{**

**if(pAlarmCmd-\>alarmArgsLength \> 0)**

**{**

**LineFatiAlm\_t \*tmpFatigue=(LineFatiAlm\_t \*)(Tmpbuf + sizeof(AlarmCmd\_t));**

**pthis-\>RegionLineAlm(tmpFatigue-\>AlarmType, tmpFatigue-\>AlarmStatus,(void \*)tmpFatigue-\>data, args);**

**}**

**}**

**break;**

**case EVENT\_ALARM\_FATIGUE:**

**pthis-\>EventHandler(ALARM\_FATIGUE, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_PRE\_FATIGUE:**

**pthis-\>EventHandler(ALARM\_FATIGUE\_PREV, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**case EVENT\_ALARM\_VOLTAGE:**

**{**

**if(3 == pAlarmCmd-\>subType)//****掉电报警**

**{**

**pthis-\>EventHandler(ALARM\_POWER\_OFF, 0, pAlarmCmd-\>enable, NULL, 0);**

**}**

**else if(1 == pAlarmCmd-\>subType)//****欠压报警**

**{**

**pthis-\>EventHandler(ALARM\_POWER\_LOW, 0, pAlarmCmd-\>enable, NULL, 0);**

**}**

**}**

**break;**

**#if defined(QD3556V2)**

**case EVENT\_ALARM\_EXTERN:**

**pthis-\>EventHandler(ALARM\_DOOR\_ERR, 0, pAlarmCmd-\>enable, NULL, 0);**

**break;**

**#endif**

**default:**

**return 0;**

**}**

**}**

**break;**

**case EVENT\_POWEROFF:**

**{**

**PowerOffCmd\_t\* pPowerOffCmd = (PowerOffCmd\_t\*)Tmpbuf;**

**if(pPowerOffCmd-\>Type == 2) //****只检测掉电报警**

**pthis-\>EventHandler(ALARM\_POWER\_OFF, 0, 1, NULL, 0);**

**}**

**break;**

**case EVENT\_USER\_DEFINE:**

**{**

**UsrDefineCmd\_t \*pUsrDefineCmd = (UsrDefineCmd\_t\*)Tmpbuf;**

**if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_ALARM\_REPORT)**

**{**

**pthis-\>EventHandler(EVENT\_NLDZ\_ALARM\_REPORT, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_USR\_808\_CMD\_PACKET)**

**{**

**pthis-\>EventHandler(EVENT\_808\_CMD\_PACKET, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_USR\_808\_POS\_EXT)**

**{**

**SerialPassCmd\_t\* pSerialPassCmd = (SerialPassCmd\_t\*)pUsrDefineCmd-\>Data;**

**pthis-\>EventHandler(EVENT\_808\_POS\_EXT, pSerialPassCmd-\>MaxCount, pSerialPassCmd-\>AddId, pSerialPassCmd-\>SerialData, pSerialPassCmd-\>SerialDataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_SEND\_FTP\_CONTROL)**

**{**

**JtFtpTransStateCtrl \*ftpFile = (JtFtpTransStateCtrl \*)pUsrDefineCmd-\>Data;**

**pthis-\>EventHandler(EVENT\_FTP\_RESULT\_PACKET, ftpFile-\>seq, ftpFile-\>result, 0, 0);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_ID\_CONNECT2)**

**{**

**pthis-\>EventHandler(EVENT\_CONNECT\_CTL\_MSG\_0, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_ID\_CONNECT)**

**{**

**pthis-\>EventHandler(EVENT\_CONNECT\_CTL\_MSG\_1, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_TTX)**

**{**

**pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_TTX, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_HZSJ)**

**{**

**pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_HZSJ, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_HD\_LCD\_LED)**

**{**

**Jt808Transpt\_t\* pTmp = (Jt808Transpt\_t\*)pUsrDefineCmd-\>Data;**

**pthis-\>EventHandler(EVENT\_TRANSIT, pTmp-\>type, 0, (char\*)pTmp-\>Data, pTmp-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_TRANSIT\_F1)**

**{**

**pthis-\>EventHandler(EVENT\_TRANSIT, TRANSIT\_F1, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_QUERY\_REPORT)**

**{**

**pthis-\>EventHandler(EVENT\_PERIPHERALS\_QUERY\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_UPGRADE\_RES)**

**{**

**pthis-\>EventHandler(EVENT\_PERIPHERALS\_UPGRADE\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_YINCAA\_QUERY\_PARAM\_RES)**

**{**

**pthis-\>EventHandler(EVENT\_YC\_QUERY\_PARAM\_RES, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT\_SET\_RUN\_INFO)**

**{**

**pthis-\>EventHandler(EVENT\_SET\_RUN\_INFO\_808, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**}**

**break;**

**case EVENT\_CONFIG\_UPDATE:**

**{**

**ConfigCmd\_t\* pConfigCmd = (ConfigCmd\_t\*)Tmpbuf;**

**if(pConfigCmd-\>Type == NET\_PARA\_SET)**

**{**

**WriteDbgLog(2, "EVENT\_CONFIG\_UPDATE: NET\_PARA\_SET jtgps\n");**

**UsrDefineCmd\_t m\_Cmd;**

**m\_Cmd.Type = EVENT4\_DATA\_SYNC;**

**m\_Cmd.DataLen = 0;**

**DistributeEvent(EVENT\_USR\_DEF4, sizeof(UsrDefineCmd\_t), (void \*)&m\_Cmd);**

**sleep(2);**

**exit(0);**

**}**

**else if(pConfigCmd-\>Type == SPEED\_PARA\_SET)**

**{**

**#if !defined(JT\_PROTOCOL)**

**WriteDbgLog(2, "EVENT\_CONFIG\_UPDATE: SPEED\_PARA\_SET jtgps\n");**

**UsrDefineCmd\_t m\_Cmd;**

**m\_Cmd.Type = EVENT4\_DATA\_SYNC;**

**m\_Cmd.DataLen = 0;**

**DistributeEvent(EVENT\_USR\_DEF4, sizeof(UsrDefineCmd\_t), (void \*)&m\_Cmd);**

**sleep(2);**

**exit(0);**

**#else**

**SpeedSet\_t speedSet = {0};**

**CConfig \*pConfig = CConfig::Instance();**

**pConfig-\>GetConfig(&speedSet);**

**pthis-\>EventHandler(EVENT\_JT\_SPEED\_MODE, 0, 0, (char\*)&speedSet.source, sizeof(speedSet.source));**

**#endif**

**}**

**}**

**break;**

**case EVENT\_JT808\_EVENT:**

**{**

**Jt808Event\_t\* pJt808Event = (Jt808Event\_t\*)Tmpbuf;**

**switch(pJt808Event-\>Type)**

**{**

**case EVENT\_REPORT:**

**{**

**EventReportCmd\_t\* pEventReportCmd = (EventReportCmd\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_REPORT, 0, 0, (char\*)pEventReportCmd, sizeof(EventReportCmd\_t));**

**}**

**break;**

**case EVENT\_QUESTION\_RES:**

**{**

**EventIssuedCmd\_t\* pEventIssuedCmd = (EventIssuedCmd\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_QUESTION\_RES, 0, 0, (char\*)pEventIssuedCmd, sizeof(EventIssuedCmd\_t));**

**}**

**break;**

**case EVENT\_MENU\_RES:**

**{**

**EventMenuCmd\_t\* pEventMenuCmd = (EventMenuCmd\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_MENU\_RES, 0, 0, (char\*)pEventMenuCmd, sizeof(EventMenuCmd\_t));**

**}**

**break;**

**case EVENT\_PASSENGER\_TRAFFIC:**

**{**

**PassengerTraffic\_t\* pPassengerTraffic = (PassengerTraffic\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_PASSENGER\_TRAFFIC,0,0, (char\*)pPassengerTraffic, sizeof(PassengerTraffic\_t));**

**}**

**break;**

**case EVENT\_MEDIA\_DATA:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MEDIA\_DATA, 0, 0, 0, 0);**

**}**

**break;**

**case EVENT\_ELEC\_WAYBILL:**

**{**

**EventElecWaybill\_t\* pEventElecWaybill = (EventElecWaybill\_t\*)pJt808Event-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_ELEC\_WAYBILL,0,0, (char\*)pEventElecWaybill, strlen(pJt808Event-\>data));**

**}**

**break;**

**case EVENT\_MANUALLY\_REG:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MANUALLY\_REG,0,0, 0, 0);**

**}**

**break;**

**case EVENT\_MANUALLY\_AUTH:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MANUALLY\_AUTH,0,0, 0, 0);**

**}**

**break;**

**case EVENT\_MANUALLY\_LOGOUT:**

**{**

**pthis-\>EventHandler(EVENT\_JT\_MANUALLY\_LOGOUT,0,0, 0, 0);**

**}**

**break;**

**default:**

**break;**

**}**

**}**

**break;**

**case EVENT\_SERIAL\_CONTROL:**

**{**

**SerialCmd\_t \*pSerialCmd = (SerialCmd\_t\*)Tmpbuf;**

**switch(pSerialCmd-\>FunctionDef)**

**{**

**case COM\_TYPE\_EXPORT\_RECORD\_LOG:**

**/\*if(pSerialCmd-\>Cmd == 1)**

**{**

**pthis-\>EventHandler(EVENT\_JT\_19056\_COLLECT, 0, 0, (char\*)pSerialCmd-\>DataBuf, pSerialCmd-\>Datalen);**

**}\*/**

**break;**

**}**

**}**

**break;**

**case EVENT\_CAN\_CONTROL:**

**{**

**CanCmd\_t \*pCanCmd = (CanCmd\_t\*)Tmpbuf;**

**if(pCanCmd-\>Cmd == 1)**

**{**

**pthis-\>EventHandler(EVENT\_CANBUS, pCanCmd-\>ID, 0,(char\*)pCanCmd-\>DataBuf, pCanCmd-\>Datalen);**

**}**

**}**

**break;**

**case EVENT\_USR\_DEF2:**

**{**

**UsrDefineCmd\_t \*pUsrDefineCmd = (UsrDefineCmd\_t\*)Tmpbuf;**

**if(pUsrDefineCmd-\>Type == EVENT2\_FT\_TO\_808)**

**{**

**pthis-\>EventHandler(EVENT\_FTTRANSPT, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT2\_MCUSET\_JT808)**

**{**

**pthis-\>EventHandler(MCU\_SET\_808\_PARAMETER, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**else if(pUsrDefineCmd-\>Type == EVENT2\_CLEAN\_AUTH)//HC 20190415**  **清除鉴权**

**{**

**pthis-\>EventHandler(STATE\_AUTH\_CLEAN, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}else if(pUsrDefineCmd-\>Type == EVENT2\_ADC\_VALUE)**

**{**

**pthis-\>EventHandler(EVENT2\_ADC\_VALUE, 0, 0, pUsrDefineCmd-\>Data, pUsrDefineCmd-\>DataLen);**

**}**

**}**

**break;**

**case EVENT\_PULSE\_SPEED:**

**{**

**EventPulseSpeed\_t \*pEventPulseSpeed = (EventPulseSpeed\_t\*)Tmpbuf;**

**pthis-\>EventHandler(EVENT\_JT\_PULSE\_SPEED, 0, 0, (char\*)pEventPulseSpeed, sizeof(EventPulseSpeed\_t));**

**}**

**break;**

**case EVENT\_USR\_DEF4:**

**{**

**UsrDefineCmd\_t \*pUsrDefine = (UsrDefineCmd\_t\*)Tmpbuf;**

**if(pUsrDefine-\>Type==EVENT4\_FillPOS)**

**{**

**pthis-\>EventHandler(EVENT4\_FillPOS, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**else if(pUsrDefine-\>Type==EVENT4\_DATA\_SYNC)**

**{**

**pthis-\>EventHandler(EVENT4\_DATA\_SYNC, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**else if(pUsrDefine-\>Type==EVENT4\_IC\_CARD\_CHANGE)**

**{**

**pthis-\>EventHandler(EVENT4\_IC\_CARD\_CHANGE, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**else if(pUsrDefine-\>Type==EVENT4\_YL\_REPORT)**

**{**

**pthis-\>EventHandler(EVENT4\_YL\_REPORT, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}else if(pUsrDefine-\>Type==EVENT4\_ADAS\_DMS)**

**{**

**pthis-\>EventHandler(EVENT\_JT\_DS\_AI, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**#if defined(FSLX400JT)**

**else if(pUsrDefine-\>Type==EVENT4\_FSLX\_MCU\_808\_CMD)**

**{**

**pthis-\>EventHandler(EVENT\_FSLX\_MCU\_808\_CMD, 0, 0, pUsrDefine-\>Data, pUsrDefine-\>DataLen);**

**}**

**#endif**

**break;**

**}**

**case EVENT\_GB19056\_EVENT:**

**{**

**GB19056Event\_t \*tmpGB19056 = (GB19056Event\_t\*)Tmpbuf;**

**switch(tmpGB19056-\>Type)**

**{**

**/\*case EVENT\_GB19056\_LINE\_FATI\_ALARM:**

**{**

**LineFatiAlm\_t \*tmpFatigue=(LineFatiAlm\_t \*)tmpGB19056-\>data;**

**pthis-\>RegionLineAlm(tmpFatigue-\>AlarmType, tmpFatigue-\>AlarmStatus,(void \*)tmpFatigue-\>data, args);**

**break;**

**}\*/**

**case EVENT\_GB19056\_POWER\_STATE:**

**{**

**EventPowerState\_t\* pEventPowerState = (EventPowerState\_t\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_19056\_POWER\_STATE,0,0, (char\*)pEventPowerState, sizeof(EventPowerState\_t));**

**break;**

**}**

**case EVENT\_GB19056\_COLLECT\_RES\_808:**

**{**

**if(tmpGB19056-\>workmode == 0) //jtgps****暂定中心 ****0**

**pthis-\>EventHandler(EVENT\_JT\_GB19056\_COLLECT\_RES,0,0, (char\*)tmpGB19056-\>data, sizeof(tmpGB19056-\>data));**

**break;**

**}**

**case EVENT\_GB19056\_IC\_CARD:**

**{**

**IcCard\_t\* pIcCard = (IcCard\_t\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_IC\_CARD\_INFOR,0,0, (char\*)pIcCard, sizeof(IcCard\_t));**

**break;**

**}**

**case EVENT\_GB19056\_IC\_CARD\_JIUTONG:**

**{**

**IcCard\_t\_jiutong\* pIcCard = (IcCard\_t\_jiutong\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_IC\_CARD\_INFOR\_JIUTONG, 0, 0, (char\*)pIcCard, sizeof(IcCard\_t\_jiutong));**

**break;**

**}**

**case EVENT\_GB19056\_SWT:**

**{**

**EventSwt\_t\* pEventSwt = (EventSwt\_t\*)tmpGB19056-\>data;**

**pthis-\>EventHandler(EVENT\_JT\_19056\_SWT,0,0, (char\*)pEventSwt, sizeof(EventSwt\_t));**

**break;**

**}**

**}**

**}**

**break;**

**case EVENT\_MISC:**

**{**

**MiscCmd\_t\* pMiscCmd = (MiscCmd\_t\*)Tmpbuf;**

**if(netMode == PROTOCOL\_808 || netMode == PROTOCOL\_XIANG\_M1)**

**{**

**if(pMiscCmd-\>Type == MISC\_EVENT\_FILE\_UPLOAD)**

**{**

**AlarmCmd\_t\* pAlarmCmd = (AlarmCmd\_t\*)pMiscCmd-\>Data;**

**if(pAlarmCmd-\>subType == 1)//**  **图片**

**pthis-\>EventHandler(ALARM\_PIC\_AUTO\_UPLOAD, pAlarmCmd-\>subType, pAlarmCmd-\>enable, pMiscCmd-\>Data+ sizeof(AlarmCmd\_t), pAlarmCmd-\>alarmArgsLength);**

**}**

**}**

**}**

**break;**

**default:**

**break;**

**}**

**return 0;**

**}**

# Appendix 3 Add a network thread to call SDK interface reference

# Appendix 4 SDK demo
