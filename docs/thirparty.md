# Client

Để kết nối với Gameplatform, Core chỉ cần gửi kèm theo [server key](install-configuration.md#socket).

!!! Warning "Chú ý"
    Server key là 1 giá trị độc nhất mà game platform sẽ cung cấp cho ViettelPay, trong các ví dụ server key được sử dụng là "defaultkey".

```js tab="JavaScript"
var client = new nakamajs.Client("defaultkey", "127.0.0.1", 7350);
client.ssl = false;
```

```java tab="Java"
Client client = new DefaultClient("defaultkey", "127.0.0.1", 7349, false)
// or same as above.
Client client = DefaultClient.defaults("defaultkey");
```

Người dùng có thể đăng nhập bằng nhiều cách khác nhau, sau đó có thể "link" các cách đăng nhập vào và hệ thống sẽ hiểu đó chỉ là một tài khoản.

## Đăng nhập

Trước ghi gọi các API của Game Platform, bạn phải lấy session token bằng cách đăng nhập. Hệ thống đăng nhập của Game Platform rất mềm dẻo. Có thể đăng nhập được bằng email, Facebook, device...

!!! Note "Ghi chú"
    Hệ thống sẽ mặc định tạo tài khoản cho người dùng nếu thông tin đăng nhập chưa tồn tại trước đó.

### Email

Nếu hệ thống của Core sử dụng email để đăng nhập, Core có thể sử dụng email để đăng nhập vào Game Platform.

Bạn có thể chọn username cho tài khoản, hoặc có thể chọn username là email.

Email phải tuân thủ theo chuẩn RFC-5322, mật khẩu tối thiểu là 8 ký tự.

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

```java tab="Java"
String email = "email@example.com";
String password = "3bc8f72e95a9";
Session session = client.authenticateEmail(email, password).get();
System.out.format("Session: %s ", session.getAuthToken());
```

### Custom id

Nếu Core không sử dụng email để đăng nhập, cách đơn giản nhất là sử dụng custom id. Custom id là 1 string từ 6-128 byte. Có thể sử dụng username làm custom id.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/authenticate/custom?create=true&username=mycustomusername" \
  --user 'defaultkey:' \
  --data '{"id":"some-custom-id"}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/logout";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
    type: 'timeout'
})

return client.doFetch(urlPath, "POST", queryParams, body, {})
```

```java tab="Java"
String urlPath = "/v2/account/user/logout";

JSONObject body = new JSONObject();
body.putString("userId", "1234");
body.putString("type", "timeout");

client.doFetch(urlPath, "POST", null, body, null);
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

```java tab="Java"
var deviceid = SystemInfo.deviceUniqueIdentifier;
Session session = client.authenticateDevice(deviceid).get();
System.out.format("Session %s", session.getAuthToken());
```

### Connect

Với session, bạn có thể gọi API trên server, trao đổi message với server. Tuy nhiên, phần lớn các client đều không tự động kết nối lại (auto-reconnect). Do đó, bạn cần phải
tự thực hiện auto-reconnect.

Hãy tham khảo các đoạn code dưới đây.

```js tab="JavaScript"
var socket = client.createSocket();
session = await socket.connect(session);
console.info("Socket connected.");
```

```java tab="Java"
SocketClient socket = client.createSocket();
socket.connect(session, new AbstractSocketListener() {}).get();
```

## Core actions

Khi người dùng thực hiện một số action, các action này sẽ được Core gửi về Game Platform qua Rest API để ghi nhận. Các action đề xuất:

* Đăng xuất: Để tính toán thời gian sử dụng ứng dụng.
* Nạp tiền (topup): Nhằm tính điểm tích lũy.
* Thực hiện giao dịch: Nhằm tính điểm tích lũy.
* Sử dụng chức năng: Nhằm tính điểm tích lũy.

Người dùng rate và bình luận về ứng dụng. Chức năng này đòi hỏi App phải nhận được các sự kiện về người dùng đã bình luận và rate cho app (thực hiện từ phía ViettelPay).
Người dùng sử dụng các tính năng có thông báo cash-back. Tiền cash-back sẽ được tính bằng số Gold và có thể đổi thành tiền thật.
Các API được liệt kê trong phần này được gọi từ Core, Core phải đăng nhập bằng acount Core rồi gọi API với thông tin session của account Core vừa đăng nhập.

### Đăng xuất

Mỗi khi người dùng thoát khỏi ứng dụng hoặc timeout, Core gọi Api đăng xuất để ghi nhận thời gian sử dụng ứng dụng.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/user/logout" \
  --user 'defaultkey:' \
  --data '{"userId":"1234","type":"timeout"}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/logout";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
    type: 'timeout'
})

return client.doFetch(urlPath, "POST", queryParams, body, {})
```

```java tab="Java"
String urlPath = "/v2/account/user/logout";

JSONObject body = new JSONObject();
body.putString("userId", "1234");
body.putString("type", "timeout");

client.doFetch(urlPath, "POST", null, body, null);
```

### Nạp tiền
Khi người dùng nạp tiền vào tài khoản, Core gọi API thông báo cho game platform để thực hiện cộng điểm tích luỹ. Tuỳ thuộc vào quy định về bảo mật thông tin, thông tin được truyền tới game platform có thể là số tiền được nạp vào hoặc là số gold tích luỹ được của user. Trong trường hợp không truyền số tiền, Core phải thực hiện việc quy đổi từ số tiền thành gold và phải đảm bảo được khả năng thay đổi tỷ lệ chuyển đổi trong quá trình quản trị.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/user/topup" \
  --user 'defaultkey:' \
  --data '{"userId":"1234", "amount": "10000"}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/topup";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
    amount: 10000,
});

return client.doFetch(urlPath, "POST", queryParams, body, {})
```

```java tab="Java"
String urlPath = "/v2/account/user/topup";

JSONObject body = new JSONObject();
body.putString("userId", "1234");
body.putLong("amount", 10000L);

client.doFetch(urlPath, "POST", null, body, null);
```

### Giao dịch
Khi người dùng chuyển tiền, Core gọi API báo cho Game Platform để ghi nhận nhằm tính điểm tích luỹ và xác định người dùng đã hoàn thành những nhiệm vụ nào liên quan đến chuyển tiền.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/user/transfer \
  --user 'defaultkey:' \
  --data '{"userId":"1234","amount": 200000, "type":"outside"}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/transfer";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
    amount: 10000,
    type: 'outside',
});

return client.doFetch(urlPath, "POST", queryParams, body, {})
```

```java tab="Java"
String urlPath = "/v2/account/user/transfer";

JSONObject body = new JSONObject();
body.putString("userId", "1234");
body.putLong("amount", 10000L);
body.putString("type", "outside");

client.doFetch(urlPath, "POST", null, body, null);
```

### Sử dụng chức năng
Mỗi khi người dùng sử dụng 1 chức năng mà hệ thống xác định là cần ghi nhận để tracking hoặc tính điểm tích luỹ, Core sẽ gửi thông tin qua API cho Game Platform.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/user/action" \
  --user 'defaultkey:' \
  --data '{"userId":"1234","action":"pay_invoice","data":{"amount":10000}}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/action";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
    action: 'pay_invoice',
    data: { amount: 10000 },
});

return client.doFetch(urlPath, "POST", queryParams, body, {})
```

```java tab="Java"
String urlPath = "/v2/account/user/action";

JSONObject body = new JSONObject();
body.putString("userId", "1234");
body.putString("action", "pay_invoice");

JSONObject data = new JSONObject();
data.putLong("amount", 10000L);
body.putJSONObject("data", data);

client.doFetch(urlPath, "POST", null, body, null);
```
### Lấy thông tin tích điểm
Core có thể sử dụng API để lấy thông tin về điểm tích lũy, level và các thông tin khác của user.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/user/loyalty" \
  --user 'defaultkey:' \
  --data '{"userId":"1234"}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/loyalty";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
});

let result = await client.doFetch(urlPath, "POST", queryParams, body, {})
console.log(result)
```

```java tab="Java"
String urlPath = "/v2/account/user/loyalty";

JSONObject body = new JSONObject();
body.putString("userId", "1234");

JSONObject ret = client.doFetch(urlPath, "POST", null, body, null);
System.out.println(ret.toString());
```
### Đổi điểm
Điểm số của người dùng được tính bằng Gold, là đơn vị tiền ảo trong Game Platform. Người dùng có thể đổi Gold thành tiền thật trong ví. Khi người dùng có yêu cầu đổi điểm, Core sẽ gửi Rest API cho Game Platform để thực hiện việc đổi điểm và trừ số Gold trong tài khoản của người dùng.

Core có thể sử dụng API để lấy thông tin về điểm tích lũy, level và các thông tin khác của user.

```sh tab="cURL"
curl "http://127.0.0.1:7350/v2/account/user/exchange_gold" \
  --user 'defaultkey:' \
  --data '{"userId":"1234","gold":100,"money":100000}'
```

```js tab="JavaScript"
const urlPath = "/v2/account/user/exchange_gold";

const queryParams = {};

const body = JSON.stringify({
    userId: '1234',
    gold: 100,
    money: 100000,
});

return client.doFetch(urlPath, "POST", queryParams, body, {})
```

```java tab="Java"
String urlPath = "/v2/account/user/exchange_gold";

JSONObject body = new JSONObject();
body.putString("userId", "1234");
body.putLong("gold", 100L);
body.putLong("money", 100000L);

client.doFetch(urlPath, "POST", null, body, null);
```

## Core Apis

Các API này yêu cầu ViettelPay cung cấp để Game Platform có thể gọi qua Core.

### Notification

Khi có sự kiện, Game Platform gọi Rest API để báo cho Core gửi notification cho user, các sự kiện được đề xuất  bao gồm:

* Người dùng hoàn thiện hoặc gần hoàn thiện một nhiệm vụ

* Người dùng tích lũy đủ điểm để lên hạng

* Friend của người dùng đạt điểm số vượt người dùng trên bảng leader board

