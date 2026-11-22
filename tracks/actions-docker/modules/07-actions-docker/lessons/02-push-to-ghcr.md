---
uid: 4056815d-2b05-4036-8586-3bd48a43f16b
slug: push-to-ghcr
title: "Build và push image lên GHCR"
estimatedMinutes: 40
objectives:
  - "Đăng nhập GitHub Container Registry bằng GITHUB_TOKEN với permissions: packages: write"
  - "Build và push image bằng docker/build-push-action, gắn tag tự động bằng docker/metadata-action"
  - "Kéo image từ GHCR về chạy trên máy"
prerequisites:
  - actions-docker/actions-docker/test-compose-in-ci
authors: [github:TienNHM]
---

# Build và push image lên GHCR

Image portfolio đang chỉ nằm trên máy bạn. **GitHub Container Registry** (GHCR, địa chỉ `ghcr.io`) là registry của GitHub, gắn với tài khoản và repo của bạn. Đẩy image lên đó thì bất kỳ máy nào cũng `docker pull` về chạy được, không cần build lại.

Bài này thêm job thứ hai vào `docker.yml`: **sau khi** job `test` xanh, build image và push lên GHCR.

## Workflow hoàn chỉnh

```yaml
name: Docker

on:
  push:
    branches: [main]
    tags: ["v*"]
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

  push-image:
    needs: test
    # Pull request (nhất là từ fork) không được push image
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v7

      - uses: docker/setup-buildx-action@v4

      - name: Đăng nhập GHCR
        uses: docker/login-action@v4
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Tính tag và label cho image
        id: meta
        uses: docker/metadata-action@v6
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha

      - name: Build và push
        uses: docker/build-push-action@v7
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## Đọc hiểu

**Đăng nhập bằng `GITHUB_TOKEN`.** Mỗi lần chạy workflow, GitHub tự tạo một token tạm tên `GITHUB_TOKEN`, hết hạn khi job xong. Bạn **không** cần tạo Personal Access Token hay thêm secret nào. Token chỉ có các quyền ghi trong `permissions`; job `push-image` xin thêm `packages: write` để được ghi lên GHCR. Job `test` không cần quyền đó nên không xin.

**`needs: test` và `if: github.event_name == 'push'`.** Chỉ push image khi stack đã qua kiểm tra, và chỉ khi có push lên `main` hoặc tag `v*`. Pull request chỉ chạy `test`. Bài sau giải thích vì sao PR không được push.

**`docker/metadata-action`** tính tên và tag cho image:

| Khi | Tag sinh ra |
| --- | --- |
| Push lên `main` | `main` |
| Mọi lần push | `sha-1a2b3c4` (7 ký tự đầu của commit) |
| Push tag Git `v1.2.0` | `1.2.0` |

Tên image là `ghcr.io/<chủ-repo>/<tên-repo>`, ví dụ `ghcr.io/an/portfolio`. GHCR chỉ nhận chữ thường; action tự đổi tên repo sang chữ thường. Nó cũng gắn label `org.opencontainers.image.source` trỏ về repo, nhờ đó GHCR nối package với repo của bạn.

**`docker/build-push-action`** build từ `Dockerfile` (giống `docker build`) rồi push. `setup-buildx-action` chuẩn bị bộ build cần cho tuỳ chọn cache. `cache-from`/`cache-to: type=gha` lưu layer vào cache của GitHub Actions, nên lần build sau nhanh hơn, đúng cái cache bạn học ở chương 4.

**Phiên bản action.** Mọi action đều ghi phiên bản (`@v4`, `@v7`). Bài sau nói thêm về cách ghim chặt hơn.

## Phát hành một phiên bản

```bash
git tag v1.0.0
git push origin v1.0.0
```

Workflow chạy với event push tag, sinh image tag `1.0.0` bên cạnh `sha-…`.

## Xem và kéo image về

Sau lần chạy xanh trên `main`, trang repo có mục **Packages** ở cột bên phải. Package được push bằng `GITHUB_TOKEN` từ workflow của repo được nối với repo đó và lấy theo quyền xem của repo: repo public thì package cũng public, ai cũng kéo về được. Nếu `docker pull` khi chưa đăng nhập báo `denied` hoặc `unauthorized`, mở package → **Package settings** → **Change visibility** → **Public**.

Thử trên máy, đổi `an/portfolio` thành repo của bạn (chữ thường):

```bash
docker pull ghcr.io/an/portfolio:main
docker run -d --name from-ghcr -p 8080:80 ghcr.io/an/portfolio:main
```

Package private (ví dụ của repo private) thì cần đăng nhập trước bằng `docker login ghcr.io` với một Personal Access Token có quyền `read:packages`.

## Khi lỗi

| Lỗi | Cách sửa |
| --- | --- |
| Bước push báo lỗi có chữ `denied` (ví dụ `permission_denied: write_package`) | Thiếu `packages: write` trong `permissions` của job. Repo trong một organization thì kiểm tra thêm quyền package của organization. |
| `repository name must be lowercase` | Tự ghép tên image có chữ hoa. Dùng `metadata-action` như trên. |
| Job `push-image` bị bỏ qua (màu xám) | Đúng ý đồ khi chạy từ pull request, hoặc job `test` đỏ |

## Tóm tắt

- `GITHUB_TOKEN` + `permissions: packages: write` là đủ để push lên GHCR, không cần secret riêng.
- `metadata-action` sinh tag `main`, `sha-…`, phiên bản; `build-push-action` build và push.
- Chỉ push sau khi `test` xanh và chỉ với event `push`.
