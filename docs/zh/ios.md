iOS接入指南
======

NOTE:这篇文档，介绍游戏中怎么快速完成官网SDK iOS平台SDK的接入，SDK采用objective-c开发。


接入准备
-------

1、添加依赖库

将 UGSDK.framework目录，拷贝到游戏项目xcode工程目录下， 然后将整个目录拖到xcode窗口中


添加系统库：

```
QuartzCore.framework
WebKit.framework
Security.framework
CoreGraphic.framework
libc++.tbd
libc++abi.tbd
```

```
打开build setting ，配置Other Linker Flags ,添加 -ObjC （注意大小写）
```

2、Info Plist配置

将如下info plist 配置项合并到游戏工程的info plist中：

```plist
  <key>UIUserInterfaceStyle</key>
  <string>Light</string>
  <key>NSAppTransportSecurity</key>
  <dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
  </dict>
  <key>ServerDomain</key>
  <string>https://sdk.quebec555.vip/api</string>
```


注意： 如果该游戏需要开微信登录，则需要进行一些额外的配置，参考文档：[微信登录配置](guide_wx_login_config.md)


3、接口调用说明

**所有接口，必须在UI线程中调用。**

必须调用的接口
-------

NOTE:所有接口调用，都通过UGSDKPlatform 单例类来调用。 引入头文件

```objectivec
#import <UGSDK/UGSDKPlatform.h>
```

1、生命周期函数

**该方法必须在游戏工程的AppDelegate.m中对应生命周期函数中调用**

```objectivec
@implementation AppDelegate
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  // other logic
  return [[UGSDKPlatform sharedInstance] application:application didFinishLaunchingWithOptions:launchOptions];
}

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(nonnull NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
  return [[UGSDKPlatform sharedInstance] application:application openURL:url options:options];
}

-(BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url
{
  return [[UGSDKPlatform sharedInstance] application:application handleOpenURL:url];
}

-(BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
  return [[UGSDKPlatform sharedInstance] application:application openURL:url sourceApplication:sourceApplication annotation:annotation];
}

-(void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
  [[UGSDKPlatform sharedInstance] application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}

-(void)application:(UIApplication *)application didReceiveRemoteNotification:(nonnull NSDictionary *)userInfo
{
  [[UGSDKPlatform sharedInstance] application:application didReceiveRemoteNotification:userInfo];
}

-(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
  [[UGSDKPlatform sharedInstance] application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
}

-(void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
  [[UGSDKPlatform sharedInstance] application:application didReceiveLocalNotification:notification];
}

- (void)applicationWillResignActive:(UIApplication *)application
{
  [[UGSDKPlatform sharedInstance] applicationWillResignActive:application];
}

- (void)applicationDidEnterBackground:(UIApplication *)application
{
  [[UGSDKPlatform sharedInstance] applicationDidEnterBackground:application];
}

- (void)applicationWillEnterForeground:(UIApplication *)application
{
  [[UGSDKPlatform sharedInstance] applicationWillEnterForeground:application];
}

- (void)applicationDidBecomeActive:(UIApplication *)application
{
  [[UGSDKPlatform sharedInstance] applicationDidBecomeActive:application];
}

- (void)applicationWillTerminate:(UIApplication *)application
{
  [[UGSDKPlatform sharedInstance] applicationWillTerminate:application];
}

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler
{
  [[UGSDKPlatform sharedInstance] application:application continueUserActivity:userActivity restorationHandler:restorationHandler];
  return YES;
}
```



2、接口回调

当初始化、登录、登出、支付等接口调用之后， 响应的调用结果，SDK会触发UGSDKDelegate中的对应的回调函数， 您可以实现UGSDKDelegate来监听对应的回调函数，并针对性地处理业务逻辑。

```objectivec
// 初始化成功后回调
-(void) onUGInitSuccess{
  NSLog(@"sdk init success");
}

// 初始化失败后回调
-(void) onUGInitFailed:(NSString*)msg{
  NSLog(@"sdk init failed.%@", msg);
}

// 登录成功后回调
-(void) onUGLoginSuccess:(UG_UserDataModel*)result{
  NSLog(@"sdk login success: %@", result);
}

// 登录失败后回调
-(void) onUGLoginFailed:(NSString*)msg{
  NSLog(@"sdk login failed: %@", msg);
}

// 登出成功后回调
- (void)onUGLogoutSuccess:(BOOL)fromUserCenter {
  NSLog(@"logout from sdk");

  if (fromUserCenter) {
    //  说明玩家从SDK的悬浮窗中点击了登出账号，游戏这里需要让玩家返回到游戏登录界面，并打开SDK的登录界面，让玩家重新登录
  }

}

// 登出失败后回调
-(void) onUGLogoutFailed:(NSString*)msg{
  NSLog(@"sdk logout failed");
}

// 支付成功后回调
-(void) onUGPaySuccess:(NSString*)msg{
  NSLog(@"sdk pay success");
}

// 支付失败后回调
-(void) onUGPayFailed:(NSString*)msg{
  NSLog(@"sdk pay failed: %@", msg);
}
```

3、初始化接口（必接）

调用初始化接口，一般在游戏启动的时候调用。 后续其他接口的调用都必须在初始化接口调用之后进行。

```objectivec
  // SDK 初始化
  NSString *appID = @"1";                     //appid
  NSString *appKey = @"111";                  //appkey
  NSString *orientation = @"landscape";       //横竖屏；横屏：landscape; 竖屏：portrait
  UG_SDKParams* sdkParams = [[UG_SDKParams alloc] initWithAppID:appID appKey:appKey orientation: orientation];
  [[UGSDKPlatform sharedInstance]initWithParams:sdkParams delegate:self];     //第二个delegate参数，就是实现了上面UGSDKDelegate的类
```



4、登录接口(必接)

调用登录接口，打开SDK登录界面。

```objectivec
  [[UGSDKPlatform sharedInstance] login];
```

5、登出接口(选接)

调用登出接口， 游戏在游戏内引导玩家返回到游戏登陆界面时，可以调用登出接口，让玩家登出SDK。

```objectivec
  [[UGSDKPlatform sharedInstance] logout];
```

6、提交扩展数据(必接)

Note: 部分渠道要求在 创建角色，进入游戏，角色升级，退出游戏 等时刻，必须要上报游戏中玩家数据，以便渠道后台统计用户数据。 每个调用时机，UGRoleData.type注意别传错了。

```objectivec
  UG_GameRole* role = [[UG_GameRole alloc] init];
  role.opType = UG_OP_ENTER_GAME;     //调用时机
  role.roleID = @"1";                 //角色ID
  role.roleName = @"role_test_1";     //角色名称
  role.roleLevel = @"1";              //角色等级
  role.serverID = @"1";               //服务器ID
  role.serverName = @"server_1";      //服务器名称
  role.vip = @"1";                    //VIP等级
  role.createTime = 0L;               //角色创建时间，Unix时间戳 秒
  role.lastLevelUpTime = 0L;          //角色升级时间, Unix时间戳 秒

  [[UGSDKPlatform sharedInstance] submitGameData:role];
```



7、支付充值(必接)

调用充值接口，打开SDK充值界面。

```objectivec
  UG_PayData* data = [[UG_PayData alloc] init];
  data.cpOrderID = [self currentTimeStamp];       //游戏自己的订单号
  data.payNotifyUrl = @"";                        //支付成功，SDK通知给游戏服务器的回调地址，如果不传，读取SDK后台配置的地址
  data.extra = @"test";                           //扩展数据，SDK回调通知给游戏服务器的时候，原样返回

  data.price = 100;                               //金额， 单位：分
  data.currency = @"CNY";                         //货币单位，默认CNY

  data.productID = @"2";                          //游戏中商品ID，需要在SDK后台配置游戏中商品ID和AppStore后台配置的商品ID
  data.productName = @"测试商品";                 //商品名称
  data.productDesc = @"购买500元宝";              //商品描述

  data.roleID = @"1";                             //角色ID
  data.roleName = @"test_role_1";                 //角色名称
  data.roleLevel = @"1";                          //角色等级
  data.vip = @"1";                                //角色VIP

  data.serverID = @"1";                           //服务器ID
  data.serverName = @"test_sever_1";              //服务器名称

  [[UGSDKPlatform sharedInstance] pay:data];
```
