# Bài tập lớn - Phát triển ứng dụng với Flutter

## Thông tin sinh viên
- **Họ và tên**: Nguyễn Ngọc Tú
- **MSSV**: 2221050290
- **Lớp**: DCCTCLC67A

## Giới thiệu đề tài

Đây là bài tập lớn của học phần **Phát triển ứng dụng di động đa nền tảng**.  
Ứng dụng được xây dựng bằng **Flutter + Dart**, áp dụng các nội dung đã học:

- Xây dựng giao diện người dùng (UI).
- Quản lý trạng thái với `Provider`.
- Thao tác dữ liệu CRUD.
- Lưu trữ dữ liệu cục bộ bằng `localstore` (JSON NoSQL).
- Gọi API HTTP bằng `http`.
- Kiểm thử tự động (unit test + widget test).
- CI/CD cơ bản với **GitHub Actions**.

Đề tài em chọn:

> **Ứng dụng quản lý chi tiêu cá nhân (Expense Tracker)**  
> Cho phép người dùng ghi lại các khoản thu/chi, xem tổng thu – chi – số dư,
> từ đó quản lý tài chính cá nhân một cách đơn giản.

---

## 1. Mục tiêu

- Luyện tập lập trình UI với Flutter.
- Thiết kế **data model** cho giao dịch chi tiêu (`Expense`).
- Áp dụng **Provider** để quản lý trạng thái toàn ứng dụng.
- Thực hiện đầy đủ các thao tác **CRUD**.
- Lưu trữ dữ liệu cục bộ bằng **localstore** để app chạy được offline.
- Viết một số bài **kiểm thử tự động**.
- Thiết lập **GitHub Actions** để chạy test khi có thay đổi mã nguồn.

---

## 2. Mô tả ứng dụng

### 2.1. Đối tượng `Expense` (giao dịch)

Thuộc tính:

- `id` (String): định danh duy nhất.
- `title` (String): tiêu đề / mô tả ngắn (Ăn trưa, Tiền nhà, ...).
- `amount` (double): số tiền.
- `category` (String): danh mục (Ăn uống, Học tập, Giải trí, ...).
- `date` (DateTime): ngày phát sinh giao dịch.
- `isIncome` (bool): `true` = khoản thu, `false` = khoản chi.

File cài đặt: `lib/models/expense.dart`

Model hỗ trợ:

- `toJson()` / `Expense.fromJson(Map)` để lưu/đọc JSON.
- `copyWith()` để tạo bản sao với một vài trường thay đổi.

### 2.2. Chức năng CRUD

1. **Create – Thêm giao dịch mới**
   - Nhấn nút **+ Thêm** (FloatingActionButton).
   - Điền tiêu đề, số tiền, danh mục, ngày, chọn loại Thu/Chi.
   - Bấm **Lưu**.

2. **Read – Xem danh sách giao dịch**
   - Màn hình chính hiển thị danh sách các giao dịch.
   - Có ô **tìm kiếm** theo tiêu đề hoặc danh mục.
   - Có thanh **tóm tắt** trên đầu: Tổng Thu, Tổng Chi, Còn lại.

3. **Update – Sửa giao dịch**
   - Chạm vào một item trong danh sách.
   - Màn hình sửa cho phép chỉnh lại thông tin và lưu.

4. **Delete – Xóa giao dịch**
   - Icon thùng rác ở từng item.
   - Có `AlertDialog` hỏi xác nhận trước khi xóa.

---

## 3. Giao diện người dùng (UI)

### 3.1. Màn hình chính `HomeScreen`

File: `lib/screens/home_screen.dart`

Thành phần:

- **AppBar**: tiêu đề *“Quản lý chi tiêu”*, căn giữa, có nút cài đặt.
- **Thanh tóm tắt** (thu/chi/còn lại) dạng 3 cột:
  - Sử dụng `Row` + `Expanded`.
  - Màu sắc trực quan: Thu (xanh), Chi (đỏ), Còn lại (xanh/đỏ tùy số dư).
- **Thanh tìm kiếm**: `TextField` với icon search.
- **Danh sách giao dịch**: `ListView.builder` hiển thị `ExpenseItem`.
- **FloatingActionButton.extended**: icon + chữ “Thêm” → mở màn hình tạo mới.

### 3.2. Màn hình thêm / sửa `ExpenseEditScreen`

File: `lib/screens/expense_edit_screen.dart`

- Form gồm:
  - TextFormField: Tiêu đề (bắt buộc).
  - TextFormField: Số tiền (bắt buộc, > 0).
  - TextFormField: Danh mục.
  - DatePicker: chọn ngày.
  - Switch: chọn khoản thu hay chi.
- Form bọc trong `SingleChildScrollView` + `SafeArea`
  → giúp giao diện không bị tràn khi mở bàn phím.

### 3.3. Màn hình cài đặt & API `SettingsScreen`

File: `lib/screens/settings_screen.dart`

- Nút **“Lấy tỷ giá USD/VND”**:
  - Gọi API `https://open.er-api.com/v6/latest/USD`.
  - Lấy trường `rates["VND"]` và hiển thị.
- Thông báo lỗi nếu gọi API thất bại.

### 3.4. Widget `ExpenseItem`

File: `lib/widgets/expense_item.dart`

- Dùng `Card` bo góc, padding hợp lý.
- Icon tròn:
  - Màu xanh nếu là khoản thu.
  - Màu đỏ nếu là khoản chi.
- Bên phải hiển thị số tiền (có + / - và đơn vị ₫).
- Dưới số tiền là nút xóa nhỏ, tránh tràn layout.
- Sửa lỗi **BOTTOM OVERFLOWED** bằng `SizedBox` và chỉnh `constraints`.

---

## 4. Quản lý trạng thái với Provider

File: `lib/providers/expense_provider.dart`

- Biến:
  - `_expenses` – danh sách giao dịch.
  - `_isLoading` – trạng thái đang tải.
  - `_errorMessage` – thông báo lỗi (nếu có).
- Hàm:
  - `loadExpenses()` – đọc dữ liệu từ localstore.
  - `addExpense(...)` – thêm giao dịch.
  - `updateExpense(...)` – cập nhật.
  - `deleteExpense(...)` – xóa.
- Tính toán:
  - `totalIncome`, `totalExpense`, `balance`.

Trong `main.dart`, Provider được inject:

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => ExpenseProvider()),
  ],
  child: MaterialApp(
    debugShowCheckedModeBanner: false,
    home: const HomeScreen(),
  ),
);
```

---

## 5. Lưu trữ dữ liệu với localstore

File: `lib/services/expense_service.dart`

- Dùng `Localstore.instance`.
- Collection: `'expenses'`.
- Hàm:
  - `saveExpense(Expense expense)`
  - `getExpenses()`
  - `deleteExpense(String id)`
- Tất cả đều bọc trong `try/catch`, nếu lỗi ném `Exception` để Provider xử lý.

---

## 6. Gọi API HTTP

File: `lib/screens/settings_screen.dart`

- Thư viện: `http`.
- API sử dụng: `https://open.er-api.com/v6/latest/USD`
- Quy trình:
  1. Gọi `http.get(uri)`.
  2. Nếu status 200:
     - Parse JSON, lấy `rates['VND']`.
  3. Nếu lỗi:
     - Hiển thị thông báo phù hợp (mã lỗi hoặc exception).

---

## 7. Kiểm thử tự động

Thư mục `test/`:

- `test/main_test.dart`: Unit test cho model `Expense`.
- `test/widget_test.dart`: Widget test cho `MainApp`.

Chạy test:

```bash
flutter test
```

---

## 8. CI/CD với GitHub Actions

Thư mục: `.github/workflows/ci.yml`

Workflow:

- Checkout code.
- Cài Flutter (kênh stable).
- Chạy `flutter pub get`.
- Chạy `flutter test`.

Mỗi lần push lên `main` hoặc mở pull request → GitHub Actions sẽ tự động chạy test.  
Nếu tất cả pass → trạng thái **Success**.

---

## 9. Hướng dẫn cài đặt và chạy

### 9.1. Clone dự án

```bash
git clone <link_repo>
cd flutter-final-project-tubeo24-main
```

### 9.2. Cài dependencies

```bash
flutter pub get
```

### 9.3. Chạy ứng dụng

```bash
flutter run
```

### 9.4. Chạy test

```bash
flutter test
```

---

## 10. Công nghệ & thư viện sử dụng

- **Flutter** – xây dựng UI đa nền tảng.
- **Dart** – ngôn ngữ lập trình.
- **provider** – quản lý state.
- **localstore** – lưu trữ dữ liệu cục bộ dạng JSON NoSQL.
- **http** – gọi REST API.
- **uuid** – sinh id ngẫu nhiên cho giao dịch.
- **flutter_test** – viết unit test & widget test.
- **GitHub Actions** – CI chạy `flutter test`.

---

## 11. Tự đánh giá

Theo thang điểm trong đề bài:

- 5/10 – Build thành công, test chạy được.
- 6/10 – CRUD cơ bản cho đối tượng `Expense`.
- 7/10 – Có quản lý trạng thái, UI rõ ràng, phản hồi lỗi.
- 8/10 – Có lưu trữ cục bộ, có gọi API, có test & CI.
- 9/10 – UI tương đối đẹp, có tóm tắt thu/chi/số dư, code cấu trúc rõ.

> **Tự đánh giá : 8.5 / 10**  
---

## 12. Hướng phát triển thêm

- Thống kê theo tháng bằng biểu đồ (pie chart, bar chart).
- Lọc giao dịch theo khoảng thời gian.
- Thêm nhiều đơn vị tiền tệ, hỗ trợ đa ngôn ngữ.
- Đồng bộ dữ liệu với backend (Firebase / API riêng).
- Thêm chức năng đăng nhập, nhiều người dùng.

