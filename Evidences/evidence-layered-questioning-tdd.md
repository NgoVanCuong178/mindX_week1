# Evidence: Layered Questioning — Nguyên tắc TDD & Thực thể Ticket

**Workflow:** Layered Questioning (Research → Brief Feature → Code Example → Validation) — theo đúng định nghĩa trong [`slides-ai-training.md`](../../../slides-ai-training.md#workflow-layered-questioning).

**Mục đích file này:** ghi lại nguyên văn phiên hỏi-đáp thật khi nghiên cứu nguyên tắc TDD, Red-Green-Refactor, và áp dụng vào thực thể `Ticket` của Ticket Manager CLI — dùng làm bằng chứng cho tiêu chí nghiệm thu của [`overview.md`](overview.md):
> *"Research process with AI is tracked: Workflows applied and iterations documented"*

Tương ứng **Phần A** trong [`week1-tdd-research.docx`](week1-tdd-research.docx) / mục 1 trong [`week1-tdd-research.md`](week1-tdd-research.md).

---

## Lượt 1 — Research

**Prompt:**
> Tôi đang hoàn thành một bài tập mang tính học thuyết nặng và cần được những thông tin chính xác nhất. Tôi cần bạn giải thích những một số khái niệm này cho tôi:
>
> TDD (Test-Driven Development) là gì?
> Giải thích chi tiết 3 bước Red - Green - Refactor: mỗi bước làm gì, và tại sao phải theo đúng thứ tự Red trước rồi mới Green

### Trả lời của AI

**Bảng từ khóa (Keywords) cốt lõi về TDD**

**Nhóm 1: Khái niệm nền tảng**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| TDD (Test-Driven Development) | Phương pháp viết test trước code chức năng |
| Test-First | Cách gọi khác nhấn mạnh thứ tự: test → code |
| Test-After | Đối lập với TDD: code → test |
| Kent Beck | Người hệ thống hóa và phổ biến TDD, tác giả *Test-Driven Development: By Example* |
| Extreme Programming (XP) | Phong trào Agile mà TDD là một thực hành cốt lõi |
| YAGNI (You Aren't Gonna Need It) | Nguyên tắc không viết code/tính năng chưa có test/yêu cầu đòi hỏi |

**Nhóm 2: Chu trình Red-Green-Refactor**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| Red | Viết test cho hành vi chưa có, xác nhận nó fail |
| Green | Viết code tối thiểu để test pass |
| Refactor | Cải thiện code mà không đổi hành vi, có test bảo vệ |
| Fake it till you make it | Kỹ thuật ở bước Green: hard-code tạm để pass nhanh, rồi tổng quát hóa sau |
| Regression | Lỗi cũ tái xuất hiện do thay đổi mới — bộ test giúp phát hiện sớm |
| False positive (test) | Test pass dù code sai — hậu quả của việc bỏ qua bước Red |

**Nhóm 3: Phạm vi/cấp độ kiểm thử**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| Double-Loop TDD | Mô hình 2 vòng lặp: Acceptance (ngoài) bao Unit (trong) |
| Acceptance TDD (ATDD) | Test ở mức hành vi người dùng cuối, chạy chậm hơn |
| Unit TDD | Test từng hàm/class nhỏ, chạy rất nhanh |
| End-to-end test | Test toàn bộ luồng thật (ví dụ chạy CLI thật, đọc file JSON thật) |

**Nhóm 4: Trường phái thiết kế (rất quan trọng, hay bị bỏ sót)**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| Classicist / Detroit School / Chicago School | Test trạng thái (state-based), hạn chế mock, đi từ trong ra ngoài (Inside-Out) |
| Mockist / London School | Test tương tác (interaction-based), mock nhiều collaborator, đi từ ngoài vào trong (Outside-In) |
| State-based testing | Kiểm tra kết quả/trạng thái cuối cùng của object |
| Interaction-based testing | Kiểm tra object có gọi đúng phương thức/tham số đến collaborator không |
| Inside-Out | Xây từ domain logic lõi trước, ghép dần ra ngoài |
| Outside-In | Bắt đầu từ giao diện ngoài (API/CLI), mock các lớp bên dưới, đào dần vào trong |
| Freeman & Pryce | Tác giả *Growing Object-Oriented Software, Guided by Tests* — văn bản gốc của London School |

**Nhóm 5: Test double (kỹ thuật cô lập phụ thuộc)**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| Test double | Thuật ngữ chung cho mọi loại đối tượng giả dùng trong test |
| Mock | Test double có xác nhận lời gọi (assert đã được gọi thế nào) |
| Stub | Test double trả về giá trị giả định sẵn, không assert lời gọi |
| Fake | Cài đặt giả nhưng hoạt động thật (ví dụ in-memory storage thay file thật) |
| Spy | Ghi lại thông tin lời gọi để kiểm tra sau, thường kết hợp object thật |

**Nhóm 6: Lỗi thường gặp / anti-pattern**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| Over-mocking | Lạm dụng mock khiến test gắn chặt vào chi tiết cài đặt |
| Test pollution / Flaky test | Test phụ thuộc lẫn nhau hoặc cho kết quả không ổn định |
| Over-engineering (ở Green) | Viết thừa code vượt quá yêu cầu của test hiện tại |
| Goodhart's Law (áp dụng vào coverage) | Khi coverage % trở thành mục tiêu, nó ngừng phản ánh chất lượng thực |
| Mutation testing | Kỹ thuật cố tình làm hỏng code để kiểm tra test có phát hiện được không |

**Nhóm 7: Nguyên tắc thiết kế liên quan**

| Từ khóa | Ý nghĩa cốt lõi |
|---|---|
| Design pressure | Áp lực từ việc viết test trước, buộc code phải dễ test → dễ tách rời (decoupled) |
| Dependency Injection | Truyền phụ thuộc (như storage) qua tham số thay vì hard-code, giúp dễ test |
| Encapsulation | Lý do lịch sử ra đời London School — tránh phải thêm getter phá vỡ đóng gói khi test Classicist |

**Gợi ý cách ôn tập của AI:** Với 7 nhóm trên, nhóm 2 (Red-Green-Refactor) và nhóm 4 (Classicist vs Mockist) là hai phần dễ bị hỏi sâu nhất trong bài học thuật, vì chúng thể hiện được cả hiểu biết về quy trình lẫn tư duy thiết kế — hai khía cạnh giám khảo thường muốn phân biệt ở người thực sự hiểu TDD so với người chỉ thuộc "3 bước".

---

## Lượt 2 — Brief Feature → Code Example → Validation (gộp trong 1 lượt hỏi-đáp)

**Prompt:**
> Tôi vừa đọc lại code example của bạn thì tôi cần bạn bổ sung về thực thể Ticket bao gồm những attribute nào và nó sẽ có những edge case kiểm tra validation (field rỗng, ticket trùng ID, thiếu field bắt buộc phải điền,... ) chỉ cần gen example cho create ticket

*(Lượt này gắn trực tiếp vào dự án Ticket Manager CLI — đúng tinh thần Brief Feature — đồng thời yêu cầu code cụ thể — Code Example — và tập trung vào edge case validation — Validation — nên 3 bước cuối của Layered Questioning gộp tự nhiên trong 1 lượt hỏi-đáp này.)*

### Trả lời của AI

**1. Định nghĩa thực thể Ticket**

Trước khi viết test, cần chốt schema rõ ràng (đây cũng là một phần của "thiết kế qua test" — nhưng ở mức schema, ta xác định trước để tránh test rời rạc, không nhất quán):

```javascript
/**
 * Ticket entity
 * {
 *   id: string       - UUID, tự sinh, không cho phép client truyền vào
 *   title: string    - bắt buộc, không rỗng, tối đa 200 ký tự
 *   description: string - tùy chọn, mặc định ''
 *   status: string   - mặc định 'open', chỉ nhận: 'open' | 'in-progress' | 'done'
 *   priority: string - tùy chọn, mặc định 'medium', chỉ nhận: 'low' | 'medium' | 'high'
 *   createdAt: string - ISO timestamp, tự sinh
 * }
 */
```

**2. Cấu trúc file test (mở rộng dần theo edge case)**

```javascript
// ticketService.test.js
const { createTicket } = require('../src/ticketService');

function createFakeStorage(initialTickets = []) {
  let tickets = [...initialTickets];
  return {
    save: jest.fn((ticket) => tickets.push(ticket)),
    loadAll: jest.fn(() => tickets),
  };
}
```

#### Vòng lặp A: Happy path (nền tảng)

**RED**
```javascript
test('createTicket trả về ticket hợp lệ với các field mặc định', () => {
  const storage = createFakeStorage();
  const ticket = createTicket({ title: 'Fix login bug' }, storage);

  expect(ticket.id).toBeDefined();
  expect(ticket.title).toBe('Fix login bug');
  expect(ticket.status).toBe('open');
  expect(ticket.priority).toBe('medium');
  expect(ticket.description).toBe('');
  expect(ticket.createdAt).toBeDefined();
});
```

#### Vòng lặp B: Field bắt buộc bị thiếu (title)

**RED**
```javascript
test('createTicket ném lỗi khi thiếu title', () => {
  const storage = createFakeStorage();
  expect(() => createTicket({}, storage)).toThrow('Title is required');
});

test('createTicket ném lỗi khi title là chuỗi rỗng', () => {
  const storage = createFakeStorage();
  expect(() => createTicket({ title: '' }, storage)).toThrow('Title is required');
});

test('createTicket ném lỗi khi title chỉ chứa khoảng trắng', () => {
  const storage = createFakeStorage();
  expect(() => createTicket({ title: '   ' }, storage)).toThrow('Title is required');
});
```

**Lưu ý quan trọng (AI nhấn mạnh):** đây là 3 test riêng biệt dù cùng kỳ vọng lỗi giống nhau — vì mỗi test đại diện một input khác nhau cần được xác nhận độc lập (thiếu hẳn field vs rỗng vs chỉ có khoảng trắng là 3 tình huống code phải xử lý riêng, dễ bị bỏ sót nếu gộp chung).

**GREEN**
```javascript
function createTicket(data, storage) {
  const title = data.title ? data.title.trim() : '';
  if (!title) {
    throw new Error('Title is required');
  }
  const ticket = {
    id: generateId(),
    title,
    description: '',
    status: 'open',
    priority: 'medium',
    createdAt: new Date().toISOString(),
  };
  storage.save(ticket);
  return ticket;
}
```

**REFACTOR** — tách hàm validate riêng để dễ mở rộng khi thêm field khác:
```javascript
function validateTitle(title) {
  const trimmed = (title || '').trim();
  if (!trimmed) throw new Error('Title is required');
  return trimmed;
}
```

#### Vòng lặp C: Title vượt quá độ dài cho phép

**RED**
```javascript
test('createTicket ném lỗi khi title vượt quá 200 ký tự', () => {
  const storage = createFakeStorage();
  const longTitle = 'a'.repeat(201);
  expect(() => createTicket({ title: longTitle }, storage))
    .toThrow('Title must not exceed 200 characters');
});

test('createTicket chấp nhận title đúng 200 ký tự (biên hợp lệ)', () => {
  const storage = createFakeStorage();
  const maxTitle = 'a'.repeat(200);
  const ticket = createTicket({ title: maxTitle }, storage);
  expect(ticket.title).toHaveLength(200);
});
```

Đây là kỹ thuật **boundary testing** — luôn viết cặp test ở đúng ranh giới (200 = hợp lệ, 201 = không hợp lệ) thay vì chỉ test một phía, vì lỗi off-by-one (`>` thay vì `>=`) là lỗi rất phổ biến.

**GREEN**
```javascript
function validateTitle(title) {
  const trimmed = (title || '').trim();
  if (!trimmed) throw new Error('Title is required');
  if (trimmed.length > 200) throw new Error('Title must not exceed 200 characters');
  return trimmed;
}
```

#### Vòng lặp D: ID trùng lặp (duplicate ID)

Đây là edge case dễ bị bỏ sót nếu chỉ nghĩ theo hướng "id tự sinh thì sao trùng được" — nhưng cần test vì: (1) đảm bảo cơ chế sinh ID thực sự an toàn, (2) phòng trường hợp client cố tình truyền id vào input.

**RED**
```javascript
test('createTicket bỏ qua id do client truyền vào, luôn tự sinh id mới', () => {
  const storage = createFakeStorage();
  const ticket = createTicket({ title: 'A', id: 'fake-id-123' }, storage);
  expect(ticket.id).not.toBe('fake-id-123');
});

test('createTicket sinh ra 2 ticket liên tiếp có id khác nhau', () => {
  const storage = createFakeStorage();
  const t1 = createTicket({ title: 'A' }, storage);
  const t2 = createTicket({ title: 'B' }, storage);
  expect(t1.id).not.toBe(t2.id);
});

test('createTicket ném lỗi nếu id sinh ra trùng với ticket đã tồn tại trong storage', () => {
  // Giả lập tình huống hiếm: generateId() trả trùng id (ví dụ do lỗi thư viện UUID)
  const existingTicket = { id: 'duplicate-id', title: 'Old' };
  const storage = createFakeStorage([existingTicket]);

  jest.spyOn(require('../src/idGenerator'), 'generateId')
    .mockReturnValueOnce('duplicate-id');

  expect(() => createTicket({ title: 'New' }, storage))
    .toThrow('Ticket ID collision, please retry');
});
```

**GREEN**
```javascript
const { generateId } = require('./idGenerator');

function createTicket(data, storage) {
  const title = validateTitle(data.title);
  const newId = generateId(); // luôn tự sinh, bỏ qua data.id nếu có

  const existing = storage.loadAll();
  if (existing.some(t => t.id === newId)) {
    throw new Error('Ticket ID collision, please retry');
  }

  const ticket = {
    id: newId,
    title,
    description: '',
    status: 'open',
    priority: 'medium',
    createdAt: new Date().toISOString(),
  };
  storage.save(ticket);
  return ticket;
}
```

**Lưu ý thực tế (AI bình luận):** nếu dùng UUID v4, xác suất trùng gần như bằng 0 — nhiều team sẽ không viết test check-collision này vì coi là quá hiếm để đáng công sức, và thay vào đó tin tưởng vào tính chất của UUID. Đây là ví dụ tốt để đưa vào bài học thuật thảo luận: TDD không có nghĩa là test mọi thứ tưởng tượng được — cần cân nhắc chi phí/lợi ích của từng test case.

#### Vòng lặp E: `priority` hoặc `status` nhận giá trị không hợp lệ

**RED**
```javascript
test('createTicket ném lỗi khi priority không nằm trong danh sách cho phép', () => {
  const storage = createFakeStorage();
  expect(() => createTicket({ title: 'A', priority: 'urgent' }, storage))
    .toThrow('Invalid priority: urgent');
});

test('createTicket chấp nhận priority hợp lệ do client truyền vào', () => {
  const storage = createFakeStorage();
  const ticket = createTicket({ title: 'A', priority: 'high' }, storage);
  expect(ticket.priority).toBe('high');
});
```

**GREEN**
```javascript
const VALID_PRIORITIES = ['low', 'medium', 'high'];

function createTicket(data, storage) {
  const title = validateTitle(data.title);
  const priority = data.priority || 'medium';
  if (!VALID_PRIORITIES.includes(priority)) {
    throw new Error(`Invalid priority: ${priority}`);
  }
  // ... phần còn lại giữ nguyên, thêm priority vào object trả về
}
```

#### Vòng lặp F: `description` nhận input không phải string

**RED**
```javascript
test('createTicket ném lỗi khi description không phải kiểu string', () => {
  const storage = createFakeStorage();
  expect(() => createTicket({ title: 'A', description: 12345 }, storage))
    .toThrow('Description must be a string');
});
```

**GREEN**
```javascript
if (data.description !== undefined && typeof data.description !== 'string') {
  throw new Error('Description must be a string');
}
```

**Bảng tổng hợp Edge Case cho `createTicket` (dùng làm checklist khi làm bài)**

| Nhóm | Edge case | Input ví dụ | Kỳ vọng |
|---|---|---|---|
| Field bắt buộc | Thiếu title | `{}` | Throw `Title is required` |
| Field bắt buộc | title rỗng | `{title: ''}` | Throw `Title is required` |
| Field bắt buộc | title chỉ có khoảng trắng | `{title: '   '}` | Throw `Title is required` |
| Giới hạn độ dài | title > 200 ký tự | `{title: 'a'.repeat(201)}` | Throw lỗi độ dài |
| Giới hạn độ dài | title = đúng 200 ký tự (biên) | `{title: 'a'.repeat(200)}` | Pass, chấp nhận |
| ID | Client cố truyền id | `{title:'A', id:'x'}` | Bỏ qua, tự sinh id mới |
| ID | 2 ticket liên tiếp | gọi 2 lần | id khác nhau |
| ID | ID trùng trong storage (hiếm) | mock `generateId` trùng | Throw collision error |
| Enum field | priority sai giá trị | `{priority: 'urgent'}` | Throw `Invalid priority` |
| Enum field | status client truyền sai (nếu cho phép truyền) | `{status: 'archived'}` | Throw hoặc bỏ qua tùy thiết kế |
| Kiểu dữ liệu | description không phải string | `{description: 123}` | Throw `Description must be a string` |
| Kiểu dữ liệu | title không phải string (ví dụ số, null) | `{title: 123}` | Throw hoặc convert tùy thiết kế — cần quyết định rõ |

**Gợi ý khi trình bày trong bài tập (AI nhấn mạnh):** Với mỗi edge case trong bảng, nên trình bày theo đúng cấu trúc Red → Green → (Refactor nếu có) như các ví dụ trên, thay vì chỉ liệt kê danh sách — điều này thể hiện đúng đã hiểu và áp dụng quy trình, không chỉ liệt kê lý thuyết edge case suông.

---

## Tóm tắt mapping với Layered Questioning

| Bước trong slide | Xảy ra ở đâu trong phiên này |
|---|---|
| 1. Research | Lượt 1 — "TDD là gì? Red-Green-Refactor?" → AI trả lời bằng bảng khái niệm/keyword đầy đủ 7 nhóm |
| 2. Brief Feature | Lượt 2 — đưa bối cảnh cụ thể ("code example của bạn", cần gắn vào thực thể Ticket) |
| 3. Code Example | Lượt 2 — AI sinh schema `Ticket` + 6 vòng lặp Red-Green(-Refactor) thực tế cho `createTicket` |
| 4. Validation | Lượt 2 — trọng tâm câu hỏi chính là edge case validation, kết thúc bằng bảng tổng hợp edge case làm checklist |
