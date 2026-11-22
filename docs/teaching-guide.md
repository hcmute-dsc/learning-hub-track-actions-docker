# Hướng dẫn cho mentor: 2 buổi online

Track **GitHub Actions và Docker: từ portfolio đến pipeline** chủ yếu là tự học trên [Learning Hub](https://learn.hcmute-dsc.io.vn/tracks/actions-docker). Hai buổi online, mỗi buổi 2 giờ, dùng để gỡ chỗ kẹt và làm cùng nhau những phần khó tự học nhất.

## Chuẩn bị chung

- **Người học:** đã học xong Git/GitHub và có repo portfolio HTML/CSS trên GitHub.
- **Mentor:** một mentor cho mỗi 5–6 người học trong phòng nhóm (breakout room). Mỗi mentor tự làm hết track một lần trước buổi 1, trên chính repo portfolio của mình.
- **Công cụ:** nền tảng họp có breakout room và chia sẻ màn hình; kênh Discord của track để người học gửi link và ảnh chụp.
- **Repo demo:** người dạy chuẩn bị một repo portfolio mẫu (vài trang HTML/CSS), chưa có workflow hay Dockerfile, để viết trực tiếp trong buổi.
- Trước mỗi buổi, xem trên trang review của Learning Hub ai đã nộp bài tập nào, để chia phòng nhóm (người chậm ở cùng mentor có kinh nghiệm).

## Buổi 1: Actions, Pages, Docker đầu tiên

**Người học làm trước buổi:** chương 1–3, cài Docker theo tab **Cài đặt**, nộp bài "Cài Docker và chạy hello-world" (ảnh chụp trong kênh Discord) và "Portfolio lên mạng" (link Pages).

| Thời gian | Hoạt động |
| --- | --- |
| 0:00–0:10 | Điểm danh theo danh sách nộp bài: ai đã có trang Pages, ai còn kẹt cài Docker. Người kẹt cài đặt ghi lỗi vào kênh Discord; một mentor hỗ trợ riêng trong phòng nhóm trong lúc phần còn lại học tiếp. |
| 0:10–0:45 | **Live:** viết `pages.yml` từ đầu trên repo demo (chương 2): bật Pages, job deploy, rồi thêm `check-links` với `needs`. Sau đó **cố tình làm hỏng** một step (ví dụ `run: ls thu-muc-khong-co`, hoặc link `abuot.html`) và đọc log cùng nhau theo 3 bước của chương 1. |
| 0:45–0:55 | Nghỉ |
| 0:55–1:45 | **Docker:** demo 10 phút `run -d -p`, `ps`, `logs`, `exec`, `rm -f` với `nginx:1.30-alpine`. Rồi chia phòng nhóm: mỗi người viết `Dockerfile` và `.dockerignore` cho portfolio của mình (chương 4, bài 1), build và mở `localhost:8080`. Mentor đi qua từng người. |
| 1:45–2:00 | Hỏi đáp. Giao tự học chương 4–6 trước buổi 2; nhắc nộp bài "Đóng gói portfolio bằng Docker". |

**Mẹo cho buổi 1:**

- Người dùng Windows mà lệnh `docker exec ... ls /usr/share/...` báo đường dẫn `C:/Program Files/Git/...`: đang dùng Git Bash. Chuyển sang PowerShell (tab **Cài đặt**, mục 2).
- Ai chưa cài được Docker thì cho mở GitHub Codespaces ngay trong buổi (tab **Cài đặt**, mục 5) để không bị tụt lại phần hands-on.
- Lỗi hay gặp khi build: quên dấu `.` cuối lệnh, file tên `Dockerfile.txt`, `index.html` nằm trong thư mục con.

## Buổi 2: Compose, gỡ lỗi, khép vòng CI

**Người học làm trước buổi:** chương 4–6, nộp "Đóng gói portfolio bằng Docker", làm thử bài "Portfolio, Umami và Postgres trên máy".

| Thời gian | Hoạt động |
| --- | --- |
| 0:00–0:10 | **Quiz nhanh** 5 câu, dựa trên "Kiểm tra: Docker Compose" và "Kiểm tra: volume và network" (dùng tính năng poll của nền tảng họp, hoặc cho làm quiz trên Learning Hub rồi xem kết quả). Câu nào nhiều người sai thì giải thích ngay. |
| 0:10–0:40 | **Viết `compose.yaml` cùng nhau** trên repo demo: bắt đầu với service `web`, thêm `db` (volume, healthcheck), rồi `umami` (`env_file`, `DATABASE_URL`, `depends_on` với `condition: service_healthy`). Tạo `.env`, `.env.example`, thêm `.env` vào `.gitignore`. `docker compose up -d --wait`, đăng nhập Umami, gắn mã theo dõi. |
| 0:40–1:10 | **Hands-on và luyện gỡ lỗi** trong phòng nhóm. Mỗi người chạy stack của mình, rồi mentor giao lần lượt 3 lỗi của bài "Gỡ lỗi Compose": **sai cổng** (`3000:3001`), **thiếu `.env`**, **db chưa sẵn sàng** (bỏ `condition: service_healthy`, `down -v` rồi `up`). Người học tự tìm nguyên nhân bằng `ps -a` và `logs` trước khi mentor gợi ý. |
| 1:10–1:20 | Nghỉ |
| 1:20–1:50 | **Khép vòng:** viết `docker.yml` trên repo demo (chương 7): job `test` với `up --wait` + `curl`, job `push-image` lên GHCR với `permissions: packages: write`. Push lên `main`, xem image xuất hiện ở **Packages**. Mở một PR để thấy `push-image` bị bỏ qua. Dành 5 phút cuối nói về `pull_request_target` và không in secret (chương 7, bài 3). |
| 1:50–2:00 | Giao dự án cuối (chương 8): nhắc rubric, hạn nộp, cách nộp trên Learning Hub, và chia mentor phụ trách review. |

**Mẹo cho buổi 2:**

- Lần `up` đầu tiên phải tải image Umami và Postgres. Nhắc người học `docker compose pull` trước buổi để không mất thời gian.
- Lỗi "db chưa sẵn sàng" lúc có lúc không, tuỳ máy. Máy nhanh không thấy lỗi thì giải thích nguyên nhân rồi chuyển sang lỗi khác, đừng mất thời gian cố tái hiện.
- Ai đổi mật khẩu trong `.env` giữa chừng sẽ gặp `password authentication failed` (Lỗi 4). Đây là dịp tốt để nói về volume.
- Repo demo của người dạy nên **public** để không tốn phút Actions và để package GHCR chuyển public được.

## Review dự án

- Bài `project` (chương 4, 7, 8) được review trên trang review của Learning Hub. Mỗi bài có `mentor-notes.md` (chỉ mentor thấy trên hub) với cách kiểm tra và lỗi hay gặp.
- Luôn chạy lại theo README của người học trên một thư mục mới clone, không đoán thay họ.
- Nhận xét nêu ít nhất một điểm làm tốt, một điểm nên sửa, gắn với tiêu chí trong rubric.
- Repo track này công khai: không ghi đáp án hay thông tin cá nhân của người học vào repo.
