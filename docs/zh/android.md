Android接入指南
======

NOTE:这篇文档，介绍游戏中怎么快速完成SDK Android平台的接入


接入准备
-------


**1、文件说明**

~~~
SDKDemo/sdk-demo: SDK Demo工程
        ----libs: SDK所有相关的依赖jar和aar
~~~

**2、添加依赖**


2.1 添加依赖库

将SDK相关的依赖库，拷贝到游戏工程目录下

~~~
将SDKDemo/sdk-core/libs下所有文件，拷贝到游戏工程的libs（依赖jar包,aar等目录存放位置）
复制 main/assets/ug_dev.properties 文件到工程相同的目录下
~~~

同时，在游戏工程的build.gradle中，添加如下依赖库：

~~~
//noinspection GradleCompatible
implementation 'com.android.support:support-fragment:28.0.0'
~~~

2.3 配置AndroidManifest.xml

将以下这段复制并黏贴到游戏出包工程的AndroidManifest.xml中application节点内：
```xml
<activity-alias
    android:name="${applicationId}.wxapi.WXPayEntryActivity"
    android:exported="true"
    android:targetActivity="com.game.sdk.service.pay.weixin.app.WXPayEntryActivity" />
```

2.4 配置混淆规则

因为SDK库已经经过混淆， 所以如果游戏工程开启了混淆， 请在混淆配置文件中加入如下规则，忽略SDK中的api:

~~~
-keep class com.game.sdk.** { *; }
-keep class com.tencent.** { *; }
-keep class com.qq.gdt.** { *; }
-keep class com.alipay.** { *; }
-keep class com.bytedance.** { *; }
-keep class org.json.** { *; }

-keep class com.umeng.** {*;}
-keep class org.repackage.** {*;}
-keep class com.uyumao.** { *; }
-keepclassmembers class * {
   public <init> (org.json.JSONObject);
}
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
~~~
SDK需要引用导入工程的资源文件，通过了反射机制得到资源引用文件R.java，但是在开发者通过proguard等混淆/优化工具处理apk时，proguard可能会将R.java删除，如果遇到这个问题，请添加如下配置：
-keep public class [您的应用包名].R$*{
public static final int *;
}

配置Application
-------

如果游戏层没有自己的Application，那么游戏需要将AndroidManifest.xml中的application指定为 **com.game.sdk.app.UGApplication**。

如果游戏有自己的Application， 则可以直接继承com.game.sdk.app.UGApplication类


必须调用的接口
-------

NOTE:所有接口调用，都通过com.game.sdk.XPlatform 单例类来调用

1、初始化(必接)

**该方法必须在游戏启动Activity的onCreate方法中，调用**

```java
String appID = "";                                  // TODO: 设置为SDK后台分配的appID参数
String appKey = "1111111";                          // TODO: 设置为SDK后台分配的appKey参数
int orientation = UInitParams.ORIENTATION_PORTRAIT; //TODO: 设置为游戏横竖屏

XPlatform.getInstance().init(this, new UInitParams(appID, appKey, orientation), new IInitListener() {
    @Override
    public void onInitFailed(int code, String msg) {
        Log.e("UGSDKDemo", "sdk init failed. code:"+code+";msg:"+msg);
    }

    @Override
    public void onInitSuccess() {
        Log.d("UGSDKDemo", "sdk init success");
    }

});
```

2、登录接口(必接)

调用登录接口，打开SDK登录界面。 SDK中根据国家最新的防沉迷政策，做了未成年人防沉迷限制， 游戏内不需要自己附加自己的防沉迷限制。

```java
XPlatform.getInstance().login(this, new ILoginListener() {
    @Override
    public void onLoginSuccess(UUser user) {
        Log.d("UGSDKDemo", "sdk login success."+user.getUid());
        // TODO: 游戏层将uid, token等发给游戏服务器， 游戏服务器再去SDK服务器端做登陆验证。 具体见服务端文档中登陆认证部分。
    }

    @Override
    public void onLoginFailed(int code, String msg) {
        Toast.makeText(MainActivity.this, "登陆失败:"+msg, Toast.LENGTH_LONG).show();
    }
});
```

3、登出接口(选接)

调用登出接口， SDK账户登出， 但是不是每个SDK都具有登出逻辑，对应没有提供登出接口的SDK，调用该函数，什么也不操作

```java
XPlatform.getInstance().logout(this);
```

4、提交扩展数据(必接)

Note: 部分渠道要求在 创建角色，登录游戏，角色升级，退出游戏 等时刻，必须要上报游戏中玩家数据，以便渠道后台统计用户数据。所以，游戏层需要在特定的地方多次调用该方法。

```java
URoleData data = new URoleData();
data.setType(type);
data.setServerID("2");
data.setServerName("test_1");
data.setRoleID("1");
data.setRoleName("test_role_1");
data.setRoleLevel("1");
data.setVip("1");
data.setCreateTime(System.currentTimeMillis()/1000);
data.setLastLevelUpTime(data.getCreateTime());

XPlatform.getInstance().submit(this, data); 
```

该方法将调用的时机分为几种类型：

1. 创建角色
2. 进入游戏
3. 等级提升
4. 退出游戏

所以在上面4个地方，都需要调用
XPlatform.getInstance().submit(URoleData data)

其中，URoleData就是游戏内玩家的数据，结构中的参数不允许空值，如果没有值，请传入默认值0。 不同的调用时机，用URoleData.dataType来区分。 创建角色的时候，dataType为1；进入游戏时，dataType为2；等级提升时，dataType为3；退出游戏时，dataType为4

关于UserExtraData 数据结构:


| 参数名称        | 参数类型          | 参数说明  |
|:------------- |:-------------|:-----|
| dataType     | int | 调用时机|
| serverID| String| 玩家所在服务器的ID|
| serverName| String| 玩家所在服务器的名称|
| roleID | String | 玩家角色ID|
| roleName| String | 玩家角色名称|
| roleLevel| String | 玩家角色等级|
| moneyNum| String | 当前角色身上拥有的游戏币数量|
| createTime| long | 角色创建时间，从1970年到现在的时间，单位秒,必须传入真实的数据，否则UC审核不过|
| lastLevelUpTime| long | 角色等级变化时间，从1970年到现在的时间，单位秒|
| vip| String | 玩家VIP等级|
| extraData| String | 附加数据，可以不传|


6、支付充值(必接)

调用充值接口，打开SDK充值界面。

```java
UOrder order = new UOrder();
order.setProductID("1");
order.setProductName("测试商品");
order.setProductDesc("测试商品描述");
order.setRoleID("1");
order.setRoleName("test_role_1");
order.setRoleLevel("1");
order.setVip("1");
order.setServerID("2");
order.setServerName("test_1");
order.setPrice(64800);          //金额单位：分
order.setCurrency("CNY");
order.setCpOrderID(System.currentTimeMillis()+"");
order.setExtra("extra data");
XPlatform.getInstance().pay(this, order, new IPayListener() {
    @Override
    public void onPaySuccess(UOrder order) {

    }

    @Override
    public void onPayFailed(UOrder order, String msg) {

    }

    @Override
    public void onPayCanceled(UOrder order) {

    }
});
```

关于PayParams对象：

| 参数名称        | 参数类型          | 参数说明  |
|:------------- |:-------------|:-----|
| productID     | String | 充值商品ID，游戏内的商品ID |
| productName      | String      |   商品名称，比如100元宝，500钻石...|
| productDesc| String      |    商品描述，比如 充值100元宝，赠送20元宝|
| price| int | 充值金额(单位：分)|
| currency| String | 货币单位，固定值CNY|
| cpOrderID| String | 游戏订单号 |
| serverID| String| 玩家所在服务器的ID|
| serverName| String| 玩家所在服务器的名称|
| roleID | String | 玩家角色ID|
| roleName| String | 玩家角色名称|
| roleLevel| String | 玩家角色等级|
| vip | String| 玩家vip等级 |
| payNotifyUrl| String | 游戏服务器支付回调地址，SDK支付成功，异步通知该地址，让游戏服务器给玩家发货|
| extra | String | 自定义数据， 支付回调通知给游戏服务器时，原封不动返回该值|



7 生命周期函数(必接)
-------

在游戏主Activity的如下生命周期函数中，调用对应的方法。

```java
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        XPlatform.getInstance().onActivityResult(this, requestCode, resultCode, data);
    }



    public void onStart() {
        super.onStart();
        XPlatform.getInstance().onStart(this);
    }


    public void onPause() {
        super.onPause();
        XPlatform.getInstance().onPause(this);
    }


    public void onResume() {
        super.onResume();
        XPlatform.getInstance().onResume(this);
    }


    public void onNewIntent(Intent newIntent) {
        super.onNewIntent(newIntent);
        XPlatform.getInstance().onNewIntent(this, newIntent);
    }


    public void onStop() {
        super.onStop();
        XPlatform.getInstance().onStop(this);
    }


    public void onDestroy() {
        super.onDestroy();
        XPlatform.getInstance().onDestroy(this);
    }


    public void onRestart() {
        super.onRestart();
        XPlatform.getInstance().onRestart(this);
    }


    public void onBackPressed() {
        super.onBackPressed();
        XPlatform.getInstance().onBackPressed(this);
    }


    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        XPlatform.getInstance().onConfigurationChanged(this, newConfig);
    }


    public void attachBaseContext(Context newBase) {
        super.attachBaseContext(newBase);
        XPlatform.getInstance().attachBaseContext(this, newBase);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        XPlatform.getInstance().onRequestPermissionResult(this, requestCode, permissions, grantResults);

    }
```


8 用户 Logout 事件监听(选接)
```java
// 监听 logout 事件(当用户点击 切换账号 按钮回调)
XPlatform.getInstance().setLogoutListener(new ILogoutListener() {
    @Override
    public void onLogout() {
        //TODO: 游戏层这里需要让玩家返回到游戏登陆界面，重新登陆
        Log.d("UGSDKDemo", "sdk logout. game to logout");
        // 可选
        // 如果游戏没有自己的登录页面, 当用户点击 切换账号 之后可以调用 login() 方法展示登录弹框
        // 在 logout 事件中, 重新调用登录方法展示登录弹框
//        XPlatform.getInstance().login(MainActivity.this, new ILoginListener() {
//            @Override
//            public void onLoginSuccess(UUser user) {
//                Log.d("UGSDKDemo", "sdk login success." + user.getUid());
//                // TODO: 游戏层将uid, token等发给游戏服务器， 游戏服务器再去SDK服务器端做登陆验证。 具体见服务端文档中登陆认证部分。
//            }
//
//            @Override
//            public void onLoginFailed(int code, String msg) {
//                Toast.makeText(MainActivity.this, "登陆失败:" + msg, Toast.LENGTH_LONG).show();
//            }
//        });
    }
});
```

9 结合 onResume() 生命周期方法监听用户登录状态 (选接)
```java
public void onResume() {
    super.onResume();
    XPlatform.getInstance().onResume(this);
    // 可选
    // 用户回到游戏的时候检测用户是否已经登录, 没有登录的情况下可以调用 登录 接口
    if (XPlatform.getInstance().isUserLogin()) {
        Log.d("UGSDKDemo", "The user has logged in.");
    } else {
        // 用户未登录
        Log.d("UGSDKDemo", "The user has not logged in yet.");
        // ⚠️ 注意 ⚠️
        // 调用登录接口之前, 一定要保证 SDK 已经完成了初始化
        // 检测当前 SDK 是否已经完成初始化
        if (XPlatform.getInstance().isSDKInitialized()) {
            XPlatform.getInstance().login(this, new ILoginListener() {
                @Override
                public void onLoginSuccess(UUser user) {
                    Log.d("UGSDKDemo", "sdk login success." + user.getUid());
                }

                @Override
                public void onLoginFailed(int code, String msg) {
                    Toast.makeText(MainActivity.this, "登陆失败:" + msg, Toast.LENGTH_LONG).show();
                }
            });
        }
    }
}
```

10 检测当前用户登录状态
```java
if (XPlatform.getInstance().isUserLogin()) {
    // 用户已登录
    Toast.makeText(this, "用户已经登录", Toast.LENGTH_SHORT).show();
} else {
    Toast.makeText(this, "用户还未登录", Toast.LENGTH_SHORT).show();
}
```


