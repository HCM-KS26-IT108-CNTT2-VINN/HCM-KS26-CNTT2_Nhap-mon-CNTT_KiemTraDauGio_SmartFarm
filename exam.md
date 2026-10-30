# Bài kiểm tra đầu buổi: Hệ Thống Theo Dõi Trạm Cảm Biến Nông Trại (SmartFarm Monitor)

## **[Chức năng 1] Dựng "căn cứ số" (Terminal):**

- Chỉ dùng **01 dòng lệnh** trong PowerShell để dựng thư mục gốc **smartfarm-system** và 4 thư mục con: **docs, scripts, data, backup** (giống như trong hình).

```
New-Item -ItemType Directory -Path "smartfarm-system" -Force; "docs","scripts","data","backup"
```

- Khi đang ở trong thư mục **data/**, dùng **đường dẫn tương đối** (..) để tạo file cấu hình **.env** ngay thư mục gốc, nội dung giả định là **DB_PASSWORD=demo_123456**

```
cd .\smartfarm-system\data
Set-Content -Path "..\.env" -Value "DB_PASSWORD=demo_123456"
```

- Gán thuộc tính **Hidden** cho file **.env**. **Cấm** dùng đường dẫn tuyệt đối kiểu **C:\....**

## **[Chức năng 2] Robot tự động lọc log (auto_report.ps1):**

- Tạo file **auto_report.ps1** bên trong **scripts/**, chạy ngay tại đó (**.\auto_report.ps1**) và sử dụng **đường dẫn tương đối**.
- Khi **ExecutionPolicy** ngăn chặn, hãy chạy bằng **powershell -ExecutionPolicy Bypass -File .\auto_report.ps1**.

```
cd scripts
New-Item auto_report_ps1 -Itemtype File
```

# Chức năng 2. Robot tự động lọc log — `auto_report.ps1`

## **Sắp xếp dữ liệu**

Bỏ **hai file** do giảng viên đưa vào thư mục **`data`**:

- **`log-he-thong.log`**
- **`quan-ly-thiet-bi.xlsx`**

**Cây thư mục hiện tại:**

```
smartfarm-system/
├── docs/
├── scripts/
├── data/
│   ├── log-he-thong.log
│   └── quan-ly-thiet-bi.xlsx
├── backup/
└── .env
```

## **Viết file script**

**Chuyển** sang thư mục **`scripts`**:

```
cd ..\scripts
```

**Tạo file:**

```
New-Item -ItemType File -Name "auto_report.ps1" -Force
```

Dùng **VS Code** hoặc **Notepad** để mở file, sau đó dán **nguyên văn** đoạn nội dung bên dưới.

```
# ==========================================
# SMARTFARM - AUTO REPORT
# ==========================================

# Lay duong dan tuong doi
$logPath = "..\data\log-he-thong.log"
$reportPath = "..\backup\bao-cao-su-co.txt"

# Kiem tra file log dau vao
if (-not (Test-Path $logPath)) {
    Write-Output "ERROR: Khong tim thay file log."
    exit 1
}

# Tao tieu de bao cao
Write-Output "BAO CAO SU CO HE THONG SMARTFARM" |
    Set-Content -Path $reportPath

Write-Output "Thoi gian tao bao cao: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')" |
    Add-Content -Path $reportPath

Write-Output "==========================================" |
    Add-Content -Path $reportPath

# Loc cac dong chua CRITICAL hoac ERROR
$errors = @(
    Get-Content -Path $logPath |
        Select-String -Pattern "CRITICAL|ERROR"
)

# Ghi noi dung su co vao bao cao
$errors |
    ForEach-Object { $_.Line } >> $reportPath

# Dem so dong su co
$total = $errors.Count

Write-Output "TONG SO SU CO: $total" >> $reportPath

# AUDIT: Da tham dinh script, khong chua lenh xoa hang loat.
```

## **Khởi chạy chương trình**

Chắc chắn rằng Terminal đang đứng trong thư mục **`scripts`**:

```
.\auto_report.ps1
```

Khi PowerShell không cho chạy script, hãy dùng:

```
powershell -ExecutionPolicy Bypass -File .\auto_report.ps1
```

Chạy thành công thì mở file:

```
..\backup\bao-cao-su-co.txt
```

**Kết quả mẫu để tham khảo** (số dòng log và số lượng thực tế còn tùy vào file đầu vào):

```
BAO CAO SU CO HE THONG SMARTFARM
Thoi gian tao bao cao: 2026-10-28 10:00:00
==========================================
2026-10-12 08:15:01 [ERROR] sensor-03 Connection TIMEOUT after 30s
2026-10-12 09:20:00 [CRITICAL] sensor-02 System failure
TONG SO SU CO: 2
```

# Chức năng 3. Rà soát tài nguyên bằng Excel

Vào **`quan-ly-thiet-bi.xlsx`**, mở sheet **`trang-thai-tram`**.

## **Thêm cột để làm sạch mã linh kiện**

**`TRIM`** cắt bỏ khoảng trắng dư ở hai đầu chuỗi và gộp các khoảng trắng bị lặp ở giữa các từ. Nhờ vậy mã linh kiện sẽ dễ trùng khớp với danh mục.

## **Dò tên thiết bị bằng VLOOKUP**

Giả định sheet **`danh-muc-linh-kien`** có:

- **Cột A:** `ma_linh_kien`
- **Cột B:** `ten_thiet_bi`
- **Cột C:** `don_gia`
- **Cột D:** `loai_storage`

Ở ô **`F2`** của sheet **`trang-thai-tram`**, gõ công thức:

```
=VLOOKUP(E2,'danh-muc-linh-kien'!$A$2:$D$100,2,FALSE)
```

Rồi **kéo công thức** xuống đến hết dữ liệu.

**Giải thích từng phần:**

- **`E2`**: mã linh kiện sau khi đã làm sạch.
- **`'danh-muc-linh-kien'!`**: cách gọi sheet có tên chứa dấu gạch ngang.
- **`$A$2:$D$100`**: vùng dò tìm, được khóa lại bằng `$`.
- **`2`**: lấy tên thiết bị nằm ở cột số hai của vùng dò tìm.
- **`FALSE`**: bắt buộc phải khớp chính xác.

**Chú ý:** vùng `$A$2:$D$100` chỉ để minh họa. Nếu danh mục có trên 99 dòng dữ liệu, hãy nới rộng vùng này cho đủ toàn bộ danh mục.

Nếu kết quả vẫn là **`#N/A`**, hãy xem lại mã sau `TRIM`, kiểu dữ liệu của hai bên và xem mã có thật sự nằm trong danh mục hay chưa. `TRIM` không xử lý được hết các loại ký tự trắng đặc biệt.

## **Bước 3: Chốt bảng dữ liệu**

Bảo đảm **công thức** đã được áp dụng cho mọi dòng dữ liệu.

Mở **AutoFilter**: chọn hàng tiêu đề rồi vào **Data → Filter**.

**Đừng** chừa hàng trống giữa vùng dữ liệu và **đừng** dùng **Merge Cells**.

Xem lại hai cột **`ma_linh_kien_sach`** và **`ten_thiet_bi`**, chắc chắn không còn **`#N/A`** ở những mã hợp lệ.

**Lưu lại file Excel** khi làm xong.
