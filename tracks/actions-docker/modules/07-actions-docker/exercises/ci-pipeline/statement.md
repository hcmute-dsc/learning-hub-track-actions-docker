Thêm workflow `docker.yml` vào repo portfolio: kiểm tra cả stack Compose trên mỗi push và pull request, rồi push image lên GHCR khi có push lên `main`.

## Yêu cầu

- Job `test`: tạo `.env` từ `.env.example`, `docker compose up -d --build --wait`, kiểm tra bằng `curl -f` cả portfolio và Umami, in log khi lỗi, dọn dẹp.
- Job `push-image`: `needs: test`, chỉ chạy với event `push`, đăng nhập GHCR bằng `GITHUB_TOKEN`, build và push bằng `docker/build-push-action` với tag từ `docker/metadata-action`.
- `permissions: contents: read` ở cấp workflow; chỉ job `push-image` có `packages: write`.
- Mọi action ghi phiên bản. Không dùng `pull_request_target`.
- Package trên GHCR để **public**.

## Nộp bài

Link repo GitHub public. README có:

- link một lần chạy xanh của `docker.yml` trên `main` (cả hai job);
- link một pull request mà job `test` chạy, `push-image` bị bỏ qua (có thể là PR bạn tự mở rồi đóng);
- lệnh `docker pull ghcr.io/<bạn>/<repo>:main` và `docker run` để chạy image từ GHCR.

## Tiêu chí chấm

| Tiêu chí | Đạt khi |
| --- | --- |
| Test stack trong CI | Job `test` dùng `--wait` và `curl -f`, có in log khi lỗi; lần chạy trên PR xanh |
| Push lên GHCR | Image có tag `main` và `sha-…`; mentor `docker pull` về chạy được |
| An toàn | Permissions đúng như yêu cầu; push chỉ chạy với event `push`; không in secret |
| README | Có đủ link lần chạy, link PR và lệnh chạy image |
