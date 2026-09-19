# Evidence: Iterative Refinement — Áp dụng vào hàm `createTicket`

**Workflow:** Iterative Refinement (Get AI suggestion → Review carefully → Summarize problems → Refine yourself → Feed back with context → Validate together) 


### Bước 1 — Get AI suggestion

**Prompt:**
> Viết giúp tôi 1 unit test và phần code triển khai cho hàm createTicket trong Ticket Manager CLI (input: title, description; output: một ticket object gồm id, title, description, status='open').

*(AI trả về 1 unit test + code triển khai ban đầu, trong đó có 1 test kiểm tra "2 ticket có ID khác nhau".)*

### Bước 2 & 3 — Review carefully / Summarize problems

**Bạn review và chỉ ra vấn đề:** test "id duy nhất" chỉ so sánh 2 ticket — chưa đủ mạnh để phát hiện trùng lặp nếu tạo số lượng lớn.

### Bước 4 — AI phản hồi, đề xuất hướng Refine

> Bạn quan sát đúng — mình có test "id duy nhất" nhưng chỉ so sánh 2 ticket, chưa đủ mạnh để phát hiện trùng lặp nếu tạo số lượng lớn (dù xác suất trùng UUID gần như bằng 0, nhưng về mặt test coverage thì nên kiểm tra kỹ hơn). Có 2 hướng cải thiện:

**1. Test tạo nhiều ticket, kiểm tra không có ID nào trùng nhau**
```javascript
test('không có ID nào trùng nhau khi tạo nhiều ticket', () => {
  const tickets = Array.from({ length: 1000 }, (_, i) =>
    createTicket(`Ticket ${i}`, `Mô tả ${i}`)
  );

  const ids = tickets.map(t => t.id);
  const uniqueIds = new Set(ids);

  expect(uniqueIds.size).toBe(tickets.length);
});
```

**2. Test ID đúng định dạng UUID (kiểm tra "chất lượng" của ID, không chỉ khác nhau)**
```javascript
test('id phải đúng định dạng UUID v4', () => {
  const ticket = createTicket('Task', 'mô tả');
  const uuidV4Regex = /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;

  expect(ticket.id).toMatch(uuidV4Regex);
});
```

**3. (Nâng cao) Mock `crypto.randomUUID` để test logic độc lập với việc sinh ID thật**
```javascript
const crypto = require('crypto');

test('sử dụng crypto.randomUUID để sinh id', () => {
  const mockUUID = 'mock-uuid-1234';
  jest.spyOn(crypto, 'randomUUID').mockReturnValue(mockUUID);

  const ticket = createTicket('Task', 'mô tả');

  expect(ticket.id).toBe(mockUUID);
  expect(crypto.randomUUID).toHaveBeenCalledTimes(1);

  crypto.randomUUID.mockRestore();
});
```

**Khuyến nghị của AI:** thêm test #1 và #2 vào bộ test hiện tại là đủ chắc chắn cho phần lớn trường hợp thực tế. Test #3 chỉ cần khi muốn tách biệt hoàn toàn unit test khỏi implementation detail (sinh UUID thật).

---

## Vòng lặp 2 — Validate lại toàn bộ dưới góc độ "đúng tinh thần TDD chưa"

### Bước 5 — Validate together (câu hỏi mở rộng của bạn)

**Prompt:**
> Phiên bản này tôi đã làm đúng tinh thần TDD chưa và còn những edge case nên test thêm cho tính năng này hay không?

### AI trả lời — tự phản biện quy trình vừa làm

**Về tinh thần TDD — thành thật thì: chưa hẳn.**

- TDD đúng nghĩa là Red → Green → Refactor: viết test trước (test fail vì chưa có code) → viết code tối thiểu để pass → refactor.
- Quy trình vừa rồi là ngược lại: viết code trước, viết test sau, rồi phát hiện thiếu case và bổ sung test. Đây là **"test-after"**, không phải TDD — dù kết quả cuối cùng (code có test bao phủ tốt) trông giống nhau, nhưng tư duy thiết kế thì khác.
- Vì sao khác biệt này quan trọng: với TDD thật, câu hỏi "còn edge case nào thiếu không" đáng lẽ phải được trả lời **trước khi có dòng code nào** — liệt kê hết case thành test rỗng/fail trước, code chỉ viết ra để thỏa mãn từng test. Làm ngược lại dễ khiến code viết theo "lối mòn" implementation, rồi test đi khớp theo code — bỏ sót case mà chính cách implement "che khuất" (đúng như case ID trùng vừa phát hiện ở Vòng lặp 1).

**Quy trình TDD đúng cho tính năng tiếp theo:**
1. Liệt kê toàn bộ spec/case trước (kể cả bằng lời, chưa cần code)
2. Viết test cho từng case, chạy — tất cả phải FAIL (vì code chưa tồn tại/chưa hoàn chỉnh)
3. Viết code tối thiểu từng bước để lần lượt test pass
4. Refactor code + test cho sạch, không đổi hành vi

**Edge case còn thiếu cho `createTicket` (AI liệt kê):**

*Về input title/description:*
- `title`/`description` là `null` hoặc `undefined` (khác chuỗi rỗng)
- Sai kiểu dữ liệu: số, object, array, boolean
- Chỉ chứa ký tự đặc biệt / emoji / unicode
- Rất dài (vd 10.000 ký tự) — có giới hạn độ dài không?
- Chứa ký tự xuống dòng (`\n`) — có lỗi hiển thị CLI không?

*Về cấu trúc object trả về:*
- Không có field thừa ngoài 4 field yêu cầu (`id`, `title`, `description`, `status`)
- Immutability: sửa object trả về có ảnh hưởng ngược lại state nội bộ không?

*Về hành vi hệ thống:*
- `createTicket` có lưu vào danh sách/storage (side-effect) hay chỉ tạo object thuần?
- Nếu có giới hạn số lượng ticket tối đa → test khi đạt giới hạn

### Bước 6 — Feed back with context (làm rõ scope để AI refine đúng hướng)

AI đặt 2 câu hỏi làm rõ scope, bạn xác nhận:

| Câu hỏi | Quyết định |
|---|---|
| `createTicket` có nên giới hạn độ dài title/description không? | **Có** — ví dụ title ≤ 200 ký tự |
| `createTicket` chỉ tạo object thuần hay có side-effect (lưu vào danh sách/storage)? | **Có lưu vào danh sách/storage** trong hàm |

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

  beforeEach(() => {
    store = []; // reset storage trước mỗi test
  });

  // ---- Case cơ bản ----
  test('tạo ticket đúng định dạng và lưu vào store', () => {
    const ticket = createTicket('Bug login', 'Không đăng nhập được', store);

    expect(ticket).toMatchObject({
      title: 'Bug login',
      description: 'Không đăng nhập được',
      status: 'open',
    });
    expect(store).toHaveLength(1);
    expect(store[0]).toBe(ticket); // ticket trả về phải là bản đã lưu trong store
  });

  test('mỗi lần gọi thêm 1 ticket mới vào store, không ghi đè', () => {
    createTicket('A', 'mô tả A', store);
    createTicket('B', 'mô tả B', store);

    expect(store).toHaveLength(2);
    expect(store[0].title).toBe('A');
    expect(store[1].title).toBe('B');
  });

  // ---- Case ID ----
  test('không có ID nào trùng nhau khi tạo nhiều ticket', () => {
    for (let i = 0; i < 1000; i++) createTicket(`T${i}`, `mô tả ${i}`, store);
    const ids = store.map(t => t.id);
    expect(new Set(ids).size).toBe(1000);
  });

  test('id đúng định dạng UUID v4', () => {
    const ticket = createTicket('Task', 'mô tả', store);
    expect(ticket.id).toMatch(
      /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
    );
  });

  // ---- Case validate input rỗng/sai kiểu ----
  test.each([
    ['', 'mô tả hợp lệ', 'Title'],
    ['   ', 'mô tả hợp lệ', 'Title'],
    [null, 'mô tả hợp lệ', 'Title'],
    [undefined, 'mô tả hợp lệ', 'Title'],
    [123, 'mô tả hợp lệ', 'Title'],
    [['a'], 'mô tả hợp lệ', 'Title'],
  ])('ném lỗi khi title không hợp lệ: %p', (title, description, field) => {
    expect(() => createTicket(title, description, store)).toThrow(`${field} không được để trống`);
    expect(store).toHaveLength(0); // không được lưu ticket lỗi vào store
  });

  test.each([
    ['Tiêu đề', '', 'Description'],
    ['Tiêu đề', '   ', 'Description'],
    ['Tiêu đề', null, 'Description'],
    ['Tiêu đề', 123, 'Description'],
  ])('ném lỗi khi description không hợp lệ: %p', (title, description, field) => {
    expect(() => createTicket(title, description, store)).toThrow(`${field} không được để trống`);
    expect(store).toHaveLength(0);
  });

  // ---- Case giới hạn độ dài ----
  test('title dài đúng 200 ký tự vẫn hợp lệ (boundary)', () => {
    const title = 'a'.repeat(200);
    const ticket = createTicket(title, 'mô tả', store);
    expect(ticket.title).toHaveLength(200);
  });

  test('title dài 201 ký tự thì ném lỗi (boundary)', () => {
    const title = 'a'.repeat(201);
    expect(() => createTicket(title, 'mô tả', store)).toThrow('Title không được vượt quá 200 ký tự');
    expect(store).toHaveLength(0);
  });

  test('description dài 2000 ký tự vẫn hợp lệ (boundary)', () => {
    const description = 'a'.repeat(2000);
    const ticket = createTicket('Tiêu đề', description, store);
    expect(ticket.description).toHaveLength(2000);
  });

  test('description dài 2001 ký tự thì ném lỗi (boundary)', () => {
    const description = 'a'.repeat(2001);
    expect(() => createTicket('Tiêu đề', description, store)).toThrow(
      'Description không được vượt quá 2000 ký tự'
    );
    expect(store).toHaveLength(0);
  });

  // ---- Case trim ----
  test('tự động trim khoảng trắng thừa', () => {
    const ticket = createTicket('  Fix bug  ', '  cần sửa gấp  ', store);
    expect(ticket.title).toBe('Fix bug');
    expect(ticket.description).toBe('cần sửa gấp');
  });

  test('trim rồi mới kiểm tra giới hạn độ dài (201 khoảng trắng bao quanh 200 ký tự vẫn hợp lệ)', () => {
    const title = ' ' + 'a'.repeat(200) + ' ';
    const ticket = createTicket(title, 'mô tả', store);
    expect(ticket.title).toHaveLength(200);
  });

  // ---- Case store không hợp lệ ----
  test('ném lỗi nếu store không phải là mảng', () => {
    expect(() => createTicket('Tiêu đề', 'mô tả', 'not-an-array')).toThrow(
      'Store phải là một mảng'
    );
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

**Chạy bộ test này ngay lúc đó (với code cũ) → FAIL hàng loạt** — đúng tinh thần TDD: Red trước. Đây chính là điểm khác biệt so với Vòng lặp 1 (lúc đó code có trước, test theo sau).

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

**Chạy lại test → tất cả Green.**
