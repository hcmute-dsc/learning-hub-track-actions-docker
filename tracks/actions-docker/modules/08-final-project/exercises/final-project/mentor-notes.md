## Khi review bài này

Làm theo thứ tự, ghi chú cho từng tiêu chí của rubric:

1. **Đọc README như người lạ.** Chỉ dùng những gì README ghi. Thiếu bước tạo `.env` hay sai lệnh là trừ ở tiêu chí README, kể cả khi bạn tự đoán ra được.
2. **Clone mới và chạy:** `cp .env.example .env`, `docker compose up -d --wait`, `docker compose ps`. Cả 3 service phải healthy. Mở `localhost:8080`, `localhost:3000`. Xong thì `docker compose down -v`.
3. **Xem workflow:** mở hai file trong `.github/workflows/`, kiểm tra `permissions`, các `if`, `needs`, phiên bản action. Tìm `pull_request_target`, `echo` có `secrets.`, `set -x`.
4. **Xem bằng chứng:** link lần chạy xanh trên `main`, link PR có `build`, `deploy` và `push-image` bị **skipped**. Job `deploy` của Pages phải có `needs: build` (upload và deploy cùng một job hay báo không tìm thấy artifact).
5. **GHCR:** `docker logout ghcr.io` (nếu đã đăng nhập) rồi `docker pull ghcr.io/<owner>/<repo>:main`, chạy thử với `-p 8080:80`.
6. **Lịch sử `.env`:** trong thư mục đã clone, `git log --all --oneline -- .env` phải không in gì. Nếu có, hướng dẫn thành viên đổi mọi mật khẩu từng nằm trong đó; xóa file ở commit mới không đủ.

## Lỗi hay gặp

- Mã theo dõi Umami trỏ tới `localhost:3000` nên trang trên GitHub Pages không đếm được lượt xem. Đó là giới hạn đã nói trong bài học, **không trừ điểm**; ảnh chụp Umami trên máy là đủ.
- Link CSS bắt đầu bằng `/` chạy ở localhost nhưng hỏng trên Pages.
- `docker.yml` chỉ có `on: push` nên PR không được kiểm tra.
- Healthcheck tự viết gọi sai cổng hoặc dùng lệnh không có trong image, nên service không bao giờ healthy và `up --wait` treo tới khi hết `retries`. Xem lý do bằng `docker inspect --format '{{json .State.Health}}' <container>`.

## Góp ý nên tránh

- Đòi deploy Umami lên máy chủ thật, hay dùng công cụ ngoài track (Kubernetes, Terraform…).
- Chấm thiết kế giao diện portfolio. Track này chấm pipeline, không chấm CSS.

## Khi nhận xét

Nêu ít nhất một điểm làm tốt và một điểm nên sửa, gắn với tiêu chí cụ thể. Bài chưa đạt: ghi rõ tiêu chí nào và cách kiểm tra lại, để thành viên sửa và nộp lại.
