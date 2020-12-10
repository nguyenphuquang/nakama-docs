Itme-platform Documentation
====================

> Documentation for Itme-platform social and realtime server.

This project uses Markdown for documentation which is compiled with [mkdocs](http://www.mkdocs.org).

## Install and setup

```shell
pip install pip --upgrade
pip install mkdocs --upgrade
pip install mkdocs-material --upgrade
```

*Note:* For Python 3.7, you may need to run the following instead
```shell
pip3 install pip --upgrade
pip3 install mkdocs --upgrade
pip3 install mkdocs-material --upgrade
```

## Development

```
mkdocs serve
```
Thư Viện Kết Nối Game Client
====================
## Kết nối thư viện

Android 
Mở build.gradle thêm implementation files('<path>/GameSDK.aar')

IOS 
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
```
Android 
gameController.startGame(context, url, token)
```
```
IOS
gameController.startGame(parent: self, url, token)
```

#### API ‘setOnMessageReceive'

Dùng để đăng ký remote từ game client

- Request: function callback
- Response: Chuỗi json kiểu string

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

#### API ‘sendMessage'

Dùng để gửi dữ liệu vào game client

- Request: Chuỗi json kiểu string
- Response: None

Example Code
```
gameController.sendMessage(“jsonString”)
```


 

