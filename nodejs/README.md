# Node.js — Kiến thức trọng tâm

## 1. Tổng quan

Node.js là môi trường thực thi (runtime environment) mã nguồn mở và đa nền tảng, cho phép chạy mã JavaScript ở phía máy chủ (server-side) bên ngoài trình duyệt. Node.js sử dụng JavaScript Engine V8 của Google Chrome (viết bằng C++) để biên dịch trực tiếp mã JavaScript sang mã máy.

```js
const message = "Hello, Node.js!";
console.log(message);
```

- **Single-threaded**: Mã nguồn JavaScript chạy trên một luồng chính duy nhất (main thread).
- **Non-blocking I/O & Asynchronous**: Khi thực hiện các tác vụ I/O (đọc ghi tệp, truy vấn cơ sở dữ liệu, gọi API mạng), Node.js không dừng luồng chính để chờ đợi mà ủy quyền cho hệ điều hành hoặc thread pool xử lý ngầm.
- **Thích hợp cho**: Các ứng dụng xử lý nhiều kết nối I/O đồng thời (I/O-intensive) như RESTful API, ứng dụng thời gian thực (real-time chat, streaming), microservices.
- **Không tối ưu cho**: Các tác vụ tiêu tốn nhiều tài nguyên xử lý của vi xử lý (CPU-intensive như mã hóa video nặng, tính toán ma trận phức tạp) vì sẽ chiếm dụng luồng chính và làm nghẽn toàn bộ ứng dụng.

## 2. Cài đặt và thực thi kịch bản

Cài đặt Node.js thông qua bộ cài chính thức (ưu tiên phiên bản LTS) hoặc công cụ quản lý phiên bản `nvm` (Node Version Manager):

```bash
# Kiểm tra phiên bản sau khi cài đặt
node -v
npm -v

# Thực thi một tệp kịch bản JavaScript
node app.js
```

Node.js cung cấp môi trường dòng lệnh tương tác REPL (Read-Evaluate-Print Loop) để kiểm tra nhanh biểu thức JavaScript:

```bash
$ node
> const x = 10;
> const y = 20;
> x + y
30
> .exit
```

## 3. npm, `package.json` và `package-lock.json`

`npm` (Node Package Manager) là trình quản lý gói tiêu chuẩn đi kèm với Node.js, chịu trách nhiệm tải về thư viện phụ thuộc, quản lý phiên bản và chạy các kịch bản tự động hóa.

```bash
# Khởi tạo tệp package.json mặc định
npm init -y

# Cài đặt thư viện vận hành ứng dụng (dependencies)
npm install express dotenv

# Cài đặt công cụ chỉ phục vụ quá trình phát triển (devDependencies)
npm install --save-dev nodemon typescript @types/node

# Cài đặt toàn bộ thư viện khai báo trong dự án
npm install

# Gỡ bỏ thư viện
npm uninstall express
```

Phân biệt `package.json` và `package-lock.json`:

```json
{
  "name": "node-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

- `package.json`: Khai báo siêu dữ liệu của dự án, các lệnh thực thi (`npm run <script>`), và dải phiên bản cho phép của dependencies (ký hiệu `^` cho phép cập nhật minor/patch, `~` chỉ cập nhật patch).
- `package-lock.json`: Lưu vết chính xác phiên bản cụ thể của từng package và toàn bộ cây phụ thuộc con được cài đặt trên máy. Bắt buộc phải commit tệp này lên Git để đảm bảo tính đồng nhất giữa môi trường phát triển của các thành viên và máy chủ triển khai.

## 4. Hệ thống Module (CommonJS)

Mỗi tệp `.js` trong Node.js là một module độc lập có phạm vi biến riêng, không gây ô nhiễm phạm vi toàn cục.

Phân loại module:
- **Core Modules**: Module tích hợp sẵn trong Node.js (`fs`, `path`, `http`, `events`, `crypto`).
- **Local Modules**: Tệp mã nguồn do lập trình viên tự viết trong dự án (`./math`, `./services/user`).
- **Third-party Modules**: Thư viện do bên thứ ba phát triển, cài đặt qua npm (`express`, `lodash`).

Cơ chế xuất và nhập với CommonJS:

```js
// math.js - Xuất hàm hoặc đối tượng
function add(a, b) {
  return a + b;
}

function sub(a, b) {
  return a - b;
}

module.exports = { add, sub };
```

```js
// app.js - Nhập module vào sử dụng
const { add, sub } = require("./math");

console.log(add(10, 5)); // 15
console.log(sub(10, 5)); // 5
```

- `module.exports` là đối tượng thực sự được hàm `require()` trả về.
- `exports` chỉ là biến tham chiếu rút gọn trỏ đến `module.exports`. Khi muốn xuất một hàm đơn lẻ hoặc ghi đè một đối tượng mới, phải gán trực tiếp vào `module.exports = ...`. Tránh gán `exports = ...` vì sẽ làm đứt gãy liên kết tham chiếu.

## 5. Đồng bộ (Sync) và Bất đồng bộ (Async)

- **Đồng bộ (Blocking)**: Các câu lệnh thực thi tuần tự từ trên xuống dưới; câu lệnh sau phải chờ câu lệnh trước hoàn thành. Nếu gặp thao tác tốn thời gian, toàn bộ luồng chính sẽ bị nghẽn.
- **Bất đồng bộ (Non-blocking)**: Node.js ủy quyền tác vụ I/O cho hệ thống xử lý ngầm và tiếp tục chạy ngay các dòng lệnh kế tiếp. Kết quả được đón nhận sau thông qua hàm callback, Promise hoặc async/await.

```js
const fs = require("fs");

// Mã đồng bộ (chặn luồng thực thi)
const dataSync = fs.readFileSync("input.txt", "utf8");
console.log(dataSync);
console.log("1. Kết thúc đọc file đồng bộ");

// Mã bất đồng bộ (không chặn luồng)
fs.readFile("input.txt", "utf8", (err, data) => {
  if (err) return console.error(err);
  console.log(data);
});
console.log("2. Dòng này in ra trước khi đọc file xong");
```

## 6. Callback và quy ước Error-First

Callback là một hàm được truyền vào dưới dạng tham số của hàm khác và được kích hoạt khi tác vụ bất đồng bộ hoàn tất.

Quy ước **Error-First Callback** trong Node.js:
- Tham số đầu tiên luôn dành riêng cho đối tượng lỗi (`err`). Nếu không có lỗi, giá trị này là `null` hoặc `undefined`.
- Tham số thứ hai trở đi chứa dữ liệu trả về (`data`).

```js
function calculateSquare(number, callback) {
  setTimeout(() => {
    if (typeof number !== "number") {
      callback(new Error("Tham số truyền vào phải là kiểu number"));
      return;
    }
    const result = number * number;
    callback(null, result);
  }, 100);
}

calculateSquare(5, (err, result) => {
  if (err) {
    console.error("Lỗi:", err.message);
    return;
  }
  console.log("Kết quả:", result); // 25
});
```

- **Callback Hell (Pyramid of Doom)**: Khi nhiều thao tác bất đồng bộ phụ thuộc nối tiếp nhau, việc lồng nhiều tầng callback khiến mã nguồn khó theo dõi và phức tạp trong xử lý lỗi.

## 7. Promise và `async` / `await`

Promise đại diện cho kết quả của một tác vụ bất đồng bộ trong tương lai, giúp làm phẳng cấu trúc mã nguồn.
- Ba trạng thái của Promise: `pending` (đang chờ), `fulfilled` (thành công), `rejected` (thất bại).

```js
const fsPromises = require("fs/promises");

// Xử lý bằng Promise (.then / .catch)
fsPromises
  .readFile("input.txt", "utf8")
  .then((data) => console.log("Nội dung:", data))
  .catch((err) => console.error("Lỗi đọc file:", err.message));
```

Cú pháp `async` / `await` là chuẩn mực viết mã bất đồng bộ hiện đại, mang cấu trúc tuần tự dễ đọc và cho phép bắt lỗi tập trung bằng khối `try...catch`:

```js
async function loadUserData(filePath) {
  try {
    const rawData = await fsPromises.readFile(filePath, "utf8");
    const user = JSON.parse(rawData);
    console.log("User:", user.name);
    return user;
  } catch (error) {
    if (error instanceof Error) {
      console.error("Lỗi xử lý:", error.message);
    }
    throw error;
  }
}
```

## 8. Kiến trúc Event Loop và luồng thực thi

Cơ chế giúp Node.js đơn luồng nhưng vẫn xử lý đồng thời hàng nghìn kết nối mạng:

- **Call Stack (LIFO)**: Ngăn xếp thực thi mã JavaScript đồng bộ. Câu lệnh trên đỉnh ngăn xếp luôn được chạy trước.
- **Node APIs & Thread Pool (libuv)**: Xử lý ngầm các tác vụ bất đồng bộ nặng về I/O, mạng, hệ thống tệp và bộ đếm thời gian (timers) ngoài luồng chính.
- **Callback Queue**: Hàng đợi chứa các hàm callback đã hoàn tất từ phía Node APIs, chờ được thực thi.
- **Event Loop**: Liên tục giám sát Call Stack. Khi Call Stack hoàn toàn rỗng, Event Loop sẽ lấy hàm callback đầu tiên trong Callback Queue đẩy vào Call Stack để thực thi.

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");

// Thứ tự in ra: A -> C -> B
```

- `setTimeout(..., 0)` chỉ chuyển callback vào hàng đợi sau khi hết thời gian chờ; callback bắt buộc phải đợi Call Stack chạy hết toàn bộ mã đồng bộ (`A` và `C`) mới được thực thi.
- **Quy tắc quan trọng**: Không thực hiện các vòng lặp tính toán quá nặng hoặc hàm đồng bộ tốn thời gian trên luồng chính để tránh làm nghẽn (block) Event Loop, khiến server không thể tiếp nhận request mới.

## 9. `EventEmitter` và lập trình hướng sự kiện

Module cốt lõi `events` cung cấp lớp `EventEmitter`, cho phép xây dựng mô hình phát và lắng nghe sự kiện (publish-subscribe).

```js
const EventEmitter = require("events");

class OrderService extends EventEmitter {
  placeOrder(orderId, amount) {
    console.log(`Đã tạo đơn hàng: #${orderId}`);
    // Phát sự kiện và truyền kèm dữ liệu
    this.emit("orderCreated", { orderId, amount, createdAt: new Date() });
  }
}

const orderService = new OrderService();

// Lắng nghe sự kiện
orderService.on("orderCreated", (order) => {
  console.log(`Gửi email thông báo đơn #${order.orderId} - ${order.amount} VND`);
});

// Lắng nghe một lần duy nhất rồi tự hủy
orderService.once("orderCreated", (order) => {
  console.log(`Ghi nhận đơn hàng đầu tiên trong phiên: #${order.orderId}`);
});

orderService.placeOrder(101, 250000);
```

- `on(event, listener)`: Đăng ký hàm lắng nghe sự kiện.
- `emit(event, ...args)`: Phát tín hiệu kích hoạt sự kiện kèm tham số.
- `once(event, listener)`: Đăng ký hàm lắng nghe chạy duy nhất một lần.
- `off(event, listener)`: Hủy đăng ký một hàm lắng nghe cụ thể.

## 10. Thao tác Hệ thống tệp (Module `fs`)

Module `fs` hỗ trợ tương tác với tệp và thư mục trên máy chủ. Khuyến nghị ưu tiên sử dụng phiên bản Promise (`fs/promises`) kết hợp cùng `async` / `await`.

```js
const fs = require("fs/promises");
const path = require("path");

async function manageFiles() {
  const filePath = path.join(__dirname, "sample.txt");

  // Ghi tệp
  await fs.writeFile(filePath, "Dữ liệu thử nghiệm Node.js", "utf8");

  // Đọc tệp
  const content = await fs.readFile(filePath, "utf8");
  console.log("Nội dung tệp:", content);

  // Thêm nội dung vào cuối tệp
  await fs.appendFile(filePath, "\nDòng bổ sung mới", "utf8");

  // Kiểm tra sự tồn tại của tệp
  try {
    await fs.access(filePath);
    console.log("Tệp tồn tại");
  } catch {
    console.log("Tệp không tồn tại");
  }
}
```

- Tránh sử dụng các hàm đồng bộ (`fs.readFileSync`, `fs.writeFileSync`) trong luồng xử lý của server production.

## 11. Xử lý đường dẫn tệp (Module `path`)

Module `path` cung cấp các tiện ích xử lý chuỗi đường dẫn tệp tin tương thích đa nền tảng giữa Windows (dùng dấu `\`) và POSIX/Linux/macOS (dùng dấu `/`).

```js
const path = require("path");

// Nối đường dẫn an toàn theo chuẩn hệ điều hành
const fullPath = path.join(__dirname, "assets", "images", "logo.png");

// Lấy phần mở rộng của tệp
console.log(path.extname("report.pdf")); // ".pdf"

// Lấy tên tệp kèm đuôi mở rộng
console.log(path.basename("/data/records/user.csv")); // "user.csv"

// Lấy tên tệp không kèm đuôi mở rộng
console.log(path.basename("/data/records/user.csv", ".csv")); // "user"

// Lấy thư mục chứa tệp
console.log(path.dirname("/data/records/user.csv")); // "/data/records"

// Phân tích đường dẫn thành đối tượng chi tiết
const parsed = path.parse(fullPath);
console.log(parsed.name, parsed.ext);
```

- Luôn dùng `path.join()` hoặc `path.resolve()` thay vì tự cộng chuỗi đường dẫn thủ công bằng dấu `+`.

## 12. Dữ liệu nhị phân (Buffer) và Dòng dữ liệu (Stream)

### Buffer
Buffer là vùng nhớ RAM tạm thời cấp phát bên ngoài V8 heap, dùng để chứa dữ liệu nhị phân thô (raw binary data).

```js
// Cấp phát buffer kích thước 10 bytes
const buf = Buffer.alloc(10);
buf.write("Node.js");

// Tạo buffer trực tiếp từ chuỗi
const bufFromString = Buffer.from("Xin chào", "utf8");
console.log(bufFromString); // Dạng hex các byte nhị phân
console.log(bufFromString.toString("utf8")); // "Xin chào"
console.log(bufFromString.length); // Kích thước tính theo byte
```

### Stream
Stream chia dữ liệu thành từng phần nhỏ (chunks) để xử lý tuần tự, không cần nạp toàn bộ dữ liệu vào bộ nhớ RAM cùng một lúc.
- 4 loại Stream chính: `Readable` (nguồn đọc), `Writable` (đích ghi), `Duplex` (vừa đọc vừa ghi), `Transform` (biến đổi dữ liệu lúc đọc/ghi).
- Ưu điểm: Tiết kiệm bộ nhớ (memory-efficient) và phản hồi nhanh (time-efficient).

```js
const fs = require("fs");

// Đọc tệp lớn và sao chép sang tệp mới qua cơ chế pipe
const readStream = fs.createReadStream("large-file.mp4");
const writeStream = fs.createWriteStream("copy-file.mp4");

// pipe tự động xử lý áp lực ngược (backpressure) giữa đọc và ghi
readStream.pipe(writeStream);

writeStream.on("finish", () => {
  console.log("Sao chép hoàn tất mà không làm tràn bộ nhớ RAM");
});
```

## 13. Xây dựng HTTP Web Server cơ bản

Module `http` tích hợp sẵn cho phép khởi tạo máy chủ web xử lý các yêu cầu HTTP mà không phụ thuộc vào thư viện ngoài.

```js
const http = require("http");

const server = http.createServer((req, res) => {
  const { method, url } = req;

  if (url === "/api/health" && method === "GET") {
    res.writeHead(200, { "Content-Type": "application/json" });
    res.end(JSON.stringify({ status: "OK", timestamp: Date.now() }));
    return;
  }

  res.writeHead(404, { "Content-Type": "text/plain; charset=utf-8" });
  res.end("Không tìm thấy trang yêu cầu");
});

const PORT = 3000;
server.listen(PORT, () => {
  console.log(`Server đang lắng nghe tại http://localhost:${PORT}`);
});
```

- Tham số `req` là một Readable Stream chứa thông tin request gửi lên (headers, url, method, body).
- Tham số `res` là một Writable Stream dùng để gửi status code, response headers và payload về cho client.

## 14. Phân tích địa chỉ web với URL Module

Sử dụng chuẩn WHATWG URL API tích hợp sẵn trong Node.js để phân tách và trích xuất các thành phần của địa chỉ web:

```js
const rawUrl = "https://example.com:8080/products/search?category=books&page=2#reviews";
const parsedUrl = new URL(rawUrl);

console.log(parsedUrl.hostname); // "example.com"
console.log(parsedUrl.port);     // "8080"
console.log(parsedUrl.pathname); // "/products/search"
console.log(parsedUrl.search);   // "?category=books&page=2"

// Thao tác với query parameters bằng URLSearchParams
const params = parsedUrl.searchParams;
console.log(params.get("category")); // "books"
console.log(params.get("page"));     // "2"
console.log(params.has("limit"));    // false
```

## 15. Nguyên tắc thực hành cốt lõi

- Tuyệt đối không chặn luồng chính (Don't Block the Event Loop): Không dùng các hàm đồng bộ `*Sync` (`fs.readFileSync`, `crypto.pbkdf2Sync`) trong logic phục vụ request của web server; chuyển các tác vụ tính toán CPU nặng sang Worker Threads.
- Luôn xử lý lỗi tập trung: Bắt buộc kiểm tra tham số `err` ở dòng đầu của callback; bọc các khối gọi `await` bằng `try...catch` và kiểm tra kiểu lỗi qua `error instanceof Error`.
- Đăng ký listener sự kiện `error`: Đối với `EventEmitter` và `Stream`, luôn khai báo sự kiện `.on('error', ...)` để tránh việc Node.js ném ngoại lệ chưa bắt (Unhandled error) làm sập toàn bộ tiến trình.
- Luôn commit `package-lock.json`: Đảm bảo toàn bộ thành viên trong nhóm phát triển và server production cài đặt chính xác các phiên bản gói phụ thuộc.
- Sử dụng biến môi trường: Lưu trữ các tham số nhạy cảm (cổng kết nối, chuỗi kết nối cơ sở dữ liệu, secret keys) qua biến môi trường (`process.env`) thay vì hardcode trực tiếp vào mã nguồn.
- Ưu tiên sử dụng Stream cho dữ liệu lớn: Khi xử lý tệp dung lượng lớn hoặc truyền nhận dữ liệu qua mạng, sử dụng Stream và `pipe()` thay vì đọc trọn vẹn tệp vào Buffer để duy trì mức tiêu thụ RAM ổn định.
- Nối đường dẫn tệp an toàn: Sử dụng `path.join()` hoặc `path.resolve()` để tương thích tự động với mọi hệ điều hành.
