# Nghiên cứu: Test-Driven Development (TDD) cho CLI Tool

Tài liệu nghiên cứu Tuần 1 — Test-Driven Development để xây dựng công cụ CLI đáng tin cậy với sự hỗ trợ của AI (Ticket Manager CLI).


## 1. Nguyên tắc TDD & Red-Green-Refactor

TDD (Test-Driven Development — Phát triển hướng kiểm thử) là phương pháp viết test cho một hành vi **trước** khi viết code triển khai hành vi đó, sau đó viết code tối thiểu để test pass, rồi cải thiện code mà vẫn giữ test pass.

**Chu trình Red → Green → Refactor:**

- **Red (Đỏ):** Viết 1 test mô tả hành vi mong muốn — hành vi này chưa được cài đặt nên khi chạy, test FAIL. Đây cũng là bước xác nhận test thực sự kiểm tra đúng thứ cần kiểm tra (nếu test pass ngay dù chưa có code, test đó có vấn đề).
- **Green (Xanh):** Viết code triển khai tối thiểu, chỉ đủ để test pass. Không cần tối ưu hay đẹp ở bước này — mục tiêu là pass nhanh nhất.
- **Refactor (Tái cấu trúc):** Cải thiện chất lượng code (đặt tên rõ, loại bỏ trùng lặp, tách hàm...) mà **không** đổi hành vi bên ngoài. Chạy lại test sau mỗi thay đổi nhỏ — test đóng vai trò lưới an toàn.
- Lặp lại chu trình cho hành vi tiếp theo.

**Lợi ích chính:**
- Test luôn song hành với code, không bị "quên viết test sau"
- Thiết kế code có xu hướng đơn giản, tối giản (chỉ code đủ để đáp ứng yêu cầu thực tế)
- Có lưới an toàn khi refactor hoặc khi AI sửa code — chạy test là biết ngay có hỏng gì không
- Test đóng vai trò tài liệu sống, mô tả hành vi hệ thống một cách chính xác

---

## 2. So sánh TDD với Traditional Workflow

Traditional Workflow (quy trình truyền thống): viết code triển khai trước theo yêu cầu, sau đó — nếu có thời gian — mới viết test để xác nhận (test-after), hoặc chỉ test thủ công bằng tay, thậm chí không viết test chính thức nào.

Tóm tắt nhanh:

| | TDD | Traditional |
|---|---|---|
| Viết test | Trước khi code | Sau khi code (hoặc không viết) |
| Tốc độ ban đầu | Chậm hơn | Nhanh hơn |
| Bắt lỗi | Sớm | Muộn |

So sánh theo từng tiêu chí:

**Thời điểm viết test**
- *TDD:* Trước khi viết code (Red trước)
- *Traditional:* Sau khi viết code xong (hoặc không viết)

**Thiết kế code**
- *TDD:* Có xu hướng modular, loose-coupled — vì phải nghĩ "làm sao để test được" ngay từ đầu
- *Traditional:* Dễ viết code chặt chẽ, phụ thuộc lẫn nhau — khó tách ra để test sau này

**Tốc độ ban đầu**
- *TDD:* Chậm hơn — tốn thời gian viết test + setup trước khi có tính năng chạy được
- *Traditional:* Nhanh hơn — có bản chạy được sớm hơn

**Thời điểm bắt lỗi**
- *TDD:* Sớm, ngay khi viết (test fail báo ngay)
- *Traditional:* Muộn hơn — chờ đến bước test thủ công, QA, hoặc production mới lộ ra

**Test coverage**
- *TDD:* Cao & nhất quán — bắt buộc theo quy trình, không dễ bị bỏ quên
- *Traditional:* Thường thấp hơn — dễ bị cắt bỏ khi gấp deadline

**Tài liệu sống**
- *TDD:* Có — test mô tả đúng hành vi hệ thống, luôn đồng bộ với code
- *Traditional:* Thường không có, hoặc tài liệu tách rời dễ lỗi thời

**An toàn khi refactor**
- *TDD:* Cao — có lưới an toàn ngay từ đầu
- *Traditional:* Thấp hơn nếu chưa có test — dễ gây hồi quy (regression) không phát hiện kịp

**Phù hợp nhất khi**
- *TDD:* Yêu cầu đã rõ, logic nghiệp vụ quan trọng, cần độ tin cậy cao, làm việc với code do AI sinh ra
- *Traditional:* Prototype/POC, yêu cầu chưa rõ/còn thay đổi liên tục, cần demo rất nhanh

**TDD hơn Traditional Workflow ở:**
- Thiết kế code tốt hơn: ràng buộc "phải test được" từ đầu khiến code có xu hướng modular, dễ tách rời, dễ tái sử dụng
- Bắt lỗi sớm hơn: lỗi lộ ra ngay khi code, thay vì đợi đến QA hoặc production — chi phí sửa lỗi thấp hơn nhiều
- Test coverage đảm bảo hơn: là một phần bắt buộc của quy trình, không dễ bị cắt bỏ khi deadline gấp
- Tự tin refactor: có lưới an toàn ngay từ đầu, dám cải thiện code liên tục mà không sợ hỏng tính năng cũ
- Phù hợp làm việc với AI: test đóng vai trò "hợp đồng khách quan" để kiểm chứng code AI sinh ra (xem mục 6)

**TDD kém hơn Traditional Workflow ở:**
- Tốc độ ban đầu chậm hơn: với prototype/MVP cần demo gấp, viết test trước có thể làm chậm tiến độ ra bản chạy được đầu tiên
- Đường cong học tập cao hơn: người mới dễ viết test sai hoặc vô nghĩa trước khi quen tư duy "test trước" (dễ dẫn tới các lỗi ở mục 7)
- Không phù hợp mọi loại code: code khám phá/thử nghiệm (spike), UI còn thay đổi liên tục, hoặc requirement chưa rõ ràng
- Overhead bảo trì: test cũng là code — nếu viết tệ, test suite trở thành gánh nặng thay vì lưới an toàn
- Không tự động đảm bảo đúng nghiệp vụ: TDD chỉ đảm bảo quy trình "viết test trước", không tự đảm bảo test phản ánh đúng yêu cầu thực tế

---

## 3. So sánh Unit Test / Integration Test / End-to-End Test

Tóm tắt nhanh:

| | Unit Test | Integration Test | E2E Test |
|---|---|---|---|
| Tốc độ | Rất nhanh | Trung bình | Chậm nhất |
| Phạm vi | 1 hàm, cô lập | Nhiều thành phần | Toàn bộ ứng dụng |

Chi tiết từng loại:

### Unit Test
- **Phạm vi:** 1 hàm/class, cô lập (mock dependency)
- **Tốc độ:** Rất nhanh (mili-giây)
- **Phát hiện lỗi gì:** Lỗi logic cục bộ trong 1 hàm
- **Ví dụ trong Ticket Manager CLI:** Test hàm `validateTicketInput()` báo lỗi khi title rỗng
- **Khi nào dùng:** Logic nghiệp vụ, validation, hàm thuần (pure function)
- **Dấu hiệu sai cấp độ:** Unit test nhưng lại ghi/đọc file thật → thực chất là integration test đội lốt

### Integration Test
- **Phạm vi:** Nhiều thành phần phối hợp (vd: logic + file storage thật)
- **Tốc độ:** Trung bình
- **Phát hiện lỗi gì:** Lỗi ở ranh giới giữa các module (vd: dữ liệu ghi/đọc file sai định dạng)
- **Ví dụ trong Ticket Manager CLI:** Gọi `createTicket()` thật, ghi xuống file JSON tạm, đọc lại kiểm tra đúng dữ liệu
- **Khi nào dùng:** Tương tác giữa các layer (command handler ↔ storage)
- **Dấu hiệu sai cấp độ:** Mock hết mọi thứ → không còn kiểm tra tích hợp thật, thành unit test yếu

### End-to-End (E2E) Test
- **Phạm vi:** Toàn bộ ứng dụng, chạy như người dùng thật
- **Tốc độ:** Chậm nhất
- **Phát hiện lỗi gì:** Lỗi tổng thể, từ input dòng lệnh đến output cuối cùng
- **Ví dụ trong Ticket Manager CLI:** Chạy lệnh `ticket create --title "Fix bug"` trong subprocess, kiểm tra output & exit code
- **Khi nào dùng:** Luồng người dùng quan trọng nhất (happy path + vài lỗi phổ biến)
- **Dấu hiệu sai cấp độ:** Test luôn cả logic nội bộ chi tiết → nên đẩy xuống unit test cho nhanh

### Cách nhận biết test bị "lẫn cấp độ"

**Bộ câu hỏi tự kiểm tra nhanh** — trước/sau khi viết xong 1 test, tự hỏi:

| Câu hỏi | Nếu trả lời... |
|---|---|
| Test này có chạm vào I/O thật không (file, DB, network, subprocess)? | Không chạm gì cả → chắc chắn là unit test, dù đặt tên file là `*.integration.test.js` |
| Nếu xóa hết `jest.mock(...)` trong file, test còn chạy được không? | Không chạy được (vì mock đang giả toàn bộ dependency) → đây là unit test đội lốt integration |
| Test có setup/teardown file thật, temp folder không? | Có → đúng chất integration. Không có gì để dọn dẹp sau test → nghi ngờ là unit |
| Nếu dependency thật đổi (schema DB đổi, API đổi) mà test vẫn xanh — có phát hiện ra không? | Không phát hiện được → đó là unit test (mock không tự cập nhật theo thực tế) |

**Triệu chứng cụ thể:**

*Integration test nhưng mock hết như unit test* — mock cả `storage` lẫn `validate` thì không còn gì được "tích hợp" thật, thực chất là unit test đặt sai tên.
```javascript
jest.mock('../src/storage');
jest.mock('../src/validate');
test('integration: create ticket', () => {
  storage.writeAll.mockReturnValue(true);
});
```

*Unit test nhưng lại đụng file/DB thật* — logic validate không cần đọc file để test; làm vậy khiến unit test chậm, flaky, và khó biết lỗi do logic hay do file system.

*E2E test nhưng "leo" vào kiểm tra chi tiết logic nội bộ* — E2E chỉ nên assert trên thứ user thấy được (stdout, exit code), không import module nội bộ để assert sâu.

*Integration test cố cover mọi input/edge case như unit test* — lãng phí vì chạy chậm hơn 10-50 lần so với unit mà kết quả giống hệt; nên gom hết case validate thuần túy về unit, integration chỉ cần 1-2 case xác nhận validate được gọi đúng chỗ trong luồng thật.

**Bảng "test signature" — nhận diện nhanh qua đặc điểm:**

| Đặc điểm quan sát được | Unit | Integration | E2E |
|---|---|---|---|
| Thời gian chạy 1 test | < 10ms | 10ms – vài trăm ms | vài trăm ms – vài giây |
| Có `beforeEach` tạo file/folder tạm không | Không | Có | Có |
| Có gọi subprocess không | Không | Không | Có |
| Số lượng `jest.mock()` | Nhiều, mock gần hết | Ít, chỉ mock thứ ở rìa (3rd-party) | Không mock gì |
| Assert vào đâu | Giá trị trả về của hàm | Trạng thái file/DB sau khi chạy | stdout/stderr/exit code/file cuối cùng |
| Góc nhìn code nội bộ | White-box (biết rõ) | Grey-box (biết một phần) | Black-box (không biết gì) |

**Quy tắc ngón tay cái:** số lượng mock tỉ lệ nghịch với cấp độ test — Unit mock nhiều nhất, Integration mock ít (chỉ thứ ngoài tầm kiểm soát như API bên thứ 3), E2E gần như không mock gì. Nếu thấy file integration test toàn `jest.mock()` giống hệt file unit test bên cạnh — đó là dấu hiệu rõ ràng nhất của việc lẫn cấp độ.

---

## 4. Cần test gì trong một CLI Tool

**a) Commands (các lệnh)**
- Mỗi command chạy đúng hành vi mong đợi: `create`, `list`, `show`, `update`, `delete`
- Đúng exit code (0 = thành công, khác 0 = lỗi) — quan trọng vì script/CI dựa vào exit code
- Output đúng định dạng (stdout cho kết quả, stderr cho lỗi)
- Đúng khi kết hợp các flags/options (`--json`, `--verbose`, `-h`...)

**b) Validation (xác thực đầu vào)**
- Input hợp lệ được chấp nhận
- Input thiếu field bắt buộc (thiếu title khi tạo ticket) bị từ chối, kèm thông báo lỗi rõ ràng
- Input sai kiểu dữ liệu, ID không tồn tại
- Input biên: chuỗi rỗng, chuỗi quá dài, ký tự đặc biệt/unicode

**c) File storage (lưu trữ file)**
- Dữ liệu được ghi đúng xuống file sau khi create/update
- Đọc lại đúng dữ liệu đã lưu (persistence — tắt mở lại app vẫn còn data)
- Xử lý khi file chưa tồn tại (lần đầu chạy app)
- Xử lý khi file bị corrupt (JSON lỗi cú pháp) — không được crash im lặng

**d) Errors (xử lý lỗi)**
- Thông báo lỗi rõ ràng, hữu ích cho người dùng (không phải stack trace khó hiểu)
- Update/delete ticket không tồn tại → lỗi đúng loại (not found), không crash
- Lỗi hệ thống (không có quyền ghi file, disk đầy...) được xử lý gracefully

---

## 5. Ví dụ test cho Ticket Manager CLI

**Ticket entity:**
```javascript
/**
 * {
 *   id: string          - UUID, tự sinh, client không thể truyền vào
 *   title: string       - bắt buộc, không rỗng, tối đa 200 ký tự
 *   description: string - tùy chọn, mặc định '', tối đa 2000 ký tự
 *   status: string      - luôn 'open' khi tạo mới, client không ghi đè được
 *   priority: string    - tùy chọn, mặc định 'medium', chỉ nhận: 'low' | 'medium' | 'high'
 *   createdAt: string   - ISO timestamp, tự sinh
 * }
 */
```

### Bước Red — viết test trước, chạy FAIL vì chưa có code

```javascript
// tests/createTicket.test.js
import { describe, it, expect, vi, beforeEach } from "vitest";
import crypto from "crypto";
import { createTicket } from "../src/ticket";

describe("createTicket", () => {
  let store;
  beforeEach(() => { store = []; });

  // ---- Happy path ----
  it("creates a ticket with defaults and saves it into the store", () => {
    const ticket = createTicket({ title: "Fix login bug" }, store);
    expect(ticket).toMatchObject({
      title: "Fix login bug",
      description: "",
      status: "open",
      priority: "medium",
    });
    expect(ticket.id).toBeDefined();
    expect(ticket.createdAt).toBeDefined();
    expect(store).toHaveLength(1);
    expect(store[0]).toBe(ticket);
  });

  it("uses an internal default store when none is provided", () => {
    expect(() => createTicket({ title: "Fix login bug" })).not.toThrow();
  });

  // ---- Field bắt buộc (title): thiếu / rỗng / chỉ khoảng trắng — 3 case riêng biệt ----
  it("rejects a missing title", () => {
    expect(() => createTicket({}, store)).toThrow("Title is required");
  });
  it("rejects an empty title", () => {
    expect(() => createTicket({ title: "" }, store)).toThrow("Title is required");
  });
  it("rejects a whitespace-only title", () => {
    expect(() => createTicket({ title: "   " }, store)).toThrow("Title is required");
  });

  // ---- Giới hạn độ dài (boundary testing) ----
  it("accepts a title of exactly 200 characters (boundary)", () => {
    const ticket = createTicket({ title: "a".repeat(200) }, store);
    expect(ticket.title).toHaveLength(200);
  });
  it("rejects a title longer than 200 characters (boundary)", () => {
    expect(() => createTicket({ title: "a".repeat(201) }, store))
      .toThrow("Title must not exceed 200 characters");
  });
  it("accepts a description of exactly 2000 characters (boundary)", () => {
    const ticket = createTicket({ title: "Task", description: "a".repeat(2000) }, store);
    expect(ticket.description).toHaveLength(2000);
  });
  it("rejects a description longer than 2000 characters (boundary)", () => {
    expect(() => createTicket({ title: "Task", description: "a".repeat(2001) }, store))
      .toThrow("Description must not exceed 2000 characters");
  });
  it("rejects a description that is not a string", () => {
    expect(() => createTicket({ title: "Task", description: 12345 }, store))
      .toThrow("Description must be a string");
  });

  // ---- ID: bỏ qua id client truyền, luôn sinh mới, không trùng ----
  it("ignores any client-supplied id and always generates a new one", () => {
    const ticket = createTicket({ title: "A", id: "fake-id-123" }, store);
    expect(ticket.id).not.toBe("fake-id-123");
  });
  it("generates different ids for two consecutive tickets", () => {
    const t1 = createTicket({ title: "A" }, store);
    const t2 = createTicket({ title: "B" }, store);
    expect(t1.id).not.toBe(t2.id);
  });
  it("throws when the generated id collides with an existing ticket (rare case, mocked)", () => {
    store.push({ id: "duplicate-id", title: "Old" });
    vi.spyOn(crypto, "randomUUID").mockReturnValueOnce("duplicate-id");
    expect(() => createTicket({ title: "New" }, store)).toThrow("Ticket ID collision, please retry");
  });

  // ---- priority: chỉ nhận enum hợp lệ ----
  it("rejects a priority outside the allowed list", () => {
    expect(() => createTicket({ title: "A", priority: "urgent" }, store))
      .toThrow("Invalid priority: urgent");
  });
  it("accepts a valid priority supplied by the client", () => {
    const ticket = createTicket({ title: "A", priority: "high" }, store);
    expect(ticket.priority).toBe("high");
  });

  // ---- status luôn cố định, không thể bị ghi đè ----
  it("always sets status to 'open', ignoring any status field passed in", () => {
    const ticket = createTicket({ title: "A", status: "done" }, store);
    expect(ticket.status).toBe("open");
  });
});
```

*(Đây chính là bộ test được xây dựng qua nhiều vòng lặp Iterative Refinement thật — xem chi tiết quá trình phát hiện từng edge case ở [`evidence-layered-questioning-tdd.md`](evidence-layered-questioning-tdd.md) và [`evidence-iterative-refinement.md`](evidence-iterative-refinement.md).)*

### Bước Green — viết code để tất cả test PASS (đã bao gồm Refactor: tách riêng từng hàm `validate*`)

```javascript
// src/ticket.js
import crypto from "crypto";

const MAX_TITLE_LENGTH = 200;
const MAX_DESCRIPTION_LENGTH = 2000;
const VALID_PRIORITIES = ["low", "medium", "high"];

function validateTitle(title) {
  const trimmed = (title || "").trim();
  if (!trimmed) throw new Error("Title is required");
  if (trimmed.length > MAX_TITLE_LENGTH) {
    throw new Error(`Title must not exceed ${MAX_TITLE_LENGTH} characters`);
  }
  return trimmed;
}

function validateDescription(description) {
  if (description === undefined) return "";
  if (typeof description !== "string") throw new Error("Description must be a string");
  if (description.length > MAX_DESCRIPTION_LENGTH) {
    throw new Error(`Description must not exceed ${MAX_DESCRIPTION_LENGTH} characters`);
  }
  return description;
}

function validatePriority(priority) {
  const value = priority || "medium";
  if (!VALID_PRIORITIES.includes(value)) throw new Error(`Invalid priority: ${value}`);
  return value;
}

export function createTicket(data, store = []) {
  const title = validateTitle(data.title);
  const description = validateDescription(data.description);
  const priority = validatePriority(data.priority);

  const id = crypto.randomUUID(); // luôn tự sinh, bỏ qua data.id nếu có
  if (store.some((t) => t.id === id)) {
    throw new Error("Ticket ID collision, please retry");
  }

  const ticket = {
    id,
    title,
    description,
    status: "open", // luôn cố định, bỏ qua data.status nếu có
    priority,
    createdAt: new Date().toISOString(),
  };
  store.push(ticket);
  return ticket;
}
```

**Refactor đã áp dụng:** tách `validateTitle` / `validateDescription` / `validatePriority` thành 3 hàm riêng thay vì gộp hết logic validate vào 1 hàm `createTicket` — dễ đọc, dễ mở rộng khi thêm field mới, và mỗi hàm có thể unit test độc lập nếu cần.

### Integration test — kiểm tra lưu trữ file thật (temp dir)

```javascript
// tests/storage.integration.test.js
import { describe, it, expect, beforeEach, afterEach } from "vitest";
import { mkdtempSync, rmSync, writeFileSync } from "fs";
import { tmpdir } from "os";
import path from "path";
import { saveTicket, loadTickets } from "../src/storage";

describe("ticket storage (integration)", () => {
  let dir;
  beforeEach(() => { dir = mkdtempSync(path.join(tmpdir(), "tickets-")); });
  afterEach(() => rmSync(dir, { recursive: true, force: true }));

  it("persists a ticket to disk and reloads it", () => {
    const file = path.join(dir, "tickets.json");
    saveTicket(file, { id: "1", title: "Fix bug", status: "open" });
    const tickets = loadTickets(file);
    expect(tickets).toHaveLength(1);
    expect(tickets[0].title).toBe("Fix bug");
  });

  it("throws a clear error when the JSON file is corrupted", () => {
    const file = path.join(dir, "tickets.json");
    writeFileSync(file, "{ not valid json");
    expect(() => loadTickets(file)).toThrow(/invalid|corrupt/i);
  });
});
```

### End-to-End test — chạy CLI thật qua subprocess

```javascript
// tests/cli.e2e.test.js
import { describe, it, expect } from "vitest";
import { execa } from "execa";

describe("ticket CLI (e2e)", () => {
  it("creates a ticket via the command line", async () => {
    const { stdout, exitCode } = await execa("node", ["cli.js", "create", "--title", "Fix bug"]);
    expect(exitCode).toBe(0);
    expect(stdout).toContain("Ticket created");
  });

  it("exits with a non-zero code for an unknown ticket id", async () => {
    const { exitCode, stderr } = await execa(
      "node", ["cli.js", "show", "--id", "does-not-exist"], { reject: false }
    );
    expect(exitCode).not.toBe(0);
    expect(stderr).toMatch(/not found/i);
  });
});
```

---

## 6. AI Validation — Test giúp kiểm chứng code do AI sinh ra

AI có thể viết code "trông đúng" — cú pháp sạch, tên biến hợp lý — nhưng vẫn sai logic, thiếu edge case, hoặc dựa trên giả định sai. Vì không tự viết từng dòng, người dùng dễ bỏ sót lỗi nếu chỉ đọc bằng mắt. Test giải quyết vấn đề này theo các cách sau:

- **Test là "hợp đồng khách quan":** viết test dựa trên yêu cầu thực tế (độc lập với code AI vừa viết). Nếu code AI sinh ra pass hết → có bằng chứng cụ thể, không phải "cảm thấy đúng".
- **Bắt lỗi logic ẩn:** ví dụ AI quên xử lý title rỗng — test "should reject empty title" sẽ fail ngay, lộ ra lỗi mà đọc code có thể bỏ sót.
- **Phát hiện hallucination:** AI có thể giả định sai cách một hàm/thư viện hoạt động — chạy test thật (không phải suy luận) sẽ báo lỗi ngay.
- **Vòng lặp AI tự sửa:** đưa AI xem test đang fail giúp AI tự sửa chính xác hơn là mô tả lại bằng lời.
- **Test là tài liệu sống:** sau nhiều vòng AI sửa code, test cho biết hành vi hiện tại thực sự là gì, không cần đọc lại toàn bộ code.

**Minh hoạ:** giả sử AI viết `createTicket()` nhưng test chỉ có assertion yếu (`toBeTruthy()`) và thiếu case title rỗng. Sau khi review, phát hiện thiếu sót, và sửa lại (thêm assertion chặt + test title rỗng — xem mục 5), phiên bản mới bắt được đúng lỗi thiếu validation — chứng minh test đã "kiểm soát" chất lượng code AI sinh ra thay vì chỉ tin tưởng mù quáng.

---

## 7. Các lỗi thường gặp khi test và cách tránh

**1) Over-testing (test quá nhiều/thừa)**

Viết test cho những thứ không cần thiết: getter/setter đơn giản, code của thư viện bên thứ 3, hoặc lặp lại cùng 1 logic theo nhiều cách không thêm giá trị. Hậu quả: test suite chạy chậm, khó bảo trì. Cách tránh: tập trung vào hành vi quan trọng (business logic, edge case) thay vì cố đạt 100% coverage.

**2) Weak assertions (assertion yếu)**

```javascript
// Sai — assertion yếu, pass dù kết quả sai
expect(ticket).toBeTruthy();

// Đúng — kiểm tra giá trị/cấu trúc cụ thể
expect(ticket).toEqual({ id: expect.any(String), title: "Fix bug", status: "open" });
```

Assertion yếu khiến test "xanh" (pass) ngay cả khi kết quả sai — tạo cảm giác an toàn giả. Cách tránh: luôn kiểm tra giá trị/cấu trúc cụ thể, không chỉ kiểm tra "có tồn tại".

**3) Testing implementation details (test chi tiết cài đặt nội bộ)**

```javascript
// Sai — test cách hàm thực hiện (internal), không phải kết quả
expect(jsonStringifySpy).toHaveBeenCalledTimes(1);

// Đúng — test kết quả quan sát được từ bên ngoài
expect(readFileSync(path)).toContain('"title":"Fix bug"');
```

Test cách hàm thực hiện thay vì test kết quả của nó. Hậu quả: chỉ cần refactor (không đổi hành vi) là test fail hàng loạt dù code không có bug — làm mất tác dụng "lưới an toàn" khi refactor. Cách tránh: test qua public interface (input → output quan sát được).

**4) Blindly trusting AI output (tin tưởng mù quáng vào AI)**

Chấp nhận code/test do AI viết mà không đọc hiểu, không tự hỏi "edge case nào chưa xử lý?". Bao gồm cả việc tin tưởng test do AI viết — AI có thể viết test khớp với chính code sai của nó, khiến test vô nghĩa. Cách tránh: luôn đọc hiểu trước khi merge, tự kiểm tra test có phản ánh đúng yêu cầu nghiệp vụ không.
