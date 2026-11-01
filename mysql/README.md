# MySQL — Kiến thức trọng tâm

## 1. Tổng quan về Cơ sở dữ liệu và RDBMS

Cơ sở dữ liệu (Database) là nơi lưu trữ dữ liệu có cấu trúc, cung cấp các cơ chế cho phép tạo, truy vấn, cập nhật và quản lý dữ liệu an toàn.

Hệ quản trị cơ sở dữ liệu quan hệ (RDBMS - Relational Database Management System) tổ chức dữ liệu dưới dạng các bảng (tables) hai chiều gồm các hàng (rows/records) và cột (columns/fields). Mối liên hệ giữa các bảng được thiết lập thông qua các khóa (Primary Key và Foreign Key), đảm bảo tính toàn vẹn tham chiếu (Referential Integrity).

```sql
-- Đăng nhập và thao tác cơ bản trong MySQL CLI
mysql -u root -p

SHOW DATABASES;
CREATE DATABASE training_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE training_db;
SHOW TABLES;
```

- MySQL hoạt động theo mô hình Client - Server.
- Nên sử dụng bộ mã `utf8mb4` để hỗ trợ đầy đủ các ký tự quốc tế và biểu tượng cảm xúc (emoji).

## 2. Các kiểu dữ liệu thông dụng (Data Types)

Việc chọn đúng kiểu dữ liệu giúp tối ưu dung lượng lưu trữ trên đĩa và tăng tốc độ xử lý câu truy vấn.

### Kiểu số (Numeric)
- `TINYINT`: 1 byte, dùng cho cờ trạng thái (`-128` đến `127` hoặc `0` đến `255` nếu có `UNSIGNED`).
- `BOOLEAN` / `BOOL`: Là alias của `TINYINT(1)`, quy ước `0` là `false`, `1` là `true`.
- `INT` / `INTEGER`: 4 bytes, kiểu số nguyên phổ biến nhất cho định danh ID (`-2` tỷ đến `+2` tỷ).
- `BIGINT`: 8 bytes, dùng cho các bảng có lượng dữ liệu cực lớn vượt quá giới hạn của `INT`.
- `FLOAT` & `DOUBLE`: Kiểu số thực dấu phẩy động (xấp xỉ), không thích hợp cho các bài toán yêu cầu độ chính xác tuyệt đối.
- `DECIMAL(M, D)`: Kiểu số có độ chính xác cố định (Fixed-point), trong đó `M` là tổng số chữ số, `D` là số chữ số phần thập phân. **Bắt buộc dùng cho tiền tệ và tính toán tài chính** (ví dụ: `DECIMAL(12, 2)`).

### Kiểu chuỗi ký tự (String)
- `CHAR(N)`: Chuỗi có độ dài cố định (`0` đến `255` ký tự). Tự động đệm khoảng trắng vào cuối cho đủ `N` ký tự. Phù hợp cho dữ liệu có độ dài chuẩn hóa như mã quốc gia (`VN`, `US`), mã bưu chính, mã số định danh.
- `VARCHAR(N)`: Chuỗi có độ dài thay đổi linh hoạt (`0` đến `65,535` bytes). Dung lượng lưu trữ thực tế phụ thuộc vào độ dài chuỗi cộng thêm 1–2 bytes header. Thích hợp cho tên, email, địa chỉ.
- `TEXT` (`TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`): Dùng cho đoạn văn bản dài (bài viết, mô tả sản phẩm). Không lưu trữ trực tiếp trên trang dữ liệu của hàng mà được lưu ở vùng nhớ riêng.
- `ENUM`: Tập hợp danh sách các giá trị hằng số định sẵn (ví dụ: `ENUM('pending', 'active', 'inactive')`).

### Kiểu ngày giờ (Date & Time)
- `DATE`: Định dạng `'YYYY-MM-DD'`.
- `TIME`: Định dạng `'HH:MM:SS'`.
- `DATETIME`: Định dạng `'YYYY-MM-DD HH:MM:SS'`, phạm vi từ năm 1000 đến 9999, không phụ thuộc múi giờ.
- `TIMESTAMP`: Lưu trữ dưới dạng Unix timestamp (UTC), tự động chuyển đổi theo múi giờ của server, phạm vi từ 1970 đến 2038. Thường dùng cho các cột `created_at` và `updated_at`.

```sql
CREATE TABLE users (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE,
  balance DECIMAL(12, 2) NOT NULL DEFAULT 0.00,
  status ENUM('active', 'inactive', 'banned') DEFAULT 'active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 3. Tạo, sửa và xóa bảng (DDL - Data Definition Language)

DDL bao gồm các câu lệnh định nghĩa cấu trúc của cơ sở dữ liệu và bảng.

```sql
-- 1. Tạo bảng
CREATE TABLE products (
  id INT UNSIGNED AUTO_INCREMENT,
  name VARCHAR(150) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  stock INT NOT NULL DEFAULT 0,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 2. Thay đổi cấu trúc bảng (ALTER TABLE)
-- Thêm cột mới
ALTER TABLE products ADD COLUMN description TEXT AFTER name;

-- Sửa kiểu dữ liệu của cột
ALTER TABLE products MODIFY COLUMN name VARCHAR(200) NOT NULL;

-- Đổi tên cột
ALTER TABLE products CHANGE COLUMN stock quantity INT NOT NULL DEFAULT 0;

-- Xóa cột
ALTER TABLE products DROP COLUMN description;

-- 3. Xóa bảng và làm sạch dữ liệu
-- DROP: Xóa hoàn toàn bảng và toàn bộ cấu trúc dữ liệu khỏi database
DROP TABLE IF EXISTS products;

-- TRUNCATE: Xóa toàn bộ dữ liệu bên trong bảng nhưng giữ nguyên cấu trúc,
-- tự động reset lại giá trị AUTO_INCREMENT về 1. Nhanh hơn DELETE không có WHERE.
TRUNCATE TABLE products;
```

## 4. Ràng buộc dữ liệu (Constraints) và Quan hệ (Relationships)

Ràng buộc dữ liệu là các quy tắc áp dụng lên cột của bảng nhằm ngăn ngừa việc lưu trữ dữ liệu không hợp lệ.

### Các ràng buộc cốt lõi
- `NOT NULL`: Bắt buộc cột phải có giá trị, không được để trống `NULL`.
- `UNIQUE`: Đảm bảo giá trị của cột không được trùng lặp giữa các hàng.
- `PRIMARY KEY`: Khóa chính định danh duy nhất cho mỗi hàng (kết hợp của `NOT NULL` và `UNIQUE`). Mỗi bảng chỉ có duy nhất một khóa chính.
- `FOREIGN KEY`: Khóa ngoại liên kết tới khóa chính của bảng khác, đảm bảo tính toàn vẹn tham chiếu.
- `DEFAULT`: Thiết lập giá trị mặc định khi câu lệnh chèn không truyền giá trị cho cột.
- `CHECK`: Kiểm tra điều kiện logic của dữ liệu (ví dụ: `CHECK (price >= 0)`).

### Hành vi toàn vẹn dữ liệu khóa ngoại (`ON DELETE` / `ON UPDATE`)
- `CASCADE`: Khi bản ghi ở bảng cha bị xóa/sửa, các bản ghi tham chiếu ở bảng con tự động bị xóa/sửa theo.
- `SET NULL`: Khi bản ghi ở bảng cha bị xóa/sửa, khóa ngoại ở bảng con được chuyển thành `NULL`.
- `RESTRICT` / `NO ACTION`: Ngăn chặn việc xóa/sửa bản ghi ở bảng cha nếu vẫn còn dữ liệu tham chiếu ở bảng con.

### Thiết kế các mối quan hệ (Relationships)

```sql
-- 1. Quan hệ 1 - 1 (One-to-One): Khóa ngoại kèm ràng buộc UNIQUE
CREATE TABLE user_profiles (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id INT UNSIGNED NOT NULL UNIQUE,
  id_card_number VARCHAR(20) NOT NULL UNIQUE,
  address VARCHAR(255),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 2. Quan hệ 1 - N (One-to-Many): Khóa ngoại nằm ở bảng phía "Nhiều"
CREATE TABLE orders (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  user_id INT UNSIGNED NOT NULL,
  total_amount DECIMAL(12, 2) NOT NULL,
  order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE RESTRICT
);

-- 3. Quan hệ N - N (Many-to-Many): Dùng bảng trung gian (Junction table) chứa 2 khóa ngoại
CREATE TABLE order_items (
  order_id INT UNSIGNED NOT NULL,
  product_id INT UNSIGNED NOT NULL,
  quantity INT NOT NULL DEFAULT 1,
  unit_price DECIMAL(10, 2) NOT NULL,
  PRIMARY KEY (order_id, product_id),
  FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
  FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE RESTRICT
);
```

## 5. Thao tác dữ liệu cơ bản (DML: INSERT, UPDATE, DELETE)

DML là tập hợp các câu lệnh tương tác trực tiếp với dữ liệu trong bảng.

```sql
-- 1. Chèn dữ liệu (INSERT)
-- Chèn một dòng
INSERT INTO products (name, price, stock)
VALUES ('Bàn phím cơ', 1200000.00, 15);

-- Chèn nhiều dòng cùng lúc (giảm thiểu số lần gửi query lên server)
INSERT INTO products (name, price, stock) VALUES
  ('Chuột không dây', 450000.00, 30),
  ('Tai nghe gaming', 890000.00, 20),
  ('Lót chuột', 150000.00, 50);

-- 2. Cập nhật dữ liệu (UPDATE)
-- Luôn luôn kiểm tra mệnh đề WHERE trước khi chạy lệnh
UPDATE products
SET price = 1100000.00, stock = stock - 1
WHERE id = 1;

-- 3. Xóa dữ liệu (DELETE)
-- Bắt buộc phải có WHERE, nếu không sẽ xóa sạch toàn bộ bản ghi trong bảng
DELETE FROM products
WHERE id = 4;
```

- **Quy tắc an toàn**: Không bao giờ thực thi `UPDATE` hoặc `DELETE` trên môi trường production mà không chỉ định mệnh đề `WHERE`. Luôn chạy thử `SELECT` với cùng điều kiện `WHERE` trước để kiểm tra tập bản ghi sẽ bị ảnh hưởng.

## 6. Truy vấn dữ liệu với câu lệnh SELECT

```sql
SELECT
  category_id,
  COUNT(id) AS total_products,
  AVG(price) AS average_price
FROM products
WHERE price >= 200000 AND status = 'active'
GROUP BY category_id
HAVING total_products > 5
ORDER BY average_price DESC
LIMIT 10 OFFSET 0;
```

### Các mệnh đề và toán tử chọn lọc
- `AS`: Đặt bí danh ngắn gọn cho tên cột hoặc bảng kết quả.
- `DISTINCT`: Loại bỏ các dòng kết quả trùng lặp giá trị (`SELECT DISTINCT category_id FROM products;`).
- Các toán tử lọc trong `WHERE`:
  - So sánh: `=`, `!=` (hoặc `<>`), `>`, `<`, `>=`, `<=`.
  - Khoảng giá trị: `BETWEEN a AND b` (tương đương `val >= a AND val <= b`).
  - Danh sách giá trị: `IN ('apple', 'banana', 'orange')`.
  - Khớp mẫu chuỗi: `LIKE '%phone%'` (`%` đại diện cho 0 hoặc nhiều ký tự, `_` đại diện cho đúng 1 ký tự).
  - Kiểm tra giá trị rỗng: `IS NULL` hoặc `IS NOT NULL` (không dùng `= NULL`).
- Sắp xếp và phân trang:
  - `ORDER BY column_name ASC` (tăng dần - mặc định) hoặc `DESC` (giảm dần).
  - `LIMIT count OFFSET offset`: Bỏ qua `offset` dòng đầu và lấy `count` dòng tiếp theo.
- Phân biệt `WHERE` và `HAVING`:
  - `WHERE`: Lọc các dòng dữ liệu riêng lẻ **trước khi** thực hiện gom nhóm (`GROUP BY`). Không được dùng hàm tổng hợp (`COUNT`, `SUM`, `AVG`) trong `WHERE`.
  - `HAVING`: Lọc các nhóm dữ liệu **sau khi** đã tính toán gom nhóm bằng `GROUP BY`.

## 7. Kỹ thuật ghép bảng (JOIN)

JOIN cho phép kết hợp các cột từ hai hoặc nhiều bảng dựa trên mối quan hệ giữa các cột chung.

```sql
-- Dữ liệu mẫu minh họa:
-- Bảng customers (id, name)
-- Bảng orders (id, customer_id, total_amount)

-- 1. INNER JOIN: Chỉ lấy các bản ghi có sự khớp nối ở cả 2 bảng
SELECT
  c.id AS customer_id,
  c.name,
  o.id AS order_id,
  o.total_amount
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;

-- 2. LEFT JOIN: Lấy toàn bộ khách hàng, nếu khách chưa có đơn thì cột đơn mang giá trị NULL
SELECT
  c.id AS customer_id,
  c.name,
  o.id AS order_id
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;

-- Tìm khách hàng chưa từng mua đơn hàng nào
SELECT c.id, c.name
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;

-- 3. SELF JOIN: Ghép bảng với chính nó để xử lý dữ liệu dạng cây phân cấp
-- Ví dụ: Bảng employees (id, name, manager_id)
SELECT
  e.name AS employee_name,
  m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

- `INNER JOIN`: Phù hợp khi chỉ cần lấy các bản ghi có dữ liệu liên quan ở cả 2 phía.
- `LEFT JOIN`: Phù hợp khi cần giữ lại toàn bộ dữ liệu bảng chính và chấp nhận dữ liệu bảng phụ có thể bị khuyết (`NULL`).

## 8. Truy vấn con (Subquery)

Truy vấn con là một câu lệnh `SELECT` được lồng bên trong một câu truy vấn khác (đặt trong dấu ngoặc đơn `(...)`).

### 1. Scalar Subquery (Truy vấn con trả về giá trị đơn)
Trả về duy nhất 1 hàng và 1 cột, có thể dùng trực tiếp với các toán tử so sánh thông thường:

```sql
-- Tìm các sản phẩm có giá cao hơn giá trung bình của toàn bộ sản phẩm
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

### 2. Multi-row Subquery (Truy vấn con trả về danh sách nhiều dòng)
Kết hợp với các toán tử tập hợp `IN`, `NOT IN`, `ANY`, `ALL`:

```sql
-- Tìm khách hàng có đơn hàng từ 1.000.000 trở lên
SELECT id, name
FROM customers
WHERE id IN (
  SELECT customer_id
  FROM orders
  WHERE total_amount >= 1000000
);
```

### 3. Correlated Subquery & `EXISTS`
Truy vấn con tương quan phụ thuộc vào dữ liệu của từng dòng từ truy vấn cha bên ngoài. Toán tử `EXISTS` kiểm tra sự tồn tại của ít nhất một bản ghi thỏa mãn điều kiện và dừng quét ngay khi tìm thấy kết quả đầu tiên:

```sql
-- Tìm khách hàng có ít nhất một đơn hàng (tối ưu hơn IN trên tập dữ liệu lớn)
SELECT c.id, c.name
FROM customers c
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.customer_id = c.id
);
```

- **Tối ưu hóa**: Hầu hết các subquery độc lập có thể được viết lại bằng `JOIN` để MySQL tận dụng tối đa chỉ mục và tối ưu hóa đường dẫn thực thi (execution plan).

## 9. Các hàm tích hợp thông dụng (Built-in Functions)

```sql
-- 1. Hàm chuỗi
SELECT
  CONCAT(first_name, ' ', last_name) AS full_name,
  UPPER(email) AS upper_email,
  LENGTH(password_hash) AS hash_length,
  TRIM('  text  ') AS clean_text,
  REPLACE('hello world', 'world', 'mysql') AS replaced;

-- 2. Hàm số học
SELECT
  ROUND(123.456, 2) AS rounded,   -- 123.46 (làm tròn)
  FLOOR(123.99)     AS floored,   -- 123 (làm tròn xuống)
  CEIL(123.01)      AS ceiled,    -- 124 (làm tròn lên)
  ABS(-50)          AS absolute;  -- 50 (giá trị tuyệt đối)

-- 3. Hàm ngày giờ
SELECT
  NOW() AS current_datetime,
  CURDATE() AS current_date,
  DATE_ADD(NOW(), INTERVAL 7 DAY) AS next_week,
  DATEDIFF('2026-12-31', NOW()) AS days_left,
  DATE_FORMAT(NOW(), '%d/%m/%Y') AS formatted_date;

-- 4. Hàm điều kiện và xử lý NULL
-- IF(điều kiện, giá trị nếu đúng, giá trị nếu sai)
SELECT name, IF(stock > 0, 'Còn hàng', 'Hết hàng') AS stock_status FROM products;

-- COALESCE: Lấy giá trị đầu tiên khác NULL
SELECT name, COALESCE(phone, email, 'Không có thông tin liên lạc') AS contact FROM users;

-- CASE WHEN đa nhánh
SELECT
  name,
  price,
  CASE
    WHEN price < 500000 THEN 'Bình dân'
    WHEN price BETWEEN 500000 AND 2000000 THEN 'Trung cấp'
    ELSE 'Cao cấp'
  END AS price_segment
FROM products;
```

## 10. Khung nhìn ảo (View)

View là một bảng ảo (virtual table) được hình thành từ kết quả của một câu lệnh `SELECT` đã lưu sẵn. View không lưu trữ dữ liệu vật lý riêng mà truy xuất trực tiếp từ các bảng gốc mỗi khi được gọi.

```sql
-- Tạo View tổng hợp thông tin đơn hàng của người dùng
CREATE OR REPLACE VIEW v_customer_order_summary AS
SELECT
  c.id AS customer_id,
  c.name AS customer_name,
  COUNT(o.id) AS total_orders,
  COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;

-- Truy vấn trên View như một bảng thông thường
SELECT *
FROM v_customer_order_summary
WHERE total_spent > 5000000;

-- Xóa View
DROP VIEW IF EXISTS v_customer_order_summary;
```

- **Mục đích sử dụng**:
  - Đơn giản hóa các câu lệnh truy vấn phức tạp (nhiều JOIN hoặc tính toán gộp) cho phía lập trình viên ứng dụng.
  - Tăng cường bảo mật: Ẩn bớt các cột nhạy cảm (như mật khẩu, số căn cước, tiền lương) và chỉ cấp quyền truy cập trên View cho người dùng.

## 11. Stored Procedures và Stored Functions

Stored Routine là tập hợp các câu lệnh SQL được biên dịch và lưu trữ trực tiếp trên máy chủ cơ sở dữ liệu.

```sql
-- Đổi ký tự phân cách câu lệnh (delimiter) để tránh xung đột dấu chấm phẩy
DELIMITER //

-- 1. Stored Procedure: Gọi bằng lệnh CALL, không bắt buộc trả về giá trị
CREATE PROCEDURE GetOrdersByCustomer(IN p_customer_id INT)
BEGIN
  SELECT id, total_amount, order_date
  FROM orders
  WHERE customer_id = p_customer_id
  ORDER BY order_date DESC;
END //

-- 2. Stored Function: Bắt buộc trả về duy nhất một giá trị, gọi trực tiếp trong SELECT
CREATE FUNCTION CalculateTax(amount DECIMAL(10, 2))
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
  DECLARE tax DECIMAL(10, 2);
  SET tax = amount * 0.10;
  RETURN tax;
END //

DELIMITER ;

-- Gọi Procedure
CALL GetOrdersByCustomer(1);

-- Gọi Function trong câu SELECT
SELECT id, total_amount, CalculateTax(total_amount) AS vat_amount FROM orders;
```

- Phân biệt: Stored Procedure được thực thi độc lập bằng lệnh `CALL` và có thể nhận tham số dạng `IN`, `OUT`, `INOUT`. Stored Function bắt buộc phải có câu lệnh `RETURN` và được gọi lồng bên trong biểu thức SQL.
- **Kinh nghiệm thực tế**: Hạn chế đặt quá nhiều logic nghiệp vụ phức tạp vào Stored Routine vì rất khó debug, khó kiểm soát phiên bản qua Git và khó mở rộng theo chiều ngang (horizontal scaling) so với việc viết logic tại tầng Application Code (Node.js/TypeScript).

## 12. Chỉ mục (Index) và Tối ưu hóa truy vấn

Chỉ mục (Index) là cấu trúc dữ liệu (mặc định là cấu trúc cây B-Tree trong InnoDB) giúp tăng tốc độ tìm kiếm và sắp xếp bản ghi mà không cần phải duyệt qua từng dòng của toàn bộ bảng (Full Table Scan).

```sql
-- 1. Tạo Index cho cột thường xuyên xuất hiện trong mệnh đề WHERE hoặc JOIN
CREATE INDEX idx_products_category ON products(category_id);

-- 2. Tạo Composite Index (Chỉ mục kết hợp nhiều cột)
CREATE INDEX idx_orders_customer_date ON orders(customer_id, order_date);

-- 3. Xóa Index khi không còn cần thiết
DROP INDEX idx_products_category ON products;

-- 4. Phân tích kế hoạch thực thi câu truy vấn bằng EXPLAIN
EXPLAIN SELECT * FROM orders WHERE customer_id = 10 AND order_date >= '2026-01-01';
```

- **Clustered Index**: Trong InnoDB, bảng dữ liệu vật lý được tổ chức và sắp xếp trực tiếp theo Khóa chính (Primary Key). Mỗi bảng chỉ có duy nhất 1 Clustered Index.
- **Secondary Index**: Các index tạo thêm chứa giá trị của cột được đánh chỉ mục và con trỏ trỏ về khóa chính.
- **Nguyên tắc Leftmost Prefix**: Với Composite Index `(A, B, C)`, câu truy vấn sẽ tận dụng được index nếu điều kiện tìm kiếm chứa `A`, hoặc `(A, B)`, hoặc `(A, B, C)`. Nếu chỉ tìm kiếm theo `B` hoặc `C`, MySQL sẽ không thể dùng index này.
- **Cái giá của Index**: Giúp câu lệnh `SELECT` nhanh hơn rõ rệt, nhưng làm chậm các thao tác ghi dữ liệu (`INSERT`, `UPDATE`, `DELETE`) vì database phải cập nhật lại cây chỉ mục, đồng thời tiêu tốn thêm dung lượng lưu trữ trên đĩa.

## 13. Giao dịch (Transaction) và Chuẩn ACID

Transaction là một tập hợp các thao tác cơ sở dữ liệu được thực thi như một đơn vị công việc độc lập theo nguyên tắc "Tất cả hoặc không gì cả" (All-or-Nothing).

### Bốn đặc tính cốt lõi (ACID)
- **Atomicity (Tính nguyên tố)**: Toàn bộ các thao tác trong transaction phải thành công trọn vẹn. Nếu có bất kỳ thao tác nào gặp lỗi, toàn bộ các thay đổi trước đó sẽ bị hủy bỏ (Rollback) về trạng thái ban đầu.
- **Consistency (Tính nhất quán)**: Dữ liệu luôn chuyển từ trạng thái hợp lệ này sang trạng thái hợp lệ khác, không vi phạm bất kỳ ràng buộc dữ liệu nào.
- **Isolation (Tính cô lập)**: Các transaction chạy đồng thời không được can thiệp lẫn nhau. Kết quả của một transaction chưa commit không bị nhìn thấy bởi transaction khác.
- **Durability (Tính bền vững)**: Khi transaction đã commit thành công, dữ liệu được ghi nhận vĩnh viễn trên đĩa và không bị mất ngay cả khi hệ thống gặp sự cố mất điện.

```sql
-- Ví dụ kinh điển: Chuyển tiền giữa hai tài khoản ngân hàng
START TRANSACTION;

-- Bước 1: Trừ tiền từ tài khoản nguồn (ID = 1)
UPDATE accounts
SET balance = balance - 500000.00
WHERE id = 1 AND balance >= 500000.00;

-- Bước 2: Cộng tiền vào tài khoản đích (ID = 2)
UPDATE accounts
SET balance = balance + 500000.00
WHERE id = 2;

-- Nếu cả hai câu lệnh thành công, xác nhận lưu vĩnh viễn vào database
COMMIT;

-- Trường hợp phát hiện lỗi (ví dụ không đủ số dư hoặc tài khoản đích bị khóa):
-- ROLLBACK;
```

## 14. Nguyên tắc thực hành cốt lõi

- Luôn dùng `DECIMAL` cho tiền tệ và số liệu tài chính; tuyệt đối không dùng `FLOAT` hay `DOUBLE` vì lỗi làm tròn dấu phẩy động.
- Sử dụng bộ mã `utf8mb4` cùng collation `utf8mb4_unicode_ci` cho toàn bộ cơ sở dữ liệu để hỗ trợ tiếng Việt có dấu và ký tự đặc biệt.
- Tuyệt đối không chạy lệnh `UPDATE` hoặc `DELETE` mà thiếu mệnh đề `WHERE` trên môi trường production.
- Hạn chế tối đa việc sử dụng `SELECT *`; chỉ chọn đúng các cột cần thiết nhằm giảm tải I/O đĩa, bộ nhớ RAM và băng thông mạng.
- Đánh chỉ mục (Index) đúng chỗ: Đánh index cho các cột khóa ngoại, các cột xuất hiện thường xuyên trong mệnh đề `WHERE`, `JOIN` và `ORDER BY`. Không đánh index bừa bãi trên các bảng có tần suất ghi cao.
- Sử dụng Transaction cho các luồng thao tác cập nhật liên quan nhiều bảng để đảm bảo tính toàn vẹn dữ liệu.
- Ngăn chặn SQL Injection: Tại tầng ứng dụng (Node.js/TypeScript), luôn luôn sử dụng Prepared Statements hoặc Parameterized Queries (hoặc qua các ORM như Prisma, TypeORM, Kysely) thay vì tự cộng chuỗi câu lệnh SQL.
- Sử dụng `EXPLAIN` để kiểm tra kế hoạch thực thi của các câu truy vấn chậm, đảm bảo query đang sử dụng index (`type` là `ref`, `range` hoặc `eq_ref` thay vì `ALL`).

