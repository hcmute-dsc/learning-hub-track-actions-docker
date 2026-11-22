Chương 1–2 chỉ cần Git, một trình soạn thảo và tài khoản GitHub (bạn đã có từ track Git/GitHub). Từ chương 3 cần **Docker**. Cài một lần, **trước buổi online đầu tiên**: mất khoảng 30–60 phút, lâu nhất là lần đầu trên Windows.

## 1. Kiểm tra máy

- **RAM:** nên có 8 GB trở lên. Máy 4 GB vẫn chạy được nhưng chậm khi chạy đủ portfolio + Umami + Postgres ở chương 6; xem mục 5 nếu máy yếu.
- **Ổ đĩa:** còn trống ít nhất 10 GB.
- **Windows:** Windows 10 bản 22H2 trở lên, hoặc Windows 11, 64-bit.

## 2. Cài Docker Desktop

Docker Desktop miễn phí cho cá nhân, học tập và doanh nghiệp nhỏ.

### Windows

1. **Bật ảo hóa (virtualization).** Mở Task Manager → **Performance** → **CPU**, xem dòng **Virtualization**. Nếu là **Disabled**, khởi động lại máy, vào BIOS/UEFI (thường bấm `F2`, `F10`, `Del` hoặc `Esc` lúc vừa bật máy, tuỳ hãng) và bật mục **Intel Virtualization Technology (VT-x)** hoặc **AMD SVM Mode**. Lưu, khởi động lại.
2. **Cài WSL2.** Mở PowerShell bằng **Run as administrator**:

   ```powershell
   wsl --install
   ```

   Khởi động lại máy khi được hỏi. Đã có WSL rồi thì chạy `wsl --update`.
3. Tải [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/), cài với lựa chọn **Use WSL 2 instead of Hyper-V**.
4. Mở Docker Desktop, chờ tới khi góc dưới báo **Engine running**.

**Dùng terminal nào?** Gõ lệnh Docker trong **PowerShell** hoặc terminal của WSL (Ubuntu). Tránh Git Bash: nó tự đổi các đường dẫn bắt đầu bằng `/` (như `/usr/share/nginx/html`, `/repo`) thành đường dẫn Windows, làm lệnh Docker báo `No such file or directory`. Bắt buộc dùng Git Bash thì đặt `MSYS_NO_PATHCONV=1` ở đầu lệnh.

### macOS

Tải [Docker Desktop for Mac](https://docs.docker.com/desktop/setup/install/mac-install/), chọn đúng bản cho chip của máy:

- **Mac with Apple chip** (M1, M2, M3…): bản Apple Silicon.
- **Mac with Intel chip**: bản Intel.

Mở Docker Desktop từ Applications, chờ biểu tượng cá voi trên thanh menu báo đang chạy.

### Ubuntu

Cài Docker Engine theo [hướng dẫn chính thức](https://docs.docker.com/engine/install/ubuntu/) (gói `docker-ce` và `docker-compose-plugin`), rồi cho tài khoản của bạn dùng Docker mà không cần `sudo`:

```bash
sudo usermod -aG docker $USER
```

Đăng xuất rồi đăng nhập lại để có hiệu lực.

## 3. Tự kiểm tra

```bash
docker --version
docker compose version
docker run hello-world
```

Thấy phiên bản Docker, phiên bản Compose (v2 trở lên) và dòng **`Hello from Docker!`** là xong. Chụp màn hình kết quả `hello-world`, gửi vào kênh Discord của track (bài tập "Cài Docker và chạy hello-world").

## 4. Chip Apple Silicon (arm64)

Mac chip Apple dùng kiến trúc **arm64**, khác với máy Windows và runner GitHub (**amd64**). Mọi image trong track (`nginx`, `postgres`, `alpine`, `umami`) đều có bản arm64, nên chạy bình thường.

Gặp image cũ chỉ có bản amd64, Docker báo `no matching manifest for linux/arm64/v8`. Khi đó thêm `--platform linux/amd64` (với Compose: `platform: linux/amd64` trong service). Image sẽ chạy qua giả lập, chậm hơn nhưng dùng được.

## 5. Máy yếu hoặc không cài được

Không cài được Docker (máy 4 GB RAM, máy trường không cho bật ảo hóa, Windows quá cũ) thì dùng Docker trên trình duyệt:

- **GitHub Codespaces** (khuyên dùng): mở repo portfolio trên GitHub → nút **Code** → tab **Codespaces** → **Create codespace on main**. Terminal trong Codespaces có sẵn `docker` và `docker compose`. Tài khoản cá nhân có một lượng giờ dùng miễn phí mỗi tháng; **dừng codespace khi không dùng** (trang [github.com/codespaces](https://github.com/codespaces)) để khỏi tốn giờ. Cổng như 8080 hiện ở tab **Ports**, bấm vào để mở trang.
- **[Play with Docker](https://labs.play-with-docker.com/)**: đăng nhập bằng tài khoản Docker Hub, mỗi phiên dùng được vài giờ rồi tự xóa. Hợp để thử lệnh ở chương 3–5, không hợp để lưu bài.

Máy chạy được nhưng chậm: tắt các service không dùng (`docker compose down`), và trên Windows có thể giới hạn RAM cho WSL bằng file `%UserProfile%\.wslconfig`:

```ini
[wsl2]
memory=3GB
```

Chạy `wsl --shutdown` rồi mở lại Docker Desktop để áp dụng.

## 6. Khi bị lỗi

| Lỗi | Cách sửa |
| --- | --- |
| `Virtualization support not detected` hoặc Docker Desktop không khởi động | Bật ảo hóa trong BIOS (mục 2, bước 1) |
| `WSL 2 installation is incomplete` hoặc `WSL update failed` | PowerShell admin: `wsl --update`, rồi khởi động lại máy |
| `Cannot connect to the Docker daemon` / `error during connect` | Docker Desktop chưa chạy. Mở nó và chờ **Engine running**. Ubuntu: `sudo systemctl start docker`. |
| `permission denied while trying to connect to the Docker daemon socket` (Ubuntu) | Chưa vào nhóm `docker`: chạy `usermod` ở mục 2 rồi đăng nhập lại |
| `Bind for 0.0.0.0:8080 failed: port is already allocated` | Cổng 8080 đang bị dùng. `docker ps` tìm container đang giữ cổng và `docker rm -f <tên>`. Không phải container thì tìm chương trình đang giữ cổng (xem dưới bảng), hoặc đổi cổng, ví dụ `-p 8081:80`. |
| `No such file or directory` với đường dẫn lạ như `C:/Program Files/Git/usr/...` | Đang dùng Git Bash. Chuyển sang PowerShell, hoặc thêm `MSYS_NO_PATHCONV=1` ở đầu lệnh. |
| `not found` hoặc `illegal option -` khi chạy file `.sh` trong container | File có xuống dòng kiểu Windows (CRLF). Xem mục 7. |

Tìm chương trình đang giữ cổng 8080:

```powershell
netstat -ano | findstr :8080
```

```bash
lsof -i :8080     # macOS, Linux
```

## 7. Xuống dòng CRLF trên Windows

Windows kết thúc dòng bằng hai ký tự `\r\n` (CRLF), Linux chỉ dùng `\n` (LF). Git for Windows thường tự đổi file sang CRLF khi checkout. File `.html`, `.yml`, `.env` không sao, nhưng **script shell** (`.sh`) chép vào container Linux sẽ hỏng với các lỗi ở bảng trên.

Cách chắc chắn nhất: thêm file `.gitattributes` vào repo để Git luôn giữ LF:

```text
* text=auto eol=lf
```

Rồi chuẩn hoá các file đã có:

```bash
git add --renormalize .
git commit -m "Dùng xuống dòng LF"
```

Trong VS Code, góc dưới bên phải hiện `CRLF` hay `LF`; bấm vào để đổi cho từng file.

Vẫn chưa được? Hỏi trong kênh Discord của track, kèm ảnh chụp lỗi và kết quả `docker --version`.
