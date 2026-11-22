---
uid: 8644f02f-d530-484d-8c49-c16f6c306d0a
slug: test-compose-in-ci
title: "Chạy cả stack trong CI"
estimatedMinutes: 35
objectives:
  - "Chạy docker compose trên runner GitHub Actions"
  - "Kiểm tra stack bằng docker compose up -d --wait và curl"
  - "In log khi lỗi và dọn dẹp với if: failure() và if: always()"
prerequisites:
  - actions-docker/compose/umami-postgres
  - actions-docker/deploy-pages/link-check
authors: [github:TienNHM]
---

# Chạy cả stack trong CI

Ở chương 6, bạn tự gõ `docker compose up -d --wait` rồi mở trình duyệt để xem mọi thứ có chạy không. Giờ để GitHub Actions làm việc đó **mỗi lần push và mỗi pull request**: ai sửa `compose.yaml` hay `Dockerfile` làm hỏng stack là biết ngay.

Runner `ubuntu-latest` có sẵn Docker và Docker Compose, không cần cài gì.

## Workflow

Tạo `.github/workflows/docker.yml`:

```yaml
name: Docker

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - name: Tạo .env cho CI từ file mẫu
        run: cp .env.example .env

      - name: Chạy cả stack, chờ đến khi healthy
        run: docker compose up -d --build --wait

      - name: Kiểm tra portfolio và Umami
        run: |
          curl -fsS http://localhost:8080 | grep -q "<h1>"
          curl -fsS http://localhost:3000/api/heartbeat

      - name: In log khi có lỗi
        if: failure()
        run: docker compose logs

      - name: Dọn dẹp
        if: always()
        run: docker compose down -v
```

## Đọc hiểu

**`.env` trong CI.** `.env` không có trong repo (đúng như vậy). CI chép `.env.example` thành `.env`: mật khẩu mẫu là đủ, vì database này chỉ sống vài phút trên runner rồi bị xóa. Nhờ vậy CI còn kiểm tra luôn rằng `.env.example` đủ biến.

**`up -d --build --wait`.** Build image portfolio từ `Dockerfile`, chạy cả ba service, và **chờ tới khi mọi healthcheck xanh**. Service nào không healthy, hoặc thoát, thì lệnh trả exit code khác 0 và step đỏ. Đây là lý do chương 6 viết healthcheck cho cả ba service.

**`curl -fsS`:**

| Cờ | Ý nghĩa |
| --- | --- |
| `-f` | Trả lỗi (exit code 22) khi HTTP status từ 400 trở lên. Không có `-f`, `curl` coi trang 404 là thành công. |
| `-s` | Không in thanh tiến trình |
| `-S` | Nhưng vẫn in thông báo lỗi khi có lỗi |

`| grep -q "<h1>"` kiểm tra trang trả về đúng là portfolio, không phải trang lỗi. Đổi `<h1>` thành một chữ chắc chắn có trong `index.html` của bạn.

**`if: failure()`** chỉ chạy khi có step trước đó đỏ: in log của mọi service để bạn đọc nguyên nhân ngay trong log Actions. **`if: always()`** chạy dù xanh hay đỏ. Runner vốn bị xóa sau job nên dọn dẹp ở đây chủ yếu là thói quen tốt; trên máy chủ dùng lâu dài (self-hosted runner) thì bắt buộc.

## Thử làm hỏng

Mở pull request đổi cổng Umami thành `"3000:3001"` (Lỗi 1 ở chương 6). `up --wait` vẫn xanh vì healthcheck chạy bên trong container, nhưng step **Kiểm tra** đỏ:

```text
curl: (52) Empty reply from server
Error: Process completed with exit code 52.
```

Healthcheck kiểm tra "bên trong có chạy không"; `curl` từ runner kiểm tra "người dùng có vào được không". Cần cả hai.

## Tóm tắt

- Runner có sẵn Docker; CI tạo `.env` từ `.env.example`.
- `docker compose up -d --build --wait` đỏ khi có service không healthy.
- `curl -fsS` kiểm tra từ ngoài vào; `if: failure()` in log, `if: always()` dọn dẹp.
