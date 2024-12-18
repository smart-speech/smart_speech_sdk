# smart_speech_sdk

智言语音评测Flutter sdk ，暂时只支持安卓和ios 两个平台

合作联系人：18519378559

合作  邮箱：  xuhaitao@samrt-speech.com 

官方 网站：https://smart-speech.com/

## 一、集成步骤

**支持平台：**

* Android API Level >=19 （Android 4.4及以上）
* iOS 12 及以上版本

**授权账号（需要商务提供）：**

* appId
* appSecret

### 1. 在项目中引入SDK

~~~
# pubspec.yaml
  dependencies:
    smart_speech_sdk:
        path: ***    # 此处填写Flutter sdk在磁盘上的存放路径
    
    logger: ^1.0.0   # 日志打印工具类
~~~



### 2. 调用步骤/示例代码

![](https://smart-speech.com/res/docs/android_flowchart.svg)

<!--  https://mermaid.live/                    -->
<!-- graph LR                                  -->
<!-- A(创建评测类对象)--\>B("设置公共参数")         -->
<!-- B--\>C("初始化引擎init()")                   -->
<!-- C--\>D("设置题型相关参数")                    -->
<!-- D--调用SDK录音--\>E1("start(recorder)")     -->
<!-- E1--\>E2("stop()")                         -->
<!-- E2--\>E4(onResult获取结果)                  -->
<!-- E1--\>E3("cancel()")                       -->
<!-- E3--\>G(取消评测 无结果)                     -->
<!-- D--传音频文件--\>F1("start(wavPath)")        -->
<!-- F1--\>E3                                   --> 
<!-- F1--发送完成后自动结束--\>F4(onResult获取结果) --> 
<!-- G--\>Z("destroy()")                        -->
<!-- F4--\>Z                                    --> 
<!-- E4--\>Z                                    --> 

#### ① 获取录音权限

```java
使用前，flutter 代码需要获取录音权限，否则录音会失败
```

#### ② 初始化引擎实例

```dart
import 'package:smart_speech_sdk/smart_speech_sdk.dart';  
import 'package:logger/logger.dart';

/// demo中写的一个全局单例，可以根据自己的业务进行处理
class GlobalEngine{
  static SmartSpeechSdk? _speechEvalSdkPlugin;
  static final logger  = Logger();
  static void loggerd(String msg){
    logger.d(TAG,msg);
  }

  static void loggere(String msg){
    logger.e(TAG,msg);
  }

  static void loggeri(String msg){
    logger.i(TAG,msg);
  }

  static void loggerw(String msg){
    logger.w(TAG,msg);
  }

  static void loggerv(String msg){
    logger.v(TAG,msg);
  }
}  

/// 创建引擎实例，并初使化
Future<void> createEngine() async {
    Map cfg = {
      "appId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxx",  // appId 
      "appSecret": "xxxxxxxxxxxx",                //app Secret 
      "sampleRate":"16000",  //固定值
      //
      //  None,  不支持
      //  OralOnLine, 配置在线
      //  OralTypeMixed;  配置混合模式，有网络时使用在线，无网络或者网络不好时，使用离线
      //
      "en": InitSettingOnline.OralOnLine.name,   //英文语种初使化配置，如果没有可以设置为None
      "zh": InitSettingOnline.OralOnLine.name,   //中文语种初使化配置，如果没有可以设置为None
      "userId":"111",  //用户的userId， 建议设置, 以方便区分不同的用户
      "serverUrl":"",  //服务器域名，通常不用设置
      "i18n":I18n.ZH.name,  // 国际化提示，默认提示中文
    };

    GlobalEngine._speechEvalSdkPlugin = SmartSpeechSdk.create(jsonEncode(cfg),RecordScoreCallBack(engineInitState: (String code,String msg){
      GlobalEngine.loggere("=========cfg:::code:$code===msg:$msg");
      if(code == "00000"){
        //创建引擎成功
        Fluttertoast.showToast(msg: "引擎初使化成功");
      }else {
        //创建引擎失败
      }
    },onWsStart: (String taskId){
      //在线评测ws建立成功时回调
    GlobalEngine?.loggeri("onWsStart=====taskId:$taskId");
    },onStartRecording: (){
      //录音开始回调，不会等ws连接成功
      GlobalEngine?.loggeri("onStartRecording=================");
    },onStopSending: (){
      //录音停止时回调
      GlobalEngine?.loggeri("onStopSending=====");
    },onGetVolume: (int volume){
      //实时返回音量信息回调
      GlobalEngine?.loggeri("onGetVolume=====volume:$volume");
    },onGetAudio: (Uint8List bytes){
      //录音方式 返回实时录音的音频
      GlobalEngine?.loggeri("onGetAudio=====bytes:$bytes");
    },onSaveAudioFile: (String localPath){
      // 设置 setSaveAudio(true) 时，此方法会回调
      GlobalEngine?.loggeri("onSaveAudioFile=====localPath:$localPath");
    },onWarning: (String taskId,String code,String msg){
      //提醒、警告信息，根据自己业务端需求进行处理
      GlobalEngine?.loggerw("onWarning=====taskId:$taskId==code:$code==msg:$msg");
    },onError: (String taskId,String code,String msg){
      ////评测错误回调
      GlobalEngine?.loggere("onError=====taskId:$taskId==code:$code==msg:$msg");
    },onRealtimeResult: (String result){
      //realtime设置为true时，中/英文 sentence、chapter、freedom 题型，以及中文 poem、recite 题型 ，
   	 //会实时返回评测数据
      GlobalEngine?.loggeri("onRealtimeResult=====result:$result");
    },onResult: (String result,bool online){
      //评测返回结果回调，online=true为在线返回的结果，false是离线返回的结果
      GlobalEngine?.loggeri("onResult====online:$online=result:$result");
    }));
}

```

初始化状态回调方法 engineInitState 对应状态码：

~~~
在线
"00000","success"
"13010", "缺少网络权限"
"13011", "无网络连接"
"10002", "参数错误"
"11011", "signature错误"
"11012", "appid不可用"
"11013", "appid不存在"
"11014", "生成token错误"
"11015", "非法的当前时间"
~~~

#### ③ 设置参数

##### 评测参数

- 使用 `eval.setKEY(VALUE);` 的方法设置评测参数
- 必填参数如缺失或赋值超出范围，将在 `onError` 中返回错误码、错误信息
- 选填参数如缺失，将取默认值；如赋值超出范围，将取默认值，并在 `onWarning` 中返回错误码、错误信息

##### 示例代码

```dart
void engineStart(){
    if(speechText.isNotEmpty){
      GlobalEngine._speechEvalSdkPlugin?.setAudioUrl(true);     //设置保存音标（服务器）
      GlobalEngine._speechEvalSdkPlugin?.setSaveAudio(true);   //设置保存音频（本地磁盘）
      GlobalEngine._speechEvalSdkPlugin?.setLangType(widget.langType == "en"? LangType.enUS: LangType.zhHans);       // 设置评测的语种
      GlobalEngine._speechEvalSdkPlugin?.setOnline(true);     // 设置在线评测
      GlobalEngine._speechEvalSdkPlugin?.setUserId("10001");      //用户userId， 建议设置, 以方便区分不同的用户
      GlobalEngine._speechEvalSdkPlugin?.setClientData("1111");  //设置自定义参数，客户可以传入自己需要的参数，会在响应结果中返回clentData字段，其它类型建议转为字符串类型
      Map paramsJson ={
        "mode": "word",
        "refText":"good morning."
      };
      GlobalEngine.loggerd("paramsjson:$paramsJson");
      GlobalEngine._speechEvalSdkPlugin?.setParamsJson(jsonEncode(paramsJson));
      GlobalEngine._speechEvalSdkPlugin?.startRecorder();
  }
```

#### ④ 开始/停止评测

**开始评测（录音方式）**

```java
 GlobalEngine._speechEvalSdkPlugin?.stop();
```

**开始评测（发送音频文件方式）**

```java
//开始
 String wavePath = "/sdcard/xxx/abc.wav"
 GlobalEngine._speechEvalSdkPlugin?.startFile(wavePath);
//传入WAV或PCM格式音频数据，要求为16000Hz、16Bit、单声道音频文件
//自动读取音频文件并发送，完成后自动停止评测
```

#### ⑤ **处理评测结果**

参考 ② ， 在 listener 中处理评测结果

#### ⑥ **释放评测资源**

```java
//一般可以放在Activity.onDestroy中调用
GlobalEngine._speechEvalSdkPlugin?.destroy();
```

### 3.  混淆机制

```java
//安卓项目中，如果要做混淆，排除SDK做混淆即可
-keep class com.zhiyan.**{*;}
```

## 二、接口文档

### SmartSpeechSdk

发音评测主类

**create**

| 函数原型 | static SmartSpeechSdk create(String cfg, RecordScoreCallBack initEngineCallBack) |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 创建评测类对象                                               |
| 输入参数 | cfg: 引擎创建前需要配置的基本参数<br>listener: 引擎创建评测过程中的监听事件 |
| 返回值   | 无                                                           |
| 使用说明 | 创建评测类对象                                               |

**setUserId**

| 函数原型 | Future<void> setUserId(String userId)                        |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置用户的uid                                                |
| 输入参数 | userId：用户uid或者用户的唯一标识                            |
| 返回值   | 无                                                           |
| 使用说明 | 产品端的用户标识，可以是自定义的自符串，建议一个用户一个userid,方便排查问题 |

**setLangType**

| 函数原型 | Future<void> setLangType(LangType langType) |
| -------- | ------------------------------------------- |
| 功能描述 | 设置评测的语种                              |
| 输入参数 | langType: 语种代码                          |
| 返回值   | 无                                          |
| 使用说明 | 设置语种                                    |

**setMini**

| 函数原型 | Future<void> setMini(int mini)                               |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置是否返回少量结果 数据                                    |
| 输入参数 | mini: 是否返回全量数据，默认为0，返回全量数据，设置为1时返回总分、流利度、完整度、准确度等少量数据 |
| 返回值   | 无                                                           |
| 使用说明 | 如果在一些小系统中，为了不影响设备内存，可以设置为1          |

**setScale**

| 函数原型 | Future<void> setScale(int scale) |
| -------- | -------------------------------- |
| 功能描述 | 设置评分分制                     |
| 输入参数 | scale: 评分分制 1-100            |
| 返回值   | 无                               |
| 使用说明 | 设置评分分制                     |

**setSampleRate**

| 函数原型 | Future<void> setSampleRate(int sampleRate) |
| -------- | ------------------------------------------ |
| 功能描述 | 设置采样率                                 |
| 输入参数 | sampleRate: 采样率 16000                   |
| 返回值   | 无                                         |
| 使用说明 | 设置采样率                                 |

**setLooseness**

| 函数原型 | Future<void>  setLooseness(int looseness) |
| -------- | ----------------------------------------- |
| 功能描述 | 设置评分宽松度                            |
| 输入参数 | looseness: 评分宽松度 0-9                 |
| 返回值   | 无                                        |
| 使用说明 | 设置评分宽松度                            |

**setConnectTimeout**

| 函数原型 | Future<void>  setConnectTimeout(int timeout) |
| -------- | -------------------------------------------- |
| 功能描述 | 设置连接超时时间                             |
| 输入参数 | timeout: 超时时间 1-60                       |
| 返回值   | 无                                           |
| 使用说明 | 设置连接超时时间                             |

**setResponseTimeout**

| 函数原型 | Future<void>  setResponseTimeout(int timeout) |
| -------- | --------------------------------------------- |
| 功能描述 | 设置响应超时时间                              |
| 输入参数 | timeout: 超时时间 1-60                        |
| 返回值   | 无                                            |
| 使用说明 | 设置响应超时时间                              |

**setRatio**

| 函数原型 | Future<void>  setRatio(float ratio)                          |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置评分调节系数                                             |
| 输入参数 | ratio: 评分调节系数 0.8-1.5                                  |
| 返回值   | 无                                                           |
| 使用说明 | 该系数的作用是对 30~90 之间的得分（百分制）进行上调或下调，其他范围内的得分不变<br/>`ratio` >1.0 时，对评分进行上调，越接近75分，调节程度越高<br/>`ratio` <1.0 时，对评分进行下调，越接近75分，调节程度越高<br/>`ratio` =1.0 时，不进行评分调节 |

**setRealtime**

| 函数原型         | Future<void>  setRealtime(boolean realtime)                  |
| ---------------- | ------------------------------------------------------------ |
| 功能描述功能描述 | 设置是否实时返回评测结果                                     |
| 输入参数输入参数 | realtime: true 评测中会返回结果，<br />realtime: false 评测中不会返回实时结果 ，评测完成后才会有结果 |
| 返回值           | 无                                                           |
| 使用说明         | 如果需要实时获取朗读的结果 ，可以设置为true 。仅支持中/英文 sentence、chapter、freedom、recite 题型，以及中文 poem 题型 |

**setAudioUrl**

| 函数原型 | Future<void>  setAudioUrl(boolean audioUrl)                  |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置是否保存录音文件                                         |
| 输入参数 | audioUrl: true 时，服务端会保存音频数据，<br />audioUrl: false 时，服务端不会保存音频数据 |
| 返回值   | 无                                                           |
| 使用说明 | 设置是否保存录音音频（服务端）                               |

**setSaveAudio**

| 函数原型 | Future<void>  setSaveAudio(boolean saveAudio)          |
| -------- | ------------------------------------------------------ |
| 功能描述 | 设置是否保存录音音频                                   |
| 输入参数 | saveAudio: true 为保存本地音频，false 为不保存本地音频 |
| 返回值   | 无                                                     |
| 使用说明 | 设置是否保存录音音频（客户端sdk）                      |

**setMaxPrefixSilenceMs**

| 函数原型 | Future<void> setMaxPrefixSilence(int maxPrefixSilenceMs)     |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置语音前置静音检测阈值（毫秒）                             |
| 输入参数 | maxPrefixSilenceMs: 范围 [0-30000]                           |
| 返回值   | 无                                                           |
| 使用说明 | 音开始后静音时长超过该阈值会返回一次Warning事件，为0时代表不进行句首静音检测，为0时不返回Warning事件 |

**setMaxSuffixSilenceMs**

| 函数原型 | Future<void> setMaxSuffixSilence(int maxSuffixSilenceMs)     |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置语音后置静音检测阈值（毫秒）                             |
| 输入参数 | maxSuffixSilenceMs: 范围 [0-30000]                           |
| 返回值   | 无                                                           |
| 使用说明 | 句尾静音时长超过该阈值会自动结束评测，为0时代表不进行句尾静音检测 |

**setPrecision**

| 函数原型 | Future<void> setPrecision(String precision)                  |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置打分精度                                                 |
| 输入参数 | precision: 默认为0，只支持0、0.1、0.25、0.5、1，如果设置的值不是这五个中的一个，则按0处理 |
| 返回值   | 无                                                           |
| 使用说明 | 根据业务需要的打分精度进行设置                               |

**setConnti**

| 函数原型 | Future<void>  setConnti(int connti)                          |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 是否开启连读和失爆检测，默认为0，只针对英文                  |
| 输入参数 | connti为0时，不开启<br/>connti为1时， 开启                   |
| 返回值   | 无                                                           |
| 使用说明 | 默认为0，无此功能可不设置，或者设置为0，设置为1时会开启连读和失去爆检测 |

**setServerAPI**

| 函数原型 | Future<void>  setServerAPI(String serverAPI) |
| -------- | -------------------------------------------- |
| 功能描述 | 设置在线服务器地址，请求指定域名服务器       |
| 输入参数 | serverAPI ，提供的域名                       |
| 返回值   | 无                                           |
| 使用说明 | 需要在初使化前设置，不设置使用通用域名       |

**setTestServerAPI**

| 函数原型 | Future<void>  setTestServerAPI(String testServerAPI) |
| -------- | ---------------------------------------------------- |
| 功能描述 | 设置在线测试服务器地址，请求指定域名服务器           |
| 输入参数 | testServerAPI ，提供的域名                           |
| 返回值   | 无                                                   |
| 使用说明 | 需要在初使化前设置，不设置使用通用域名               |

**setI18n**

| 函数原型 | Future<void>  setI18n(SpeechEval.I18n i18n)                  |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置国际化返回语言类型，根据设置的语言类型返回对应的提示信息（warning和error) |
| 输入参数 | i18n ，语言类型参数 ，支持枚举EN、ZH、None  ,  EN: 英文   ， ZH:中文 ，  None: sdk 会取系统语言 |
| 返回值   | 无                                                           |
| 使用说明 | 需要在初使化前设置，不设置此方法或者设置为None，则取系统语言，系统语言为中文返回中文提示信息，其它语言类型返回英文提示信息 |

**setParamsJson**

| 函数原型 | Future<void>  setParamsJson(JSONObject json)                 |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置题型参数对象，每个题型的参数对象都不一样，详情请查看题型文档 |
| 输入参数 | 无                                                           |
| 返回值   | 无                                                           |
| 使用说明 | 用于设置 JSON 参数                                           |

**setClientData**

| 函数原型 | Future<void>  setClientData(String clientData)               |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 设置自定义参数，客户可以传入自己需要的参数，会在响应结果中返回clentData字段 |
| 输入参数 | 无                                                           |
| 返回值   | 无                                                           |
| 使用说明 | 用于设置自定义请求参数，只支持字符串类型，如果想传入其它类型，建议转成字符串类型传入。 |

**startRecorder**

| 函数原型 | Future<void> startRecorder() |
| -------- | ---------------------------- |
| 功能描述 | 开始评测（使用 SDK 录音）    |
| 返回值   | 错误码对象                   |
| 使用说明 | 开始评测（使用 SDK 录音）    |

**startFile**

| 函数原型 | Future<void> startFile(String wavPath)                       |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 开始评测（传音频文件）                                       |
| 输入参数 | wavPath: 音频文件路径                                        |
| 使用说明 | 开始评测（手动发数据），开始后自动发送音频，发送完成后自动结束完成评测 |

**start**

| 函数原型 | Future<void> start()                                    |
| -------- | ------------------------------------------------------- |
| 功能描述 | 开始评测调用这个方法后，通过feed 方法完成音频数据的发送 |
| 使用说明 | 开始评测（手动发数据），需要结合feed 方法一起使用       |

**feed**

| 函数原型 | Future<void> feed(Uint8List bytes, int length)   |
| -------- | ------------------------------------------------ |
| 功能描述 | 调用start方法后，通过feed 方法完成音频数据的发送 |
| 输入参数 | bytes：音频数据<br />length: 音频数据长度        |
| 使用说明 | 由客户实现录音功能和音频数据的发送               |

**stop**

| 函数原型 | Future<void> stop() |
| -------- | ------------------- |
| 功能描述 | 停止录音            |
| 输入参数 | 无                  |
| 返回值   | 无                  |
| 使用说明 | 停止录音            |


**cancel**

| 函数原型 | Future<void> cancel()              |
| -------- | ---------------------------------- |
| 功能描述 | 开始评测后取消评测                 |
| 输入参数 | 无                                 |
| 返回值   | 无                                 |
| 使用说明 | 开始评测后取消评测，不返回评测结果 |

**destroy**

| 函数原型 | Future<void> destroy()                  |
| -------- | --------------------------------------- |
| 功能描述 | 释放评测资源                            |
| 输入参数 | 无                                      |
| 返回值   | 无                                      |
| 使用说明 | 一般可以放在 引擎不再使用的时候进行销毁 |

**getSDKVersion**

| 函数原型 | Future<String> getSdkVersion()            |
| -------- | ----------------------------------------- |
| 功能描述 | 获取使用的sdk 版本号                      |
| 输入参数 | 无                                        |
| 返回值   | 版本号                                    |
| 使用说明 | 客户端可以获取sdk版本号，可以方便排查问题 |

**getDeviceId**

| 函数原型 | Future<String> getDeviceId()                                 |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 获取设备唯一id ，安卓获取的是AndroidId , iOS获取的是getDeviceIDInKeychain |
| 输入参数 | 无                                                           |
| 返回值   | 返回设备的唯一id                                             |
| 使用说明 | 客户端可以获取sdk版本号，可以方便排查问题                    |

**getPlatformOS**

| 函数原型 | Future<String> getPlatformOS()                               |
| -------- | ------------------------------------------------------------ |
| 功能描述 | 获取平台类型（Android或者iOS）                               |
| 输入参数 | 无                                                           |
| 返回值   | 返回Android 或者iOS                                          |
| 使用说明 | Flutter 可能会根据 sdk 平台去处理一些逻辑的时候，可以通过此方法进行获取 |

### SmartSpeechSdk.LangType

发音评测语种

| 成员   | 含义 |
| ------ | ---- |
| enUS   | 英语 |
| zhHans | 中文 |



### SmartSpeechSdk.I18n

| 成员 | 含义                         |
| ---- | ---------------------------- |
| ZH   | 警告信息和错误信息以中文返回 |
| EN   | 警告信息和错误信息以英文返回 |



### **SmartSpeechSdk.InitSettingOnline**

| 成员          | 含义                 |
| ------------- | -------------------- |
| OralOnline    | 初使化时设置在线     |
| OralTypeMixed | 初使化时设置混合模式 |



### SmartSpeechSdk.RecordScoreCallBack

发音评测回调接口

**onResult**

| 函数原型               | void onResult(String result) |
| ---------------------- | ---------------------------- |
| 功能描述               | 评测结果回调方法             |
| 输入参数（回调返回值） | result 评测结果              |
| 返回值                 | 无                           |
| 使用说明               | 评测结果回调方法             |

**onStartRecording**

| 函数原型               | void onStartRecording(String taskId) |
| ---------------------- | ------------------------------------ |
| 功能描述               | 开始录音回调方法                     |
| 输入参数（回调返回值） | 无                                   |
| 返回值                 | 无                                   |
| 使用说明               | 开始录音回调方法                     |


**onStopSending**

| 函数原型               | void onStopSending() |
| ---------------------- | -------------------- |
| 功能描述               | 评测结束回调方法     |
| 输入参数（回调返回值） | 无                   |
| 返回值                 | 无                   |
| 使用说明               | 评测结束回调方法     |


**onGetAudio**

| 函数原型               | void onGetAudio(byte[] data)             |
| ---------------------- | ---------------------------------------- |
| 功能描述               | 实时返回录音方式调用评测时的音频         |
| 输入参数（回调返回值） | data: 音频数据                           |
| 返回值                 | 无                                       |
| 使用说明               | 使用录音方式进行评测时，实时返回音频数据 |

**onGetVolume**

| 函数原型               | void onGetVolume(double volume)          |
| ---------------------- | ---------------------------------------- |
| 功能描述               | 实时返回录音方式调用评测时的音量         |
| 输入参数（回调返回值） | volume: 音量大小                         |
| 返回值                 | 无                                       |
| 使用说明               | 使用录音方式进行评测时，实时返回音量大小 |

**onError**

| 函数原型               | void onError(String taskId,String code, String msg) |
| ---------------------- | --------------------------------------------------- |
| 功能描述               | 评测错误时的回调                                    |
| 输入参数（回调返回值） | code: 错误码<br>msg: 错误信息                       |
| 返回值                 | 无                                                  |
| 使用说明               | 评测错误时的回调，onResult 无结果                   |

**onWarning**

| 函数原型               | void onWarning(String taskId,String code, String msg) |
| ---------------------- | ----------------------------------------------------- |
| 功能描述               | 评测警告时的回调                                      |
| 输入参数（回调返回值） | code: 错误码<br/>msg: 错误信息                        |
| 返回值                 | 无                                                    |
| 使用说明               | 评测警告时的回调，onResult 有结果                     |

**engineInitState**

| 函数原型               | Function(String code, String msg) |
| ---------------------- | --------------------------------- |
| 功能描述               | 评测初使化的回调                  |
| 输入参数（回调返回值） | code: 错误码<br>msg: 错误信息     |
| 返回值                 | 无                                |
| 使用说明               | 评测错误时的回调，onResult 无结果 |



## 三、响应结果

请参考对应题型的 <a href="#/help?url=mode/intro" target="_blank">接口参数</a> 文档。



## 四、状态码表

请参考 <a href="#/help?url=sdk/status_full" target="_blank">状态码</a> 文档。



## 五、常见问题

1.**Flutter SDK 支持的最小系统版本号是多少？**

> 答：Android版本SDK目前支持4.4及以上版本（即19），iOS 支持 12及以上的版本



2.**libc++_shared.so 冲突解决办法**

> 在工程的build文件中加入如下代码

~~~groovy
 android {
     packagingOptions {
          pickFirst 'lib/*/libc++_shared.so'
     }
 }
~~~



3.**如果使用代码混淆功能时，不能正常评测**

> 在proguard-rules.pro文件中检查是否添加了如下规则：

~~~
-keep class com.zhiyan.**{*;}
~~~



6.**语音评测支持哪些应用平台？**

> 目前语音评测支持：Android、iOS、HarmonyOS、Web、小程序、Linux、Windows、词典笔、RTOS等应用平台



7.**语音评测支持的音频格式有哪些？**

> 音频格式介绍，请查看 <a href="#/help?url=public/datadict/format" target="_blank">支持音频格式</a> 文档



8.**错误码及相关的解决方案**

> 错误码介绍，请查看 <a href="#/help?url=sdk/status_full" target="_blank">状态码</a> 文档



9.**语音评测支持的题型、文本、音频时长限制、结果及各字段的含义**

> 语音引擎支持题型、文本、音频时长限制，请查看 <a href="#/help?url=mode/intro" target="_blank">题型</a>文档 和 <a href="#/help?url=mode/common" target="_blank">参数介绍</a> 文档



10.**为什么返回结果中没有音频 audioUrl地址，或者audioUrl地址为空？**

> 需要在评测前传递audioUrl=true ，结果中才会返回audioUrl音频地址



11.**评测音频地址存储多久？**

> 默认保留**30**天，如果需要更长时间存储，建议下载至自己的服务器，保存音频 需要给评分引擎设置audioUrl=true。



12.**如果某次打分有问题，需要智言协助分析，需要提供哪些信息？**

> 提供智言返回的 audioUrl 即可，或者提供taskId



13.**使用麦克风录制的音频，没有朗读任何内容，此时评测为何会有分数？**

> 录音音频的环境中有噪声，噪声中的部分声音与评测文本中的发音相同，所以出现有分数的情况，但分数是偏低的。



14.**评测参数中，设置userId的作用是什么？是否需要设置？**

> userId用来标识不同的用户，建议使用评测的时候，将产品中的userId设置成和产品登录账号相关联的内容。若有用户反馈评分方面的问题，智言可以快速定位到评分数据进行分析。



15.**单词为什么没有流利度得分？**

> 流利度这项维度只有句子、段落等 这种长文本的题型才有，单词发音只要朗读清晰、饱满、正确，即可正常打分。



16.**语音评测Licence 是如何计数的？**

> 请求token或者获取激活码后，即用掉一个Licence，同一个设备多次向服务器发起注册请求，只计一次。



17.**语音评测并发是如何计算的？**

> 并发就是某一时刻同时在进行录音评测的请求个数。



18.**语音评测调用次数是否包含失败的情况？**

> 语音评测调用次数，只包含服务端成功响应的次数，失败的次数不会计算在内。



19.**句子评测和段落评测中，为何只读了部分内容，但流利度很很高？**

> 流利度是对已朗读的部分进行评价，这种未朗读完的情况，完整度的得分是偏低的。所以总分在这种情况下不会出现高分。
