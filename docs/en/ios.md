iOS Integration Guide
======

Integration Preparation
-------

1. Add Dependency Libraries

Copy the `UGSDK.framework` directory to your game project's Xcode project directory, and then drag the entire directory into the Xcode window.


Add system libraries:

```
QuartzCore.framework
WebKit.framework
Security.framework
CoreGraphic.framework
libc++.tbd
libc++abi.tbd
```

Open build settings, configure Other Linker Flags, and add `-ObjC` (case-sensitive).

2. Info.plist Configuration

Merge the following info.plist configuration items into your game project's info.plist:

```xml
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


Note: If you need to enable WeChat login for this game, additional configuration is required. Refer to the document: [WeChat Login Configuration](guide_wx_login_config.md)


3. Interface Call Description

**All interfaces must be called on the UI thread.**

Required Interfaces
-------

NOTE: All interface calls are made through the `UGSDKPlatform` singleton class. Import the header file:

```objectivec
#import <UGSDK/UGSDKPlatform.h>
```

1. Lifecycle Methods

**This method must be called in the corresponding lifecycle method of your game project's `AppDelegate.m`.**

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



2. Interface Callbacks

After calling interfaces such as initialization, login, logout, and payment, the SDK will trigger the corresponding callback functions in `UGSDKDelegate`. You can implement `UGSDKDelegate` to listen for the corresponding callback functions and handle the business logic accordingly.

```objectivec
// Callback after successful initialization
-(void) onUGInitSuccess{
  NSLog(@"sdk init success");
}

// Callback after failed initialization
-(void) onUGInitFailed:(NSString*)msg{
  NSLog(@"sdk init failed.%@", msg);
}

// Callback after successful login
-(void) onUGLoginSuccess:(UG_UserDataModel*)result{
  NSLog(@"sdk login success: %@", result);
}

// Callback after failed login
-(void) onUGLoginFailed:(NSString*)msg{
  NSLog(@"sdk login failed: %@", msg);
}

// Callback after successful logout
- (void)onUGLogoutSuccess:(BOOL)fromUserCenter {
  NSLog(@"logout from sdk");

  if (fromUserCenter) {
    //  If the player logs out from the SDK's floating window, the game needs to return the player to the game's login screen and open the SDK's login screen for the player to log in again.
  }

}

// Callback after failed logout
-(void) onUGLogoutFailed:(NSString*)msg{
  NSLog(@"sdk logout failed");
}

// Callback after successful payment
-(void) onUGPaySuccess:(NSString*)msg{
  NSLog(@"sdk pay success");
}

// Callback after failed payment
-(void) onUGPayFailed:(NSString*)msg{
  NSLog(@"sdk pay failed: %@", msg);
}
```

3. Initialization Interface (Required)

Call the initialization interface, usually when the game starts. All other interface calls must be made after the initialization interface call.

```objectivec
// SDK Initialization
NSString *appID = @"1";                     //appid
NSString *appKey = @"111";                  //appkey
NSString *orientation = @"landscape";       //orientation; landscape or portrait
UG_SDKParams* sdkParams = [[UG_SDKParams alloc] initWithAppID:appID appKey:appKey orientation: orientation];
[[UGSDKPlatform sharedInstance]initWithParams:sdkParams delegate:self];     //The second delegate parameter is the class that implements the UGSDKDelegate above.
```



4. Login Interface (Required)

Call the login interface to open the SDK's login screen.

```objectivec
[[UGSDKPlatform sharedInstance] login];
```

5. Logout Interface (Optional)

Call the logout interface. When you guide the player back to the game's login screen in the game, you can call the logout interface to log the player out of the SDK.

```objectivec
[[UGSDKPlatform sharedInstance] logout];
```

6. Submit Extended Data (Required)

Note: Some channels require that you report player data in the game at specific times, such as creating a character, entering the game, leveling up, and exiting the game, so that the channel backend can collect user data statistics. Be careful not to pass the wrong `UGRoleData.type` for each call timing.

```objectivec
UG_GameRole* role = [[UG_GameRole alloc] init];
role.opType = UG_OP_ENTER_GAME;     //Call timing
role.roleID = @"1";                 //Role ID
role.roleName = @"role_test_1";     //Role name
role.roleLevel = @"1";              //Role level
role.serverID = @"1";               //Server ID
role.serverName = @"server_1";      //Server name
role.vip = @"1";                    //VIP level
role.createTime = 0L;               //Role creation time, Unix timestamp in seconds
role.lastLevelUpTime = 0L;          //Role level up time, Unix timestamp in seconds

[[UGSDKPlatform sharedInstance] submitGameData:role];
```



7. Payment (Required)

Call the payment interface to open the SDK's payment screen.

```objectivec
UG_PayData* data = [[UG_PayData alloc] init];
data.cpOrderID = [self currentTimeStamp];       //Your own order number
data.payNotifyUrl = @"";                        //The callback address for the SDK to notify the game server after successful payment. If not passed, the address configured in the SDK backend will be read.
data.extra = @"test";                           //Extended data, which is returned as is when the SDK notifies the game server with a callback.

data.price = 100;                               //Amount, in cents
data.currency = @"CNY";                         //Currency unit, default is CNY

data.productID = @"2";                          //Product ID in the game. You need to configure the product ID in the game and the product ID configured in the AppStore backend in the SDK backend.
data.productName = @"Test Product";                 //Product name
data.productDesc = @"Buy 500 gold";              //Product description

data.roleID = @"1";                             //Role ID
data.roleName = @"test_role_1";                 //Role name
data.roleLevel = @"1";                          //Role level
data.vip = @"1";                                //Role VIP

data.serverID = @"1";                           //Server ID
data.serverName = @"test_sever_1";              //Server name

[[UGSDKPlatform sharedInstance] pay:data];
```
