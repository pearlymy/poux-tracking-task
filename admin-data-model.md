# Admin Data Model

Tài liệu này mô tả data model cho phần Admin, tập trung vào 4 nhóm chính:

1. `Block` (`Khối`)
2. `Department` (`Phòng`)
3. `Team` (`Nhóm`)
4. `State` (`Workflow State`)

Phạm vi bám theo yêu cầu hiện tại:

- CRUD `Khối`
- CRUD `Phòng`
- CRUD `Nhóm`
- CRUD `State` đơn giản
- `State` chỉ cần hỗ trợ `thứ tự`, `wording`, `thêm mới`

## 1. Nguyên tắc model

- Tách riêng `Khối`, `Phòng`, `Nhóm` thành 3 bảng để CRUD độc lập.
- Không lưu gộp `DB.WEB.PO` thành một entity duy nhất.
- `code` của từng cấp là mã ngắn, dùng để ghép thành mã đầy đủ khi cần.
- Mỗi `Phòng` phải thuộc đúng một `Khối`.
- Mỗi `Nhóm` phải thuộc đúng một `Phòng`.
- `State` là danh mục riêng, không cần quản lý transition phức tạp ở giai đoạn này.

## 2. Mô hình quan hệ

```text
Block (Khối)
  1 --- n Department (Phòng)
            1 --- n Team (Nhóm)

State
  danh mục độc lập, dùng cho cột kanban / wording hiển thị
```

Ví dụ:

```text
Block.code      = DB
Department.code = WEB
Team.code       = PO

=> full_team_code = DB.WEB.PO
```

## 3. Bảng `blocks`

Mục đích: lưu danh mục `Khối`.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `code` | VARCHAR(20) | Yes | Unique | Mã khối, ví dụ `DB` |
| `name` | VARCHAR(255) | Yes |  | Tên khối, ví dụ `Digital Banking` |
| `display_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_active` | BOOLEAN | Yes | Default true | Trạng thái hoạt động |
| `description` | VARCHAR(500) | No |  | Ghi chú |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Ràng buộc:

- `code` là duy nhất toàn hệ thống.
- Nên chuẩn hóa `code` thành uppercase.

## 4. Bảng `departments`

Mục đích: lưu danh mục `Phòng`.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `block_id` | UUID / BIGINT | Yes | FK -> `blocks.id` | Phòng thuộc khối nào |
| `code` | VARCHAR(20) | Yes |  | Mã phòng, ví dụ `WEB` |
| `name` | VARCHAR(255) | Yes |  | Tên phòng, ví dụ `Website` |
| `display_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_active` | BOOLEAN | Yes | Default true | Trạng thái hoạt động |
| `description` | VARCHAR(500) | No |  | Ghi chú |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Ràng buộc:

- `block_id` phải tồn tại trong `blocks`.
- `code` nên unique trong phạm vi một `Khối`.
- Có thể dùng unique key: `unique(block_id, code)`.

## 5. Bảng `teams`

Mục đích: lưu danh mục `Nhóm`.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `department_id` | UUID / BIGINT | Yes | FK -> `departments.id` | Nhóm thuộc phòng nào |
| `code` | VARCHAR(20) | Yes |  | Mã nhóm, ví dụ `PO` |
| `name` | VARCHAR(255) | Yes |  | Tên nhóm, ví dụ `Product Owner` |
| `leader_user_id` | VARCHAR(50) / UUID | No | FK mềm tới user | Trưởng nhóm |
| `display_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_active` | BOOLEAN | Yes | Default true | Trạng thái hoạt động |
| `description` | VARCHAR(500) | No |  | Ghi chú |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Ràng buộc:

- `department_id` phải tồn tại trong `departments`.
- `code` nên unique trong phạm vi một `Phòng`.
- Có thể dùng unique key: `unique(department_id, code)`.

## 6. Mã đầy đủ để hiển thị

Không bắt buộc lưu cứng cột `full_code` trong DB nếu có thể tính từ quan hệ.

Quy tắc:

```text
full_team_code = block.code + '.' + department.code + '.' + team.code
```

Ví dụ:

```text
DB + WEB + PO = DB.WEB.PO
```

Nếu hệ thống cần tìm kiếm rất nhiều theo mã đầy đủ, có thể thêm:

| Field | Table | Ý nghĩa |
|---|---|---|
| `full_code` | `teams` | Cache mã đầy đủ `DB.WEB.PO` |

Khi đó cần cơ chế đồng bộ nếu `Block.code` hoặc `Department.code` thay đổi.

## 7. Bảng `states`

Mục đích: lưu danh sách state hiển thị trên board.

Theo yêu cầu hiện tại, bảng này chỉ cần phục vụ:

- thêm mới state
- sửa wording hiển thị
- sắp xếp thứ tự hiển thị

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `code` | VARCHAR(50) | Yes | Unique | Mã state, ví dụ `IN_PROGRESS` |
| `label` | VARCHAR(255) | Yes |  | Wording hiển thị, ví dụ `In Progress` |
| `short_label` | VARCHAR(100) | No |  | Tên ngắn, ví dụ `Review` |
| `display_order` | INT | Yes |  | Thứ tự hiển thị trên board |
| `is_active` | BOOLEAN | Yes | Default true | Còn dùng hay không |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Ràng buộc:

- `code` là duy nhất toàn hệ thống.
- `display_order` không nên trùng trong tập state active nếu UI cần thứ tự rõ ràng.

## 8. Quan hệ với `users`

Prototype hiện tại đã có `USERS` trong [index.html](/workspaces/poux-tracking-task/poux-tracking-task/index.html:2290), nhưng chưa có model tổ chức chuẩn hóa.

Để nối `user` với `Khối / Phòng / Nhóm`, mình đề xuất tối thiểu như sau:

### Cách đơn giản nhất cho giai đoạn đầu

Thêm `team_id` vào bảng `users`.

| Field | Type | Required | Ý nghĩa |
|---|---|---:|---|
| `team_id` | UUID / BIGINT | No | User thuộc nhóm nào |

Khi đó:

- từ `team_id` suy ra `department`
- từ `department` suy ra `block`

### Nếu một user có thể thuộc nhiều nhóm

Dùng bảng trung gian `user_teams`.

| Field | Type | Required | Ý nghĩa |
|---|---|---:|---|
| `id` | UUID / BIGINT | Yes | PK |
| `user_id` | UUID / BIGINT | Yes | FK tới user |
| `team_id` | UUID / BIGINT | Yes | FK tới team |
| `is_primary` | BOOLEAN | Yes | Nhóm chính hay phụ |
| `created_at` | DATETIME | Yes | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No | Người tạo |

Với prototype hiện tại, nếu bạn đang dùng `team_code` trong phân quyền như `DB.WEB.PO`, mình khuyên:

- giữ `team_code` ở lớp view/API trong giai đoạn chuyển tiếp
- nhưng DB nên tham chiếu bằng `team_id`

## 9. API CRUD gợi ý

### `blocks`

- `GET /admin/blocks`
- `POST /admin/blocks`
- `GET /admin/blocks/:id`
- `PUT /admin/blocks/:id`
- `DELETE /admin/blocks/:id`

### `departments`

- `GET /admin/departments`
- `POST /admin/departments`
- `GET /admin/departments/:id`
- `PUT /admin/departments/:id`
- `DELETE /admin/departments/:id`

Filter nên có:

- `block_id`
- `is_active`
- `keyword`

### `teams`

- `GET /admin/teams`
- `POST /admin/teams`
- `GET /admin/teams/:id`
- `PUT /admin/teams/:id`
- `DELETE /admin/teams/:id`

Filter nên có:

- `block_id`
- `department_id`
- `is_active`
- `keyword`

### `states`

- `GET /admin/states`
- `POST /admin/states`
- `GET /admin/states/:id`
- `PUT /admin/states/:id`
- `DELETE /admin/states/:id`
- `PATCH /admin/states/reorder`

## 10. Payload mẫu

### Tạo `Block`

```json
{
  "code": "DB",
  "name": "Digital Banking",
  "display_order": 1,
  "is_active": true,
  "description": "Khoi Digital Banking"
}
```

### Tạo `Department`

```json
{
  "block_id": "blk_001",
  "code": "WEB",
  "name": "Website",
  "display_order": 1,
  "is_active": true
}
```

### Tạo `Team`

```json
{
  "department_id": "dep_001",
  "code": "PO",
  "name": "Product Owner",
  "leader_user_id": "U0001",
  "display_order": 1,
  "is_active": true
}
```

### Tạo `State`

```json
{
  "code": "IN_PROGRESS",
  "label": "In Progress",
  "short_label": "Doing",
  "display_order": 2,
  "is_active": true
}
```

### Reorder `State`

```json
{
  "items": [
    { "id": "st_001", "display_order": 1 },
    { "id": "st_002", "display_order": 2 },
    { "id": "st_003", "display_order": 3 }
  ]
}
```

## 11. DDL tham khảo

```sql
create table blocks (
  id varchar(36) primary key,
  code varchar(20) not null unique,
  name varchar(255) not null,
  display_order int default 0,
  is_active boolean not null default true,
  description varchar(500),
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50)
);

create table departments (
  id varchar(36) primary key,
  block_id varchar(36) not null,
  code varchar(20) not null,
  name varchar(255) not null,
  display_order int default 0,
  is_active boolean not null default true,
  description varchar(500),
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50),
  constraint fk_departments_block foreign key (block_id) references blocks(id),
  constraint uq_departments_block_code unique (block_id, code)
);

create table teams (
  id varchar(36) primary key,
  department_id varchar(36) not null,
  code varchar(20) not null,
  name varchar(255) not null,
  leader_user_id varchar(50),
  display_order int default 0,
  is_active boolean not null default true,
  description varchar(500),
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50),
  constraint fk_teams_department foreign key (department_id) references departments(id),
  constraint uq_teams_department_code unique (department_id, code)
);

create table states (
  id varchar(36) primary key,
  code varchar(50) not null unique,
  label varchar(255) not null,
  short_label varchar(100),
  display_order int not null,
  is_active boolean not null default true,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50)
);
```

## 12. Khuyến nghị triển khai

- Giai đoạn đầu chưa cần soft delete, có thể dùng `is_active`.
- Không nên cho phép xóa `Khối` nếu còn `Phòng`.
- Không nên cho phép xóa `Phòng` nếu còn `Nhóm`.
- Không nên cho phép xóa `State` nếu đang được task sử dụng.
- Nếu sửa `code`, nên kiểm tra ảnh hưởng tới dữ liệu phân quyền đang dùng `team_code`.

## 13. Đề xuất bước tiếp theo

- Chuẩn hóa model `users` để gắn với `team_id`.
- Chuẩn hóa model `roles` và bảng mapping user-role-team.
- Tạo mock API response cho 4 module admin này.
- Chuyển model này thành schema SQL hoặc Prisma nếu bạn chuẩn bị code backend.

## 14. Mở rộng model cho `users`, `roles`, `permissions`

Phần này bám theo cấu trúc prototype hiện có trong [index.html](/workspaces/poux-tracking-task/poux-tracking-task/index.html:2290), gồm:

- `USERS`
- `JOB_TITLES`
- `ROLES`
- `ROLE_PERMISSIONS`
- `USER_ROLES_TEAM`

Mục tiêu của bản model mở rộng:

- chuẩn hóa user
- chuẩn hóa chức danh
- chuẩn hóa role và permission
- chuẩn hóa quan hệ `user - role - team`
- vẫn hỗ trợ dữ liệu cũ đang dùng `team_code`

## 15. Bảng `job_titles`

Mục đích: chuẩn hóa chức danh như `HEAD`, `PO`, `UX`, `DEV_MANAGER`, `BE`, `FE`, `QC`.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `code` | VARCHAR(50) | Yes | Unique | Mã chức danh |
| `name` | VARCHAR(255) | Yes |  | Tên hiển thị |
| `display_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_active` | BOOLEAN | Yes | Default true | Trạng thái hoạt động |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

## 16. Bảng `users`

Mục đích: lưu hồ sơ user phục vụ phân công task, lọc board và phân quyền.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | VARCHAR(50) / UUID | Yes | PK | Mã user, có thể giữ kiểu `U0001` để tương thích prototype |
| `username` | VARCHAR(100) | Yes | Unique | Tên đăng nhập |
| `email` | VARCHAR(255) | Yes | Unique | Email |
| `full_name` | VARCHAR(255) | Yes |  | Tên hiển thị |
| `job_title_id` | UUID / BIGINT | No | FK -> `job_titles.id` | Chức danh |
| `color` | VARCHAR(20) | No |  | Màu nhận diện avatar/tag |
| `sort_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_active` | BOOLEAN | Yes | Default true | Trạng thái hoạt động |
| `last_login_at` | DATETIME | No |  | Lần đăng nhập gần nhất |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Ràng buộc:

- `username` là duy nhất.
- `email` là duy nhất.
- Nếu muốn tương thích prototype nhanh, có thể giữ `id` dạng text như `U0001`.

Gợi ý mapping với prototype:

| Prototype field | DB field |
|---|---|
| `id` | `users.id` |
| `username` | `users.username` |
| `email` | `users.email` |
| `full_name` | `users.full_name` |
| `job_title_id` | `users.job_title_id` |
| `color` | `users.color` |
| `sort` | `users.sort_order` |
| `is_active` | `users.is_active` |

## 17. Quan hệ `user - team`

Để tránh khóa cứng user vào một nhóm duy nhất, nên dùng bảng trung gian `user_teams`.

### Bảng `user_teams`

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `user_id` | VARCHAR(50) / UUID | Yes | FK -> `users.id` | User |
| `team_id` | UUID / BIGINT | Yes | FK -> `teams.id` | Nhóm |
| `is_primary` | BOOLEAN | Yes | Default false | Nhóm chính của user |
| `joined_at` | DATETIME | No |  | Ngày tham gia |
| `left_at` | DATETIME | No |  | Ngày rời nhóm |
| `is_active` | BOOLEAN | Yes | Default true | Còn hiệu lực hay không |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Ràng buộc:

- `unique(user_id, team_id)` để tránh gán trùng.
- mỗi user chỉ nên có tối đa 1 dòng `is_primary = true` đang active.

## 18. Bảng `roles`

Mục đích: lưu vai trò nghiệp vụ như `SUPER_ADMIN`, `MANAGER`, `MEMBER`, `INTERN`, `VIEWER`.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính nội bộ |
| `code` | VARCHAR(50) | Yes | Unique | Mã role |
| `name` | VARCHAR(255) | Yes |  | Tên hiển thị |
| `description` | VARCHAR(500) | No |  | Mô tả |
| `scope_type` | VARCHAR(30) | Yes |  | Phạm vi mặc định của role |
| `display_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_system` | BOOLEAN | Yes | Default false | Role hệ thống hay tự tạo |
| `is_active` | BOOLEAN | Yes | Default true | Còn sử dụng hay không |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Gợi ý `scope_type`:

- `GLOBAL`
- `TEAM`

Ví dụ:

- `SUPER_ADMIN` thường mang tính `GLOBAL`
- `MANAGER`, `MEMBER`, `INTERN`, `VIEWER` thường mang tính `TEAM`

## 19. Bảng `permissions`

Mục đích: chuẩn hóa action hệ thống để gán vào role.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `code` | VARCHAR(100) | Yes | Unique | Mã permission |
| `name` | VARCHAR(255) | Yes |  | Tên hiển thị |
| `module` | VARCHAR(100) | No |  | Nhóm chức năng, ví dụ `TASK`, `ADMIN`, `WORKFLOW` |
| `description` | VARCHAR(500) | No |  | Mô tả |
| `display_order` | INT | No | Default 0 | Thứ tự hiển thị |
| `is_active` | BOOLEAN | Yes | Default true | Còn sử dụng hay không |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Gợi ý danh sách permission tối thiểu:

- `VIEW_BOARD`
- `CREATE_TASK`
- `EDIT_TASK`
- `MOVE_TASK`
- `DELETE_TASK`
- `MANAGE_USERS`
- `MANAGE_ORG`
- `MANAGE_ROLES`
- `MANAGE_STATES`
- `VIEW_AUDIT_LOG`

## 20. Bảng `role_permissions`

Mục đích: mapping nhiều-nhiều giữa `roles` và `permissions`.

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `role_id` | UUID / BIGINT | Yes | FK -> `roles.id` | Role |
| `permission_id` | UUID / BIGINT | Yes | FK -> `permissions.id` | Permission |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |

Ràng buộc:

- `unique(role_id, permission_id)`

Gợi ý mapping từ prototype:

| Prototype | DB |
|---|---|
| `ROLE_PERMISSIONS.role_id` | `roles.code` |
| `ROLE_PERMISSIONS.permission_id` | `permissions.code` |

## 21. Bảng `user_role_assignments`

Mục đích: chuẩn hóa `USER_ROLES_TEAM`.

Đây là bảng quan trọng nhất để xác định:

- user nào có role gì
- trong phạm vi nào
- còn hiệu lực hay không

| Field | Type | Required | Gợi ý | Ý nghĩa |
|---|---|---:|---|---|
| `id` | UUID / BIGINT | Yes | PK | Khóa chính |
| `user_id` | VARCHAR(50) / UUID | Yes | FK -> `users.id` | User được gán quyền |
| `role_id` | UUID / BIGINT | Yes | FK -> `roles.id` | Role được gán |
| `scope_type` | VARCHAR(30) | Yes |  | Phạm vi áp dụng |
| `team_id` | UUID / BIGINT | No | FK -> `teams.id` | Dùng khi role áp dụng theo nhóm |
| `legacy_team_code` | VARCHAR(100) | No |  | Giữ để tương thích dữ liệu cũ |
| `starts_at` | DATETIME | No |  | Thời điểm bắt đầu hiệu lực |
| `ends_at` | DATETIME | No |  | Thời điểm hết hiệu lực |
| `is_active` | BOOLEAN | Yes | Default true | Còn hiệu lực hay không |
| `created_at` | DATETIME | Yes |  | Ngày tạo |
| `created_by` | VARCHAR(50) / UUID | No |  | Người tạo |
| `updated_at` | DATETIME | Yes |  | Ngày cập nhật |
| `updated_by` | VARCHAR(50) / UUID | No |  | Người cập nhật |

Quy ước `scope_type`:

- `GLOBAL`
- `TEAM`

Quy tắc dùng:

- nếu `scope_type = GLOBAL` thì `team_id = null`
- nếu `scope_type = TEAM` thì `team_id` là bắt buộc

Gợi ý mapping từ prototype:

| Prototype field | DB field |
|---|---|
| `user_id` | `user_role_assignments.user_id` |
| `role_id` | map sang `roles.id` thông qua `roles.code` |
| `team_code` | `legacy_team_code` hoặc quy đổi sang `team_id` |

## 22. Tại sao nên dùng `team_id` thay vì chỉ dùng `team_code`

`team_code` như `DB.WEB.PO` rất tốt cho hiển thị và trao đổi nghiệp vụ, nhưng không nên là khóa chính để join lâu dài vì:

- đổi `code` ở `Khối` hoặc `Phòng` sẽ kéo theo đổi toàn bộ chuỗi
- dễ phát sinh sai chính tả hoặc lệch format
- khó enforce foreign key

Hướng khuyến nghị:

- DB nội bộ dùng `team_id`
- API response vẫn có thể trả thêm `team_code`
- giai đoạn chuyển đổi có thể giữ `legacy_team_code`

## 23. Luồng xác định quyền

Quy tắc gợi ý khi check quyền:

1. Lấy tất cả assignment đang active của user.
2. Giữ các assignment có `scope_type = GLOBAL`.
3. Nếu đang đứng ở một team cụ thể, lấy thêm assignment có `scope_type = TEAM` và `team_id` tương ứng.
4. Từ danh sách role tìm ra permission qua `role_permissions`.
5. Kết luận user có quyền hay không.

Pseudo logic:

```text
effective_roles =
  global_roles(user_id)
  + team_roles(user_id, current_team_id)

effective_permissions =
  permissions_of_roles(effective_roles)
```

## 24. API CRUD gợi ý cho phần mở rộng

### `job_titles`

- `GET /admin/job-titles`
- `POST /admin/job-titles`
- `PUT /admin/job-titles/:id`
- `DELETE /admin/job-titles/:id`

### `users`

- `GET /admin/users`
- `POST /admin/users`
- `GET /admin/users/:id`
- `PUT /admin/users/:id`
- `DELETE /admin/users/:id`
- `PATCH /admin/users/:id/status`

Filter nên có:

- `keyword`
- `job_title_id`
- `team_id`
- `is_active`

### `roles`

- `GET /admin/roles`
- `POST /admin/roles`
- `GET /admin/roles/:id`
- `PUT /admin/roles/:id`
- `DELETE /admin/roles/:id`

### `permissions`

- `GET /admin/permissions`
- `POST /admin/permissions`
- `GET /admin/permissions/:id`
- `PUT /admin/permissions/:id`
- `DELETE /admin/permissions/:id`

### `role_permissions`

- `GET /admin/roles/:id/permissions`
- `PUT /admin/roles/:id/permissions`

### `user_role_assignments`

- `GET /admin/user-role-assignments`
- `POST /admin/user-role-assignments`
- `PUT /admin/user-role-assignments/:id`
- `DELETE /admin/user-role-assignments/:id`

Filter nên có:

- `user_id`
- `role_id`
- `team_id`
- `scope_type`
- `is_active`

## 25. Payload mẫu

### Tạo `User`

```json
{
  "id": "U0015",
  "username": "linh.nguyen",
  "email": "linh.nguyen@vib.com.vn",
  "full_name": "Linh",
  "job_title_id": "job_po",
  "color": "#0EA5E9",
  "sort_order": 15,
  "is_active": true
}
```

### Gán user vào team

```json
{
  "user_id": "U0015",
  "team_id": "team_po_001",
  "is_primary": true,
  "is_active": true
}
```

### Tạo `Role`

```json
{
  "code": "MANAGER",
  "name": "Manager",
  "description": "Quan ly task va thanh vien trong nhom",
  "scope_type": "TEAM",
  "display_order": 2,
  "is_system": true,
  "is_active": true
}
```

### Tạo `Permission`

```json
{
  "code": "MANAGE_STATES",
  "name": "Manage States",
  "module": "WORKFLOW",
  "description": "Duoc quan ly danh muc state",
  "display_order": 8,
  "is_active": true
}
```

### Gán permission cho role

```json
{
  "permission_ids": [
    "perm_view_board",
    "perm_create_task",
    "perm_edit_task"
  ]
}
```

### Gán role cho user theo team

```json
{
  "user_id": "U0002",
  "role_id": "role_member",
  "scope_type": "TEAM",
  "team_id": "team_po_001",
  "legacy_team_code": "DB.WEB.PO",
  "is_active": true
}
```

### Gán role toàn hệ thống

```json
{
  "user_id": "U0001",
  "role_id": "role_super_admin",
  "scope_type": "GLOBAL",
  "team_id": null,
  "legacy_team_code": null,
  "is_active": true
}
```

## 26. DDL tham khảo cho phần mở rộng

```sql
create table job_titles (
  id varchar(36) primary key,
  code varchar(50) not null unique,
  name varchar(255) not null,
  display_order int default 0,
  is_active boolean not null default true,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50)
);

create table users (
  id varchar(50) primary key,
  username varchar(100) not null unique,
  email varchar(255) not null unique,
  full_name varchar(255) not null,
  job_title_id varchar(36),
  color varchar(20),
  sort_order int default 0,
  is_active boolean not null default true,
  last_login_at datetime,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50),
  constraint fk_users_job_title foreign key (job_title_id) references job_titles(id)
);

create table user_teams (
  id varchar(36) primary key,
  user_id varchar(50) not null,
  team_id varchar(36) not null,
  is_primary boolean not null default false,
  joined_at datetime,
  left_at datetime,
  is_active boolean not null default true,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50),
  constraint fk_user_teams_user foreign key (user_id) references users(id),
  constraint fk_user_teams_team foreign key (team_id) references teams(id),
  constraint uq_user_teams unique (user_id, team_id)
);

create table roles (
  id varchar(36) primary key,
  code varchar(50) not null unique,
  name varchar(255) not null,
  description varchar(500),
  scope_type varchar(30) not null,
  display_order int default 0,
  is_system boolean not null default false,
  is_active boolean not null default true,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50)
);

create table permissions (
  id varchar(36) primary key,
  code varchar(100) not null unique,
  name varchar(255) not null,
  module varchar(100),
  description varchar(500),
  display_order int default 0,
  is_active boolean not null default true,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50)
);

create table role_permissions (
  id varchar(36) primary key,
  role_id varchar(36) not null,
  permission_id varchar(36) not null,
  created_at datetime not null,
  created_by varchar(50),
  constraint fk_role_permissions_role foreign key (role_id) references roles(id),
  constraint fk_role_permissions_permission foreign key (permission_id) references permissions(id),
  constraint uq_role_permissions unique (role_id, permission_id)
);

create table user_role_assignments (
  id varchar(36) primary key,
  user_id varchar(50) not null,
  role_id varchar(36) not null,
  scope_type varchar(30) not null,
  team_id varchar(36),
  legacy_team_code varchar(100),
  starts_at datetime,
  ends_at datetime,
  is_active boolean not null default true,
  created_at datetime not null,
  created_by varchar(50),
  updated_at datetime not null,
  updated_by varchar(50),
  constraint fk_user_role_assignments_user foreign key (user_id) references users(id),
  constraint fk_user_role_assignments_role foreign key (role_id) references roles(id),
  constraint fk_user_role_assignments_team foreign key (team_id) references teams(id)
);
```

## 27. Mapping từ prototype hiện tại sang model mới

### `USERS`

- map vào bảng `users`

### `JOB_TITLES`

- map vào bảng `job_titles`

### `ROLES`

- map vào bảng `roles`

### `ROLE_PERMISSIONS`

- map vào bảng `role_permissions`

### `USER_ROLES_TEAM`

- map vào bảng `user_role_assignments`

Ví dụ:

```text
{ user_id: 'U0001', role_id: 'SUPER_ADMIN', team_code: null }
=> user_role_assignments:
   - user_id = U0001
   - role = SUPER_ADMIN
   - scope_type = GLOBAL
   - team_id = null
   - legacy_team_code = null
```

```text
{ user_id: 'U0002', role_id: 'MEMBER', team_code: 'DB.WEB.PO' }
=> user_role_assignments:
   - user_id = U0002
   - role = MEMBER
   - scope_type = TEAM
   - team_id = team_po_001
   - legacy_team_code = DB.WEB.PO
```

## 28. Khuyến nghị triển khai

- Dùng `users.id` dạng text nếu cần đi nhanh với dữ liệu mẫu hiện tại.
- Ở tầng backend, tạo hàm resolve `legacy_team_code -> team_id` cho giai đoạn chuyển tiếp.
- Không nên cho xóa cứng role hệ thống như `SUPER_ADMIN`, `MANAGER`, `MEMBER`.
- Nên có validation ngăn một user bị gán trùng cùng `role + scope + team`.
- Khi disable user, nên giữ lịch sử `user_role_assignments` nhưng đánh `is_active = false`.

## 29. Bước tiếp theo phù hợp nhất

- Viết luôn seed data mẫu cho `job_titles`, `roles`, `permissions`.
- Thiết kế schema JSON/API response cho màn `Người dùng` và `Vai trò & quyền`.
- Nếu bạn chuẩn bị code backend, mình có thể chuyển toàn bộ tài liệu này thành file SQL hoặc Prisma schema.

## 30. Seed data mẫu

Mục tiêu của seed data:

- đủ dữ liệu để dựng màn admin
- bám sát prototype hiện tại
- dễ import vào DB hoặc mock API

### 30.1. `job_titles`

```json
[
  { "id": "job_head", "code": "HEAD", "name": "Head", "display_order": 1, "is_active": true },
  { "id": "job_po", "code": "PO", "name": "PO", "display_order": 2, "is_active": true },
  { "id": "job_ux", "code": "UX", "name": "UX", "display_order": 3, "is_active": true },
  { "id": "job_dev_manager", "code": "DEV_MANAGER", "name": "DEV Manager", "display_order": 4, "is_active": true },
  { "id": "job_be", "code": "BE", "name": "BE", "display_order": 5, "is_active": true },
  { "id": "job_fe", "code": "FE", "name": "FE", "display_order": 6, "is_active": true },
  { "id": "job_qc", "code": "QC", "name": "QC", "display_order": 7, "is_active": true }
]
```

### 30.2. `roles`

Lưu ý:

- không nên cho xóa cứng role hệ thống như `SUPER_ADMIN`, `MANAGER`, `MEMBER`
- các role hệ thống nên có `is_system = true`

```json
[
  {
    "id": "role_super_admin",
    "code": "SUPER_ADMIN",
    "name": "Super Admin",
    "description": "Toan quyen quan tri he thong",
    "scope_type": "GLOBAL",
    "display_order": 1,
    "is_system": true,
    "is_active": true
  },
  {
    "id": "role_manager",
    "code": "MANAGER",
    "name": "Manager",
    "description": "Quan ly task va thanh vien trong nhom",
    "scope_type": "TEAM",
    "display_order": 2,
    "is_system": true,
    "is_active": true
  },
  {
    "id": "role_member",
    "code": "MEMBER",
    "name": "Member",
    "description": "Thanh vien thuc hien cong viec",
    "scope_type": "TEAM",
    "display_order": 3,
    "is_system": true,
    "is_active": true
  },
  {
    "id": "role_intern",
    "code": "INTERN",
    "name": "Intern",
    "description": "Thanh vien thuc tap",
    "scope_type": "TEAM",
    "display_order": 4,
    "is_system": true,
    "is_active": true
  },
  {
    "id": "role_viewer",
    "code": "VIEWER",
    "name": "Viewer",
    "description": "Chi co quyen xem",
    "scope_type": "TEAM",
    "display_order": 5,
    "is_system": true,
    "is_active": true
  }
]
```

### 30.3. `permissions`

```json
[
  { "id": "perm_view_board", "code": "VIEW_BOARD", "name": "View Board", "module": "TASK", "display_order": 1, "is_active": true },
  { "id": "perm_create_task", "code": "CREATE_TASK", "name": "Create Task", "module": "TASK", "display_order": 2, "is_active": true },
  { "id": "perm_edit_task", "code": "EDIT_TASK", "name": "Edit Task", "module": "TASK", "display_order": 3, "is_active": true },
  { "id": "perm_move_task", "code": "MOVE_TASK", "name": "Move Task", "module": "TASK", "display_order": 4, "is_active": true },
  { "id": "perm_delete_task", "code": "DELETE_TASK", "name": "Delete Task", "module": "TASK", "display_order": 5, "is_active": true },
  { "id": "perm_manage_users", "code": "MANAGE_USERS", "name": "Manage Users", "module": "ADMIN", "display_order": 6, "is_active": true },
  { "id": "perm_manage_org", "code": "MANAGE_ORG", "name": "Manage Org", "module": "ADMIN", "display_order": 7, "is_active": true },
  { "id": "perm_manage_roles", "code": "MANAGE_ROLES", "name": "Manage Roles", "module": "ADMIN", "display_order": 8, "is_active": true },
  { "id": "perm_manage_states", "code": "MANAGE_STATES", "name": "Manage States", "module": "WORKFLOW", "display_order": 9, "is_active": true },
  { "id": "perm_view_audit_log", "code": "VIEW_AUDIT_LOG", "name": "View Audit Log", "module": "ADMIN", "display_order": 10, "is_active": true }
]
```

### 30.4. `role_permissions`

```json
[
  { "role_id": "role_super_admin", "permission_id": "perm_view_board" },
  { "role_id": "role_super_admin", "permission_id": "perm_create_task" },
  { "role_id": "role_super_admin", "permission_id": "perm_edit_task" },
  { "role_id": "role_super_admin", "permission_id": "perm_move_task" },
  { "role_id": "role_super_admin", "permission_id": "perm_delete_task" },
  { "role_id": "role_super_admin", "permission_id": "perm_manage_users" },
  { "role_id": "role_super_admin", "permission_id": "perm_manage_org" },
  { "role_id": "role_super_admin", "permission_id": "perm_manage_roles" },
  { "role_id": "role_super_admin", "permission_id": "perm_manage_states" },
  { "role_id": "role_super_admin", "permission_id": "perm_view_audit_log" },

  { "role_id": "role_manager", "permission_id": "perm_view_board" },
  { "role_id": "role_manager", "permission_id": "perm_create_task" },
  { "role_id": "role_manager", "permission_id": "perm_edit_task" },
  { "role_id": "role_manager", "permission_id": "perm_move_task" },
  { "role_id": "role_manager", "permission_id": "perm_delete_task" },

  { "role_id": "role_member", "permission_id": "perm_view_board" },
  { "role_id": "role_member", "permission_id": "perm_create_task" },
  { "role_id": "role_member", "permission_id": "perm_edit_task" },
  { "role_id": "role_member", "permission_id": "perm_move_task" },

  { "role_id": "role_intern", "permission_id": "perm_view_board" },
  { "role_id": "role_intern", "permission_id": "perm_create_task" },

  { "role_id": "role_viewer", "permission_id": "perm_view_board" }
]
```

### 30.5. `blocks`

```json
[
  {
    "id": "blk_db",
    "code": "DB",
    "name": "Digital Banking",
    "display_order": 1,
    "is_active": true,
    "description": "Khoi Digital Banking"
  }
]
```

### 30.6. `departments`

```json
[
  {
    "id": "dep_web",
    "block_id": "blk_db",
    "code": "WEB",
    "name": "Website",
    "display_order": 1,
    "is_active": true
  }
]
```

### 30.7. `teams`

```json
[
  {
    "id": "team_po_001",
    "department_id": "dep_web",
    "code": "PO",
    "name": "Product Owner",
    "leader_user_id": "U0001",
    "display_order": 1,
    "is_active": true
  },
  {
    "id": "team_dev_001",
    "department_id": "dep_web",
    "code": "DEV",
    "name": "Developer",
    "leader_user_id": "U0007",
    "display_order": 2,
    "is_active": true
  }
]
```

### 30.8. `states`

```json
[
  { "id": "st_backlog", "code": "BACKLOG", "label": "Backlog", "short_label": "Backlog", "display_order": 1, "is_active": true },
  { "id": "st_in_progress", "code": "IN_PROGRESS", "label": "In Progress", "short_label": "Doing", "display_order": 2, "is_active": true },
  { "id": "st_in_review", "code": "IN_REVIEW", "label": "In Review", "short_label": "Review", "display_order": 3, "is_active": true },
  { "id": "st_done", "code": "DONE", "label": "Done", "short_label": "Done", "display_order": 4, "is_active": true },
  { "id": "st_pending", "code": "PENDING", "label": "Pending", "short_label": "Pending", "display_order": 5, "is_active": true },
  { "id": "st_cancel", "code": "CANCEL", "label": "Cancel", "short_label": "Cancel", "display_order": 6, "is_active": true }
]
```

### 30.9. `users`

Giữ `users.id` dạng text để đi nhanh với dữ liệu mẫu hiện tại.

```json
[
  { "id": "U0001", "username": "an.nguyen", "email": "an.nguyen@vib.com.vn", "full_name": "a.AN", "job_title_id": "job_head", "color": "#2563EB", "sort_order": 1, "is_active": true },
  { "id": "U0002", "username": "hieu.phan", "email": "hieu.phan@vib.com.vn", "full_name": "Hiếu", "job_title_id": "job_po", "color": "#DB2777", "sort_order": 2, "is_active": true },
  { "id": "U0003", "username": "dinh.tran", "email": "dinh.tran@vib.com.vn", "full_name": "Định", "job_title_id": "job_ux", "color": "#16A34A", "sort_order": 3, "is_active": true },
  { "id": "U0007", "username": "tuan.le2", "email": "tuan.le2@vib.com.vn", "full_name": "Đắc", "job_title_id": "job_dev_manager", "color": "#B45309", "sort_order": 7, "is_active": true },
  { "id": "U0009", "username": "tan", "email": "tan@vib.com.vn", "full_name": "Tấn", "job_title_id": "job_be", "color": "#DC2626", "sort_order": 9, "is_active": true },
  { "id": "U0010", "username": "thinh", "email": "thinh@vib.com.vn", "full_name": "Thịnh", "job_title_id": "job_fe", "color": "#0284C7", "sort_order": 10, "is_active": true }
]
```

### 30.10. `user_teams`

```json
[
  { "id": "ut_001", "user_id": "U0001", "team_id": "team_po_001", "is_primary": true, "is_active": true },
  { "id": "ut_002", "user_id": "U0002", "team_id": "team_po_001", "is_primary": true, "is_active": true },
  { "id": "ut_003", "user_id": "U0003", "team_id": "team_po_001", "is_primary": true, "is_active": true },
  { "id": "ut_004", "user_id": "U0007", "team_id": "team_dev_001", "is_primary": true, "is_active": true },
  { "id": "ut_005", "user_id": "U0009", "team_id": "team_dev_001", "is_primary": true, "is_active": true },
  { "id": "ut_006", "user_id": "U0010", "team_id": "team_dev_001", "is_primary": true, "is_active": true }
]
```

### 30.11. `user_role_assignments`

Validation quan trọng:

- không cho gán trùng cùng `user + role + scope + team`
- khi disable user, không xóa lịch sử mà chuyển `user_role_assignments.is_active = false`

```json
[
  {
    "id": "ura_001",
    "user_id": "U0001",
    "role_id": "role_super_admin",
    "scope_type": "GLOBAL",
    "team_id": null,
    "legacy_team_code": null,
    "is_active": true
  },
  {
    "id": "ura_002",
    "user_id": "U0001",
    "role_id": "role_manager",
    "scope_type": "TEAM",
    "team_id": "team_po_001",
    "legacy_team_code": "DB.WEB.PO",
    "is_active": true
  },
  {
    "id": "ura_003",
    "user_id": "U0007",
    "role_id": "role_manager",
    "scope_type": "TEAM",
    "team_id": "team_dev_001",
    "legacy_team_code": "DB.WEB.DEV",
    "is_active": true
  },
  {
    "id": "ura_004",
    "user_id": "U0002",
    "role_id": "role_member",
    "scope_type": "TEAM",
    "team_id": "team_po_001",
    "legacy_team_code": "DB.WEB.PO",
    "is_active": true
  }
]
```

## 31. Schema JSON/API response cho màn `Người dùng`

Mục tiêu của màn này:

- hiển thị danh sách user
- lọc theo phòng ban, nhóm, chức danh, trạng thái
- xem nhanh team chính, team phụ, role hiện tại
- thao tác active/inactive user

### 31.1. `GET /admin/users`

Query gợi ý:

```text
/admin/users?keyword=an&block_id=blk_db&department_id=dep_web&team_id=team_po_001&job_title_id=job_po&is_active=true&page=1&page_size=20
```

Response:

```json
{
  "data": [
    {
      "id": "U0001",
      "username": "an.nguyen",
      "email": "an.nguyen@vib.com.vn",
      "full_name": "a.AN",
      "job_title": {
        "id": "job_head",
        "code": "HEAD",
        "name": "Head"
      },
      "color": "#2563EB",
      "sort_order": 1,
      "is_active": true,
      "primary_team": {
        "id": "team_po_001",
        "code": "PO",
        "name": "Product Owner",
        "full_code": "DB.WEB.PO",
        "department": {
          "id": "dep_web",
          "code": "WEB",
          "name": "Website"
        },
        "block": {
          "id": "blk_db",
          "code": "DB",
          "name": "Digital Banking"
        }
      },
      "teams": [
        {
          "id": "team_po_001",
          "code": "PO",
          "name": "Product Owner",
          "full_code": "DB.WEB.PO",
          "is_primary": true
        }
      ],
      "roles": [
        {
          "assignment_id": "ura_001",
          "role_code": "SUPER_ADMIN",
          "role_name": "Super Admin",
          "scope_type": "GLOBAL",
          "team_id": null,
          "team_code": null
        },
        {
          "assignment_id": "ura_002",
          "role_code": "MANAGER",
          "role_name": "Manager",
          "scope_type": "TEAM",
          "team_id": "team_po_001",
          "team_code": "DB.WEB.PO"
        }
      ],
      "created_at": "2026-04-19T00:00:00Z",
      "updated_at": "2026-04-19T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 1,
    "total_pages": 1
  },
  "filters": {
    "keyword": "an",
    "block_id": "blk_db",
    "department_id": null,
    "team_id": null,
    "job_title_id": null,
    "is_active": true
  }
}
```

### 31.2. `GET /admin/users/:id`

Response:

```json
{
  "data": {
    "id": "U0001",
    "username": "an.nguyen",
    "email": "an.nguyen@vib.com.vn",
    "full_name": "a.AN",
    "job_title": {
      "id": "job_head",
      "code": "HEAD",
      "name": "Head"
    },
    "color": "#2563EB",
    "sort_order": 1,
    "is_active": true,
    "last_login_at": "2026-04-19T08:30:00Z",
    "teams": [
      {
        "assignment_id": "ut_001",
        "id": "team_po_001",
        "code": "PO",
        "name": "Product Owner",
        "full_code": "DB.WEB.PO",
        "is_primary": true,
        "department": {
          "id": "dep_web",
          "code": "WEB",
          "name": "Website"
        },
        "block": {
          "id": "blk_db",
          "code": "DB",
          "name": "Digital Banking"
        }
      }
    ],
    "role_assignments": [
      {
        "assignment_id": "ura_001",
        "role": {
          "id": "role_super_admin",
          "code": "SUPER_ADMIN",
          "name": "Super Admin"
        },
        "scope_type": "GLOBAL",
        "team": null,
        "is_active": true
      },
      {
        "assignment_id": "ura_002",
        "role": {
          "id": "role_manager",
          "code": "MANAGER",
          "name": "Manager"
        },
        "scope_type": "TEAM",
        "team": {
          "id": "team_po_001",
          "code": "PO",
          "name": "Product Owner",
          "full_code": "DB.WEB.PO"
        },
        "is_active": true
      }
    ],
    "audit": {
      "created_at": "2026-04-19T00:00:00Z",
      "created_by": "U0001",
      "updated_at": "2026-04-19T00:00:00Z",
      "updated_by": "U0001"
    }
  }
}
```

### 31.3. `POST /admin/users`

Request:

```json
{
  "id": "U0015",
  "username": "linh.nguyen",
  "email": "linh.nguyen@vib.com.vn",
  "full_name": "Linh",
  "job_title_id": "job_po",
  "color": "#0EA5E9",
  "sort_order": 15,
  "is_active": true,
  "team_assignments": [
    {
      "team_id": "team_po_001",
      "is_primary": true
    }
  ]
}
```

### 31.4. `PATCH /admin/users/:id/status`

Request:

```json
{
  "is_active": false,
  "reason": "User nghỉ việc"
}
```

Quy tắc nghiệp vụ:

- khi `is_active = false`, user bị ẩn khỏi danh sách assign active
- giữ lịch sử `user_role_assignments`
- cập nhật `user_role_assignments.is_active = false` cho các assignment còn hiệu lực

Response:

```json
{
  "message": "User status updated successfully",
  "data": {
    "id": "U0002",
    "is_active": false
  }
}
```

## 32. Schema JSON/API response cho màn `Vai trò & quyền`

Mục tiêu của màn này:

- hiển thị danh sách role
- xem permission của từng role
- chỉnh permission cho role
- xem user nào đang mang role nào theo team

### 32.1. `GET /admin/roles`

Response:

```json
{
  "data": [
    {
      "id": "role_super_admin",
      "code": "SUPER_ADMIN",
      "name": "Super Admin",
      "description": "Toan quyen quan tri he thong",
      "scope_type": "GLOBAL",
      "display_order": 1,
      "is_system": true,
      "is_active": true,
      "permission_count": 10,
      "user_assignment_count": 1
    },
    {
      "id": "role_manager",
      "code": "MANAGER",
      "name": "Manager",
      "description": "Quan ly task va thanh vien trong nhom",
      "scope_type": "TEAM",
      "display_order": 2,
      "is_system": true,
      "is_active": true,
      "permission_count": 5,
      "user_assignment_count": 2
    }
  ]
}
```

### 32.2. `GET /admin/roles/:id`

Response:

```json
{
  "data": {
    "id": "role_manager",
    "code": "MANAGER",
    "name": "Manager",
    "description": "Quan ly task va thanh vien trong nhom",
    "scope_type": "TEAM",
    "display_order": 2,
    "is_system": true,
    "is_active": true,
    "permissions": [
      { "id": "perm_view_board", "code": "VIEW_BOARD", "name": "View Board", "module": "TASK" },
      { "id": "perm_create_task", "code": "CREATE_TASK", "name": "Create Task", "module": "TASK" },
      { "id": "perm_edit_task", "code": "EDIT_TASK", "name": "Edit Task", "module": "TASK" },
      { "id": "perm_move_task", "code": "MOVE_TASK", "name": "Move Task", "module": "TASK" },
      { "id": "perm_delete_task", "code": "DELETE_TASK", "name": "Delete Task", "module": "TASK" }
    ],
    "user_assignments": [
      {
        "assignment_id": "ura_002",
        "user": {
          "id": "U0001",
          "username": "an.nguyen",
          "full_name": "a.AN"
        },
        "scope_type": "TEAM",
        "team": {
          "id": "team_po_001",
          "code": "PO",
          "name": "Product Owner",
          "full_code": "DB.WEB.PO"
        },
        "is_active": true
      }
    ]
  }
}
```

### 32.3. `GET /admin/permissions`

Response:

```json
{
  "data": [
    { "id": "perm_view_board", "code": "VIEW_BOARD", "name": "View Board", "module": "TASK", "is_active": true },
    { "id": "perm_create_task", "code": "CREATE_TASK", "name": "Create Task", "module": "TASK", "is_active": true },
    { "id": "perm_manage_users", "code": "MANAGE_USERS", "name": "Manage Users", "module": "ADMIN", "is_active": true }
  ]
}
```

### 32.4. `PUT /admin/roles/:id/permissions`

Request:

```json
{
  "permission_ids": [
    "perm_view_board",
    "perm_create_task",
    "perm_edit_task",
    "perm_move_task",
    "perm_delete_task"
  ]
}
```

Response:

```json
{
  "message": "Role permissions updated successfully",
  "data": {
    "role_id": "role_manager",
    "permission_ids": [
      "perm_view_board",
      "perm_create_task",
      "perm_edit_task",
      "perm_move_task",
      "perm_delete_task"
    ]
  }
}
```

### 32.5. `POST /admin/user-role-assignments`

Request:

```json
{
  "user_id": "U0002",
  "role_id": "role_member",
  "scope_type": "TEAM",
  "team_id": "team_po_001",
  "legacy_team_code": "DB.WEB.PO",
  "is_active": true
}
```

Validation:

- không cho trùng `user_id + role_id + scope_type + team_id`
- nếu `scope_type = GLOBAL` thì `team_id` phải là `null`
- nếu `scope_type = TEAM` thì `team_id` là bắt buộc

Response:

```json
{
  "message": "User role assigned successfully",
  "data": {
    "assignment_id": "ura_010",
    "user_id": "U0002",
    "role_id": "role_member",
    "scope_type": "TEAM",
    "team_id": "team_po_001",
    "legacy_team_code": "DB.WEB.PO",
    "is_active": true
  }
}
```

### 32.6. `DELETE /admin/roles/:id`

Quy tắc nghiệp vụ:

- không cho xóa cứng role hệ thống như `SUPER_ADMIN`, `MANAGER`, `MEMBER`
- với role custom, ưu tiên `is_active = false` thay vì hard delete nếu đã phát sinh assignment

Response lỗi mẫu:

```json
{
  "error_code": "SYSTEM_ROLE_DELETE_FORBIDDEN",
  "message": "System role cannot be deleted"
}
```

## 33. Gợi ý validation tổng hợp

- `users.id` có thể giữ dạng text để tương thích dữ liệu mẫu hiện tại.
- `username` và `email` phải unique.
- chỉ một `user_teams.is_primary = true` cho mỗi user đang active.
- không cho trùng `user + role + scope + team` trong `user_role_assignments`.
- không cho xóa cứng role hệ thống.
- khi disable user, chỉ deactivate dữ liệu liên quan thay vì xóa lịch sử.
