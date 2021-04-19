# IM SDK for Unity3D 开发指引

## 概述
游密即时通讯SDK（IM SDK）为玩家提供完整的游戏内互动服务，游戏开发者无需关注IM通讯复杂的内部工作流程，只需调用IM SDK提供的接口，即可快速实现世界聊天、公会聊天、组队聊天、文字、表情、语音等多项功能。

## IM SDK接口调用顺序

![IM SDK接口调用顺序](https://youme.im/doc/images/im_sdk_interface_call_order.png)


## 四步集成IM SDK
### 第一步 注册账号
在[游密官网](https://console.youme.im/user/register)注册游密账号。

### 第二步 添加游戏，获取Appkey
在控制台添加游戏，获得接入需要的Appkey、Appsecret。

### 第三步 下载IM SDK包体
[下载地址](https://www.youme.im/download.php?type=IM)

### 第四步 开发环境配置
[开发环境配置](#快速接入)

## 快速接入
### 1. 导入IM SDK
#### 导入IM插件包

- 右键点击`Assets`->`Import Package`->`Custom Package…`->选中`im_unity_xxx.unitypackage`。
![添加SDK](https://www.youme.im/doc/images/im_add_sdk.jpg)
- 在弹出的`Import Unity Package`对话框中，勾选所有的复选框。
- 为应用选择需要支持的CPU架构，以Android平台armeabi架构为例：
    - 在`Project View`中`Assets/Plugins/Android/libs`目录下选择特定的CPU架构；
    - 在`Inspector View`中`Select platforms for plugin`勾选`Android`。

#### 导出iOS工程
根据需要选择导出iOS或者Android工程。

- 在`File`->`Build Settings…`的Platform列表中选择iOS；
- 点击`Build Settings`->`Build`，选择输出iOS工程的路径，输入工程名字，导出iOS工程；
- 在Xcode中打开上一步输出的iOS工程，在工程配置中`Build Phases`->`Link Binary With Libraries`下拉菜单中添加下面几个库文件：
 >`libc++.1.tbd`  
 >`libsqlite3.0.tbd`  
 >`libresolv.9.tbd`  
 >`SystemConfiguration.framework`  
 >`libYouMeCommon.a`  
 >`libyim.a`  
 >`libz.tbd`  
 >`CoreTelephony.framework`  
 >`AVFoundation.framework`  
 >`AudioToolbox.framework`   
 >`CoreLocation.framework`   
 >`iflyMSC.framework`       //如果是带语音转文字的sdk需要添加.  
 >`AliyunNlsSdk.framework`  //如果是带语音转文字的sdk需要添加，动态库，需要在`General -> Embedded Binaries`也进行添加.  
 >`USCModule.framework`     //如果是带语音转文字的sdk需要添加.  
 >`libBoringSSL.a`      //如果是带google语音转文字的sdk需要
 >`libgoogleapis.a`//如果是带google语音转文字的sdk需要
 >`libgRPC-Core.a`//如果是带google语音转文字的sdk需要
 >`libgRPC-ProtoRPC.a`//如果是带google语音转文字的sdk需要
 >`libgRPC-RxLibrary.a`//如果是带google语音转文字的sdk需要
 >`libgRPC.a`//如果是带google语音转文字的sdk需要
 >`libnanopb.a`//如果是带google语音转文字的sdk需要
 >`libProtobuf.a`//如果是带google语音转文字的sdk需要
   
- 如果接入带google语音识别的版本，需在`Build Phases  ->Copy Bundle Resources` 里加入 `gRPCCertificates.bundle` 

- **为iOS10添加录音权限**
iOS 10 使用录音权限，需要在`info`新加`Privacy - Microphone Usage Description`键，值为字符串，比如“语音聊天需要录音权限”。
首次录音时会向用户申请权限。配置方式如图(选择Private-Microphone Usage Description)。
![iOS10录音权限配置](https://www.youme.im/doc/images/im_iOS_record_config.jpg)

- **为iOS10添加地理位置权限【可选】**
若使用LBS，需要在`info`新加`Privacy - Location Usage Description`键，值为字符串，比如“查看附近的玩家需要获取地理位置权限”。首次使用定位时会向用户申请权限，配置方式如上图录音权限。

#### 导出Android包
根据需要选择导出iOS或者Android工程。
在 AndroidManifest.xml 添加权限和网络变化监听配置，完成欲用功能编写后直接打包apk即可：

``` xml
  <!-- 添加到application节点内 -->
  <application xxxx>
      <receiver android:name="com.youme.im.NetworkStatusReceiver" android:label="NetworkConnection" >
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
      </receiver>
  </application>
  <!-- 添加到跟application平级 -->
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
  <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.READ_PHONE_STATE" />
  <uses-permission android:name="android.permission.RECORD_AUDIO" />
   <!-- 获取地理位置的权限，可选 -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

#### 混淆配置
- 如果打包或生成APK时需要混淆，则需要在proguard.cfg文件中添加如下代码：

```

    -keep class com.youme.** {*;}
    -keep class com.iflytek.**
    -keepattributes Signature  
```

### 2. 初始化
成功导入SDK后，应用启动的第一时间需要调用初始化接口。

#### 获取 IMAPI
- 引入IM SDK命名空间：

  ``` csharp
  using YIMEngine;
  ```

  所有API操作接口都在`IMAPI`对象里。这是一个单例类，可以通过`Instance ()`获取到它的实例。通过它就可以调用IM SDK的API接口。

#### 初始化IM SDK
获得`IMAPI`之后，需要调用初始化接口初始化IM SDK。

- **调用示例：**

  ``` csharp      
      Errorcode err = IMAPI.Instance().Init("strAppKey","strSecrect",ServerZone.China);
  ```
  
- **相关接口与参数：**
  - `初始化接口`：
    `public ErrorCode Init(string strAppKey,string strSecrect, ServerZone serverZone)`
    
  - `strAppKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `strSecrect`：用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `serverZone`：服务器区域参数，设置值查看[服务器部署地区定义](IMSDKAndroid.php#服务器部署地区定义)
  
- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **备注：**
  此接口本是异步操作，未给出异步回调，一般返回的错误码是Success即表示初始化成功。
  
### 3. 设置回调监听  
IM引擎底层对于耗时操作都采用异步回调的方式，函数调用会立即返回，操作结果C#层会同步回调。因此，用户必须实现相关接口并在初始化完成以后进行注册。常用功能需要注册登录回调，聊天室回调，消息回调，如需其它功能查看API接口。

- **调用示例：**

  ``` csharp      
      // 设置监听可在调初始化之前执行
  public class YIMVoiceTest : MonoBehaviour,
    LoginListen,
    MessageListen,
    ChatRoomListen,
    DownloadListen    
  {
    ...
    void Start()
    {
        //注册回调
        IMAPI.Instance().SetLoginListen(this);
        IMAPI.Instance().SetMessageListen(this);
        IMAPI.Instance().SetChatRoomListen(this);
        IMAPI.Instance().SetDownloadListen(this);
        
        //初始化IM
        Errorcode err = IMAPI.Instance().Init("strAppKey","strSecrect",ServerZone.China);
    }    
    
    ...
    public void OnLogin(ErrorCode errorcode, string strYouMeID)
    {
        Debug.Log("OnLogin: errorcode " + errorcode + " userID:" + strYouMeID);
        if(errorcode == ErrorCode.Success){
            Debug.Log("登录成功");
            if(needJoinRoom){
                needJoinRoom=false;
                //可在登录成功的回调里面加入频道
                IMAPI.Instance().JoinChatRoom("12345");
            }
        }
    }
    ....
    //其余回调接口实现
  }
  ```
  
- **相关接口：**
	- `登录回调注册接口`：
	  `public void SetLoginListen(YIMEngine.LoginListen listen)`
	  
	- `聊天室回调注册接口`：
	  `public void SetChatRoomListen(YIMEngine.ChatRoomListen listen)`
	  
	- `消息回调注册接口`：
	  `public void SetMessageListen(YIMEngine.MessageListen listen)`
	  
### 4. 登录IM系统
![登录界面介绍](https://www.youme.im/doc/images/im_login.png)

成功初始化IM后，需要使用IM功能时，先调用IM SDK登录接口。

- **调用示例：**

  ``` csharp
      ErrorCode errorcode = IMAPI.Instance().Login("1001", "123456");
  ```
  
- **相关接口与参数：**
	- `登录接口`：
	  `public ErrorCode Login(string strYouMeID,string strPasswd,string strToken="")`
	  
	- `strYouMeID`：由调⽤者分配，不可为空字符串，只可由字母或数字或下划线组成，用户首次登录会自动注册所用的用户ID和密码
	- `strPasswd`：⽤户密码，不可为空字符串，由调用者分配，二次登录时需与首次登录一致，否则会报UsernamePasswordError
	- `strToken`：用户验证token，可选，如不使用token验证传入:""
	
- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **回调接口与参数：**
   登录接口是异步操作，登录成功的标识是`OnLogin`回调的错误码是Success；调用`Login`同步返回值是Success才能收到`OnLogin`回调；特别的如果调用`Login`多次，收到AlreadyLogin错误码也表示登录成功。 
   
   - `登录回调接口`：
     `void OnLogin (YIMEngine.ErrorCode errorcode, string strYouMeID)`
     
   - `strYouMeID`：用户ID
   - `errorcode`：错误码
   
### 5. 进入聊天频道
![频道](https://www.youme.im/doc/images/im_room.png)

登录成功后，如果有群组聊天需要，比如游戏里面的世界、工会、区域等，需要进入聊天频道，调用方为聊天频道设置一个唯一的频道ID，可以进入多个频道。

- **调用示例：**

  ``` csharp
      // 可在登录回调中执行
      IMAPI.Instance().JoinChatRoom("1001");
  ```
  
- **相关接口与参数**
	- `进入频道接口`：
	  `public ErrorCode JoinChatRoom (string strChatRoomID)`
	  
	- `strChatRoomID`：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)

- **回调接口与参数：**
  进入频道接口是异步操作，进入频道成功的标识是`OnJoinRoom`回调的错误码是Success；调用`JoinChatRoom`同步返回值是Success才能收到`OnJoinRoom`回调。
  
   - `进入频道回调接口`：
     `void OnJoinRoom(YIMEngine.ErrorCode errorcode,string strChatRoomID)`
     
   - `errorcode`：错误码
   - `strChatRoomID`：频道ID
   
### 6. 发送文本消息
![频道](https://www.youme.im/doc/images/im_room_send_btn.png)

在成功登录IM后，即可发送IM消息。

- **调用示例：**

  ``` csharp
      ulong iRequestID = 0;
      // 发送私聊文本消息
      ErrorCode errorcode = IMAPI.Instance().SendTextMessage("1001", ChatType.PrivateChat, "测试发送文本", "attachMsg", ref iRequestID);
      
      // 发送频道文本消息，需成功进入频道后发消息，频道内的其它成员才能接收到
      ErrorCode errorcode = IMAPI.Instance().SendTextMessage("12345", ChatType.RoomChat, "测试发送文本", "attachMsg", ref iRequestID);
  ```
  
- **相关接口与参数：**
	- `发文本消息接口`：
     `public ErrorCode SendTextMessage(string strRecvID, YIMEngine.ChatType chatType,string strContent, string strAttachParam, ref ulong iRequestID)`
     
	- `strRecvID`：接收者ID,（⽤户ID或者频道ID）
	- `chatType`：聊天类型,私聊传`1`，频道聊天传`2`；需要频道聊天得先成功进入频道后发文本消息，此频道内的成员才能接收到消息
	- `strContent`：聊天内容
	- `strAttachParam`：发送文本附加信息
	- `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **回调接口与参数：**
  发送文本消息接口是异步操作，发送文本消息成功的标识是`OnSendMessageStatus`回调的错误码是Success；调用`SendTextMessage`同步返回值是Success才能收到`OnSendMessageStatus`回调；文本消息发送成功，接收方会收到`OnRecvMessage`回调，能从该回调中取得消息内容，消息发送者等信息。
  
   - `发送文本消息回调接口`：
     `void OnSendMessageStatus(ulong iRequestID,  YIMEngine.ErrorCode errorcode, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime)`
     
   - `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
   - `errorcode`：错误码
   - `sendTime`：消息发送时间
   - `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言
   - `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
   - `forbidEndTime`：若在频道被禁言，禁言结束时间   

- **拓展功能：**
  发送消息时，可以将玩家头像、昵称、角色等级、vip等级等要素打包成json格式发送。

### 7. 发送语音消息
![发送语音消息](https://www.youme.im/doc/images/im_send_voice_message.jpg)

在成功登录IM后，可以发送语音消息
>- 按住语音按钮时，调用`SendAudioMessage`启动录音
>- 松开按钮，调用`StopAudioMessage`接口，发送语音消息
>- 按住过程若需要取消发送，调用`CancleAudioMessage`取消本次语音发送

- **调用示例：**

  ``` csharp
      ulong iRequestID = 0;
      // 开始语音
      ErrorCode errorcode = IMAPI.Instance().SendAudioMessage("1001", ChatType.PrivateChat, ref iRequestID);
      
      // 结束并发送语音
      ErrorCode errorcode = IMAPI.Instance().StopAudioMessage("");
      
      // 如果不想发送语音，调取消本次语音，调取消后收不到消息的发送回调
      ErrorCode errorcode = IMAPI.Instance().CancleAudioMessage();
  ```
  
- **相关接口与参数：**
	- `发送语音消息接口`：
	  `public ErrorCode SendAudioMessage(string strRecvID,YIMEngine.ChatType chatType,ref ulong iRequestID)`
	  
	  若不需要文字识别的语音，使用下面接口。
	  `public ErrorCode SendOnlyAudioMessage(string strRecvID,YIMEngine.ChatType chatType,ref ulong iRequestID)`
	  
	- `结束录音接口`：
	  `public ErrorCode StopAudioMessage(string strParam)`
	  
	- `取消录音接口`：
	  `public ErrorCode CancleAudioMessage()`
	  
	- `strRecvID`：接收者ID（⽤户ID或者频道ID）
	- `chatType`：消息类型，表示私聊还是频道消息
	- `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
	- `strParam`：发送语音消息附加信息，主要用于调用方特别需求

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **回调接口与参数：**
   调用`StopAudioMessage`接口后，会收到发送语音的回调，如果录音成功会收到开始发送语音的回调`OnStartSendAudioMessage`，无论录音是否成功都会收到发送语音消息结果的回调`OnSendAudioMessageStatus`，`OnStartSendAudioMessage`在接收时间上比`OnSendAudioMessageStatus`快，常用于上屏显示。
   
   - `发送语音消息回调接口`：
     `void OnStartSendAudioMessage(ulong iRequestID, YIMEngine.ErrorCode errorcode, string strText, string strAudioPath, int iDuration)`
     
     `void OnSendAudioMessageStatus(ulong iRequestID,YIMEngine.ErrorCode errorcode,string strText,string strAudioPath,int iDuration, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime)`
     
   - `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
   - `errorcode`：错误码
   - `strText`：语音转文字识别的文本内容，如果没有用带语音转文字的接口，该字段为空字符串
   - `strAudioPath`：录音生成的wav文件的本地完整路径
   - `iDuration`：录音时长(单位秒)
   
   - `sendTime`：消息发送时间
   - `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言
   - `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
   - `forbidEndTime`：若在频道被禁言，禁言结束时间
   
- **备注：**
  此套接口是异步操作，发送语音消息成功的标识是收到`OnStartSendAudioMessage`，或者`OnSendAudioMessageStatus`回调的错误码是Success；调用`StopSendAudioMessage`同步返回值是Success才能收到回调；语音消息发送成功，接收方会收到`OnRecvMessage`回调，能从该回调中下载语音文件。

### 8. 接收消息
![接收消息](https://www.youme.im/doc/images/im_receive_message.jpg)
>- 通过`OnRecvMessage`接口被动接收消息，需要开发者实现

- **调用示例：**

  ``` csharp
  public void OnRecvMessage(MessageInfoBase message)
  {
        if (message.MessageType == MessageBodyType.TXT)
        {
            TextMessage textMsg = (TextMessage)message;
            Debug.Log("OnRecvMessage text:" + textMsg.Content + " send:" + textMsg.SenderID + "recv:" + textMsg.RecvID);
        }        
        else if (message.MessageType == MessageBodyType.Voice)
        {
            VoiceMessage voiceMsg = (VoiceMessage)message;
            Debug.Log("OnRecvMessage voice 文本识别结果:" + voiceMsg.Text + " send:" + voiceMsg.SenderID + "recv:" + voiceMsg.RecvID +" 时长:"+voiceMsg.Duration);

            //下载收到的语音消息
            IMAPI.Instance().DownloadAudioFile(voiceMsg.RequestID, "savePath");
        }
    }
  ```   

### 9. 接收语音消息并播放
![接收语音消息](https://www.youme.im/doc/images/im_receive_message-2.jpg)
>- 通过`MessageType`分拣出语音消息`（MessageBodyType.Voice）`
>- ⽤`RequestID`获得消息ID
>- 点击语音气泡，调用函数`DownloadAudioFile`下载语音消息
>- 调用方调用函数`StartPlayAudio`播放语音消息

- **调用示例：**

  ``` csharp
      ErrorCode errorcode = IMAPI.Instance().DownloadAudioFile(voiceMsg.RequestID, "savePath");
      
      ...
      
      public void OnDownload( YIMEngine.ErrorCode errorcode, YIMEngine.MessageInfoBase message, string strSavePath)
	   {
		   //如果下载收到的语音消息成功，播放语音消息
		   if((errorcode == ErrorCode.Success) && (message.MessageType == MessageBodyType.Voice)){
			   IMAPI.Instance().StartPlayAudio(strSavePath);
		   }
	   }
  ```
  
- **相关接口和参数：**
  - `下载语音消息接口`：
    `public ErrorCode DownloadAudioFile(ulong iRequestID, string strSavePath)`
    
  - `播放语音消息接口`：
    `public ErrorCode StartPlayAudio(string path)`  
    
	- `iRequestID`：消息ID
	- `strSavePath`：语音文件保存路径
	- `path`：待播放文件路径

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)

- **回调接口与参数：**
  下载语音接口是异步操作，下载语音成功的标识是`OnDownload`回调的错误码为Success，调用`DownloadAudioFile`接口同步返回值是Success才能收到`OnDownload`回调。成功下载语音消息后即可播放语音消息。
  
  - `下载回调接口`：
    `void OnDownload( YIMEngine.ErrorCode errorcode, YIMEngine.MessageInfoBase message, string strSavePath)`	
    
  - `errorcode`：错误码
  - `message`：消息基类
  - `strSavePath`：保存路径

  - `播放完成回调接口`：
    `void OnPlayCompletion(YIMEngine.ErrorCode errorcode, string path)` 
    
  - `errorcode`：错误码
  - `path`：文件路径

### 10. 登出IM系统
注销账号时，调用登出接口`Logout`登出IM系统。

- **调用示例：**

  ``` csharp
      ErrorCode errorcode = IMAPI.Instance().Logout();
  ```
  
- **相关接口：**
	- `登出接口`：
	  `public ErrorCode Logout()`  
	  
- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义) 
	  
- **回调接口：**
  登出是异步操作，登出成功的标识是`OnLogout`回调的错误码为Success，调用`Logout`接口同步返回值是Success才能收到`OnLogout`回调。
  
  - `登出回调接口`：
    `void OnLogout()`

## 典型场景集成方案
主要分为世界频道聊天，用户私聊，直播聊天室；集成的时都需要先初始化SDK，登录IM系统

***流程图

### 启动应用，初始化IM SDK
应用启动的第一时间需要调用初始化接口。

- **接口与参数：**
  - `Init`：初始化接口
  - `strAppKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `strSecrect`：用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看
  - `serverZone`：服务器区域参数，可选参数，默认是中国，其余设置值查看[服务器部署地区定义](IMSDKAndroid.php#服务器部署地区定义)

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	  
- **代码示例与详细说明：**
	- [Unity3D示例](#初始化IM SDK)

### 登录应用时，登录IM系统
![登录界面介绍](https://www.youme.im/doc/images/im_login.png)
>- 点击`登录`按钮时，调用IM SDK登录接口。

- **接口与参数：**
	- `Login`：登录接口
	- `strYouMeID`：由调⽤者分配，不可为空字符串，只可由字母或数字或下划线组成，用户首次登录会自动注册所用的用户ID和密码
	- `strPasswd`：⽤户密码，不可为空字符串，由调用者分配，二次登录时需与首次登录一致，否则会报UsernamePasswordError
	- `strToken`：用户验证token，可选，如不使用token验证传入:""
	
- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)

- **代码示例与详细说明：**
	- [Unity3D示例](#4. 登录IM系统)

### 典型场景
#### 世界频道聊天
![频道](https://www.youme.im/doc/images/im_room.png)
>- 进入应用后，调用加入频道接口，进入世界、公会、区域等需要进入的聊天频道。
>- 应用需要为各个聊天频道设置一个唯一的频道ID。
>- 成功进入频道后，发送频道消息，文本、语音消息等。

- **接口与参数**
	- `JoinChatRoom`：加入频道
	- `strChatRoomID`：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **代码示例与详细说明：**
	- [Unity3D示例](#5. 进入聊天频道)

#### 用户私聊
>- 成功登录IM系统后，可和其它登录IM的用户发私聊消息，文本、语音消息等；发文本消息接口的聊天类型参数使用 `1`-私聊类型
>- 调用流程查看[发送文本消息](#发送文本消息)，[发送语音消息](#发送语音消息)

#### 直播聊天室
>- 用户集成直播SDK后，导入IM SDK
>- 初始化IM SDK
>- 登录IM 系统
>- 进入指定的聊天频道
>- 发送频道消息（例如：弹幕式），调用流程查看[发送文本消息](#发送文本消息)

### 发送文本消息
![频道](https://www.youme.im/doc/images/im_room_send_btn.png)
>- 点击发送按钮，调用发消息接口，将输入框中的内容发送出去
>- 发送出的消息出现在聊天框右侧
>- 表情消息可以将表情信息打包成Json格式发送

- **相关接口与参数：**
	- `SendTextMessage`：发文字消息接口
	- `strRecvID`：接收者ID,（⽤户ID或者频道ID）
	- `chatType`：聊天类型,私聊传`1`，频道聊天传`2`；需要频道聊天得先成功进入频道后发文本消息，此频道内的成员才能接收到消息
	- `strContent`：聊天内容
	- `strAttachParam`：发送文本附加信息
	- `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)

- **代码示例与详细说明：**
	- [Unity3D示例](#6. 发送文本消息)

- **拓展功能：**
发送消息时，可以将玩家头像、昵称、角色等级、vip等级等要素打包成json格式发送。

### 发送语音消息
![发送语音消息](https://www.youme.im/doc/images/im_send_voice_message.jpg)
>- 按住语音按钮时，调用`SendAudioMessage`
>- 松开按钮，调用`StopAudioMessage`接口，发送语音消息
>- 按住过程若需要取消发送，调用`CancleAudioMessage`取消发送

- **相关接口与参数：**
	- `SendAudioMessage`：发送语音消息接口
	- `StopAudioMessage`：结束录音接口
	- `CancleAudioMessage`：取消录音接口
	
	- `strRecvID`：接收者ID（⽤户ID或者群ID）
	- `chatType`：消息类型，表示私聊还是频道消息
	- `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
	- `strParam`：发送语音消息附加信息，主要用于调用方特别需求

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)

- **代码示例与详细说明：**
	- [Unity3D示例](#7. 发送语音消息)

### 接收消息
![接收消息](https://www.youme.im/doc/images/im_receive_message.jpg)
>- 通过`OnRecvMessage`接口被动接收消息，需要开发者实现，接收消息进行相应的展示，如果是语音消息需要下载语音，以及下载完成后的语音播放

- **相关接口与参数：**
	- `OnRecvMessage`：收消息接口

- **代码示例与详细说明：**
	- [Unity3D示例](#8. 接收消息)

### 接收语音消息并播放
![接收语音消息](https://www.youme.im/doc/images/im_receive_message-2.jpg)
>- 通过`MessageType`分拣出语音消息`（MessageBodyType.Voice）`
>- ⽤`RequestID`获得消息ID
>- 点击语音气泡，调用`DownloadAudioFile`接口下载语音文件
>- 调用方播放语音文件

- **相关接口与参数：**
  - `DownloadAudioFile`：下载语音文件接口
  - `StartPlayAudio`：播放语音文件接口
   
	- `iRequestID`：消息ID
	- `strSavePath`：语音文件保存路径
	- `path`：待播放文件路径

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **代码示例与详细说明：**
	- [Unity3D示例](#9. 接收语音消息并播放)

### 登出IM系统
![更换账号](https://www.youme.im/doc/images/im_change_account.jpg)
>- 如下两种情况需要登出IM系统：
>- 注销账号时，调用`Logout`接口登出IM系统
>- 退出应用时，调用`Logout`接口登出IM系统

- **相关接口：**
	- `Logout`：登出接口

- **返回值：**
	- `ErrorCode`：错误码，详细描述见[错误码定义](IMSDKUnityC.php#错误码定义)
	
- **代码示例与详细说明：**
	- [Unity3D示例](#10. 登出IM系统)

