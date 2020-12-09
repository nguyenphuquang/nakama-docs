# Authentication

Server đã có built-in authentication nên client chỉ cần gửi request và kết nối nếu đã có [server key](install-configuration.md#socket) để đăng nhập. Sau khi đăng nhập thành công, client nhận được 1 session tương ứng với [tài khoản người dùng](user-accounts.md).

!!! Warning "Quan trọng"
    Server key mặc định là `defaultkey`, bạn cần đặt 1 giá trị bí mật và [duy nhất](install-configuration.md#socket). Giá trị này sẽ được đưa vào trong client code.

```js tab="JavaScript"
var client = new nakamajs.Client("defaultkey", "127.0.0.1", 7350);
client.ssl = false;
```

```csharp tab=".NET"
// Use "https" scheme if you've setup SSL.
var client = new Client("http", "127.0.0.1", 7350, "defaultkey");
```

```csharp tab="Unity"
// Use "https" scheme if you've setup SSL.
var client = new Client("http", "127.0.0.1", 7350, "defaultkey");
```

```cpp tab="Cocos2d-x C++"
NClientParameters parameters;
parameters.serverKey = "defaultkey";
parameters.host = "127.0.0.1";
parameters.port = DEFAULT_PORT;
NClientPtr client = NCocosHelper::createDefaultClient(parameters);
```

```js tab="Cocos2d-x JS"
var serverkey = "defaultkey";
var host = "127.0.0.1";
var port = 7350;
var useSSL = false;
var timeout = 7000; // ms

var client = new nakamajs.Client(serverkey, host, port, useSSL, timeout);
```

```cpp tab="C++"
NClientParameters parameters;
parameters.serverKey = "defaultkey";
parameters.host = "127.0.0.1";
parameters.port = DEFAULT_PORT;
NClientPtr client = createDefaultClient(parameters);
```

```java tab="Java"
Client client = new DefaultClient("defaultkey", "127.0.0.1", 7349, false)
// or same as above.
Client client = DefaultClient.defaults("defaultkey");
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let client : Client = Builder("defaultkey")
    .host("127.0.0.1")
    .port(7350)
    .ssl(false)
    .build()
// or same as above.
let client : Client = Builder.defaults(serverKey: "defaultkey")
```

Mỗi tài khoản người dùng được tạo từ [các cách đăng nhập](#authenticate). Chúng tôi gọi mỗi cách là "link" bởi vì đó là 1 cách để truy cập vào thông tin của người dùng. Bạn có thể thêm một hoặc nhiều link để cho phép người dùng có thể login theo nhiều cách khác nhau trên nhiều thiết bị.

## Authenticate

Trước khi tương tác với server, bạn cần phải lấy được session token bằng cách đăng nhập vào hệ thống. Server hỗ trợ các phương thức authentication rất mềm dẻo, bạn có thể đăng ký user với email, facebook hoặc device.

!!! Note "Chú ý"
    Mặc định hệ thống sẽ tự động tạo tài khoản người dùng nếu thông tin này không có trước đây.

Để biết cách đăng ký và đăng nhập, hãy quan sát các ví dụ cho các ngôn ngữ lập trình phổ biến dưới đây.

### Device

Định danh thiết bị có thể được sử dụng như một cách để đăng ký người dùng với server. Phương thức này có thể làm người dùng thấy thoải mái vì không phải nhớ mật khẩu nhưng có thể không đáng tin cậy vì số định danh thiết bị đôi khi có thể thay đổi trong bản cập nhật thiết bị.

Bạn có thể chọn tên người dùng khi tạo tài khoản. Để làm điều này, hãy đặt giá trị `username`. Nếu bạn chỉ muốn xác thực mà không tạo tài khoản người dùng, hãy đặt `create` là false.

Một định danh thiết bị phải chứa các ký tự chữ và số có dấu gạch ngang và nằm trong khoảng từ 10 đến 128 byte.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/authenticate/custom?create=true&username=mycustomusername" \
  --user 'defaultkey:' \
  --data '{"id":"uniqueidentifier"}'
```

```js tab="JavaScript"
// This import is only required with React Native
var deviceInfo = require('react-native-device-info');

var deviceId = null;
try {
  const value = await AsyncStorage.getItem('@MyApp:deviceKey');
  if (value !== null){
    deviceId = value
  } else {
    deviceId = deviceInfo.getUniqueID();
    AsyncStorage.setItem('@MyApp:deviceKey', deviceId).catch(function(error){
      console.log("An error occured: %o", error);
    });
  }
} catch (error) {
  console.log("An error occured: %o", error);
}

const session = await client.authenticateDevice({ id: deviceId, create: true, username: "mycustomusername" });
console.info("Successfully authenticated:", session);
```

```csharp tab=".NET"
// Should use a platform API to obtain a device identifier.
var deviceId = System.Guid.NewGuid().ToString();
var session = await client.AuthenticateDeviceAsync(deviceId);
System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
```

```csharp tab="Unity"
var deviceId = PlayerPrefs.GetString("nakama.deviceid");
if (string.IsNullOrEmpty(deviceId)) {
    deviceId = SystemInfo.deviceUniqueIdentifier;
    PlayerPrefs.SetString("nakama.deviceid", deviceId); // cache device id.
}
var session = await client.AuthenticateDeviceAsync(deviceId);
Debug.LogFormat("New user: {0}, {1}", session.Created, session);
```

```cpp tab="Cocos2d-x C++"
auto loginFailedCallback = [](const NError& error)
{
};

auto loginSucceededCallback = [](NSessionPtr session)
{
  CCLOG("Successfully authenticated");
};

std::string deviceId = "unique device id";

client->authenticateDevice(
        deviceId,
        opt::nullopt,
        opt::nullopt,
        {},
        loginSucceededCallback,
        loginFailedCallback);
```

```js tab="Cocos2d-x JS"
var deviceId = "unique device id";
client.authenticateDevice({ id: deviceId, create: true, username: "mycustomusername" })
  .then(function(session) {
        cc.log("Authenticated successfully. User id:", session.user_id);
    },
    function(error) {
        cc.error("authenticate failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto loginFailedCallback = [](const NError& error)
{
};

auto loginSucceededCallback = [](NSessionPtr session)
{
  cout << "Successfully authenticated" << endl;
};

std::string deviceId = "unique device id";

client->authenticateDevice(
        deviceId,
        opt::nullopt,
        opt::nullopt,
        {},
        loginSucceededCallback,
        loginFailedCallback);
```

```java tab="Java"
String id = UUID.randomUUID().toString();
Session session = client.authenticateDevice(id).get();
System.out.format("Session: %s ", session.getAuthToken());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let defaults = UserDefaults.standard
let deviceKey = "device_id"

var deviceId : String? = defaults.string(forKey: deviceKey)
if deviceId == nil {
  deviceId = UIDevice.current.identifierForVendor!.uuidString
  defaults.set(deviceId!, forKey: deviceKey)
}

let message = AuthenticateMessage(device: deviceId!)
client.login(with: message).then { session in
  print("Login successful")
}.catch{ err in
  if (err is NakamaError) {
    switch err as! NakamaError {
    case .userNotFound(_):
      let _ = self.client.register(with: message)
      return
    default:
      break
    }
  }
  print("Could not login: %@", err)
}
```

```tab="REST"
POST /v2/account/authenticate/device?create=true&username=mycustomusername
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Basic base64(ServerKey:)

{
    "id": "uniqueidentifier"
}
```

Trong các trò chơi, nên sử dụng [Google](#google) hoặc [Game Center](#game-center) để đăng ký người dùng.

### Email


Người dùng có thể đăng ký bằng email và mật khẩu. Mật khẩu được hash trước khi lưu trữ trong database và không thể được đọc hoặc "recovered" bởi các quản trị viên. Bạn có thể yên tâm về bảo vệ sự riêng tư của người dùng.

Bạn có thể chọn tên người dùng khi tạo tài khoản. Để làm điều này, đặt giá trị `username`. Nếu bạn muốn chỉ xác thực mà không tạo tài khoản người dùng, hãy đặt `create` là false.

Địa chỉ email phải hợp lệ theo quy định của RFC-5322 và mật khẩu phải có ít nhất 8 ký tự.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/authenticate/email?create=true&username=mycustomusername" \
  --user 'defaultkey:' \
  --data '{"email":"email@example.com", "password": "3bc8f72e95a9"}'
```

```js tab="JavaScript"
const email = "email@example.com";
const password = "3bc8f72e95a9";
const session = await client.authenticateEmail({ email: email, password: password, create: true, username: "mycustomusername" })
console.info("Successfully authenticated:", session);
```

```csharp tab=".NET"
const string email = "email@example.com";
const string password = "3bc8f72e95a9";
var session = await client.AuthenticateEmailAsync(email, password);
System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
```

```csharp tab="Unity"
const string email = "email@example.com";
const string password = "3bc8f72e95a9";
var session = await client.AuthenticateEmailAsync(email, password);
Debug.LogFormat("New user: {0}, {1}", session.Created, session);
```

```cpp tab="Cocos2d-x C++"
auto successCallback = [](NSessionPtr session)
{
  CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
};

auto errorCallback = [](const NError& error)
{
};

string email = "email@example.com";
string password = "3bc8f72e95a9";
string username = "mycustomusername";
bool create = true;
client->authenticateEmail(email, password, username, create, {}, successCallback, errorCallback);
```

```js tab="Cocos2d-x JS"
const email = "email@example.com";
const password = "3bc8f72e95a9";
client.authenticateEmail({ email: email, password: password, create: true, username: "mycustomusername" })
  .then(function(session) {
      cc.log("Authenticated successfully. User ID:", session.user_id);
    },
    function(error) {
      cc.error("authenticate failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto successCallback = [](NSessionPtr session)
{
    std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
};

auto errorCallback = [](const NError& error)
{
};

string email = "email@example.com";
string password = "3bc8f72e95a9";
string username = "mycustomusername";
bool create = true;
client->authenticateEmail(email, password, username, create, {}, successCallback, errorCallback);
```

```java tab="Java"
String email = "email@example.com";
String password = "3bc8f72e95a9";
Session session = client.authenticateEmail(email, password).get();
System.out.format("Session: %s ", session.getAuthToken());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let email = "email@example.com"
let password = "3bc8f72e95a9"

let message = AuthenticateMessage(email: email, password: password)
client.register(with: message).then { session in
  NSLog("Session: %@", session.token)
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
// Use client.login(...) after register.
```

```tab="REST"
POST /v2/account/authenticate/email?create=true&username=mycustomusername
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Basic base64(ServerKey:)

{
    "email": "email@example.com",
    "password": "3bc8f72e95a9"
}
```

### Đăng nhập bằng tài khoản mạng xã hội

Server hỗ trợ rất nhiều mạng xã hội khác nhau. Với mỗi nhà cung cấp, tài khoản người dùng của mạng sẽ được sử dụng để thiết lập người dùng trên Game Platform. Trong một số trường hợp, [friends](social-friends.md) của người dùng cũng sẽ được tải về và thêm vào danh sách friend của họ.

#### Facebook

Để làm việc với Facebook, bạn cần thêm Facebook SDK vào project. Tham khảo <a href="https://developers.facebook.com/docs/" target="\_blank">tại đây</a>. Thực hiện theo các hướng dẫn về cách tích hợp code. Nếu là project mobile, bạn cần hoàn thành các hướng dẫn riêng cho cả iOS và Android.

Bạn có thể chọn tên người dùng khi tạo tài khoản. Để làm điều này, đặt giá trị `username`. Nếu bạn muốn chỉ xác thực mà không tạo tài khoản người dùng, hãy đặt `create` là false.

Bạn có thể tùy ý import [bạn bè](social-friends.md) trên Facebook khi xác thực. Để làm điều này, đặt `import` là true.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/authenticate/facebook?create=true&username=mycustomusername&import=true" \
  --user 'defaultkey:' \
  --data '{"token":"valid-oauth-token"}'
```

```js tab="JavaScript"
const oauthToken = "...";
const session = await client.authenticateFacebook({ token: oauthToken, create: true, username: "mycustomusername", import: true });
console.log("Successfully authenticated:", session);
```

```csharp tab=".NET"
const string oauthToken = "...";
var session = await client.AuthenticateFacebookAsync(oauthToken);
System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
```

```csharp tab="Unity"
// using Facebook.Unity;
// https://developers.facebook.com/docs/unity/examples#init
var perms = new List<string>(){"public_profile", "email"};
FB.LogInWithReadPermissions(perms, async (ILoginResult result) => {
    if (FB.IsLoggedIn) {
        var accessToken = Facebook.Unity.AccessToken.CurrentAccessToken;
        var session = await client.LinkFacebookAsync(session, accessToken);
        Debug.LogFormat("New user: {0}, {1}", session.Created, session);
    }
});
```

```cpp tab="Cocos2d-x C++"
auto loginFailedCallback = [](const NError& error)
{
};

auto loginSucceededCallback = [](NSessionPtr session)
{
  CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
};

std::string oauthToken = "...";
bool importFriends = true;

client->authenticateFacebook(
        oauthToken,
        "mycustomusername",
        true,
        importFriends,
        {},
        loginSucceededCallback,
        loginFailedCallback);
```

```js tab="Cocos2d-x JS"
const oauthToken = "...";
client.authenticateFacebook({ token: oauthToken, create: true, username: "mycustomusername", import: true })
  .then(function(session) {
      cc.log("Authenticated successfully. User ID:", session.user_id);
    },
    function(error) {
      cc.error("authenticate failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto loginFailedCallback = [](const NError& error)
{
};

auto loginSucceededCallback = [](NSessionPtr session)
{
  cout << "Authenticated successfully. User ID: " << session->getUserId() << endl;
};

std::string oauthToken = "...";
bool importFriends = true;

client->authenticateFacebook(
        oauthToken,
        "mycustomusername",
        true,
        importFriends,
        {},
        loginSucceededCallback,
        loginFailedCallback);
```

```java tab="Java"
String oauthToken = "...";
Session session = client.authenticateFacebook(oauthToken).get();
System.out.format("Session %s", session.getAuthToken());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let oauthToken = "..."
let message = AuthenticateMessage(facebook: oauthToken)
client.register(with: message).then { session in
  NSLog("Session: %@", session.token)
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
POST /v2/account/authenticate/facebook?create=true&username=mycustomusername&import=true
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Basic base64(ServerKey:)

{
  "token": "...",
}
```

Bạn có thể thêm một button vào giao diện của mình để đăng nhập bằng Facebook.

```csharp tab="Unity"
FB.Login("email", (ILoginResult result) => {
  if (FB.IsLoggedIn) {
    var oauthToken = Facebook.Unity.AccessToken.CurrentAccessToken.TokenString;
    var session = await client.LinkFacebookAsync(session, accesstoken);
    Debug.LogFormat("Session: '{0}'.", session.AuthToken);
});
```

#### Google

Tương tự như Facebook để đăng ký và đăng nhập, bạn cần sử dụng một trong các client SDK của Google.

Bạn có thể chọn tên người dùng khi tạo tài khoản. Để làm điều này, đặt giá trị `username`. Nếu bạn muốn chỉ xác thực mà không tạo tài khoản người dùng, hãy đặt `create` là false.  

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/authenticate/google?create=true&username=mycustomusername" \
  --user 'defaultkey:' \
  --data '{"token":"valid-oauth-token"}'
```

```js tab="JavaScript"
const playerIdToken = "...";
const session = await client.authenticateGoogle({ token: oauthToken, create: true, username: "mycustomusername" });
console.info("Successfully authenticated: %o", session);
```

```csharp tab=".NET"
const string playerIdToken = "...";
var session = await client.AuthenticateGoogleAsync(playerIdToken);
System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
```

```csharp tab="Unity"
const string playerIdToken = "...";
var session = await client.AuthenticateGoogleAsync(playerIdToken);
Debug.LogFormat("New user: {0}, {1}", session.Created, session);
```

```cpp tab="Cocos2d-x C++"
auto successCallback = [](NSessionPtr session)
{
  CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
};

auto errorCallback = [](const NError& error)
{
};

string oauthToken = "...";
client->authenticateGoogle(oauthToken, "mycustomusername", true, {}, successCallback, errorCallback);
```

```js tab="Cocos2d-x JS"
const oauthToken = "...";
client.authenticateGoogle({ token: oauthToken, create: true, username: "mycustomusername" })
  .then(function(session) {
      cc.log("Authenticated successfully. User ID:", session.user_id);
    },
    function(error) {
      cc.error("authenticate failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto successCallback = [](NSessionPtr session)
{
    std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
};

auto errorCallback = [](const NError& error)
{
};

string oauthToken = "...";
client->authenticateGoogle(oauthToken, "mycustomusername", true, {}, successCallback, errorCallback);
```

```java tab="Java"
String playerIdToken = "...";
Session session = client.authenticateGoogle(oauthToken).get();
System.out.format("Session %s", session.getAuthToken());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let playerIdToken = "..."
let message = AuthenticateMessage(google: oauthToken)
client.register(with: message).then { session in
  NSLog("Session: %@", session.token)
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
POST /v2/account/authenticate/google?create=true&username=mycustomusername
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Basic base64(ServerKey:)

{
  "token": "...",
}
```

### Custom id

Một __custom id__ có thể được sử dụng theo cách tương tự như __device id__ để đăng nhập hoặc đăng ký. Bạn chỉ cần cung cấp 1 custom id duy nhất cho 1 người dùng..

Một __custom id__ phải chứa các ký tự chữ và số có dấu gạch ngang và nằm trong khoảng từ 6 đến 128 byte.

Bạn có thể chọn tên người dùng khi tạo tài khoản. Để làm điều này, hãy đặt giá trị `username`. Nếu bạn muốn chỉ xác thực mà không tạo tài khoản người dùng, hãy đặt `create` là false.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/authenticate/custom?create=true&username=mycustomusername" \
  --user 'defaultkey:' \
  --data '{"id":"some-custom-id"}'
```

```js tab="JavaScript"
const customId = "some-custom-id";
const session = await client.authenticateCustom({ id: customId, create: true, username: "mycustomusername" });
console.info("Successfully authenticated:", session);
```

```csharp tab=".NET"
const string customId = "some-custom-id";
var session = await client.AuthenticateCustomAsync(customId);
System.Console.WriteLine("New user: {0}, {1}", session.Created, session);
```

```csharp tab="Unity"
const string customId = "some-custom-id";
var session = await client.AuthenticateCustomAsync(customId);
Debug.LogFormat("New user: {0}, {1}", session.Created, session);
```

```cpp tab="Cocos2d-x C++"
auto successCallback = [](NSessionPtr session)
{
  CCLOG("Authenticated successfully. User ID: %s", session->getUserId().c_str());
};

auto errorCallback = [](const NError& error)
{
};

string id = "some-custom-id";
string username = "mycustomusername";
bool create = true;
client->authenticateCustom(id, username, create, {}, successCallback, errorCallback);
```

```js tab="Cocos2d-x JS"
const customId = "some-custom-id";
client.authenticateCustom({ id: customId, create: true, username: "mycustomusername" })
  .then(function(session) {
      cc.log("Authenticated successfully. User ID:", session.user_id);
    },
    function(error) {
      cc.error("authenticate failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto successCallback = [](NSessionPtr session)
{
    std::cout << "Authenticated successfully. User ID: " << session->getUserId() << std::endl;
};

auto errorCallback = [](const NError& error)
{
};

string id = "some-custom-id";
string username = "mycustomusername";
bool create = true;
client->authenticateCustom(id, username, create, {}, successCallback, errorCallback);
```

```java tab="Java"
String customId = "some-custom-id";
Session session = client.authenticateCustom(customId).get();
System.out.format("Session %s", session.getAuthToken());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let customID = "some-custom-id"

let message = AuthenticateMessage(custom: customID)
client.register(with: message).then { session in
  NSLog("Session: %@", session.token)
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
// Use client.login(...) after register.
```

```tab="REST"
POST /v2/account/authenticate/custom?create=true&username=mycustomusername
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Basic base64(ServerKey:)

{
  "id": "some-custom-id",
}
```

## Sessions

Api đăng nhập thành công sẽ trả lại thông tin về session. Session chứa các thông tin về userId, userName cho đến khi hết hạn.

!!! Tip "Mẹo"
    Bạn có thể thay đổi thời gian hết hạn (expire) của sessiong ở mục [configuration](install-configuration.md) của Console server. Thời gian hết hạn mặc định là 60 giây.

```js tab="JavaScript"
const id = "3e70fd52-7192-11e7-9766-cb3ce5609916";
const session = await client.authenticateDevice({ id: id })
console.info("id:", session.user_id, "username:", session.username);
console.info("Session expired?", session.isexpired(Date.now() / 1000));
```

```csharp tab=".NET"
const string id = "3e70fd52-7192-11e7-9766-cb3ce5609916";
var session = await client.AuthenticateDeviceAsync(id);
System.Console.WriteLine("Id '{0}' Username '{1}'", session.UserId, session.Username);
System.Console.WriteLine("Session expired? {0}", session.IsExpired);
```

```csharp tab="Unity"
var deviceId = SystemInfo.deviceUniqueIdentifier;
var session = await client.AuthenticateDeviceAsync(deviceId);
Debug.LogFormat("Id '{0}' Username '{1}'", session.UserId, session.Username);
Debug.LogFormat("Session expired? {0}", session.IsExpired);
```

```cpp tab="Cocos2d-x C++"
auto loginFailedCallback = [](const NError& error)
{
};

auto loginSucceededCallback = [](NSessionPtr session)
{
  CCLOG("id %s username %s", session->getUserId().c_str(), session->getUsername().c_str());
  CCLOG("Session expired? %s", session->isExpired() ? "yes" : "no");
};

std::string deviceId = "3e70fd52-7192-11e7-9766-cb3ce5609916";

client->authenticateDevice(
        deviceId,
        opt::nullopt,
        opt::nullopt,
        {},
        loginSucceededCallback,
        loginFailedCallback);
```

```js tab="Cocos2d-x JS"
var deviceId = "3e70fd52-7192-11e7-9766-cb3ce5609916";
client.authenticateDevice({ id: deviceId })
  .then(function(session) {
        cc.log("Authenticated successfully. User id:", session.user_id);
    },
    function(error) {
        cc.error("authenticate failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto loginFailedCallback = [](const NError& error)
{
};

auto loginSucceededCallback = [](NSessionPtr session)
{
  cout << "id " << session->getUserId() << " username " << session->getUsername() << endl;
  cout << "Session expired? " << (session->isExpired() ? "yes" : "no") << endl;
};

std::string deviceId = "3e70fd52-7192-11e7-9766-cb3ce5609916";

client->authenticateDevice(
        deviceId,
        opt::nullopt,
        opt::nullopt,
        {},
        loginSucceededCallback,
        loginFailedCallback);
```

```java tab="Java"
var deviceid = SystemInfo.deviceUniqueIdentifier;
Session session = client.authenticateDevice(deviceid).get();
System.out.format("Session %s", session.getAuthToken());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let id = "3e70fd52-7192-11e7-9766-cb3ce5609916"
let message = AuthenticateMessage(device: id)
client.login(with: message).then { session in
  let expired = session.isExpired(currentTimeSince1970: Date().timeIntervalSince1970)
  NSLog("Session id '%@' handle '%@'.", session.userID, session.handle)
  NSLog("Session expired: '%@'", expired)
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

### Connect

Với session, bạn có thể gọi API trên server, trao đổi message với server. Tuy nhiên, phần lớn các client đều không tự động kết nối lại (auto-reconnect). Do đó, bạn cần phải
tự thực hiện auto-reconnect. Hãy tham khảo các đoạn code dưới đây.

```js tab="JavaScript"
var socket = client.createSocket();
session = await socket.connect(session);
console.info("Socket connected.");
```

```csharp tab=".NET"
var socket = Socket.From(client);
await socket.ConnectAsync(session);
System.Console.WriteLine("Socket connected.");
```

```csharp tab="Unity"
var socket = client.NewSocket();
await socket.ConnectAsync(session);
Debug.Log("Socket connected.");
```

```cpp tab="Cocos2d-x C++"
#include "NakamaCocos2d/NWebSocket.h"

int port = 7350; // different port to the main API port
bool createStatus = true; // if the server should show the user as online to others.
// define realtime client in your class as NRtClientPtr rtClient;
rtClient = client->createRtClient(port, NRtTransportPtr(new NWebSocket()));
// define listener in your class as NRtDefaultClientListener listener;
listener.setConnectCallback([]()
{
  CCLOG("Socket connected.");
});
rtClient->setListener(&listener);
rtClient->connect(session, createStatus);
```

```js tab="Cocos2d-x JS"
const socket = client.createSocket();
socket.connect(session)
  .then(
      function() {
        cc.log("Socket connected.");
      },
      function(error) {
        cc.error("connect failed:", JSON.stringify(error));
      }
    );
```

```cpp tab="C++"
int port = 7350; // different port to the main API port
bool createStatus = true; // if the server should show the user as online to others.
// define realtime client in your class as NRtClientPtr rtClient;
rtClient = client->createRtClient(port);
// define listener in your class as NRtDefaultClientListener listener;
listener.setConnectCallback([]()
{
  cout << "Socket connected." << endl;
});
rtClient->setListener(&listener);
rtClient->connect(session, createStatus);
```

```java tab="Java"
SocketClient socket = client.createSocket();
socket.connect(session, new AbstractSocketListener() {}).get();
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let session : Session = someSession // obtained from register or login.
client.connect(with: session).then { _ in
  NSLog("Socket connected.")
});
```

### Expiry

Session có thể hết hạn và trở thành không hợp lệ. Nếu điều này xảy ra, bạn cần phải xác nhận lại với server và nhận session mới. Bạn có thể điều chỉnh thời gian tồn tại của session trên server bằng cmdflag "--session.token_exiry_sec 604800". Xem trang [Cấu hình session] (#install-configuration.md#session).

Bạn có thể kiểm tra thời hạn của phiên bằng code sau:

```js tab="JavaScript"
const nowUnixTime = Math.floor(Date.now() / 1000);
if (session.isexpired(nowUnixTime)) {
  console.log("Session has expired. Must reauthenticate!");
}
```

```csharp tab=".NET"
var nowUnixTime = DateTime.UtcNow;
if (session.HasExpired(nowUnixTime))
{
  System.Console.WriteLine("Session has expired. Must reauthenticate!");
}
```

```csharp tab="Unity"
var nowUnixTime = DateTime.UtcNow;
if (session.HasExpired(nowUnixTime))
{
  Debug.Log("Session has expired. Must reauthenticate!");
}
```

```cpp tab="Cocos2d-x C++"
if (session->isExpired())
{
  CCLOG("Session has expired. Must reauthenticate!");
}
```

```js tab="Cocos2d-x JS"
const nowUnixTime = Math.floor(Date.now() / 1000);
if (session.isexpired(nowUnixTime)) {
  cc.log("Session has expired. Must reauthenticate!");
}
```

```cpp tab="C++"
if (session->isExpired())
{
  cout << "Session has expired. Must reauthenticate!";
}
```

```java tab="Java"
if (session.isExpired(new Date())) {
  System.out.println("Session has expired. Must reauthenticate!");
}
```

## Link, unlink

Bạn có thể liên kết một hoặc nhiều tùy chọn đăng nhập khác với người dùng hiện tại. Điều này giúp dễ dàng hỗ trợ nhiều lần đăng nhập với mỗi người dùng và dễ dàng xác định người dùng trên các thiết bị.

Bạn chỉ có thể liên kết các device Id, custom Ids, social provider Ids... chưa được sử dụng với tài khoản người dùng khác.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/link/custom" \
  --header 'Authorization: Bearer $session' \
  --data '{"id":"some-custom-id"}'
```

```js tab="JavaScript"
const customId = "some-custom-id";
const success = await client.linkCustom(session, { id: customId });
console.log("Successfully linked custom ID to current user.");
```

```csharp tab=".NET"
const string customId = "some-custom-id";
await client.LinkCustomAsync(session, customId);
System.Console.WriteLine("Id '{0}' linked for user '{1}'", customId, session.UserId);
```

```csharp tab="Unity"
const string customid = "some-custom-id";
await client.LinkCustomAsync(session, customId);
Debug.LogFormat("Id '{0}' linked for user '{1}'", customId, session.UserId);
```

```cpp tab="Cocos2d-x C++"
auto linkFailedCallback = [](const NError& error)
{
};

auto linkSucceededCallback = []()
{
  CCLOG("Linked successfully");
};

std::string customid = "some-custom-id";

client->linkCustom(customid, linkSucceededCallback, linkFailedCallback);
```

```js tab="Cocos2d-x JS"
const customId = "some-custom-id";
client.linkCustom(session, { id: customId })
  .then(function() {
      cc.log("Linked successfully");
    },
    function(error) {
      cc.error("link failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto linkFailedCallback = [](const NError& error)
{
};

auto linkSucceededCallback = []()
{
  cout << "Linked successfully" << endl;
};

std::string customid = "some-custom-id";

client->linkCustom(customid, linkSucceededCallback, linkFailedCallback);
```

```java tab="Java"
String customId = "some-custom-id";
client.linkCustom(session, customId).get();
System.out.format("Id %s linked for user %s", customId, session.getUserId());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let id = "some-custom-id"
var message = SelfLinkMessage(device: id);
client.send(with: message).then {
  NSLog("Successfully linked device ID to current user.")
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
POST /v2/account/link/custom
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Bearer <session token>

{
    "id":"some-custom-id"
}
```

You can unlink any linked login options for the current user.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/unlink/custom" \
  --header 'Authorization: Bearer $session' \
  --data '{"id":"some-custom-id"}'
```

```js tab="JavaScript"
const customId = "some-custom-id";
const success = await client.unlinkCustom(session, { id: customId });
console.info("Successfully unlinked custom ID from the current user.");
```

```csharp tab=".NET"
const string customId = "some-custom-id";
await client.UnlinkCustomAsync(session, customId);
System.Console.WriteLine("Id '{0}' unlinked for user '{1}'", customId, session.UserId);
```

```csharp tab="Unity"
const string customId = "some-custom-id";
await client.UnlinkCustomAsync(session, customId);
Debug.LogFormat("Id '{0}' unlinked for user '{1}'", customId, session.UserId);
```


```cpp tab="Cocos2d-x C++"
auto unlinkFailedCallback = [](const NError& error)
{
};

auto unlinkSucceededCallback = []()
{
  CCLOG("Successfully unlinked custom ID from the current user.");
};

std::string customid = "some-custom-id";

client->unlinkCustom(customid, unlinkSucceededCallback, unlinkFailedCallback);
```

```js tab="Cocos2d-x JS"
const customId = "some-custom-id";
client.unlinkCustom(session, { id: customId })
  .then(function() {
      cc.log("Successfully unlinked custom ID from the current user.");
    },
    function(error) {
      cc.error("unlink failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto unlinkFailedCallback = [](const NError& error)
{
};

auto unlinkSucceededCallback = []()
{
  cout << "Successfully unlinked custom ID from the current user." << endl;
};

std::string customid = "some-custom-id";

client->unlinkCustom(customid, unlinkSucceededCallback, unlinkFailedCallback);
```

```java tab="Java"
String customId = "some-custom-id";
client.unlinkCustom(session, customId).get();
System.out.format("Id %s unlinked for user %s", customId, session.getUserId());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let id = "some-custom-id"
var message = SelfUnlinkMessage(device: id);
client.send(with: message).then {
  NSLog("Successfully unlinked device ID from current user.")
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
POST /v2/account/unlink/custom
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Bearer <session token>

{
  "id":"some-custom-id"
}
```

Bạn có thể link hoặc unlink nhiều tuỳ chọn tài khoản khác nhau.

| Link | Mô tả |
| ---- | ----------- |
| Custom | Một định danh tùy chỉnh từ một dịch vụ nhận dạng khác. |
| Device | Một định danh duy nhất cho một thiết bị thuộc về người dùng. |
| Email | Một email và mật khẩu được đặt bởi người dùng. |
| Facebook | Một tài khoản xã hội Facebook. Bạn có thể tùy chọn nhập Bạn bè trên Facebook khi liên kết. |
| Google | Một tài khoản Google. |
