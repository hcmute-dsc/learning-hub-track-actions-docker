Hoàn thiện repo portfolio của bạn với pipeline đầy đủ, theo bài "Hướng dẫn dự án cuối".

## Yêu cầu

- **Pages:** `pages.yml` kiểm tra link hỏng bằng lychee, rồi đóng gói (job `build`) và deploy (job `deploy`, `needs: build`) lên GitHub Pages khi push lên `main`; trên pull request chỉ kiểm tra.
- **Docker:** `Dockerfile` dùng image nền có tag cụ thể; `.dockerignore` bỏ `.git` và `.env`.
- **Compose:** `compose.yaml` chạy portfolio, Umami và Postgres; có healthcheck cho cả ba và `depends_on` với `condition: service_healthy`; named volume cho Postgres; `.env` ngoài Git, có `.env.example`. Portfolio có gắn mã theo dõi Umami.
- **CI:** `docker.yml` test cả stack (`up --wait` + `curl -f`) trên push và pull request, push image lên GHCR (public) chỉ khi push lên `main`.
- **An toàn:** `permissions` tối thiểu, không `pull_request_target`, không in secret, mọi action và image ghi phiên bản.
- **README:** link Pages, hai badge, cách chạy trên máy (kể cả tạo `.env`), cách chạy image từ GHCR, mô tả pipeline.

## Nộp bài

Link repo GitHub public. Trong README, thêm mục **Bằng chứng** gồm:

- link một lần chạy xanh của mỗi workflow trên `main`;
- link một pull request có `check-links` và `test` chạy, `build`, `deploy` và `push-image` bị bỏ qua;
- ảnh chụp trang thống kê Umami có lượt xem portfolio trên máy bạn.

## Tiêu chí chấm

Mentor clone repo, làm đúng theo README, rồi chấm theo rubric trong bài tập:

| Tiêu chí | Đạt khi |
| --- | --- |
| Pages và kiểm tra link | Trang Pages chạy; check-links chặn được deploy khi có link hỏng |
| Dockerfile | Build được; `.dockerignore` đúng; tag cụ thể |
| Compose | Clone mới, `cp .env.example .env`, `up -d --wait` là cả 3 service healthy |
| CI và GHCR | `docker.yml` xanh; image public pull về chạy được |
| An toàn | Đi qua đủ danh sách tự kiểm tra ở bài "Workflow an toàn" |
| README | Người chưa biết gì về repo làm theo được; có badge và bằng chứng |
