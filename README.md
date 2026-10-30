# ai-test-manual

Repo lưu các tài liệu phục vụ **kiểm thử thủ công (Manual Testing) có hỗ trợ AI**: dùng Claude để khảo sát ứng dụng và sinh tài liệu đặc tả yêu cầu (SRS), làm đầu vào cho việc thiết kế test case.

## Nội dung

| File | Mô tả |
|---|---|
| [SRS_Login_AnhTester.md](SRS_Login_AnhTester.md) | SRS module **Đăng nhập** của nền tảng ANHTESTER LMS (https://anhtester.com/login) |

### SRS_Login_AnhTester.md
Tài liệu gồm:
- Giới thiệu, phạm vi, đối tượng sử dụng
- Danh sách thành phần giao diện của trang đăng nhập
- 12 yêu cầu chức năng (đăng nhập Email/Mật khẩu, reCAPTCHA, ghi nhớ đăng nhập, Google/GitHub, quên mật khẩu, điều hướng, giao diện Sáng/Tối…)
- 13 yêu cầu phi chức năng (bảo mật, hiệu năng, tương thích, truy cập…)
- Quy tắc nghiệp vụ, ma trận truy vết sang nhóm test, danh sách điểm cần làm rõ với BA/Dev

## Quy ước đánh dấu trong tài liệu

- **[Quan sát]**: thông tin xác nhận trực tiếp trên giao diện hoặc DOM của trang.
- **[Giả định]**: suy luận hoặc hành vi thông thường chưa kiểm chứng, cần xác nhận trước khi dùng làm căn cứ test.

## Giới hạn hiện tại

- Tài liệu được lập từ trang công khai `/login` và `/forgot-password`, **chưa đăng nhập thật** (trang có reCAPTCHA). Vì vậy thông báo lỗi từ server, trang đích sau đăng nhập, chính sách mật khẩu, giới hạn số lần thử… vẫn là giả định.
- Không lưu thông tin tài khoản, mật khẩu hay token trong repo này.

## Cách dùng

1. Đọc SRS để nắm phạm vi và các yêu cầu của module.
2. Dựa vào mục "Ma trận truy vết" để thiết kế test case cho từng nhóm yêu cầu.
3. Tự đăng nhập bằng tài khoản test, đối chiếu và cập nhật các mục **[Giả định]** thành **[Quan sát]**.

## Hướng phát triển

- Bổ sung SRS cho các module khác (Đăng ký, Quên mật khẩu, Khoá học, Thanh toán…).
- Sinh test case và checklist từ SRS.
- Cập nhật SRS sau khi xác nhận các điểm còn mở với BA/Dev.
