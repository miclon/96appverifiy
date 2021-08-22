# 96云网络验证

> 96云网络验证支持多平台、多终端、多开发环境，此文档将帮助你将96云网络验证的功能集成到你的应用或脚本中。

## 签名算法（sign）
```
sign = md5(appToken + param)
```
!>注意<br>参数必须按照顺序放置

|参数名|参数说明|
|---|---|
|appToken|项目密钥,在项目管理获取|
|param|1.将全部请求参数格式化成k=v<br>2.将格式化后的参数按升序排列，然后用&符号相隔拼接在一起，不想写排序代码<br>也可严格按照接口文档中的请求参数表写死顺序<br>（注意：1.不包括sign参数， 2.V不应进行url encode）|


## 公共参数详细释义

**time_stamp :** `当前时间戳，保证签名只在5分钟内有效，请求到达服务器后会将时间戳参数与当前服务器时间相比较，是否超过了300s。防止别有用心的人抓包并重放请求。`

**appid :** `AppId：在项目管理获取`

## 接口返回码对照表
|返回码|说明|
|---|---|
|401|Authorization无效|
|402|token必须填写|
|403|token失效|
|1000|调用成功|
|1001|参数错误，请检查请求参数，详细字段错误会通过message返回|
|1002|无效的签名，请检查签名算法|
|1003|签名已过期，签名只在5分钟内有效且是一次性的|
|1004|appid不正确或软件不存在|
|1005|软件状态已失效:可能原因：开发者不存在，开发者已被封禁，软件已过期，软件已被封建|
|1005|时间戳转失败：请联系客服|
|10010|该软件不是卡密模式|
|10011|卡密不存在|
|10012|创建设备失败：请联系客服|
|10013|卡密设置失败：请联系客服|
|10013|卡密不正确|
|10014|心跳失败：设备不存在|
|10015|充值失败：设备不存在|
|10016|充值失败：被充值的卡密不存在|
|10017|充值失败：充值使用的卡密不存在|
|10018|充值失败：具体原因会通过message返回|
|10019|充值失败：账号设置失败|
|10020|该软件不是用户模式|
|10021|注册失败：用户存在|
|10022|登录失败：用户不存在|
|10023|登录失败：密码错误|
|10024|token:账号或密码错误|
|10025|修改密码:账号或密码错误|
|10026|修改密码:密码错误|
|10027|修改密码:新密码不能跟旧密码相同|
|10028|修改密码失败|
|10029|用户充值失败：账号或密码错误|
|10030|卡密已被使用|
|10031|用户充值失败|
|10032|卡密换绑失败,具体原因会通过message返回|
|10033|系统没有开启试用功能|
|10034|请在开发者后台开启试用功能|
|10035|请开发者在开发者后台配置信息|
|10036|试用重置未开启|
|10037|试用次数已用完|
|10038|试用设置失败|
|10039|软件未开启试用功能|
|10040|试用重置失败|
|10041|试用重置时间未到|
|10042|用户不存在|
|10043|系统未开启更新功能|
|10044|更新功能未开启|
|10045|变量数据不存在|
|10046|变量状态异常|
|10047|变量修改失败|


## 卡密模式
### 卡密登录
**接口地址 :** `/v1/code/login`

**调用模式 :** `POST`

**请求参数** 

|参数名|是否必须|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|code|是|string|卡密|
|device|是|string|用户的设备唯一标识码|
|timestamp|是|int|时间戳，秒级（10位数）|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "登录成功",
    "result": {
        "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVaWQiOjYsIkNvZGUiOiI1RklWVTJQUiIsIkRldmljZSI6IjVGSVZVMlBSLTVGSVZVMlBSLTVGSVZVMlBSLTVGSVZVMlBSIiwiQXBwaWQiOjIsIkFwcHRva2VuIjoiZjA0ZDg3NmQ0MjE0ZjkyMDc4NTg3ZjM1NDc5OTE1ZjMwZmIwOWU1MyIsImV4cCI6MTYyOTY1MDI3MSwiaWF0IjoxNjI5NjQzMDcxLCJpc3MiOiIxMjcuMC4wLjEiLCJzdWIiOiJrbXRva2VuIn0.rn0FD-5EdFnJIZ_oh4FnbucQLmFEgPn0cHSLp0aJBEA",
        "Free": 1,
        "Endtime": "2021-09-18 23:20:07",
        "Activetime": "2021-08-15 14:00",
        "Exp": 1629650271
    }
}
```
**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Token|string|本次登录的标识，鉴权用的令牌|
|Free|int|激活状态，1表示已激活，0表示未激活|
|Endtime|datetime|卡密到期时间|
|Activetime|datetime|激活时间|
|Exp|int64|过期时间戳,秒级（10位数）|

### 卡密心跳💛
!>接口说明<br>用于保持登录状态，刷新token的有效期，token在超过2小时<br>未收到心跳请求将自动失效。

**接口地址 :** `/v1/code/heart`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "心跳成功",
    "result": {
        "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVaWQiOjYsIkNvZGUiOiI1RklWVTJQUiIsIkRldmljZSI6IjVGSVZVMlBSLTVGSVZVMlBSLTVGSVZVMlBSLTVGSVZVMlBSIiwiQXBwaWQiOjIsIkFwcHRva2VuIjoiZjA0ZDg3NmQ0MjE0ZjkyMDc4NTg3ZjM1NDc5OTE1ZjMwZmIwOWU1MyIsImV4cCI6MTYyOTY1MjQyMywiaWF0IjoxNjI5NjQ1MjIzLCJpc3MiOiIxMjcuMC4wLjEiLCJzdWIiOiJrbXRva2VuIn0.Za2RYO6YjjVUfC9jeXspv2MoCHFUUvvP0d9SBs_IT7w",
        "Free": 1,
        "Endtime": "2021-09-18 23:20:07",
        "Activetime": "2021-08-15 14:00",
        "Exp": 1629652423
    }
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Token|string|本次登录的标识，鉴权用的令牌|
|Free|int|激活状态，1表示已激活，0表示未激活|
|Endtime|datetime|卡密到期时间|
|Activetime|datetime|激活时间|
|Exp|int64|过期时间戳,秒级（10位数）|

### 卡密充值（以卡充卡）

**接口地址 :** `/v1/code/paykm`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|km|是|string|充值的卡密|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "充值成功"
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|

## 用户模式
### 用户注册

**接口地址 :** `/v1/user/reg`

**调用模式 :** `POST`


**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int|时间戳，秒级（10位数）|
|user|是|string|账号|
|pass|是|string|密码|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "注册成功"
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|

### 用户登录

**接口地址 :** `/v1/user/login`

**调用模式 :** `POST`


**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int|时间戳，秒级（10位数）|
|user|是|string|账号|
|pass|是|string|密码|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "登录成功",
    "result": {
        "endtime": "2026-08-21T16:37:48+08:00",
        "expire": "2021-08-23 01:30:31",
        "expireTime": 1629653431,
        "free": 1,
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVaWQiOjQsIlVzZXIiOiJ3ZWJ3c28iLCJQYXNzIjoiMTIzNDU2IiwiQXBwaWQiOjI5LCJBcHB0b2tlbiI6IjBjYjI3NTgzZjA2ZWRkZjJhNzliYTdmMmQyZGU1ODdhYjgzNDg0MzIiLCJleHAiOjE2Mjk2NTM0MzEsImlhdCI6MTYyOTY0NjIzMSwiaXNzIjoiMTI3LjAuMC4xIiwic3ViIjoiYXBwdXNlcnRva2VuIn0.3QUSFOI0FO_C9bKJbY3aKItFxJD1RbMIbYjSFB4XX0A"
    }
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|endtime|date|到期时间|
|expire|date|token过期时间|
|expireTime|int|token过期时间戳|
|free|int|0标识未激活 1表示已激活|
|Token|string|本次登录的标识，鉴权用的令牌|

### 用户心跳

**接口地址 :** `/v1/user/heart`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "心跳成功",
    "result": {
        "expire": "2021-08-23 01:34:41",
        "expireTime": 1629653681,
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVaWQiOjQsIlVzZXIiOiJ3ZWJ3c28iLCJQYXNzIjoiMTIzNDU2IiwiQXBwaWQiOjI5LCJBcHB0b2tlbiI6IjBjYjI3NTgzZjA2ZWRkZjJhNzliYTdmMmQyZGU1ODdhYjgzNDg0MzIiLCJleHAiOjE2Mjk2NTM2ODEsImlhdCI6MTYyOTY0NjQ4MSwiaXNzIjoiMTI3LjAuMC4xIiwic3ViIjoiYXBwdXNlcnRva2VuIn0.wQTfgIwxngzWwLjDj59ewl4O_ZHSieXtYjK0X13buaM"
    }
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Token|string|本次登录的标识，鉴权用的令牌|
|expire|date|过期时间|
|expireTime|int64|过期时间戳,秒级（10位数）|

### 用户修改密码

**接口地址 :** `/v1/user/password`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|password|是|string|当前用户密码|
|new_password|是|string|新密码|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "修改成功",
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|

### 用户卡密充值

**接口地址 :** `/v1/user/pay`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|km|是|string|卡密|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "充值成功",
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|

## 卡密试用功能
### 试用登录
**接口地址 :** `/v1/trial/code/login`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|device|是|string|设备|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "登录成功",
    "result": {
        "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVaWQiOjIzLCJDb2RlIjoiIiwiRGV2aWNlIjoiNUZJVlUyUHNzIiwiQXBwaWQiOjIsIkFwcHRva2VuIjoiZjA0ZDg3NmQ0MjE0ZjkyMDc4NTg3ZjM1NDc5OTE1ZjMwZmIwOWU1MyIsImV4cCI6MTYyOTY1NDE3MiwiaWF0IjoxNjI5NjQ2OTcyLCJpc3MiOiIxMjcuMC4wLjEiLCJzdWIiOiJrbXRva2VuIn0.R3_UFKbCI5LPqqjiwMfXLE8G1GzvKGMKfvSuf3ooVho",
        "Free": 0,
        "Number": 1,
        "Num": 10,
        "Activetime": "2021-08-22 23:42:52",
        "Exp": 1629654172
    }
}
```
**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Token|string|本次登录的标识，鉴权用的令牌|
|Free|int|0表示未激活 1表示已激活|
|Number|int|当前试用次数,当number超过num则试用次数已用完|
|Num|int|软件试用次数|
|Activetime|是|date|激活时间|
|Exp|int64|token过期时间戳|

### 试用心跳

**接口地址 :** `/v1/trial/code/heart`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "登录成功",
    "result": {
        "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVaWQiOjIzLCJDb2RlIjoiIiwiRGV2aWNlIjoiNUZJVlUyUHNzIiwiQXBwaWQiOjIsIkFwcHRva2VuIjoiZjA0ZDg3NmQ0MjE0ZjkyMDc4NTg3ZjM1NDc5OTE1ZjMwZmIwOWU1MyIsImV4cCI6MTYyOTY1NDE3MiwiaWF0IjoxNjI5NjQ2OTcyLCJpc3MiOiIxMjcuMC4wLjEiLCJzdWIiOiJrbXRva2VuIn0.R3_UFKbCI5LPqqjiwMfXLE8G1GzvKGMKfvSuf3ooVho",
        "Free": 0,
        "Activetime": "2021-08-22 23:42:52",
        "Endtime": "2021-08-23 00:00:52",
        "Exp": 1629654172
    }
}
```
**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Token|string|本次登录的标识，鉴权用的令牌|
|Free|int|0表示未激活 1表示已激活|
|Activetime|是|date|激活时间|
|Endtime|是|date|试用到期时间|
|Exp|int64|token过期时间戳|

## 卡密换绑
**接口地址 :** `/v1/bound/code`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌(支持 Authorization Bearer)|
|code|是|string|登录的卡密|
|device|是|string|设备|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "换绑成功",
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|

## 变量管理
### 获取变量
**接口地址 :** `/v1/var/get`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|name|是|string|变量名称|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "获取成功",
    "result":{
        "name":"demo",
        "value":"demoValue",
        "bz":"备注",
    }
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|name|string|变量名称|
|value|string|变量内容|
|bz|string|变量备注|

### 修改变量
**接口地址 :** `/v1/var/set`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|name|是|string|变量名称|
|value|是|string|变量内容|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "修改成功",
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|

## 软件相关
### 获取版本更新
**接口地址 :** `/v1/software/up`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "获取成功",
    "result": {
        "Coerce": 1,
        "context": "更新测试",
        "new_ver": "1.34",
        "url": "http://oss.318ka.cn/image/20210725/a9cd2f54faa3f9c9911abf8bc2c9a7ad.apk",
        "ver": "1.34"
    }
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Coerce|string|1为强制更新 0未不强制更新|
|new_ver|string|最新版本|
|url|string|下载地址|
|ver|string|当前版本|

### 获取软件信息
**接口地址 :** `/v1/software/config`

**调用模式 :** `POST`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|timestamp|是|int64|时间戳，秒级（10位数）|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "获取成功",
    "result": {
        "Appid": 1002,
        "Name": "demo",
        "Icon": "http://oss.318ka.cn/image/20210713/8fe4d8d873504ed843a269b5bd5b89bf.png",
        "Ver": "1.34",
        "Summary": "my demo is ios android",
        "Contact": "123456",
        "Lang": "C#",
        "Trial": 1,
        "Bound": 1,
        "Up": 1
    }
}
```

**响应参数**

|参数名|类型|参数说明|
|---|---|---|
|code|int|返回码，1000表示正确，详细参照返回码对照表|
|message|string|请求信息|
|result|object|请求正确时，若有额外数据要返回，则结果封装在该字段。若无额外数据，则无此字段。|
|Appid|string|软件Appid|
|Name|string|软件名称|
|Icon|string|软件图标|
|ver|string|当前版本|
|Summary|string|软件介绍|
|Contact|string|联系方式|
|Lang|string|语言平台|
|Trial|int|试用功能 1为机器码 2为注册码 0为关闭|
|Bound|int|换绑功能 1为开启 0为关闭|
|Up|int|更新功能 1为开启 0为关闭|