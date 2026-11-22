---
uid: e7d94467-faeb-4a80-a9a1-5b20169b2c7a
slug: first-dockerfile
title: "Dockerfile đầu tiên cho portfolio"
estimatedMinutes: 35
objectives:
  - "Viết Dockerfile với FROM, COPY, EXPOSE"
  - "Build image bằng docker build và chạy portfolio trong container"
  - "Dùng .dockerignore để không đưa file thừa và file bí mật vào image"
prerequisites:
  - actions-docker/docker-basics/docker-commands
authors: [github:TienNHM]
---

# Dockerfile đầu tiên cho portfolio

Chương 3 chạy image người khác làm sẵn. Giờ bạn tự làm image chứa **portfolio của bạn**: lấy `nginx:1.30-alpine` làm nền rồi chép trang web vào.

**Dockerfile** là file văn bản liệt kê từng bước tạo image. Docker đọc từ trên xuống.

## Viết Dockerfile

Ở thư mục gốc repo portfolio (cùng chỗ với `index.html`), tạo file tên đúng là `Dockerfile`, không có đuôi:

```dockerfile
# Ảnh nền: nginx bản 1.30 trên Alpine Linux
FROM nginx:1.30-alpine

# Chép trang web vào thư mục nginx phục vụ mặc định
COPY . /usr/share/nginx/html

# Cổng nginx lắng nghe bên trong container (chỉ để ghi chú)
EXPOSE 80
```

| Lệnh | Ý nghĩa |
| --- | --- |
| `FROM` | Image nền. Mọi Dockerfile bắt đầu bằng `FROM`. |
| `COPY <nguồn> <đích>` | Chép file từ máy bạn (thư mục build) vào image. `.` là cả thư mục hiện tại. |
| `EXPOSE 80` | Ghi chú rằng container lắng nghe cổng 80. Nó **không** tự mở cổng; bạn vẫn cần `-p` khi chạy. |

Không cần `CMD`: image `nginx` đã có sẵn lệnh khởi động nginx, image của bạn thừa hưởng nó.

## Tạo .dockerignore

`COPY .` chép **mọi thứ** trong thư mục, kể cả `.git` và file `.env` chứa mật khẩu (chương 6). Mọi thứ trong `/usr/share/nginx/html` đều được nginx phục vụ cho người xem, nên một file `.env` lọt vào image thì ai cũng tải được nó ở `http://<trang-của-bạn>/.env`.

Tạo `.dockerignore` cạnh `Dockerfile`, cú pháp giống `.gitignore`:

```text
.git
.github
.env
Dockerfile
.dockerignore
compose.yaml
README.md
```

## Build

```bash
docker build -t portfolio:dev .
```

- `-t portfolio:dev`: đặt tên và tag cho image.
- `.` ở cuối: **thư mục build** (build context), nơi Docker tìm `Dockerfile` và lấy file cho `COPY`. Đừng quên dấu chấm.

Output kết thúc bằng dòng `naming to docker.io/library/portfolio:dev` là xong. Kiểm tra:

```bash
docker image ls portfolio
```

## Chạy

```bash
docker run -d --name portfolio -p 8080:80 portfolio:dev
```

Mở <http://localhost:8080>: portfolio của bạn, giờ chạy trong container. Thử mở <http://localhost:8080/Dockerfile>: phải là trang **404**, nhờ `.dockerignore`.

Sửa `index.html` thì phải **build lại** và chạy container mới, vì image đã đóng băng bản cũ:

```bash
docker rm -f portfolio
docker build -t portfolio:dev .
docker run -d --name portfolio -p 8080:80 portfolio:dev
```

## Khi lỗi

| Lỗi | Cách sửa |
| --- | --- |
| `failed to read dockerfile: open Dockerfile: no such file or directory` | Đang đứng sai thư mục, hoặc file tên `Dockerfile.txt` (Windows hay ẩn đuôi file). |
| `'docker buildx build' requires 1 argument` | Quên dấu `.` ở cuối lệnh build. |
| Trang hiện **Welcome to nginx!** thay vì portfolio | `index.html` không nằm ở thư mục gốc. Nếu trang nằm trong `site/`, dùng `COPY site/ /usr/share/nginx/html`. |
| `port is already allocated` | Container cũ còn chạy: `docker rm -f portfolio`. |

## Tóm tắt

- `FROM` chọn image nền, `COPY` đưa file vào, `EXPOSE` chỉ là ghi chú.
- `docker build -t tên:tag .` (nhớ dấu chấm), rồi `docker run -p`.
- Luôn có `.dockerignore`, ít nhất bỏ `.git` và `.env`.
