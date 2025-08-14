Android向けSDK組込みガイド
======

統合の準備
-------

**1. ファイル構成**

```
SDKDemo/sdk-demo: SDKデモプロジェクト
        ----libs: SDKに関連するすべての依存jarおよびaar
```

**2. 依存関係の追加**

2.1 依存ライブラリの追加

SDKに関連する依存ライブラリをゲームプロジェクトのディレクトリにコピーします。

- `SDKDemo/sdk-core/libs` のすべてのファイルを、ゲームプロジェクトのlibs（依存jarパッケージ、aarなどのディレクトリの場所）にコピーします。  
- `main/assets/ug_dev.properties` ファイルをプロジェクトの同じディレクトリにコピーします。

ゲームプロジェクトのbuild.gradleに、以下の依存ライブラリを追加します。

```gradle
//noinspection GradleCompatible
implementation 'com.android.support:support-fragment:28.0.0'
```

2.3 AndroidManifest.xmlの設定

以下のスニペットをコピーして、ゲームのパッケージ化プロジェクトのAndroidManifest.xmlのapplicationノード内に貼り付けます。
```xml
<activity-alias
    android:name="${applicationId}.wxapi.WXPayEntryActivity"
    android:exported="true"
    android:targetActivity="com.game.sdk.service.pay.weixin.app.WXPayEntryActivity" />
```

2.4 難読化ルールの設定

SDKライブラリはすでに難読化されているため、ゲームプロジェクトで難読化を有効にしている場合は、難読化設定ファイルに以下のルールを追加して、SDKのAPIを無視するようにしてください。

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
SDKは、インポートされたプロジェクトのリソースファイルを参照する必要があり、リフレクションメカニズムを介してリソース参照ファイルR.javaを取得します。ただし、開発者がproguardなどの難読化/最適化ツールを使用してapkを処理すると、proguardがR.javaを削除する可能性があります。この問題が発生した場合は、次の設定を追加してください。

```
-keep public class [あなたのアプリケーションパッケージ名].R$*{
	public static final int *;
}
```

Applicationの設定
-------

ゲーム層に独自のApplicationがない場合、ゲームはAndroidManifest.xmlのapplicationを **com.game.sdk.app.UGApplication** に指定する必要があります。

ゲームに独自のApplicationがある場合は、com.game.sdk.app.UGApplicationクラスを直接継承できます。


必須インターフェース
-------

NOTE: すべてのインターフェース呼び出しは、com.game.sdk.XPlatformシングルトンクラスを介して行われます。

1. 初期化（必須）

**このメソッドは、ゲームの起動ActivityのonCreateメソッドで呼び出す必要があります。**

```java
String appID = ""; // TODO: SDKバックエンドで割り当てられたappIDパラメータに設定
String appKey = "1111111"; // TODO: SDKバックエンドで割り当てられたappKeyパラメータに設定
int orientation = UInitParams.ORIENTATION_PORTRAIT; //TODO: ゲームの向き（縦/横）に設定

XPlatform.getInstance().init(this, new UInitParams(appID, appKey, orientation), new IInitListener() {
  @Override
  public void onInitFailed(int code, String msg) {
    Log.e("UGSDKDemo", "sdk init failed. code:" + code + ";msg:" + msg);
  }

  @Override
  public void onInitSuccess() {
    Log.d("UGSDKDemo", "sdk init success");
  }
});
```

2. ログインインターフェース（必須）

ログインインターフェースを呼び出して、SDKのログイン画面を開きます。SDKは、国の最新の未成年者保護ポリシーに基づいて、未成年者の依存症防止制限を実装しています。ゲーム内で独自の依存症防止制限を追加する必要はありません。

```java
XPlatform.getInstance().login(this, new ILoginListener() {
  @Override
  public void onLoginSuccess(UUser user) {
    Log.d("UGSDKDemo", "sdk login success." + user.getUid());
    // TODO: ゲーム層はuid、tokenなどをゲームサーバーに送信し、ゲームサーバーはSDKサーバー側でログイン検証を行います。詳細については、サーバーサイドドキュメントのログイン認証部分を参照してください。
  }

  @Override
  public void onLoginFailed(int code, String msg) {
    Toast.makeText(MainActivity.this, "ログイン失敗:" + msg, Toast.LENGTH_LONG).show();
  }
});
```

3. ログアウトインターフェース（任意）

ログアウトインターフェースを呼び出すと、SDKアカウントがログアウトします。ただし、すべてのSDKにログアウトロジックがあるわけではありません。ログアウトインターフェースを提供していないSDKの場合、この関数を呼び出しても何も起こりません。

```java
XPlatform.getInstance().logout(this);
```

4. 拡張データの送信（必須）

Note: 一部のチャネルでは、キャラクターの作成、ゲームへのログイン、キャラクターのレベルアップ、ゲームの終了などの時点で、ゲーム内のプレイヤーデータを報告する必要があります。これにより、チャネルのバックエンドでユーザーデータを統計できます。したがって、ゲーム層は特定の時点でこのメソッドを複数回呼び出す必要があります。

```java
URoleData data = new URoleData();
data.setType(type);
data.setServerID("2");
data.setServerName("test_1");
data.setRoleID("1");
data.setRoleName("test_role_1");
data.setRoleLevel("1");
data.setVip("1");
data.setCreateTime(System.currentTimeMillis() / 1000);
data.setLastLevelUpTime(data.getCreateTime());

XPlatform.getInstance().submit(this, data);
```

このメソッドは、呼び出しのタイミングをいくつかのタイプに分類します。

1. キャラクター作成
2. ゲーム参加
3. レベルアップ
4. ゲーム終了

したがって、上記の4つのタイミングで、`XPlatform.getInstance().submit(URoleData data)` を呼び出す必要があります。

`URoleData` はゲーム内のプレイヤーデータであり、構造内のパラメータはnullであってはなりません。値がない場合は、デフォルト値0を渡してください。呼び出しのタイミングは `URoleData.dataType` で区別されます。キャラクター作成時は1、ゲーム参加時は2、レベルアップ時は3、ゲーム終了時は4です。

UserExtraDataのデータ構造について:

| パラメータ名        | パラメータ型          | パラメータ説明  |
|:------------- |:-------------|:-----|
| dataType     | int | 呼び出しタイミング |
| serverID| String| プレイヤーがいるサーバーのID |
| serverName| String| プレイヤーがいるサーバーの名前 |
| roleID | String | プレイヤーのキャラクターID |
| roleName| String | プレイヤーのキャラクター名 |
| roleLevel| String | プレイヤーのキャラクターレベル |
| moneyNum| String | 現在のキャラクターが所持しているゲーム内通貨の数量 |
| createTime| long | キャラクター作成時間（1970年からの秒単位）。実際のデータを渡さないとUCの審査に通りません |
| lastLevelUpTime| long | キャラクターレベル変更時間（1970年からの秒単位） |
| vip| String | プレイヤーのVIPレベル |
| extraData| String | 添付データ（任意） |


6. 決済（必須）

決済インターフェースを呼び出して、SDKの決済画面を開きます。

```java
UOrder order = new UOrder();
order.setProductID("1");
order.setProductName("テスト商品");
order.setProductDesc("テスト商品の説明");
order.setRoleID("1");
order.setRoleName("test_role_1");
order.setRoleLevel("1");
order.setVip("1");
order.setServerID("2");
order.setServerName("test_1");
order.setPrice(64800);          // 金額単位：銭
order.setCurrency("CNY");
order.setCpOrderID(System.currentTimeMillis() + "");
order.setExtra("extra data");
XPlatform.getInstance().pay(this, order, new IPayListener() {
  @Override
  public void onPaySuccess(UOrder order) {}

  @Override
  public void onPayFailed(UOrder order, String msg) {}

  @Override
  public void onPayCanceled(UOrder order) {}
});
```

PayParamsオブジェクトについて：

| パラメータ名        | パラメータ型          | パラメータ説明  |
|:------------- |:-------------|:-----|
| productID     | String | 課金アイテムID、ゲーム内のアイテムID |
| productName      | String      |   アイテム名、例：100元宝、500ダイヤ... |
| productDesc| String      |    アイテム説明、例：100元宝チャージで20元宝プレゼント |
| price| int | 課金額（単位：銭） |
| currency| String | 通貨単位、固定値CNY |
| cpOrderID| String | ゲーム注文番号 |
| serverID| String| プレイヤーがいるサーバーのID |
| serverName| String| プレイヤーがいるサーバーの名前 |
| roleID | String | プレイヤーのキャラクターID |
| roleName| String | プレイヤーのキャラクター名 |
| roleLevel| String | プレイヤーのキャラクターレベル |
| vip | String| プレイヤーのVIPレベル |
| payNotifyUrl| String | ゲームサーバーの決済コールバックアドレス。SDKの決済が成功すると、このアドレスに非同期で通知され、ゲームサーバーがプレイヤーにアイテムを配布します |
| extra | String | カスタムデータ。決済コールバックがゲームサーバーに通知される際に、そのまま返されます |

7. ライフサイクルメソッド（必須）
-------

ゲームのメインActivityの以下のライフサイクルメソッドで、対応するメソッドを呼び出します。

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

8. ユーザーログアウトイベントリスナー（任意）
```java
XPlatform.getInstance().setLogoutListener(new ILogoutListener() {
  @Override
  public void onLogout() {
    //TODO: ゲーム層はここでプレイヤーをゲームのログイン画面に戻し、再ログインさせる必要があります
    Log.d("UGSDKDemo", "sdk logout. game to logout");
  }
});
```

ゲームに独自のログインページがない場合、login()メソッドを呼び出してログインダイアログを表示できます(任意)
```java
XPlatform.getInstance().setLogoutListener(new ILogoutListener() {
  @Override
  public void onLogout() {
    XPlatform.getInstance().login(MainActivity.this, new ILoginListener() {
      @Override
      public void onLoginSuccess(UUser user) {
        Log.d("UGSDKDemo", "sdk login success." + user.getUid());
        // TODO: ゲーム層はuid、tokenなどをゲームサーバーに送信し、ゲームサーバーはSDKサーバー側でログイン検証を行います。詳細については、サーバーサイドドキュメントのログイン認証部分を参照してください。
      }

      @Override
      public void onLoginFailed(int code, String msg) {
        Toast.makeText(MainActivity.this, "ログイン失敗:" + msg, Toast.LENGTH_LONG).show();
      }
    });
  }
});
```

9. onResume()ライフサイクルメソッドと組み合わせてログイン状態をリッスンする（任意）  

ユーザーがゲームに戻ったときにログインしているかどうかを確認し、ログインしていない場合はログインインターフェースを呼び出すことができます
```java
public void onResume() {
  super.onResume();
  XPlatform.getInstance().onResume(this);
  if (XPlatform.getInstance().isUserLogin()) {
    Log.d("UGSDKDemo", "The user has logged in.");
  } else {
    Log.d("UGSDKDemo", "The user has not logged in yet.");
    // ⚠️ 注意 ⚠️
    // ログインインターフェースを呼び出す前に、SDKが初期化されていることを確認してください
    // 現在のSDKが初期化されているかどうかを確認します
    if (XPlatform.getInstance().isSDKInitialized()) {
      XPlatform.getInstance().login(this, new ILoginListener() {
        @Override
        public void onLoginSuccess(UUser user) {
          Log.d("UGSDKDemo", "sdk login success." + user.getUid());
        }

        @Override
        public void onLoginFailed(int code, String msg) {
          Toast.makeText(MainActivity.this, "ログイン失敗:" + msg, Toast.LENGTH_LONG).show();
        }
      });
    }
  }
}
```

10. 現在のユーザーのログイン状態を確認する
```java
if (XPlatform.getInstance().isUserLogin()) {
  Toast.makeText(this, "ユーザーはすでにログインしています", Toast.LENGTH_SHORT).show();
} else {
  Toast.makeText(this, "ユーザーはまだログインしていません", Toast.LENGTH_SHORT).show();
}
```
