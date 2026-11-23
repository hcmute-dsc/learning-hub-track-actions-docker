# GitHub Actions và Docker: từ portfolio đến pipeline

Repo nội dung của **một track** trên [HCMUTE DSC Learning Hub](https://learn.hcmute-dsc.io.vn). Mỗi track một repo `hcmute-dsc/learning-hub-track-<slug>`, cùng tạo từ template [`learning-hub-track-template`](https://github.com/hcmute-dsc/learning-hub-track-template) ([ADR-10](https://github.com/hcmute-dsc/learning-hub-docs/blob/main/docs/product/DECISIONS.md#adr-10--mỗi-track-một-repo-learning-hub-content-thành-registry)).

## Track này dạy gì

Người học đã xong Git/GitHub và có một portfolio HTML/CSS. Portfolio là dự án xuyên suốt 8 chương; phần lớn tự học, kèm 2 buổi online × 2 giờ.

| Chương | Nội dung | Bài tập |
| --- | --- | --- |
| 1. GitHub Actions cơ bản | workflow, job, step, runner, đọc log | quiz · self_check |
| 2. Deploy portfolio lên GitHub Pages | `actions/deploy-pages`, kiểm tra link hỏng bằng lychee | quiz · self_check |
| 3. Docker là gì | image, container, `run`/`ps`/`logs`/`exec`/`rm`, `-p`, `-e` | self_check (hello-world) · quiz |
| 4. Dockerfile | `nginx:alpine`, layer, cache, `.dockerignore`, tag | quiz · **project** |
| 5. Volume và network | named volume với Postgres, gọi nhau bằng tên | quiz |
| 6. Docker Compose | portfolio + Umami + Postgres, `.env`, healthcheck, gỡ lỗi | quiz · self_check |
| 7. Quay lại Actions | test stack trong CI, push image lên GHCR, workflow an toàn | quiz · **project** |
| 8. Dự án cuối | repo portfolio với pipeline đầy đủ, README, badge | **project** |

- **Mentor dạy 2 buổi online:** [docs/teaching-guide.md](docs/teaching-guide.md).
- Cài đặt cho người học: [`tracks/actions-docker/docs/setup.md`](tracks/actions-docker/docs/setup.md).
- Mọi code mẫu (Dockerfile, `compose.yaml`, workflow) đã được chạy thử với Docker 29 / Compose v5 và kiểm tra bằng actionlint 1.7.12. Khi nâng phiên bản image hay action, chạy thử lại trước khi sửa bài.

## Cấu trúc

```
tracks/actions-docker/        # đúng MỘT thư mục, tên = slug của track
  track.yaml                 # thông tin track + danh sách chương
  docs/setup.md              # tuỳ chọn: trang "Cài đặt" (cài công cụ, tự kiểm tra bài)
  docs/resources.md          # tuỳ chọn: trang "Tài liệu thêm"
  assets/                    # ảnh (≤ 500 KB/ảnh)
  modules/01-actions-basics/
    module.yaml              # danh sách bài học và bài tập của chương
    lessons/*.md             # bài học: thông tin ở đầu file + Markdown
    exercises/<slug>/
      exercise.yaml          # loại bài, độ khó, đề/đáp án (quiz), checklist, rubric
      statement.md           # đề bài
      mentor-notes.md        # tuỳ chọn: ghi chú cho mentor khi review (công khai, không ghi đáp án)
tools/init-track.mjs         # chạy một lần sau khi tạo repo từ template
```

Mỗi file mẫu có comment giải thích từng trường. Đặc tả đầy đủ: [CONTENT-SPECIFICATION](https://github.com/hcmute-dsc/learning-hub-docs/blob/main/docs/product/CONTENT-SPECIFICATION.md).

## Tạo track mới (cho mentor)

Hướng dẫn đầy đủ, kể cả phần của Ban chủ nhiệm và các lỗi thường gặp: [NEW-TRACK.md](https://github.com/hcmute-dsc/learning-hub-docs/blob/main/docs/ops/NEW-TRACK.md). Dùng agent (Claude Code…) thì gọi skill `new-track`.

1. Mở issue "Đề xuất bài mới hoặc lộ trình mới" trong [learning-hub-content](https://github.com/hcmute-dsc/learning-hub-content/issues/new/choose) để Ban chủ nhiệm biết và tạo repo cho bạn.
2. Ban chủ nhiệm tạo repo `hcmute-dsc/learning-hub-track-<slug>` bằng nút **Use this template** của repo này, cấp quyền cho bạn.
3. Clone repo về, rồi chạy (cần Node 22):

   ```bash
   node tools/init-track.mjs <slug> "<Tên track>" <github-của-bạn>
   ```

   Lệnh này đổi tên thư mục, slug, tên track, người phụ trách và sinh `uid` mới cho mọi bài.
4. Sửa `track.yaml`, viết bài học và bài tập theo các file mẫu. Xóa những bài mẫu không dùng (nhớ xóa cả tên trong `module.yaml`).
5. Kiểm tra trên máy, rồi mở Pull Request theo [CONTRIBUTING.md](CONTRIBUTING.md). CI `validate` phải xanh.
6. **Đưa lên site:** mở PR vào `learning-hub-content`, đổi `commit` của track trong `tracks.yaml` sang commit mới nhất trên `main` của repo này. Ban chủ nhiệm duyệt PR đó rồi publish.

## Kiểm tra trên máy

```bash
git clone https://github.com/hcmute-dsc/learning-hub ../learning-hub
cd ../learning-hub && npx -y pnpm@9.15.4 install --filter @hub/content... && npx -y pnpm@9.15.4 --filter @hub/content build && cd -
node ../learning-hub/packages/content/dist/cli.js .
```
