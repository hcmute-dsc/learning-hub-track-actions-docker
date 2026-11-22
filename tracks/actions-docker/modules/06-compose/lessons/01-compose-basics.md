---
uid: 990fe99b-62b9-4749-90b4-d875c3ffa116
slug: compose-basics
title: "compose.yaml đầu tiên"
estimatedMinutes: 25
objectives:
  - "Viết compose.yaml có một service build từ Dockerfile"
  - "Dùng docker compose up, ps, logs, down"
prerequisites:
  - actions-docker/dockerfile/first-dockerfile
  - actions-docker/volumes-networks/networks
authors: [github:TienNHM]
---

# compose.yaml đầu tiên

Ở chương 5, chạy hai container nói chuyện với nhau đã cần cả chục lệnh. **Docker Compose** gom tất cả vào một file YAML, rồi một lệnh dựng cả hệ thống, một lệnh gỡ.

## Một service

Ở thư mục gốc repo portfolio (cạnh `Dockerfile`), tạo `compose.yaml`:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
```

- `services`: danh sách các container cần chạy. Mỗi mục là một **service**.
- `web`: tên service, cũng là tên máy trong network Compose tạo (chương 5).
- `build: .`: build image từ `Dockerfile` ở thư mục hiện tại. Dùng image có sẵn thì ghi `image: nginx:1.30-alpine` thay cho `build`.
- `ports`: giống `-p 8080:80`. Luôn để trong dấu nháy kép.

So với `docker build` + `docker run -d --name ... -p 8080:80 ...`, file này ngắn hơn và nằm trong Git: ai clone repo về cũng chạy được giống bạn.

## Các lệnh

```bash
docker compose up -d          # build nếu cần, tạo network, chạy mọi service ở chế độ nền
docker compose ps             # xem service đang chạy
docker compose logs -f web    # xem log của service web
docker compose down           # dừng và xóa container, network
```

Mở <http://localhost:8080>: portfolio.

Sửa `index.html` thì chạy `docker compose up -d --build` để build lại image. Không có `--build`, Compose dùng lại image đã build lần trước.

Lệnh Compose phải chạy **trong thư mục có `compose.yaml`**. Tên container có dạng `<thư-mục>-<service>-1`, ví dụ `portfolio-web-1`.

## `docker compose`, không phải `docker-compose`

Bản cũ là chương trình riêng tên `docker-compose` (có gạch nối, file thường tên `docker-compose.yml`). Bản hiện tại là lệnh con `docker compose` (dấu cách), đi kèm Docker Desktop. Tài liệu cũ trên mạng còn dùng cách viết cũ; ý nghĩa giống nhau.

## Tóm tắt

- `compose.yaml` mô tả các service: `build` hoặc `image`, `ports`…
- `up -d` dựng, `ps` xem, `logs` đọc log, `down` gỡ.
- Sửa code thì `up -d --build`.
