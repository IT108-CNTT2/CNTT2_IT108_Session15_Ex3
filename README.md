# BÁO CÁO: HYBRID WORKFLOW LOCAL – CLOUD

## 1. Phân tích vấn đề

- `node_modules` chứa hàng nghìn file thư viện, làm đồng bộ chậm, tốn dung lượng và dễ gây xung đột.
- Chỉ cần lưu **source code** (`src`, `public`, `package.json`, `package-lock.json`...), không cần tải `node_modules` lên Cloud.
- `.gitignore` chỉ áp dụng cho **Git**, **không** tự loại trừ file khỏi OneDrive.

## 2. Thiết kế Hybrid Workflow

- **Local (SSD):** Viết code, chạy chương trình và cài `node_modules` để có tốc độ tốt.
- **Cloud (OneDrive):** Lưu bản sao source code, tài liệu và tài nguyên quan trọng.
- **Git/GitHub:** Quản lý phiên bản code, hỗ trợ khôi phục khi sửa/xóa nhầm.
- **Files On-Demand:** Chọn **Free up space** cho file Cloud ít dùng để giảm dung lượng SSD; file vẫn nằm trên Cloud.

### Sơ đồ thư mục

```text
D:\Startup\SmartFood\             # Local: làm việc trực tiếp
├── src/
├── public/
├── package.json
├── package-lock.json
├── node_modules/               # Chỉ ở Local
└── .env                        # Bí mật, không đưa lên Cloud

OneDrive/Startup/               # Cloud: dữ liệu cần bảo vệ
├── Code/
│   └── SmartFood/              # Bản sao source, KHÔNG có node_modules
├── Docs/                       # Tài liệu, kế hoạch, báo cáo
└── Assets/                     # Ảnh, logo, thiết kế
```

## 3. Loại trừ `node_modules` khi đồng bộ

**Cách làm:** Đặt dự án gốc **ngoài OneDrive**, rồi chỉ sao chép file cần thiết vào thư mục OneDrive bằng lệnh:

```powershell
robocopy "D:\Startup\SmartFood" "$env:OneDrive\Startup\Code\SmartFood" /E /XD node_modules .git dist build /XF .env *.log /R:1 /W:1
```

- `/E`: Sao chép các thư mục con.
- `/XD`: Bỏ qua thư mục `node_modules`, `.git`, `dist`, `build`.
- `/XF`: Bỏ qua file `.env` và file log.
- Tạo **Task Scheduler** chạy lệnh mỗi 5 phút (hoặc cấu hình tác vụ sau khi lưu trong VS Code) để tự động cập nhật bản sao.
- Khi chuyển sang máy khác, chạy `npm ci` để cài lại thư viện từ `package-lock.json`.

## 4. Luồng dữ liệu I/O: Ctrl+S → tích xanh

```text
Ctrl+S trong VS Code
      ↓
Ghi source code xuống SSD Local
      ↓
Tác vụ sao chép tự động chạy (ví dụ: mỗi 5 phút)
      ↓
Source code được cập nhật vào OneDrive/Startup/Code
      ↓
OneDrive phát hiện thay đổi và tải file lên Cloud
      ↓
Đồng bộ hoàn tất → file hiện dấu tích xanh
```

**Lưu ý:** Tích xanh chỉ xuất hiện **sau khi OneDrive đồng bộ xong**, không nhất thiết ngay lúc Ctrl+S. Files On-Demand tiết kiệm **dung lượng máy**, không tiết kiệm dung lượng Cloud. Xóa nhầm file vẫn có thể bị đồng bộ việc xóa, nên cần thêm **Git History/Version History** để khôi phục.

## 5. Kết luận

**Giải pháp:** Chạy code trên SSD, chỉ đồng bộ source cần thiết lên OneDrive, loại `node_modules` và file bí mật khỏi bản sao; dùng Git để quản lý phiên bản. Cách này vừa nhanh, vừa tiết kiệm dung lượng, vừa giảm nguy cơ mất dữ liệu.
