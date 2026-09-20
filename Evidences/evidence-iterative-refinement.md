# Evidence: Iterative Refinement — Áp dụng vào hàm `createTicket`

**Workflow:** Iterative Refinement (Get AI suggestion → Review carefully → Summarize problems → Refine yourself → Feed back with context → Validate together)

---

## Vòng lặp 1 — Phát hiện test ID chưa đủ mạnh

### Bước 1 — Get AI suggestion

**Prompt:**
> Viết giúp tôi 1 unit test và phần code triển khai cho hàm createTicket trong Ticket Manager CLI (input: title, description; output: một ticket object gồm id, title, description, status='open').

**Trả lời của AI:**

```javascript
// ticketManager.js
const crypto = require('crypto');

function createTicket(title, description) {
  if (typeof title !== 'string' || title.trim() === '') {
    throw new Error('Title không được để trống');
  }
  if (typeof description !== 'string' || description.trim() === '') {
    throw new Error('Description không được để trống');
  }
  return {
    id: crypto.randomUUID(),
    title: title.trim(),
    description: description.trim(),
    status: 'open',
  };
}

module.exports = { createTicket };
```

```javascript
// ticketManager.test.js (Jest)
const { createTicket } = require('./ticketManager');

describe('createTicket', () => {
  test('tạo ticket với đầy đủ thông tin đúng định dạng', () => {
    const ticket = createTicket('Bug login', 'Không đăng nhập được trên Safari');
    expect(ticket).toHaveProperty('id');
    expect(ticket.title).toBe('Bug login');
    expect(ticket.description).toBe('Không đăng nhập được trên Safari');
    expect(ticket.status).toBe('open');
  });

  test('mỗi ticket có id duy nhất', () => {
    const t1 = createTicket('A', 'mô tả A');
    const t2 = createTicket('B', 'mô tả B');
    expect(t1.id).not.toBe(t2.id);
  });

  test('tự động trim khoảng trắng thừa ở title/description', () => {
    const ticket = createTicket('  Fix bug  ', '  cần sửa gấp  ');
    expect(ticket.title).toBe('Fix bug');
    expect(ticket.description).toBe('cần sửa gấp');
  });

  test('ném lỗi khi title rỗng', () => {
    expect(() => createTicket('', 'mô tả hợp lệ')).toThrow('Title không được để trống');
  });

  test('ném lỗi khi title chỉ có khoảng trắng', () => {
    expect(() => createTicket('   ', 'mô tả hợp lệ')).toThrow('Title không được để trống');
  });

  test('ném lỗi khi description rỗng', () => {
    expect(() => createTicket('Tiêu đề', '')).toThrow('Description không được để trống');
  });

  test('ném lỗi khi title không phải string', () => {
    expect(() => createTicket(123, 'mô tả')).toThrow('Title không được để trống');
  });

  test('status luôn mặc định là "open"', () => {
    expect(createTicket('Task mới', 'Chi tiết task').status).toBe('open');
  });
});
```

AI dùng `crypto.randomUUID()` (built-in Node.js ≥ 14.17) để sinh id, không cần thư viện ngoài. AI cũng lưu ý: nếu CLI không cần strict validate có thể bỏ phần throw Error, và có thể đổi sang TypeScript/Mocha/Vitest nếu cần.

### Bước 2 & 3 — Review carefully / Summarize problems

**Prompt:**
> tôi thấy test chưa có test cho phần dupicate ID có đúng như vậy không

### Bước 4 — AI phản hồi, đề xuất hướng Refine

> Bạn quan sát đúng — mình có test "id duy nhất" nhưng chỉ so sánh 2 ticket, chưa đủ mạnh để phát hiện trùng lặp nếu tạo số lượng lớn (dù xác suất trùng UUID gần như bằng 0, nhưng về mặt test coverage thì nên kiểm tra kỹ hơn). Có 2 hướng cải thiện:

**1. Test tạo nhiều ticket, kiểm tra không có ID nào trùng nhau**
```javascript
test('không có ID nào trùng nhau khi tạo nhiều ticket', () => {
  const tickets = Array.from({ length: 1000 }, (_, i) =>
    createTicket(`Ticket ${i}`, `Mô tả ${i}`)
  );
  const ids = tickets.map(t => t.id);
  expect(new Set(ids).size).toBe(tickets.length);
});
```

**2. Test ID đúng định dạng UUID (kiểm tra "chất lượng" của ID, không chỉ khác nhau)**
```javascript
test('id phải đúng định dạng UUID v4', () => {
  const ticket = createTicket('Task', 'mô tả');
  expect(ticket.id).toMatch(
    /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
  );
});
```

**3. (Nâng cao) Mock `crypto.randomUUID` để test logic độc lập với việc sinh ID thật**
```javascript
test('sử dụng crypto.randomUUID để sinh id', () => {
  jest.spyOn(crypto, 'randomUUID').mockReturnValue('mock-uuid-1234');
  const ticket = createTicket('Task', 'mô tả');
  expect(ticket.id).toBe('mock-uuid-1234');
  crypto.randomUUID.mockRestore();
});
```

**Khuyến nghị của AI:** thêm test #1 và #2 vào bộ test hiện tại là đủ chắc chắn cho phần lớn trường hợp thực tế. Test #3 chỉ cần khi muốn tách biệt hoàn toàn unit test khỏi implementation detail (sinh UUID thật). AI hỏi thêm: "Bạn có muốn mình gộp lại thành file test hoàn chỉnh (đã update) không?"

---

## Vòng lặp 2 — Validate lại toàn bộ dưới góc độ "đúng tinh thần TDD chưa"

### Bước 5 — Validate together

**Prompt:**
> Phiên bản này tôi đã làm đúng tinh thần TDD chưa và còn những edge case nên test thêm cho tính năng này hay không?

### AI trả lời — tự phản biện quy trình vừa làm

**Về tinh thần TDD — thành thật thì: chưa hẳn.**

- TDD đúng nghĩa là Red → Green → Refactor: viết test trước (test fail vì chưa có code) → viết code tối thiểu để pass → refactor.
- Quy trình vừa rồi là ngược lại: viết code trước, viết test sau, rồi phát hiện thiếu case và bổ sung test. Đây là "test-after", không phải TDD — dù kết quả cuối cùng (code có test bao phủ tốt) trông giống nhau, nhưng tư duy thiết kế thì khác.
- Vì sao khác biệt này quan trọng: với TDD thật, câu hỏi "còn edge case nào thiếu không" đáng lẽ phải được trả lời trước khi có dòng code nào — liệt kê hết case thành test rỗng/fail trước, code chỉ viết ra để thỏa mãn từng test. Làm ngược lại dễ khiến code viết theo "lối mòn" implementation, rồi test đi khớp theo code — bỏ sót case mà chính cách implement "che khuất" (đúng như case ID trùng vừa phát hiện ở Vòng lặp 1).

**Quy trình TDD đúng cho tính năng tiếp theo:**
1. Liệt kê toàn bộ spec/case trước (kể cả bằng lời, chưa cần code)
2. Viết test cho từng case, chạy — tất cả phải FAIL (vì code chưa tồn tại/chưa hoàn chỉnh)
3. Viết code tối thiểu từng bước để lần lượt test pass
4. Refactor code + test cho sạch, không đổi hành vi

**Edge case còn thiếu cho `createTicket` (AI liệt kê):**

*Về input title/description:* `title`/`description` là `null`/`undefined` (khác chuỗi rỗng); sai kiểu dữ liệu (số, object, array, boolean); chỉ chứa ký tự đặc biệt/emoji/unicode; rất dài (vd 10.000 ký tự) — có giới hạn độ dài không; chứa ký tự xuống dòng (`\n`) — có lỗi hiển thị CLI không.

*Về cấu trúc object trả về:* không có field thừa ngoài 4 field yêu cầu (`id`, `title`, `description`, `status`); immutability — sửa object trả về có ảnh hưởng ngược lại state nội bộ không.

*Về hành vi hệ thống:* `createTicket` có lưu vào danh sách/storage (side-effect) hay chỉ tạo object thuần; nếu có giới hạn số lượng ticket tối đa thì cần test khi đạt giới hạn.

### Bước 6 — Feed back with context

AI đặt 2 câu hỏi làm rõ scope, bạn xác nhận:

| Câu hỏi | Quyết định |
|---|---|
| `createTicket` có nên giới hạn độ dài title/description không? | Có — ví dụ title ≤ 200 ký tự |
| `createTicket` chỉ tạo object thuần hay có side-effect (lưu vào danh sách/storage)? | Có lưu vào danh sách/storage trong hàm |

---

## Refine lần cuối — Thiết kế lại đúng tinh thần TDD (Red trước, Green sau)

> Rõ rồi. Với 2 quyết định này, mình sẽ thiết kế lại theo đúng tinh thần TDD: liệt kê spec → viết test trước (fail) → viết code sau. Mình đưa ra giả định: description giới hạn 2000 ký tự (vì bạn chưa chốt) — bạn có thể chỉnh số này.

### Bước 1 — Spec cập nhật

- `createTicket(title, description, store)` — `store` là mảng chứa danh sách ticket (dependency injection để dễ test, mặc định mảng nội bộ nếu không truyền)
- Validate `title`/`description` không rỗng, đúng kiểu string
- `title` tối đa 200 ký tự, `description` tối đa 2000 ký tự
- Tạo ticket `{ id, title, description, status: 'open' }`
- Đẩy ticket vào `store` (side-effect) rồi trả về ticket đó

### Bước 2 — Viết test trước (Red — sẽ FAIL vì code chưa đổi)

```javascript
const { createTicket } = require('./ticketManager');

describe('createTicket - TDD', () => {
  let store;
  beforeEach(() => { store = []; });

  // ---- Case cơ bản ----
  test('tạo ticket đúng định dạng và lưu vào store', () => {
    const ticket = createTicket('Bug login', 'Không đăng nhập được', store);
    expect(ticket).toMatchObject({ title: 'Bug login', description: 'Không đăng nhập được', status: 'open' });
    expect(store).toHaveLength(1);
    expect(store[0]).toBe(ticket);
  });

  test('mỗi lần gọi thêm 1 ticket mới vào store, không ghi đè', () => {
    createTicket('A', 'mô tả A', store);
    createTicket('B', 'mô tả B', store);
    expect(store).toHaveLength(2);
  });

  // ---- Case ID ----
  test('không có ID nào trùng nhau khi tạo nhiều ticket', () => {
    for (let i = 0; i < 1000; i++) createTicket(`T${i}`, `mô tả ${i}`, store);
    const ids = store.map(t => t.id);
    expect(new Set(ids).size).toBe(1000);
  });

  test('id đúng định dạng UUID v4', () => {
    const ticket = createTicket('Task', 'mô tả', store);
    expect(ticket.id).toMatch(/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i);
  });

  // ---- Case validate input rỗng/sai kiểu (title) ----
  test.each([
    ['', 'mô tả hợp lệ'],
    ['   ', 'mô tả hợp lệ'],
    [null, 'mô tả hợp lệ'],
    [undefined, 'mô tả hợp lệ'],
  ])('ném lỗi khi title không hợp lệ: %p', (title, description) => {
    expect(() => createTicket(title, description, store)).toThrow('Title không được để trống');
    expect(store).toHaveLength(0);
  });
  // (và tương tự cho title là số / mảng — cùng logic throw)

  // ---- Case validate description ----
  test.each([
    ['Tiêu đề', ''],
    ['Tiêu đề', null],
  ])('ném lỗi khi description không hợp lệ: %p', (title, description) => {
    expect(() => createTicket(title, description, store)).toThrow('Description không được để trống');
  });

  // ---- Case giới hạn độ dài (boundary) ----
  test('title dài đúng 200 ký tự vẫn hợp lệ (boundary)', () => {
    const ticket = createTicket('a'.repeat(200), 'mô tả', store);
    expect(ticket.title).toHaveLength(200);
  });

  test('title dài 201 ký tự thì ném lỗi (boundary)', () => {
    expect(() => createTicket('a'.repeat(201), 'mô tả', store))
      .toThrow('Title không được vượt quá 200 ký tự');
  });

  test('description dài 2000 ký tự vẫn hợp lệ, 2001 thì ném lỗi (boundary)', () => {
    expect(createTicket('Tiêu đề', 'a'.repeat(2000), store).description).toHaveLength(2000);
    expect(() => createTicket('Tiêu đề', 'a'.repeat(2001), store))
      .toThrow('Description không được vượt quá 2000 ký tự');
  });

  // ---- Case trim ----
  test('tự động trim khoảng trắng thừa', () => {
    const ticket = createTicket('  Fix bug  ', '  cần sửa gấp  ', store);
    expect(ticket.title).toBe('Fix bug');
  });

  // ---- Case store không hợp lệ ----
  test('ném lỗi nếu store không phải là mảng', () => {
    expect(() => createTicket('Tiêu đề', 'mô tả', 'not-an-array')).toThrow('Store phải là một mảng');
  });

  test('nếu không truyền store, dùng store nội bộ mặc định và không lỗi', () => {
    expect(() => createTicket('Tiêu đề', 'mô tả')).not.toThrow();
  });

  // ---- Case status luôn cố định ----
  test('status luôn là "open", không thể ghi đè dù có truyền field lạ', () => {
    const ticket = createTicket('Task', 'mô tả', store);
    expect(ticket.status).toBe('open');
    expect(Object.keys(ticket).sort()).toEqual(['description', 'id', 'status', 'title']);
  });
});
```

Chạy bộ test này ngay lúc đó (với code cũ) → FAIL hàng loạt — đúng tinh thần TDD: Red trước. Đây chính là điểm khác biệt so với Vòng lặp 1 (lúc đó code có trước, test theo sau).

### Bước 3 — Code triển khai để làm Green

```javascript
const crypto = require('crypto');

const MAX_TITLE_LENGTH = 200;
const MAX_DESCRIPTION_LENGTH = 2000;

function createTicket(title, description, store = []) {
  if (!Array.isArray(store)) {
    throw new Error('Store phải là một mảng');
  }
  if (typeof title !== 'string' || title.trim() === '') {
    throw new Error('Title không được để trống');
  }
  if (typeof description !== 'string' || description.trim() === '') {
    throw new Error('Description không được để trống');
  }

  const trimmedTitle = title.trim();
  const trimmedDescription = description.trim();

  if (trimmedTitle.length > MAX_TITLE_LENGTH) {
    throw new Error(`Title không được vượt quá ${MAX_TITLE_LENGTH} ký tự`);
  }
  if (trimmedDescription.length > MAX_DESCRIPTION_LENGTH) {
    throw new Error(`Description không được vượt quá ${MAX_DESCRIPTION_LENGTH} ký tự`);
  }

  const ticket = {
    id: crypto.randomUUID(),
    title: trimmedTitle,
    description: trimmedDescription,
    status: 'open',
  };

  store.push(ticket);
  return ticket;
}

module.exports = { createTicket, MAX_TITLE_LENGTH, MAX_DESCRIPTION_LENGTH };
```

Chạy lại test → tất cả Green.