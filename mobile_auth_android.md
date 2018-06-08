# 1. 开发环境配置

## 1.1. 总体使用流程

1. 调用SDK方法来获得`token`，步骤如下：

    a. 构造SDK中认证工具类AuthnHelper的对象；</br>

    b. 使用AuthnHelper中的getToken方法，获得token。

2. 使用平台本机号码校验接口，进行号码校验


## 1.2. 新建工程并导入SDK的jar文件

将`mobile_auth_android_*.jar`拷贝到应用工程的libs目录下，如没有该目录，可新建；

## 1.3. 配置AndroidManifest

注意：为避免出错，请直接从Demo中复制带<!-- required -->标签的代码

**配置权限**

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_SETTINGS"/>

```

通过以上步骤，工程就已经配置完成了。接下来就可以在代码里使用统一认证的SDK进行开发了

## 1.4. SDK使用步骤

**1. 创建一个AuthnHelper实例** 

`AuthnHelper`是SDK的功能入口，所有的接口调用都得通过`AuthnHelper`进行调用。因此，调用SDK，首先需要创建一个`AuthnHelper`实例，其代码如下：

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mContext = this;    
    ……
    mAuthnHelper = AuthnHelper.getInstance(mContext);
    }
```

**2. 实现回调**

所有的SDK接口调用，都会传入一个回调，用以接收SDK返回的调用结果。结果以`JsonObjent`的形式传递，`TokenListener`的实现示例代码如下：

```java
mListener = new TokenListener() {
    @Override
    public void onGetTokenComplete(JSONObject jObj) {
        if (jObj != null) {
            mResultString = jObj.toString();
            mHandler.sendEmptyMessage(RESULT);
            if (jObj.has("token")) {
                mtoken = jObj.optString("token");
            }
        }
    }
};
```
日志打印回调，`TraceLogger`的实现示例代码如下：

```java
mTraceLogger = new TraceLogger() {
            @Override
            public void verbose(String tag, String msg) {

            }

            @Override
            public void debug(String tag, String msg) {
                Log.d(tag, msg);
            }

            @Override
            public void info(String tag, String msg) {
                Log.i(tag, msg);
            }

            @Override
            public void warn(String tag, String msg, Throwable tr) {

            }

            @Override
            public void error(String tag, String msg, Throwable tr) {
                Log.e(tag, msg);
                if (tr!= null){
                    tr.printStackTrace();
                }

            }
        };
```

**3. 接口调用**

```java
mAuthnHelper.umcLoginByType(Constant.APP_ID, 
        	Constant.APP_KEY,
		"200", 
		System.currentTimeMillis()+"",
		"",
		12000, 
		12000,
        	mListener,
		mTraceLogger);
```



<div STYLE="page-break-after: always;"></div>

# 2. SDK方法说明

## 2.1. 获取管理类的实例对象

### 2.1.1. 方法描述

获取管理类的实例对象

**原型**

```java
public AuthnHelper (Context context)
```

### 2.1.2. 参数说明

| 参数      | 类型      | 说明                              |
| ------- | ------- | ------------------------------- |
| context | Context | 调用者的上下文环境，其中activity中this即可以代表。 |

## 2.2. 获取校验凭证token

### 2.2.1. 方法描述

开发者向统一认证服务器获取用户身份标识`openId`和临时凭证`token`。</br>

**openId：**每个APP每个手机号码对应唯一的openId。</br>

**临时凭证token：**开发者服务端可凭临时凭证token通过3.1本机号码校验接口对本机号码进行验证。

**原型**

```java
public void umcLoginByType(final String appId, 
            final String appKey, 
	    final String capaid, 
	    final String capaidTime,
	    String scene,
	    int connTimeOut, 
	    int readTimeOut,
            final TokenListener listener, 
	    TraceLogger mTraceLogger)
```

### 2.2.2. 参数说明

**请求参数**

| 参数           | 类型            | 说明                                       |
| :----------- | :------------ | :--------------------------------------- |
| appId        | String        | 应用的AppID                                 |
| appkey       | String        | 应用密钥                                     |
| capaid       | String        | 授权内容：获取手机号码 200                          |
| capaidTime   | String        | 授权时间 为13位的时间毫秒值 例如：  1509330580234       |
| scene        | String        | 场景参数  默认为空                               |
| connTimeOut  | int           | 网络请求连接超时参数  单位为ms                        |
| readTimeOut  | int           | 网络请求读取超时参数  单位为ms                        |
| listener     | TokenListener | TokenListener为回调监听器，是一个java接口，需要调用者自己实现；TokenListener是接口中的认证登录token回调接口，OnGetTokenComplete是该接口中唯一的抽象方法，即void OnGetTokenComplete(JSONObject  jsonobj) |
| mTraceLogger | TraceLogger   | TraceLogger为回调日志打印，是一个java接口，需要调用者自己实现；  |

**响应参数**

OnGetTokenComplete的参数JSONObject，含义如下：

| 字段          | 类型     | 含义                                       |
| ----------- | ------ | ---------------------------------------- |
| resultCode  | String | 接口返回码，“103000”为成功。具体响应码见4.1. 本机号码校验接口返回码 |
| authType    | String | 登录类型返回码                                  |
| authTypeDes | String | 登录类型返回码描述                                |
| resultDesc  | String | 失败时返回：返回错误码说明                            |
| token       | String | 成功时返回：身份标识，字符串形式的token，第三方应用将该凭证经应用平台向统一认证平台请求认证 |
| openId      | String | 成功时返回：用户身份唯一标识                           |
| securityphone  | String | 成功时返回：用户手机号掩码                           |
| traceId      | String | 请求唯一标示                        |

### 2.2.3. 示例

**请求示例代码**

```java
mAuthnHelper.umcLoginByType(Constant.APP_ID, 
        	Constant.APP_KEY,
		"200", 
		System.currentTimeMillis()+"",
		"",
		12000, 
		12000,
        	mListener,
		mTraceLogger);
```

**响应示例代码**

```
{
    "resultCode":"103000",
    "authType":"2",
    "authTypeDes":"网关鉴权",
    "traceId":"75de5ad8f5b04074a1e85b3935ad197a",
    "openId":"9M7RaoZH1Q95QzY99YFkeFDO4xDfOv5q4BVlwn_0zJNNlNYUkxrw",
    "securityphone":"138****5380",
    "token":"STsid0000001528250537839kDSwAonpXeFGHlwwbT2TZQRaYKOWOC2j"
}
```

## 2.3. 取消获取校验凭证token

### 2.3.1. 方法描述

开发者调用SDK获取token 超过一定时间没有返回token的时候调用此方法进行取消。</br>


**原型**

```java
public void cancel()
```

### 2.3.2. 示例

**请求示例代码**

```java

  mAuthnHelper.cancel();               
 
```


<div STYLE="page-break-after: always;"></div>

# 3. 平台接口说明

## 3.1. 获取用户信息接口

### 3.1.1. 接口说明

客户端SDK携带用户授权成功后的token来调用SDK服务端的相应访问用户资源的接口。

### 3.1.2. 接口描述

**请求地址：**https://www.cmpassport.com/unisdk/rsapi/tokenValidate

**协议：**HTTPS

**请求方法：**POST+JSON

### 3.1.3.参数说明

**请求参数**

| 参数名称          | 约束      | 层级   | 参数类型   | 说明                                       |
| ------------- | ------- | ---- | ------ | ---------------------------------------- |
| header        | 必选      | 1    |        |                                          |
| version       | 必选      | 2    | string | 填1.0                                     |
| msgid         | 必选      | 2    | string | 标识请求的随机数即可(1-36位)                        |
| systemtime    | 必选      | 2    | string | 请求消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| strictcheck   | 必选      | 2    | string | 暂时填写"0"                                  |
| sourceid      | 可选      | 2    | string | 业务集成统一认证的标识                              |
| ssotosourceid | 可选      | 2    | string | 单点登录时使用，填写被登录业务的sourceid                 |
| appid         | 必选      | 2    | string | 业务在统一认证申请的应用                             |
| apptype       | 必选      | 2    | string | 1:BOSS</br>2:web</br>3:wap</br>4:pc客户端</br>5:手机客户端 |
| expandparams  | 扩展参数    | 2    | Map    | map(key,value)                           |
| sign          | 当有密钥时必填 | 2    | String | 业务端RSA私钥签名（appid+token）, 服务端使用开发者提供的公钥进行RSA公钥解密开发者 |
| body          | 必选      | 1    |        |                                          |
| token         | 必选      | 2    | string | 需要解析的凭证值。                                |

**请求示例**

```
{
    "header": {
        "strictcheck": "0",
        "version": "1.0",
        "msgid": "4d1953f426a14b2ba9edd07b269cea70",
        "systemtime": "20171109145619809",
        "appid": "300005687666",
        "apptype": "5",
        "sourceid": "800120170818101447",
        "sign": "751E95E65384F2563E9285E94248AF63E972B2326F648CB947D8308543BCF4902D82A9CA7174880A388D6649BD66B2848A4B814143FA61584506CD7E9FB6B7A8AB5AF40F944CC930703ADC3356DE13B93FB429CFC393D8B73FDC02711B0B60848A735D1F90810FBEB8475FEEFBE6D8E774ABE9CB77E857A1CF49AFBA097B0413"
    },
    "body": {
        "token": "STsid0000001510210576710ivYn20UXdDa8Nhw0LpL1szQi5KJhgf5r"
    }
}
```

**响应参数**

| 参数名称                | 约束   | 层级   | 参数类型   | 说明                                       |
| ------------------- | ---- | ---- | ------ | ---------------------------------------- |
| header              | 必选   | 1    |        |                                          |
| version             | 必选   | 2    | string | 1.0                                      |
| inresponseto        | 必选   | 2    | string | 对应的请求消息中的msgid                           |
| systemtime          | 必选   | 2    | string | 响应消息发送的系统时间，精确到毫秒，共17位，格式：20121227180001165 |
| resultCode          | 必选   | 2    | string | 返回码，返回码请参考SDK返回码说明                       |
| body                | 必选   | 1    |        |                                          |
| userid              | 必选   | 2    | string | 系统中用户的唯一标识                               |
| pcid                | 必选   | 2    | string | 伪码id（显示和隐试登录都有）                          |
| usessionid          | 可选   | 2    | string | 暂忽略                                      |
| passid              | 可选   | 2    | string | 用户统一账号的系统标识                              |
| andid               | 可选   | 2    | string | 用户的“和ID”                                 |
| msisdn              | 可选   | 2    | string | 表示手机号码(当appid有开发者提供的密钥时，手机号用RSA公钥加密返回。使用私钥可以) |
| email               | 可选   | 2    | string | 表示邮箱地址                                   |
| loginidtype         | 可选   | 2    | string | 登录使用的用户标识：</br>0：手机号码</br>1：邮箱           |
| msisdntype          | 可选   | 2    | string | 手机号码的归属运营商：</br>0：中国移动</br>1：中国电信</br>2：中国联通</br>99：未知的异网手机号码 |
| province            | 可选   | 2    | string | 用户所属省份(暂无)                               |
| authtype            | 可选   | 2    | string | 认证类型：</br>0:其他；</br>1:WiFi下网关鉴权；</br>2:网关鉴权；</br>3:短信上行鉴权；</br>7:短信验证码登录 |
| authtime            | 可选   | 2    | string | 统一认证平台认证用户的时间                            |
| lastactivetime      | 可选   | 2    | string | 暂无                                       |
| relateToAndPassport | 可选   | 2    | string | 是否已经关联到统一账号，暂无用处                         |
| fromsourceid        | 可选   | 2    | string | 来源sourceid（即签发token sourceid）            |
| tosourceid          | 可选   | 2    | string | 目的sourceid（即被登录业务sourceid）               |

**响应示例**

```
{
    "body": {
        "msisdntype": "0",
        "lastactivetime": "2017-11-09 14:56:16",
        "authtype": "WAPGW",
        "fromsourceid": "800120170714100001",
        "loginidtype": "0",
        "authtime": "2017-11-09 14:56:16",
        "msisdn": "48DB711B1BA151E20DBD2F726404E44F352561A30A58C74368A8A8C64851AA9E652AA55E8924FA6BACA660DFBE0F3E9EEA5FF7369CED03817445397DA0E0D590E0A6871655464C0F336192F00147718FD4C05511E3C0F400F65DCC18FD80D368F3F429C67FD4D6B6F38673AF8D3388891099847203897CD4025D5E54BFC3B570"
    },
    "header": {
        "inresponseto": "4d1953f426a14b2ba9edd07b269cea70",
        "resultcode": "103000",
        "systemtime": "20171109145620587",
        "version": "1.0"
    }
}
```

<div STYLE="page-break-after: always;"></div>

# 4. 平台返回码说明

## 4.1. 平台返回码

| 错误编号   | 返回码描述                |
| ------ | -------------------- |
| 103101 | 签名错误                 |
| 103103 | 用户不存在                |
| 103104 | 用户不支持该种登录方式          |
| 103105 | 密码错误                 |
| 103106 | 用户名错误                |
| 103107 | 已存在相同的随机数            |
| 103108 | 短信验证码错误              |
| 103109 | 短信验证码超时              |
| 103111 | WAP网关IP不合法           |
| 103112 | 请求错误 reqError        |
| 103113 | Token内容错误            |
| 103114 | token验证 KS过期         |
| 103115 | token验证 KS不存在        |
| 103116 | token验证 sqn错误        |
| 103117 | mac异常 macError       |
| 103118 | sourceid不存在          |
| 103119 | appid不存在appidNOExist |
| 103120 | clientauth不存在        |
| 103121 | passid不存在            |
| 103122 | btid不存在              |
| 103123 | redisinfo不存在         |
| 103124 | ksnaf校验不一致           |
| 103125 | 手机格式错误               |
| 103126 | 手机号不存在               |
| 103127 | 证书验证，版本过期            |
| 103128 | gba webservice接口调用失败 |
| 103129 | 获取短信验证码的msgtype异常    |
| 103130 | 新密码不能与当前密码相同         |
| 103131 | 密码过于简单               |
| 103132 | 用户注册失败               |
| 103133 | sourceid不合法          |
| 103134 | wap方式手机号为空           |
| 103135 | 昵称非法                 |
| 103136 | 邮箱非法                 |
| 103138 | appid已存在             |
| 103139 | sourceid已存在          |
| 103200 | 不需要更新ks              |
| 103204 | 缓存随机数不存在             |
| 103205 | 服务器内部异常              |
| 103207 | 发送短信失败               |
| 103212 | 校验密码失败               |
| 103213 | 旧密码错误                |
| 103214 | 访问缓存或数据库错误           |
| 103226 | sqn过小或过大             |
| 103265 | 用户已存在                |
| 103901 | 短信验证码下发次数已达上限        |
| 103902 | 凭证校验失败               |
| 104001 | APPID和APPKEY已存在      |
| 105001 | 联通网关取号失败             |
| 105002 | 移动网关取号失败             |
| 105003 | 电信网关取号失败             |
| 105004 | 短信上行ip检测不合法          |
| 105005 | 短信上行发送信息为空           |
| 105006 | 手机号码为空               |
| 105007 | 手机号码格式错误             |
| 105008 | 短信内容为空               |
| 105009 | 解析失败                 |

## 4.2. SDK返回码说明

| 错误编号   | 返回码描述                                  |
| ------ | -------------------------------------- |
| 103000 | 成功                                     |
| 102101 | 无网络                                    |
| 102203 | 输入参数缺失（缺少appid、appkey、capaId其中任何一个时返回） |
| 102506 | 请求出错                                   |
| 102507 | 请求超时                                   |
| 102508 | 数据网络切换失败                                   |
| 102509 | 未知错误，错误详情请看异常堆栈信息                                 |
| 200002 | 没有sim卡                                 |
| 200005 | 用户未授权READ_PHONE_STATE                  |
| 200009 | 应用合法性校验失败                              |
| 200010 | 不支持的认证方式                               |
