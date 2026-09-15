# Thực Hành Vẽ Biểu Đồ Tuần Tự Chức Năng Đăng Nhập RikkeiShop

## 1. Phân tích kịch bản

### Các thành phần tham gia

| Thành phần  | Loại   | Vai trò                                     |
| ----------- | ------ | ------------------------------------------- |
| Khách hàng  | Actor  | Nhập thông tin và nhận kết quả đăng nhập    |
| Màn hình UI | Object | Tiếp nhận thông tin và gửi yêu cầu xác thực |
| AuthServer  | Object | Xử lý và kiểm tra thông tin đăng nhập       |

---

## 2. Phân tích các thông điệp

### Bước 1: Khách hàng nhập thông tin đăng nhập

**Thông điệp:**

`nhapThongTin(username, password)`

**Từ:** Khách hàng
**Đến:** Màn hình UI
**Loại:** `Sync Message`

**Giải thích:** Khách hàng gửi username và password lên Màn hình UI để thực hiện đăng nhập.

---

### Bước 2: Màn hình UI yêu cầu xác thực tài khoản

**Thông điệp:**

`verifyAccount()`

**Từ:** Màn hình UI
**Đến:** AuthServer
**Loại:** `Sync Message`

**Giải thích:** Màn hình UI gửi yêu cầu xác thực đến AuthServer và **chờ kết quả trả về**, vì vậy đây là thông điệp đồng bộ (Sync).

---

### Bước 3: AuthServer tự kiểm tra thông tin đăng nhập

**Thông điệp:**

`checkCredentials()`

**Từ:** AuthServer
**Đến:** AuthServer
**Loại:** `Self Message`

**Giải thích:** Việc kiểm tra username và password là xử lý nội bộ của AuthServer. Không có đối tượng hoặc Database nào khác tham gia, vì vậy AuthServer tự gọi chính nó để thực hiện `checkCredentials()`.

---

### Bước 4: Rẽ nhánh theo kết quả xác thực

Sau khi AuthServer kiểm tra xong, sử dụng **Combined Fragment `alt`** để biểu diễn hai trường hợp.

#### Nhánh 1: [Thông tin hợp lệ]

AuthServer trả về Token thành công cho Màn hình UI.

**Thông điệp:**

`Token thành công`

**Từ:** AuthServer
**Đến:** Màn hình UI
**Loại:** `Return Message`

Sau đó Màn hình UI trả phản hồi cho Khách hàng:

`Hiển thị Trang chủ`

**Từ:** Màn hình UI
**Đến:** Khách hàng
**Loại:** `Return Message`

---

#### Nhánh 2: [Thông tin không hợp lệ]

AuthServer trả về lỗi:

`Sai mật khẩu`

**Từ:** AuthServer
**Đến:** Màn hình UI
**Loại:** `Return Message`

Sau đó Màn hình UI trả phản hồi cho Khách hàng:

`Hiển thị cảnh báo lỗi`

**Từ:** Màn hình UI
**Đến:** Khách hàng
**Loại:** `Return Message`

---

## 3. Tổng hợp Sequence Flow

```text
Khách hàng
    |
    | 1. nhapThongTin(username, password)
    |    [Sync]
    ↓
Màn hình UI
    |
    | 2. verifyAccount()
    |    [Sync]
    ↓
AuthServer
    |
    | 3. checkCredentials()
    |    [Self]
    ↻
    |
    | 4. alt
    |
    |── [Thông tin hợp lệ]
    |       |
    |       |── Return: Token thành công
    |       ↓
    |    Màn hình UI
    |       |
    |       |── Return: Hiển thị Trang chủ
    |       ↓
    |    Khách hàng
    |
    |── [Thông tin không hợp lệ]
            |
            |── Return: "Sai mật khẩu"
            ↓
         Màn hình UI
            |
            |── Return: Hiển thị cảnh báo lỗi
            ↓
         Khách hàng
```

---

## 4. Bảng tổng hợp các thông điệp

| Bước | Từ          | Đến         | Thông điệp                         | Loại       |
| ---: | ----------- | ----------- | ---------------------------------- | ---------- |
|    1 | Khách hàng  | Màn hình UI | `nhapThongTin(username, password)` | **Sync**   |
|    2 | Màn hình UI | AuthServer  | `verifyAccount()`                  | **Sync**   |
|    3 | AuthServer  | AuthServer  | `checkCredentials()`               | **Self**   |
|   4a | AuthServer  | Màn hình UI | `Token thành công`                 | **Return** |
|   5a | Màn hình UI | Khách hàng  | `Hiển thị Trang chủ`               | **Return** |
|   4b | AuthServer  | Màn hình UI | `"Sai mật khẩu"`                   | **Return** |
|   5b | Màn hình UI | Khách hàng  | `Hiển thị cảnh báo lỗi`            | **Return** |

---

## 5. Khối rẽ nhánh `alt`

Khối `alt` được đặt **ngay sau khi AuthServer hoàn thành `checkCredentials()`**.

```text
┌─────────────────────────────────────────────┐
│ alt                                         │
│                                             │
│ [Thông tin hợp lệ]                         │
│                                             │
│ AuthServer ────────> Màn hình UI            │
│             Return: Token thành công        │
│                                             │
│ Màn hình UI ───────> Khách hàng             │
│             Return: Hiển thị Trang chủ      │
│                                             │
├─────────────────────────────────────────────┤
│ [Thông tin không hợp lệ]                    │
│                                             │
│ AuthServer ────────> Màn hình UI            │
│             Return: "Sai mật khẩu"          │
│                                             │
│ Màn hình UI ───────> Khách hàng             │
│             Return: Hiển thị cảnh báo lỗi   │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 6. Kết luận

Sequence Diagram của chức năng **Đăng nhập RikkeiShop** gồm 3 participant:

**Khách hàng → Màn hình UI → AuthServer**

Các loại thông điệp được sử dụng:

* **Sync:** `nhapThongTin()` và `verifyAccount()`
* **Self:** `checkCredentials()`
* **Return:** Token thành công, `"Sai mật khẩu"`, hiển thị Trang chủ và hiển thị cảnh báo lỗi
* **alt:** Phân biệt hai trường hợp **[Thông tin hợp lệ]** và **[Thông tin không hợp lệ]**

Sơ đồ thể hiện đúng trình tự tương tác từ trên xuống dưới và phản ánh đầy đủ luồng đăng nhập thành công hoặc thất bại của hệ thống RikkeiShop.
