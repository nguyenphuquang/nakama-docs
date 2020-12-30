## Giới thiệu¶

- VNPAY game client được cung cấp dưới dạng SDK (iOS & Android SDK)
- Tài liệu này mô tả cách sử dụng và các hàm SDK cung cấp


## Kết nối thư viện

Android 
Mở build.gradle thêm implementation files('[path]/GameSDK.aar')

IOS 
Vào general -> Frameworks, Libraries, Embedded content
add GameSDK.framework

## Sử dụng thư viện để kết nối Game Client

GameController sử dụng để kết nối giữa ứng dụng ví VNPay với GameSDK và được khởi tạo sau khi đăng nhập thành công vào màn hình Home của Ví

```
val gameController = GameController(httpUrl)

```
httpUrl: url game server

#### phương thức ‘getListGame'
- Dùng lấy danh sách game
- Fields: none
- Return:  [{Game}]


|Field|Description|Type
| ------ | ------ | ------ |
|id|id của game|string
|name|Tên của game|string
|groupid|id nhóm game|number
|logo|url ảnh logo của game|string
|cover|url ảnh cover của game|string

Example Code
```
Android 
gameController.getListGame(){ games: List<Game> ->
	//TODO
}
```
```
IOS
gameController.getListGame({ completion: ([Game]?) -> Void in 
})
```
#### Phương thức ‘startGame'
Dùng để mở 1 game lên

- Fields: context, url, token

Android

|Variable|Description|Type
| ------ | ------ | ------ |
|context|context của activity|Context
|url|url của game|String
|token|token của Ví|String

IOS

|Variable|Description|Type
| ------ | ------ | ------ |
|parent|ViewController chứa game|UIViewController
|url|url của game|String
|token|token của Ví|String

- Return: None

Example Code
```
Android 
gameController.startGame(context, url, token)
```
```
IOS
gameController.startGame(parent: self, url, token)
```

#### Phương thức ‘setOnMessageReceive'

Dùng để đăng ký remote từ game client

- Fields: function callback
- Callback: Chuỗi json kiểu string

Example Code
```
Android 
gameController.setOnMessageReceive { jsonString ->
	//TODO
}
```
```
IOS
gameController.setOnMessageReceive { jsonString in
	//TODO
}
```
Mô tả object data trả về
|action|code|Description
| ------ | ------ | ------ |
|use-gift|03|Sữ dụng quà khi trạng thái ví không hợp lệ (Active A, Active B)
|use-gift|06|Sữ dụng quà khi khách hàng chưa thực hiện KYC
|use-gift|07|Sữ dụng quà khi khách hàng chưa liên kết ngân hàng

#### Phương thức ‘sendMessage'

Dùng để gửi dữ liệu vào game client

- Fields: Chuỗi json kiểu string
- Return: None

Example Code
```
gameController.sendMessage(“jsonString”)
```
