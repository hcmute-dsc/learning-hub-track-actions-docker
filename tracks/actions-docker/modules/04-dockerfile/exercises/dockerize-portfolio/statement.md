Đóng gói portfolio của bạn thành image Docker.

## Yêu cầu

- `Dockerfile` ở repo portfolio, dùng image nền có tag cụ thể (ví dụ `nginx:1.30-alpine`, không dùng `latest`).
- `.dockerignore` bỏ ít nhất `.git` và `.env`.
- Build và chạy được bằng đúng các lệnh ghi trong README; mở `http://localhost:8080` thấy portfolio đủ CSS và ảnh.
- Build ít nhất một image có tag phiên bản (ví dụ `portfolio:1.0`).

## Nộp bài

Link repo GitHub public (hoặc link pull request thêm Dockerfile). README có mục **Chạy bằng Docker** gồm:

- lệnh `docker build` và `docker run`;
- ảnh chụp trang chạy ở `localhost:8080`;
- output của `docker image ls portfolio` (dán dạng text).

## Tiêu chí chấm

Mentor build và chạy lại theo README của bạn, rồi chấm theo rubric:

| Tiêu chí | Đạt khi |
| --- | --- |
| Build và chạy được | Làm theo README là thấy portfolio ở cổng 8080 |
| `.dockerignore` | Có `.git` và `.env`; mở `/Dockerfile` hay `/.env` trên trang trả về 404 |
| Image nền và tag | Image nền ghi tag cụ thể; có image tag phiên bản |
| README | Có lệnh, ảnh chụp và output `docker image ls` |
