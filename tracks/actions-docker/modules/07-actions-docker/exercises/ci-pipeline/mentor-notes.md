## Khi review bài này

- Mở file `.github/workflows/docker.yml` trước khi xem lần chạy. Kiểm tra theo thứ tự: `permissions` cấp workflow, `permissions` trong `push-image`, `if: github.event_name == 'push'`, `needs: test`.
- Tìm `pull_request_target` trong mọi workflow của repo (`.github/workflows/*`). Có là chưa đạt tiêu chí an toàn, dù workflow chạy được.
- Tìm `echo` có `secrets.` hoặc `set -x` trong step có secret.
- Chạy `docker pull ghcr.io/<owner>/<repo>:main` khi chưa đăng nhập GHCR. Lỗi `unauthorized` hoặc `denied` thường nghĩa là package còn private.
- Mở link PR trong README: job `push-image` phải hiện **skipped**.

## Lỗi hay gặp

- Ghi `packages: write` ở cấp workflow thay vì cấp job: chạy được nhưng job `test` cũng có quyền ghi. Góp ý, chấm thiếu ở tiêu chí an toàn.
- Quên `.env.example` hoặc thiếu biến, nên job `test` đỏ ở bước `up`. Hướng dẫn đọc log bước "In log khi có lỗi".
- `grep -q "<h1>"` không khớp với portfolio của thành viên (trang không có thẻ `h1`). Gợi ý đổi chữ cần tìm, không bỏ hẳn bước kiểm tra.

## Góp ý nên tránh

- Bắt ghim action bằng SHA hay bật Dependabot: bài học chỉ giới thiệu, không bắt buộc.
