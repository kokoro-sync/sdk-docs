Android接続ガイド
======

NOTE: このドキュメントでは、ゲームにSDKのAndroidプラットフォームを素早く導入する方法を説明します。

接続準備
-------

1. ファイル説明

SDKDemo/sdk-demo: SDKデモプロジェクト  
    ----libs: SDK関連の依存jar/aar

2. 依存ライブラリの追加

SDK関連の依存ライブラリをゲームプロジェクトディレクトリにコピーしてください。

SDKDemo/sdk-core/libs内の全ファイルをゲームプロジェクトのlibs（依存jar/aar格納ディレクトリ）にコピーします。
main/assets/ug_dev.propertiesファイルも同じディレクトリにコピーしてください。

また、ゲームプロジェクトのbuild.gradleに以下の依存を追加します：
```java
//noinspection GradleCompatible
implementation 'com.android.support:support-fragment:28.0.0'
```

3. AndroidManifest.xmlの設定

以下をAndroidManifest.xmlのapplicationノード内に追加してください：
```xml
<activity-alias
    android:name="${applicationId}.wxapi.WXPayEntryActivity"
    android:exported="true"
    android:targetActivity="com.game.sdk.service.pay.weixin.app.WXPayEntryActivity" />
```

4. ProGuard設定

SDKライブラリはすでにProGuard設定済みですが、ゲームプロジェクトでProGuardを有効にする場合は、以下の設定をproguardファイルに追加してください：
```
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
```
R.javaの削除を防ぐため、以下も追加してください：
-keep public class [アプリのパッケージ名].R$*{
public static final int *;
}

5. Applicationの設定

ゲームプロジェクトに独自のApplicationがない場合、AndroidManifest.xmlのapplicationを**com.game.sdk.app.UGApplication**に指定してください。
独自のApplicationがある場合は、com.game.sdk.app.UGApplicationを継承してください。

必須API
-------

NOTE: すべてのAPI呼び出しはcom.game.sdk.XPlatformシングルトンクラス経由で行います。

1. 初期化（必須）

このメソッドはゲーム起動ActivityのonCreateで呼び出してください。

```java
String appID = "";                                  // TODO: SDK管理画面で発行されたappID
String appKey = "1111111";                          // TODO: SDK管理画面で発行されたappKey
int orientation = UInitParams.ORIENTATION_PORTRAIT;  // TODO: ゲームの画面向き

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

2. ログインAPI（必須）

ログインAPIを呼び出すとSDKのログイン画面が表示されます。SDKは最新の未成年者保護政策に対応しているため、ゲーム側で独自の制限を追加する必要はありません。

```java
XPlatform.getInstance().login(this, new ILoginListener() {
    @Override
    public void onLoginSuccess(UUser user) {
        // ログイン成功時の処理
    }
    @Override
    public void onLoginFailed(int code, String msg) {
        // ログイン失敗時の処理
    }
});
```
// ...以降も同様
