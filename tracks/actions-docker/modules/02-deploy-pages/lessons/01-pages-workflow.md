---
uid: e1756944-c5cf-4bb6-b588-6e98a16fbbf0
slug: pages-workflow
title: "Deploy bằng actions/deploy-pages"
estimatedMinutes: 35
objectives:
  - "Bật GitHub Pages với nguồn là GitHub Actions"
  - "Viết workflow đóng gói và deploy portfolio mỗi lần push lên main"
  - "Giải thích permissions, environment và needs trong hai job build và deploy"
prerequisites:
  - actions-docker/actions-basics/read-the-log
authors: [github:TienNHM]
---

# Deploy bằng actions/deploy-pages

**GitHub Pages** phục vụ trang web tĩnh (HTML, CSS, JS, ảnh) miễn phí cho repo public. Bài này cho GitHub Actions tự deploy portfolio mỗi khi bạn push lên `main`.

## 1. Bật Pages

Trong repo portfolio: **Settings** → **Pages** → mục **Build and deployment**, chọn **Source: GitHub Actions**.

Địa chỉ trang:

- repo tên `<tên-github>.github.io` → `https://<tên-github>.github.io/`;
- repo tên khác, ví dụ `portfolio` → `https://<tên-github>.github.io/portfolio/`.

Với địa chỉ thứ hai, trang nằm trong thư mục con `/portfolio/`. Dùng link **tương đối** (`about.html`, `images/me.png`), đừng bắt đầu bằng `/`, nếu không ảnh và CSS sẽ không tải được.

## 2. Workflow

Tạo `.github/workflows/pages.yml`. Bản này có hai job: `build` đóng gói trang, `deploy` đưa lên mạng. Bài sau thêm job kiểm tra link.

```yaml
name: Deploy Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

# Quyền tối thiểu cho mọi job. Job nào cần thêm thì tự xin.
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7

      - uses: actions/configure-pages@v6

      - name: Đóng gói trang web
        uses: actions/upload-pages-artifact@v5
        with:
          path: .

  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    # Không cho hai lần deploy chạy chồng lên nhau
    concurrency:
      group: pages
      cancel-in-progress: false
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy lên GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

## 3. Đọc hiểu

| Phần | Ý nghĩa |
| --- | --- |
| `permissions` | Quyền của token `GITHUB_TOKEN` mà GitHub cấp cho job. Cấp workflow chỉ cho đọc code (`contents: read`), job `build` dùng mức này. Job `deploy` ghi `permissions` riêng, **thay hẳn** mức của workflow: `pages: write` (ghi lên Pages) và `id-token: write` (để Pages xác minh lần deploy đến từ workflow này). Job `deploy` không checkout nên không cần `contents: read`. |
| `needs: build` | Job `deploy` **chờ** job `build` xong và chỉ chạy khi `build` xanh. Không có dòng này, hai job chạy song song (mỗi job một runner riêng) và `deploy` đi tìm artifact khi nó chưa có. |
| `concurrency` | Hai lần push liền nhau thì lần sau chờ lần trước deploy xong, không chạy chồng. |
| `environment: github-pages` | Môi trường deploy GitHub tạo sẵn khi bật Pages. Mặc định chỉ nhánh `main` được deploy vào đây. `url` hiện link trang ngay trên lần chạy. |
| `configure-pages` | Đọc cấu hình Pages của repo. |
| `upload-pages-artifact` | Nén thư mục `path` (ở đây là cả repo, `.`) thành một **artifact**. File và thư mục bắt đầu bằng dấu chấm như `.github` không bị đưa lên. |
| `deploy-pages` | Lấy artifact mà job `build` đã upload, đưa lên Pages, trả về `page_url`. Job `deploy` không cần code, chỉ cần artifact, nên không có bước checkout. |

**Vì sao hai job?** Artifact được cất trên GitHub, ngoài runner, nên job `deploy` chạy trên một runner khác vẫn lấy được. Nếu upload rồi deploy ngay trong **cùng một job**, deploy-pages thỉnh thoảng tìm artifact khi nó chưa kịp "hiện ra" và báo `No artifacts named "github-pages" were found for this workflow run` (lỗi đã được báo ở [actions/deploy-pages#451](https://github.com/actions/deploy-pages/issues/451)). Tách hai job thì giữa upload và deploy có vài giây chuyển job, và các mẫu workflow chính thức của GitHub cho Pages cũng làm vậy.

Portfolio để trong thư mục con, ví dụ `site/`? Đổi `path: site`.

Với `path: .`, mọi file không bắt đầu bằng dấu chấm đều lên site, kể cả `Dockerfile` và `compose.yaml` bạn thêm ở các chương sau. Không sao, vì chúng không chứa bí mật; mật khẩu nằm trong `.env`, file này không bao giờ vào repo (chương 6).

## 4. Chạy

Commit và push lên `main`. Trong tab **Actions**, lần chạy **Deploy Pages** có hai job `build` → `deploy`, và link trang ngay dưới tên job `deploy`. Cuối log của step **Deploy lên GitHub Pages** có dòng `Reported success!`. Lần đầu có thể mất 1–2 phút trang mới hiện.

## Khi lỗi

| Lỗi trong log | Cách sửa |
| --- | --- |
| `Get Pages site failed` hoặc `Not Found` ở `configure-pages` | Chưa bật Pages với **Source: GitHub Actions** (bước 1) |
| `Branch "feat/..." is not allowed to deploy to github-pages` | Chỉ `main` được deploy. Merge vào `main` trước. |
| `No artifacts named "github-pages" were found for this workflow run` | Upload và deploy nằm trong cùng một job: tách thành hai job `build` và `deploy` như mục 2 |
| `Multiple artifacts named "github-pages" were unexpectedly found` | Đã bấm **Re-run** một lần chạy có upload và deploy cùng job, nên có thêm artifact trùng tên: tách hai job như mục 2, rồi push hoặc **Run workflow** để có lần chạy mới |
| `Resource not accessible by integration` | Thiếu `pages: write` hoặc `id-token: write` trong `permissions` |
| Trang hiện nhưng không có CSS/ảnh | Link bắt đầu bằng `/`, hoặc sai hoa thường (`Style.css` khác `style.css` trên server) |

## Tóm tắt

- Bật Pages với **Source: GitHub Actions**.
- Hai job nối bằng `needs`: `build` (`configure-pages` → `upload-pages-artifact`) rồi `deploy` (`deploy-pages`).
- Job deploy xin đúng quyền nó cần: `pages: write`, `id-token: write`.
