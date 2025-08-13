iOS Integration Guide
======

NOTE: This document describes how to quickly integrate the official SDK for iOS platform into your game. The SDK is developed in Objective-C.

Preparation
-------

1. Add dependencies

Copy the UGSDK.framework directory to your game's Xcode project directory, then drag the entire directory into the Xcode window.

Add system libraries:
~~~
QuartzCore.framework
WebKit.framework
Security.framework
CoreGraphic.framework
libc++.tbd
libc++abi.tbd
~~~

~~~
Open build settings and add -ObjC (case sensitive) to Other Linker Flags.
~~~

2. Info Plist Configuration

Merge the following Info.plist configuration items into your game's Info.plist:
~~~plist
    <key>UIUserInterfaceStyle</key>
    <string>Light</string>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <true/>
    </dict>
    <key>ServerDomain</key>
    <string>https://sdk.quebec555.vip/api</string>
~~~

Note: If your game requires WeChat login, additional configuration is needed. See "guide_wx_login_config.md" for details.

3. API Call Instructions

**All APIs must be called on the UI thread.**

Required APIs
-------

NOTE: All API calls are made through the UGSDKPlatform singleton class. Import the header file:
~~~objectivec
#import <UGSDK/UGSDKPlatform.h>
~~~

1. Lifecycle Functions

**This method must be called in the corresponding lifecycle functions of AppDelegate.m in your game project.**

~~~objectivec
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // other logic
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
// ...and so on
~~~~~~
