# Admin Sitemap + Wireframe

Tài liệu này mô tả cấu trúc trang Admin cho hệ thống `Task Tracking Elite`, bám theo 7 nhóm chức năng:

1. Dashboard
2. Phòng ban
3. Người dùng
4. Vai trò & quyền
5. Workflow / State
6. Cấu hình hệ thống
7. Nhật ký hệ thống

## 1. Sitemap

```text
Admin
|
+-- 1. Dashboard
|   +-- Tổng quan hệ thống
|   +-- Thống kê nhanh
|   +-- Cảnh báo cấu hình
|   +-- Hoạt động gần đây
|
+-- 2. Phòng ban
|   +-- Danh sách phòng ban
|   +-- Tạo phòng ban mới
|   +-- Chi tiết phòng ban
|   +-- Cấu trúc cha - con
|   +-- Gán trưởng nhóm / thành viên
|
+-- 3. Người dùng
|   +-- Danh sách user
|   +-- Tạo user mới
|   +-- Chi tiết user
|   +-- Gán phòng ban
|   +-- Gán chức danh
|   +-- Trạng thái hoạt động
|
+-- 4. Vai trò & quyền
|   +-- Danh sách role
|   +-- Danh sách permission
|   +-- Ma trận role - permission
|   +-- Gán role cho user theo team
|   +-- Phạm vi quyền toàn hệ thống / theo team
|
+-- 5. Workflow / State
|   +-- Danh sách state
|   +-- Tạo state mới
|   +-- Sắp xếp thứ tự state
|   +-- Cấu hình chuyển trạng thái
|   +-- Màu sắc / nhãn hiển thị
|   +-- State mặc định / state đóng / state hủy
|
+-- 6. Cấu hình hệ thống
|   +-- Thông tin workspace
|   +-- Múi giờ / định dạng ngày
|   +-- Cấu hình thông báo
|   +-- Cấu hình UI cơ bản
|   +-- Import / export dữ liệu
|
+-- 7. Nhật ký hệ thống
    +-- Audit log
    +-- Lịch sử thay đổi quyền
    +-- Lịch sử thay đổi workflow
    +-- Lịch sử thao tác dữ liệu
```

## 2. Điều hướng đề xuất

### Sidebar trái

```text
+--------------------------------------------------+
| LOGO / WORKSPACE                                 |
|--------------------------------------------------|
| Dashboard                                        |
| Phòng ban                                        |
| Người dùng                                       |
| Vai trò & quyền                                  |
| Workflow / State                                 |
| Cấu hình hệ thống                                |
| Nhật ký hệ thống                                 |
|--------------------------------------------------|
| Quay về Kanban                                   |
| Hồ sơ cá nhân                                    |
+--------------------------------------------------+
```

### Header trên

```text
+----------------------------------------------------------------------------------+
| Breadcrumb | Tìm kiếm nhanh | Thông báo | User menu                              |
+----------------------------------------------------------------------------------+
```

## 3. Wireframe tổng thể

```text
+--------------------------------------------------------------------------------------------------+
| Header: Breadcrumb / Search / Notification / Profile                                             |
+----------------------+---------------------------------------------------------------------------+
| Sidebar              | Page title                                                                |
| - Dashboard          | Mô tả ngắn trang                                                          |
| - Phòng ban          |---------------------------------------------------------------------------|
| - Người dùng         | Toolbar: Search | Filter | Sort | Export | Add New                        |
| - Vai trò & quyền    |---------------------------------------------------------------------------|
| - Workflow / State   | Content area                                                              |
| - Cấu hình hệ thống  |                                                                           |
| - Nhật ký hệ thống   |                                                                           |
|                      |                                                                           |
+----------------------+---------------------------------------------------------------------------+
```

## 4. Wireframe chi tiết theo 7 module

### 4.1. Dashboard

Mục tiêu: cho admin thấy tình trạng hệ thống ngay khi vào trang.

```text
+--------------------------------------------------------------------------------------------------+
| Dashboard                                                                                        |
| Tổng quan vận hành hệ thống                                                                      |
|--------------------------------------------------------------------------------------------------|
| [Tổng phòng ban]   [Tổng user]   [Tổng role]   [Tổng state]   [Lỗi cấu hình]                    |
|--------------------------------------------------------------------------------------------------|
| Biểu đồ user theo phòng ban         | Biểu đồ user theo role                                    |
|-------------------------------------+------------------------------------------------------------|
| Cảnh báo cấu hình                   | Hoạt động gần đây                                          |
| - User chưa gán phòng ban           | - Admin A cập nhật workflow                                |
| - State chưa có màu                 | - Admin B tạo user mới                                     |
| - Role chưa có permission           | - Admin C đổi quyền Manager                                |
+--------------------------------------------------------------------------------------------------+
```

### 4.2. Khối/Phòng/Nhóm

Mục tiêu: quản lý cấu trúc tổ chức theo 3 cấp riêng biệt gồm `Khối`, `Phòng`, `Nhóm`.

Ví dụ mã cấu trúc:

```text
DB.WEB.PO
DB   = Khối Digital Banking
WEB  = Phòng Website
PO   = Nhóm Product Owner
```

Phạm vi admin ở module này:

- CRUD `Khối`
- CRUD `Phòng`
- CRUD `Nhóm`
- Xem quan hệ phân cấp `Khối > Phòng > Nhóm`

```text
+--------------------------------------------------------------------------------------------------+
| Khối / Phòng / Nhóm                                                                             |
|--------------------------------------------------------------------------------------------------|
| Tab: [Khối] [Phòng] [Nhóm]                                                                       |
|--------------------------------------------------------------------------------------------------|
| Search | Filter trạng thái | Filter khối | Filter phòng | Export | [ + Tạo mới ]               |
|--------------------------------------------------------------------------------------------------|
| Cây cấu trúc tổ chức                     | Danh sách / Chi tiết                                   |
|------------------------------------------+--------------------------------------------------------|
| - DB | Digital Banking                   | Đối tượng đang xem: Nhóm                               |
|   - WEB | Website                        | Mã nhóm: PO                                            |
|     - PO | Product Owner                 | Tên nhóm: Product Owner                                |
|     - DEV | Developer                    | Thuộc phòng: WEB                                       |
|   - APP | Mobile App                     | Thuộc khối: DB                                         |
| - RB | Retail Banking                    | Trưởng nhóm: An Nguyen                                 |
|                                          | Trạng thái: Active                                     |
|                                          | Số user: 12                                            |
|                                          | Mã đầy đủ: DB.WEB.PO                                   |
|                                          | [Sửa] [Xóa] [Xem user]                                 |
+--------------------------------------------------------------------------------------------------+
```

Wireframe CRUD `Khối`:

```text
+--------------------------------------------------------------------------------------------------+
| Khối                                                         [ + Tạo khối ]                     |
|--------------------------------------------------------------------------------------------------|
| Search khối | Filter trạng thái | Export                                                      |
|--------------------------------------------------------------------------------------------------|
| Mã khối | Tên khối            | Trạng thái | Số phòng | Action                                 |
|--------------------------------------------------------------------------------------------------|
| DB      | Digital Banking     | Active     | 8        | View Edit Delete                       |
| RB      | Retail Banking      | Active     | 5        | View Edit Delete                       |
+--------------------------------------------------------------------------------------------------+
```

Wireframe CRUD `Phòng`:

```text
+--------------------------------------------------------------------------------------------------+
| Phòng                                                        [ + Tạo phòng ]                    |
|--------------------------------------------------------------------------------------------------|
| Search phòng | Filter khối | Filter trạng thái | Export                                          |
|--------------------------------------------------------------------------------------------------|
| Mã phòng | Tên phòng          | Thuộc khối | Trạng thái | Số nhóm | Action                     |
|--------------------------------------------------------------------------------------------------|
| WEB      | Website            | DB         | Active     | 3       | View Edit Delete           |
| APP      | Mobile App         | DB         | Active     | 4       | View Edit Delete           |
+--------------------------------------------------------------------------------------------------+
```

Wireframe CRUD `Nhóm`:

```text
+--------------------------------------------------------------------------------------------------+
| Nhóm                                                         [ + Tạo nhóm ]                     |
|--------------------------------------------------------------------------------------------------|
| Search nhóm | Filter khối | Filter phòng | Filter trạng thái | Export                           |
|--------------------------------------------------------------------------------------------------|
| Mã nhóm | Tên nhóm           | Thuộc phòng | Thuộc khối | Trạng thái | Action                  |
|--------------------------------------------------------------------------------------------------|
| PO      | Product Owner      | WEB         | DB         | Active     | View Edit Delete         |
| DEV     | Developer          | WEB         | DB         | Active     | View Edit Delete         |
+--------------------------------------------------------------------------------------------------+
```

Form tạo/sửa `Khối`:

```text
+--------------------------------------------------------------+
| Tạo / Sửa khối                                               |
|--------------------------------------------------------------|
| Mã khối            [__________________________]              |
| Tên khối           [__________________________]              |
| Thứ tự hiển thị    [________]                                |
| Trạng thái         [ Active v ]                              |
| Ghi chú            [______________________________]          |
|                                      [Hủy] [Lưu thay đổi]    |
+--------------------------------------------------------------+
```

Form tạo/sửa `Phòng`:

```text
+--------------------------------------------------------------+
| Tạo / Sửa phòng                                              |
|--------------------------------------------------------------|
| Mã phòng            [__________________________]             |
| Tên phòng           [__________________________]             |
| Thuộc khối          [ dropdown                ]              |
| Thứ tự hiển thị     [________]                               |
| Trạng thái          [ Active v ]                             |
| Ghi chú             [______________________________]         |
|                                      [Hủy] [Lưu thay đổi]    |
+--------------------------------------------------------------+
```

Form tạo/sửa `Nhóm`:

```text
+--------------------------------------------------------------+
| Tạo / Sửa nhóm                                               |
|--------------------------------------------------------------|
| Mã nhóm             [__________________________]             |
| Tên nhóm            [__________________________]             |
| Thuộc khối          [ dropdown                ]              |
| Thuộc phòng         [ dropdown                ]              |
| Trưởng nhóm         [ dropdown                ]              |
| Thứ tự hiển thị     [________]                               |
| Trạng thái          [ Active v ]                             |
| Ghi chú             [______________________________]         |
|                                      [Hủy] [Lưu thay đổi]    |
+--------------------------------------------------------------+
```

### 4.3. Người dùng

Mục tiêu: quản lý hồ sơ user, phân bổ phòng ban, trạng thái hoạt động.

```text
+--------------------------------------------------------------------------------------------------+
| Người dùng                                                  [ + Tạo user ]                      |
|--------------------------------------------------------------------------------------------------|
| Search tên/email | Filter phòng ban | Filter role | Filter trạng thái | Import | Export         |
|--------------------------------------------------------------------------------------------------|
| Table users                                                                                      |
|--------------------------------------------------------------------------------------------------|
| Avatar | Username | Họ tên | Email | Chức danh | Phòng ban | Role | Status | Action            |
|--------------------------------------------------------------------------------------------------|
| a.AN   | an...    | ...    | ...   | Head      | Web PO    | Admin| Active | View Edit Disable |
| Hiếu   | hieu...  | ...    | ...   | PO        | Web PO    | Mgr  | Active | ...               |
+--------------------------------------------------------------------------------------------------+
```

Drawer chi tiết user:

```text
+-----------------------------------------------------------------------+
| Chi tiết user                                                         |
|-----------------------------------------------------------------------|
| Avatar / màu nhận diện                                                |
| Username: an.nguyen                                                   |
| Họ tên: a.AN                                                          |
| Email: an.nguyen@...                                                  |
| Chức danh: Head                                                       |
| Phòng ban chính: Web PO                                               |
| Vai trò hiện tại: Super Admin, Manager                                |
| Trạng thái: Active                                                    |
|-----------------------------------------------------------------------|
| [Reset mật khẩu] [Gán role] [Chuyển phòng ban] [Ngưng hoạt động]      |
+-----------------------------------------------------------------------+
```

### 4.4. Vai trò & quyền

Mục tiêu: quản lý quyền theo vai trò và phạm vi.

```text
+--------------------------------------------------------------------------------------------------+
| Vai trò & quyền                                            [ + Tạo role ]                       |
|--------------------------------------------------------------------------------------------------|
| Tab: [Role] [Permission] [Ma trận quyền] [Gán quyền user/team]                                   |
|--------------------------------------------------------------------------------------------------|
| Danh sách role                         | Chi tiết role                                            |
|----------------------------------------+---------------------------------------------------------|
| Super Admin                            | Tên role: Manager                                       |
| Admin                                  | Mô tả: Quản lý team và task                             |
| Manager                                | Permission:                                              |
| Member                                 | [x] View board                                           |
| Viewer                                 | [x] Create task                                          |
|                                        | [x] Edit task                                            |
|                                        | [x] Move task                                            |
|                                        | [ ] Delete task                                          |
|                                        | Scope: [Toàn hệ thống / Theo team]                       |
|                                        | [Lưu]                                                    |
+--------------------------------------------------------------------------------------------------+
```

Ma trận quyền:

```text
+----------------------------------------------------------------------------------------------+
| Permission            | Super Admin | Admin | Manager | Member | Viewer                      |
|----------------------------------------------------------------------------------------------|
| View board            |      x      |   x   |    x    |   x    |   x                         |
| Create task           |      x      |   x   |    x    |   x    |                             |
| Edit task             |      x      |   x   |    x    |   x    |                             |
| Move task             |      x      |   x   |    x    |   x    |                             |
| Delete task           |      x      |   x   |    x    |        |                             |
| Manage users          |      x      |   x   |         |        |                             |
| Manage workflow       |      x      |   x   |         |        |                             |
+----------------------------------------------------------------------------------------------+
```

### 4.5. Workflow / State

Mục tiêu: chỉ cấu hình state ở mức đơn giản, gồm:

- Sắp xếp lại thứ tự hiển thị
- Điều chỉnh wording hiển thị
- Thêm mới một state

```text
+--------------------------------------------------------------------------------------------------+
| Workflow / State                                           [ + Tạo state ]                      |
|--------------------------------------------------------------------------------------------------|
| Search state | Drag to reorder | Preview board                                                   |
|--------------------------------------------------------------------------------------------------|
| Danh sách state                         | Cấu hình state                                          |
|-----------------------------------------+--------------------------------------------------------|
| 1. Backlog                              | Mã state: BACKLOG                                       |
| 2. In Progress                          | Wording hiển thị: In Progress                           |
| 3. In Review                            | Tên ngắn: Review                                        |
| 4. Done                                 | Thứ tự hiển thị: 2                                      |
| 5. Pending                              | Trạng thái: Active                                      |
| 6. Cancel                               | [Lưu]                                                   |
+--------------------------------------------------------------------------------------------------+
```

Form thêm mới state:

```text
+--------------------------------------------------------------+
| Tạo state mới                                                |
|--------------------------------------------------------------|
| Mã state             [__________________________]            |
| Wording hiển thị     [__________________________]            |
| Tên ngắn             [__________________________]            |
| Thứ tự hiển thị      [________]                              |
| Trạng thái           [ Active v ]                            |
|                                      [Hủy] [Lưu state]       |
+--------------------------------------------------------------+
```

### 4.6. Cấu hình hệ thống

Mục tiêu: cấu hình thông số chung của workspace.

```text
+--------------------------------------------------------------------------------------------------+
| Cấu hình hệ thống                                                                                |
|--------------------------------------------------------------------------------------------------|
| Tab: [Chung] [Thông báo] [Hiển thị] [Import/Export]                                              |
|--------------------------------------------------------------------------------------------------|
| Tên workspace            [ Task Tracking Elite                ]                                  |
| Múi giờ                  [ UTC+7                              ]                                  |
| Định dạng ngày           [ dd/MM/yyyy                         ]                                  |
| Màu thương hiệu          [ #2563EB ]                                                           |
| Trang mặc định sau login [ Dashboard v ]                                                       |
|--------------------------------------------------------------------------------------------------|
| Thông báo khi tạo task        [x]                                                               |
| Thông báo khi assign user     [x]                                                               |
| Thông báo khi đổi state       [x]                                                               |
| Cảnh báo quá hạn              [x]                                                               |
|--------------------------------------------------------------------------------------------------|
|                                              [Hủy] [Lưu cấu hình]                               |
+--------------------------------------------------------------------------------------------------+
```

### 4.7. Nhật ký hệ thống

Mục tiêu: theo dõi ai đã thay đổi gì trong hệ thống.

```text
+--------------------------------------------------------------------------------------------------+
| Nhật ký hệ thống                                                                                 |
|--------------------------------------------------------------------------------------------------|
| Search log | Filter module | Filter action | Filter user | Date range | Export                  |
|--------------------------------------------------------------------------------------------------|
| Thời gian           | User        | Module           | Hành động        | Chi tiết               |
|--------------------------------------------------------------------------------------------------|
| 19/04 09:20         | an.nguyen   | Workflow         | UPDATE_STATE     | đổi màu In Review      |
| 19/04 09:05         | hieu.phan   | User             | CREATE_USER      | tạo user U0015         |
| 19/04 08:55         | an.nguyen   | Permission       | UPDATE_ROLE      | thêm MOVE_TASK         |
| 19/04 08:30         | van.le      | Department       | UPDATE_TEAM      | đổi trưởng nhóm        |
+--------------------------------------------------------------------------------------------------+
```

## 5. Luồng sử dụng chính

### Luồng 1: tạo khối / phòng / nhóm

```text
Admin > Phòng ban > Chọn tab Khối/Phòng/Nhóm > + Tạo mới > Nhập thông tin > Lưu > Xuất hiện trong cây cấu trúc
```

### Luồng 2: tạo user và gán phòng ban

```text
Admin > Người dùng > + Tạo user > Nhập hồ sơ > Chọn phòng ban > Chọn chức danh > Lưu
```

### Luồng 3: cấp quyền cho user theo team

```text
Admin > Vai trò & quyền > Gán quyền user/team > Chọn user > Chọn team > Chọn role > Lưu
```

### Luồng 4: thêm state mới vào workflow

```text
Admin > Workflow / State > + Tạo state > Nhập mã + wording + thứ tự > Lưu > Preview board
```

## 6. Ưu tiên triển khai UI

### Phase 1

- Dashboard
- Phòng ban
- Người dùng
- Workflow / State
- Vai trò & quyền

### Phase 2

- Vai trò & quyền
- Cấu hình hệ thống
- Nhật ký hệ thống

## 7. Gợi ý UI pattern

- Danh sách lớn dùng `table + filter + bulk action`.
- Chi tiết nhanh dùng `drawer` thay vì mở page mới.
- Tạo/sửa dùng `modal` cho form ngắn, dùng `full page form` cho form dài.
- Workflow nên có `preview board` để admin nhìn thấy tác động ngay.
- Audit log nên hỗ trợ `export` để phục vụ kiểm tra nội bộ.

## 8. Gợi ý bước tiếp theo

- Dựng mockup HTML cho toàn bộ trang Admin.
- Tạo dữ liệu mẫu cho `department`, `user`, `role`, `state`.
- Tách `index.html` thành các section rõ ràng: `kanban`, `admin`, `settings`.
- Chuẩn hóa model dữ liệu trước khi nối API thật.
