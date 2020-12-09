# User accounts

Một người dùng đại diện cho một id trong máy chủ. Mọi người dùng đều được đăng ký và có một profile để những người dùng khác tìm và trở thành [friend](social-friends.md) hoặc tham gia [group](social-groups-clans.md) và [chat](storage-access-controls.md).

Một người dùng có thể sở hữu [hồ sơ](storage-access-controls.md), chia sẻ thông tin công khai với những người dùng khác và xác thực thông qua nhiều nhà cung cấp xã hội khác nhau.

## Lấy thông tin tài khoản

Khi người dùng đăng nhập thành công và có session, bạn có thể truy xuất tài khoản của họ. Profile chứa nhiều thông tin bao gồm các nhà cung cấp xã hội khác nhau.

```sh tab="cURL"
curl -X GET "http://127.0.0.1:7350/v2/account" \
  -H 'authorization: Bearer <session token>'
```

```js tab="JavaScript"
const account = await client.getAccount(session);
const user = account.user;
console.info("User id '%o' and username '%o'.", user.id, user.username);
console.info("User's wallet:", account.wallet);
```

```csharp tab=".NET"
var account = await client.GetAccountAsync(session);
var user = account.User;
System.Console.WriteLine("User id '{0}' username '{1}'", user.Id, user.Username);
System.Console.WriteLine("User wallet: '{0}'", account.Wallet);
```

```csharp tab="Unity"
var account = await client.GetAccountAsync(session);
var user = account.User;
Debug.LogFormat("User id '{0}' username '{1}'", user.Id, user.Username);
Debug.LogFormat("User wallet: '{0}'", account.Wallet);
```

```cpp tab="Cocos2d-x C++"
auto successCallback = [](const NAccount& account)
{
  CCLOG("User's wallet: %s", account.wallet.c_str());
};
client->getAccount(session, successCallback);
```

```js tab="Cocos2d-x JS"
client.getAccount(session)
  .then(function(account) {
      cc.log("User's wallet:", account.wallet);
    },
    function(error) {
      cc.error("get account failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto successCallback = [](const NAccount& account)
{
  CCLOG("User's wallet: %s", account.wallet.c_str());
};
client->getAccount(session, successCallback);
```

```java tab="Java"
Account account = client.getAccount(session);
User user = account.getUser();
System.out.format("User id %s username %s", user.getId(), user.getUsername());
System.out.format("User wallet %s", account.getWallet());
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let message = SelfFetchMessage()
client.send(message: message).then { selfuser in
  NSLog("User id '%@' and handle '%@'", selfuser.id, selfuser.handle)
  NSLog("User has JSON metadata '%@'", selfuser.metadata)
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
GET /v2/account
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Bearer <session token>
```

Một số thông tin như wallet, device Id và customer Id là riêng tư nhưng một phần của profile được hiển thị cho những người dùng khác.

| Public field | Mô tả |
| ----- | ----------- |
| user.id | id duy nhất cho mỗi người dùng |
| user.username | nickname duy nhất cho mỗi người dùng |
| user.display_name | Tên hiển thị của người dùng (mặc định là trống) |
| user.avatar_url | URL hình ảnh đại diện của người dùng (mặc định là trống) |
| user.lang | Cài đặt ngôn ngữ của người dùng (mặc định là "en") |
| user.location | Vị trí của người dùng (mặc định là trống) |
| user.timezone | Timezone của người dùng (mặc định là trống) |
| user.metadata | Lưu các thông tin bổ sung của người dùng - chỉ đọc từ client |
| user.edge_count | Số lượng friend của người dùng |
| user.facebook_id | Facebook id liên kết với người dùng |
| user.google_id | Google id liên kết với người dùng |
| user.gamecenter_id | GameCenter id liên kết với người dùng |
| user.steam_id | Steam id liên kết với người dùng |
| user.create_time | Thời gian tài khoản được tạo |
| user.update_time | Thời gian tài khoản được cập nhật |
| user.online | Giá trị xác định người dùng đang trực tuyến hoặc không |

| Private fields | Description |
| ----- | ----------- |
| email | Email address associated with this user. |
| devices | List of device IDs associated with this user. |
| custom_id | Custom identifier associated with this user. |
| wallet | User's wallet - only readable from the client. |
| verifiy_time | A timestamp for when the user was verified (currently only used by Facebook). |

### User metadata

You can store additional fields for a user in `user.metadata` which is useful to share data you want to be public to other users. Metadata is limited to 16KB per user. This can be set only via the [script runtime](runtime-code-basics.md), similar to the `wallet`.

!!! Tip
    We recommend you choose user metadata to store very common fields which other users will need to see. For all other information you can store records with [public read permissions](storage-collections.md) which other users can find.

### Virtual wallet

Itme-platform has the concept of a virtual wallet and transaction ledger. Itme-platform allows developers to create, update and list changes to the user's wallet. This operation has transactional guarantees and is only achievable with the [script runtime](runtime-code-basics.md).

With server-side code it's possible to update the user's wallet.

```lua tab="Lua"
local nk = require("nakama")

local user_id = "95f05d94-cc66-445a-b4d1-9e262662cf79"
local content = {
  reward_coins = 1000 -- Add 1000 coins to the user's wallet.
}

local status, err = pcall(nk.wallet_update, user_id, content)
if (not status) then
    nk.logger_error(("User wallet update error: %q"):format(err))
end
```

```go tab="Go"
userID := "8f4d52c7-bf28-4fcf-8af2-1d4fcf685592"
content := map[string]interface{}{
	"reward_coins": 1000, // Add 1000 coins to the user's wallet.
}
if err := nk.WalletUpdate(ctx, userID, content, nil, true); err != nil {
	logger.Error("User wallet update error: %v", err.Error())
}
```

The wallet is private to a user and cannot be seen by other users. You can fetch wallet information for a user via [Fetch Account](user-accounts.md#fetch-account) operation.

### Online indicator

Itme-platform can report back user online indicators in two ways:

1. [Fetch user](user-accounts.md#fetch-users) information. This will give you a quick snapshot view of the user's online indicator and is not a reliable way to detect user presence.
2. Publish and subscribe to user [status presence](social-status.md) updates. This will give you updates when the online status of the user changes (along side a custom message).

## Fetch users

You can fetch one or more users by their IDs or handles. This is useful for displaying public profiles with other users.

```sh tab="cURL"
curl -X GET "http://127.0.0.1:7350/v2/user?ids=userid1&ids=userid2&usernames=username1&usernames=username2&facebook_ids=facebookid1" \
  -H 'authorization: Bearer <session token>'
```

```js tab="JavaScript"
const users = await client.getUsers(session, ["user_id1"], ["username1"], ["facebookid1"]);
users.foreach(user => {
  console.info("User id '%o' and username '%o'.", user.id, user.username);
});
```

```csharp tab=".NET"
var ids = new[] {"userid1", "userid2"};
var usernames = new[] {"username1", "username2"};
var facebookIds = new[] {"facebookid1"};
var result = await client.GetUsersAsync(session, ids, usernames, facebookIds);
foreach (var u in result.Users)
{
    System.Console.WriteLine("User id '{0}' username '{1}'", u.Id, u.Username);
}
```

```csharp tab="Unity"
var ids = new[] {"userid1", "userid2"};
var usernames = new[] {"username1", "username2"};
var facebookIds = new[] {"facebookid1"};
var result = await client.GetUsersAsync(session, ids, usernames, facebookIds);
foreach (var u in result.Users)
{
    Debug.LogFormat("User id '{0}' username '{1}'", u.Id, u.Username);
}
```

```cpp tab="Cocos2d-x C++"
auto successCallback = [](const NUsers& users)
{
  for (auto& user : users.users)
  {
    CCLOG("User id '%s' username %s", user.id.c_str(), user.username.c_str());
  }
};
client->getUsers(session,
    { "user_id1" },
    { "username1" },
    { "facebookid1" },
    successCallback);
```

```js tab="Cocos2d-x JS"
client.getUsers(session, ["user_id1"], ["username1"], ["facebookid1"])
  .then(function(users) {
      cc.log("Users:", JSON.stringify(users));
    },
    function(error) {
      cc.error("get users failed:", JSON.stringify(error));
    });
```

```cpp tab="C++"
auto successCallback = [](const NUsers& users)
{
  for (auto& user : users.users)
  {
    std::cout << "User id '" << user.id << "' username " << user.username << std::endl;
  }
};
client->getUsers(session,
    { "user_id1" },
    { "username1" },
    { "facebookid1" },
    successCallback);
```

```java tab="Java"
List<String> ids = Arrays.asList("userid1", "userid2");
List<String> usernames = Arrays.asList("username1", "username1");
String[] facebookIds = new String[] {"facebookid1"};
Users users = client.getUsers(session, ids, usernames, facebookIds).get();

for (User user : users.getUsersList()) {
  System.out.format("User id %s username %s", user.getId(), user.getUsername());
}
```

```swift tab="Swift"
// Requires Itme-platform 1.x
let userID // a User ID
var message = UsersFetchMessage()
message.userIDs.append(userID)
client.send(message: message).then { users in
  for user in users {
    NSLog("User id '%@' and handle '%@'", user.id, user.handle)
  }
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
GET /v2/user?ids=userid1&ids=userid2&usernames=username1&usernames=username2&facebook_ids=facebookid1
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Bearer <session token>
```

You can also fetch one or more users in server-side code.

```lua tab="Lua"
local nk = require("nakama")

local user_ids = {
  "3ea5608a-43c3-11e7-90f9-7b9397165f34",
  "447524be-43c3-11e7-af09-3f7172f05936"
}
local users = nk.users_get_id(user_ids)
for _, u in ipairs(users)
do
  local message = ("username: %q, displayname: %q"):format(u.username, u.display_name)
  nk.logger_info(message)
end
```

```go tab="Go"
if users, err := nk.UsersGetId(ctx, []string{
	"3ea5608a-43c3-11e7-90f9-7b9397165f34",
	"447524be-43c3-11e7-af09-3f7172f05936",
}); err != nil {
	// Handle error.
} else {
	for _, u := range users {
		logger.Info("username: %s, displayname: %s", u.Username, u.DisplayName)
	}
}
```

## Update account

When a user is registered most of their profile is setup with default values. A user can update their own profile to change fields but cannot change any other user's profile.

```sh tab="cURL"
curl -X PUT "http://127.0.0.1:7350/v2/account" \
  -H 'authorization: Bearer <session token>' \
  --data '{
    "display_name": "My new name",
    "avatar_url": "http://graph.facebook.com/avatar_url",
    "location": "San Francisco"
  }'
```

```js tab="JavaScript"
await client.updateAccount(session, {
  display_name: "My new name",
  avatar_url: "http://graph.facebook.com/avatar_url",
  location: "San Francisco"
});
```

```csharp tab=".NET"
const string displayName = "My new name";
const string avatarUrl = "http://graph.facebook.com/avatar_url";
const string location = "San Francisco";
await client.UpdateAccountAsync(session, null, displayName, avatarUrl, null, location);
```

```csharp tab="Unity"
const string displayName = "My new name";
const string avatarUrl = "http://graph.facebook.com/avatar_url";
const string location = "San Francisco";
await client.UpdateAccountAsync(session, null, displayName, avatarUrl, null, location);
```

```cpp tab="Cocos2d-x C++"
client->updateAccount(session,
    opt::nullopt,
    "My new name", // display name
    "http://graph.facebook.com/avatar_url", // avatar URL
    opt::nullopt,
    "San Francisco" // location
    );
```

```js tab="Cocos2d-x JS"
client.updateAccount(session, {
  display_name: "My new name",
  avatar_url: "http://graph.facebook.com/avatar_url",
  location: "San Francisco"
});
```

```cpp tab="C++"
client->updateAccount(session,
    opt::nullopt,
    "My new name", // display name
    "http://graph.facebook.com/avatar_url", // avatar URL
    opt::nullopt,
    "San Francisco" // location
    );
```

```java tab="Java"
String displayName = "My new name";
String avatarUrl = "http://graph.facebook.com/avatar_url";
String location = "San Francisco";
client.updateAccount(session, null, displayName, avatarUrl, null, location);
```

```swift tab="Swift"
// Requires Itme-platform 1.x
var message = SelfUpdateMessage()
message.avatarUrl = "http://graph.facebook.com/avatar_url"
message.fullname = "My New Name"
message.location = "San Francisco"
client.send(message: message).then {
  NSLog("Successfully updated yourself.")
}.catch { err in
  NSLog("Error %@ : %@", err, (err as! NakamaError).message)
}
```

```tab="REST"
PUT /v2/account HTTP/1.1
Host: 127.0.0.1:7350
Accept: application/json
Content-Type: application/json
Authorization: Bearer <session token>

{
  "display_name": "My new name",
  "avatar_url": "http://graph.facebook.com/avatar_url",
  "location": "San Francisco"
}
```

With server-side code it's possible to update any user's profile.

```lua tab="Lua"
local nk = require("nakama")

local user_id = "4ec4f126-3f9d-11e7-84ef-b7c182b36521" -- some user's id.
local metadata = {}
local username = "my-new-username"
local display_name = "My new Name"
local timezone = nil
local location = "San Francisco"
local lang_tag = nil
local avatar_url = "http://graph.facebook.com/avatar_url"

local status, err = pcall(nk.account_update_id, user_id, metadata, username, display_name, timezone, location, lang_tag, avatar_url)
if (not status) then
  nk.logger_info(("Account update error: %q"):format(err))
end
```

```go tab="Go"
userID := "4ec4f126-3f9d-11e7-84ef-b7c182b36521" // some user's id.
username := "my-new-username" // must be unique
metadata := make(map[string]interface{})
displayName := "My new name"
timezone := ""
location := "San Francisco"
langTag := ""
avatarUrl := "http://graph.facebook.com/avatar_url"
if err := nk.AccountUpdateId(ctx, userID, username, metadata, displayName, timezone, location, langTag, avatarUrl); err != nil {
	// Handle error.
	logger.Error("Account update error: %s", err.Error())
}
```
