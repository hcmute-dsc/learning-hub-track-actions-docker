## Khi review bài này

- Clone repo và chạy đúng các lệnh trong README, không sửa gì. Không chạy được thì chưa đạt tiêu chí đầu tiên, dù Dockerfile trông đúng.
- Sau khi chạy, mở thêm `http://localhost:8080/.env`, `/Dockerfile`, `/.git/config`. Cả ba phải là 404.
- Xem `FROM`: `nginx` hay `nginx:latest` là chưa đạt tiêu chí image nền.
- Repo dùng thư mục con (ví dụ `site/`) thì `COPY site/ /usr/share/nginx/html` là đúng; đừng bắt đổi cấu trúc.

## Lỗi hay gặp

- File tên `Dockerfile.txt` do Windows ẩn đuôi file.
- Link CSS bắt đầu bằng `/` chạy đúng ở localhost nhưng hỏng trên GitHub Pages (chương 2). Nhắc nếu thấy.
- Có `CMD` hoặc `RUN nginx` thừa làm container thoát ngay. Gợi ý xem `docker logs`.

## Góp ý nên tránh

- Yêu cầu multi-stage build hay cấu hình nginx riêng: ngoài phạm vi chương này.
