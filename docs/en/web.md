H5 Integration Guide
======

NOTE: This document describes how to call the H5 SDK API in H5 games. It mainly includes interfaces for initialization, login, and payment.


Include Dependencies
-------

Before the H5 game page loads and before any H5 SDK API calls are made, include and load the H5 SDK's JS file: xsdk.js. Please reference the full URL of xsdk.js directly (do not download the JS file locally). For the complete URL of the JS file, please contact your business representative.

For example, in the main file of the H5 game, `index.html`:

```html
<script src="https://oss.bytesdk.com/h5/xsdk.js"></script>
```


Required Interfaces
-------

#### Initialization (Required)

Initialization should be called after the game interface has finished rendering or after the game resources have finished loading. You can also set callbacks for successful logout and account switching.

```js
window.XSDKApi.init(function(data) {
  // Need to perform other operations after the initialization callback
  console.log('sdk init success:', data)

  // Set the logout callback
  window.XSDKApi.setLogoutCallback(function() {
    console.log('logout from sdk. in game_test.html')
  });    
});
```

#### Login Interface (Required)

```js
window.XSDKApi.login(function(loginResult) {
  console.log('login result called:', loginResult)
})
```

`loginResult` object structure:

| Key | Description |
| :--- | :--- |
| code | Status code (1: success, 0: failure) |
| msg | Error message |
| data | User data |

`data` object structure:

| Key | Description |
| :--- | :--- |
| uid | User ID |
| name | User name |
| token | Session token |

#### Logout Interface (Optional)

```js
window.XSDKApi.logout()
```

#### Submit Extended Data (Required)

```js
const roleData = {
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
  roleData.type = 1       //Create character type
} else {
  roleData.type = 2       //Enter game type, 3: Level up; 4: Exit game
}

window.XSDKApi.submit(roleData, function(result) {
  if (result && result.code == 0) {
    console.log('role submit success')
  } else {
    console.log('role submit failed')
  }
}) 

```
This method classifies the timing of the call into several types:


1. Create character
2. Enter game
3. Level up
4. Exit game

At the points above, you need to call `window.XSDKApi.submit`. Here, `type` is the current timing of the call.


About the `roleData` data structure:


| Parameter Name        | Parameter Type          | Parameter Description  |
|:------------- |:-------------|:-----|
| type     | int | Timing of the call (1,2,3,4)|
| serverID| String| The ID of the server the player is on|
| serverName| String| The name of the server the player is on|
| roleID | String | Player's character ID|
| roleName| String | Player's character name|
| roleLevel| String | Player's character level|
| moneyNum| String | The amount of in-game currency the current character has|
| createTime| long | Character creation time, in seconds since 1970. You must pass the real data, otherwise the UC review will not pass.|
| lastLevelUpTime| long | Character level change time, in seconds since 1970|
| vip| String | Player's VIP level|


#### Payment/Recharge (Required)

```js
const productPrice = 100           //Current item price (in cents)
const cpOrderID = parseInt("" + new Date().getTime() / 1000)          // Game's own order number
const cpPayNotifyUrl = "http://172.16.0.109:12201/game/pay/callback"  // After payment is complete, the address where the game server receives the SDK server's callback notification

const orderData = {
  price: productPrice,
  productID: '1',
  productName: 'H5 Test Product',
  productDesc: 'Recharge 10 yuan and get 50 diamonds',
  serverID: '1',
  serverName: 'H5 Demo Server',
  roleID: '100',
  roleName: 'Character_&#@!%^_+·~|{}[]',
  roleLevel: 1,
  vip: '1',
  cpOrderID: cpOrderID,
  extra: "Custom data, returned as is in the callback notification",
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

About the `OrderData` object:

| Parameter Name        | Parameter Type          | Parameter Description  |
|:------------- |:-------------|:-----|
| productID     | String | Recharge product ID, the product ID within the game |
| productName      | String      |   Product name, e.g., 100 gold, 500 diamonds...|
| productDesc| String      |    Product description, e.g., Recharge 100 gold, get 20 gold free|
| price| int | Recharge amount (in cents)|
| serverID| String| The ID of the server the player is on|
| serverName| String| The name of the server the player is on|
| roleID | String | Player's character ID|
| roleName| String | Player's character name|
| roleLevel| int | Player's character level|
| vip | String| Player's VIP level |
| payNotifyUrl| String | Game server payment callback address. After successful payment, the SDK Server notifies the game server to deliver the goods according to this address; if not set, the fixed callback address configured in the SDK background will be used.|
| cpOrderID | String | The game's own order number; after successful payment, the SDK Server returns it to the game server as is when notifying the game server.|
| extra | String | Extended data; after successful payment, the SDK Server returns it to the game server as is when notifying the game server.|


Other Interfaces
-----------

1. Show User Center (Optional)

If you need to actively display the user center in the game, you can call the following interface:

```js
window.XSDKApi.showUserCenter()
```


2. Set Listener

When the player logs out within the SDK, the SDK will trigger a logout callback to the game layer. After receiving this callback, the game layer should guide the player back to the game login screen to log in again. This can be set after initialization is complete.

```js
window.XSDKApi.setLogoutCallback(function() {
  // Note: After receiving this callback in the game, you need to return the player to the game login screen and call the login interface again to have the player log in again.
  console.log('logout from sdk. in game_test.html')
});
```
