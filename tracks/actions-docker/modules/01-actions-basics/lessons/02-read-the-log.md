---
uid: 94d513b8-24fc-4143-8773-752fad0e5777
slug: read-the-log
title: "Khi workflow bị đỏ: đọc log"
estimatedMinutes: 25
objectives:
  - "Tìm đúng step bị lỗi và dòng báo lỗi trong log"
  - "Sửa lỗi rồi chạy lại workflow"
  - "Biết chỗ học tiếp về secrets và lịch chạy"
prerequisites:
  - actions-docker/actions-basics/first-workflow
authors: [github:TienNHM]
---

# Khi workflow bị đỏ: đọc log

Workflow sẽ có lúc đỏ, chuyện bình thường. Kỹ năng quan trọng nhất là **đọc log** để biết vì sao.

## Cố tình làm hỏng

Sửa step cuối trong `hello.yml` thành một lệnh sai:

```yaml
      - name: Xem các file trong repo
        run: ls thu-muc-khong-co
```

Commit và push. Trong tab **Actions**, lần chạy mới có dấu ✗ đỏ.

## Đọc log theo 3 bước

1. **Mở lần chạy đỏ**, chọn job có dấu ✗.
2. **Tìm step đỏ đầu tiên.** Các step sau nó bị bỏ qua (màu xám), đừng đọc chúng.
3. **Đọc từ dưới lên** trong step đó. Dòng cuối thường là `Error: Process completed with exit code 2.`; dòng ngay trên mới là nguyên nhân:

```text
ls: cannot access 'thu-muc-khong-co': No such file or directory
Error: Process completed with exit code 2.
```

**Exit code** khác 0 nghĩa là lệnh thất bại, và GitHub dừng job ngay. Mọi lệnh trong `run` đều theo luật này: `curl -f` gặp lỗi HTTP, `grep -q` không tìm thấy chữ, test không qua… đều làm job đỏ. Chính nhờ vậy mà CI chặn được lỗi trước khi nó lên site.

## Những lỗi hay gặp

| Bạn thấy | Nguyên nhân thường gặp |
| --- | --- |
| Không có lần chạy nào trong tab Actions | File không nằm đúng `.github/workflows/`, hoặc đuôi không phải `.yml`/`.yaml` |
| `Invalid workflow file` trên trang lần chạy | Lỗi YAML: thụt lề sai, dùng Tab, thiếu dấu `:` |
| `No such file or directory` | Quên step `actions/checkout`, hoặc sai đường dẫn |
| `command not found` | Lệnh không có trên runner, hoặc gõ sai tên |
| `exit code 1` mà không rõ lý do | Thêm `set -x` ở đầu `run` để in từng lệnh trước khi chạy (đừng làm vậy ở step có secret) |

## Sửa và chạy lại

Sửa lại thành `run: ls -la`, commit, push: một lần chạy mới bắt đầu. Nếu lỗi không do code (mạng chập chờn khi tải gì đó), bấm **Re-run jobs** → **Re-run failed jobs** ở góc trên trang lần chạy, không cần commit.

Muốn bắt lỗi YAML trên máy trước khi push thì dùng [actionlint](https://github.com/rhysd/actionlint). Khi đã cài Docker (chương 3), chạy được mà không cần cài thêm gì. Trong thư mục repo:

```bash
docker run --rm -v "$(pwd):/repo" -w /repo rhysd/actionlint:1.7.12 -no-color
```

Không in gì nghĩa là không thấy lỗi. Trên Windows, chạy lệnh Docker trong **PowerShell** và đổi `$(pwd)` thành `${PWD}`; Git Bash tự đổi các đường dẫn như `/repo` nên lệnh sẽ hỏng (xem tab **Cài đặt**).

## Học thêm: secrets và lịch chạy

Track này chỉ dùng phần Actions cần cho portfolio. Hai chủ đề hay dùng khác đã có ở track [Tự động hoá với Discord](https://learn.hcmute-dsc.io.vn/tracks/discord-webhook-automation):

- **Chương 5 "Giữ bí mật":** lưu giá trị bí mật trong **GitHub Secrets** và đọc nó trong workflow.
- **Chương 6 "Tự động theo lịch":** chạy workflow theo giờ hẹn bằng `schedule` và cron, báo khi workflow lỗi.

## Tóm tắt

- Mở job đỏ, tìm step đỏ đầu tiên, đọc log từ dưới lên.
- Exit code khác 0 làm job dừng và đỏ.
- Lỗi tạm thời thì **Re-run failed jobs**; lỗi trong code thì sửa rồi push.
