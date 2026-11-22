---
uid: 8e95bb56-acf9-44f2-9241-d8ed7667c440
slug: docker-commands
title: "Chạy, xem, vào trong, dọn dẹp"
estimatedMinutes: 40
objectives:
  - "Chạy container nền với docker run -d, đặt tên và mở cổng bằng -p"
  - "Truyền biến môi trường bằng -e"
  - "Dùng ps, logs, exec, stop, rm để quản lý container"
prerequisites:
  - actions-docker/docker-basics/image-container
authors: [github:TienNHM]
---

# Chạy, xem, vào trong, dọn dẹp

Bài này dùng image `nginx:1.30-alpine`: web server nginx trên Alpine Linux, chính là thứ chương 4 dùng để chạy portfolio.

Trên Windows, gõ các lệnh Docker trong **PowerShell** hoặc terminal của WSL. Git Bash tự đổi các đường dẫn bắt đầu bằng `/` (như `/usr/share/nginx/html`) thành đường dẫn Windows, làm lệnh `exec` bên dưới báo lỗi `No such file or directory`.

## Chạy một web server

```bash
docker run -d --name web -p 8080:80 nginx:1.30-alpine
```

| Phần | Ý nghĩa |
| --- | --- |
| `-d` | Chạy nền (detached), trả lại terminal cho bạn |
| `--name web` | Đặt tên container là `web`. Không đặt thì Docker tự đặt tên ngẫu nhiên. |
| `-p 8080:80` | Nối cổng **8080 trên máy bạn** với cổng **80 trong container** |
| `nginx:1.30-alpine` | Image dùng để chạy |

Mở <http://localhost:8080>: trang **Welcome to nginx!**.

`-p` luôn là `cổng-máy-bạn:cổng-trong-container`. nginx lắng nghe cổng 80 bên trong container. Bên ngoài, bạn chọn cổng nào cũng được, miễn còn trống.

## Xem container

```bash
docker ps
```

```text
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                  NAMES
3f2a1c9d8e7b   nginx:1.30-alpine   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   0.0.0.0:8080->80/tcp   web
```

`docker ps -a` hiện cả container đã dừng.

## Xem log

```bash
docker logs web
docker logs -f web      # theo dõi liên tục, Ctrl+C để thoát
```

Tải lại trang <http://localhost:8080> vài lần: mỗi lần là một dòng `"GET / HTTP/1.1" 200` mới trong log. Khi container không chạy như ý, `docker logs` là chỗ xem đầu tiên, giống đọc log của GitHub Actions.

## Vào bên trong

```bash
docker exec web ls /usr/share/nginx/html
```

```text
50x.html
index.html
```

`exec` chạy một lệnh trong container đang chạy. Muốn mở hẳn một shell để đi lại bên trong:

```bash
docker exec -it web sh
```

`-it` giữ bàn phím và màn hình nối với shell. Gõ `exit` để ra; container vẫn chạy.

## Biến môi trường với -e

Nhiều image đọc cấu hình từ biến môi trường. Thử với `alpine`:

```bash
docker run --rm -e TEN=An alpine:3.22 sh -c 'echo "Xin chào $TEN"'
```

```text
Xin chào An
```

`--rm` tự xóa container khi nó chạy xong, khỏi phải dọn. Chương 5 dùng `-e` để đặt mật khẩu cho Postgres.

## Dừng và xóa

```bash
docker stop web      # dừng
docker rm web        # xóa container đã dừng
docker rm -f web     # dừng và xóa luôn
```

Container đã xóa thì mọi file được tạo **bên trong** nó cũng mất. Chương 5 học cách giữ dữ liệu bằng volume.

Dọn image không dùng nữa: `docker image ls` để xem, `docker image rm <tên:tag>` để xóa.

## Khi lỗi

| Lỗi | Nguyên nhân và cách sửa |
| --- | --- |
| `Bind for 0.0.0.0:8080 failed: port is already allocated` | Cổng 8080 đang bị dùng, thường bởi container cũ. `docker ps` để tìm, `docker rm -f <tên>`, hoặc đổi sang `-p 8081:80`. |
| `Conflict. The container name "/web" is already in use` | Đã có container tên `web` (có thể đang dừng). `docker rm -f web` rồi chạy lại. |
| `Cannot connect to the Docker daemon` | Docker Desktop chưa mở. Mở nó, chờ biểu tượng cá voi báo đang chạy. |
| Trang không mở được dù `docker ps` thấy container | Sai thứ tự `-p`: phải là `8080:80`, không phải `80:8080`. |

## Tóm tắt

| Lệnh | Làm gì |
| --- | --- |
| `docker run -d --name web -p 8080:80 <image>` | Chạy nền, đặt tên, mở cổng |
| `docker run --rm -e KEY=value <image> <lệnh>` | Chạy một lần, truyền biến môi trường, tự xóa |
| `docker ps` / `docker ps -a` | Container đang chạy / tất cả |
| `docker logs -f web` | Xem log |
| `docker exec -it web sh` | Mở shell bên trong |
| `docker rm -f web` | Dừng và xóa |
