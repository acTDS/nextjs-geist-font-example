# HƯỚNG DẪN TRIỂN KHAI HỆ THỐNG QUẢN LÝ TRUNG TÂM NGOẠI NGỮ 68

## TỔNG QUAN TRIỂN KHAI

Hệ thống được thiết kế để triển khai trong môi trường Windows với kiến trúc hybrid desktop application sử dụng Windows Forms + WebView2 + SQL Server.

## BƯỚC 1: CHUẨN BỊ MÔI TRƯỜNG

### 1.1 Yêu cầu phần cứng
- **CPU**: Intel Core i3 hoặc AMD Ryzen 3 trở lên
- **RAM**: Tối thiểu 4GB, khuyến nghị 8GB
- **Ổ cứng**: 2GB trống (500MB cho ứng dụng + 1.5GB cho SQL Server)
- **Màn hình**: Độ phân giải tối thiểu 1366x768

### 1.2 Yêu cầu phần mềm
- **OS**: Windows 10 version 1903 trở lên (khuyến nghị Windows 11)
- **.NET Framework**: 4.8 trở lên
- **SQL Server**: 2016 Express trở lên (khuyến nghị 2019)
- **WebView2 Runtime**: Phiên bản mới nhất

### 1.3 Cài đặt SQL Server

#### Tải SQL Server Express:
```
https://www.microsoft.com/en-us/sql-server/sql-server-downloads
```

#### Cấu hình SQL Server:
1. Chọn "Basic" installation
2. Chấp nhận license terms
3. Chọn thư mục cài đặt (mặc định: C:\Program Files\Microsoft SQL Server)
4. Hoàn tất cài đặt

#### Kích hoạt SQL Server Authentication:
1. Mở SQL Server Management Studio (SSMS)
2. Connect to server với Windows Authentication
3. Right-click server name → Properties
4. Security → chọn "SQL Server and Windows Authentication mode"
5. Restart SQL Server service

### 1.4 Cài đặt WebView2 Runtime

#### Tải WebView2 Runtime:
```
https://developer.microsoft.com/en-us/microsoft-edge/webview2/
```

#### Cài đặt:
1. Tải "Evergreen Standalone Installer"
2. Chạy file installer với quyền Administrator
3. Hoàn tất cài đặt

## BƯỚC 2: THIẾT LẬP CƠ SỞ DỮ LIỆU

### 2.1 Tạo Database

#### Mở SQL Server Management Studio:
1. Server name: `localhost` hoặc `.\SQLEXPRESS`
2. Authentication: Windows Authentication
3. Click Connect

#### Chạy script tạo database:
```sql
-- Mở file Database/01_CreateDatabase.sql
-- Chạy toàn bộ script (F5)
```

#### Kiểm tra database đã tạo:
```sql
USE LanguageCenterDB;
SELECT COUNT(*) FROM sys.tables;
-- Kết quả phải > 15 bảng
```

### 2.2 Insert dữ liệu mẫu

#### Chạy script insert data:
```sql
-- Mở file Database/02_InsertSampleData.sql
-- Chạy toàn bộ script (F5)
```

#### Kiểm tra dữ liệu:
```sql
SELECT COUNT(*) FROM UserAccounts; -- Phải có ít nhất 5 records
SELECT COUNT(*) FROM Students;     -- Phải có ít nhất 10 records
SELECT COUNT(*) FROM Employees;    -- Phải có ít nhất 10 records
```

### 2.3 Tạo Stored Procedures

#### Chạy script stored procedures:
```sql
-- Mở file Database/03_StoredProcedures.sql
-- Chạy toàn bộ script (F5)
```

#### Kiểm tra stored procedures:
```sql
SELECT name FROM sys.procedures WHERE name LIKE 'sp_%';
-- Phải có ít nhất 6 stored procedures
```

## BƯỚC 3: BUILD ỨNG DỤNG

### 3.1 Chuẩn bị môi trường phát triển

#### Cài đặt Visual Studio:
- Visual Studio 2019 Community trở lên
- Workload: ".NET desktop development"
- Individual components: ".NET Framework 4.8 targeting pack"

### 3.2 Build ứng dụng

#### Mở solution:
1. Mở Visual Studio
2. File → Open → Project/Solution
3. Chọn file `LanguageCenterManagement.sln`

#### Restore NuGet packages:
1. Right-click Solution → Restore NuGet Packages
2. Đợi quá trình restore hoàn tất

#### Build solution:
1. Build → Build Solution (Ctrl+Shift+B)
2. Kiểm tra Output window không có lỗi
3. Kiểm tra thư mục `bin/Release/` có file exe

### 3.3 Copy HTML files

#### Copy các file HTML vào thư mục WebContent:
```
LanguageCenterManagement/bin/Release/WebContent/
├── Login.html
├── DashBoard.html
├── AccountandAccessManagement.html
├── StudentList.html
├── StaffList.html
├── ClassList.html
├── BranchManagement.html
├── FinancialDashBoard.html
└── [Tất cả các file HTML khác]
```

## BƯỚC 4: CẤU HÌNH ỨNG DỤNG

### 4.1 Cập nhật Connection String

#### Mở file App.config:
```xml
<connectionStrings>
    <!-- Cho Windows Authentication -->
    <add name="DefaultConnection" 
         connectionString="Server=localhost;Database=LanguageCenterDB;Integrated Security=true;TrustServerCertificate=true;" />
    
    <!-- Hoặc cho SQL Authentication -->
    <add name="DefaultConnection" 
         connectionString="Server=localhost;Database=LanguageCenterDB;User Id=sa;Password=YourPassword;TrustServerCertificate=true;" />
</connectionStrings>
```

### 4.2 Cấu hình Email (tùy chọn)

#### Cập nhật SMTP settings:
```xml
<appSettings>
    <add key="SmtpServer" value="smtp.gmail.com" />
    <add key="SmtpPort" value="587" />
    <add key="SmtpUsername" value="your-email@gmail.com" />
    <add key="SmtpPassword" value="your-app-password" />
    <add key="SmtpEnableSSL" value="true" />
</appSettings>
```

## BƯỚC 5: TESTING

### 5.1 Test kết nối Database

#### Chạy ứng dụng:
1. Double-click file `LanguageCenterManagement.exe`
2. Nếu có lỗi kết nối database, kiểm tra:
   - SQL Server đã khởi động
   - Connection string đúng
   - Database đã tồn tại

### 5.2 Test đăng nhập

#### Sử dụng tài khoản admin:
- Username: `admin`
- Password: `Password123!`

#### Kiểm tra các chức năng:
1. Dashboard hiển thị đúng
2. Menu navigation hoạt động
3. T-Code system hoạt động
4. CRUD operations hoạt động

### 5.3 Test phân quyền

#### Test với các tài khoản khác nhau:
- `minh.nguyen` (Giám đốc chi nhánh)
- `hung.le` (Giáo viên)
- `mai.pham` (Kế toán)

#### Kiểm tra:
- Mỗi user chỉ thấy menu/chức năng được phép
- T-Code chỉ hoạt động với user có quyền
- Data filtering theo branch đúng

## BƯỚC 6: DEPLOYMENT

### 6.1 Tạo Installer (tùy chọn)

#### Sử dụng Advanced Installer hoặc Inno Setup:
1. Tạo project installer mới
2. Add files từ thư mục `bin/Release/`
3. Add prerequisites: .NET Framework 4.8, WebView2 Runtime
4. Tạo shortcuts
5. Build installer

### 6.2 Manual Deployment

#### Copy toàn bộ thư mục bin/Release/ đến máy đích:
```
C:\Program Files\LanguageCenterManagement\
├── LanguageCenterManagement.exe
├── LanguageCenterManagement.exe.config
├── Microsoft.Web.WebView2.Core.dll
├── Microsoft.Web.WebView2.WinForms.dll
├── Newtonsoft.Json.dll
├── WebContent/
│   └── [Tất cả file HTML]
└── [Các DLL khác]
```

#### Tạo shortcut:
1. Right-click `LanguageCenterManagement.exe`
2. Send to → Desktop (create shortcut)
3. Rename shortcut thành "Trung Tâm Ngoại Ngữ 68"

## BƯỚC 7: BẢO TRÌ

### 7.1 Backup Database

#### Tạo backup job tự động:
```sql
BACKUP DATABASE LanguageCenterDB 
TO DISK = 'C:\Backups\LanguageCenterDB_backup.bak'
WITH FORMAT, INIT;
```

#### Schedule backup hàng ngày:
1. Mở SQL Server Agent
2. Tạo Job mới
3. Add step chạy backup script
4. Schedule chạy hàng ngày lúc 2:00 AM

### 7.2 Monitoring

#### Kiểm tra log files:
- Application logs: `Logs/Error_YYYYMMDD.log`
- SQL Server logs: SQL Server Management Studio → Management → SQL Server Logs

#### Performance monitoring:
- Task Manager → Performance tab
- SQL Server Activity Monitor

### 7.3 Updates

#### Cập nhật ứng dụng:
1. Build version mới
2. Stop ứng dụng trên tất cả máy client
3. Replace files trong thư mục cài đặt
4. Test trước khi deploy rộng rãi

#### Cập nhật database:
1. Backup database trước khi update
2. Chạy migration scripts
3. Test với dữ liệu thực
4. Rollback nếu có vấn đề

## TROUBLESHOOTING

### Lỗi thường gặp:

#### "Cannot connect to database"
- Kiểm tra SQL Server service đang chạy
- Kiểm tra connection string
- Kiểm tra firewall settings

#### "WebView2 not found"
- Cài đặt WebView2 Runtime
- Kiểm tra Windows version

#### "Access denied"
- Chạy ứng dụng với quyền Administrator
- Kiểm tra user permissions trong database

#### "Page not found"
- Kiểm tra thư mục WebContent có đầy đủ file HTML
- Kiểm tra file paths trong code

## LIÊN HỆ HỖ TRỢ

- **Email**: support@languagecenter68.com
- **Phone**: 024-1234-5678
- **Website**: https://languagecenter68.com

---

**Lưu ý**: Tài liệu này được cập nhật cho phiên bản 1.0.0. Vui lòng kiểm tra phiên bản mới nhất trước khi triển khai.
