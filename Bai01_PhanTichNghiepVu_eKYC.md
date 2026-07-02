# TÀI LIỆU PHÂN TÍCH NGHIỆP VỤ HỆ THỐNG eKYC
## Ngân hàng số ABC Bank

---

## 1. Actors (Tác nhân tham gia hệ thống)

| Actor | Mô tả | Vai trò |
|-------|-------|---------|
| **Khách hàng (Customer)** | Người dùng cuối muốn mở tài khoản ngân hàng | Đăng ký, upload giấy tờ, xác thực khuôn mặt, nhận kết quả |
| **Hệ thống eKYC** | Nền tảng xử lý tự động | OCR, Liveness Detection, đối chiếu dữ liệu, sinh tài khoản |
| **Giao dịch viên (Teller)** | Nhân viên ngân hàng (can thiệp tối thiểu) | Xử lý exception cases, duyệt manual khi cần |
| **Admin/Hệ thống Core Banking** | Hệ thống ngân hàng lõi | Kích hoạt tài khoản, quản lý hạn mức, đồng bộ dữ liệu |
| **Cơ quan quản lý (Regulator)** | NHNN, cơ quan pháp luật | Giám sát tuân thủ, lưu trữ lịch sử giao dịch |

---

## 2. Business Flow (Luồng nghiệp vụ chi tiết)

### 2.1. Sơ đồ luồng tổng quan

```
[BẮT ĐẦU]
    │
    ▼
[1. Đăng ký thông tin cơ bản]
    │ ● Nhập fullName, phone, email, citizenId
    │ ● Validation dữ liệu đầu vào
    │ ● Kiểm tra trùng lặp
    │
    ▼
[2. Upload & OCR CCCD]
    │ ● Chụp mặt trước CCCD
    │ ● Chụp mặt sau CCCD
    │ ● OCR trích xuất thông tin
    │ ● Đối chiếu với dữ liệu đăng ký
    │
    ▼
[3. Xác thực khuôn mặt (Liveness)]
    │ ● Chụp ảnh chân dung / Quay video ngắn
    │ ● Phát hiện khuôn mặt thật (chống deepfake)
    │ ● So khớp khuôn mặt với ảnh trên CCCD (Face Matching)
    │
    ▼
[4. Hệ thống đối chiếu & Xét duyệt]
    │ ● Kiểm tra điểm tín dụng (nếu có)
    │ ● Kiểm tra danh sách đen / AML
    │ ● Đánh giá rủi ro
    │
    ▼
[5. Kích hoạt tài khoản]
    │ ● Sinh số tài khoản
    │ ● Tạo PENDING/ACTIVE status
    │ ● Gửi SMS/Email thông báo
    │
    ▼
[KẾT THÚC]
```

### 2.2. Luồng chi tiết từng bước

#### Bước 1: Đăng ký thông tin cơ bản
1. Khách hàng tải ứng dụng ABC Bank
2. Chọn chức năng "Mở tài khoản mới"
3. Nhập các thông tin: Họ và tên, Số điện thoại, Email, Số CCCD
4. Hệ thống kiểm tra định dạng và tính hợp lệ
5. Hệ thống kiểm tra trùng lặp với cơ sở dữ liệu
6. Nếu hợp lệ → chuyển sang bước 2

#### Bước 2: Upload & Xử lý CCCD
1. Hệ thống hiển thị hướng dẫn chụp ảnh CCCD
2. Khách hàng chụp mặt trước CCCD
3. Hệ thống kiểm tra chất lượng ảnh (độ sáng, nét, góc chụp)
4. OCR trích xuất: số CCCD, họ tên, ngày sinh, giới tính, quê quán
5. Khách hàng chụp mặt sau CCCD
6. OCR trích xuất: ngày cấp, nơi cấp
7. Hệ thống đối chiếu dữ liệu OCR với thông tin đã nhập
8. Nếu khớp → chuyển sang bước 3

#### Bước 3: Xác thực khuôn mặt
1. Hệ thống yêu cầu khách hàng đưa mặt vào khung hình
2. Thực hiện Liveness Detection (chớp mắt, quay đầu...)
3. Chụp ảnh chân dung
4. Face Matching: so sánh ảnh chân dung với ảnh trên CCCD
5. Tính điểm tin cậy (confidence score)
6. Nếu điểm >= ngưỡng → chuyển sang bước 4

#### Bước 4: Đối chiếu & Xét duyệt
1. Hệ thống kiểm tra thông tin với cơ sở dữ liệu quốc gia (nếu có)
2. Kiểm tra danh sách đen/AML
3. Nếu có vấn đề → chuyển sang xét duyệt thủ công
4. Nếu OK → chuyển sang bước 5

#### Bước 5: Kích hoạt tài khoản
1. Hệ thống sinh số tài khoản duy nhất
2. Tạo bản ghi tài khoản với trạng thái PENDING
3. Gửi thông báo qua SMS/Email
4. Nếu đủ điều kiện → tự động kích hoạt (ACTIVE)

---

## 3. Functional Requirements (Yêu cầu chức năng)

| ID | Yêu cầu | Mô tả | Module |
|----|---------|-------|--------|
| FR-01 | Đăng ký thông tin | Cho phép nhập họ tên, SĐT, email, CCCD | Đăng ký |
| FR-02 | Validate dữ liệu | Kiểm tra định dạng, độ dài, tính hợp lệ | Đăng ký |
| FR-03 | Kiểm tra trùng lặp | Kiểm tra CCCD/SĐT/email đã tồn tại | Đăng ký |
| FR-04 | Upload ảnh CCCD | Chụp/upload ảnh mặt trước và mặt sau | OCR |
| FR-05 | Kiểm tra chất lượng ảnh | Kiểm tra độ nét, sáng, góc chụp | OCR |
| FR-06 | Trích xuất OCR | Nhận dạng và trích xuất thông tin từ CCCD | OCR |
| FR-07 | Đối chiếu dữ liệu | So sánh thông tin OCR vs thông tin đã nhập | OCR |
| FR-08 | Liveness Detection | Phát hiện khuôn mặt thật (chống giả mạo) | Face Auth |
| FR-09 | Face Matching | So khớp khuôn mặt với ảnh trên CCCD | Face Auth |
| FR-10 | Tính điểm tin cậy | Tính và đánh giá confidence score | Face Auth |
| FR-11 | Sinh số tài khoản | Tự động sinh số tài khoản duy nhất | Activation |
| FR-12 | Quản lý trạng thái | PENDING → ACTIVE / REJECTED | Activation |
| FR-13 | Thông báo kết quả | Gửi SMS/Email thông báo kết quả | Activation |
| FR-14 | Xử lý ngoại lệ | Chuyển các trường hợp không rõ ràng sang duyệt thủ công | All |

---

## 4. Non-Functional Requirements (Yêu cầu phi chức năng)

| ID | Yêu cầu | Mô tả | Chỉ tiêu |
|----|---------|-------|----------|
| NFR-01 | Bảo mật dữ liệu | Mã hóa dữ liệu cá nhân (AES-256), truyền qua HTTPS | PCI DSS |
| NFR-02 | Xác thực & Phân quyền | JWT token, OAuth2 cho API | OWASP |
| NFR-03 | Hiệu năng xử lý OCR | Thời gian xử lý OCR | < 5 giây/ảnh |
| NFR-04 | Hiệu năng Face Match | Thời gian xử lý face matching | < 10 giây |
| NFR-05 | Thời gian phản hồi API | Response time cho các API thông thường | < 500ms (P95) |
| NFR-06 | Tính sẵn sàng | Uptime hệ thống | 99.9% (downtime < 8.7h/năm) |
| NFR-07 | Khả năng mở rộng | Horizontal scaling, load balancing | +200% traffic |
| NFR-08 | Lưu trữ & Audit Log | Lưu toàn bộ log giao dịch tối thiểu 5 năm | Luật định |
| NFR-09 | Tuân thủ pháp lý | Đáp ứng Nghị định 52/2013/NĐ-CP về thương mại điện tử | Pháp luật VN |
| NFR-10 | UX/UI | Thời gian hoàn thành eKYC | < 5 phút |

---

## 5. Assumptions & Business Rules (Giả định & Quy tắc nghiệp vụ)

### 5.1. Assumptions (Giả định)
1. Khách hàng đã có smartphone kết nối internet ổn định
2. Khách hàng đã đủ 18 tuổi theo quy định pháp luật
3. CCCD/CMND còn hạn sử dụng (không quá 15 năm)
4. Khách hàng không nằm trong danh sách đen tín dụng
5. Hệ thống Core Banking đã có sẵn API tích hợp
6. Dịch vụ OCR và Face Matching sử dụng bên thứ ba (third-party)

### 5.2. Business Rules (Quy tắc nghiệp vụ)

**BR-01: Điều kiện độ tuổi**
- Khách hàng phải từ đủ 18 tuổi trở lên
- Tuổi tính tại thời điểm đăng ký

**BR-02: Hạn mức tài khoản**
- Tài khoản chưa xác thực eKYC: hạn mức giao dịch 10.000.000 VND/ngày
- Tài khoản đã xác thực eKYC: hạn mức 100.000.000 VND/ngày

**BR-03: Chính sách chống rửa tiền (AML)**
- Giao dịch > 300.000.000 VND phải báo cáo NHNN
- Phát hiện giao dịch bất thường → đóng băng tài khoản tạm thời

**BR-04: Xử lý trùng lặp**
- Mỗi số CCCD chỉ được đăng ký 1 tài khoản eKYC
- Mỗi số điện thoại chỉ đăng ký được 1 tài khoản
- Mỗi email chỉ đăng ký được 1 tài khoản

**BR-05: Thời gian hiệu lực**
- Yêu cầu eKYC chưa hoàn tất sẽ tự động hủy sau 24 giờ
- Link kích hoạt tài khoản có hiệu lực 72 giờ

**BR-06: Điểm tín dụng tối thiểu**
- Điểm Face Matching confidence >= 85% mới được chấp nhận
- OCR accuracy >= 90%

---

## 6. User Stories

| ID | User Story |
|----|------------|
| US-01 | **As a** Customer, **I want to** register my basic information (name, phone, email, citizenId), **So that** I can start the account opening process |
| US-02 | **As a** Customer, **I want to** upload photos of my ID card, **So that** the system can extract my information via OCR |
| US-03 | **As a** Customer, **I want to** perform a facial scan/liveness check, **So that** the system can verify my identity against my ID photo |
| US-04 | **As a** Customer, **I want to** receive notification of my account opening result, **So that** I know whether my account has been activated |
| US-05 | **As a** System, **I want to** validate all input data against business rules, **So that** only valid registration requests are processed |
| US-06 | **As a** System, **I want to** perform OCR on uploaded ID images, **So that** identity data can be extracted automatically |
| US-07 | **As a** System, **I want to** perform face liveness detection and matching, **So that** identity fraud can be prevented |
| US-08 | **As a** System, **I want to** generate unique account numbers automatically, **So that** approved customers receive their accounts immediately |
| US-09 | **As a** Teller, **I want to** review and handle edge/exception cases, **So that** customers with special circumstances can still be served |
| US-10 | **As a** Admin, **I want to** view audit logs of all eKYC activities, **So that** compliance and regulatory requirements are met |

---

## 7. Use Cases

### UC-01: Đăng ký thông tin cơ bản
| Thành phần | Mô tả |
|------------|-------|
| **Tên Use Case** | Đăng ký thông tin cơ bản (Basic Registration) |
| **Mô tả** | Khách hàng nhập thông tin cá nhân để bắt đầu quy trình eKYC |
| **Actors** | Khách hàng, Hệ thống eKYC |
| **Preconditions** | Khách hàng đã tải ứng dụng ABC Bank |
| **Basic Flow** | 1. KH nhập thông tin → 2. Hệ thống validate → 3. Hệ thống kiểm tra trùng lặp → 4. Hệ thống lưu tạm thông tin → 5. Chuyển sang bước tiếp theo |
| **Alternative Flow** | 2a. Dữ liệu không hợp lệ → hiển thị lỗi yêu cầu nhập lại. 3a. Trùng lặp → thông báo tài khoản đã tồn tại |
| **Postconditions** | Thông tin khách hàng được lưu tạm, sẵn sàng cho bước OCR |

### UC-02: Upload & Xử lý CCCD
| Thành phần | Mô tả |
|------------|-------|
| **Tên Use Case** | Upload và xử lý CCCD (ID Card OCR Processing) |
| **Mô tả** | Khách hàng chụp ảnh CCCD và hệ thống trích xuất thông tin |
| **Actors** | Khách hàng, Hệ thống eKYC |
| **Preconditions** | UC-01 đã hoàn tất |
| **Basic Flow** | 1. KH chụp mặt trước CCCD → 2. Hệ thống kiểm tra chất lượng ảnh → 3. OCR trích xuất dữ liệu → 4. KH chụp mặt sau → 5. OCR trích xuất → 6. Đối chiếu dữ liệu |
| **Alternative Flow** | 2a. Ảnh kém chất lượng → yêu cầu chụp lại. 6a. Dữ liệu không khớp → yêu cầu kiểm tra lại |
| **Postconditions** | Thông tin CCCD được trích xuất và đối chiếu thành công |

### UC-03: Xác thực khuôn mặt
| Thành phần | Mô tả |
|------------|-------|
| **Tên Use Case** | Xác thực khuôn mặt (Face Authentication) |
| **Mô tả** | Xác thực khuôn mặt thật và so khớp với ảnh trên CCCD |
| **Actors** | Khách hàng, Hệ thống eKYC |
| **Preconditions** | UC-02 đã hoàn tất |
| **Basic Flow** | 1. KH đưa mặt vào khung hình → 2. Hệ thống phát hiện khuôn mặt → 3. Thực hiện Liveness Check → 4. Chụp ảnh chân dung → 5. Face Matching với ảnh CCCD → 6. Tính confidence score |
| **Alternative Flow** | 3a. Phát hiện giả mạo → từ chối, yêu cầu làm lại. 5a. Điểm khớp thấp → yêu cầu làm lại hoặc chuyển duyệt thủ công |
| **Postconditions** | Khuôn mặt được xác thực với confidence score >= ngưỡng cho phép |

### UC-04: Kích hoạt tài khoản
| Thành phần | Mô tả |
|------------|-------|
| **Tên Use Case** | Kích hoạt tài khoản (Account Activation) |
| **Mô tả** | Hệ thống sinh số tài khoản và kích hoạt sau khi xác thực thành công |
| **Actors** | Hệ thống eKYC, Core Banking System |
| **Preconditions** | UC-03 đã hoàn tất, tất cả kiểm tra đều đạt |
| **Basic Flow** | 1. Hệ thống kiểm tra lần cuối → 2. Sinh số tài khoản → 3. Tạo bản ghi tài khoản (PENDING) → 4. Gửi SMS/Email thông báo → 5. Kích hoạt (ACTIVE) |
| **Alternative Flow** | 1a. Phát hiện rủi ro → chuyển duyệt thủ công. 5a. Lỗi Core Banking → thử lại sau, thông báo cho KH |
| **Postconditions** | Tài khoản được kích hoạt hoặc ở trạng thái chờ duyệt |

---

## 8. Ma trận ánh xạ Yêu cầu - User Story - Use Case

| Yêu cầu | User Story | Use Case |
|---------|------------|----------|
| FR-01, FR-02, FR-03 | US-01 | UC-01 |
| FR-04, FR-05, FR-06, FR-07 | US-02 | UC-02 |
| FR-08, FR-09, FR-10 | US-03 | UC-03 |
| FR-11, FR-12, FR-13 | US-04, US-08 | UC-04 |
| FR-14 | US-09 | - |
| NFR-08 | US-10 | - |

---

*Tài liệu được tạo bởi AI - Senior Business Analyst (FinTech)*
*Dự án: Hệ thống eKYC - Ngân hàng số ABC Bank*
