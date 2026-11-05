# API Development — Kiến thức trọng tâm

## 1. Tổng quan về API và Kiến trúc RESTful

API (Application Programming Interface) là giao diện lập trình ứng dụng, tập hợp các đoạn mã và quy tắc cho phép các hệ thống phần mềm giao tiếp và trao đổi dữ liệu với nhau.

REST (Representational State Transfer) là kiểu kiến trúc phần mềm định ra các quy tắc xây dựng Web Service. Các API tuân thủ đúng kiến trúc này được gọi là **RESTful API**.

### 5 ràng buộc cốt lõi của REST (REST Constraints)
1. **Client - Server**: Tách biệt độc lập giữa giao diện client (Web, Mobile) và logic dữ liệu của server.
2. **Statelessness (Phi trạng thái)**: Mỗi request phải chứa đầy đủ mọi thông tin cần thiết. Server không lưu giữ bất kỳ trạng thái phiên (session state) nào của client giữa các lần gọi.
3. **Cacheability**: Phản hồi từ server phải chỉ rõ dữ liệu có được phép lưu bộ nhớ đệm (cache) hay không để tối ưu tài nguyên mạng.
4. **Layered System**: Kiến trúc phân tầng, client không cần biết đang kết nối trực tiếp với server đích hay qua các tầng trung gian (Gateway, Proxy, Load Balancer).
5. **Uniform Interface**: Giao diện đồng nhất qua 4 tiêu chuẩn:
   - *Resource identification*: Tài nguyên được định danh rõ ràng qua URI.
   - *Resource manipulation*: Thao tác với tài nguyên thông qua định dạng biểu diễn (JSON, XML).
   - *Self-descriptive messages*: Mỗi message tự mô tả cách xử lý thông qua headers (Content-Type).
   - *HATEOAS*: Phản hồi chứa các liên kết dẫn hướng đến các hành động tiếp theo.

### Quy chuẩn đặt tên URI
- **Dùng danh từ số nhiều** đại diện cho tài nguyên: `/api/v1/users`, `/api/v1/products` (không dùng động từ như `/getUsers`, `/createProduct`).
- **Phản ánh phân cấp quan hệ**: `/api/v1/users/5/orders` (danh sách đơn hàng của user ID 5).
- **Quy tắc chữ thường**: Sử dụng kebab-case cho từ ghép (`/api/v1/order-items`).
- **Đánh phiên bản**: Luôn có tiền tố phiên bản (`/api/v1/`) để kiểm soát breaking changes.

## 2. Cấu trúc Request - Response & Content-Type

Mọi giao tiếp đều tuân theo chu kỳ Request - Response trên giao thức HTTP.

### Cấu trúc HTTP Request
Gồm 3 phần:
1. **Request-line**: `<Method> <Request-URL> <HTTP-Version>` (ví dụ: `POST /api/v1/login HTTP/1.1`).
2. **Request-headers**: Cung cấp metadata về client và dữ liệu gửi lên:
   - `Host`: Địa chỉ máy chủ.
   - `Authorization`: Token hoặc thông tin chứng thực danh tính.
   - `Content-Type`: Định dạng của dữ liệu gửi lên trong body.
   - `Accept`: Định dạng dữ liệu client mong muốn nhận về (`application/json`).
3. **Request body**: Dữ liệu gửi lên server (thường dùng cho `POST`, `PUT`, `PATCH`).

### Cấu trúc HTTP Response
Gồm 3 phần:
1. **Status-line**: `<HTTP-Version> <Status-Code> <Reason-Phrase>` (ví dụ: `HTTP/1.1 200 OK`).
2. **Response-headers**: Cung cấp metadata về phản hồi:
   - `Content-Type`: Kiểu MIME của dữ liệu trả về.
   - `Cache-Control`: Chính sách lưu bộ nhớ đệm.
3. **Response body**: Dữ liệu thực tế server trả về cho client (thường là chuỗi JSON).

### Các định dạng Content-Type thông dụng
- `application/json`: Định dạng chuẩn trao đổi dữ liệu REST API hiện đại.
- `application/x-www-form-urlencoded`: Định dạng mặc định của form HTML khi submit (dạng `key1=val1&key2=val2`).
- `multipart/form-data`: Định dạng bắt buộc khi tải tệp tin (upload file, ảnh) lên server.
- `text/plain`: Gửi văn bản thuần không qua mã hóa.

## 3. Phương thức HTTP và Mã trạng thái (Status Codes)

### Các phương thức HTTP (HTTP Methods)

| Method | Mục đích | Safe | Idempotent | Body |
| :--- | :--- | :---: | :---: | :---: |
| `GET` | Lấy thông tin tài nguyên | Có | Có | Không |
| `POST` | Tạo mới một tài nguyên | Không | Không | Có |
| `PUT` | Cập nhật/thay thế toàn bộ tài nguyên | Không | Có | Có |
| `PATCH` | Cập nhật một phần thuộc tính tài nguyên | Không | Tùy trường hợp | Có |
| `DELETE` | Xóa tài nguyên chỉ định | Không | Có | Không |

- Nếu gọi phương thức không được hỗ trợ tại endpoint, server trả về mã **`405 Method Not Allowed`**.
- **Safe**: Phương thức chỉ đọc, không làm thay đổi trạng thái hệ thống.
- **Idempotent**: Thực hiện gọi lặp lại nhiều lần cùng một request sẽ luôn cho ra cùng một trạng thái kết quả.

### Các nhóm mã trạng thái HTTP quan trọng
- `1xx` (Informational): Thông tin tạm thời.
- `2xx` (Success): Xử lý thành công:
  - `200 OK`: Thành công (GET, PUT, PATCH).
  - `201 Created`: Tạo mới tài nguyên thành công (POST).
  - `204 No Content`: Thành công nhưng không có dữ liệu trả về trong body (DELETE).
- `3xx` (Redirection): Chuyển hướng:
  - `304 Not Modified`: Dữ liệu không thay đổi (dùng cho cache).
- `4xx` (Client Error): Lỗi do phía client gửi lên:
  - `400 Bad Request`: Sai cú pháp hoặc thiếu thông tin cơ bản.
  - `401 Unauthorized`: Chưa xác thực danh tính (chưa đăng nhập/token sai).
  - `403 Forbidden`: Đã xác thực nhưng không có quyền truy cập tài nguyên.
  - `404 Not Found`: Đường dẫn hoặc tài nguyên không tồn tại.
  - `409 Conflict`: Xung đột dữ liệu (ví dụ trùng email).
  - `422 Unprocessable Entity`: Dữ liệu vi phạm logic validation.
- `5xx` (Server Error): Lỗi xuất phát từ server:
  - `500 Internal Server Error`: Lỗi logic chưa được bắt ngoại lệ trên backend.
  - `502 Bad Gateway`: Lỗi giao tiếp proxy/gateway.
  - `503 Service Unavailable`: Server quá tải hoặc đang bảo trì.

## 4. Cơ chế xác thực API: HTTP Basic vs JWT

### 1. HTTP Basic Authentication (Session & Cookie)
Cơ chế truyền thống sử dụng Session lưu trữ trên server:
- Client gửi username/password qua request đăng nhập.
- Server kiểm tra, tạo một bản ghi session trong bộ nhớ/DB và gửi `Session ID` về client qua Cookie.
- Các request sau, client tự động đính kèm Cookie để server đối chiếu.
- **Ưu điểm**: Đơn giản, dễ thu hồi phiên đăng nhập ngay từ phía server.
- **Nhược điểm**: Tốn bộ nhớ RAM server, khó mở rộng (scale) khi chạy nhiều server đồng thời, dễ gặp rủi ro bảo mật CSRF.

### 2. JSON Web Token (JWT - RFC 7519)
Cơ chế xác thực dựa trên Token (Token-based) không lưu trạng thái trên server (Stateless):
- Sau khi đăng nhập thành công, server trả về một chuỗi JWT.
- Client lưu token và gửi kèm trong header mỗi request:
  ```http
  Authorization: Bearer <access_token>
  ```
- **Ưu điểm**: Khả năng mở rộng (Scalability) tuyệt vời, server không cần lưu session, phù hợp cho Microservices và Mobile app.
- **Nhược điểm**: Kích thước token lớn hơn Session ID; khó thu hồi token tức thì trước khi hết hạn (cần cơ chế blacklist qua Redis nếu muốn revoke).

### Cấu trúc 3 phần của JWT
Một chuỗi JWT gồm 3 phần ngăn cách bởi dấu chấm (`.`): `Header.Payload.Signature`
1. **Header**: Chứa thuật toán mã hóa (`alg`: HS256/RS256) và loại token (`typ`: JWT).
2. **Payload**: Chứa các thông tin (Claims) về người dùng:
   - *Registered claims*: `iss` (bên phát hành), `sub` (user ID), `exp` (thời điểm hết hạn).
   - *Custom claims*: `role`, `email`.
   > [!WARNING]
   > Payload chỉ được mã hóa dạng **Base64Url** (ai cũng đọc được nội dung). Tuyệt đối **không lưu mật khẩu hoặc dữ liệu nhạy cảm** trong Payload.
3. **Signature (Chữ ký số)**: Đảm bảo tính toàn vẹn của token:
   ```text
   HMACSHA256(base64UrlEncode(Header) + "." + base64UrlEncode(Payload), SECRET_KEY)
   ```

## 5. Ủy quyền với giao thức OAuth

OAuth (Open Authorization - RFC 5849, RFC 6749) là giao thức ủy quyền tiêu chuẩn mở, cho phép ứng dụng bên thứ ba truy cập tài nguyên của người dùng mà không cần biết mật khẩu của họ.

### OAuth 1.0 vs OAuth 2.0
- **OAuth 1.0**: Phức tạp, bắt buộc tính toán chữ ký số (cryptographic signature) trên từng request độc lập.
- **OAuth 2.0**: Chuẩn mới thay thế hoàn toàn OAuth 1.0, sử dụng cơ chế Access Token qua kênh bảo mật HTTPS, đơn giản và hỗ trợ đa nền tảng tốt hơn.

### 4 vai trò trong OAuth 2.0
1. **Resource Owner**: Người dùng sở hữu dữ liệu tài nguyên.
2. **Client**: Ứng dụng bên thứ ba cần truy cập tài nguyên.
3. **Authorization Server**: Server xác minh danh tính người dùng và cấp phát token.
4. **Resource Server**: Server lưu trữ tài nguyên thực tế, phục vụ API khi có token hợp lệ.

### Cặp đôi Access Token và Refresh Token
- **Access Token**: Thời gian sống ngắn (15–30 phút), dùng để gọi các API tài nguyên trực tiếp.
- **Refresh Token**: Thời gian sống dài (7–30 ngày), dùng để gửi lên Authorization Server xin cấp lại Access Token mới khi token cũ hết hạn mà không bắt người dùng phải đăng nhập lại.

## 6. Tài liệu hóa API & Chuẩn thiết kế Sun*

Tài liệu API (API Documentation) là bản giao ước kỹ thuật giữa Backend, Frontend và đối tác, giúp giảm thiểu thời gian trao đổi và hạn chế lỗi tích hợp.

### Thông tin bắt buộc trong tài liệu API
Mỗi endpoint cần mô tả đầy đủ:
- URL và HTTP Method.
- Quyền truy cập (Public, User hay Admin).
- Request Headers (`Content-Type`, `Authorization`).
- Danh sách tham số đầu vào (Path params, Query params, Request body kèm kiểu dữ liệu và cờ required).
- Response thành công (Status code + JSON mẫu) và Response lỗi (Status code + Error code + Message).

### Cấu trúc file "API Design" Template tại Sun Asterisk
Trong các dự án tại Sun*, tài liệu API thường được chuẩn hóa theo template chung:
- **Tiêu đề file**: `[Project ID] Project Name - API Design`.
- **Nội dung gồm các sheet chính**:
  1. *History of changes*: Nhật ký ghi lại các thay đổi của tài liệu qua các phiên bản.
  2. *Common Conventions*: Các quy ước chung của dự án (cách đặt tên, format ngày giờ, cấu trúc response).
  3. *Status code*: Danh sách toàn bộ mã lỗi và HTTP status code sử dụng trong dự án.
  4. *API List*: Danh sách tổng thể các endpoint của hệ thống.
  5. *API Detail*: Sheet chi tiết mô tả request/response cho từng API cụ thể.
  6. *Appendix*: Phụ lục về action, member, input types & parameter types.

## 7. Công cụ phát triển: Postman & Swagger

### 1. Postman
Nền tảng kiểm thử và quản lý API thông dụng:
- **Tổ chức**: Gom nhóm theo *Collections* và *Folders* theo từng module chức năng.
- **Biến môi trường (Environments)**: Quản lý biến động (`baseUrl`, `accessToken`) linh hoạt giữa local, staging và production.
- **Kiểm thử tự động (Tests Script)**: Viết script JavaScript kiểm tra status code và tự động lưu Access Token vào môi trường sau khi login:
  ```js
  pm.test("Status code is 200", () => {
    pm.response.to.have.status(200);
  });

  const res = pm.response.json();
  if (res.data && res.data.token) {
    pm.environment.set("accessToken", res.data.token);
  }
  ```

### 2. Swagger và OpenAPI Specification (OAS)
Chuẩn mô tả RESTful API độc lập với ngôn ngữ lập trình bằng định dạng YAML hoặc JSON:
- **OpenAPI Specification**: File đặc tả mô tả toàn bộ endpoints, tham số, request, response và phương thức authentication.
- **Swagger Editor**: Trình soạn thảo và kiểm tra cú pháp file OpenAPI spec.
- **Swagger UI**: Giao diện tài liệu web trực quan, cho phép người dùng click và chạy thử (Try it out) trực tiếp từ trình duyệt.
- **Swagger Codegen**: Tự động sinh mã nguồn khởi tạo cho server stubs hoặc client SDK theo nhiều ngôn ngữ.

## 8. Nguyên tắc thực hành cốt lõi

- **Tuân thủ đúng phương thức HTTP**: Dùng đúng `GET` để đọc, `POST` để tạo, `PUT`/`PATCH` để sửa, `DELETE` để xóa. Không dùng `GET` cho các hành động thay đổi dữ liệu.
- **Trả về đúng mã HTTP Status**: Không bao giờ trả `200 OK` kèm nội dung báo lỗi trong body (`{ "status": 200, "error": "Sai pass" }`); lỗi phía client bắt buộc trả về đầu mã `4xx`.
- **Bắt buộc phân trang (Pagination)**: Luôn giới hạn số bản ghi trả về (`limit`, `page`) cho các API danh sách để tránh nghẽn mạng và sập RAM server.
- **Bảo vệ Payload của JWT**: Tuyệt đối không lưu dữ liệu nhạy cảm vào Payload; luôn kiểm tra chữ ký ở server trước khi tin cậy nội dung token.
- **Sử dụng đúng Content-Type**: Dùng `application/json` cho dữ liệu thông thường và `multipart/form-data` khi upload tệp tin.
- **Thống nhất cấu trúc Response**: Giữ một cấu trúc phản hồi chuẩn (Envelope pattern: `success`, `data`, `error`) xuyên suốt toàn bộ dự án để frontend dễ dàng xử lý.
