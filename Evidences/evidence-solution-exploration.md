# Evidence: Solution Exploration — Chiến lược kiểm thử CLI & Lỗi thường gặp

**Workflow:** Solution Exploration (Present problem → Identify problems → Explore options → Choose with context → Synthesize)
---

## Bước 1 — Present problem

**Prompt (đã đính kèm code thật để làm context):**

> Tôi đang xây dựng Ticket Manager CLI với các lệnh create/list/show/update, có validate input, và lưu dữ liệu vào file JSON. Đây là code hiện tại của tôi cho phần tạo ticket:
>
> ```javascript
> const MAX_TITLE_LENGTH = 200;
> const MAX_DESCRIPTION_LENGTH = 2000;
> const VALID_PRIORITIES = ["low", "medium", "high"];
>
> function validateTitle(title) {
>   const trimmed = (title || "").trim();
>   if (!trimmed) throw new Error("Title is required");
>   if (trimmed.length > MAX_TITLE_LENGTH) {
>     throw new Error(`Title must not exceed ${MAX_TITLE_LENGTH} characters`);
>   }
>   return trimmed;
> }
>
> export function createTicket(data, store = []) {
>   const title = validateTitle(data.title);
>   const description = validateDescription(data.description);
>   const priority = validatePriority(data.priority);
>   const id = crypto.randomUUID();
>   if (store.some((t) => t.id === id)) {
>     throw new Error("Ticket ID collision, please retry");
>   }
>   const ticket = { id, title, description, status: "open", priority, createdAt: new Date().toISOString() };
>   store.push(ticket);
>   return ticket;
> }
> ```
>
> Tôi cần một chiến lược test bao phủ đủ 4 mảng: lệnh (commands), validation, lưu file (storage), và xử lý lỗi (errors) — mà không bị test thừa. Bạn thấy vấn đề gì với cách làm hiện tại của tôi?

Đọc code bạn gửi, mình thấy vài vấn đề trước khi bàn đến chuyện test.

Trước hết đoạn này chưa chạy được ngay: `validateDescription` và `validatePriority` được gọi nhưng không thấy định nghĩa ở đâu — nếu đây là toàn bộ file thì sẽ crash. `crypto` cũng chưa được import, cần thêm `import crypto from "crypto"` ở đầu file.

Chỗ mình băn khoăn nhất là đoạn check trùng ID (`store.some(...)` rồi throw). UUID v4 gần như không bao giờ trùng — xác suất thấp hơn cả khả năng phần cứng bị lỗi — nên đoạn check này không thực sự cần thiết, nó chỉ tạo thêm một nhánh lỗi mà bạn phải viết test giả lập để cover, trong khi nhánh đó gần như không bao giờ xảy ra thật. Đây đúng là kiểu "test thừa" bạn đang muốn tránh, mình nghĩ nên bỏ đi và cứ tin tưởng UUID.

Một điểm khác: `store = []` mặc định nghĩa là nếu không truyền store vào, mỗi lần gọi `createTicket` sẽ tạo ra một mảng rỗng mới — ticket vừa tạo xong là "mất" luôn. Cách này ổn khi viết unit test (vì mỗi test cần cô lập), nhưng nếu code thật cũng gọi hàm kiểu này thì ticket sẽ không bao giờ được lưu thật. Nên tách rõ: `createTicket` chỉ lo phần logic tạo ticket, còn việc đọc/ghi file là trách nhiệm của phần gọi nó (CLI command).

Cũng chưa thấy phần đọc/ghi file JSON trong code bạn gửi — đây lại là chỗ dễ phát sinh lỗi nhất (file không tồn tại, JSON hỏng, không có quyền ghi...) nên cần được test riêng.

---

## Bước 2 — Identify problems

**Prompt:**
>  Xếp hạng ưu tiên từ nghiêm trọng nhất đến ít nghiêm trọng nhất.

Xếp theo mức độ nghiêm trọng, từ chỗ chặn chạy code đến chỗ chỉ ảnh hưởng lâu dài:

1. Thiếu định nghĩa hàm — code không chạy được
validateDescription và validatePriority được gọi nhưng không tồn tại. Đây là lỗi nghiêm trọng nhất vì nó chặn hoàn toàn, không có gì để test cho tới khi sửa xong. Nếu bạn có định nghĩa hai hàm này ở file khác thì bỏ qua, nhưng nếu đây là toàn bộ logic thì phải viết trước tiên.

2. Thiếu import crypto
Cũng là lỗi chặn chạy, nhưng dễ sửa hơn nhiều so với mục 1 — chỉ thêm một dòng import. Xếp sau vì mức độ công sửa thấp hơn hẳn.

3. store = [] mặc định — lỗi thiết kế âm thầm, nguy hiểm nhất về lâu dài
Đây là vấn đề nghiêm trọng hơn 2 lỗi cú pháp ở trên xét về hậu quả thực tế, vì nó không gây crash — code vẫn chạy, ticket vẫn "được tạo", nhưng biến mất ngay sau đó nếu CLI gọi sai cách. Loại bug này rất khó phát hiện qua test thông thường (vì mỗi unit test tự nhiên đã cô lập store), nên nó có thể lọt qua toàn bộ test suite mà vẫn vỡ trận trong production. Cần tách rõ trách nhiệm: createTicket chỉ tạo object, còn CLI layer chịu trách nhiệm đọc store hiện có, gọi hàm, rồi ghi lại.

4. Thiếu phần đọc/ghi file JSON
Không phải "lỗi" trong đoạn code bạn gửi, nhưng là khoảng trống lớn nhất về mặt test coverage — đây là nơi dễ vỡ nhất trong CLI thực tế (file không tồn tại, JSON hỏng, quyền ghi, ghi đè khi có lỗi giữa chừng...). Xếp ở đây vì nó không chặn code chạy ngay bây giờ, nhưng sẽ là nguồn bug lớn nếu không thiết kế module riêng và test riêng.

5. Check trùng ID bằng UUID — thấp nhất, chỉ là "thừa" chứ không sai
Không gây lỗi, không ảnh hưởng đến độ tin cậy của hệ thống. Vấn đề duy nhất là nó tạo ra một nhánh code gần như không bao giờ chạy tới trong thực tế, buộc bạn phải viết test giả lập (mock crypto.randomUUID để trả về ID trùng) chỉ để cover một dòng code không có giá trị thực. Xếp cuối vì đây là vấn đề về "test thừa" mà bạn đã tự nhận ra, không phải rủi ro thực sự.

Tóm lại thứ tự sửa nên là: (1+2) làm code chạy được → (3) tách trách nhiệm store/CLI → (4) thiết kế module storage riêng để test I/O độc lập → (5) cân nhắc bỏ đoạn check UUID để tránh test thừa.

---

## Bước 3 — Explore options

**Prompt:**

> Với lỗi số 1, những phương pháp nào có thể để giải quyết vấn đề đó, Bạn có thể đưa tôi các giải pháp được không và ưu nhược điểm của nó

Với lỗi số 1 (thiếu validateDescription và validatePriority), có vài hướng giải quyết, tùy vào việc bạn muốn tổ chức code thế nào:

1. Viết inline ngay trong cùng file, theo đúng pattern của validateTitle

javascript
function validateDescription(description) {
  const trimmed = (description || "").trim();
  if (trimmed.length > MAX_DESCRIPTION_LENGTH) {
    throw new Error(`Description must not exceed ${MAX_DESCRIPTION_LENGTH} characters`);
  }
  return trimmed;
}

function validatePriority(priority) {
  const value = priority || "medium";
  if (!VALID_PRIORITIES.includes(value)) {
    throw new Error(`Priority must be one of: ${VALID_PRIORITIES.join(", ")}`);
  }
  return value;
}
Ưu điểm: Nhanh, nhất quán với validateTitle đã có, dễ đọc vì mọi validation nằm chung một chỗ.
Nhược điểm: Nếu sau này có thêm nhiều field cần validate (assignee, labels, due date...), file sẽ phình to, khó quản lý.

2. Tách riêng thành module validators.js

Đưa toàn bộ validateTitle, validateDescription, validatePriority (và các validator sau này) vào một file riêng, export ra rồi import vào createTicket.

Ưu điểm: Dễ test độc lập từng validator mà không cần chạy qua createTicket; dễ tái sử dụng khi có updateTicket cũng cần validate lại các field này.
Nhược điểm: Với dự án nhỏ, tách file sớm có thể là over-engineering — thêm độ phức tạp không cần thiết nếu chỉ có 3 validator.

3. Dùng schema validation library (Zod, Joi, Yup...)

javascript
import { z } from "zod";

const TicketSchema = z.object({
  title: z.string().trim().min(1).max(200),
  description: z.string().trim().max(2000).default(""),
  priority: z.enum(["low", "medium", "high"]).default("medium"),
});
Ưu điểm: Validation logic khai báo (declarative), gọn, tự động sinh type nếu dùng TypeScript, dễ maintain khi schema phức tạp dần lên, giảm code tự viết tay (ít bug hơn).
Nhược điểm: Thêm dependency bên ngoài cho một CLI tool nhỏ; error message mặc định của thư viện có thể không khớp format bạn đang dùng (throw new Error(...)), phải customize lại; tăng chi phí học cho người đọc code sau này nếu họ chưa biết thư viện đó.

4. Validate ngay trong createTicket, không tách hàm riêng

javascript
if (!data.description || data.description.trim().length <= MAX_DESCRIPTION_LENGTH) { ... }
Ưu điểm: Ít hàm, ít indirection.
Nhược điểm: Phá vỡ pattern nhất quán đã có (validateTitle đã tách riêng), khó test riêng từng rule, và khiến createTicket phình to, lẫn lộn giữa logic validate và logic tạo ticket — đi ngược lại tinh thần "test không thừa" bạn đang hướng tới, vì giờ muốn test một rule validate phải chạy qua toàn bộ createTicket.

Gợi ý cho quy mô dự án của bạn: với một CLI có 4 lệnh (create/list/show/update) và sẽ còn updateTicket cần validate lại các field tương tự, mình nghiêng về phương án 2 (tách validators.js) — vì updateTicket chắc chắn sẽ cần dùng lại validatePriority, validateDescription, tách sớm giúp tránh copy-paste code. Phương án 3 (Zod) chỉ đáng cân nhắc nếu bạn dự đoán số lượng field/rule sẽ tăng nhanh trong tương lai gần.

---

## Bước 4 — Choose with context

**Prompt:**

> Với dự án của tôi sử dụng TDD, và thời gian ngắn hạn và muốn test đáng tin cậy mà không làm chậm tiến độ của dự án thì cách nào sẽ hợp lý nhất và tại sao

Với ràng buộc "TDD + thời gian ngắn hạn + không làm chậm tiến độ", mình sẽ chọn phương án 1 (inline cùng file, theo pattern validateTitle) làm điểm khởi đầu, không phải phương án 2 hay 3. Lý do:

Tại sao không phải phương án 2 (tách validators.js) ngay bây giờ

TDD vận hành theo chu kỳ đỏ-xanh-refactor: viết test fail → viết code ít nhất để pass → refactor. Tách module là một quyết định kiến trúc, và trong TDD bạn chỉ nên đưa ra quyết định kiến trúc đó khi có bằng chứng cụ thể cần nó — cụ thể là khi bạn thực sự viết updateTicket và thấy mình copy-paste validatePriority lần thứ hai. Tách trước khi có bằng chứng đó là đoán trước tương lai (speculative generalization), tốn thời gian setup file/export/import mà chưa chắc đúng shape bạn cần — đây chính là dạng "thừa" giống hệt cái bẫy check UUID ở lỗi trước.

Tại sao không phải phương án 3 (Zod/Joi)

Thêm dependency giữa lúc đang chạy TDD nghĩa là bạn phải học API của thư viện, viết lại error message cho khớp throw new Error(...) đã dùng, và test phải điều chỉnh theo format lỗi của thư viện thay vì format bạn tự định nghĩa. Với 3 validator đơn giản (length check, enum check), overhead học + tích hợp lớn hơn nhiều lợi ích, nhất là khi deadline ngắn.

Cách làm cụ thể theo TDD

Viết test cho validateDescription trước:

javascript
test("rejects description over max length", () => {
  expect(() => validateDescription("x".repeat(2001))).toThrow(/2000 characters/);
});
test("defaults empty description to empty string", () => {
  expect(validateDescription(undefined)).toBe("");
});

Rồi mới viết hàm inline y hệt pattern validateTitle cho pass. Làm tương tự cho validatePriority. Vì cả ba hàm đã cùng nằm một file, test cũng import từ đúng một chỗ, không phải quyết định trước cấu trúc thư mục.

Khi nào nên chuyển sang phương án 2

Áp dụng "rule of three": khi bạn viết updateTicket và thấy mình cần gọi lại validatePriority/validateDescription lần thứ hai, đó là tín hiệu rõ ràng để tách — lúc đó refactor (bước cuối trong chu kỳ TDD) chính là thời điểm tự nhiên để làm việc này, và bạn tách với đầy đủ context thực tế thay vì đoán trước.

Tóm lại: đi phương án 1 để giữ nhịp TDD nhanh, và để việc tách module xảy ra như một bước refactor có động lực thật, chứ không phải quyết định trước.

---

## Bước 5 — Synthesize

**Prompt:**

> Bạn có thể tóm tắt lại: nguyên nhân gây lỗi và solution mà bạn đã chọn cho lỗi số 1 hay không

Nguyên nhân gây lỗi số 1:
Trong hàm createTicket, code gọi hai hàm validateDescription(data.description) và validatePriority(data.priority), nhưng chỉ có validateTitle được định nghĩa trong file. Hai hàm còn lại không tồn tại — nếu đây là toàn bộ logic thì code sẽ crash ngay khi chạy, không có gì để test cho tới khi sửa xong.

Solution đã chọn: Viết validateDescription và validatePriority inline ngay trong cùng file, theo đúng pattern của validateTitle đã có sẵn — không tách module riêng (validators.js), không dùng schema library (Zod/Joi).

Lý do chọn:

Phù hợp nhịp TDD: viết test fail cho từng hàm trước, rồi viết code tối thiểu để pass, không cần quyết định kiến trúc (tách file) ngay từ đầu.
Nhất quán với code hiện có, không tốn thời gian học API thư viện ngoài hay chỉnh error message cho khớp.
Tránh "thừa" kiểu speculative generalization — tách module chỉ nên làm khi có bằng chứng thật (ví dụ updateTicket cần dùng lại validator lần thứ hai), theo rule of three, và nên diễn ra như một bước refactor tự nhiên trong chu kỳ TDD chứ không phải chuẩn bị trước.

---

