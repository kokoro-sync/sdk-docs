H5接続ガイド
======

NOTE: このドキュメントでは、H5ゲームでH5 SDKのAPIをどのように呼び出すかを説明します。主に初期化、ログイン、支払いなどのインターフェースを含みます。

依存関係の読み込み
-------

H5ゲームのページがロードされる前、またはH5SDK APIを呼び出す前に、xsdk.jsファイルを読み込んでください。jsファイルは必ずURLで直接参照し、ローカルにダウンロードしないでください。URLは営業担当にお問い合わせください。

例：H5ゲームのメインファイルindex.htmlで参照する場合

~~~
<script src="https://oss.bytesdk.com/h5/xsdk.js"></script>
~~~

必須API
-------

#### 初期化（必須）

初期化はゲーム画面の描画完了後、またはゲームのリソースロード完了後に呼び出してください。ログアウトやアカウント切替成功時のコールバックも設定できます。

```
window.XSDKApi.init(function(data) {
    // 初期化コールバック後に他の処理を行う
    console.log('sdk init success:', data)

    // ログアウトコールバック設定
    window.XSDKApi.setLogoutCallback(function() {
        // ゲーム内でこのコールバックを受け取ったら、ログイン画面に戻し、login APIを再度呼び出して再ログインさせてください
        console.log('logout from sdk. in game_test.html')
    });    
});
```

#### ログインAPI（必須）

```
window.XSDKApi.login(function(loginResult) {
    console.log('login result called:', loginResult)
    /**
     *  loginResult構造：
     *     ::code： 状態(int) 1：ログイン成功；0：ログイン失敗
     *     ::msg :  エラーメッセージ
     *     ::data: ユーザーデータ(Object)
     *         ::uid: SDK内のユーザーグローバルID。ゲームはこのIDでキャラクター情報を紐付け可能
     *         ::name：ユーザー名
     *         ::token: ゲームサーバーに渡すトークン。サーバー側で二次認証時に必要
     */         
})
```

#### ログアウトAPI（任意）

```
window.XSDKApi.logout()
```

#### 拡張データ送信（必須）

```
var roleData = {
    serverID: '1',
    serverName: 'H5デモサーバー',
    roleID: '100',
    roleName: 'キャラクター_&#@!%^_+·~|{}[]',
    roleLevel: '1',
    vip: '1',
    moneyNum: 0,
    createTime: parseInt("" + new Date().getTime()/1000),
    lastLevelUpTime: parseInt("" + new Date().getTime()/1000)
};

if (createRole) {
    roleData.type = 1       //キャラクター作成
} else {
    roleData.type = 2       //ゲーム開始。3：レベルアップ；4：ゲーム終了
}

window.XSDKApi.submit(roleData, function(result) {
    if (result && result.code == 0) {
        console.log('role submit success')
    } else {
        console.log('role submit failed')
    }
})
```

呼び出しタイミング：
1：キャラクター作成
2：ゲーム開始
3：レベルアップ
4：ゲーム終了

typeは現在の呼び出しタイミングを示します。

roleData構造：
| パラメータ名        | 型          | 説明  |
|:------------- |:-------------|:-----|
| type     | int | 呼び出しタイミング(1,2,3,4)|
| serverID| String| サーバーID|
| serverName| String| サーバー名|
| roleID | String | キャラクターID|
| roleName| String | キャラクター名|
| roleLevel| String | キャラクターレベル|
| moneyNum| String | 所持ゲーム通貨|
| createTime| long | キャラクター作成時刻（1970年からの秒数。正確な値必須）|
| lastLevelUpTime| long | レベル変化時刻（1970年からの秒数）|
| vip| String | VIPレベル|

#### 支払い（必須）

```
var productPrice = 100           //アイテム価格（単位：分）
var cpOrderID = parseInt("" + new Date().getTime() / 1000)          // ゲーム独自の注文番号
var cpPayNotifyUrl = "http://172.16.0.109:12201/game/pay/callback"  // 支払い完了後のサーバー通知URL

var orderData = {
    price: productPrice,
    productID: '1',
    productName: 'H5テスト商品',
    productDesc: '10元チャージで50ダイヤプレゼント',
    serverID: '1',
    serverName: 'H5デモサーバー',
    roleID: '100',
    roleName: 'キャラクター_&#@!%^_+·~|{}[]',
    roleLevel: 1,
    vip: '1',
    cpOrderID: cpOrderID,
    extra: "カスタムデータ。通知時にそのまま返却",
    payNotifyUrl: cpPayNotifyUrl
};

window.XSDKApi.pay(orderData, function(result){
    if (result && result.code == 0) {
        console.log('sdk pay success')
    } else {
        console.log('sdk pay failed')
    }
});
```

OrderData構造：
| パラメータ名        | 型          | 説明  |
|:------------- |:-------------|:-----|
| productID     | String | 商品ID |
| productName      | String      |   商品名|
| productDesc| String      |    商品説明|
| price| int | 金額（単位：分）|
| serverID| String| サーバーID|
| serverName| String| サーバー名|
| roleID | String | キャラクターID|
| roleName| String | キャラクター名|
| roleLevel| int | キャラクターレベル|
| vip | String| VIPレベル |
| payNotifyUrl| String | サーバー通知URL|
| cpOrderID | String | 注文番号|
| extra | String | 拡張データ|

#### その他API

1、ユーザーセンター表示（任意）

~~~
window.XSDKApi.showUserCenter()
~~~

2、リスナー設定

~~~
window.XSDKApi.setLogoutCallback(function() {
    // ゲーム内でこのコールバックを受け取ったら、ログイン画面に戻し、login APIを再度呼び出して再ログインさせてください
    console.log('logout from sdk. in game_test.html')
});
~~~
