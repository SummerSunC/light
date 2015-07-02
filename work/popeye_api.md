# Popeye API
## API 说明
**此api为第一版v1，后续会改成oauth2方式**
---
- API说明 - 所有API采用http方式，请求地址以http://{项目地址}/v1开头(后面以{apiPath}代替)。
- 所有请求必须带上名为x-auth-token的header作为签名，以验证请求合法性。
- 所有请求返回数据均为json格式数据。
- 所有URL路径，参数名均区分大小写 - 编码均为UTF-8。
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
url:{apiPath}/code
method:GET,POST
```
参数
```
phone:手机号码
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
token:用户令牌（根据用户名密码加密过后的字符串）

// 加密方法如下(以java代码为例)
String key = 这里为appid
String input = 待加密字符串（格式为：u:{username},p:{password}  ）
String result = null;
Cipher cipher = Cipher.getInstance("DES/ECB/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, getCryptKey(key));  

byte[] inputBytes = input.getBytes("UTF-8");
byte[] outputBytes = cipher.doFinal(inputBytes);

BASE64Encoder encoder = new BASE64Encoder();
result = encoder.encode(outputBytes);
```
如果用户已经注册则返回失败，code为451

成功返回：
```
{
"success": true,
"code": 200,
"msg": "注册成功",
"result": {
    "id": "1df821599130453ba24d6d0dbf152e14",
    "schoolId": null,
    "username": "15096454536",
    "phone": "15096454536",
    "nickName": null,
    "email": null,
    "headPic": null,
    "sex": null,
    "totalUseTime": 0,
    "goldenCoin": 0,
    "silverCoin": 0,
    "copperCoin": 0,
    "updateAt": "2015-07-02 15:14:27",
    "createAt": "2015-07-02 15:14:27",
    "status": true
    }
}
```
### 更多账户信息
```
url:{apiPath}/user/info
method:POST
```
参数:
```
id:用户id  // 这里改为id 
headPic: #这里是用流的方式还是先存到七牛直接返回给我url，在讨论,这里改为headPic
nickName: 昵称
username:手机号码
school:schoolId
```
失败返回code为451，成功返回code为200

### 获得school列表（注册更多这个页面应该需要选择省市好定位学校）
```
url:{apiPath}/schools
method:GET,POST
```
参数 （不传参数返回所有）
```
city（可选）:市id
```
成功：
```
{
"success": true,
"code": 200,
"msg": "提交成功",
"result": [
        {
            "id": "ed60a3c307e444b788174d3cb940e57f",
            "name": "西安交通大学",
            "pic": null,
            "cityId": 287,
            "provId": 27,
            "distId": 3868,
            "distName": "未央区",
            "cityName": "西安市",
            "provName": "陕西省",
            "address": "128号",
            "updateAt": "2015-06-29 14:49:40",
            "createAt": "2015-06-29 13:14:44",
            "status": true
        },
        {
            "id": "f1edf1a12d8f4d60a316d96ae6024bb9",
            "name": "上海医科大学",
            "pic": null,
            "cityId": 73,
            "provId": 9,
            "distId": 720,
            "distName": "黄浦区",
            "cityName": "上海市",
            "provName": "上海市",
            "address": "128号",
            "updateAt": "2015-06-29 15:16:09",
            "createAt": "2015-06-29 14:48:32",
            "status": true
        }
    ]
}   
```
### 登录
```
url:{apiPath}/login
method:POST
```

参数：
```
token:用户令牌（根据用户名密码加密过后的字符串）//加密方式见注册
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
    rankToday:今日排名（名次） 这里改为rankToday
    rankRateToday:今日排名比例
  }
}
```
### 获得兑换券列表
```
url:{apiPath}/tickets
method:GET,POST
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
    [
    {
    id:奖券id,
    name:名称,
    type:兑换券类型 0：优惠券，1：代金券，2：折扣券,
    pic:图片url,
    totalNum:发放总数, //这里改为totalNum
    shopNames:投放门店名, //这里改为shopNames
    goldenCoin:所需金币数量,
    silverCoin:所需银币数量,
    copperCoin:所需铜币数量，
    eventCloseAt:活动截止日期
    conversionCloseAt:兑换截止日期,
    instruction:说明,
    depict:描述
    },
    {……}……
    ]
  }
}
```

### 获得兑换券详细信息
```
url:{apiPath}/ticket
method:GET,POST
```
参数：
```
ticketId:奖券id
```
成功:
```
{ 
  "success": true, 
  "code": "200", 
  "msg": "成功", 
  "result": {
    [
    {
    id:奖券id,
    name:名称,
    type:兑换券类型 0：优惠券，1：代金券，2：折扣券,
    pic:图片url,
    totalNum:发放总数,
    shopNames:投放门店名,
    goldenCoin:所需金币数量,
    silverCoin:所需银币数量,
    copperCoin:所需铜币数量，
    eventCloseAt:活动截止日期
    conversionCloseAt:兑换截止日期,
    instruction:说明,
    depict:描述,
    exchangeUserCount:成功兑换用户总数
    },
    {……}……
    ]
  }
}
```

### 获得历史记录
```
url:{apiPath}/history
method:GET,POST
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
    [
    {
    id:记录id,
    goldenCoin:金币数量,
    silverCoin:银币数量,
    copperCoin:铜币数量，
    finishTime:完成任务的分钟数
    createAt:完成的时间 （yyyy-MM-dd HH:mm:ss）
    remark:备注
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









