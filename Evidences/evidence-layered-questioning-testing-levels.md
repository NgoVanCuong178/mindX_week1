# Evidence: Layered Questioning — Các cấp độ kiểm thử (Unit / Integration / E2E)

**Workflow:** Layered Questioning (Research → Brief Feature → Code Example → Validation) — theo đúng định nghĩa trong [`slides-ai-training.md`](../../../slides-ai-training.md#workflow-layered-questioning).

**Mục đích file này:** ghi lại nguyên văn phiên hỏi-đáp thật khi nghiên cứu 3 cấp độ kiểm thử (unit/integration/e2e) — dùng làm bằng chứng cho tiêu chí nghiệm thu của [`overview.md`](overview.md):
> *"Research process with AI is tracked: Workflows applied and iterations documented"*

Tương ứng **Phần B** trong `week1-tdd-research.docx` / mục 3 trong [`week1-tdd-research.md`](week1-tdd-research.md). Đây là vòng Layered Questioning thứ 2, độc lập với vòng đầu (nguyên tắc TDD) — đúng tinh thần tách riêng mỗi chủ đề học thành 1 vòng hỏi-đáp riêng.

---

## Lượt 1 — Research (mở rộng sang khái niệm nhận biết "lẫn cấp độ")

**Prompt:**
> Tôi cần kiến thức về các cấp độ kiểm thử trong một hệ thống: unit test, integration test, end-to-end test. (định nghĩa, phạm vi, và các phương pháp của level test đó nếu có, khi nào nên dùng phương pháp nào, và các khái niệm khác cần biết trong các test level nào).

### Trả lời của AI — Dấu hiệu nhận biết test có bị "lẫn cấp độ" hay không

**1. Bộ câu hỏi tự kiểm tra nhanh**

Trước khi viết/sau khi viết xong 1 test, tự hỏi:

| Câu hỏi | Nếu trả lời... |
|---|---|
| Test này có chạm vào I/O thật không (file, DB, network, subprocess)? | Không chạm gì cả → chắc chắn là unit test, dù bạn đặt tên file là `*.integration.test.js` |
| Nếu tôi xóa hết `jest.mock(...)` trong file, test còn chạy được không? | Nếu không chạy được (vì mock đang giả toàn bộ dependency) → đây là unit test đội lốt integration |
| Test có setup/teardown file thật, temp folder, container không? | Có → đúng chất integration. Không có gì để dọn dẹp sau test → nghi ngờ là unit |
| Nếu code thật của dependency bị đổi (schema DB đổi, API đổi) mà test vẫn xanh — test này có phát hiện ra không? | Không phát hiện được → đó là unit test (vì mock không tự cập nhật theo thực tế) |

**2. Dấu hiệu cụ thể theo triệu chứng**

🚩 **Integration test nhưng mock hết như unit test**

```javascript
// Đặt tên là "integration" nhưng...
jest.mock('../src/storage'); // mock toàn bộ storage
jest.mock('../src/validate'); // mock luôn cả validate

test('integration: create ticket', () => {
  storage.writeAll.mockReturnValue(true);
  // ...
});
```

→ Đây thực chất là unit test bị đặt sai tên. Không có bất kỳ thành phần thật nào được tích hợp — chỉ đang test xem hàm có gọi đúng mock hay không, không test xem 2 module có thực sự làm việc được với nhau hay không.

**Cách sửa:** Integration test phải để ít nhất 1 lớp là thật (thường là storage/file/DB), chỉ mock các thứ ở rất xa như third-party API bên ngoài không kiểm soát được (payment gateway, email service...).

🚩 **Unit test nhưng lại đụng file/DB thật**

```javascript
test('validateTicket rejects empty title', () => {
  const fs = require('fs');
  fs.writeFileSync('./test-data.json', '...'); // ơ, sao unit test lại ghi file?
  const result = validateTicket(readFromFile());
  // ...
});
```

→ Validate logic không cần đọc file để test. Việc này làm unit test chậm, flaky, và khi fail không biết lỗi do logic validate hay do file system.

**Cách sửa:** Tách hẳn phần đọc file ra ngoài, chỉ truyền object thuần vào hàm cần test.

🚩 **E2E test nhưng lại "leo" vào kiểm tra chi tiết logic nội bộ**

```javascript
test('e2e: create ticket', () => {
  const result = runCli(['create', '--title', 'Bug A']);
  // rồi lại import trực tiếp module nội bộ để assert:
  const { validateTicket } = require('../src/validate');
  expect(validateTicket({ title: 'Bug A' }).isValid).toBe(true); // sai chỗ!
});
```

→ E2E không nên biết gì về cấu trúc code bên trong. Nó chỉ nên quan tâm input/output từ góc nhìn người dùng (stdout, exit code, file kết quả).

**Cách sửa:** E2E chỉ assert trên những gì user thấy được: output text, exit code, nội dung file cuối cùng — không import module nội bộ.

🚩 **Integration test cố gắng cover mọi input/edge case như unit test**

Triệu chứng: viết 20 integration test cho `validateTicket` với file JSON thật, test đủ mọi trường hợp title rỗng, status sai, priority sai...

→ Đây là lãng phí — các case validate logic thuần túy này chạy y hệt kết quả dù có file JSON thật hay không, nhưng chạy chậm hơn 10-50 lần vì phải đọc/ghi file mỗi lần. Nên gom hết case validate về unit test, integration chỉ cần 1-2 case chạm validate để xác nhận nó có được gọi đúng chỗ trong luồng thật.

**3. Bảng "test signature" — nhận diện nhanh qua đặc điểm**

| Đặc điểm quan sát được | Unit | Integration | E2E |
|---|---|---|---|
| Thời gian chạy 1 test | < 10ms | 10ms – vài trăm ms | vài trăm ms – vài giây |
| Có `beforeEach` tạo file/folder tạm không | Không | Có | Có |
| Có gọi subprocess/`execFileSync` không | Không | Không | Có |
| Số lượng `jest.mock()` | Nhiều, mock gần hết | Ít, chỉ mock thứ ở rìa (3rd-party) | Không mock gì (test hộp đen hoàn toàn) |
| Assert vào đâu | Giá trị trả về của hàm | Trạng thái file/DB sau khi chạy | stdout/stderr/exit code/file cuối cùng |
| Biết tên hàm/module nội bộ không | Biết rõ (white-box) | Biết một phần | Không biết gì (black-box) |
| Nếu đổi tên 1 biến nội bộ (không đổi behavior) | Có thể phải sửa test | Thường không cần sửa | Không bao giờ cần sửa |

**4. Một quy tắc ngón tay cái dễ nhớ**

> Số lượng mock tỉ lệ nghịch với cấp độ test.

- **Unit test:** mock nhiều nhất (lý tưởng là mock hết dependency ngoài đơn vị đang test)
- **Integration test:** mock ít, chỉ mock cái ngoài tầm kiểm soát (API bên thứ 3, service ngoài)
- **E2E test:** 0 mock — nếu thấy `jest.mock()` xuất hiện trong file E2E, gần như chắc chắn có gì đó sai

Nếu một ngày nhìn vào file integration test và thấy nó toàn `jest.mock()` giống hệt file unit test bên cạnh — đó chính là dấu hiệu rõ ràng nhất của việc lẫn cấp độ.

---

## Lượt 2 — Validation (chốt lại bằng bảng tổng hợp so sánh)

**Prompt:**
> Cuối cùng tôi cần bạn bảng tóm tắt lại so sánh 3 level test đó với một số các yếu tố cụ thể (tốc độ, phạm vi, khi nào nên dùng,..)

### Trả lời của AI — Bảng tóm tắt so sánh 3 cấp độ Test

| Yếu tố | Unit Test | Integration Test | End-to-End Test |
|---|---|---|---|
| Định nghĩa | Test một đơn vị code nhỏ nhất (hàm/class), cô lập hoàn toàn | Test sự tương tác giữa 2+ thành phần (module ↔ file/DB) | Test toàn bộ luồng nghiệp vụ từ góc nhìn người dùng |
| Phạm vi | Rất hẹp — 1 hàm/1 class | Vừa — ranh giới giữa các module (thường 2-3 lớp) | Rộng nhất — toàn hệ thống từ đầu vào đến đầu ra |
| Tốc độ chạy | Rất nhanh (< 10ms/test) | Trung bình (10ms – vài trăm ms) | Chậm (vài trăm ms – vài giây) |
| Số lượng trong dự án | Nhiều nhất (đáy pyramid) | Vừa phải (giữa pyramid) | Ít nhất (đỉnh pyramid) |
| Mock/Stub dùng | Mock/stub hầu hết dependency | Chỉ mock thứ ngoài tầm kiểm soát (3rd-party API) | Không mock (black-box hoàn toàn) |
| Đụng I/O thật (file/DB/network) | Không | Có (file/DB thật, nhưng môi trường cô lập/tạm) | Có (chạy như production thật) |
| Độ tin cậy / flaky | Rất ổn định, hiếm flaky | Ổn định vừa phải | Dễ flaky nhất (phụ thuộc timing, network, UI render) |
| Chi phí viết & bảo trì | Thấp | Trung bình | Cao (dễ vỡ khi UI/luồng đổi) |
| Ai thường viết | Developer (khi code) | Developer / QA | QA / Developer (ít hơn) |
| Phát hiện loại lỗi | Bug logic trong hàm (validate sai, tính toán sai) | Sai kết nối giữa module (schema đổi, API contract vỡ, ghi/đọc file sai) | Sai luồng nghiệp vụ thực tế, trải nghiệm người dùng bị hỏng |
| Góc nhìn (white-box/black-box) | White-box — biết rõ code nội bộ | Xám (grey-box) — biết một phần | Black-box — không biết code bên trong |
| Khi nào nên dùng | Logic nghiệp vụ, validate, thuật toán, hàm tính toán, các nhánh điều kiện | Khi có nhiều module/service giao tiếp, đọc-ghi DB/file, kiểm tra "khớp nối" | Luồng quan trọng nhất (critical path): checkout, đăng nhập, luồng chính của CLI/app |
| Ví dụ cụ thể (Ticket CLI) | `validateTicket()`, merge logic của update, filter logic của list | `createTicket()` ghi đúng vào file JSON thật, `showTicket()` đọc đúng theo ID | Chạy `ticketcli create → list → update → show` qua subprocess thật |
| Chạy khi nào trong CI/CD | Mỗi lần commit/PR (rất nhanh) | Mỗi lần PR hoặc trước khi merge | Trước khi release / trên staging, không cần chạy mọi commit |

**Ghi nhớ nhanh (rule of thumb):**
```
Unit         →  Nhiều nhất, nhanh nhất, mock nhiều nhất, test "1 hàm có đúng không"
Integration  →  Vừa phải, mock ít, test "2 module ghép lại có chạy đúng không"
E2E          →  Ít nhất, chậm nhất, không mock, test "người dùng dùng có được không"
```

**Tỷ lệ tham khảo phổ biến (Testing Pyramid):** khoảng 70% Unit – 20% Integration – 10% E2E, tùy vào độ phức tạp và rủi ro của hệ thống.

---

## Tóm tắt mapping với Layered Questioning

| Bước trong slide | Xảy ra ở đâu trong phiên này |
|---|---|
| 1. Research | Lượt 1 — hỏi định nghĩa/phạm vi/phương pháp/khi nào dùng của 3 cấp độ test |
| 2. Brief Feature | Ngầm định trong câu hỏi ("khái niệm khác cần biết trong test level") — không có lượt riêng tách bạch trong phiên này |
| 3. Code Example | Ví dụ code minh họa 3 dấu hiệu 🚩 (integration mock quá tay, unit đụng file thật, e2e leo vào nội bộ) — đóng vai trò code example cho việc *nhận diện* đúng/sai cấp độ; ví dụ test code đầy đủ cho từng cấp độ đã có sẵn ở mục 5 của [`week1-tdd-research.md`](week1-tdd-research.md) |
| 4. Validation | Lượt 2 — yêu cầu bảng tổng hợp so sánh, dùng làm "validation" chốt lại toàn bộ hiểu biết về 3 cấp độ |

**Nhận xét:** phiên này không tách rõ 4 bước thành 4 lượt hỏi riêng biệt như lý thuyết — bước "Brief Feature" (gắn vào dự án cụ thể) không xuất hiện tường minh mà chỉ ngầm trong yêu cầu ban đầu. Đây là điểm có thể cải thiện nếu muốn bằng chứng bám sát 100% cấu trúc 4 bước: có thể hỏi thêm 1 lượt kiểu *"Với Ticket Manager CLI của tôi, phần nào nên là unit/integration/e2e?"* để tách bạch rõ Brief Feature khỏi Research.
