## Giới thiệu
- VNPAY game client được cung cấp dưới dạng SDK (iOS & Android SDK)
- Tài liệu này mô tả cách sử dụng và các hàm SDK cung cấp

## Kết nối thư viện

### Android 

Mở build.gradle thêm implementation files('<path>/GameSDK.aar')

### IOS 
Vào general -> Frameworks, Libraries, Embedded content
add GameSDK.framework

## Sử dụng thư viện để kết nối Game Client

GameController sử dụng để kết nối giữa ứng dụng ví VNPay với GameSDK và được khởi tạo sau khi đăng nhập thành công vào màn hình Home của Ví

```
val gameController = GameController()
```

#### API ‘getListGame'
- Dùng lấy danh sách game
- Request: none
- Response:  [{Game}]


|Field|Description|Type
| ------ | ------ | ------ |
|id|id của game|string
|name|Tên của game|string
|groupid|id nhóm game|number
|logo|url ảnh logo của game|string
|cover|url ảnh cover của game|string

Example Code
Android 
```
gameController.getListGame(){ games: List<Game> ->
	//TODO
}
```

IOS
```
gameController.getListGame({ completion: ([Game]?) -> Void in 
})
```
#### API ‘startGame'
Dùng để mở 1 game lên

- Request: context, url, token

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

- Response: None

Example Code

Android 
```
gameController.startGame(context, url, token)
```

IOS
```
gameController.startGame(parent: self, url, token)
```

#### API ‘setOnMessageReceive'

Đăng ký hàm gọi lại (callback) để gửi thông báo từ game đến app ví VNPAY, sử dụng trong các trường hợp

- Khi người dùng chọn sử dụng Voucher QR quay được.
- Ngưởi dùng sử dụng Lì xì
- Các trường hợp giao tiếp khác

- Request: function callback
- Response: Chuỗi json kiểu string

Example Code

Android 
```
gameController.setOnMessageReceive { jsonString ->
	//TODO
}
```

IOS
```
gameController.setOnMessageReceive { jsonString in
	//TODO
}
```

#### API ‘sendMessage'

Dùng để gửi dữ liệu vào game client

- Request: Chuỗi json kiểu string
- Response: None

Example Code
```
gameController.sendMessage(“jsonString”)
```
