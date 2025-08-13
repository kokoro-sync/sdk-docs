H5接入指南
======

NOTE:这篇文档，介绍H5游戏中怎么调用H5 SDK的API。 主要包含初始化、登录、支付等接口


引用依赖
-------

在H5游戏的页面加载之前和H5SDK API调用之前，就引用并加载H5SDK的JS文件: xsdk.js。 请直接引用xsdk.js的完整URL地址（不要将js文件下载到本地）。 js文件完整的URL地址，请联系商务获取。

比如在H5游戏的主文件index.html中引用：

~~~
<script src="https://oss.bytesdk.com/h5/xsdk.js"></script>
~~~


必须调用的接口
-------

#### 初始化(必接)

初始化应该在游戏界面渲染完成后调用或者在游戏加载资源完成后调用。 并且可以设置登出和切换账号成功回调。

```
window.XSDKApi.init(function(data) {
    // 需要在初始化回调后， 进行其他操作
    console.log('sdk init success:', data)

    // 设置登出回调
    window.XSDKApi.setLogoutCallback(function() {
        // 注意：游戏内收到这个回调后， 需要让玩家返回到游戏登录界面，重新掉login接口 ，让玩家重新登录
        console.log('logout from sdk. in game_test.html')
    });    
});

```

#### 登录接口(必接)

```

window.XSDKApi.login(function(loginResult) {
    console.log('login result called:', loginResult)

    /**
     *  loginResult结构：
     *     ::code： 状态(int) 1： 登录成功；0：登录失败
     *     ::msg :  错误说明文字
     *     ::data: 用户数据(Object)
             *     ::uid: SDK中用户全局唯一ID， 游戏可以使用该ID绑定角色信息
             *     ::name：用户名
             *     ::token: token，游戏传给游戏服务器，游戏服务器去Server二次登录验证时需要该参数
     */         

})
```

#### 登出接口(选接)

```
window.XSDKApi.logout()
```

#### 提交扩展数据(必接)

```
var roleData = {
    serverID: '1',
    serverName: 'H5演示服务器',
    roleID: '100',
    roleName: '角色_&#@!%^_+·~|{}[]',
    roleLevel: '1',
    vip: '1',
    moneyNum: 0,
    createTime: parseInt("" + new Date().getTime()/1000),
    lastLevelUpTime: parseInt("" + new Date().getTime()/1000)
};

if (createRole) {
    roleData.type = 1       //创建角色类型
} else {
    roleData.type = 2       //进入游戏类型， 3： 等级升级；4：退出游戏
}

window.XSDKApi.submit(roleData, function(result) {

    if (result && result.code == 0) {
        console.log('role submit success')
    } else {
        console.log('role submit failed')
    }

}) 

```
该方法将调用的时机分为几种类型：


1：创建角色
2：进入游戏
3：等级提升
4：退出游戏

在上面几个地方，都需要调用window.XSDKApi.submit。其中，type就是当前调用的时机。


关于roleData 数据结构:


| 参数名称        | 参数类型          | 参数说明  |
|:------------- |:-------------|:-----|
| type     | int | 调用时机(1,2,3,4)|
| serverID| String| 玩家所在服务器的ID|
| serverName| String| 玩家所在服务器的名称|
| roleID | String | 玩家角色ID|
| roleName| String | 玩家角色名称|
| roleLevel| String | 玩家角色等级|
| moneyNum| String | 当前角色身上拥有的游戏币数量|
| createTime| long | 角色创建时间，从1970年到现在的时间，单位秒,必须传入真实的数据，否则UC审核不过|
| lastLevelUpTime| long | 角色等级变化时间，从1970年到现在的时间，单位秒|
| vip| String | 玩家VIP等级|


#### 支付充值(必接)

```
var productPrice = 100           //当前道具金额（单位分）
var cpOrderID = parseInt("" + new Date().getTime() / 1000)          // 游戏自己的订单号
var cpPayNotifyUrl = "http://172.16.0.109:12201/game/pay/callback"  // 支付完成后， 游戏服务器接收SDK服务器回调通知地址

var orderData = {
    price: productPrice,
    productID: '1',
    productName: 'H5测试商品',
    productDesc: '充值10元送50钻石',
    serverID: '1',
    serverName: 'H5演示服务器',
    roleID: '100',
    roleName: '角色_&#@!%^_+·~|{}[]',
    roleLevel: 1,
    vip: '1',
    cpOrderID: cpOrderID,
    extra: "自定义数据，回调通知时原样返回",
    payNotifyUrl: cpPayNotifyUrl
};

window.XSDKApi.pay(orderData, function(result){

    if (result && result.code == 0) {
        console.log('sdk pay success')
    } else {
        console.log('sdk pay failed')
    }
    

});
```

关于OrderData对象：

| 参数名称        | 参数类型          | 参数说明  |
|:------------- |:-------------|:-----|
| productID     | String | 充值商品ID，游戏内的商品ID |
| productName      | String      |   商品名称，比如100元宝，500钻石...|
| productDesc| String      |    商品描述，比如 充值100元宝，赠送20元宝|
| price| int | 充值金额(单位：分)|
| serverID| String| 玩家所在服务器的ID|
| serverName| String| 玩家所在服务器的名称|
| roleID | String | 玩家角色ID|
| roleName| String | 玩家角色名称|
| roleLevel| int | 玩家角色等级|
| vip | String| 玩家vip等级 |
| payNotifyUrl| String | 游戏服务器支付回调地址，支付成功后，SDKServer根据该地址，通知游戏服务器发货；如果不设置，使用SDK后台配置的固定回调地址|
| cpOrderID | String | 游戏自己的订单号；支付成功之后，SDKServer通知游戏服务器时，原样返回给游戏服务器|
| extra | String | 扩展数据；支付成功之后，SDK Server通知游戏服务器时，原样返回给游戏服务器|


#### 其他接口

1、 展示用户中心（可选）

如果游戏内需要主动展示用户中心，可以调用如下接口

~~~
window.XSDKApi.showUserCenter()
~~~


2、 设置监听

当玩家在SDK内登出时，SDK将触发一个登出回调给游戏层，游戏层收到该回调后，应该引导玩家返回到游戏登录界面，重新登录。 可以在初始化完成后设置

~~~
window.XSDKApi.setLogoutCallback(function() {
    // 注意：游戏内收到这个回调后， 需要让玩家返回到游戏登录界面，重新掉login接口 ，让玩家重新登录
    console.log('logout from sdk. in game_test.html')
});
~~~
