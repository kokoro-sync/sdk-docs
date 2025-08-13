iOS接続ガイド
======

NOTE: このドキュメントでは、ゲームに公式SDKのiOSプラットフォームSDKを素早く導入する方法を説明します。SDKはObjective-Cで開発されています。

接続準備
-------

1. 依存ライブラリの追加

UGSDK.frameworkディレクトリをゲームプロジェクトのXcodeプロジェクトディレクトリにコピーし、Xcodeウィンドウにドラッグ＆ドロップしてください。

システムライブラリの追加：
~~~
QuartzCore.framework
WebKit.framework
Security.framework
CoreGraphic.framework
libc++.tbd
libc++abi.tbd
~~~

~~~
build settingを開き、Other Linker Flagsに-ObjC（大文字小文字に注意）を追加してください。
~~~

2. Info Plist設定

以下のInfo.plist設定項目をゲームプロジェクトのInfo.plistに統合してください：
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

注意：ゲームでWeChatログインが必要な場合は、追加設定が必要です。詳細は「guide_wx_login_config.md」を参照してください。

3. インターフェース呼び出しについて

**すべてのインターフェースはUIスレッドで呼び出してください。**

必須API
-------

NOTE: すべてのAPI呼び出しはUGSDKPlatformシングルトンクラス経由で行います。ヘッダーファイルをインポートしてください。
```objectivec
#import <UGSDK/UGSDKPlatform.h>
```

1. ライフサイクル関数

**このメソッドはゲームプロジェクトのAppDelegate.mの対応するライフサイクル関数内で呼び出してください。**

```objectivec
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // その他のロジック
    return [[UGSDKPlatform sharedInstance] application:application didFinishLaunchingWithOptions:launchOptions];
}

- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(nonnull NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
    return [[UGSDKPlatform sharedInstance] application:application openURL:url options:options];
}

-(BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url {
    return [[UGSDKPlatform sharedInstance] application:application handleOpenURL:url];
}

-(BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    return [[UGSDKPlatform sharedInstance] application:application openURL:url sourceApplication:sourceApplication annotation:annotation];
}

-(void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    [[UGSDKPlatform sharedInstance] application:application didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}

-(void)application:(UIApplication *)application didReceiveRemoteNotification:(nonnull NSDictionary *)userInfo {
    [[UGSDKPlatform sharedInstance] application:application didReceiveRemoteNotification:userInfo];
}

-(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler {
    [[UGSDKPlatform sharedInstance] application:application didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
}

-(void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {
    [[UGSDKPlatform sharedInstance] application:application didReceiveLocalNotification:notification];
}

- (void)applicationWillResignActive:(UIApplication *)application {
    [[UGSDKPlatform sharedInstance] applicationWillResignActive:application];
}

- (void)applicationDidEnterBackground:(UIApplication *)application {
    [[UGSDKPlatform sharedInstance] applicationDidEnterBackground:application];
}
// ...以降も同様
```
