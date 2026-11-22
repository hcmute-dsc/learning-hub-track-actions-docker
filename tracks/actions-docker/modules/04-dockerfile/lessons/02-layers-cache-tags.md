---
uid: 6a038f0e-2627-4976-a4e0-6ef9354785b3
slug: layers-cache-tags
title: "Layer, cache và tag"
estimatedMinutes: 30
objectives:
  - "Giải thích image được xếp từ các layer và cache giúp build nhanh thế nào"
  - "Sắp thứ tự lệnh trong Dockerfile để tận dụng cache"
  - "Đặt tag phiên bản cho image"
prerequisites:
  - actions-docker/dockerfile/first-dockerfile
authors: [github:TienNHM]
---

# Layer, cache và tag

## Layer

Image được xếp từ nhiều **layer** (lớp). Mỗi lệnh `FROM`, `COPY`, `RUN` trong Dockerfile tạo một layer chứa phần thay đổi của bước đó. Xem các layer của image portfolio:

```bash
docker history portfolio:dev
```

Các dòng trên cùng là của bạn (`COPY . /usr/share/nginx/html`, `EXPOSE 80`); các dòng bên dưới đến từ image `nginx:1.30-alpine`. Nhiều image dùng chung một image nền thì chỉ cần lưu layer nền một lần.

## Cache

Build lại mà không sửa gì:

```bash
docker build -t portfolio:dev .
```

Trong output, các bước có chữ `CACHED`: Docker thấy đầu vào của bước giống lần trước nên dùng lại layer cũ, gần như không tốn thời gian. Giờ sửa `index.html` rồi build lại: bước `COPY . /usr/share/nginx/html` chạy lại vì file đã đổi.

Luật quan trọng: **một bước đổi thì mọi bước sau nó đều chạy lại**. Vì vậy thứ **ít đổi để lên trên**, thứ **hay đổi để xuống dưới**.

Portfolio tĩnh chỉ có một bước `COPY`, nên chưa thấy rõ. Với một app Node.js, thứ tự đúng trông như sau:

```dockerfile
FROM node:24-alpine
WORKDIR /app
# 1. Chỉ chép file khai báo thư viện, rồi cài thư viện
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
# 2. Rồi mới chép code, thứ bạn sửa mỗi ngày
COPY . .
CMD ["npm", "start"]
```

Sửa code thì chỉ bước `COPY . .` chạy lại; `npm ci` (bước chậm) vẫn `CACHED`. Nếu đảo thành `COPY . .` trước `npm ci`, mỗi lần sửa một dòng code là cài lại toàn bộ thư viện.

## Tag

Tag giúp phân biệt các phiên bản của **cùng một** image:

```bash
docker build -t portfolio:1.0 .
docker tag portfolio:1.0 portfolio:latest
docker image ls portfolio
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
portfolio    1.0       1d33cef8f57c   10 seconds ago   93.5MB
portfolio    latest    1d33cef8f57c   10 seconds ago   93.5MB
```

`docker tag` không tạo bản sao: hai tag cùng trỏ vào một IMAGE ID. Quy ước hay dùng:

| Tag | Dùng khi |
| --- | --- |
| `1.0`, `1.1`… | Phiên bản phát hành, không bao giờ ghi đè |
| `main` | Bản mới nhất từ nhánh `main` |
| `sha-1a2b3c4` | Đúng một commit, dễ truy ngược |
| `latest` | Bản mới nhất; tiện nhưng không cho biết là bản nào |

Chương 7 để GitHub Actions tự gắn các tag `main`, `sha-…` và phiên bản khi build.

Dọn image cũ: `docker image prune` xóa các image không còn tên (thường sinh ra sau nhiều lần build lại).

## Tóm tắt

- Mỗi lệnh tạo một layer; layer không đổi được lấy từ cache.
- Một bước đổi thì các bước sau chạy lại: ít đổi để trên, hay đổi để dưới.
- Tag cụ thể như `1.0` cho bản phát hành; `docker tag` thêm tên mới cho cùng một image.
