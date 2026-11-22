Chạy portfolio cùng Umami và Postgres bằng Docker Compose trên máy bạn, gắn mã theo dõi Umami vào portfolio. Đánh dấu từng mục khi xong.

Bài này chuẩn bị cho chương 7: CI sẽ chạy đúng file `compose.yaml` này để kiểm tra tự động.

<details>
<summary>Gợi ý nếu bị lỗi</summary>

- Làm theo quy trình ở bài "Gỡ lỗi Compose": `docker compose ps -a`, rồi `docker compose logs <service>`.
- Umami `Exited (1)` với `password authentication failed`: bạn đã đổi mật khẩu sau khi volume được tạo. Xem Lỗi 4.
- Cổng 8080 hoặc 3000 đã bị dùng: đổi số bên trái trong `ports`, ví dụ `"8081:80"`.

</details>
