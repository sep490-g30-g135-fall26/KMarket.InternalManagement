# Kiến trúc hiện tại

Tài liệu ghi nhận cấu trúc đang có trong source và solution. Trách nhiệm của các lớp bên dưới giải thích cách tổ chức bộ khung; không phải bộ quy chuẩn đã được nhóm phê duyệt.

## Tổng quan

Solution có chín project trên .NET 10: một ASP.NET Core API Host và tám class library thuộc hai module `Auth`, `BaseModule`.

Cấu trúc hiện tại hướng đến modular monolith: một Host tham chiếu các module và chạy trong cùng ứng dụng. Chưa có service triển khai độc lập hoặc cơ chế giao tiếp liên module được triển khai.

Tất cả project đang bật nullable reference types và implicit usings. API Host sử dụng package `Microsoft.AspNetCore.OpenApi` phiên bản `10.0.12`.

## API Host

Vị trí: `src/Host/KMarket.InternalManagement.Api/`.

`Program.cs` hiện thực hiện:

1. Tạo application builder.
2. Đăng ký controller và OpenAPI.
3. Tạo ứng dụng.
4. Map tài liệu OpenAPI khi môi trường là Development.
5. Bật HTTPS redirection và authorization middleware.
6. Map controller và chạy ứng dụng.

Host tham chiếu trực tiếp `Infrastructure` và `Presentation` của cả hai module. Chưa có hàm đăng ký dependency injection riêng cho module.

Việc gọi `UseAuthorization()` chưa đồng nghĩa với việc đã triển khai bảo mật. Hiện chưa có cấu hình authentication, policy hoặc endpoint nghiệp vụ được bảo vệ.

## Các lớp trong module

| Lớp | Trách nhiệm dự kiến theo cấu trúc | Trạng thái hiện tại |
| --- | --- | --- |
| Domain | Entity, value object và quy tắc nghiệp vụ | Auth chưa có logic; BaseModule chỉ có placeholder |
| Application | Các luồng xử lý, DTO và hợp đồng cần cho ứng dụng | Auth chưa có logic; BaseModule có thư mục DTOs, Interfaces, Services với placeholder |
| Infrastructure | Hiện thực lưu trữ và tích hợp bên ngoài | Chưa có persistence hoặc integration thực tế |
| Presentation | Tiếp nhận HTTP request và trả response | Chưa có controller thực tế; BaseModule có placeholder cho Controllers và Filters |

### Auth

Vị trí: `src/Modules/Auth/`.

Hiện module mới có bốn file project và các tham chiếu giữa chúng. Tên module thể hiện phạm vi dự kiến về xác thực/quyền truy cập; chưa có đăng nhập, quản lý người dùng hoặc phân quyền được triển khai.

### BaseModule

Vị trí: `src/Modules/BaseModule/`.

```text
BaseModule/
├── BaseModule.Domain/
│   ├── Entities/
│   ├── Exceptions/
│   └── ValueObjects/
├── BaseModule.Application/
│   ├── DTOs/
│   ├── Interfaces/
│   └── Services/
├── BaseModule.Infrastructure/
│   └── Persistence/
│       ├── Configurations/
│       ├── Migrations/
│       └── Repositories/
└── BaseModule.Presentation/
    ├── Controllers/
    └── Filters/
```

Các file `*Placeholder.cs` trong các thư mục trên chỉ chứa comment, chưa khai báo thành phần thực thi. Đặc biệt, thư mục `Migrations` chưa chứa database migration thật.

BaseModule đang được đưa vào solution và được Host tham chiếu. Chưa có quyết định chuyển nó thành template hoặc thư viện dùng chung.

## Tham chiếu project thực tế

Mũi tên dưới đây có nghĩa là “tham chiếu project”, không biểu diễn luồng xử lý request:

```text
API Host ──> Auth.Presentation
         ├─> Auth.Infrastructure
         ├─> BaseModule.Presentation
         └─> BaseModule.Infrastructure

Trong mỗi module:
Presentation   ──> Application
Infrastructure ──> Application
Infrastructure ──> Domain
Application    ──> Domain
Domain         ──> Không tham chiếu project khác
```

Chưa có tham chiếu project trực tiếp giữa Auth và BaseModule. Các quan hệ này được ghi nhận từ `.csproj`; chưa có architecture test tự động kiểm tra chúng.

## Cấu hình và dữ liệu

- Cấu hình ứng dụng hiện nằm tại Host, gồm mức log và `AllowedHosts`.
- Hai profile local `http`, `https` đều đặt môi trường là Development.
- OpenAPI chỉ được map trong Development; chưa có Swagger UI.
- Chưa chọn hoặc tích hợp database trong source hiện tại.
- Chưa có DbContext, connection string, repository thực thi hoặc migration thực tế.

Hướng dẫn khởi chạy và địa chỉ local nằm trong [README](../README.md).

## Thư mục logic trong solution

Solution khai báo `src/BuildingBlocks`, `tests/UnitTests` và `tests/IntegrationTests` dưới dạng solution folder. Hiện chưa có project tương ứng được đăng ký bên trong các mục này.

Các mục đó thể hiện vị trí dự kiến trong solution, chưa thể xem là thư viện dùng chung hoặc bộ kiểm thử đã tồn tại.

## Những nội dung chưa được thống nhất

Các nội dung sau cần cập nhật khi nhóm có quyết định hoặc triển khai thực tế:

- Quy ước viết code, tổ chức feature và đăng ký dependency injection.
- Vai trò lâu dài của BaseModule và phạm vi BuildingBlocks.
- Công nghệ database, quyền sở hữu dữ liệu và cách quản lý migration.
- Cách triển khai xác thực, phân quyền và quản lý cấu hình nhạy cảm.
- Cách giao tiếp giữa các module và định dạng hợp đồng API.
- Công cụ, phạm vi kiểm thử và quy trình CI/CD.
- Quy trình branch, review, phát hành và môi trường triển khai.

Chưa mặc định lựa chọn CQRS, mediator, generic repository, message broker hoặc microservices trong tài liệu này.
