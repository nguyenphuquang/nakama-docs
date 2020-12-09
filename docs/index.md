[itme-platform_logo]: images/itme-platform-logo.png "Itme-platform Logo"

# Itme-platform server

Itme-platform là một server mạng xã hội, game thời gian thực.

Với Itme-platform, bạn có thể đăng nhập, kết bạn, lưu trữ và trao đổi dữ liệu giữa các ứng dụng và game. Itme-platform là 1 server đóng gói, cho phép bạn mở rộng bằng cách viết thêm các plugin nên rất thích hợp trong việc kết nối với các hạ tầng hoặc ứng dụng có sẵn, bổ sung thêm các chức năng gamification và đưa mini-games vào trong ứng dụng đang có.

Server được thiết kế để có thể scale cho số lượng lớn người dùng.

Trong quá trình phát triển, bạn có thể chạy server trên các hệ điều hành macOS, Linux hay Windows. Khi deploy cho môi trường production, bạn có thể sử dụng docker để deploy một cách nhanh chóng và dễ dàng.

## Các tính năng chính

Bạn chỉ cần tập trung vào phát triển logic, server của chúng tôi sẽ xử lý tất cả các phần còn lại, bao gồm [user accounts](user-accounts.md), [social profiles](authentication.md#social-providers), [realtime chat](social-realtime-chat.md), [data storage](storage-collections.md), [multiplayer matches](gameplay-multiplayer-realtime.md), và nhiều chức năng khác nữa.

<div style="display: flex">
  <div style="flex: 1; margin: 0 1em 0 0">
    <strong>User accounts</strong>
    <p>Mỗi <a href="./user-accounts/">user</a> được đăng ký có 1 tài khoản lưu trữ các thông tin về điểm số, gold, coin và có thể kết bạn với các user khác cũng như tham gia vào các nhóm chat.</p>
  </div>
  <div style="flex: 1">
    <strong>Friends</strong>
    <p><a href="./social-friends/">Bạn bè</a> là phương thức hiệu quả nhất để xây dựng cộng đồng.</p>
  </div>
</div>

<div style="display: flex">
  <div style="flex: 1; margin: 0 1em 0 0">
    <strong>Groups &amp; Clans</strong>
    <p><a href="./social-groups-clans/">Group</a> tập hợp người dùng thành các cộng đồng nhỏ hoặc xây dựng đội nhóm trong game.</p>
  </div>
  <div style="flex: 1">
    <strong>Realtime chat</strong>
    <p>Người dùng có thể <a href="./social-realtime-chat/">chat</a> với người khác theo các chế độ: chát 1 vs 1, chat trong group hoặc trong chat room.</p>
  </div>
</div>

<div style="display: flex">
  <div style="flex: 1; margin: 0 1em 0 0">
    <strong>In-app notifications</strong>
    <p><a href="./social-in-app-notifications/">In-app notifications</a> giúp bạn dễ dàng gửi thông điệp từ user đến các user khác.</p>
  </div>
  <div style="flex: 1">
    <strong>Leaderboards</strong>
    <p><a href="./gameplay-leaderboards/">Leaderboards</a> là cách thức hiệu quả nhất để tăng tính cạnh tranh trong các game và app của bạn.</p>
  </div>
</div>

<div style="display: flex">
  <div style="flex: 1; margin: 0 1em 0 0">
    <strong>Matchmaker</strong>
    <p><a href="./gameplay-matchmaker/">Matchmaker</a> người dùng dễ dàng tìm được bạn chơi trong các game thời gian thực hoặc game theo lượt.</p>
  </div>
  <div style="flex: 1">
    <strong>Multiplayer</strong>
    <p><a href="./gameplay-multiplayer-realtime/">Multiplayer engine</a> giúp bạn dễ dàng phát triển các game nhiều người chơi, Game Platform sẽ quảng lý việc thiết lập các trận đấu, trao đổi dữ liệu giữa nhiều người chơi trong trận đấu đó.</p>
  </div>
</div>

__Server-side code__

Để bổ sung logic cho server, bạn có thể viết các plugin bằng [ngôn ngữ Lua](runtime-code-basics.md).

This is useful to run custom logic which isn't running on the device or browser. The code you deploy with the server can be used immediately by clients so you can change behavior on the fly and add new features faster.
