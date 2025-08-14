Android SDK Integration Guide
======

Integration Preparation
-------

**1. File Structure**

```
SDKDemo/sdk-demo: SDK Demo Project
        ----libs: All dependency jars and aars related to the SDK
```

**2. Add Dependencies**

2.1 Add Dependency Libraries

Copy the SDK-related dependency libraries to your game project directory.

- Copy all files from `SDKDemo/sdk-core/libs` to the libs directory of your game project (where dependency jar packages, aars, etc. are stored).
- Copy the `main/assets/ug_dev.properties` file to the same directory in your project.

Add the following dependency library to your game project's build.gradle:

```gradle
//noinspection GradleCompatible
implementation 'com.android.support:support-fragment:28.0.0'
```

2.3 Configure AndroidManifest.xml

Copy and paste the following snippet into the application node of your game's packaging project's AndroidManifest.xml:
```xml
<activity-alias
    android:name="${applicationId}.wxapi.WXPayEntryActivity"
    android:exported="true"
    android:targetActivity="com.game.sdk.service.pay.weixin.app.WXPayEntryActivity" />
```

2.4 Configure Proguard Rules

Since the SDK library has already been obfuscated, if your game project has obfuscation enabled, please add the following rules to your proguard configuration file to ignore the SDK's APIs:

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
The SDK needs to reference the resource files of the imported project and obtains the resource reference file R.java through the reflection mechanism. However, when developers use obfuscation/optimization tools such as proguard to process the apk, proguard may delete R.java. If you encounter this problem, please add the following configuration:

```
-keep public class [your application package name].R$*{
	public static final int *;
}
```

Configure Application
-------

If the game layer does not have its own Application, the game needs to specify the application in AndroidManifest.xml as **com.game.sdk.app.UGApplication**.

If the game has its own Application, it can directly inherit the com.game.sdk.app.UGApplication class.


Required Interfaces
-------

NOTE: All interface calls are made through the com.game.sdk.XPlatform singleton class.

1. Initialization (Required)

**This method must be called in the onCreate method of the game's startup Activity.**

```java
String appID = ""; // TODO: Set to the appID parameter assigned by the SDK backend
String appKey = "1111111"; // TODO: Set to the appKey parameter assigned by the SDK backend
int orientation = UInitParams.ORIENTATION_PORTRAIT; //TODO: Set to the game's orientation (portrait/landscape)

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

2. Login Interface (Required)

Call the login interface to open the SDK's login screen. The SDK implements anti-addiction restrictions for minors based on the country's latest policies. You do not need to add your own anti-addiction restrictions in the game.

```java
XPlatform.getInstance().login(this, new ILoginListener() {
  @Override
  public void onLoginSuccess(UUser user) {
    Log.d("UGSDKDemo", "sdk login success." + user.getUid());
    // TODO: The game layer sends the uid, token, etc. to the game server, and the game server performs login verification on the SDK server side. For details, see the login authentication section of the server-side documentation.
  }

  @Override
  public void onLoginFailed(int code, String msg) {
    Toast.makeText(MainActivity.this, "Login failed:" + msg, Toast.LENGTH_LONG).show();
  }
});
```

3. Logout Interface (Optional)

Call the logout interface to log out of the SDK account. However, not all SDKs have logout logic. For SDKs that do not provide a logout interface, calling this function does nothing.

```java
XPlatform.getInstance().logout(this);
```

4. Submit Extended Data (Required)

Note: Some channels require reporting player data in the game at specific times, such as creating a character, logging into the game, leveling up a character, and exiting the game. This allows the channel's backend to collect user data statistics. Therefore, the game layer needs to call this method multiple times at specific points.

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

This method classifies the timing of the call into several types:

1. Create character
2. Enter game
3. Level up
4. Exit game

Therefore, you need to call `XPlatform.getInstance().submit(URoleData data)` at the four points above.

`URoleData` is the player data in the game, and the parameters in the structure cannot be null. If there is no value, please pass the default value 0. The timing of the call is distinguished by `URoleData.dataType`. It is 1 when creating a character, 2 when entering the game, 3 when leveling up, and 4 when exiting the game.

About the UserExtraData data structure:

| Parameter Name        | Parameter Type          | Parameter Description  |
|:------------- |:-------------|:-----|
| dataType     | int | Timing of the call |
| serverID| String| The ID of the server the player is on |
| serverName| String| The name of the server the player is on |
| roleID | String | Player's character ID |
| roleName| String | Player's character name |
| roleLevel| String | Player's character level |
| moneyNum| String | The amount of in-game currency the current character has |
| createTime| long | Character creation time (in seconds since 1970). You must pass the real data, otherwise the UC review will not pass. |
| lastLevelUpTime| long | Character level change time (in seconds since 1970) |
| vip| String | Player's VIP level |
| extraData| String | Attached data (optional) |


6. Payment (Required)

Call the payment interface to open the SDK's payment screen.

```java
UOrder order = new UOrder();
order.setProductID("1");
order.setProductName("Test Product");
order.setProductDesc("Test Product Description");
order.setRoleID("1");
order.setRoleName("test_role_1");
order.setRoleLevel("1");
order.setVip("1");
order.setServerID("2");
order.setServerName("test_1");
order.setPrice(64800);          // Amount unit: cents
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

About the PayParams object:

| Parameter Name        | Parameter Type          | Parameter Description  |
|:------------- |:-------------|:-----|
| productID     | String | Recharge product ID, the product ID within the game |
| productName      | String      |   Product name, e.g., 100 gold, 500 diamonds... |
| productDesc| String      |    Product description, e.g., Recharge 100 gold, get 20 gold free |
| price| int | Recharge amount (in cents) |
| currency| String | Currency unit, fixed value CNY |
| cpOrderID| String | Game order number |
| serverID| String| The ID of the server the player is on |
| serverName| String| The name of the server the player is on |
| roleID | String | Player's character ID |
| roleName| String | Player's character name |
| roleLevel| String | Player's character level |
| vip | String| Player's VIP level |
| payNotifyUrl| String | Game server payment callback address. After the SDK payment is successful, this address is notified asynchronously to let the game server deliver the goods to the player |
| extra | String | Custom data. When the payment callback is notified to the game server, this value is returned as is |

7. Lifecycle Methods (Required)
-------

In the following lifecycle methods of the game's main Activity, call the corresponding methods.

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

8. User Logout Event Listener (Optional)
```java
XPlatform.getInstance().setLogoutListener(new ILogoutListener() {
  @Override
  public void onLogout() {
    //TODO: The game layer needs to return the player to the game login screen to log in again
    Log.d("UGSDKDemo", "sdk logout. game to logout");
  }
});
```

If the game does not have its own login page, you can call the login() method to display the login dialog (optional)
```java
XPlatform.getInstance().setLogoutListener(new ILogoutListener() {
  @Override
  public void onLogout() {
    XPlatform.getInstance().login(MainActivity.this, new ILoginListener() {
      @Override
      public void onLoginSuccess(UUser user) {
        Log.d("UGSDKDemo", "sdk login success." + user.getUid());
        // TODO: The game layer sends the uid, token, etc. to the game server, and the game server performs login verification on the SDK server side. For details, see the login authentication section of the server-side documentation.
      }

      @Override
      public void onLoginFailed(int code, String msg) {
        Toast.makeText(MainActivity.this, "Login failed:" + msg, Toast.LENGTH_LONG).show();
      }
    });
  }
});
```

9. Listen for User Login Status in Conjunction with the onResume() Lifecycle Method (Optional)

Check if the user is logged in when they return to the game, and call the login interface if they are not
```java
public void onResume() {
  super.onResume();
  XPlatform.getInstance().onResume(this);
  if (XPlatform.getInstance().isUserLogin()) {
    Log.d("UGSDKDemo", "The user has logged in.");
  } else {
    Log.d("UGSDKDemo", "The user has not logged in yet.");
    // ⚠️ Note ⚠️
    // Before calling the login interface, make sure the SDK has been initialized
    // Check if the current SDK has been initialized
    if (XPlatform.getInstance().isSDKInitialized()) {
      XPlatform.getInstance().login(this, new ILoginListener() {
        @Override
        public void onLoginSuccess(UUser user) {
          Log.d("UGSDKDemo", "sdk login success." + user.getUid());
        }

        @Override
        public void onLoginFailed(int code, String msg) {
          Toast.makeText(MainActivity.this, "Login failed:" + msg, Toast.LENGTH_LONG).show();
        }
      });
    }
  }
}
```

10. Check Current User Login Status
```java
if (XPlatform.getInstance().isUserLogin()) {
  Toast.makeText(this, "The user is already logged in", Toast.LENGTH_SHORT).show();
} else {
  Toast.makeText(this, "The user is not logged in yet", Toast.LENGTH_SHORT).show();
}
```