# Popeye API

## API 说明
---
- API说明 - 所有API采用http方式，请求地址以http://{项目地址}/v1开头(后面以{apiPath}代替)。
- 所有请求必须带上名为x-auth-token的header作为签名，以验证请求合法性。
- 所有请求返回数据均为json格式数据。
- 所有URL路径，参数名均区分大小写 - 编码均为UTF-8。
- 在设置广告跳转链接时请勿在链接中带有_taskId的参数(系统会自动在任务跳转链接上补上该参数)，否则广告审核不予通过。
所有请求返回格式如下：
```
{ 
    "success": true, //是否成功 
    "code": "200", //code 
    "msg": "", //详细消息 
    "result": { //data } 
} 
```

## 鉴权说明 
- 首先通过申请获取appId,appKey,token
- 根据appKey,token,timestamp(当前系统时间戳转成毫秒数)生成签名，即sign
- 签名算法：appKey+token用MD5生成16进制摘要md5Hex，再对md5Hex+timestamp用MD5生成16进制签名，然后 按字母顺序对签名排序，最后对排序后的签名再次MD5生成16进制最终签名，下面是一个Java版的示例：
``` java
String appId = "3138ea97e3304c029a947c3b4122af79";
String appKey = "4296ce912e604214a15f3163235fb551"; 
String appToken = "8aafb36058be4386afabd94cf953c23e"; 
String sourcre = appKey + appToken; 
String md5Hex = DigestUtils.md5Hex(sourcre); 
long time = System.currentTimeMillis(); 
// System.out.println(time); // 1416447113075 
String digest = DigestUtils.md5Hex(md5Hex + time);
// System.out.println(digest); // ba2d43d7a4bda462c1f11416887bb8b8 
cha r[] digestArray = digest.toCharArray();
Arrays.sort(digestArray);
digest = DigestUtils.md5Hex(new String(digestArray));
// System.out.println(digest); // d28420809770d0bff3c9faf20d8a173c
//System.out.println(appId + ":" + digest + ":" + time);
//3138ea97e3304c029a947c3b4122af79:d28420809770d0bff3c9faf20d8a173c:1416447113075
```
在具体的请求头中带上鉴权头：x-auth-token：{appId}:{sign}:{timestamp}
请自行保证服务器时间准确性，如果{timestamp}误差超过5分钟，鉴权不通过
对于不提供头或鉴权不通过的请求将直接返回401状态码
对于鉴权通过但无权限访问的数据，返回失败，code为403
```
{ 
    "success": false, 
    "code": "403", 
    "msg": "禁止访问", 
    "result": null 
} 
```

## API
### 获取验证码
### 注册
```
url:{apiPath}/register
method:POST
```

参数：

>  **taskId**(任务ID，必填,可从广告跳转url中名为_taskId的参数中获取)，

> wxOpenId (微信的用户open_id)

> wxUserName(微信用户昵称)

> result(任务结果，完成：2，审核中：3，失败：4，不传或传其他数字则默认为2)

对于用户开始任务后24小时仍未回调接口的，视为任务失败，将返还广告金额，如果此时再回调接口，将不做任何处理，并返回失败，code为451
返回格式：
```
{
    "success": true,
    "code": "200",
    "msg": null,
    "result": "ok"
}
```









