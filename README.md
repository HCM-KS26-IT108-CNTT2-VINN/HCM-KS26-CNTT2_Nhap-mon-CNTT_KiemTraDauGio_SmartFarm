# SmartFarm System

**Giải pháp theo dõi nhật ký máy chủ và rà soát tài nguyên thiết bị** dành cho công ty khởi nghiệp nông nghiệp công nghệ cao **SmartFarm**.

## 1. Cách vận hành script

### **Bước 1: Sắp xếp dữ liệu**

- Copy file **`log-he-thong.log`** vào trong thư mục **`data/`**.

### **Bước 2: Khởi chạy PowerShell Script**

- Mở cửa sổ PowerShell tại thư mục **`scripts/`** và nhập lệnh:

```
.\auto_report.ps1
```

- Trường hợp **ExecutionPolicy** không cho chạy, dùng lệnh thay thế:

```
powershell -ExecutionPolicy Bypass -File .\auto_report.ps1
```

### **Bước 3: Đọc báo cáo**

- Vào file **`backup/bao-cao-su-co.txt`** để đọc các dòng chứa **`CRITICAL`**, **`ERROR`** và **số lượng sự cố** tổng cộng.

## 2. Danh sách việc cần rà soát

- **Xác nhận** file nhật ký của hệ thống đã có mặt.
- **Xem xét** tình trạng liên lạc của từng trạm cảm biến.
- **So khớp** mã linh kiện với bảng danh mục thiết bị.
- **Rà lại** dung lượng RAM cùng kiểu ổ lưu trữ của từng thiết bị.
- **Đọc lại** báo cáo sự cố khi script đã chạy xong.

## 3. Thông số phần cứng

Dữ liệu trong bảng lấy từ sheet **`danh-muc-linh-kien`** của file **`quan-ly-thiet-bi.xlsx`**.

| **Tên thiết bị** | **Đơn giá** | **Loại storage** |
|---|---:|---|
| Ten thiet bi 1 | 1500000 | SSD |
| Ten thiet bi 2 | 2000000 | HDD |
| Ten thiet bi 3 | 2500000 | SSD |

## 4. Phân tích hệ thống

### 4.1. Mô hình IPO

**Input (Đầu vào)**

- File **`log-he-thong.log`**.
- Những dòng nhật ký thuộc mức **`INFO`**, **`WARNING`**, **`ERROR`** hoặc **`CRITICAL`**.

**Process (Xử lý)**

- Đọc file log bằng **`Get-Content`**.
- Nối **Pipeline (`|`)** với **`Select-String`** để giữ lại những dòng chứa **`ERROR`** hoặc **`CRITICAL`**.
- **Tính** tổng số dòng sự cố.
- **Lưu** thời điểm lập báo cáo, chi tiết sự cố và tổng số sự cố vào file báo cáo nhờ toán tử **`>>`**.

**Output (Đầu ra)**

- File **`backup/bao-cao-su-co.txt`**.
- Nội dung báo cáo có **thời điểm lập**, **các sự cố** và **tổng số sự cố** tìm thấy.

### 4.2. Mô hình DIKW

| **Tầng DIKW** | **Phát biểu** | **Giải thích** |
|---|---|---|
| **Data (Dữ liệu)** | (b) 2026-10-12 08:15:01 [ERROR] sensor-03 Connection TIMEOUT after 30s | Dữ liệu nguyên bản, lấy thẳng từ nhật ký của hệ thống. |
| **Information (Thông tin)** | (c) Trong hôm nay có 8 sự cố được ghi lại, 4 sự cố trong số đó thuộc về trạm sensor-03. | Dữ liệu sau khi thống kê trở thành thông tin có ý nghĩa. |
| **Knowledge (Tri thức)** | (d) Trạm sensor-03 thường rớt kết nối vào buổi chiều vì tín hiệu yếu, vì vậy nên gắn thêm anten. | Phân tích dữ liệu để thấy được quy luật và lý do có thể có. |
| **Wisdom (Trí tuệ)** | (a) Quyết định đổi modem của trạm sensor-03 trước thứ Sáu. | Biến tri thức thành quyết định hợp lý. |

## 5. Bẫy phần cứng: Thời gian tải file log

**Đề bài:** Một file log nặng **500 MB** tải qua đường truyền **100 Mbps** hết bao nhiêu thời gian?

### **Bước 1: Chuyển đổi đơn vị**

Do **1 byte = 8 bit** nên:

**100 Mbps / 8 = 12,5 MB/s**

### **Bước 2: Tính thời gian**

**Thời gian = Dung lượng file / Tốc độ tải**

**500 MB / 12,5 MB/s = 40 giây**

**Kết luận:** Về lý thuyết, việc tải mất **40 giây**, với điều kiện đường truyền chạy hết tốc độ liên tục và không thất thoát. Ngoài đời thực thường chậm hơn vì giao thức mạng, độ trễ và nhiều yếu tố khác.

## 6. Cây thư mục của dự án

```
smartfarm-system/
├── docs/
│   └── README.md
├── scripts/
│   └── auto_report.ps1
├── data/
│   ├── log-he-thong.log
│   └── quan-ly-thiet-bi.xlsx
├── backup/
│   └── bao-cao-su-co.txt
└── .env
```

## 7. Tổng kết

Hệ thống **SmartFarm** nhờ **PowerShell** để lọc nhật ký sự cố một cách tự động, nhờ **Excel** để rà soát danh mục thiết bị và nhờ **Markdown** để ghi chép quy trình. Việc phối hợp **IPO** với **DIKW** cho thấy rõ sự khác nhau giữa dữ liệu thô, thông tin đã tổng hợp, tri thức rút ra từ phân tích và quyết định quản lý.
