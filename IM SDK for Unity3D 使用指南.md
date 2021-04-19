# IM SDK for Unity3D 使用指南

## 导入SDK
### 导入IM插件包
- 右键点击`Assets`->`Import Package`->`Custom Package…`->选中`im_unity_xxx.unitypackage`。
![添加SDK](https://www.youme.im/doc/images/im_add_sdk.jpg)
- 在弹出的`Import Unity Package`对话框中，勾选所有的复选框。
- 为应用选择需要支持的CPU架构，以Android平台armeabi架构为例：
    - 在`Project View`中`Assets/Plugins/Android/libs`目录下选择特定的CPU架构；
    - 在`Inspector View`中`Select platforms for plugin`勾选`Android`。

### 导出iOS工程

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
 
 >`iflyMSC.framework`       //如果是带语音转文字的sdk需要添加。  
 
 >`AliyunNlsSdk.framework`  //如果是带语音转文字的sdk需要添加，该库是动态库，需要在`General -> Embedded Binaries`也进行添加。  
 
 >`USCModule.framework`     //如果是带语音转文字的sdk需要添加。  
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

### 导出Android包
在 AndroidManifest.xml 添加权限和网络变化监听配置，然后直接打包apk即可：

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

## 特别注意事项
* 重连有可能失败，如果设置了重连监听，开始重连时能收到OnStartReconnect通知，重连结果由OnRecvReconnectResult通知给出，如果OnRecvReconnectResult通知给出的结果是重连失败，随即也会收到OnLogout的通知，此时需要判断游戏是否还在线，如果游戏还在线，就重新调用 登陆->进频道 接口
* 初始化接口整个运行期间，只能调用一次
* 不同设备或者不同的运行期，消息id可能重复
* 返回值为ErrorCode的接口，如果返回了非ErrorCode.Success表示接口调用失败，就不会再有回调通知（如果是有回调的接口）
* 请务必使用游密SDK提供的`android-support-v4.jar` ，这样可以兼容`targetSdkVersion="23"`及其以上版本的录音授权模式

## 基本接口使用时序图

![基本接口使用时序图](https://youme.im/doc/images/im_sdk_flow_client_csharp.png)

## 初始化

### 处理方法介绍
Unity3D 版本的 IM SDK 提供的 API 全部为 csharp 接口，接口调用都不会阻塞主线程，凡是本身需要较长耗时的接口调用都会采用异步回调，所以全部接口都可以在主线程中直接使用。调用方需要实现监听类并添加到监听队列里，操作结果通过回调的方式通知到调用方。

### SDK对象
>`IMAPI`：SDK管理器，用来进行初始化、登录、登出等操作
>`IMDefine`：SDK声明，SDK的结构体，错误码等声明
>`ChatRoomListen`：群管理监听接口，负责处理聊天室管理操作结果
>`LoginListen`：登录登出监听器接口，负责处理登录、登出结果
>`MessageListen`：消息监听器接口，负责处理消息收发操作结果
>`DownloadListen`：下载监听接口，负责处理下载完成操作
>`ContactListen`：获取最近私聊联系人结果监听接口
>`AudioPlayListen`：监听音频播放结束通知
>`LocationListen`：地理位置相关功能监听接口
>`NoticeListen`：公告相关功能监听接口
>`ReconnectListen`：重连功能监听接口
>`UserProfileListen`：用户信息管理监听接口
>`FriendListen`：好友管理监听接口

### 获取 IMAPI 

- 引入IM SDK命名空间：

  ``` csharp
  using YIMEngine;
  ```

  所有API操作接口都在`IMAPI`对象里。这是一个单例类，可以通过`Instance ()`获取到它的实例。通过它就可以调用IM SDK的API接口。

- 原型：

  ``` csharp
  public static IMAPI Instance();
  ```

- 返回：
  `IMAPI`实例

- 示例：

  ``` csharp
  YIMEngine.IMAPI imAPI = IMAPI.Instance();
  ```

### 初始化SDK管理器 

获得`IMAPI`之后，需要调用初始化接口初始化IM SDK。

- 接口：

  ``` csharp
  public ErrorCode Init(string strAppKey,string strSecrect, ServerZone serverZone);
  ```

- 参数：
  `strAppKey`：用户游戏产品区别于其它游戏产品的标识，可以在[游密官网](https://account.youme.im/login)获取、查看
  `strSecrect`：用户游戏产品的密钥，可以在[游密官网](https://account.youme.im/login)获取、查看
  `serverZone`：服务器区域参数，参考ServerZone枚举

- 返回：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)

- 示例：

  ``` csharp
    var errorcode = IMAPI.Instance().Init ("YoumeAppkey"," YoumeAppSecurity",ServerZone.China);
    if(errorcode !=  ErrorCode.Success)
    {
        Debug.Log("Init errorcode: " + errorcode);
    }
  ```

### 设置监听 
需要开发者实现的监听类有:

>`LoginListen`  
>`MessageListen`  
>`ChatRoomListen`  
>`DownloadListen`  
>`ContactListen`  
>`LocationListen`  
>`NoticeListen`  
>`ReconnectListen`  
>`AudioPlayListen`  

若需使用关系链，需要增加两个监听类的实现：
>`UserProfileListen`  
>`FriendListen`  

它们的功能介绍见SDK对象简介。如果不设定就收不到相应事件的通知。建议初始化和设置监听在一起完成。
** 注：所有回调都在主线程调用**

- 接口原型：

  ``` csharp
      public void SetLoginListen(YIMEngine.LoginListen listen);
      public void SetMessageListen(YIMEngine.MessageListen listen);
      public void SetChatRoomListen(YIMEngine.ChatRoomListen listen);
      public void SetDownloadListen(YIMEngine.DownloadListen listen) ;
      public void SetContactListen (YIMEngine.ContactListen listen); //可选
      public void SetNoticeListen(NoticeListen listen); //可选
      public void SetAudioPlayListen(AudioPlayListen listen);
      public void SetLocationListen(LocationListen listen); //可选
      public void SetReconnectListen(ReconnectListen listen); //可选
      public void SetFriendListen(FriendListen listen); //可选
      public void SetUserProfileListen (UserProfileListen listen); //可选
      
  ```

- 示例：
    ``` csharp
    public class Test : MonoBehaviour,
                            YIMEngine.LoginListen,
                            YIMEngine.ChatRoomListen,
                            YIMEngine.DownloadListen,
                            YIMEngine.ContactListen,
                            YIMEngine.MessageListen,
                            YIMEngine.LocationListen,
                            YIMEngine.AudioPlayListen,
                            YIMEngine.NoticeListen,
                            YIMEngine.ReconnectListen{
        // Use this for initialization
        void Start () {
            IMAPI.Instance().SetLoginListen (this);
            IMAPI.Instance().SetMessageListen (this);
            IMAPI.Instance().SetChatRoomListen(this);
            IMAPI.Instance().SetDownloadListen(this);
            IMAPI.Instance().SetContactListen(this);
            IMAPI.Instance().SetAudioPlayListen(this);
            IMAPI.Instance().SetLocationListen(this);
            IMAPI.Instance().SetLoginListen(this);
            IMAPI.Instance().SetReconnectListen(this);
            IMAPI.Instance().Init ("YoumeAppkey", "YoumeAppSecurity");
        }
    }
    ```
     
### 重连回调 

- 开始重连回调：
  
  ``` csharp
  public interface ReconnectListen
  {
     void OnStartReconnect();
  }
  ```
  
- 重连结果回调：
  
  ``` csharp
  public interface ReconnectListen
  {
    void OnRecvReconnectResult(ReconnectResult result);
  } 
  ```
  
- 参数:
  `result `：重连结果，枚举类型，
             ReconnectResult.RECONNECTRESULT_SUCCESS=0 //重连成功
             ReconnectResult.RECONNECTRESULT_FAIL_AGAIN=1 //重连失败，再次重连
             ReconnectResult.RECONNECTRESULT_FAIL=2 //重连失败
  
### 录音音量回调 
  音量值范围:0~1, 频率:1s ios 约2次，android 约8次
  
  ``` csharp
  public interface MessageListen
  {
    void OnRecordVolumeChange(float volume);
  }
  ```
  
  - 参数：
    `volume `：音量值
  
## 用户管理
### 用户登录 
完成以上的步骤后就可以使用IM功能了，IM用户登录IM后台服务器后即可以正常收发消息。登录为异步过程，通过回调函数返回是否成功，成功后方能进行后续操作。用户首次登录会自动注册。

- 原型：

  ``` csharp
  public ErrorCode Login(string strYouMeID,string strPasswd,string strToken="");
  ```

- 参数说明：
  `strYouMeID`：用户ID，由调用者分配，不可为空字符串，只可由字母或数字或下划线组成，长度限制为255字节
  `strPasswd`：用户密码，不可为空字符串，如无特殊要求可以设置为固定字符串
  `strToken`：使用服务器token验证模式时使用该参数，否则使用默认值""即可，由restAPI获取token值

- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)

- 登录结果通知接口：

  ``` csharp
  public interface LoginListen
  {
     void OnLogin (YIMEngine.ErrorCode errorcode, string strYouMeID);
  }
  ```

- 参数说明：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `strYouMeID`：用户ID

- 示例：

  ``` csharp
      var errorcode = IMAPI.Instance().Login("123456","123456");
      Debug.Log("login,errorcode: " + errorcode);
  ```

- 异步结果处理：

  ``` csharp
      public void OnLogin (YIMEngine.ErrorCode errorcode, string strYouMeID)
      {
        Debug.Log ("OnLogin, errorcode" + errorcode + " userid:" + strYouMeID.ToString());
      }
  ```

### 用户登出 
如用户主动退出或需要进行用户切换，则需要调用登出操作。

- 原型：

  ``` csharp
  // 登出
  public ErrorCode Logout();
  ```
  
- 参数：
  无。  
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
- 登出结果通知接口：

  ``` csharp
  public interface LoginListen
  {
     void OnLogout();
  }
  ```

- 示例：

  ``` csharp
  //登出
  var errorcode =  IMAPI.Instance().Logout();
  Debug.Log("Logout,errorcode: " + errorcode);
  ```
  
### 被用户踢出通知
同一个用户ID在多台设备上登录时，后登录的会把先登录的踢下线，收到OnKickOff()通知。

- 原型：

``` csharp
public interface LoginListen
{
    void OnKickOff();
}
```   
    
### 用户信息
#### 设置用户的详细信息 
设置用户信息后，在游密消息管理后台才能看到消息发送者的详细信息。

- 原型：
  
  ``` csharp
  public ErrorCode SetUserInfo(IMUserInfo userInfo)
  ```

- 参数：
  `userInfo`：用户详细信息，请设置IMUserInfo的以下属性：
  `NickName`：用户昵称
  `ServerAreaID`：游戏服ID
  `ServerArea`：游戏服名称
  `LocationID`：游戏大区ID
  `Location`：游戏大区名
  `Platform`：平台名称，比如：应用宝
  `PlatformID`：平台ID
  `Level`：角色等级
  `VipLevel`：玩家VIP等级
  `Extra`：其它自定义扩展信息，建义用json字符串
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
#### 获取指定用户的详细信息 

- 原型：
  
  ``` csharp
  public ErrorCode GetUserInfo(string userID)
  ```

- 参数：
  `userID`：用户的ID（登录IM时用的ID）
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface ContactListen
  {
     void OnGetUserInfo(ErrorCode code, string userID, IMUserInfo userInfo);
  }   
  ```
  
- 参数：
  `code`：错误码
  `userID`：用户ID
  `userInfo`：用户详细信息
  
- 备注：
  `IMUserInfo` 类成员如下：
  `NickName`：昵称
  `ServerArea`：游戏服名称
  `ServerAreaID`：游戏服ID
  `Location`：游戏大区名称
  `LocationID`：游戏大区ID
  `Platform`：平台名称，比如：应用宝
  `PlatformID`：平台ID   
  `Level`：角色等级
  `VipLevel`：玩家VIP等级
  `Extra`：扩展信息

#### 查询用户在线状态 

- 原型：
    
  ``` csharp
  public ErrorCode QueryUserStatus(string userID);
  ```

- 参数：
  `userID`：用户的ID（登录IM时用的ID）
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：
  
  ``` csharp
  public interface ContactListen
  {      
     void OnQueryUserStatus(ErrorCode code, string userID, UserStatus status);
  }   
  ```
  
- 参数：
  `code`：错误码，查询请求是否成功的通知 
  `userID`：查询的用户ID  
  `status`：登录状态，枚举类型，
            UserStatus.STATUS_ONLINE=0  //在线
            UserStatus.STATUS_OFFLINE=1 //离线

## 频道管理
### 加入频道 
通过`IMAPI`的`JoinChatRoom`接口加入聊天频道，如果频道不存在则后台自动创建。有了这个ID就可以收发频道消息。开发者要保证频道号全局唯一，以避免用户进入错误频道。结果需要开发者在实现`ChatRoomListen`的`OnJoinRoom`里处理。

- 原型：

  ``` csharp
  public ErrorCode JoinChatRoom (string strChatRoomID);
  ```

- 参数：
  `strChatRoomID`：请求加入的频道ID，仅支持数字、字母、下划线组成的字符串，区分大小写，长度限制为255字节
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
- 异步回调接口：

  ``` csharp
  public interface ChatRoomListen
  {
     void OnJoinRoom(YIMEngine.ErrorCode errorcode,string strChatRoomID);
  }
  ```

- 参数说明：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `strChatRoomID`：频道ID  

- 示例：
  
  ``` csharp
  var errorcode = IMAPI.Instance(). JoinChatRoom("123456");
  Debug.Log("JoinChatRoom(): " + "errorcode: " + errorcode);

  // 异步结果处理
  public void OnJoinRoom(YIMEngine.ErrorCode errorcode,string strChatRoomID);
  {
      …
  }
  ```

### 离开频道 
通过`IMAPI`的`LeaveChatRoom`接口离开频道。

- 原型：

  ``` csharp
  public ErrorCode LeaveChatRoom(string strChatRoomID);
  ```

- 参数：
  `strChatRoomID`：请求离开的频道ID
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
- 异步回调接口：

  ``` csharp
  public interface ChatRoomListen
  {
     void OnLeaveRoom(YIMEngine.ErrorCode errorcode,string strChatRoomID);
  }
  ```

- 参数说明：
  `errorcode`：错误码，详细描述见[错误码定义](#错误码定义)
  `strChatRoomID`：频道ID  

- 示例：
  
  ``` csharp
  var errorcode = IMAPI.Instance(). LeaveChatRoom("123456");
  Debug.Log("LeaveChatRoom(): " + "errorcode: " + errorcode);
  ```
  
### 离开所有频道 
通过`IMAPI`的`LeaveAllChatRooms`接口离开所有频道，异步返回结果通过`ChatRoomListen`的`OnLeaveAllRooms`接口返回。

- 原型：

  ``` csharp
  public ErrorCode LeaveAllChatRooms();
  ```
  
- 参数：
  无
  
- 异步回调接口：

  ``` csharp
  public interface ChatRoomListen
  {  
     void OnLeaveAllRooms(YIMEngine.ErrorCode errorcode);
  }  
  ```
  
- 参数：
  `errorcode`:  错误码
  
- 示例：
  
  ``` csharp
      var errorcode = IMAPI.Instance().LeaveAllChatRooms();
  Debug.Log("LeaveAllChatRooms(), errorcode: " + errorcode);
  
  // 异步结果处理
  void OnLeaveAllRooms(YIMEngine.ErrorCode errorcode);
  {
      …
  }
  ```  

### 用户进出频道通知 
此功能默认不开启，需要的请联系我们开启此服务。联系我们，可以通过专属游密支持群或者技术支持的大群。

小频道(小于100人)内，当其他用户进入频道，会收到`OnUserJoinChatRoom`通知

- 原型：
  
  ``` csharp
  public interface ChatRoomListen
  {
    void OnUserJoinChatRoom(string strRoomID, string strUserID);
  }
  ```

- 参数：
  `strRoomID `：频道ID
  `strUserID`：用户ID

小频道(小于100人)内，当其他用户退出频道，会收到`OnUserLeaveChatRoom`通知

- 原型：
  
  ``` csharp
  public interface ChatRoomListen
  {
     void OnUserLeaveChatRoom(string strRoomID, string strUserID);
  }
  ```

- 参数：
  `strRoomID `：频道ID
  `strUserID`：用户ID

### 获取房间成员数量 
获取进入频道的用户数量。

- 原型：

  ``` csharp
  public ErrorCode GetRoomMemberCount(string strChatRoomID)
  ```
    
- 参数：
  `strChatRoomID `：频道ID(已成功加入此频道才能获取该频道的人数)
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface ChatRoomListen
  {
     void OnGetRoomMemberCount(YIMEngine.ErrorCode errorcode, string strRoomID, uint count);
  }
  ```
    
- 参数：
  `errorcode `：错误码
  `strRoomID`：频道ID
  `count`：频道中的成员数量 

## 消息管理
### 接收消息 
通过`MessageListen`的`OnRecvMessage`接口被动接收消息，需要开发者实现。

- 原型：
  
  ``` csharp
  public interface MessageListen
  {  
     void OnRecvMessage(YIMEngine.MessageInfoBase message);
  }
  ```

- 参数：
  `message`：IM消息基类

- 备注：  
  `YIMEngine.MessageInfoBase` 消息基类成员如下：
  `SenderID`：消息发送者ID
  `RecvID`：消息接收者ID
  `ChatType`：聊天类型，私聊/频道聊天
  `RequestID`：消息ID
  `MessageType`：消息类型，枚举类型，
                 MessageBodyType.Unknow = 0,         //未知类型
		            MessageBodyType.TXT = 1,            //文本消息
		            MessageBodyType.CustomMesssage = 2, //自定义消息
		            MessageBodyType.Emoji = 3,          //表情
		            MessageBodyType.Image = 4,          //图片
		            MessageBodyType.Voice = 5,          //语音
		            MessageBodyType.Video = 6,          //视频
		            MessageBodyType.File = 7,           //文件
		            MessageBodyType.Gift = 8            //礼物
  `CreateTime`：消息发送时间 
  `Distance`：若是附近频道的消息，此值是该消息用户与自己的地理位置距离
  `IsRead`：消息是否已读
  
  `TextMessage`继承`MessageInfoBase`类，类成员如下：
  `Content`：文本消息内容，string型
  `AttachParam`：发送文本附加信息，string型
  
  `CustomMessage`继承`MessageInfoBase`类，类成员如下：
  `Content`：自定义消息内容，byte[]型
  
  `FileMessage`继承`MessageInfoBase`类，类成员如下：
  `FileName`：文件名，string型
  `FileType`：文件类型，枚举类型，
               FileType.FileType_Other = 0   //其它
               FileType.FileType_Audio = 1   //音频（语音）
               FileType.FileType_Image = 2   //图片
               FileType.FileType_Video = 3   //视频
  `FileSize`：文件大小，int型
  `FileExtension`：文件扩展信息，string型
  `ExtParam`：附加信息，string型
  
  `GiftMessage`继承`MessageInfoBase`类，类成员如下：
  `Anchor`：主播ID，string型
  `GiftID`：礼物ID，int型
  `GiftCount`：礼物数量，int型
  `ExtParam`：附加参数，ExtraGifParam型
  
  `VoiceMessage`继承`MessageInfoBase`类，类成员如下：
  `Text`：若使用的是语音转文字录音，此值为语音识别的文本内容，否则是""，string型
  `Duration`：语音时长（单位：秒），int型
  `Param`：发送语音时的附加参数，string型
    
- 示例：

  ``` csharp
  public void OnRecvMessage (YIMEngine.MessageInfoBase message)
  {
      if(message.ChatType == YIMEngine.ChatType.PrivateChat){
          Debug.Log ("私聊消息");
      }else{
          Debug.Log ("频道消息");
      }
      Debug.Log ("消息发送时间："+ message.CreateTime);
      if (message.MessageType == YIMEngine.MessageBodyType.TXT)
      {
          YIMEngine.TextMessage textMsg = (YIMEngine.TextMessage)message;
          Debug.Log ("OnRecvMessage text:" + textMsg.Content
                    + " 发送者id:" + textMsg.SenderID + " 接收者id:" + textMsg.RecvID);
      }
      else if (message.MessageType ==
               YIMEngine.MessageBodyType.CustomMesssage)
      {
           YIMEngine.CustomMessage customMsg = (YIMEngine.CustomMessage)message;
           Debug.Log ("OnRecvMessage custom:" +
           System.Convert.ToBase64String(customMsg.Content) + " send:" + customMsg.SenderID
                      + "recv:" + customMsg.RecvID);
      }else if (message.MessageType == YIMEngine.MessageBodyType.Voice) {
          YIMEngine.VoiceMessage voiceMsg = (YIMEngine.VoiceMessage)message;
          Debug.Log (
            "OnRecvMessage voice 文字识别结果:" + voiceMsg.Text
            + " send:" + voiceMsg.SenderID
            + "recv:" + voiceMsg.RecvID
            + " 语音时长:" + voiceMsg.Duration +"秒"
            + " 自定义参数:" + voiceMsg.Param
          );
          //下载语音文件
          IMAPI.Instance().DownloadAudioFile(voiceMsg.RequestID,"/sdcard/abc.wav");
      }
  }
  ```

### 发送文本消息 
可通过`IMAPI`的`SendTextMessage`接口发送消息。异步返回结果通过`MessageListen`的`OnSendMessageStatus`接口返回。

- 原型：
    
  ``` csharp
  public ErrorCode SendTextMessage(string strRecvID, YIMEngine.ChatType chatType,string strContent, string strAttachParam, ref ulong iRequestID);
  ```

- 参数：
  `strRecvID`：接收者ID（用户ID或者频道ID）
  `chatType`：聊天类型，私聊/频道聊天
  `strContent`：文本内容
  `strAttachParam`：发送文本附加信息
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
     void OnSendMessageStatus(ulong iRequestID,  YIMEngine.ErrorCode errorcode, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID);
  }
  ```
  
- 参数：
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码
  `sendTime`：消息发送时间
  `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
  `forbidEndTime`：若在频道被禁言，禁言结束时间
  `messageID`: 发送语音消息的消息ID

- 示例：

  ``` csharp
    ulong iRequestID = 0;
    var errorcode =
    IMAPI.Instance().SendTextMessage("123456",YIMEngine.ChatType.PrivateChat,"u3d DocMsg",ref iRequestID);
    Debug.Log("sendmessage，RequestID:" + iRequestID + " errorcode: " + errorcode);

    异步结果处理:
    public void OnSendMessageStatus (ulong iRequestID,  YIMEngine.ErrorCode errorcode ,uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID)
    {
        Debug.Log ("OnSendMessageStatus requestID:" + iRequestID +" errorcode:" + errorcode);
    }
   ```
    
### 群发文本消息 
用于群发文本消息的接口，**每次不要超过200个用户**。

- 原型：

  ``` csharp
  public ErrorCode MultiSendTextMessage(List<string> recvLists, string strText);
  ```

- 参数：
  `recvLists`：接受消息的用户ID列表
  `strText`：文本消息内容
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)     
  
### 发送礼物消息 
给主播发送礼物消息的接口，支持在游密主播后台查看礼物消息的详细信息和统计信息。客户端还是通过`OnRecvMessage`接收消息。

- 原型：

  ``` csharp
  public ErrorCode SendGift(string strAnchorId,string strChannel,int iGiftID,int iGiftCount,ExtraGifParam extParam, ref ulong serial);
    ```

- 参数：
  `strAnchorId`：游密后台设置的对应的主播游戏id
  `strChannel`：主播所进入的频道ID，通过`JoinChatRoom`进入此频道的用户可以接收到消息。
  `iGiftID`：礼物物品id，特别的是`0`表示只是留言
  `iGiftCount`：礼物物品数量
  `extParam`：扩展内容，参考ExtraGifParam类详细说明
  `serial`：消息序列号
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
     void OnSendMessageStatus(ulong iRequestID,  YIMEngine.ErrorCode errorcode, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID);
  }
  ```
  
- 参数：
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码
  `sendTime`：文件发送时间
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它 
  `forbidEndTime`：若在频道被禁言，禁言结束时间 
  `messageID`: 发送语音消息的消息ID

### 发送自定义消息 
可通过`IMAPI`的`SendCustomMessage`接口发送用户自定义消息，自定义数据是二进制数据。异步返回结果通过`MessageListen`的`OnSendMessageStatus`接口返回。

- 原型：

  ``` csharp
  public ErrorCode SendCustomMessage(string strRecvID, YIMEngine.ChatType chatTpye, byte[] customMsg, ref ulong iRequestID);
  ```

- 参数：
  `strRecvID`：接收者ID（用户ID或者频道ID）
  `chatType`：聊天类型，私聊/频道聊天
  `customMsg`：自定义消息内容
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
     void OnSendMessageStatus (ulong iRequestID,  YIMEngine.ErrorCode errorcode, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID);
  }
  ```
  
- 参数：
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码
  `sendTime`：消息发送时间
  `isForbidRoom`：若发送的是频道消息，显示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它  
  `forbidEndTime`：若在频道被禁言，禁言结束时间 
  `messageID`: 发送语音消息的消息ID

- 示例：

  ``` csharp
    YIMEngine.ErrorCode errorcode =
    IMAPI.Instance().SendCustomMessage("123456",YIMEngine.ChatType.PrivateChat,"u3d custom msg",ref iRequestID);
    Debug.Log("sendmessage: RequestID:" + iRequestID + "errorcode: " + errorcode);

    //异步结果处理：
    public void OnSendMessageStatus (ulong iRequestID,  YIMEngine.ErrorCode errorcode, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID)
    {
       Debug.Log ("OnSendMessageStatus request:" + iRequestID  + "errorcode:" + errorcode);
    }
    ```
### 发送文件消息 
#### 发送文件
用于发送文件的接口。客户端还是通过`OnRecvMessage`接收消息。

- 原型：

  ``` csharp
  public ErrorCode SendFile(string strRecvID,ChatType chatType,string strFilePath,string strExtParam,FileType fileType, ref ulong iRequestID);
  ```

- 参数：
  `strRecvID`：接收方id
  `chatType`：聊天类型，私聊/频道。
  `strFilePath`：要发送的文件绝对路径，包括文件名
  `strExtParam`：额外信息，格式自己定义，自己解析
  `fileType`：文件类型，枚举类型，
              FileType.FileType_Other=0 //其它
              FileType.FileType_Audio=1 //音频
              FileType.FileType_Image=2 //图片
              FileType.FileType_Video=3 //视频
  `iRequestID`：消息序列号
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
     void OnSendMessageStatus(ulong iRequestID,  YIMEngine.ErrorCode errorcode, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID);
  }
  ```
  
- 参数：
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码
  `sendTime`：文件发送时间
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言，（errorcode==ForbiddenSpeak(被禁言)才有效）
  `reasonType`：若在频道被禁言，禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它 
  `forbidEndTime`：若在频道被禁言，禁言结束时间
  `messageID`: 发送语音消息的消息ID
  
#### 下载文件  
接收消息接口参考[收消息](#收消息)通过`GetMessageType()`分拣出文件消息`File`。然后调用函数`DownloadAudioFile`下载文件。
**windows下，会对下载接口的保存路径参数的'/'转换为'\'；如果传入的保存路径参数不符合windows下的路径格式，下载回调中的保存路径可能和传入的保存路径不同**

- 原型：

  ``` csharp
    //下载语音
  public  ErrorCode DownloadAudioFile(ulong iRequestID, string strSavePath);
  ```

- 参数：
  `iRequestID`：消息ID
  `strSavePath`：文件保存路径（带文件名的全路径），如果目录不存在，SDK会自动创建，必须保证该路径有可写权限
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
    //下载回调接口。
  public interface DownloadListen
  {
     void OnDownload( YIMEngine.ErrorCode errorcode, YIMEngine.MessageInfoBase message, string strSavePath);
  }
  ```

- 参数：
  `errorcode`：下载结果错误码
  `message`：消息结构，`YIMEngine.MessageInfoBase`消息基类详细，查看 [接收消息](#接收消息)备注   
  `strSavePath`：保存路径
  
### 语音消息管理
语音消息聊天简要流程：

- 调用IM的语音发送接口就开始录音；
- 调用结束录音接口后自动停止录音并发送。

接收方接收语音消息通知后，调用方控制是否下载，调用下载接口就可以获取录音内容。然后开发者调用播放接口播放wav音频文件。

#### 设置是否自动下载语音消息 
SetDownloadAudioMessageSwitch()在初始化之后，启动语音之前调用;若设置了自动下载语音消息，不需再调用DownloadAudioFile()接口，收到语音消息时会自动下载，自动下载完成也会收到DownloadAudioFile()接口对应的OnDownload()回调。

- 原型：

  ``` csharp
  public ErrorCode SetDownloadAudioMessageSwitch(bool download);
  ```
  
- 参数：
  `download `：true-自动下载语音消息，false-不自动下载语音消息(默认) 
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
#### 设置语音识别的语言 
`SendAudioMessage`提供语音识别功能，将语音识别为文字，默认输入语音为普通话，识别文字为简体中文，可通过`SetSpeechRecognizeLanguage`函数设置语言；若需要识别为繁体中文，需要联系我们配置此服务。

- 原型：
  
  ``` csharp
  public ErrorCode SetSpeechRecognizeLanguage(SpeechLanguage language)
  ```

- 参数：
  `language`：语言，枚举类型，               
   SpeechLanguage.SPEECHLANG_MANDARIN=0 //普通话
   SpeechLanguage.SPEECHLANG_YUEYU=1    //粤语
   SpeechLanguage.SPEECHLANG_SICHUAN=2  //四川话（windows不支持）
   SpeechLanguage.SPEECHLANG_HENAN=3    //河南话（只支持iOS）
   SpeechLanguage.SPEECHLANG_ENGLISH=4  //英语
   SpeechLanguage.SPEECHLANG_TRADITIONAL=5 //繁体中文
   
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)   

#### 发送语音消息
##### 启动录音（带文字识别）
调用`IMAPI`的`SendAudioMessage`语音发送接口（该接口支持语音转文字，若不需要转文字建议使用`SendOnlyAudioMessage`）就开始录音，调用`StopAudioMessage`接口后自动停止录音并发送，或者调用`CancleAudioMessage`取消本次消息发送。注意：语音消息最大的时长是1分钟（超过1分钟就自动发出去）。

- 原型：

  ``` csharp
    //开始录制语音消息，支持语音转文本
  public ErrorCode SendAudioMessage(string strRecvID,YIMEngine.ChatType chatType,ref ulong iRequestID) ;
  ```

- 参数：
  `strRecvID`：接收者ID，私聊传入userid，频道聊天传入roomid
  `chatType`：聊天类型,私聊/频道聊天
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 启动录音（不带文字识别）

- 原型：

  ``` csharp
    //开始录制语音消息，不支持语音转文本
  public ErrorCode SendOnlyAudioMessage(string strRecvID,YIMEngine.ChatType chatType,ref ulong iRequestID) ;
  ```

- 参数：
  `strRecvID`：接收者ID，私聊传入userid，频道聊天传入roomid
  `chatType`：聊天类型,私聊/频道聊天
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 结束录音并发送 

- 原型：

  ``` csharp
    //结束录音并发送
  public ErrorCode StopAudioMessage(string strParam);
  ```

- 参数：
  `strParam`：发送语音消息的附加参数，格式自定义，自己解析 （json、xml...），可为空字符串。
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义) 
  
- 异步回调接口：
  结束录音并发送接口对应两个回调接口，若语音成功发送出去能得到两个回调通知，语音发送失败则只会得到`发送语音结果回调`。 

  ``` csharp
  public interface MessageListen
  {
    //开始上传语音回调，录音结束，开始发送录音的通知，这个时候已经可以拿到语音文件进行播放
    void OnStartSendAudioMessage(ulong iRequestID, YIMEngine.ErrorCode errorcode, string strText, string strAudioPath, int iDuration);

    //发送语音结果回调，自己的语音消息发送成功或者失败的通知
    void OnSendAudioMessageStatus(ulong iRequestID,YIMEngine.ErrorCode errorcode,string strText,string strAudioPath,int iDuration, uint sendTime, bool isForbidRoom, int reasonType, ulong forbidEndTime, ulong messageID);
   } 
   ```

- 参数：
  `iRequestID`：消息序列号，用于校验一条消息发送成功与否的标识
  `errorcode`：错误码，`ErrorCode.Success` 或者 `ErrorCode.PTT_ReachMaxDuration` 均表示成功
  `strText`：语音转文字识别的文本内容，如果没有用带语音转文字的接口，该字段为空字符串
  `strAudioPath`：录音生成的wav文件的本地完整路径
  `iDuration`：录音时长(单位秒)    
  
  `sendTime`：语音发送时间  
  `isForbidRoom`：若发送的是频道消息，表示在此频道是否被禁言，true-被禁言，false-未被禁言（errorcode==ForbiddenSpeak(被禁言)才有效）    
  `reasonType`：禁言原因，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
  `forbidEndTime`：禁言截止时间戳（errorcode==ForbiddenSpeak(被禁言)才有效）   
  `messageID`: 发送语音消息的消息ID

##### 取消本次录音 

- 原型：

  ``` csharp
    //取消本次录音
  public ErrorCode CancleAudioMessage();
  ```
  
- 参数：
  无
    
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 接收语音消息 
接收消息接口参考[收消息](#收消息)通过`GetMessageType()`分拣出语音消息`Voice`。然后调用函数`DownloadAudioFile`下载语音消息。然后调用方播放。
**windows下，会对下载接口的保存路径参数的'/'转换为'\'；如果传入的保存路径参数不符合windows下的路径格式，下载回调中的保存路径可能和传入的保存路径不同**

- 原型：

  ``` csharp
    //下载语音
  public  ErrorCode DownloadAudioFile(ulong iRequestID, string strSavePath);
  ```

- 参数：
  `iRequestID`：消息ID
  `strSavePath`：文件保存路径（带文件名的全路径），如果目录不存在，SDK会自动创建，必须保证该路径有可写权限
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
    //下载回调接口。
  public interface DownloadListen
  {
     void OnDownload( YIMEngine.ErrorCode errorcode, YIMEngine.MessageInfoBase message, string strSavePath);
  }
  ```

- 参数：
  `errorcode`：下载结果错误码
  `message`：消息结构，`YIMEngine.MessageInfoBase`消息基类详细，查看 [接收消息](#接收消息)备注   
  `strSavePath`：保存路径 

#### 语音播放
##### 播放语音 
可以使用SDK内置的播放接口进行语音播放，目前只支持WAV格式。

- 原型：

  ``` csharp
  public ErrorCode StartPlayAudio(string path);
  ```

- 参数：
  `path`：语音文件的完整路径。
 
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)，`Success`或者`PTT_IsPlaying`，都是操作成功，不过`PTT_IsPlaying`表示覆盖了当前正在播放的音频文件。

- 异步回调接口：
  播放结束的通知。
  
  ``` csharp
  public interface AudioPlayListen
  {     
     void OnPlayCompletion(YIMEngine.ErrorCode errorcode, string path);
   }  
    ```
    
 - 参数：
   `errorcode`：错误码，errorcode == ErrorCode.Success表示正常播放结束，errorcode == ErrorCode.PTT_CancelPlay表示播放到中途被打断。 
   `path`：被播放的音频文件地址。  

##### 停止语音播放 
停止播放当前的语音。

- 原型：

  ``` csharp
  public ErrorCode StopPlayAudio();
  ```

- 参数：
  无。
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 设置语音播放音量 
设置语音播放的音量大小。

- 原型：

  ``` csharp
  public void SetVolume(float volume);
  ```

- 参数：
  `volume`：音量值，取值范围`0.0f` - `1.0f`，默认值为 `1.0f`。

##### 查询播放状态 
查询当前音频播放器的状态。

- 原型：

  ``` csharp
  public bool IsPlaying();
  ```

- 参数：
  无
  
- 返回值：
  `true`：表示正在播放，`false`：表示当前没有播放。

#### 录音缓存
##### 设置录音缓存目录 
设置录音时用于保存录音文件的缓存目录，如果没有设置，SDK会在APP默认缓存路径下创建一个文件夹用于保存音频文件。
**该接口建议初始化之后立即调用**

- 原型：

  ``` csharp
  public void SetAudioCachePath(string cachePath);
  ```

- 参数：
  `cachePath`：缓存目录绝对路径，如果目录不存在，SDK会自动创建。 

##### 获取当前设置的录音缓存目录 
获取当前设置的录音缓存目录。

- 原型：

  ``` csharp
  public string GetAudioCachePath()；
  ```
  
- 参数：
  无  

- 返回值：
  返回当前设置的录音缓存目录的完整路径。

##### 清空录音缓存目录 
清空当前设置的录音缓存目录(注意清空语音缓存目录后历史记录中会无法读取到音频文件，调用清理历史记录接口也会自动删除对应的音频缓存文件)。

- 原型：

  ``` csharp
  public bool ClearAudioCachePath();
  ```
  
- 参数：
  无
  
- 返回值：
  `true`：表示清理成功，`false`：表示清理操作失败。
  
#### 设置下载保存目录 
下载保存目录是DownloadAudioFile的默认下载目录，对下载语音消息和文件均适用；设置下载保存目录在初始化之后，启动语音之前调用，若设置了下载保存目录，调用DownloadAudioFile()时其中的savePath参数可以为空字符串。

- 原型：

  ``` csharp
  public ErrorCode SetDownloadDir(string path);
  ```
  
- 参数：
  `path`：下载目录路径，必须保证该路径可写。
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
#### 设置只识别语音文字 
此功能是为了只获取语音识别的文字，不发送语音消息，接口调用与发送语音消息相同，但在成功录音后仅会收到识别的语音文本通知,不会收到OnSendAudioMessage回调。
实现只识别语音文字功能的接口调用顺序：SetOnlyRecognizeSpeechText->SendAudioMessage->StopAudioMessage->OnGetRecognizeSpeechText

- 原型：

  ``` csharp
  public ErrorCode SetOnlyRecognizeSpeechText(bool recognition);
  ```
  
- 参数：
  `recognition `：true-只识别语音文字，false-识别语音文字并发送语音消息
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 成功调用发送语音消息接口后的通知接口：

  ``` csharp
  public interface MessageListen
  {
     void OnGetRecognizeSpeechText(ulong iRequestID, YIMEngine.ErrorCode errorcode, string text);
  }   
  ```
  
- 参数：
  `iRequestID`：消息ID
  `errorcode`：错误码
  `text`：返回的语音识别的文本内容 
    
#### 获取麦克风状态 
这是一个异步操作，操作结果会通过回调参数返回。（该接口会临时占用麦克风）
- 原型：

  ``` csharp
  public ErrorCode GetMicrophoneStatus();
  ```
  
- 参数：
  无  
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface AudioPlayListen
  {
    void OnGetMicrophoneStatus(YIMEngine.AudioDeviceStatus status);
  }
  ```
  
- 参数：
  `status`：麦克风状态值，枚举类型，
  AudioDeviceStatus.STATUS_AVAILABLE=0    //可用
  AudioDeviceStatus.STATUS_NO_AUTHORITY=1 //无权限
  AudioDeviceStatus.STATUS_MUTE=2         //静音
  AudioDeviceStatus.STATUS_UNAVAILABLE=3  // 不可用     
  
### 录音上传
#### 启动录音 

- 原型：

  ``` csharp
  public ErrorCode StartAudioSpeech(ref ulong requestID, bool translate );
  ```

- 参数：
  `requestID`：返回消息ID
  `translate`：是否进行语音转文字识别
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 停止录音并上传 
该接口只上传到服务器并异步返回音频文件的下载链接，不会自动发送。

- 原型：

  ``` csharp
  public ErrorCode StopAudioSpeech( );
  ```
  
- 参数：
  无  
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)

- 相关接口：
  取消录音，调用该接口后，此次录音不会上传。
  
  ``` csharp  
  public ErrorCode CancleAudioMessage();
  ```

- 异步回调接口：
  
  ``` csharp
  public interface MessageListen
  {  
     void OnStopAudioSpeechStatus(YIMEngine.ErrorCode errorcode, ulong iRequestID,string strDownloadURL,int iDuration,int iFileSize,string strLocalPath,string strText);
  }
  ```
  
- 参数：
  `errorcode`：错误码
  `iRequestID`：消息ID 
  `strDownloadURL`：语音文件的下载地址
  `iDuration`：录音时长，单位秒
  `iFileSize`：文件大小，字节
  `strLocalPath`：本地语音文件的路径
  `strText`：语音识别结果，可能为空null or ""
    
#### 根据Url下载语音文件 

- 原型：
    
  ``` csharp
  public ErrorCode DownloadFileByUrl( string strFromUrl, string strSavePath )
  ```

- 参数：
  `strFromUrl`：语音文件的url地址，从MessageListen.OnStopAudioSpeechStatus回调获得。
  `strSavePath`：下载语音文件的本地存放地址,带文件名的全路径
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：
    
  ``` csharp
  public interface DownloadListen
  {
    void OnDownloadByUrl( YIMEngine.ErrorCode errorcode, string strFromUrl, string strSavePath );
  }
  ```

- 回调参数：
  `errorcode`：错误码
  `strFromUrl`：下载的语音文件url
  `strSavePath`：本地存放地址
  
### 消息屏蔽
#### 屏蔽/解除屏蔽用户消息 
若屏蔽用户的消息，此屏蔽用户发送的私聊/频道消息都接收不到。

- 原型：

  ``` csharp
  public ErrorCode BlockUser(string userID, bool block)
  ```
  
- 参数：
  `userID `：需屏蔽/解除屏蔽的用户ID
  `block `：true-屏蔽 false-解除屏蔽
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
     void OnBlockUser(YIMEngine.ErrorCode errorcode, string userID, bool block);
  }  
  ```
    
- 参数：
  `errorcode `：错误码
  `userID `：用户ID
  `block `：true-屏蔽 false-解除屏蔽    

#### 解除所有已屏蔽用户 
- 原型：

  ``` csharp
  public ErrorCode UnBlockAllUser();
  ```
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
    void OnUnBlockAllUser(YIMEngine.ErrorCode errorcode);
  }  
  ```
    
- 参数：
  `errorcode `：错误码
  
#### 获取被屏蔽消息用户 
获取被自己屏蔽接收消息的所有用户。

- 原型：

  ``` csharp
  public ErrorCode GetBlockUsers();
  ```

- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
     void OnGetBlockUsers(YIMEngine.ErrorCode errorcode, List<string> userList);
  }  
  ```
    
- 参数：
  `errorcode`：错误码
  `userList`：屏蔽用户ID列表    

### 消息功能设置
#### 设置消息已读 

- 功能：收到消息后，将设置消息为已读。

- 原型：

``` csharp   
public ErrorCode SetMessageRead(ulong messageID, bool read);
```

- 参数：
  `messageID`：消息ID
  `read`：是否已读，`true`为已读，`false`为未读
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 切换获取消息模式 
在自动接收消息和手动接收消息间切换，默认是自动接收消息。

- 原型：

  ``` csharp
  public ErrorCode SetAutoRecvMsg(List<string> targets, bool bAutoRecv)；
  ```

- 参数：
  `targets`：频道ID，可以指定多个
  `bAutoRecv`：`true`为自动接收消息，`false`为手动接收消息，默认为`true`。
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)

#### 手动获取消息 
如果设置了手动接收消息模式，需要调用该接口后才能收到 `OnRecvMessage` 通知。

- 原型：

  ``` csharp
  public ErrorCode GetNewMessage(List<string> targets)；
  ```

- 参数：
  `targets`：频道ID，可以指定多个
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

#### 新消息通知 
在手动接收消息模式，若设置了OnRecvNewMessage的监听，会通知新消息的数量。

- 通知接口：
    
  ``` csharp
  public interface MessageListen
  {    
     void OnRecvNewMessage(YIMEngine.ChatType chatType,string targetID);
  }
  ```
  
- 参数：
  `chatType`：聊天类型，可以用于区分是频道聊天还是私聊 
  `targetID`：如果是频道聊天，该值为频道ID；私聊该值为空 

### 消息记录管理  
#### 设置是否保存频道聊天记录 
设置是否在本地保存频道聊天记录，默认不保存。`私聊历史记录默认保存`。

- 原型：

  ``` csharp
  public void SetRoomHistoryMessageSwitch(List<string> roomIDs,bool save);
  ```

- 参数：
  `roomIDs`：频道id，指定要保存本地历史记录的频道ID，可以一次设置多个，对应`JoinChatRoom` 指定的ID。
  `save`：`true`表示保存聊天记录，`false`表示不保存聊天记录，默认为`false`。
  
#### 拉取频道最近聊天记录 
从服务器拉取频道最近的聊天历史记录。
这个功能默认不开启，需要的请联系我们开启此服务。联系我们，可以通过专属游密支持群或者技术支持的大群。

- 原型：

  ``` csharp
  public ErrorCode QueryRoomHistoryMessageFromServer(string roomID, int count, int direction);
  ```

- 参数：
  `roomID`：频道ID
  `count`：消息数量(最大200条)
  `directon`：历史消息排序方向 0：按时间戳升序	1：按时间戳逆序

- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)
  
- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {
      void OnQueryRoomHistoryMessageFromServer(YIMEngine.ErrorCode errorcode, string roomID, int remain, List<YIMEngine.MessageInfoBase> messageList);
  }
  ```
  
- 参数：   
  `errorcode`：错误码
  `roomID`：频道ID
  `remain`：剩余消息数量
  `messageList`：消息列表
  
- 备注：
  `YIMEngine.MessageInfoBase`消息基类详细，查看 [接收消息](#接收消息)备注   
     
#### 本地历史记录管理
##### 查询本地聊天历史记录 

- 原型：

  ``` csharp
  public ErrorCode QueryHistoryMessage(string targetID, ChatType chatType, ulong startMessageID, int count, int direction);
  ```

- 参数：
  `targetID`：用户ID或者频道ID
  `chatType`：表示查询私聊或者频道聊天的历史记录
  `startMessageID`：起始历史记录消息id（与requestid不同），为`0`表示首次查询，将倒序获取`count`条记录
  `count`：最多获取多少条
  `direction`：历史记录查询方向，`startMessageID=0`时,direction使用默认值0；`startMessageID>0`时，`0`表示查询比`startMessageID`小的消息，`1`表示查询比`startMessageID`大的消息
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 相关函数：
  对于房间本地记录，需要先设置自动保存房间消息。`SetRoomHistoryMessageSwitch`

- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {      
      void OnQueryHistoryMessage(YIMEngine.ErrorCode errorcode, string targetID, int remain, List <YIMEngine.HistoryMsg> messageList);
  }
  ```
  
- 参数：
  `errorcode`：错误码
  `targetID`：用户ID/频道ID
  `remain`：剩余历史消息条数
  `messageList`：消息列表
  
- 备注：
  `YIMEngine.HistoryMsg`类成员如下：
  `SenderID`：消息发送者ID
  `ReceiveID`：消息接收者ID
  `MessageID`：消息ID
  `ChatType`：聊天类型，私聊/频道聊天
  `MessageType`：消息类型，枚举类型，
                 MessageBodyType.Unknow = 0,         //未知类型
		            MessageBodyType.TXT = 1,            //文本消息
		            MessageBodyType.CustomMesssage = 2, //自定义消息
		            MessageBodyType.Emoji = 3,          //表情
		            MessageBodyType.Image = 4,          //图片
		            MessageBodyType.Voice = 5,          //语音
		            MessageBodyType.Video = 6,          //视频
		            MessageBodyType.File = 7,           //文件
		            MessageBodyType.Gift = 8            //礼物
  
  `Text`：文本消息内容/语音消息的文本识别内容
  `IsRead`：消息是否已读，true-已读，false-未读
  `LocalPath`：语音消息文件的本地路径
  `CreateTime`：消息收发时间  
  `Param`：如果是语音消息，该值表示语音消息的自定义附加参数
  `Duration`：如果是语音消息，该值表示语音消息时长  
  `CustomMsg`：如果是自定义消息，该值为自定义消息内容
  
##### 根据时间清理本地聊天历史记录 
建议定期清理本地历史记录。

- 原型：

  ``` csharp
  public ErrorCode DeleteHistoryMessage(ChatType chatType, ulong time );
  ```

- 参数：
  `chatType`：指定是清理频道消息还是私聊消息
  `time`：Unix timestamp,精确到秒，表示删除这个时间点之前的所有历史记录
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 根据消息ID清理本地聊天历史记录 
可以根据消息ID删除对应的本地私聊历史记录。

- 原型：

  ``` csharp
  public ErrorCode DeleteHistoryMessageByID(ulong messageID);
  ```

- 参数：
  `messageID`：将删除指定消息id的历史记录，此值对应 `YIMEngine.HistoryMsg` 的 `MessageID` 属性值。
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
##### 根据用户ID清理本地私聊历史记录 
可以根据用户ID删除对应的本地私聊历史记录,保留消息ID白名单中的消息记录,白名单列表为空时删除与该用户ID间的所有本地私聊记录。

- 原型：

  ``` csharp
  public ErrorCode DeleteSpecifiedHistoryMessage(string targetID, ChatType chatType, ulong[] excludeMesList);
  ```

- 参数：
  `targetID`：指定的用户ID。
  `chatType`：指定清理私聊消息。
  `excludeMesList`：与该用户ID的私聊消息ID白名单。
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
##### 根据用户或房间ID清理本地聊天历史记录 
可以根据用户ID或者房间ID删除以指定的起始消息ID开始的n条本地私聊历史记录，起始消息ID和消息数量使用默认值时则删除所有消息。

- 原型：

  ``` csharp
  public ErrorCode DeleteHistoryMessageByTarget(string targetID, ChatType chatType, ulong startMessageID, uint count);
  ```

- 参数：
  `targetID`：用户ID或者频道ID
  `chatType`：聊天类型，私聊/频道聊天
  `startMessageID`：起始消息ID(默认值0表示最近的一条消息)
  `count`：消息数量(默认值0表示删除所有消息)
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

##### 获取最近私聊联系人列表 
该接口是根据本地历史消息记录生成的最近联系人列表，按最后聊天时间倒序排列。该列表会受清理历史记录消息的接口影响。

- 原型：

  ``` csharp
  public ErrorCode GetHistoryContact();
  ```
  
- 参数：
  无  
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ``` csharp
  public interface ContactListen
  {      
     void OnGetContact(List<ContactsSessionInfo> contactLists);
  }
  ```
  
- 参数：
  `contactLists`：最近联系人列表
  
- 备注：
  `ContactsSessionInfo`类成员如下：
  `ContactID`：联系人ID
  `MessageType`：消息类型，枚举类型，
                 MessageBodyType.Unknow = 0,         //未知类型
		            MessageBodyType.TXT = 1,            //文本消息
		            MessageBodyType.CustomMesssage = 2, //自定义消息
		            MessageBodyType.Emoji = 3,          //表情
		            MessageBodyType.Image = 4,          //图片
		            MessageBodyType.Voice = 5,          //语音
		            MessageBodyType.Video = 6,          //视频
		            MessageBodyType.File = 7,           //文件
		            MessageBodyType.Gift = 8            //礼物 
  `MessageContent`：消息内容
  `CreateTime`：消息创建时间 
  `NotReadMsgNum`：未读消息数量（此值有意义需结合SetMessageRead接口使用，可在收到消息后设置消息已读）
  
### 高级功能 
#### 文本翻译 
将文本翻译成指定语言的文本，异步返回结果

- 原型：
  
  ``` csharp
  public void TranslateText( string text, LanguageCode destLangCode, LanguageCode srcLangCode,System.Action<ErrorCode,string, LanguageCode, LanguageCode> callback)
  ```

- 参数：
  `text`：待翻译文本
  `destLangCode`：目标语言编码
  `srcLangCode`：原语言编码
  `System.Action<ErrorCode,string,LanguageCode,LanguageCode>`： 回调

- 备注：
  System.Action<ErrorCode,string,LanguageCode,LanguageCode> 参数：
  `ErrorCode`：错误码
  `string`：翻译成指定语言的文本
  `LanguageCode`：srcLangCode 原语言
  `LanguageCode`：destLangCode 目标语言
    
#### 敏感词过滤 
考虑到客户端可能需要传递一些自定义消息，关键字过滤方法就直接提供出来，客户端可以选择是否过滤关键字，并且可以根据匹配到的等级能进行广告过滤。比如`level`返回的值为'2'表示广告，那么可以在发送时选择不发送，接收方可以选择不展示。

- 原型：

  ``` csharp
  public static string GetFilterText(string strSource, ref int level);
  ```

- 参数：
  `strSource`：消息原文
  `level`：匹配的策略词等级，默认为0，`0`表示正常，其余值是按需求可配

- 返回值：
  过滤关键字后的消息内容，敏感字会替换为"*"
  
- 备注：
  若对等级无需求可使用下面接口
  
  ``` csharp
  public static string GetFilterText(string strSource);
  ```

## 游戏暂停与恢复
### 游戏暂停通知 
建议游戏放入后台时通知该接口，以便于得到更好重连效果；
调用OnPause(false),在游戏暂停后,若IM是登录状态，依旧接收IM消息；
调用OnPause(true),游戏暂停，即使IM是登录状态也不会接收IM消息；在游戏恢复运行时会主动拉取暂停期间未接收的消息，收到OnRecvMessage()回调。

- 原型：

  ``` csharp
  public void OnPause(bool pauseReceiveMessage);
  ```
  
- 参数：
  `pauseReceiveMessage`：是否暂停接收IM消息，true-暂停接收 false-不暂停接收  

### 游戏恢复运行通知 
建议游戏恢复运行时通知该接口，以便于得到更好重连效果。建议添加到全局MonoBehaviour的OnApplicationPause通知里。

- 原型：

  ``` csharp
  public void OnResume();
  ```
  
## 用户举报和禁言 
### 用户举报 
`Accusation`接口提供举报功能，对用户违规的发言内容进行举报，管理员在后台进行审核处理并将结果通知给用户。

- 原型
  
  ``` csharp
  public ErrorCode Accusation(string userID, ChatType source, int reason, string description, string extraParam)
  ```

- 参数：
  `userID`：被举报用户ID
  `source`：来源（私聊/频道）
  `reason`：原因，由用户自定义枚举
  `description`：原因描述
  `extraParam`：附加信息JSON格式 ({"nickname":"","server_area":"","level":"","vip_level":""}）
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)   

- 举报处理结果通知
当管理员对举报进行审核处理后，会将举办处理结果通知用户

- 原型：
  
  ``` csharp
  public interface MessageListen
  {
     void OnAccusationResultNotify(AccusationDealResult result, string userID, uint accusationTime);
  }
  ```

- 参数：
  `result`：处理结果,枚举类型，
  AccusationDealResult.ACCUSATIONRESULT_IGNORE=0 // 忽略   
  AccusationDealResult.ACCUSATIONRESULT_WARNING=1 // 警告
  AccusationDealResult.ACCUSATIONRESULT_FROBIDDEN_SPEAK=2 // 禁言        
  `userID`：被举报用户ID
  `accusationTime`：举报时间
  
### 禁言查询 
用户查询其所在频道的禁言状态。

- 原型：

  ``` csharp
  public ErrorCode GetForbiddenSpeakInfo();
  ```
  
- 参数：
  无  

- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)    
    
- 异步回调接口：

  ``` csharp
  public interface MessageListen
  {    
    void OnGetForbiddenSpeakInfo( YIMEngine.ErrorCode errorcode, List<ForbiddenSpeakInfo> forbiddenSpeakList );
  }  
  ```
  
- 参数：
  `errorcode`：错误码
  `forbiddenSpeakList`：禁言状态列表,每一个代表一个频道的禁言状态  
  
- 备注：
  `ForbiddenSpeakInfo` 类成员如下：
  `ChannelID`：频道ID
  `IsForbidRoom`：此频道是否被禁言，true-被禁言，false-未被禁言
  `ReasonType`：禁言原因类型，0-未知，1-发广告，2-侮辱，3-政治敏感，4-恐怖主义，5-反动，6-色情，7-其它
  `EndTime`：若被禁言，禁言结束时间   

## 公告管理
基本流程：申请开通公告功能->后台添加新公告然后设置公告发送时间,公告消息类型,发送时间,接收公告的频道等。->(客户端流程) 设置对应监听-> 调用对应接口->回调接收
### 公告功能简述

1.公告发送由后台配置，如类型、周期、发送时间、内容、链接、目标频道、次数、起始结束时间等。

2.公告三种类型：跑马灯，聊天框，置顶公告，
(1) 跑马灯，聊天框公告可设置发送时间，次数和间隔（从指定时间点开始隔固定间隔时间发送多次，界面展示及显示时长由客户端决定）；
(2) 置顶公告需设置开始和结束时间（该段时间内展示）。

3.三种公告均有一次性、周期性两种循环属性：
  一次性公告，到达指定时间点，发送该条公告;
  周期性公告，跟一次性公告发送规则一致，但是可以设置发送周期（在每周哪几天的指定时间发送）。
  
4.跑马灯与聊天框公告只有发送时间点在线的用户才能收到该公告，显示规则由客户端自己决定，两者区别主要是界面显示的区分。

5.置顶公告有显示起始和结束时间，表示该时段内显示，公告发送时间点在线的用户会收到该公告，公告发送时间点未在线用户，在公告显示时段登录，登录后可通过查询公告接口查到该公告。

6.公告撤销
仅针对置顶公告，公告显示时段撤销公告，客户端会收到公告撤销通知，界面进行更新。

### 接收公告 
管理员在后台发布公告，当到达指定时间会收到该公告，界面可以根据不同类型的公告进行展示。

- 原型：
  
  ``` csharp
  public interface NoticeListen
  {
     void OnRecvNotice(YIMEngine.Notice notice);
  }
  ```

- 参数：
  `notice`：公告信息

- 备注：
  `YIMEngine.Notice`类成员如下：
  `NoticeID`：公告ID，ulong型
  `NoticeType`：公告类型，int型，1-跑马灯公告  2-聊天框公告  3-置顶公告
  `ChannelID`：发布公告的频道ID，string型
  `Content`：公告内容，string型
  `LinkText`：公告链接关键字，string型
  `LinkAddr`：公告链接地址，string型
  `BeginTime`：公告开始时间，uint型
  `EndTime`：公告结束时间，uint型
  
### 撤销公告 
对于某些类型的公告（如置顶公告），需要在界面展示一段时间，如果管理员在该时间段执行撤销该公告，会收到撤销公告通知，界面进行相应更新。

- 原型：
  
  ``` csharp
  public interface NoticeListen
  {
    void OnCancelNotice(ulong noticeID, string channelID);
  }
  ```

- 参数：
  `noticeID`：公告ID
  `channelID`：发布公告的频道ID

### 查询公告 
公告在配置的时间点下发到客户端，对于某些类型的公告（如置顶公告）需要在某个时间段显示在，如果用户在公告下发时间点未在线，而在公告展示时间段内登录，应用可根据自己的需要决定是否展示该公告，`QueryNotice`查询公告，结果通过上面的`OnRecvNotice`异步回调返回。

- 原型：
  
  ``` csharp
  public ErrorCode QueryNotice();
  ```
  
- 参数：
  无  
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  
  
## 地理位置
### 获取当前地理位置 
获取用户当前的地理位置

- 原型：

  ``` csharp
  public ErrorCode GetCurrentLocation()
  ```
  
- 参数：
  无  

- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)
  
- 异步回调接口：
  地理位置信息的通知。
  
  ``` csharp
  public interface LocationListen
  {       
      void OnUpdateLocation(YIMEngine.ErrorCode errorcode, YIMEngine.GeographyLocation location);

  }
  ```
  
- 参数：
  `errorCode`：错误码 
  `location`：当前地理位置及行政区划信息  

- 备注：
  `GeographyLocation`类成员如下：
  `DistrictCode`：uint类型，地理区域编码，可根据此参数组合附近频道id
  `Country`：string类型，国家
  `Province`：string类型，省份
  `City`：string类型，城市
  `DistrictCounty`：string类型，区县
  `Street`：string类型，街道
  `Longitude`：double类型，经度
  `Latitude`：double类型，纬度
  
### 获取附近的人 
获取附近的目标(人 房间) ，若需要此功能，请联系我们开启LBS服务。若已开启服务，此功能生效的前提是自己和附近的人都获取了自己的地理位置，即调用了IM的获取当前地理位置接口。

- 条件：需要开通LBS定位服务才能使用

- 原型：

 ``` csharp
 public ErrorCode GetNearbyObjects(int count, string serverAreaID, DistrictLevel districtlevel = YIMEngine.DistrictLevel.DISTRICT_UNKNOW, bool resetStartDistance = false)
 ```

- 参数：
  `count`: 获取附近的目标数量（一次最大200）
  `serverAreaID`：区服（对应设置用户信息中的区服，如果只需要获得本服务器的，要填；否则填""）
  `districtlevel`：行政区划等级，枚举类型，
  DistrictLevel.DISTRICT_UNKNOW=0   //未知
  DistrictLevel.DISTRICT_COUNTRY=1  //国家
  DistrictLevel.DISTRICT_PROVINCE=2 //省份
  DistrictLevel.DISTRICT_CITY=3     //城市
  DistrictLevel.DISTRICT_COUNTY=4   //区县
  DistrictLevel.DISTRICT_STREET=5   //街道
  `resetStartDistance`：是否重置查找起始距离,true-从距自己0米开始查找，false-从上次查找返回的最远用户开始查找
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：
 
  ``` csharp
  public interface LocationListen
  {
        /**
         *获取附近目标回调
         * errorcode 错误码
         * neighbourList：附近的人地理位置信息列表
        *  startDistance：查找起始距离
        *  endDistance：查找终止距离
         */
     void OnGetNearbyObjects(YIMEngine.ErrorCode errorcode, List<YIMEngine.RelativeLocation> neighbourList, uint startDistance, uint endDistance);
    }

   ```

 - 参数：
   `errorcode`：错误码
   `neighbourList`：附近的人地理位置信息列表   
   `startDistance`：搜索的起始距离
   `endDistance`：搜索的结束距离
   
 - 备注：  
   `YIMEngine.RelativeLocation`类成员如下：
   `Distance`：与自己的距离
   `Longitude`：经度
   `Latitude`：纬度
   `UserID`：附近用户ID
   `Country`：国家
   `Province`：省份
   `City`：城市
   `DistrictCounty`：区县
   `Street`：街道

### 地理位置更新 
从资源和耗电方面的考虑，SDK不自动监听地理位置的变化，如果调用方有需要可调用`SetUpdateInterval`接口，设置更新时间间隔，SDK会按设定的时间间隔监听位置变化并通知上层。（如果应用对地理位置变化关注度不大，最好不要设置自动更新）

- 原型：

  ``` csharp
  public void SetUpdateInterval(uint interval)
  ```

- 参数：
  `interval`:更新间隔时间（单位：分钟）
  
### 获取与指定用户距离 
获取与指定用户距离之前，需要调用`GetCurrentLocation`成功获取自己的地理位置,指定的用户也调用`GetCurrentLocation`成功获取其地理位置。

- 原型：

  ```csharp
  public ErrorCode GetDistance(string userID);
  ```

- 参数：
  `userID`：用户ID
  
- 返回值：
  `ErrorCode`：错误码，详细描述见[错误码定义](#错误码定义)  

- 异步回调接口：

  ```csharp
  public interface LocationListen
  {
     void OnGetDistance(YIMEngine.ErrorCode errorcode, string userID, uint distance);
  }  
  ```

- 参数：
  `errorcode`：错误码
  `userID`: 用户ID
  `distance`: 距离（米）  
  
## 关系链管理
### 用户信息管理
#### 用户资料变更的通知
当好友的用户资料变更时会收到此通知，使用方根据需要决定是否重新获取资料变更好友的用户信息。

- 原型

  ``` csharp
  void OnUserInfoChangeNotify(string userID);
  ```

- 参数：
  `userID`：资料变更用户的用户ID 
  
#### 设置用户信息
设置用户的基本资料，昵称，性别，个性签名，地理位置等。

- 原型

  ``` csharp
  public ErrorCode SetUserProfileInfo (IMUserSettingInfo settingInfo)
  ```

- 参数：
  `settingInfo`：用户基本信息
  
- 备注：
  `IMUserSettingInfo`类成员如下：  
  `NickName`：string类型，昵称
  `Sex`：枚举类型，性别，IMUserSex.SEX_UNKNOWN=0  //未知性别
                      IMUserSex.SEX_MALE=1     //男性
                      IMUserSex.SEX_FEMALE=2   //女性
  `Signature`：string类型，个性签名
  `Country`：string类型，国家  
  `Province`：string类型，省份
  `City`：string类型，城市
  `ExtraInfo`：string类型，扩展信息，JSON格式   
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp  
  public interface UserProfileListen
  {  
     void OnSetUserInfo(YIMEngine.ErrorCode errorcode);
  }
  ```
  
- 参数：
  `errorcode`：错误码
  
#### 查询用户基本资料
查询用户的基本资料，昵称，性别，个性签名，头像，地理位置，被添加权限等。

- 原型

  ``` csharp
  public ErrorCode GetUserProfileInfo (string userID = "");
  ```

- 参数：
  `userID`：用户ID,可选参数(为空查询自己的信息)
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp  
  public interface UserProfileListen
  {  
     void OnQueryUserInfo(YIMEngine.ErrorCode errorcode, IMUserProfileInfo userInfo);
  }
  ```
  
- 参数：
  `errorcode`：错误码
  `userInfo`：用户资料
  
- 备注：
  `IMUserProfileInfo`继承于IMUserSettingInfo，类成员如下：  
  `UserID`：string类型，用户ID
  `PhotoURL`：string类型，头像url
  `OnlineState`：枚举类型，在线状态，
                 IMUserStatus.USER_STATUS_ONLINE=0   //在线，默认值（已登录）
                 IMUserStatus.USER_STATUS_OFFLINE=1  //离线                
  `BeAddPermission`：枚举类型，被好友添加权限，
                   IMUserBeAddPermission.NOT_ALLOW_ADD=0    //不允许被添加
                   IMUserBeAddPermission.NEED_VALIDATE=1    //需要验证
                   IMUserBeAddPermission.NO_ADD_PERMISSION=2 //允许被添加，不需要验证, 默认值  
  `FoundPermission`：枚举类型，被查找时是否显示被添加权限，
                    IMUserFoundPermission.CAN_BE_FOUND=0  //能被其它用户查找到，默认值
                    IMUserFoundPermission.CAN_NOT_BE_FOUND=1  //不能被其它用户查找到                    
    
#### 设置用户头像

- 原型

  ``` csharp
  public ErrorCode SetUserProfilePhoto (string photoPath);
  ```

- 参数：
  `photoPath`：本地图片绝对路径
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp  
  public interface UserProfileListen
  {  
     void OnSetPhotoUrl(YIMEngine.ErrorCode errorcode, string photoUrl);
  }
  ```
  
- 参数：
  `errorcode`：错误码
  `photoUrl`：图片下载路径  
  
#### 切换用户状态

- 原型

  ``` csharp
  public ErrorCode SwitchUserStatus (string userID, IMUserStatus userStatus);
  ```

- 参数：
  `userID`：用户ID
  `userStatus`：用户在线状态，枚举类型，
               IMUserStatus.USER_STATUS_ONLINE=0   //在线，默认值（已登录）
               IMUserStatus.USER_STATUS_OFFLINE=1  //离线
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp  
  public interface UserProfileListen
  {  
     void OnSwitchUserOnlineState(YIMEngine.ErrorCode errorcode);
  }
  ```
  
- 参数：
  `errorcode`：错误码 
  
#### 设置好友添加权限

- 原型

  ``` csharp
  public ErrorCode SetAddPermission (bool beFound, IMUserBeAddPermission beAddPermission)
  ```

- 参数：
  `displayPermission`：查找时是否显示自己的被添加权限，true-显示，false-不显示
  `addPermission`：被其它用户添加的权限，枚举类型，
                   IMUserBeAddPermission.NOT_ALLOW_ADD=0    //不允许被添加
                   IMUserBeAddPermission.NEED_VALIDATE=1    //需要验证
                   IMUserBeAddPermission.NO_ADD_PERMISSION=2 //允许被添加，不需要验证, 默认值  
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp 
  public interface UserProfileListen
  {   
     void OnSetUserInfo(YIMEngine.ErrorCode errorcode);
  }
  ```
  
- 参数：
  `errorcode`：错误码  
  
### 好友管理  
#### 查找好友
查找将要添加的好友，获取该好友的简要信息。

- 原型

  ``` csharp
  ErrorCode FindUser(int findType, string target)
  ```

- 参数：
  `findType`：查找类型，0-按ID查找，1-按昵称查找
  `target`：对应查找类型选择的用户ID或昵称
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
  void OnFindUser(YIMEngine.ErrorCode errorcode, List<UserBriefInfo> users);
  ```
  
- 参数：
  `errorcode`：错误码 
  `users`：用户简要信息列表 
  
- 备注：
  `UserBriefInfo`类成员如下：  
  `UserID`：string类型，用户ID
  `NickName`：string类型，用户昵称
  `Status`：枚举类型，在线状态，UserStatus:
                 STATUS_ONLINE=0   //在线
                 STATUS_OFFLINE=1  //离线
                 STATUS_INVISIBLE=2 //隐身     
       
#### 添加好友

- 原型

  ``` csharp
  ErrorCode RequestAddFriend(List<string> users, string comments)
  ```

- 参数：
  `users`：需要添加为好友的用户ID列表
  `comments`：备注或验证信息，长度最大128bytes
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
  void OnRequestAddFriend(YIMEngine.ErrorCode errorcode, string userID);
  ```
  
- 参数：
  `errorcode`：错误码 
  `userID`：用户ID
  
- 备注：
  被请求方会收到被添加为好友的通知，被请求方如果设置了需要验证才能被添加，会收到下面的回调：
  
  ``` csharp   
  void OnBeRequestAddFriendNotify(string userID, string comments);
  ```   
  
  - 参数：     
    `userID`：请求方的用户ID
    `comments`：备注或验证信息
    
  被请求方如果设置了不需要验证就能被添加，收到下面的回调：
  
  ``` csharp   
  void OnBeAddFriendNotify(string userID, string comments);
  ```   
  
  - 参数：     
    `userID`：请求方的用户ID
    `comments`：备注或验证信息  
  
#### 处理好友请求
当前用户有被其它用户请求添加为好友的请求时，处理添加请求。

- 原型

  ``` csharp
  ErrorCode DealAddFriend(string userID, int dealResult,ulong reqID)
  ```

- 参数：
  `userID`：请求方的用户ID
  `dealResult`：处理结果，0-同意，1-拒绝
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
 void OnDealBeRequestAddFriend(YIMEngine.ErrorCode errorcode, string userID, string comments, int dealResult, ulong reqID);
  ```
  
- 参数：
  `errorcode`：错误码 
  `userID`：请求方的用户ID
  `comments`：备注或验证信息
  `dealResult`：处理结果，0-同意，1-拒绝 
  
- 备注：
 
   如果被请求方设置了需要验证才能被添加为好友，在被请求方成功处理了请求方的好友请求后，请求方能收到添加好友请求结果的通知，收到下面的回调：
   
  ``` csharp   
  void OnRequestAddFriendResultNotify(string userID, string comments, int dealResult);
  ```
  
- 参数：   
  `userID`：被请求方的用户ID
  `comments`：备注或验证信息
  `dealResult`：处理结果，0-同意，1-拒绝  
    
#### 删除好友

- 原型

  ``` csharp
  ErrorCode DeleteFriend(List<string> users, int deleteType = 1)
  ```

- 参数：
  `users`：需删除的好友用户ID列表(NSArray<NSString*>)
  `deleteType`：删除类型，0-双向删除，1-单向删除(删除方在被删除方好友列表依然存在)
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
  void OnDeleteFriend(YIMEngine.ErrorCode errorcode, string userID);
  ```
  
- 参数：
  `errorcode`：错误码 
  `userID`：被删除好友的用户ID
  
- 备注：
 
   如果删除方采用的是双向删除，被删除方能收到其被好友删除的通知，收到下面的回调：
   
  ``` csharp   
  void OnBeDeleteFriendNotify(string userID);
  ```
  
- 参数：   
  `userID`：删除方的用户ID   
  
#### 拉黑好友

- 原型

  ``` csharp
  ErrorCode BlackFriend(int type, List<string> users)
  ```

- 参数：
  `type`：拉黑类型，0-拉黑，1-解除拉黑
  `users`：需拉黑的好友用户ID列表(NSArray<NSString*>)
  
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
  void OnBlackFriend(YIMEngine.ErrorCode errorcode, int type, string userID);
  ```
  
- 参数：
  `errorcode`：错误码 
  `type`：拉黑类型，0-拉黑，1-解除拉黑
  `userID`：被拉黑方用户ID    
  
#### 查询好友列表
查询当前用户已添加的好友，也能查找被拉黑的好友。

- 原型

  ``` csharp
  ErrorCode QueryFriends(int type = 0, int startIndex = 0, int count = 50)
  ```

- 参数：
  `type`：查找类型，0-正常状态的好友，1-被拉黑的好友
  `startIndex`：起始序号，作用为分批查询好友（例如：第一次从序号0开始查询30条，第二次就从序号30开始查询相应的count数）
  `count`：数量（一次最大100）  
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
  void OnQueryFriends(YIMEngine.ErrorCode errorcode, int type, int startIndex, List<UserBriefInfo> friends);
  ```
  
- 参数：
  `errorcode`：错误码 
  `type`：查找类型，0-正常状态的好友，1-被拉黑的好友
  `startIndex`：起始序号
  `friends`：好友列表 
  
- 备注：
  `UserBriefInfo`类成员如下：  
  `UserID`：string类型，用户ID
  `NickName`：string类型，用户昵称
  `Status`：枚举类型，在线状态，UserStatus:
                 STATUS_ONLINE=0   //在线
                 STATUS_OFFLINE=1  //离线
                 STATUS_INVISIBLE=2 //隐身    
  
#### 查询好友请求列表
查询当前用户收到的被添加请求的好友列表。

- 原型

  ``` csharp
  ErrorCode QueryFriendRequestList(int startIndex = 0, int count = 20)
  ```

- 参数：  
  `startIndex`：起始序号，作用为分批查询好友请求（例如：第一次从序号0开始查询10条，第二次就从序号10开始查询相应的count数）
  `count`：数量（一次最大20）  
  
- 返回：
  错误码    
    
- 异步回调接口：

  ``` csharp   
  void OnQueryFriendRequestList(YIMEngine.ErrorCode errorcode, int startIndex, List<FriendRequestInfo> requestList);
  ```
  
- 参数：
  `errorcode`：错误码 
  `startIndex`：起始序号
  `requestList`：好友请求列表
  
- 备注：
  `FriendRequestInfo`类成员如下：  
  `AskerID`：string类型，请求方ID
  `AskerNickname`：string类型，请求方昵称
  `InviteeID`：string类型，受邀方ID
  `InviteeNickname`：string类型，受邀方昵称
  `ValidateInfo`：NSString类型，验证信息
  `Status`：枚举类型，好友请求状态，AddFriendStatus:
                 STATUS_ADD_SUCCESS=0         //添加完成
                 STATUS_ADD_FAILED=1          //添加失败  
                 STATUS_WAIT_OTHER_VALIDATE=2 //等待对方验证 
                 STATUS_WAIT_ME_VALIDATE=3    //等待我验证
  `createTime`：unsigned int类型，请求的发送或接收时间  

## 短连接模式
### 设置短连接模式
设置短连接模式，仅支持语音识别
- 接口

  ``` csharp   
  public void SetShortConnectionMode();
  ```

- 备注：
  设置短连接模式后，发送其他类型消息将被阻断，仅支持语音识别。logout后，短连接恢复正常。
   
## 服务器部署地区定义 

``` csharp
ServerZone.China = 0         // 中国
ServerZone.Singapore = 1     // 新加坡  
ServerZone.America = 2 	   // 美国
ServerZone.HongKong = 3 	   // 香港
ServerZone.Korea = 4 		   // 韩国
ServerZone.Australia = 5     // 澳洲
ServerZone.Deutschland = 6	// 德国
ServerZone.Brazil = 7  		// 巴西
ServerZone.India = 8  		   // 印度
ServerZone.Japan = 9		   // 日本
ServerZone.Ireland = 10	   // 爱尔兰
ServerZone.ServerZone_Unknow = 9999
```

## 错误码定义
| 错误码                                               | 含义         |
| -------------                                       | -------------|
| ErrorCode.Success = 0                        | 成功|
| ErrorCode.EngineNotInit = 1                  | IM SDK未初始化 |
| ErrorCode.NotLogin = 2                       | IM SDK未登录 |
| ErrorCode.ParamInvalid = 3                   | 无效的参数 |
| ErrorCode.TimeOut = 4                        | 超时 |
| ErrorCode.StatusError = 5                    | 状态错误 |
| ErrorCode.SDKInvalid = 6                     | Appkey无效 |
| ErrorCode.AlreadyLogin = 7                   | 已经登录 |
| ErrorCode.LoginInvalid = 1001                | 登录无效 |
| ErrorCode.ServerError = 8                    | 服务器错误 |
| ErrorCode.NetError = 9                       | 网络错误 |
| ErrorCode.LoginSessionError = 10             | 登录状态出错 |
| ErrorCode.NotStartUp = 11                    | SDK未启动 |
| ErrorCode.FileNotExist = 12                  | 文件不存在 |
| ErrorCode.SendFileError = 13                 | 文件发送出错 |
| ErrorCode.UploadFailed = 14                  | 文件上传失败,上传失败 一般都是网络限制上传了 |
| ErrorCode.UsernamePasswordError = 15          | 用户名密码和最后一次登录成功的不匹配 |
| ErrorCode.UserStatusError = 16                | 状态错误，一般是非登录状态调用了需要登录后才能操作的接口 |
| ErrorCode.MessageTooLong = 17                 | 消息太长，主控制在2k以内 |
| ErrorCode.ReceiverTooLong = 18                | 接收方ID过长(检查频道名) |
| ErrorCode.InvalidChatType = 19                | 无效聊天类型(私聊、聊天室) |
| ErrorCode.InvalidReceiver = 20                | 无效用户ID（私聊接受者为数字格式ID）|
| ErrorCode.UnknowError = 21                    | 请求错误，常见的情况是指定的接收者不存在 |
| ErrorCode.InvalidAppkey = 22                  | APPKEY校验不通过 |
| ErrorCode.ForbiddenSpeak = 23                 | 被禁言 |
| ErrorCode.CreateFileFailed = 24               | 创建文件失败 |
| ErrorCode.UnsupportFormat = 25                | 不支持的文件格式 |
| ErrorCode.ReceiverEmpty = 26                  | 接收方为空 |
| ErrorCode.RoomIDTooLong = 27                  | 频道名太长，不要超过128字节 
| ErrorCode.ContentInvalid = 28,                | 聊天内容严重非法|
| ErrorCode.NoLocationAuthrize = 29,            | 未打开定位权限|
| ErrorCode.UnknowLocation = 30,                | 未知位置|
| ErrorCode.Unsupport = 31,                     | 不支持该接口|
| ErrorCode.NoAudioDevice = 32,                 | 无音频设备|
| ErrorCode.AudioDriver = 33,                   | 音频驱动问题|
| ErrorCode.DeviceStatusInvalid = 34,           | 设备状态错误|
| ErrorCode.ResolveFileError = 35,              | 文件解析错误|
| ErrorCode.ReadWriteFileError = 36,            | 文件读写错误|
| ErrorCode.NoLangCode = 37,                    | 语言编码错误|
| ErrorCode.TranslateUnable = 38,               | 翻译接口不可用|
| ErrorCode.SpeechAccentInvalid = 39,           | 语音识别方言无效|
| ErrorCode.SpeechLanguageInvalid = 40,         | 语音识别语言无效|
| ErrorCode.HasIllegalText = 41,                | 消息含非法字符|
| ErrorCode.AdvertisementMessage = 42,          | 消息涉嫌广告|
| ErrorCode.AlreadyBlock = 43,                  | 用户已经被屏蔽|
| ErrorCode.NotBlock = 44,                      | 用户未被屏蔽|
| Errorcode.MessageBlocked = 45,                | 消息被屏蔽|
| Errorcode.LocationTimeout = 46,               | 定位超时|
| Errorcode.NotJoinRoom = 47,                   | 未加入该房间|
| Errorcode.LoginTokenInvalid = 48,             | 登录token错误|
| Errorcode.CreateDirectoryFailed = 49,         | 创建目录失败|
| Errorcode.InitFailed = 50,	                   | 初始化失败|
| Errorcode.Disconnect = 51,	                   | 与服务器断开|
| Errorcode.TheSameParam = 52,                  | 设置参数相同|
| Errorcode.QueryUserInfoFail = 53,             | 查询用户信息失败|
| Errorcode.SetUserInfoFail = 54,               | 设置用户信息失败|
| Errorcode.UpdateUserOnlineStateFail = 55,     | 更新用户在线状态失败|
| Errorcode.NickNameTooLong = 56,               | 昵称太长(> 64 bytes)|
| Errorcode.SignatureTooLong = 57,              | 个性签名太长(> 120 bytes)|
| Errorcode.NeedFriendVerify = 58,              | 需要好友验证信息|
| Errorcode.BeRefuse = 59,	                   | 添加好友被拒绝|
| Errorcode.HasNotRegisterUserInfo = 60,        | 未注册用户信息|
| Errorcode.AlreadyFriend = 61,                 | 已经是好友|
| Errorcode.NotFriend = 62,	                   | 非好友|
| Errorcode.NotBlack = 63,	                   | 不在黑名单中|
| Errorcode.PhotoUrlTooLong = 64,               | 头像url过长(>500 bytes)|
| Errorcode.PhotoSizeTooLarge = 65,             | 头像太大（>100 kb）|
| Errorcode.ChannelMemberOverflow = 66,         | 达到频道人数上限|
| Errorcode.ALREADYFRIENDS = 1000,              | 服务器错误|
| Errorcode.LoginInvalid = 1001,                | 服务器错误登录无效|
| ErrorCode.PTT_Start = 2000                    | 开始录音 |
| ErrorCode.PTT_Fail = 2001                     | 录音失败 |
| ErrorCode.PTT_DownloadFail = 2002             | 语音消息文件下载失败 |
| ErrorCode.PTT_GetUploadTokenFail = 2003       | 获取语音消息Token失败 |
| ErrorCode.PTT_UploadFail = 2004               | 语音消息文件上传失败 |
| ErrorCode.PTT_NotSpeech = 2005                | 没有录音内容 |
| ErrorCode.PTT_DeviceStatusError = 2006        | 音频设备状态错误 |
| ErrorCode.PTT_IsSpeeching = 2007              | 已经在录音中 |
| ErrorCode.PTT_FileNotExist = 2008             | 找不到录音生成的文件 |
| ErrorCode.PTT_ReachMaxDuration = 2009         | 达到语音最大时长限制 |
| ErrorCode.PTT_SpeechTooShort = 2010           | 语音时长太短 |
| ErrorCode.PTT_StartAudioRecordFailed = 2011   | 启动语音失败 |
| ErrorCode.PTT_SpeechTimeout = 2012            | 音频输入超时 |
| ErrorCode.PTT_IsPlaying = 2013                | 正在播放 |
| ErrorCode.PTT_NotStartPlay = 2014             | 未开始播放 |
| ErrorCode.PTT_CancelPlay = 2015               | 主动取消播放 |
| ErrorCode.PTT_NotStartRecord = 2016,          | 未开始语音|
| ErrorCode.PTT_NotInit = 2017,                 | 未初始化|
| ErrorCode.PTT_InitFailed = 2018,              | 初始化失败|
| ErrorCode.PTT_Authorize = 2019,               | 录音权限|
| ErrorCode.PTT_StartRecordFailed = 2020,       | 启动录音失败|
| ErrorCode.PTT_StopRecordFailed = 2021,        | 停止录音失败|
| ErrorCode.PTT_UnsupprtFormat = 2022,          | 不支持的格式|
| ErrorCode.PTT_ResolveFileError = 2023,        | 解析文件错误|
| ErrorCode.PTT_ReadWriteFileError = 2024,      | 读写文件错误|
| ErrorCode.PTT_ConvertFileFailed = 2025,       | 文件转换失败|
| ErrorCode.PTT_NoAudioDevice = 2026,           | 无音频设备|
| ErrorCode.PTT_NoDriver = 2027,                | 驱动问题|
| ErrorCode.PTT_StartPlayFailed = 2028,         | 启动播放失败|
| ErrorCode.PTT_StopPlayFailed = 2029,          | 停止播放失败|
| Errorcode.PTT_RecognizeFailed = 2030,         | 识别失败|
| Errorcode.PTT_ShortConnectionMode = 2031,     | 短连接模式，不支持发送|
| ErrorCode.Fail = 10000                        | 语音服务启动失败|




