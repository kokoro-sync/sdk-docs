Android Integration Guide
======

NOTE: This document describes how to quickly integrate the SDK for Android platform into your game.

Preparation
-------

1. File Description

SDKDemo/sdk-demo: SDK demo project
    ----libs: All SDK-related dependency jars/aars

2. Add dependencies

Copy all SDK-related dependencies to your game's project directory.

Copy all files from SDKDemo/sdk-core/libs to your game's libs directory (where jars/aars are stored).
Also copy main/assets/ug_dev.properties to the same directory in your project.

Add the following dependency to your game's build.gradle:
~~~
//noinspection GradleCompatible
implementation 'com.android.support:support-fragment:28.0.0'
~~~

3. AndroidManifest.xml Configuration

Add the following to the application node in AndroidManifest.xml:
```xml
<activity-alias
    android:name="${applicationId}.wxapi.WXPayEntryActivity"
    android:exported="true"
    android:targetActivity="com.game.sdk.service.pay.weixin.app.WXPayEntryActivity" />
```

4. Proguard (Obfuscation) Configuration

The SDK libraries are already obfuscated, but if your game project enables obfuscation, add the following rules to your proguard file:
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
To prevent R.java from being removed, also add:
-keep public class [your app package name].R$*{
public static final int *;
}

5. Application Configuration

If your game does not have its own Application, set the application in AndroidManifest.xml to **com.game.sdk.app.UGApplication**.
If you have your own Application, inherit from com.game.sdk.app.UGApplication.

Required APIs
-------

NOTE: All API calls are made through the com.game.sdk.XPlatform singleton class.

1. Initialization (Required)

This method must be called in the onCreate of your game's launch Activity.

```java
String appID = "";                                  // TODO: Set to appID assigned by SDK backend
String appKey = "1111111";                          // TODO: Set to appKey assigned by SDK backend
int orientation = UInitParams.ORIENTATION_PORTRAIT;  // TODO: Set to game orientation

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

2. Login API (Required)

Call the login API to open the SDK login screen. The SDK implements the latest anti-addiction policy for minors, so you do not need to add your own restrictions.

```java
XPlatform.getInstance().login(this, new ILoginListener() {
    @Override
    public void onLoginSuccess(UUser user) {
        // Handle login success
    }
    @Override
    public void onLoginFailed(int code, String msg) {
        // Handle login failure
    }
});
```
// ...and so on
