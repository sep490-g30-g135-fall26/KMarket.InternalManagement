# KMarket.InternalManagement

Codebase backend cho hệ thống quản lý nội bộ KMarket, sử dụng ASP.NET Core trên .NET 10. Project hiện ở giai đoạn dựng khung: có API Host và hai module `Auth`, `BaseModule`, chưa triển khai chức năng nghiệp vụ.

Tài liệu này mô tả trạng thái hiện tại. Quy trình làm việc và các quy ước phát triển của nhóm chưa được thống nhất.

## Trạng thái hiện tại

- API Host đã cấu hình controller, OpenAPI trong môi trường Development, HTTPS redirection và authorization middleware.
- `Auth` có bốn project theo lớp, chưa có logic xác thực hoặc phân quyền.
- `BaseModule` có các thư mục mẫu; các file `*Placeholder.cs` chỉ chứa comment.
- Chưa có controller nghiệp vụ, DbContext hoặc migration thực tế. Đã có Docker Compose để tạo PostgreSQL local; API chưa tích hợp database này.
- Chưa có project test. Các mục `tests`, `UnitTests`, `IntegrationTests` trong solution hiện chỉ là thư mục logic.

## Yêu cầu môi trường

- .NET 10 SDK để restore, build và chạy project (`TargetFramework` là `net10.0`).
- Có thể dùng terminal và editor tùy chọn. Nếu dùng IDE, cần phiên bản hỗ trợ .NET 10 và solution `.slnx`.
- Truy cập được nguồn NuGet để tải package khi chưa có trong cache.
- Chưa cần database để chạy API ở trạng thái hiện tại. Để chạy PostgreSQL local theo hướng dẫn bên dưới, cần Docker Desktop với Linux containers và Docker Compose.

Kiểm tra SDK đã cài:

```powershell
dotnet --list-sdks
```

Project chưa có `global.json` để cố định phiên bản SDK.

## Cách chạy local

Mở terminal tại thư mục chứa `KMarket.InternalManagement.slnx`.

### 1. Restore và build

```powershell
dotnet restore KMarket.InternalManagement.slnx
dotnet build KMarket.InternalManagement.slnx --no-restore
```

### 2. Chạy API bằng profile HTTPS

Trên máy phát triển Windows, thiết lập chứng chỉ HTTPS nếu chưa có. Lệnh trust có thể hiển thị hộp thoại xác nhận của hệ điều hành:

```powershell
dotnet dev-certs https --trust
```

Chạy ứng dụng:

```powershell
dotnet run --project src/Host/KMarket.InternalManagement.Api/KMarket.InternalManagement.Api.csproj --launch-profile https
```

Theo `launchSettings.json`, profile này chạy ở môi trường `Development` và lắng nghe:

- HTTPS: `https://localhost:7066`
- HTTP: `http://localhost:5063`

HTTP có thể được chuyển hướng sang HTTPS bởi middleware hiện tại. Trình duyệt không tự mở.

### 3. Kiểm tra ứng dụng

Mở tài liệu OpenAPI JSON tại:

```text
https://localhost:7066/openapi/v1.json
```

Host hiện chưa cấu hình giao diện Swagger UI. Chưa có endpoint nghiệp vụ nên phần đường dẫn API trong tài liệu có thể rỗng; truy cập `/` trả về 404 là phù hợp với bộ khung hiện tại.

Nhấn `Ctrl+C` trong terminal để dừng ứng dụng.

Profile HTTP cũng có sẵn:

```powershell
dotnet run --project src/Host/KMarket.InternalManagement.Api/KMarket.InternalManagement.Api.csproj --launch-profile http
```

Khi chỉ chạy profile HTTP, ứng dụng lắng nghe ở `http://localhost:5063`. Do HTTPS redirection vẫn được bật, có thể xuất hiện cảnh báo không xác định được cổng HTTPS. Dùng profile HTTPS cho hướng dẫn chạy mặc định ở trên.

## Cấu trúc chính

```text
KMarket.InternalManagement/
├── KMarket.InternalManagement.slnx
├── README.md
├── docker-compose.yml
├── .env.example
├── docs/
│   └── architecture.md
└── src/
    ├── Host/
    │   └── KMarket.InternalManagement.Api/
    └── Modules/
        ├── Auth/
        │   ├── Auth.Domain/
        │   ├── Auth.Application/
        │   ├── Auth.Infrastructure/
        │   └── Auth.Presentation/
        └── BaseModule/
            ├── BaseModule.Domain/
            ├── BaseModule.Application/
            ├── BaseModule.Infrastructure/
            └── BaseModule.Presentation/
```

Xem [mô tả kiến trúc](docs/architecture.md) để biết trách nhiệm dự kiến của các lớp và tham chiếu project thực tế.

## Cấu hình hiện tại

| File | Nội dung |
| --- | --- |
| `docker-compose.yml` | PostgreSQL 17 local, cổng, volume dữ liệu và health check |
| `.env.example` | Mẫu biến mật khẩu để mỗi thành viên tạo `.env` riêng |
| `src/Host/KMarket.InternalManagement.Api/appsettings.json` | Mức log mặc định và `AllowedHosts` |
| `src/Host/KMarket.InternalManagement.Api/appsettings.Development.json` | Mức log cho môi trường Development |
| `src/Host/KMarket.InternalManagement.Api/Properties/launchSettings.json` | Profile chạy local, cổng và biến `ASPNETCORE_ENVIRONMENT` |

Chưa có connection string hoặc cấu hình nhà cung cấp danh tính. Không cần bổ sung các giá trị này để chạy trạng thái hiện tại.

## Kiểm thử

Hiện chưa có automated test để chạy. Build thành công chỉ xác nhận các project biên dịch được, chưa xác nhận hành vi nghiệp vụ hoặc bảo mật.

## Hướng dẫn Docker Compose

Hướng dẫn chạy PostgreSQL local bằng Docker Compose. Các lệnh dưới đây dùng PowerShell trên Windows.

### 1. Yêu cầu

Cài:

- Docker Desktop (chạy Linux containers)
- .NET 10 SDK
- Git

Mở Docker Desktop và chờ Docker Engine chạy. Kiểm tra:

```powershell
docker --version
docker compose version
```

---

### 2. Clone project

Thay `<repository-url>` bằng URL repository của nhóm:

```powershell
git clone <repository-url> KMarket.InternalManagement
cd KMarket.InternalManagement
```

Các lệnh bên dưới chạy tại thư mục root có file `docker-compose.yml`.

---

### 3. Tạo file `.env`

Project có sẵn `.env.example`. Lần đầu, nếu chưa có `.env`, chạy trong Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

Sau đó mở `.env` và đặt password riêng cho máy bạn, ví dụ:

```dotenv
POSTGRES_PASSWORD=YourLocalPassword123!
```

> `.env` chỉ dùng trên máy cá nhân và không được commit lên Git. Không sao chép đè nếu đã có file này.

---

### 4. Chạy PostgreSQL

Tại thư mục root của project:

```powershell
docker compose up -d
```

Kiểm tra:

```powershell
docker compose ps
```

Lần đầu Docker sẽ tải image và khởi tạo database. Chờ trạng thái gần giống:

```text
kmarket-postgres   Up (healthy)
```

---

### 5. Thông tin database local

```text
Host: localhost
Port: 5432
Database: kmarket_internal_management
Username: kmarket
Password: giá trị POSTGRES_PASSWORD trong file .env
```

Ví dụ connection string, thay password bằng giá trị trên máy bạn:

```text
Host=localhost;Port=5432;Database=kmarket_internal_management;Username=kmarket;Password=YourLocalPassword123!
```

> Mỗi thành viên có database riêng trên máy mình. Đổi password trong `.env` không tự đổi password của database đã khởi tạo.

---

### 6. Chạy Backend

PostgreSQL chạy bằng Docker. Backend ASP.NET Core chạy trực tiếp bằng Visual Studio hoặc terminal.

Nếu chưa thiết lập chứng chỉ HTTPS local, chạy một lần:

```powershell
dotnet dev-certs https --trust
```

Chạy backend từ thư mục root:

```powershell
dotnet run --project src/Host/KMarket.InternalManagement.Api --launch-profile https
```

Kiểm tra OpenAPI tại `https://localhost:7066/openapi/v1.json`. Nhấn `Ctrl+C` trong terminal để dừng backend.

Mô hình kết nối local dự kiến:

```text
ASP.NET Core (chạy trên máy)
     |
     | localhost:5432
     v
PostgreSQL (chạy trong Docker)
```

> Backend hiện chưa tích hợp database. Connection string trên là mẫu để cấu hình khi triển khai phần dữ liệu; ASP.NET Core không tự đọc `.env` của Compose.

---

### 7. Xem PostgreSQL đang chạy

```powershell
docker compose ps
```

Xem log:

```powershell
docker compose logs postgres
```

---

### 8. Dừng PostgreSQL

```powershell
docker compose down
```

Lệnh này không xóa dữ liệu trong named volume của database.

Khi cần chạy lại, dùng cùng thư mục project:

```powershell
docker compose up -d
```

---

### 9. Reset database local

Chỉ sử dụng khi muốn xóa toàn bộ database local:

```powershell
docker compose down -v
```

> ⚠️ Lệnh này xóa volume PostgreSQL và toàn bộ dữ liệu local của Compose project. Chỉ chạy khi dữ liệu có thể bỏ đi hoặc đã được sao lưu.

Sau đó tạo lại:

```powershell
docker compose up -d
```

---

### 10. Các file liên quan

```text
docker-compose.yml
.env.example
.env
```

Quy tắc Git:

```text
docker-compose.yml   ✅ Commit
.env.example         ✅ Commit
.env                 ❌ Không commit
```

`.gitignore` đã có:

```gitignore
.env
.env.*
!.env.example
```

---

### Quy trình hằng ngày

Lần đầu:

```text
Clone project
   ↓
Copy .env.example → .env và đặt password
   ↓
Mở Docker Desktop
   ↓
Chạy docker compose up -d
   ↓
Thiết lập chứng chỉ HTTPS local nếu chưa có
   ↓
Chạy backend bằng lệnh dotnet run --project ở trên
```

Các lần sau, mở Docker Desktop rồi chạy:

```powershell
docker compose up -d
dotnet run --project src/Host/KMarket.InternalManagement.Api --launch-profile https
```

Khi kết thúc, nhấn `Ctrl+C` để dừng backend, sau đó:

```powershell
docker compose down
```
