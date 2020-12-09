### Server to server

API của Game server sử dụng rpc để làm việc giữa client với server. Kết nối server to server cũng có thể thực hiện được thông qua http_keys. Các điểm cần lưu ý của kết nối dạng này bao gồm:

* Các request từ VNPay đến Game server cần sử dụng http_key do Game server cung cấp

* Dữ liệu trao đổi được encode json 2 lần, ví dụ: {"request_id": "1234"} khi gửi qua Game server qua rest api sẽ được encode thành "{\"request_id\": \"1234\"}". Quy tắc này được áp dụng trong cả dữ liệu request và response (Tham khảo trong các ví dụ sử dụng bằng cURL)

* Các API sẽ được cung cấp kèm ví dụ bằng cURL để tham khảo

### Lấy danh sách game group

Game server chia các loại game thành nhiều nhóm, giữa các nhóm. Hiện tại, lượt chơi người chơi được tặng sẽ tính theo nhóm.

!!! Tip
    http_key trong ví dụ bằng cURL có thể chạy được với server test. 

```shell
curl "http://itme.mana.vn:7350/v2/rpc/list_game_group?http_key=defaultkey_http_123" \
     -v \
     --data '"{\"request_id\": \"1234\"}"' \
     -H 'Content-Type: application/json' \
     -H 'Accept: application/json'
```

Dữ liệu trả về

```shell
{"payload":"{\"code\":\"00\",\"game_groups\":[{\"id\":1,\"name\":\"Game trúng thưởng\"}],\"request_id\":\"1234\"}"}
```

Payload
```shell
{
  "code": "00",
  "game_groups": [
    {
      "id": 1,
      "name": "Game trúng thưởng"
    }
  ],
  "request_id": "1234"
}
```

#### Request
| Field          | Description                                          | Type                     |
| -------------- | -----------------------------------------------------| ------------------------ |
| request_id     |  request_id do VNPay gửi để tracking request         | string                   |


#### Response
| Field          | Description                                      | Type              |
| -------------- | -------------------------------------------------| ------------------|
| code           |  Mã trả về: 00 - Thành công, 04 - Lỗi            | string            |
| game_groups    |  Danh sách các game group                        | game_group[]      |

#### game_group
| Field          | Description                                                      |
| -------------- | -----------------------------------------------------------------|
| id             |  id của group                                                    |
| name           |  Tên của group                                                   |


### Lấy danh sách game

```shell
curl "http://itme.mana.vn:7350/v2/rpc/list_game?http_key=defaultkey_http_123" \
     -v \
     --data '"{\"request_id\": \"1234\"}"' \
     -H 'Content-Type: application/json' \
     -H 'Accept: application/json'
```

Dữ liệu trả về dạng text

```shell
{"payload":"{\"code\":\"00\",\"games\":[{\"cover\":\"https://t4.ftcdn.net/jpg/02/07/81/77/240_F_207817790_tfeI0FsBKhJNpcpS78jxmF7UMRaJkrhT.jpg\",\"groupid\":1,\"id\":\"quay-thuong\",\"logo\":\"https://t4.ftcdn.net/jpg/02/07/81/77/240_F_207817790_tfeI0FsBKhJNpcpS78jxmF7UMRaJkrhT.jpg\",\"name\":\"Quay thưởng\"}],\"request_id\":\"1234\"}"} 
```

Payload

```json
{
  "code": "00",
  "games": [
    {
      "cover": "https: //t4.ftcdn.net/jpg/02/07/81/77/240_F_207817790_tfeI0FsBKhJNpcpS78jxmF7UMRaJkrhT.jpg",
      "groupid": 1,
      "id": "quay-thuong",
      "logo": "https: //t4.ftcdn.net/jpg/02/07/81/77/240_F_207817790_tfeI0FsBKhJNpcpS78jxmF7UMRaJkrhT.jpg",
      "name": "Quaythưởng"
    }
  ],
  "request_id": "1234"
}
```

#### Request

| Field          | Description                                      | Type                     |
| -------------- | ------------------------------------------------ | ------------------------ |
| request_id     |  request_id do VNPay gửi để tracking request     | string                   |


#### Response

| Field          | Description                                  | Type              |
| -------------- | ---------------------------------------------| ------------------|
| code           |  Mã trả về: 00 - Thành công, 04 - Lỗi        | string            |
| games          |  Danh sách game                              | game[]            |

#### game

| Field          | Description                                  | Type              |
| -------------- | ---------------------------------------------| ------------------|
| id             |  id của game                                 | string            |
| name           |  Tên của game                                | string            |
| groupid        |  id nhóm game                                | number            |
| logo           |  url ảnh logo của game                       | string            |
| cover          |  url ảnh cover của game                      | string            |

### Thêm lượt chơi

VNPay gọi sang Game server thông báo cộng thêm lượt chơi cho người chơi

```shell
curl "http://itme.mana.vn:7350/v2/rpc/add_turn?http_key=defaultkey_http_123" \
     -v \
     --data '"{\"request_id\": \"1234\", \"turns\": 10, \"description\": \"Test request\", \"group\": 1, \"token\":\"7826248d60fc4c7cbb080f31c2baaaed\"}"' \
     -H 'Content-Type: application/json' \
     -H 'Accept: application/json'
```

Dữ liệu trả về dạng text

```shell
{"payload":"{\"code\":\"00\",\"request_id\":\"1234\"}"}
```

Payload

```json
{
  "code": "00",
  "request_id": "1234"
}
```

#### Request

| Field          | Description                                      | Type                     |
| -------------- | ------------------------------------------------ | ------------------------ |
| request_id     |  request_id do VNPay gửi để tracking request     | string                   |
| turns          |  Số lượt chơi cộng thêm của người chơi           | number                   |
| description    |  Mô tả về lý do cộng lượt                        | string                   |
| group          |  Cộng lượt cho group game nào                    | string                   |
| token          |  Game token của người chơi                       | string                   |
| acc_no         |  Số tài khoản của người chơi                     | string                   |

VNPay cung cấp 1 trong 2 field là token hoặc acc_no. Sử dụng acc_no sẽ nhanh hơn nếu người chơi đã đăng ký game.

#### Response

| Field          | Description                                  | Type              |
| -------------- | ---------------------------------------------| ------------------|
| code           |  Mã trả về: 00 - Thành công, 04 - Lỗi        | string            |
| request_id     |  request_id do VNPay gửi đi                  | string            |
