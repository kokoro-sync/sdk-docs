iOS向け開発ガイド
======

統合の準備
-------

1. 依存ライブラリの追加

`UGSDK.framework` ディレクトリをゲームプロジェクトのXcodeプロジェクトディレクトリにコピーし、ディレクトリ全体をXcodeウィンドウにドラッグします。


システムライブラリの追加：

```
QuartzCore.framework
WebKit.framework
Security.framework
CoreGraphic.framework
libc++.tbd
libc++abi.tbd
```

build settingを開き、Other Linker Flagsを設定し、`-ObjC`を追加します（大文字と小文字を区別します）。

2. Info.plistの設定

以下のinfo.plist設定項目をゲームプロジェクトのinfo.plistにマージします。

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


注意：このゲームでWeChatログインを有効にする必要がある場合は、追加の設定が必要です。ドキュメントを参照してください：[WeChatログイン設定](guide_wx_login_config.md)


3. インターフェース呼び出しの説明

**すべてのインターフェースは、UIスレッドで呼び出す必要があります。**

必須インターフェース
-------

NOTE: すべてのインターフェース呼び出しは、`UGSDKPlatform`シングルトンクラスを介して行われます。ヘッダーファイルをインポートします。

```objectivec
#import <UGSDK/UGSDKPlatform.h>
```

1. ライフサイクルメソッド

**このメソッドは、ゲームプロジェクトの`AppDelegate.m`の対応するライフサイクルメソッドで呼び出す必要があります。**

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



2. インターフェースコールバック

初期化、ログイン、ログアウト、決済などのインターフェース呼び出し後、SDKは`UGSDKDelegate`の対応するコールバック関数をトリガーします。`UGSDKDelegate`を実装して、対応するコールバック関数をリッスンし、ビジネスロジックを処理できます。

```objectivec
// 初期化成功時のコールバック
-(void) onUGInitSuccess{
  NSLog(@"sdk init success");
}

// 初期化失敗時のコールバック
-(void) onUGInitFailed:(NSString*)msg{
  NSLog(@"sdk init failed.%@", msg);
}

// ログイン成功時のコールバック
-(void) onUGLoginSuccess:(UG_UserDataModel*)result{
  NSLog(@"sdk login success: %@", result);
}

// ログイン失敗時のコールバック
-(void) onUGLoginFailed:(NSString*)msg{
  NSLog(@"sdk login failed: %@", msg);
}

// ログアウト成功時のコールバック
- (void)onUGLogoutSuccess:(BOOL)fromUserCenter {
  NSLog(@"logout from sdk");

  if (fromUserCenter) {
    //  プレイヤーがSDKのフローティングウィンドウからアカウントをログアウトした場合、ゲームはプレイヤーをゲームのログイン画面に戻し、SDKのログイン画面を開いて再ログインさせる必要があります。
  }

}

// ログアウト失敗時のコールバック
-(void) onUGLogoutFailed:(NSString*)msg{
  NSLog(@"sdk logout failed");
}

// 決済成功時のコールバック
-(void) onUGPaySuccess:(NSString*)msg{
  NSLog(@"sdk pay success");
}

// 決済失敗時のコールバック
-(void) onUGPayFailed:(NSString*)msg{
  NSLog(@"sdk pay failed: %@", msg);
}
```

3. 初期化インターフェース（必須）

初期化インターフェースを呼び出します。通常、ゲームの起動時に呼び出されます。他のすべてのインターフェース呼び出しは、初期化インターフェース呼び出し後に行う必要があります。

```objectivec
// SDKの初期化
NSString *appID = @"1";                     //appid
NSString *appKey = @"111";                  //appkey
NSString *orientation = @"landscape";       //向き；横向き：landscape; 縦向き：portrait
UG_SDKParams* sdkParams = [[UG_SDKParams alloc] initWithAppID:appID appKey:appKey orientation: orientation];
[[UGSDKPlatform sharedInstance]initWithParams:sdkParams delegate:self];     //2番目のdelegateパラメータは、上記のUGSDKDelegateを実装したクラスです。
```



4. ログインインターフェース(必須)

ログインインターフェースを呼び出して、SDKのログイン画面を開きます。

```objectivec
[[UGSDKPlatform sharedInstance] login];
```

5. ログアウトインターフェース(任意)

ログアウトインターフェースを呼び出します。ゲーム内でプレイヤーをゲームのログイン画面に戻すように誘導する場合、ログアウトインターフェースを呼び出して、プレイヤーをSDKからログアウトさせることができます。

```objectivec
[[UGSDKPlatform sharedInstance] logout];
```

6. 拡張データの送信(必須)

Note: 一部のチャネルでは、キャラクターの作成、ゲームへの参加、キャラクターのレベルアップ、ゲームの終了などの時点で、ゲーム内のプレイヤーデータを報告する必要があります。これにより、チャネルのバックエンドでユーザーデータを統計できます。各呼び出しのタイミングで、`UGRoleData.type`を間違えないように注意してください。

```objectivec
UG_GameRole* role = [[UG_GameRole alloc] init];
role.opType = UG_OP_ENTER_GAME;     //呼び出しタイミング
role.roleID = @"1";                 //キャラクターID
role.roleName = @"role_test_1";     //キャラクター名
role.roleLevel = @"1";              //キャラクターレベル
role.serverID = @"1";               //サーバーID
role.serverName = @"server_1";      //サーバー名
role.vip = @"1";                    //VIPレベル
role.createTime = 0L;               //キャラクター作成時間、Unixタイムスタンプ（秒）
role.lastLevelUpTime = 0L;          //キャラクターレベルアップ時間、Unixタイムスタンプ（秒）

[[UGSDKPlatform sharedInstance] submitGameData:role];
```



7. 決済(必須)

決済インターフェースを呼び出して、SDKの決済画面を開きます。

```objectivec
UG_PayData* data = [[UG_PayData alloc] init];
data.cpOrderID = [self currentTimeStamp];       //ゲーム独自の注文番号
data.payNotifyUrl = @"";                        //決済成功後、SDKがゲームサーバーに通知するコールバックアドレス。渡されない場合は、SDKバックエンドで設定されたアドレスを読み取ります。
data.extra = @"test";                           //拡張データ。SDKがゲームサーバーにコールバック通知する際に、そのまま返されます。

data.price = 100;                               //金額、単位：銭
data.currency = @"CNY";                         //通貨単位、デフォルトはCNY

data.productID = @"2";                          //ゲーム内の商品ID。SDKバックエンドでゲーム内の商品IDとAppStoreバックエンドで設定された商品IDを設定する必要があります。
data.productName = @"テスト商品";                 //商品名
data.productDesc = @"500元宝を購入";              //商品説明

data.roleID = @"1";                             //キャラクターID
data.roleName = @"test_role_1";                 //キャラクター名
data.roleLevel = @"1";                          //キャラクターレベル
data.vip = @"1";                                //キャラクターVIP

data.serverID = @"1";                           //サーバーID
data.serverName = @"test_sever_1";              //サーバー名

[[UGSDKPlatform sharedInstance] pay:data];
```
