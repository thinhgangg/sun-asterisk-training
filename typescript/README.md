# TypeScript — Kiến thức trọng tâm

## 1. Tổng quan

TypeScript là ngôn ngữ mở rộng từ JavaScript bằng cách bổ sung hệ thống kiểm tra kiểu tĩnh (static typing). Trình biên dịch TypeScript (`tsc`) kiểm tra lỗi kiểu dữ liệu trước khi dịch mã thành JavaScript thuần; toàn bộ thông tin kiểu sẽ bị xóa bỏ khi chạy ở runtime (type erasure).

```ts
const message: string = "Hello, TypeScript!";
console.log(message);
```

- TypeScript giúp phát hiện lỗi sai kiểu, gõ nhầm tên thuộc tính ngay trong quá trình phát triển (compile-time).
- Trình duyệt và Node.js không chạy trực tiếp file `.ts`, bắt buộc phải biên dịch về file `.js`.
- TypeScript kiểm tra theo cấu trúc (structural typing): hai đối tượng có cùng thuộc tính và kiểu dữ liệu sẽ được coi là tương thích với nhau.

## 2. Cài đặt, biên dịch và `tsconfig.json`

Cài đặt trình biên dịch TypeScript vào dự án:

```bash
npm install --save-dev typescript
npx tsc --init
npx tsc
```

Trong quá trình phát triển (development), có thể dùng `tsx` để thực thi trực tiếp file `.ts` mà không cần biên dịch thủ công qua file `.js`:

```bash
# Chạy trực tiếp hoặc theo dõi thay đổi (watch mode)
npx tsx src/index.ts
npx tsx watch src/index.ts
```

Cấu hình nền tảng trong `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

- `target`: phiên bản JavaScript xuất ra sau khi biên dịch.
- `module`: định dạng hệ thống module (commonjs, nodenext, ES2022).
- `rootDir`: thư mục chứa mã nguồn TypeScript gốc.
- `outDir`: thư mục chứa mã JavaScript sau khi build.
- `strict`: kích hoạt toàn bộ các chế độ kiểm tra kiểu nghiêm ngặt.
- `strictNullChecks`: bắt buộc kiểm tra các giá trị `null` và `undefined`.

## 3. Suy luận kiểu (Type Inference) và Annotation

TypeScript tự động suy luận kiểu dữ liệu từ giá trị khởi tạo ban đầu. Chỉ cần khai báo kiểu tường minh (type annotation) khi cần làm rõ ý định, khi khai báo tham số hàm hoặc khi biến chưa được gán giá trị ngay.

```ts
let language = "TypeScript"; // suy luận thành string
let count = 10; // suy luận thành number

function add(a: number, b: number): number {
  return a + b;
}
```

- Biến khai báo bằng `const` với giá trị nguyên thủy được suy luận thành literal type hẹp nhất:

```ts
const status = "active"; // kiểu "active"
let nextStatus = "active"; // kiểu string
```

## 4. Các kiểu dữ liệu cơ bản

```ts
const fullName: string = "Nguyen Van A";
const age: number = 22;
const isActive: boolean = true;
const bigValue: bigint = 100n;
const sym: symbol = Symbol("id");
const empty: null = null;
const notFound: undefined = undefined;

const numbers: number[] = [1, 2, 3];
const names: Array<string> = ["An", "Binh"];
```

`void` dùng cho hàm không trả về giá trị hữu ích. `never` biểu diễn các trường hợp không bao giờ xảy ra (hàm luôn ném lỗi hoặc lặp vô tận).

```ts
function logMessage(msg: string): void {
  console.log(msg);
}

function throwError(errorMsg: string): never {
  throw new Error(errorMsg);
}
```

## 5. Object, thuộc tính tùy chọn và `readonly`

```ts
type User = {
  readonly id: number;
  name: string;
  email?: string; // thuộc tính tùy chọn
};

const user: User = {
  id: 1,
  name: "Hoang",
};

user.name = "Hoang Nguyen";
// user.id = 2; // Lỗi: id là thuộc tính readonly
```

- Dấu `?` đánh dấu thuộc tính có thể vắng mặt (mang giá trị `undefined`).
- `readonly` chỉ được kiểm tra ở compile-time, ngăn không cho gán lại thuộc tính sau khi khởi tạo.

## 6. Mảng (Array) và Tuple

Mảng chứa các phần tử có cùng kiểu dữ liệu. Tuple mô tả mảng có độ dài cố định và kiểu dữ liệu tại từng vị trí đã được xác định trước.

```ts
// Mảng thông thường
const scores: number[] = [8, 9, 10];

// Tuple
type Point = [x: number, y: number];
const coordinates: Point = [105.85, 21.02];

type ApiResponse = readonly [status: number, message: string];
const res: ApiResponse = [200, "OK"];
// res.push("extra"); // Lỗi: readonly ngăn thay đổi độ dài tuple
```

## 7. `interface` và `type`

Cả hai đều dùng để định nghĩa cấu trúc dữ liệu. `interface` phù hợp để mô tả đối tượng và class có tính mở rộng; `type` linh hoạt hơn cho union, tuple và các phép kết hợp kiểu.

```ts
// Interface hỗ trợ kế thừa bằng extends
interface Person {
  id: number;
  name: string;
}

interface Employee extends Person {
  department: string;
}

// Type Alias linh hoạt cho union và intersection
type ID = string | number;
type Worker = Person & { role: string };
```

- `interface` hỗ trợ declaration merging (hai interface trùng tên ở các nơi khác nhau sẽ tự động gộp thuộc tính).
- `type` không thể khai báo trùng tên trong cùng một phạm vi.

## 8. Hàm (Function)

Khai báo kiểu cho tham số đầu vào, kiểu trả về, tham số tùy chọn, tham số mặc định và rest parameter.

```ts
function createUser(
  name: string,
  age: number = 18,
  email?: string,
  ...tags: string[]
): { name: string; age: number; email?: string; tags: string[] } {
  return { name, age, email, tags };
}

// Định nghĩa kiểu cho hàm (Function Type Signature)
type Calculate = (a: number, b: number) => number;
const multiply: Calculate = (x, y) => x * y;
```

- Tham số tùy chọn (`?`) phải luôn đặt sau các tham số bắt buộc.

## 9. Union, Literal và Intersection

```ts
// Union Types: nhận một trong các kiểu
type Status = "pending" | "success" | "error";
let currentStatus: Status = "pending";

// Literal Types: giới hạn giá trị chính xác
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

// Intersection Types: gộp các kiểu lại với nhau
type Timestamps = { createdAt: Date; updatedAt: Date };
type Post = { id: number; title: string } & Timestamps;
```

## 10. Thu hẹp kiểu (Type Narrowing)

Khi biến có kiểu union, cần kiểm tra kiểu bằng các câu lệnh điều kiện để TypeScript thu hẹp về kiểu cụ thể trước khi sử dụng.

```ts
function formatValue(value: string | number): string {
  if (typeof value === "string") {
    return value.trim(); // value là string
  }
  return value.toFixed(2); // value là number
}
```

Các kỹ thuật thu hẹp kiểu phổ biến:

- `typeof`: kiểm tra kiểu nguyên thủy (`string`, `number`, `boolean`).
- `instanceof`: kiểm tra instance của class hoặc đối tượng Date.
- `in`: kiểm tra sự tồn tại của thuộc tính trong object.
- Discriminated Union: dùng thuộc tính định danh chung (như `status`, `type`) để phân nhánh.
- Type Predicate (Custom Type Guard): sử dụng từ khóa `is` trong kiểu trả về để định nghĩa hàm kiểm tra kiểu dùng lại, đặc biệt hữu ích khi lọc mảng (`Array.prototype.filter`).

```ts
// Discriminated Union
type State =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; error: string };

function handleState(state: State) {
  switch (state.status) {
    case "loading":
      return "Đang tải...";
    case "success":
      return state.data.join(", ");
    case "error":
      return state.error;
  }
}

// Custom Type Guard với toán tử "is"
function isString(value: unknown): value is string {
  return typeof value === "string";
}

const rawList: (string | number)[] = ["apple", 42, "banana", 100];
const stringList: string[] = rawList.filter(isString); // TypeScript suy luận đúng string[]
```

## 11. `any`, `unknown` và Type Assertion

- `any`: vô hiệu hóa kiểm tra kiểu, dễ gây lỗi runtime. Hạn chế tối đa việc sử dụng.
- `unknown`: an toàn hơn `any`. Có thể gán mọi giá trị vào `unknown`, nhưng bắt buộc phải kiểm tra kiểu trước khi gọi hàm hoặc đọc thuộc tính.

```ts
let input: unknown = "Hello";

if (typeof input === "string") {
  console.log(input.toUpperCase());
}
```

- Trong chế độ `strict: true`, biến trong khối `catch (error)` có kiểu mặc định là `unknown`. Cần thu hẹp kiểu bằng `instanceof Error` trước khi truy cập thuộc tính:

```ts
try {
  JSON.parse("{ malformed json }");
} catch (error) {
  if (error instanceof Error) {
    console.error("Lỗi:", error.message);
  } else {
    console.error("Lỗi không xác định:", error);
  }
}
```

- Type Assertion (`as`): khẳng định kiểu với trình biên dịch khi lập trình viên chắc chắn về kiểu hơn TypeScript. Không thay đổi giá trị thực tế lúc runtime.

```ts
const element = document.getElementById("username") as HTMLInputElement;
if (element) {
  console.log(element.value);
}
```

- Const Assertion (`as const`): yêu cầu TypeScript coi toàn bộ biểu thức là hằng số bất biến (đóng băng kiểu thành literal type hẹp nhất, thuộc tính trở thành `readonly`). Thường dùng làm giải pháp thay thế linh hoạt cho `enum`:

```ts
// Biến mảng thành tuple readonly của các literal types
const HTTP_METHODS = ["GET", "POST", "PUT", "DELETE"] as const;
type HttpMethod = (typeof HTTP_METHODS)[number]; // "GET" | "POST" | "PUT" | "DELETE"

// Biến object thành readonly literal types
const CONFIG = {
  endpoint: "https://api.example.com",
  timeout: 5000,
} as const;
// CONFIG.timeout = 10000; // Lỗi: thuộc tính chỉ đọc
```

## 12. Generic

Generic cho phép truyền tham số kiểu (`<T>`), giúp tái sử dụng mã nguồn cho nhiều kiểu dữ liệu khác nhau mà vẫn giữ nguyên an toàn kiểu.

```ts
// Hàm Generic
function getFirstElement<T>(items: T[]): T | undefined {
  return items[0];
}

const firstNumber = getFirstElement([1, 2, 3]); // number | undefined
const firstString = getFirstElement(["a", "b"]); // string | undefined

// Generic Interface cho API Response
interface ApiResponse<T> {
  code: number;
  message: string;
  data: T;
}

type UserData = { id: number; name: string };
const response: ApiResponse<UserData> = {
  code: 200,
  message: "Success",
  data: { id: 1, name: "An" },
};
```

Ràng buộc Generic bằng từ khóa `extends`:

```ts
interface HasId {
  id: number;
}

function printId<T extends HasId>(item: T): void {
  console.log(item.id);
}
```

## 13. `keyof` và Indexed Access Types

- `keyof`: lấy hợp union của tất cả các khóa trong một kiểu object.
- Indexed Access Type (`T[K]`): truy xuất kiểu của một thuộc tính cụ thể.

```ts
type Product = {
  id: number;
  name: string;
  price: number;
};

type ProductKey = keyof Product; // "id" | "name" | "price"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const item: Product = { id: 1, name: "Bàn phím", price: 300000 };
const productName = getProperty(item, "name"); // string
```

## 14. Các Utility Types thông dụng

TypeScript cung cấp sẵn các kiểu tiện ích để biến đổi kiểu dữ liệu có sẵn:

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

// Làm tất cả thuộc tính thành tùy chọn
type UpdateUserDto = Partial<User>;

// Làm tất cả thuộc tính thành bắt buộc
type CompleteUser = Required<UpdateUserDto>;

// Khóa chỉ đọc tất cả thuộc tính
type ReadonlyUser = Readonly<User>;

// Chỉ chọn một số thuộc tính
type UserPreview = Pick<User, "id" | "name">;

// Bỏ bớt thuộc tính chỉ định
type CreateUserDto = Omit<User, "id">;

// Tạo object với key và value định sẵn
type UserRoles = Record<string, boolean>;
```

Utility types cho hàm và bất đồng bộ (async/Promise):

```ts
function getUser() {
  return { id: 1, username: "admin" };
}

// Lấy kiểu giá trị trả về của hàm
type UserResult = ReturnType<typeof getUser>; // { id: number; username: string }

// Lấy kiểu dữ liệu bên trong Promise sau khi resolve (Awaited)
async function fetchUser() {
  return { id: 1, name: "Nguyen Van A" };
}

type AsyncUserData = Awaited<ReturnType<typeof fetchUser>>; // { id: number; name: string }

// Lọc kiểu trong Union: Exclude (loại trừ) và Extract (trích xuất)
type AllRoles = "admin" | "editor" | "guest";
type StaffRoles = Exclude<AllRoles, "guest">; // "admin" | "editor"
```

## 15. Xử lý an toàn `null` và `undefined`

Khi bật `strictNullChecks: true`, `null` và `undefined` không tự động gán được vào các kiểu khác nếu không được khai báo rõ ràng.

```ts
// Optional Chaining (?.): dừng truy cập an toàn nếu gặp null/undefined
type Profile = { address?: { city?: string } };
const userProfile: Profile = {};
console.log(userProfile.address?.city); // undefined, không sập ứng dụng

// Nullish Coalescing (??): chỉ dùng giá trị dự phòng khi là null hoặc undefined
let count = 0;
console.log(count ?? 10); // in ra 0 (đúng), khác với || sẽ in ra 10

// Non-null assertion operator (!): khẳng định giá trị chắc chắn tồn tại
const el = document.getElementById("root");
el!.innerHTML = "Hello";
```

## 16. Class và Access Modifiers

```ts
interface IPrintable {
  print(): void;
}

class ProductItem implements IPrintable {
  // Cú pháp viết tắt Parameter Properties
  constructor(
    public readonly id: number,
    public name: string,
    private costPrice: number,
    protected category: string = "General",
  ) {}

  print(): void {
    console.log(`${this.id} - ${this.name}`);
  }

  getCost(): number {
    return this.costPrice;
  }
}
```

- `public`: truy cập ở mọi nơi (mặc định).
- `private`: chỉ truy cập trong nội bộ class khai báo (chỉ bảo vệ ở compile-time; để đảm bảo tính đóng tuyệt đối lúc runtime nên dùng `#private` của ECMAScript).
- `protected`: truy cập trong class đó và các class con kế thừa.
- `readonly`: chỉ đọc, không gán lại sau khi khởi tạo.

## 17. Nguyên tắc thực hành cốt lõi

- Luôn bật `strict: true` trong `tsconfig.json`.
- Tận dụng suy luận kiểu tự động; không viết type annotation cho các khai báo đơn giản, rõ ràng.
- Ưu tiên sử dụng `unknown` thay cho `any` với dữ liệu chưa xác định từ bên ngoài; xử lý lỗi trong `catch (error)` bằng kiểm tra `error instanceof Error`.
- Ưu tiên dùng `as const` thay cho `enum` để định nghĩa tập hợp hằng số gọn gàng và giữ an toàn kiểu.
- Mô hình hóa các trạng thái bất đồng bộ bằng Discriminated Union thay vì dùng nhiều cờ boolean rời rạc.
- Sử dụng Custom Type Guard (`param is Type`) khi cần hàm kiểm tra kiểu tái sử dụng hoặc lọc mảng.
- Sử dụng `interface` cho cấu trúc dữ liệu đối tượng có thể mở rộng, dùng `type` cho union, tuple và kiểu kết hợp.
- Hạn chế lạm dụng ép kiểu (`as`) và dấu khẳng định không null (`!`); ưu tiên thu hẹp kiểu bằng các câu lệnh điều kiện.
