# 96云网络验证

> 96云网络验证支持多平台、多终端、多开发环境，此文档将帮助你将96云网络验证的功能集成到你的应用或脚本中。

## 签名算法（sign）
```
sign = md5(appToken + param + appToken)
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
|1000|调用成功|
|1001|参数错误，请检查请求参数，详细字段错误会通过message返回|
|1002|无效的签名，请检查签名算法|
|1003|签名已过期，签名只在1分钟内有效且是一次性的|
|1004|软件状态,详细字段错误会通过message返回|
|10011|卡密不可用（卡密不存在）|
|10012|卡密入库失败：请联系客服|
|10013|卡密设置失败|
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
|time_stamp|是|string|时间戳，秒级（10位数）|
|sign|是|string|签名|

**响应示例** 
```json
{
    "code": 1000,
    "message": "登录成功",
    "result": {
        "Token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJjb2RlIjoiNUZJVlUyUFItNUZJVlUyUFItNUZJVlUyUFItNUZJVlUyUFIiLCJleHAiOjE2MjkwMjQ0Mzl9.Y0BFoMgVHL1h3gvW8pLdcmAoMKm6xgSPfPiI7UMcUzk",
        "Free": 1,
        "Endtime": "2021-09-15 23:20",
        "Activetime": "2021-08-15 14:00",
        "Exp": 1629024439
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
|Exp|int64|过去时间戳,秒级（10位数）|

### 卡密心跳💛
!>接口说明<br>用于保持登录状态，刷新token的有效期，token在超过2小时<br>未收到心跳请求将自动失效。

**接口地址 :** `/v1/code/heart`

**请求参数**

|参数名|是否必需|类型|参数说明|
|---|---|---|---|
|appid|是|int|项目的AppId|
|code|是|string|卡密|
|deivce|是|string|设备|
|time_stamp|是|string|时间戳，秒级（10位数）|
|token|是|string|登录成功后返回的令牌|
|sign|是|string|签名|