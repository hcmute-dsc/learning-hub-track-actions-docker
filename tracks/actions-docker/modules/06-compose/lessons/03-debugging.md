---
uid: 4ca8ed08-5851-4776-9250-c3a0c36bbb1b
slug: compose-debugging
title: "Gỡ lỗi Compose: 4 lỗi hay gặp"
estimatedMinutes: 35
objectives:
  - "Nhận ra và sửa lỗi sai cổng, thiếu .env, database chưa sẵn sàng và sai mật khẩu với volume cũ"
  - "Dùng docker compose ps và logs để tìm nguyên nhân"
prerequisites:
  - actions-docker/compose/umami-postgres
authors: [github:TienNHM]
---

# Gỡ lỗi Compose: 4 lỗi hay gặp

Mỗi mục dưới đây là một lỗi được **cố tình tạo ra**. Hãy làm theo để thấy lỗi tận mắt, đọc thông báo, rồi sửa. Buổi online thứ hai cũng luyện đúng các lỗi này.

Quy trình chung khi stack không chạy:

1. `docker compose ps -a`: service nào không `healthy`, service nào `Exited`?
2. `docker compose logs <service>`: đọc từ dưới lên, như log GitHub Actions.
3. Sửa, rồi `docker compose up -d --wait`.

## Lỗi 1: sai cổng

Sửa cổng của Umami thành `"3000:3001"` rồi chạy lại:

```bash
docker compose up -d --wait
curl http://localhost:3000/api/heartbeat
```

```text
curl: (52) Empty reply from server
```

`docker compose ps` vẫn báo Umami **healthy**, vì healthcheck chạy **bên trong** container, gọi đúng cổng 3000. Lỗi nằm ở chỗ nối cổng: máy bạn gửi vào cổng 3001 của container, nơi không có gì lắng nghe.

**Sửa:** số bên phải của `ports` phải là cổng ứng dụng lắng nghe trong container (Umami: 3000, nginx: 80). Số bên trái tuỳ bạn chọn.

## Lỗi 2: thiếu .env

Đổi tên `.env` thành `.env.bak` rồi chạy:

```text
level=warning msg="The \"POSTGRES_PASSWORD\" variable is not set. Defaulting to a blank string."
env file /.../portfolio/.env not found: ...
```

Hai thông báo, hai cơ chế ở bài trước: `${POSTGRES_PASSWORD}` không có giá trị (chỉ là cảnh báo, nguy hiểm vì dễ bị bỏ qua), và `env_file: .env` không tìm thấy file (lỗi, dừng luôn).

**Sửa:** `cp .env.example .env` rồi điền giá trị. Ai mới clone repo về cũng gặp lỗi này, nên README phải ghi bước tạo `.env`.

## Lỗi 3: database chưa sẵn sàng

Đổi `depends_on` của Umami thành dạng ngắn, bỏ điều kiện healthy:

```yaml
    depends_on:
      - db
```

Xóa dữ liệu cũ để Postgres phải khởi tạo lại từ đầu (lúc chậm nhất), rồi chạy:

```bash
docker compose down -v
docker compose up -d
docker compose ps -a
docker compose logs umami
```

Có thể thấy Umami `Exited (1)`, log có dòng:

```text
Raw query failed. Code: `N/A`. Message: `Can't reach database server at 172.25.0.2:5432`
```

Compose khởi động Umami ngay khi container `db` **bắt đầu chạy**, lúc Postgres còn đang khởi tạo. Lỗi kiểu này lúc có lúc không, tuỳ máy nhanh hay chậm, nên rất khó đoán nếu không biết nguyên nhân.

**Sửa:** trả lại `condition: service_healthy` và healthcheck `pg_isready` cho `db`.

## Lỗi 4: đổi mật khẩu khi volume đã có

Đổi `POSTGRES_PASSWORD` trong `.env` sang giá trị mới, rồi:

```bash
docker compose down
docker compose up -d --wait
```

```text
container portfolio-umami-1 exited (1)
```

Log của Umami:

```text
Raw query failed. Code: `28P01`. Message: `password authentication failed for user "umami"`
```

Postgres chỉ đặt mật khẩu **lần đầu khởi tạo volume** (chương 5). Volume cũ vẫn giữ mật khẩu cũ, còn Umami dùng mật khẩu mới.

**Sửa:** trả lại mật khẩu cũ. Hoặc, nếu dữ liệu không quan trọng, `docker compose down -v` để xóa volume và khởi tạo lại với mật khẩu mới.

## Bảng tra nhanh

| Triệu chứng | Xem ở đâu | Nguyên nhân hay gặp |
| --- | --- | --- |
| Trình duyệt không mở được trang, service vẫn healthy | `ports` trong `compose.yaml` | Sai cổng bên phải, hoặc cổng bên trái đã bị chiếm |
| `variable is not set` | `.env` | Thiếu `.env` hoặc thiếu biến, sai tên biến |
| Service `Exited (1)` ngay sau khi chạy | `docker compose logs <service>` | Database chưa sẵn sàng, sai mật khẩu, sai `DATABASE_URL` |
| `port is already allocated` | `docker ps` | Container khác (có thể của project khác) đang giữ cổng |
| Sửa code mà trang không đổi | lệnh đã chạy | Quên `--build` |

## Tóm tắt

- `ps -a` tìm service hỏng, `logs` tìm nguyên nhân.
- Healthy bên trong chưa chắc truy cập được từ ngoài: kiểm tra `ports`.
- Database cần healthcheck và `condition: service_healthy`.
- Mật khẩu Postgres gắn với volume.
