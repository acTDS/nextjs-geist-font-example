# Hệ thống Quản lý Trung tâm Ngoại ngữ 68

## Tổng quan
Phần mềm quản lý trung tâm ngoại ngữ đa chi nhánh sử dụng công nghệ Windows Forms với WebView2 và SQL Server.

## Yêu cầu hệ thống

### Minimum Requirements:
- Windows 10 version 1903 trở lên
- .NET Framework 4.8
- SQL Server 2016 trở lên
- WebView2 Runtime
- RAM: 4GB
- Disk: 500MB

### Recommended:
- Windows 11
- .NET Framework 4.8
- SQL Server 2019+
- RAM: 8GB
- SSD storage

## Cài đặt

### 1. Cài đặt SQL Server
1. Tải và cài đặt SQL Server 2019 Express hoặc cao hơn
2. Đảm bảo SQL Server đang chạy
3. Tạo database bằng script `Database/01_CreateDatabase.sql`
4. Chạy script insert dữ liệu mẫu `Database/02_InsertSampleData.sql`
5. Tạo stored procedures bằng script `Database/03_StoredProcedures.sql`

### 2. Cài đặt WebView2 Runtime
1. Tải WebView2 Runtime từ Microsoft
2. Cài đặt WebView2 Runtime trên máy tính

### 3. Cấu hình ứng dụng
1. Mở file `App.config`
2. Cập nhật connection string trong phần `<connectionStrings>`
3. Thay đổi server name và authentication method nếu cần

### 4. Build và chạy ứng dụng
1. Mở solution trong Visual Studio 2019+
2. Restore NuGet packages
3. Build solution (Ctrl+Shift+B)
4. Chạy ứng dụng (F5)

## Cấu trúc dự án

```
LanguageCenterManagement/
├── Forms/
│   └── MainForm.cs              # Form chính chứa WebView2
├── Models/
│   ├── User.cs                  # Model người dùng
│   └── Student.cs               # Model học viên và các entity khác
├── Services/
│   ├── DatabaseService.cs       # Service kết nối database
│   └── AuthenticationService.cs # Service xác thực
├── Utils/
│   └── PasswordHelper.cs        # Utility mã hóa mật khẩu
├── Properties/
│   └── AssemblyInfo.cs          # Thông tin assembly
├── WebContent/                  # Thư mục chứa file HTML
├── App.config                   # File cấu hình
└── Program.cs                   # Entry point
```

## Tính năng chính

### 1. Quản lý đa chi nhánh
- Hỗ trợ nhiều chi nhánh với dữ liệu riêng biệt
- Phân quyền theo chi nhánh
- Báo cáo tổng hợp và chi tiết

### 2. Hệ thống phân quyền tinh vi
- Role-based permissions
- Custom permissions cho từng user
- T-Code system như SAP
- Module-based access control

### 3. Giao diện hiện đại
- Responsive design với Tailwind CSS
- Font Awesome icons
- Dark/Light theme support
- Mobile-friendly

### 4. Quản lý học viên
- Thông tin chi tiết học viên
- Lịch sử học tập
- Theo dõi tiến độ
- Quản lý thanh toán

### 5. Quản lý nhân sự
- Thông tin nhân viên
- Chấm công
- Quản lý lương
- Phân công công việc

### 6. Quản lý lớp học
- Tạo và quản lý lớp
- Phân công giáo viên
- Lịch học
- Điểm danh

### 7. Báo cáo và thống kê
- Dashboard tổng quan
- Báo cáo tài chính
- Báo cáo học viên
- Báo cáo nhân sự

## Tài khoản mặc định

### Admin
- Username: `admin`
- Password: `Password123!`

### Các tài khoản khác
- Username: `minh.nguyen` - Password: `Password123!` (Giám đốc chi nhánh)
- Username: `lan.tran` - Password: `Password123!` (Trưởng phòng)
- Username: `hung.le` - Password: `Password123!` (Giáo viên)
- Username: `mai.pham` - Password: `Password123!` (Kế toán)

## Cấu hình Database

### Connection String mẫu:

#### Windows Authentication:
```xml
<add name="DefaultConnection" 
     connectionString="Server=localhost;Database=LanguageCenterDB;Integrated Security=true;TrustServerCertificate=true;" 
     providerName="System.Data.SqlClient" />
```

#### SQL Server Authentication:
```xml
<add name="DefaultConnection" 
     connectionString="Server=localhost;Database=LanguageCenterDB;User Id=sa;Password=YourPassword;TrustServerCertificate=true;" 
     providerName="System.Data.SqlClient" />
```

## Troubleshooting

### Lỗi kết nối Database
1. Kiểm tra SQL Server đã khởi động
2. Kiểm tra tên server trong connection string
3. Kiểm tra authentication method
4. Kiểm tra firewall settings

### Lỗi WebView2
1. Cài đặt WebView2 Runtime
2. Kiểm tra Windows version (cần Windows 10 1903+)
3. Kiểm tra .NET Framework 4.8

### Lỗi quyền truy cập
1. Kiểm tra user có trong database
2. Kiểm tra role và permissions
3. Kiểm tra branch access

## Phát triển

### Thêm tính năng mới
1. Tạo model trong `Models/`
2. Thêm stored procedure trong database
3. Cập nhật `DatabaseService.cs`
4. Tạo HTML page trong `WebContent/`
5. Cập nhật `MainForm.cs` để handle T-Code

### Thêm quyền mới
1. Insert vào bảng `SystemPermissions`
2. Gán quyền cho role trong `RolePermissions`
3. Cập nhật JavaScript để check quyền

## Hỗ trợ

Để được hỗ trợ, vui lòng liên hệ:
- Email: support@languagecenter68.com
- Phone: 024-1234-5678

## License

Copyright © 2024 Trung Tâm Ngoại Ngữ 68. All rights reserved.
