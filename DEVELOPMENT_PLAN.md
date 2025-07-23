# KẾ HOẠCH PHÁT TRIỂN HỆ THỐNG QUẢN LÝ TRUNG TÂM NGOẠI NGỮ ĐA CHI NHÁNH

## THÔNG TIN TỔNG QUAN

### Công nghệ sử dụng:
- **Frontend**: HTML5, CSS3, JavaScript, Tailwind CSS, Font Awesome
- **Backend**: C# Windows Forms Application với WebView2
- **Database**: Microsoft SQL Server
- **Architecture**: Hybrid Desktop Application (Windows Forms + Web UI)

### Phân tích UI hiện có:
Từ các file HTML đã cung cấp, hệ thống có 22 module chính:

1. **Login.html** - Đăng nhập hệ thống
2. **DashBoard.html** - Bảng điều khiển tổng quan
3. **AccountandAccessManagement.html** - Quản lý tài khoản & phân quyền
4. **StudentList.html** - Danh sách học viên
5. **StaffList.html** - Danh sách nhân viên
6. **TeacherEditShiftLearn.html** - Quản lý ca học giáo viên
7. **ClassList.html** - Danh sách lớp học
8. **ClassDescription.html** - Mô tả lớp học
9. **BranchManagement.html** - Quản lý chi nhánh
10. **DepartmentManagement.html** - Quản lý phòng ban
11. **FinancialDashBoard.html** - Bảng điều khiển tài chính
12. **SalaryDescription.html** - Mô tả lương
13. **EditSalaryTable.html** - Chỉnh sửa bảng lương
14. **RequestCreate.html** - Tạo yêu cầu
15. **RequestSummary.html** - Tóm tắt yêu cầu
16. **ReportCreate.html** - Tạo báo cáo
17. **DocumentList.html** - Danh sách tài liệu
18. **Keepingtime.html** - Chấm công
19. **ChangePassWord.html** - Đổi mật khẩu
20. **MajorList.html** - Danh sách chuyên ngành
21. **Log.html** - Nhật ký hệ thống
22. **Tusision.html** - Quản lý học phí

## GIAI ĐOẠN 1: THIẾT KẾ CƠ SỞ DỮ LIỆU (Ngày 1-2)

### 1.1 Thiết kế Database Schema

#### Bảng chính:
```sql
-- Bảng chi nhánh
CREATE TABLE Branches (
    BranchID INT PRIMARY KEY IDENTITY(1,1),
    BranchCode NVARCHAR(10) UNIQUE NOT NULL,
    BranchName NVARCHAR(100) NOT NULL,
    Address NVARCHAR(255),
    Phone NVARCHAR(20),
    Email NVARCHAR(100),
    ManagerID INT,
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng phòng ban
CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY IDENTITY(1,1),
    DepartmentCode NVARCHAR(10) UNIQUE NOT NULL,
    DepartmentName NVARCHAR(100) NOT NULL,
    BranchID INT FOREIGN KEY REFERENCES Branches(BranchID),
    Description NVARCHAR(255),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng chuyên ngành
CREATE TABLE Majors (
    MajorID INT PRIMARY KEY IDENTITY(1,1),
    MajorCode NVARCHAR(10) UNIQUE NOT NULL,
    MajorName NVARCHAR(100) NOT NULL,
    Description NVARCHAR(255),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng nhân viên
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY IDENTITY(1,1),
    EmployeeCode NVARCHAR(20) UNIQUE NOT NULL,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    FullName AS (LastName + ' ' + FirstName),
    Email NVARCHAR(100),
    Phone NVARCHAR(20),
    Address NVARCHAR(255),
    BranchID INT FOREIGN KEY REFERENCES Branches(BranchID),
    DepartmentID INT FOREIGN KEY REFERENCES Departments(DepartmentID),
    Position NVARCHAR(100),
    HireDate DATE,
    Salary DECIMAL(18,2),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng tài khoản người dùng
CREATE TABLE UserAccounts (
    AccountID INT PRIMARY KEY IDENTITY(1,1),
    EmployeeID INT FOREIGN KEY REFERENCES Employees(EmployeeID),
    Username NVARCHAR(50) UNIQUE NOT NULL,
    PasswordHash NVARCHAR(255) NOT NULL,
    Email NVARCHAR(100),
    DisplayName NVARCHAR(100),
    StatusID INT DEFAULT 1,
    FirstLoginChangePwd BIT DEFAULT 1,
    LastLogin DATETIME,
    WrongLoginCount INT DEFAULT 0,
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng vai trò
CREATE TABLE Roles (
    RoleID INT PRIMARY KEY IDENTITY(1,1),
    RoleName NVARCHAR(50) UNIQUE NOT NULL,
    Description NVARCHAR(255),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng quyền hệ thống
CREATE TABLE SystemPermissions (
    PermissionID INT PRIMARY KEY IDENTITY(1,1),
    PermissionCode NVARCHAR(50) UNIQUE NOT NULL,
    PermissionName NVARCHAR(100) NOT NULL,
    Description NVARCHAR(255),
    ModuleName NVARCHAR(50),
    PermissionType NVARCHAR(50),
    IsTCode BIT DEFAULT 0,
    TCodeValue NVARCHAR(50),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng phân quyền vai trò
CREATE TABLE RolePermissions (
    RoleID INT FOREIGN KEY REFERENCES Roles(RoleID),
    PermissionID INT FOREIGN KEY REFERENCES SystemPermissions(PermissionID),
    PRIMARY KEY (RoleID, PermissionID)
);

-- Bảng phân quyền người dùng
CREATE TABLE UserRoles (
    UserID INT FOREIGN KEY REFERENCES UserAccounts(AccountID),
    RoleID INT FOREIGN KEY REFERENCES Roles(RoleID),
    AssignedDate DATETIME DEFAULT GETDATE(),
    PRIMARY KEY (UserID, RoleID)
);

-- Bảng quyền tùy chỉnh người dùng
CREATE TABLE UserCustomPermissions (
    UserID INT FOREIGN KEY REFERENCES UserAccounts(AccountID),
    PermissionID INT FOREIGN KEY REFERENCES SystemPermissions(PermissionID),
    IsGranted BIT NOT NULL, -- TRUE = thêm quyền, FALSE = bỏ quyền
    AssignedDate DATETIME DEFAULT GETDATE(),
    PRIMARY KEY (UserID, PermissionID)
);

-- Bảng học viên
CREATE TABLE Students (
    StudentID INT PRIMARY KEY IDENTITY(1,1),
    StudentCode NVARCHAR(20) UNIQUE NOT NULL,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    FullName AS (LastName + ' ' + FirstName),
    DateOfBirth DATE,
    Gender NVARCHAR(10),
    Email NVARCHAR(100),
    Phone NVARCHAR(20),
    Address NVARCHAR(255),
    BranchID INT FOREIGN KEY REFERENCES Branches(BranchID),
    EnrollmentDate DATE,
    Status NVARCHAR(20) DEFAULT 'Active',
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng khóa học
CREATE TABLE Courses (
    CourseID INT PRIMARY KEY IDENTITY(1,1),
    CourseCode NVARCHAR(20) UNIQUE NOT NULL,
    CourseName NVARCHAR(100) NOT NULL,
    Description NVARCHAR(500),
    MajorID INT FOREIGN KEY REFERENCES Majors(MajorID),
    Duration INT, -- Số giờ học
    Fee DECIMAL(18,2),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng lớp học
CREATE TABLE Classes (
    ClassID INT PRIMARY KEY IDENTITY(1,1),
    ClassCode NVARCHAR(20) UNIQUE NOT NULL,
    ClassName NVARCHAR(100) NOT NULL,
    CourseID INT FOREIGN KEY REFERENCES Courses(CourseID),
    BranchID INT FOREIGN KEY REFERENCES Branches(BranchID),
    TeacherID INT FOREIGN KEY REFERENCES Employees(EmployeeID),
    StartDate DATE,
    EndDate DATE,
    Schedule NVARCHAR(255), -- Lịch học (JSON hoặc text)
    MaxStudents INT DEFAULT 30,
    CurrentStudents INT DEFAULT 0,
    Status NVARCHAR(20) DEFAULT 'Active',
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng đăng ký học
CREATE TABLE StudentEnrollments (
    EnrollmentID INT PRIMARY KEY IDENTITY(1,1),
    StudentID INT FOREIGN KEY REFERENCES Students(StudentID),
    ClassID INT FOREIGN KEY REFERENCES Classes(ClassID),
    EnrollmentDate DATE,
    Fee DECIMAL(18,2),
    Status NVARCHAR(20) DEFAULT 'Active',
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng chấm công
CREATE TABLE Attendance (
    AttendanceID INT PRIMARY KEY IDENTITY(1,1),
    EmployeeID INT FOREIGN KEY REFERENCES Employees(EmployeeID),
    AttendanceDate DATE,
    CheckInTime TIME,
    CheckOutTime TIME,
    WorkingHours DECIMAL(4,2),
    Status NVARCHAR(20), -- Present, Absent, Late, etc.
    Notes NVARCHAR(255),
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng yêu cầu
CREATE TABLE Requests (
    RequestID INT PRIMARY KEY IDENTITY(1,1),
    RequestCode NVARCHAR(20) UNIQUE NOT NULL,
    RequestType NVARCHAR(50) NOT NULL,
    Title NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000),
    RequesterID INT FOREIGN KEY REFERENCES Employees(EmployeeID),
    BranchID INT FOREIGN KEY REFERENCES Branches(BranchID),
    Status NVARCHAR(20) DEFAULT 'Pending',
    Priority NVARCHAR(20) DEFAULT 'Medium',
    CreatedDate DATETIME DEFAULT GETDATE(),
    UpdatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng tài liệu
CREATE TABLE Documents (
    DocumentID INT PRIMARY KEY IDENTITY(1,1),
    DocumentCode NVARCHAR(20) UNIQUE NOT NULL,
    DocumentName NVARCHAR(200) NOT NULL,
    DocumentType NVARCHAR(50),
    FilePath NVARCHAR(500),
    FileSize BIGINT,
    UploadedBy INT FOREIGN KEY REFERENCES Employees(EmployeeID),
    BranchID INT FOREIGN KEY REFERENCES Branches(BranchID),
    IsActive BIT DEFAULT 1,
    CreatedDate DATETIME DEFAULT GETDATE()
);

-- Bảng nhật ký hệ thống
CREATE TABLE SystemLogs (
    LogID INT PRIMARY KEY IDENTITY(1,1),
    UserID INT FOREIGN KEY REFERENCES UserAccounts(AccountID),
    Action NVARCHAR(100) NOT NULL,
    TableName NVARCHAR(50),
    RecordID INT,
    OldValues NVARCHAR(MAX),
    NewValues NVARCHAR(MAX),
    IPAddress NVARCHAR(45),
    UserAgent NVARCHAR(500),
    CreatedDate DATETIME DEFAULT GETDATE()
);
```

### 1.2 Tạo Stored Procedures và Functions

#### Stored Procedures chính:
```sql
-- SP đăng nhập
CREATE PROCEDURE sp_UserLogin
    @Username NVARCHAR(50),
    @Password NVARCHAR(255)
AS
BEGIN
    -- Logic xác thực người dùng
END

-- SP lấy quyền người dùng
CREATE PROCEDURE sp_GetUserPermissions
    @UserID INT
AS
BEGIN
    -- Logic lấy quyền từ vai trò và quyền tùy chỉnh
END

-- SP quản lý học viên
CREATE PROCEDURE sp_ManageStudents
    @Action NVARCHAR(20), -- INSERT, UPDATE, DELETE, SELECT
    @StudentID INT = NULL,
    @StudentData NVARCHAR(MAX) = NULL -- JSON data
AS
BEGIN
    -- Logic CRUD cho học viên
END
```

## GIAI ĐOẠN 2: TẠO ỨNG DỤNG WINDOWS FORMS (Ngày 3-4)

### 2.1 Cấu trúc Project

```
LanguageCenterManagement/
├── LanguageCenterManagement.sln
├── LanguageCenterManagement/
│   ├── Forms/
│   │   ├── MainForm.cs (Form chính chứa WebView2)
│   │   ├── LoginForm.cs (Form đăng nhập nếu cần)
│   │   └── SplashForm.cs (Form loading)
│   ├── Services/
│   │   ├── DatabaseService.cs
│   │   ├── AuthenticationService.cs
│   │   ├── PermissionService.cs
│   │   └── LoggingService.cs
│   ├── Models/
│   │   ├── User.cs
│   │   ├── Student.cs
│   │   ├── Employee.cs
│   │   └── Permission.cs
│   ├── WebContent/
│   │   ├── Login.html
│   │   ├── DashBoard.html
│   │   ├── AccountandAccessManagement.html
│   │   └── [Tất cả các file HTML khác]
│   ├── Utils/
│   │   ├── PasswordHelper.cs
│   │   ├── JsonHelper.cs
│   │   └── ConfigHelper.cs
│   └── App.config
```

### 2.2 MainForm.cs - Form chính

```csharp
using Microsoft.Web.WebView2.Core;
using Microsoft.Web.WebView2.WinForms;
using System;
using System.IO;
using System.Text.Json;
using System.Windows.Forms;

public partial class MainForm : Form
{
    private WebView2 webView;
    private DatabaseService dbService;
    private AuthenticationService authService;
    private string currentUser;

    public MainForm()
    {
        InitializeComponent();
        InitializeWebView();
        InitializeServices();
    }

    private async void InitializeWebView()
    {
        webView = new WebView2()
        {
            Dock = DockStyle.Fill
        };
        
        this.Controls.Add(webView);
        
        // Đảm bảo WebView2 Runtime đã được cài đặt
        await webView.EnsureCoreWebView2Async(null);
        
        // Thiết lập Host Object để JavaScript có thể gọi C#
        webView.CoreWebView2.AddHostObjectToScript("bridge", new WebViewBridge(this));
        
        // Xử lý tin nhắn từ JavaScript
        webView.CoreWebView2.WebMessageReceived += CoreWebView2_WebMessageReceived;
        
        // Tải trang đăng nhập
        LoadLoginPage();
    }

    private void InitializeServices()
    {
        dbService = new DatabaseService();
        authService = new AuthenticationService(dbService);
    }

    private void LoadLoginPage()
    {
        string loginPath = Path.Combine(Application.StartupPath, "WebContent", "Login.html");
        webView.CoreWebView2.Navigate($"file:///{loginPath}");
    }

    private void LoadDashboard()
    {
        string dashboardPath = Path.Combine(Application.StartupPath, "WebContent", "DashBoard.html");
        webView.CoreWebView2.Navigate($"file:///{dashboardPath}");
    }

    private async void CoreWebView2_WebMessageReceived(object sender, CoreWebView2WebMessageReceivedEventArgs e)
    {
        string message = e.TryGetWebMessageAsString();
        
        if (message.StartsWith("TCode:"))
        {
            string tCode = message.Substring(6);
            await HandleTCode(tCode);
        }
        else
        {
            var messageObj = JsonSerializer.Deserialize<dynamic>(message);
            await HandleWebMessage(messageObj);
        }
    }

    private async Task HandleTCode(string tCode)
    {
        // Xử lý T-Code và điều hướng đến trang tương ứng
        string targetPage = GetPageFromTCode(tCode);
        if (!string.IsNullOrEmpty(targetPage))
        {
            string pagePath = Path.Combine(Application.StartupPath, "WebContent", targetPage);
            webView.CoreWebView2.Navigate($"file:///{pagePath}");
        }
    }

    private string GetPageFromTCode(string tCode)
    {
        return tCode switch
        {
            "STUDENT_VIEW" => "StudentList.html",
            "CLASS_SCHEDULE" => "ClassList.html",
            "HR_MANAGEMENT" => "StaffList.html",
            "BRANCH_MANAGEMENT" => "BranchManagement.html",
            "DOCUMENT_MANAGEMENT" => "DocumentList.html",
            _ => null
        };
    }
}

// Host Object để JavaScript gọi C#
[System.Runtime.InteropServices.ComVisible(true)]
public class WebViewBridge
{
    private MainForm mainForm;

    public WebViewBridge(MainForm form)
    {
        mainForm = form;
    }

    public async Task<string> Login(string username, string password)
    {
        // Xử lý đăng nhập
        var result = await mainForm.authService.LoginAsync(username, password);
        return JsonSerializer.Serialize(result);
    }

    public async Task<string> GetUserData()
    {
        // Lấy thông tin người dùng hiện tại
        var userData = await mainForm.dbService.GetUserDataAsync(mainForm.currentUser);
        return JsonSerializer.Serialize(userData);
    }
}
```

### 2.3 DatabaseService.cs

```csharp
using System.Data.SqlClient;
using System.Configuration;

public class DatabaseService
{
    private string connectionString;

    public DatabaseService()
    {
        connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
    }

    public async Task<bool> TestConnectionAsync()
    {
        try
        {
            using (var connection = new SqlConnection(connectionString))
            {
                await connection.OpenAsync();
                return true;
            }
        }
        catch
        {
            return false;
        }
    }

    public async Task<UserLoginResult> ValidateUserAsync(string username, string password)
    {
        using (var connection = new SqlConnection(connectionString))
        {
            await connection.OpenAsync();
            using (var command = new SqlCommand("sp_UserLogin", connection))
            {
                command.CommandType = System.Data.CommandType.StoredProcedure;
                command.Parameters.AddWithValue("@Username", username);
                command.Parameters.AddWithValue("@Password", password);
                
                using (var reader = await command.ExecuteReaderAsync())
                {
                    if (await reader.ReadAsync())
                    {
                        return new UserLoginResult
                        {
                            Success = true,
                            UserId = reader.GetInt32("AccountID"),
                            Username = reader.GetString("Username"),
                            DisplayName = reader.GetString("DisplayName"),
                            Role = reader.GetString("RoleName")
                        };
                    }
                }
            }
        }
        
        return new UserLoginResult { Success = false, Message = "Invalid credentials" };
    }

    // Các method khác cho CRUD operations
    public async Task<List<Student>> GetStudentsAsync(int branchId = 0)
    {
        // Implementation
    }

    public async Task<List<Employee>> GetEmployeesAsync(int branchId = 0)
    {
        // Implementation
    }
}
```

## GIAI ĐOẠN 3: TÍCH HỢP WEB UI VỚI BACKEND (Ngày 5-6)

### 3.1 Cập nhật JavaScript trong các file HTML

Thêm vào mỗi file HTML:

```javascript
// Kiểm tra xem có đang chạy trong WebView2 không
const isInWebView = window.chrome && window.chrome.webview;

// Hàm gọi C# từ JavaScript
async function callCSharpMethod(methodName, parameters) {
    if (isInWebView && window.chrome.webview.hostObjects.bridge) {
        try {
            const result = await window.chrome.webview.hostObjects.bridge[methodName](...parameters);
            return JSON.parse(result);
        } catch (error) {
            console.error('Error calling C# method:', error);
            return { success: false, error: error.message };
        }
    } else {
        console.warn('Not running in WebView2, using mock data');
        return getMockData(methodName, parameters);
    }
}

// Hàm tải dữ liệu từ C#
async function loadDataFromCSharp() {
    showFullScreenLoading();
    
    try {
        const userData = await callCSharpMethod('GetUserData', []);
        const studentsData = await callCSharpMethod('GetStudents', []);
        const employeesData = await callCSharpMethod('GetEmployees', []);
        
        // Cập nhật UI với dữ liệu thực
        updateUIWithData(userData, studentsData, employeesData);
        
    } catch (error) {
        showCustomMessage('Lỗi khi tải dữ liệu: ' + error.message);
    } finally {
        hideFullScreenLoading();
    }
}

// Hàm lưu dữ liệu qua C#
async function saveDataToCSharp(data) {
    showFullScreenLoading();
    
    try {
        const result = await callCSharpMethod('SaveData', [JSON.stringify(data)]);
        if (result.success) {
            showCustomMessage('Lưu dữ liệu thành công!');
        } else {
            showCustomMessage('Lỗi khi lưu: ' + result.message);
        }
    } catch (error) {
        showCustomMessage('Lỗi khi lưu dữ liệu: ' + error.message);
    } finally {
        hideFullScreenLoading();
    }
}
```

### 3.2 Cập nhật các chức năng CRUD

Trong mỗi file HTML, thay thế mock data bằng calls đến C#:

```javascript
// Trong StudentList.html
async function loadStudents() {
    const students = await callCSharpMethod('GetStudents', [currentBranchId]);
    renderStudentsTable(students);
}

async function saveStudent(studentData) {
    const result = await callCSharpMethod('SaveStudent', [studentData]);
    if (result.success) {
        await loadStudents(); // Reload data
        closeModal();
        showCustomMessage('Lưu thông tin học viên thành công!');
    }
}

async function deleteStudent(studentId) {
    const result = await callCSharpMethod('DeleteStudent', [studentId]);
    if (result.success) {
        await loadStudents(); // Reload data
        showCustomMessage('Xóa học viên thành công!');
    }
}
```

## GIAI ĐOẠN 4: BẢO MẬT VÀ PHÂN QUYỀN (Ngày 6-7)

### 4.1 Hệ thống mã hóa mật khẩu

```csharp
public class PasswordHelper
{
    public static string HashPassword(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password);
    }

    public static bool VerifyPassword(string password, string hash)
    {
        return BCrypt.Net.BCrypt.Verify(password, hash);
    }

    public static string GenerateRandomPassword(int length = 12)
    {
        const string chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*";
        var random = new Random();
        return new string(Enumerable.Repeat(chars, length)
            .Select(s => s[random.Next(s.Length)]).ToArray());
    }
}
```

### 4.2 Hệ thống phân quyền

```csharp
public class PermissionService
{
    private DatabaseService dbService;

    public PermissionService(DatabaseService dbService)
    {
        this.dbService = dbService;
    }

    public async Task<List<string>> GetUserPermissionsAsync(int userId)
    {
        // Lấy quyền từ vai trò + quyền tùy chỉnh
        var rolePermissions = await GetRolePermissionsAsync(userId);
        var customPermissions = await GetCustomPermissionsAsync(userId);
        
        // Kết hợp và xử lý quyền
        return CombinePermissions(rolePermissions, customPermissions);
    }

    public async Task<bool> HasPermissionAsync(int userId, string permission)
    {
        var userPermissions = await GetUserPermissionsAsync(userId);
        return userPermissions.Contains(permission);
    }
}
```

## GIAI ĐOẠN 5: TESTING VÀ DEPLOYMENT (Ngày 7)

### 5.1 Unit Testing

```csharp
[TestClass]
public class DatabaseServiceTests
{
    [TestMethod]
    public async Task TestConnection_ShouldReturnTrue()
    {
        var dbService = new DatabaseService();
        var result = await dbService.TestConnectionAsync();
        Assert.IsTrue(result);
    }

    [TestMethod]
    public async Task ValidateUser_WithValidCredentials_ShouldReturnSuccess()
    {
        var dbService = new DatabaseService();
        var result = await dbService.ValidateUserAsync("admin", "password123");
        Assert.IsTrue(result.Success);
    }
}
```

### 5.2 Deployment Package

Tạo installer với:
- .NET Framework 4.8 hoặc .NET 6
- WebView2 Runtime
- SQL Server connection setup
- Initial database scripts

## TIMELINE TỔNG QUAN

| Ngày | Công việc chính | Deliverables |
|------|----------------|--------------|
| 1-2  | Thiết kế & tạo database | Database schema, stored procedures |
| 3-4  | Tạo Windows Forms app | MainForm, Services, Models |
| 5-6  | Tích hợp Web UI với Backend | Hoàn chỉnh tất cả chức năng CRUD |
| 7    | Testing & Deployment | Ứng dụng hoàn chỉnh, installer |

## YÊU CẦU HỆ THỐNG

### Minimum Requirements:
- Windows 10 version 1903 trở lên
- .NET Framework 4.8 hoặc .NET 6
- SQL Server 2016 trở lên
- WebView2 Runtime
- RAM: 4GB
- Disk: 500MB

### Recommended:
- Windows 11
- .NET 6
- SQL Server 2019+
- RAM: 8GB
- SSD storage

## TÍNH NĂNG CHÍNH ĐÃ PHÂN TÍCH

1. **Quản lý đa chi nhánh**: Hỗ trợ nhiều chi nhánh với dữ liệu riêng biệt
2. **Hệ thống phân quyền tinh vi**: Role-based + custom permissions
3. **Giao diện responsive**: Tương thích nhiều kích thước màn hình
4. **T-Code system**: Hệ thống mã lệnh nhanh như SAP
5. **Notification system**: Thông báo real-time
6. **Audit logging**: Ghi log đầy đủ các thao tác
7. **Multi-language support**: Sẵn sàng cho đa ngôn ngữ
8. **Modern UI**: Sử dụng Tailwind CSS, Font Awesome

Kế hoạch này đảm bảo hoàn thành trong 1 tuần với chất lượng cao và khả năng mở rộng tốt.
