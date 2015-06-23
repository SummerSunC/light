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
```
url:{apiPath}/code/{phone}
method:GET
```
如果手机验证失败或已经注册返回失败，code为451

成功
```
{ 
  "success": true, 
  "code": "200", 
  "msg": "成功", 
  "result": {code}
}
```

### 注册
```
url:{apiPath}/register
method:POST
```

参数：
```
username:用户手机号
password:加密后密码
```
如果用户已经注册则返回失败，code为451

成功返回：
```
{ 
  "success": true, 
  "code": "200", 
  "msg": "成功", 
  "result": {userId}
}
```
### 更多账户信息
```
url:{apiPath}/info
method:POST
```
参数:
```
userId:用户id
headUrl: #这里是用流的方式还是先存到七牛直接返回给我url，在讨论
nickName: 昵称
username:手机号码
school:schoolId
```
失败返回code为451，成功返回code为200

### 获得school列表（注册更多这个页面应该需要选择省市好定位学校）
```
url:{apiPath}/schools
method:GET
```
参数 （不传参数返回所有）
```
province(可选):省id
city（可选）:市id
```
成功：
```
{
    "success": true,
    "code": "200",
    "msg": null,
    "result": {
        id:学校id,
        name:学校名称
        pic:学校图片url
        cityId:城市id
        dictName:区/乡/镇名
        cityName:市/县名
        provName:省/市名
    }
}
```
### 登录
```
url:{apiPath}/login
method:POST
```

参数：
```
username:用户手机号
password:加密后密码
```
如果用户不存在则返回失败，code为451

成功返回：
```
{ 
  "success": true, 
  "code": "200", 
  "msg": "成功", 
  "result": {
    id:用户id,
    schoolId:学校id,
    username:登录名称,
    nickName:昵称,
    phone:手机号,
    headUrl:头像,
    goldenCoin:金币数量,
    silverCoin:银币数量,
    copperCoin:铜币数量，
    totalUseTime:累计使用时间,
    rank_today:今日排名
  }
}
```
### 获得兑换券列表
```
url:{apiPath}/tickets
method:GET
```
参数：
```
userId:用户id
```
成功:
```
{ 
  "success": true, 
  "code": "200", 
  "msg": "成功", 
  "result": {
    [{id:用户id,
    schoolId:学校id,
    username:登录名称,
    nickName:昵称,
    phone:手机号,
    headUrl:头像,
    goldenCoin:金币数量,
    silverCoin:银币数量,
    copperCoin:铜币数量，
    totalUseTime:累计使用时间,
    rank_today:今日排名
    },
    {……}……
    ]
  }
}
```


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









