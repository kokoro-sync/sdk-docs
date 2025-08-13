H5 Integration Guide
======

NOTE: This document describes how to call the H5 SDK API in H5 games. It mainly covers initialization, login, payment, and other interfaces.

Dependency Reference
-------

Before loading the H5 game page and before calling any H5SDK API, include and load the xsdk.js file. Always reference the full URL of xsdk.js (do not download the file locally). Please contact your business representative for the full URL.

For example, reference in the main file index.html of your H5 game:

~~~
<script src="https://oss.bytesdk.com/h5/xsdk.js"></script>
~~~

Required APIs
-------

#### Initialization (Required)

Initialization should be called after the game interface is rendered or after game resources are loaded. You can also set callbacks for logout and account switch success.

```
window.XSDKApi.init(function(data) {
    // Perform other operations after the initialization callback
    console.log('sdk init success:', data)

    // Set logout callback
    window.XSDKApi.setLogoutCallback(function() {
        // When this callback is received in-game, return the player to the login screen and call the login API again for re-login
        console.log('logout from sdk. in game_test.html')
    });    
});
```

#### Login API (Required)

```
window.XSDKApi.login(function(loginResult) {
    console.log('login result called:', loginResult)
    /**
     *  loginResult structure:
     *     ::code: status (int) 1: login success; 0: login failed
     *     ::msg : error message
     *     ::data: user data (Object)
     *         ::uid: globally unique user ID in SDK, can be used to bind character info
     *         ::name: username
     *         ::token: token to pass to game server for secondary authentication
     */         
})
```

#### Logout API (Optional)

```
window.XSDKApi.logout()
```

#### Submit Extended Data (Required)

```
var roleData = {
    serverID: '1',
    serverName: 'H5 Demo Server',
    roleID: '100',
    roleName: 'Character_&#@!%^_+·~|{}[]',
    roleLevel: '1',
    vip: '1',
    moneyNum: 0,
    createTime: parseInt("" + new Date().getTime()/1000),
    lastLevelUpTime: parseInt("" + new Date().getTime()/1000)
};

if (createRole) {
    roleData.type = 1       // Character creation
} else {
    roleData.type = 2       // Enter game; 3: Level up; 4: Exit game
}

window.XSDKApi.submit(roleData, function(result) {
    if (result && result.code == 0) {
        console.log('role submit success')
    } else {
        console.log('role submit failed')
    }
})
```

Call timing types:
1: Character creation
2: Enter game
3: Level up
4: Exit game

type indicates the current call timing.

roleData structure:
| Parameter Name        | Type          | Description  |
|:------------- |:-------------|:-----|
| type     | int | Call timing (1,2,3,4)|
| serverID| String| Server ID|
| serverName| String| Server name|
| roleID | String | Character ID|
| roleName| String | Character name|
| roleLevel| String | Character level|
| moneyNum| String | Game currency owned|
| createTime| long | Character creation time (seconds since 1970; must be accurate)|
| lastLevelUpTime| long | Level change time (seconds since 1970)|
| vip| String | VIP level|

#### Payment (Required)

```
var productPrice = 100           // Item price (in cents)
var cpOrderID = parseInt("" + new Date().getTime() / 1000)          // Game's own order number
var cpPayNotifyUrl = "http://172.16.0.109:12201/game/pay/callback"  // Server notification URL after payment

var orderData = {
    price: productPrice,
    productID: '1',
    productName: 'H5 Test Product',
    productDesc: 'Recharge 10 yuan, get 50 diamonds',
    serverID: '1',
    serverName: 'H5 Demo Server',
    roleID: '100',
    roleName: 'Character_&#@!%^_+·~|{}[]',
    roleLevel: 1,
    vip: '1',
    cpOrderID: cpOrderID,
    extra: "Custom data, returned as-is in notification",
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

OrderData structure:
| Parameter Name        | Type          | Description  |
|:------------- |:-------------|:-----|
| productID     | String | Product ID |
| productName      | String      |   Product name|
| productDesc| String      |    Product description|
| price| int | Amount (in cents)|
| serverID| String| Server ID|
| serverName| String| Server name|
| roleID | String | Character ID|
| roleName| String | Character name|
| roleLevel| int | Character level|
| vip | String| VIP level |
| payNotifyUrl| String | Server notification URL|
| cpOrderID | String | Order number|
| extra | String | Extended data|

#### Other APIs

1. Show User Center (Optional)

~~~
window.XSDKApi.showUserCenter()
~~~

2. Set Listener

~~~
window.XSDKApi.setLogoutCallback(function() {
    // When this callback is received in-game, return the player to the login screen and call the login API again for re-login
    console.log('logout from sdk. in game_test.html')
});
~~~
