# SRS – Module Đăng nhập (Login)

**Hệ thống:** ANHTESTER LMS – https://anhtester.com
**Phiên bản tài liệu:** 1.0 (Draft)
**Ngày khảo sát:** 2026-10-30
**Phương pháp:** Khảo sát giao diện trang `/login` và `/forgot-password` ở góc nhìn người dùng + đọc thuộc tính DOM của form (không gửi request đăng nhập).

> **Quy ước:** Mục đánh dấu **[Quan sát]** là thông tin xác nhận trực tiếp trên giao diện/DOM. Mục đánh dấu **[Giả định]** là suy luận hoặc hành vi thông thường chưa kiểm chứng, cần xác nhận với BA/Dev trước khi dùng làm căn cứ test.

---

## 1. Giới thiệu

### 1.1 Mục đích
Mô tả yêu cầu chức năng và phi chức năng của module Đăng nhập trên nền tảng học Test Automation ANHTESTER LMS, làm cơ sở cho việc thiết kế test case và kiểm thử.

### 1.2 Phạm vi
Bao gồm:
- Đăng nhập bằng Email + Mật khẩu.
- Tuỳ chọn "Ghi nhớ đăng nhập".
- Xác thực reCAPTCHA.
- Đăng nhập qua Google, GitHub (OAuth).
- Liên kết sang Quên mật khẩu và Đăng ký.

Không bao gồm (chỉ nêu điểm giao tiếp): chức năng Đăng ký, nội dung chi tiết luồng Quên mật khẩu, trang sau đăng nhập.

### 1.3 Định nghĩa & viết tắt
| Thuật ngữ | Ý nghĩa |
|---|---|
| LMS | Learning Management System |
| SRS | Software Requirements Specification |
| OAuth | Giao thức đăng nhập qua bên thứ ba (Google, GitHub) |
| CSRF | Cross-Site Request Forgery |
| FR / NFR | Functional / Non-Functional Requirement |

### 1.4 Đối tượng sử dụng
| Vai trò | Mô tả |
|---|---|
| Học viên | Người đã có tài khoản, đăng nhập để học khoá học đã đăng ký |
| Khách (Guest) | Chưa đăng nhập; có thể xem trang chủ, danh sách khoá học |
| Giảng viên / Admin | [Giả định] Dùng chung form đăng nhập; chưa xác nhận được cơ chế phân quyền |

---

## 2. Mô tả tổng quan

### 2.1 Vị trí trong hệ thống
- Truy cập từ nút **"Đăng nhập"** ở header trang chủ (`https://anhtester.com/`) → `/login`.
- **[Quan sát]** Form gửi `POST` tới `https://anhtester.com/custom-login`.
- **[Quan sát]** Form có hidden field `_token` (dấu hiệu CSRF token, framework dạng Laravel).

### 2.2 Giao diện trang đăng nhập [Quan sát]
Bố cục 2 cột:
- **Cột trái:** Hình minh hoạ + tiêu đề "Welcome to ANHTESTER LMS Courses." + slogan "Nền tảng học Test Automation chuyên nghiệp, ứng dụng AI trong Testing."
- **Cột phải:** Logo, nút "Về trang chủ", tiêu đề "Đăng nhập tài khoản", form đăng nhập, đăng nhập mạng xã hội, liên kết đăng ký.
- Có nút **chuyển giao diện Sáng / Tối** và nút **Lên đầu trang**.
- Có popup/dialog **"Xác nhận"** (nút "Huỷ", "Xác nhận") ẩn trong DOM, chưa rõ điều kiện hiển thị.

### 2.3 Ràng buộc & giả định chung
- Trình duyệt phải bật JavaScript và cho phép tải reCAPTCHA của Google.
- Giao diện tiếng Việt (có chọn ngôn ngữ "VIE" ở trang chủ).
- Phía người dùng, validation cơ bản dựa trên HTML5 native của trình duyệt (form không có `novalidate`).

---

## 3. Danh sách thành phần giao diện (UI Elements)

| # | Thành phần | Loại | `name` | Bắt buộc | Ghi chú [Quan sát] |
|---|---|---|---|---|---|
| 1 | Email | Textbox `type=email` | `email` | Có (`required`, dấu *) | Không có placeholder, `maxlength`, `pattern`; có icon phong bì |
| 2 | Mật khẩu | Textbox `type=password` | `password` | Có (`required`, dấu *) | Không có `maxlength`/`minlength`; có icon mắt (hiện/ẩn mật khẩu) |
| 3 | Ghi nhớ đăng nhập | Checkbox | `remember` | Không | Mặc định không chọn |
| 4 | Quên mật khẩu? | Link | – | – | → `/forgot-password` |
| 5 | reCAPTCHA | Widget "Tôi không phải là người máy" | – | Có [Giả định] | reCAPTCHA v2 (checkbox) |
| 6 | Đăng nhập | Button `submit` | – | – | Gửi form |
| 7 | Google | Link/Button | – | – | → `/auth/google/redirect` |
| 8 | GitHub | Link/Button | – | – | → `/auth/github/redirect` |
| 9 | Đăng ký | Link | – | – | → `/register` |
| 10 | Về trang chủ | Link | – | – | → `/index` |
| 11 | Chuyển giao diện Sáng/Tối | Button | – | – | Đổi theme |

---

## 4. Yêu cầu chức năng (Functional Requirements)

### FR-LOGIN-01 – Hiển thị trang đăng nhập
- Hệ thống hiển thị trang `/login` đầy đủ các thành phần ở mục 3.
- Trường Email và Mật khẩu để trống khi mở trang.
- Mật khẩu mặc định ở dạng che (dấu chấm).

### FR-LOGIN-02 – Đăng nhập bằng Email và Mật khẩu
- **Tiền điều kiện:** Người dùng có tài khoản hợp lệ, chưa đăng nhập.
- **Luồng chính:**
  1. Người dùng nhập Email và Mật khẩu.
  2. Tích reCAPTCHA "Tôi không phải là người máy".
  3. Nhấn **Đăng nhập**.
  4. Hệ thống kiểm tra thông tin; nếu đúng, tạo phiên đăng nhập và chuyển hướng khỏi trang login.
- **Hậu điều kiện:** Người dùng ở trạng thái đã đăng nhập; header hiển thị trạng thái tài khoản thay cho nút "Đăng nhập/Đăng ký" [Giả định].
- **Luồng thay thế / ngoại lệ:** xem FR-LOGIN-03, 04, 05.
- **Chưa xác định:** trang đích sau đăng nhập (trang chủ, dashboard hay trang trước đó).

### FR-LOGIN-03 – Kiểm tra dữ liệu đầu vào (client-side)
| Tình huống | Kết quả mong đợi |
|---|---|
| Để trống Email, nhấn Đăng nhập | Không gửi form; trường Email được tô viền đỏ, tooltip **"Vui lòng điền vào trường này."** **[Quan sát]** |
| Để trống Mật khẩu | Tương tự trên, tại trường Mật khẩu [Giả định theo `required`] |
| Email sai định dạng (thiếu `@`, thiếu domain…) | Trình duyệt chặn gửi và báo lỗi định dạng email (HTML5 `type=email`) [Giả định] |
| Email có khoảng trắng đầu/cuối | Chưa xác định: tự cắt hay báo lỗi |
| Chưa tích reCAPTCHA | Không cho đăng nhập / báo lỗi reCAPTCHA [Giả định, chưa xác nhận thông điệp] |

> Lưu ý: thông báo native phụ thuộc trình duyệt và ngôn ngữ hệ điều hành; test case nên ghi rõ trình duyệt.

### FR-LOGIN-04 – Xử lý thông tin đăng nhập sai (server-side)
Chưa gửi được request thật nên **chưa quan sát** các thông báo sau; cần xác nhận khi test:
| Tình huống | Kết quả mong đợi [Giả định] |
|---|---|
| Email chưa đăng ký | Báo lỗi chung, ở lại trang login, giữ lại Email đã nhập |
| Email đúng, mật khẩu sai | Báo lỗi chung, ở lại trang login, xoá Mật khẩu |
| Tài khoản bị khoá / chưa kích hoạt | Báo lỗi tương ứng |
| Lỗi hệ thống / mất kết nối | Hiển thị thông báo lỗi thân thiện, không lộ thông tin kỹ thuật |

Đề xuất: thông báo lỗi nên dùng chung cho "email không tồn tại" và "mật khẩu sai" để tránh dò tìm tài khoản (user enumeration).

### FR-LOGIN-05 – Xác thực reCAPTCHA
- Form hiển thị reCAPTCHA v2 checkbox (**[Quan sát]**, có iframe `google.com/recaptcha/api2/anchor`).
- Hệ thống chỉ chấp nhận đăng nhập khi reCAPTCHA hợp lệ [Giả định].
- Khi reCAPTCHA hết hạn (token hết hiệu lực sau một thời gian) hoặc lỗi tải, người dùng phải xác thực lại.
- Sau đăng nhập thất bại, reCAPTCHA được reset [Giả định].

### FR-LOGIN-06 – Hiện / ẩn mật khẩu
- Nhấn icon mắt trong ô Mật khẩu để chuyển giữa che / hiện mật khẩu (**[Quan sát]** có icon; hành vi chi tiết [Giả định]).

### FR-LOGIN-07 – Ghi nhớ đăng nhập
- Checkbox "Ghi nhớ đăng nhập" (`remember`), mặc định không chọn.
- Khi chọn và đăng nhập thành công, phiên đăng nhập được duy trì lâu hơn (sau khi đóng và mở lại trình duyệt vẫn đăng nhập) [Giả định theo cơ chế `remember` của Laravel].
- Khi không chọn, phiên kết thúc theo vòng đời session thông thường.
- Click vào nhãn cũng đổi trạng thái checkbox (nhãn gắn với `flexCheckDefault`).

### FR-LOGIN-08 – Đăng nhập bằng Google
- Nhấn **Google** → chuyển tới `/auth/google/redirect` → màn hình chọn tài khoản/đồng ý của Google → quay về hệ thống và đăng nhập.
- Nếu email Google chưa có tài khoản: chưa xác định (tự tạo tài khoản hay báo lỗi).
- Nếu người dùng huỷ ở màn hình Google: quay lại trang login, không đăng nhập.

### FR-LOGIN-09 – Đăng nhập bằng GitHub
- Tương tự FR-LOGIN-08 với `/auth/github/redirect`.
- Tình huống GitHub không trả về email công khai: chưa xác định cách xử lý.

### FR-LOGIN-10 – Quên mật khẩu
- Nhấn **Quên mật khẩu?** → `/forgot-password`.
- **[Quan sát]** Trang này có trường Email (`type=email`), nút **"Gửi liên kết"** và link **"Đăng nhập"** quay về `/login`.
- Chi tiết luồng gửi email, thông báo, thời hạn link: nằm ngoài phạm vi tài liệu này.

### FR-LOGIN-11 – Điều hướng
| Thao tác | Điều hướng |
|---|---|
| Nhấn "Đăng ký" | `/register` |
| Nhấn "Về trang chủ" | `/index` |
| Truy cập `/login` khi đã đăng nhập | Chuyển hướng về trang chính, không hiện lại form [Giả định] |
| Truy cập trang yêu cầu đăng nhập khi chưa đăng nhập | Chuyển về `/login`, sau đăng nhập quay lại trang đích [Giả định] |

### FR-LOGIN-12 – Giao diện Sáng / Tối
- Nút "Chuyển giao diện sáng / tối" đổi theme của trang đăng nhập; tất cả thành phần vẫn đọc được ở cả 2 chế độ.

---

## 5. Yêu cầu phi chức năng (NFR)

| Mã | Nhóm | Yêu cầu |
|---|---|---|
| NFR-01 | Bảo mật | Toàn bộ trang và request đăng nhập dùng HTTPS |
| NFR-02 | Bảo mật | Form có CSRF token (`_token`) **[Quan sát]**; request thiếu/sai token bị từ chối |
| NFR-03 | Bảo mật | Mật khẩu không được hiển thị rõ trong URL, log, hay response |
| NFR-04 | Bảo mật | Chống brute-force: reCAPTCHA **[Quan sát]**; khoá tạm/giới hạn số lần thử [Giả định, cần xác nhận] |
| NFR-05 | Bảo mật | Chống SQL Injection / XSS ở trường Email, Mật khẩu |
| NFR-06 | Bảo mật | Cookie phiên có cờ `HttpOnly`, `Secure`, `SameSite` [Giả định] |
| NFR-07 | Bảo mật | Thông báo lỗi không tiết lộ email có tồn tại hay không |
| NFR-08 | Hiệu năng | Trang login tải trong ≤ 3 giây trên mạng thường; phản hồi đăng nhập ≤ 3 giây [Đề xuất] |
| NFR-09 | Tương thích | Chạy đúng trên Chrome, Firefox, Safari, Edge bản mới |
| NFR-10 | Responsive | Hiển thị đúng trên desktop, tablet, mobile (cột minh hoạ bên trái có thể ẩn ở màn hình nhỏ) |
| NFR-11 | Khả dụng | Tab theo thứ tự Email → Mật khẩu → Ghi nhớ → Quên mật khẩu → reCAPTCHA → Đăng nhập; nhấn Enter trong ô nhập sẽ gửi form |
| NFR-12 | Truy cập | Nhãn (label) gắn đúng với trường; trường bắt buộc đánh dấu `*` |
| NFR-13 | Ngôn ngữ | Thông báo và nhãn bằng tiếng Việt, nhất quán |

---

## 6. Quy tắc nghiệp vụ

| Mã | Quy tắc |
|---|---|
| BR-01 | Email và Mật khẩu là bắt buộc |
| BR-02 | Email là định danh đăng nhập |
| BR-03 | Mật khẩu đúng chính xác, phân biệt hoa/thường [Giả định] |
| BR-04 | Phải qua reCAPTCHA mới đăng nhập được |
| BR-05 | Hỗ trợ 3 hình thức đăng nhập: Email/Mật khẩu, Google, GitHub |

**Chưa có thông tin về:** độ dài mật khẩu tối thiểu/tối đa, độ phức tạp mật khẩu, số lần đăng nhập sai tối đa, thời gian hết hạn phiên, đăng nhập đồng thời trên nhiều thiết bị.

---

## 7. Ma trận truy vết yêu cầu → ý tưởng kiểm thử

| Yêu cầu | Nhóm test chính |
|---|---|
| FR-LOGIN-01 | Kiểm tra UI, nội dung, giá trị mặc định |
| FR-LOGIN-02 | Đăng nhập thành công, chuyển hướng, trạng thái header |
| FR-LOGIN-03 | Bỏ trống, sai định dạng email, khoảng trắng, ký tự đặc biệt, chuỗi quá dài |
| FR-LOGIN-04 | Email không tồn tại, sai mật khẩu, tài khoản khoá, SQLi/XSS |
| FR-LOGIN-05 | Không tích captcha, captcha hết hạn, reset sau lỗi |
| FR-LOGIN-06 | Hiện/ẩn mật khẩu |
| FR-LOGIN-07 | Có/không ghi nhớ, đóng mở lại trình duyệt |
| FR-LOGIN-08/09 | Đăng nhập OAuth thành công, huỷ, email chưa có tài khoản |
| FR-LOGIN-10/11 | Điều hướng giữa các trang, truy cập trực tiếp URL |
| FR-LOGIN-12 | Đổi theme |
| NFR-01…07 | Kiểm thử bảo mật (HTTPS, CSRF, brute-force, thông báo lỗi) |
| NFR-09…12 | Tương thích trình duyệt, responsive, bàn phím, truy cập |

---

## 8. Điểm cần làm rõ với BA / Dev (Open Issues)

1. Nội dung và vị trí hiển thị các thông báo lỗi đăng nhập (sai email/mật khẩu, thiếu captcha, tài khoản bị khoá).
2. Trang đích sau đăng nhập thành công; có quay lại trang trước đó hay không.
3. Chính sách mật khẩu và giới hạn số lần đăng nhập sai (khoá tạm, thời gian khoá).
4. Thời hạn phiên và thời hạn "Ghi nhớ đăng nhập".
5. Hành vi khi đăng nhập Google/GitHub bằng email chưa có tài khoản hoặc đã có tài khoản Email/Mật khẩu (liên kết tài khoản hay tạo trùng).
6. Điều kiện hiển thị dialog "Xác nhận" có sẵn trong trang.
7. Có phân vai (học viên / giảng viên / admin) tại form này hay không.
8. Email có tự cắt khoảng trắng và phân biệt hoa/thường không.

---

## 9. Phụ lục – Phạm vi và giới hạn khi khảo sát

- Khảo sát chỉ dựa trên trang công khai `/login` và `/forgot-password`, đọc thuộc tính form và thử gửi form trống (bị chặn bởi validation trình duyệt).
- **Chưa thực hiện đăng nhập thật**: trang yêu cầu reCAPTCHA (không thể vượt qua tự động), nên các hành vi sau đăng nhập và thông báo lỗi từ server chưa được kiểm chứng. Các mục này được đánh dấu **[Giả định]** và cần bổ sung sau khi người kiểm thử tự đăng nhập bằng tài khoản test.
- Thông tin tài khoản test không được lưu trong tài liệu này.
