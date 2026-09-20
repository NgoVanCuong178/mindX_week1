# Evidence: Layered Questioning — Các cấp độ kiểm thử (Unit / Integration / E2E)

**Workflow:** Layered Questioning (Research → Brief Feature → Code Example → Validation)

---

## Lượt 1 — Research

**Prompt:**
> Tôi cần kiến thức về các cấp độ kiểm thử trong một hệ thống: unit test, integration test, end-to-end test. (định nghĩa, phạm vi, và các phương pháp của level test đó nếu có, khi nào nên dùng phương pháp nào, và các khái niệm khác cần biết trong các test level nào).

### Trả lời của AI (tóm tắt — bản đầy đủ dài, giữ lại các ý chính)

**Unit Test** — kiểm thử đơn vị nhỏ nhất có thể cô lập (hàm/method/class), không chạm DB/API/filesystem thật, chạy rất nhanh (mili giây). Kỹ thuật chính: Mocking (giả lập, verify được lời gọi), Stubbing (trả giá trị cố định), Faking (cài đặt giả nhưng có logic thật, vd in-memory DB), Spying (bọc quanh object thật để theo dõi lời gọi) — gọi chung là Test Doubles; cấu trúc chuẩn AAA Pattern (Arrange-Act-Assert); TDD (viết test trước); Parameterized/table-driven test (test nhiều bộ input/output cùng logic). Khái niệm liên quan: code coverage (không đồng nghĩa test tốt), test isolation, dependency injection, flaky test.

**Integration Test** — kiểm thử tương tác giữa 2+ thành phần/module khi kết hợp (vd service gọi DB thật). Phương pháp: Big Bang Integration (tích hợp tất cả cùng lúc, hệ thống nhỏ), Top-down (test từ module cấp cao xuống, dùng stub cho module con), Bottom-up (test từ module cấp thấp lên, dùng driver giả lập module cha), Sandwich/Hybrid (kết hợp cả hai, hệ thống lớn nhiều team), Contract Testing (vd Pact — test API giữa 2 service khớp nhau mà không cần chạy cả hai, dùng cho microservices), Testcontainers (dùng Docker thật cho DB/message broker khi test). Khái niệm liên quan: test double vẫn dùng cho service bên thứ 3 không kiểm soát được, in-memory database (H2, SQLite — đánh đổi tốc độ/độ chính xác), data seeding/fixtures, test environment riêng biệt.

**End-to-End Test** — kiểm thử toàn bộ luồng nghiệp vụ từ góc nhìn người dùng thật, từ UI qua backend, DB, các service liên quan. Rộng nhất, chậm nhất, dễ flaky nhất, chi phí bảo trì cao nhất. Phương pháp: UI-driven (Selenium/Playwright/Cypress — điều khiển trình duyệt thật), API-driven (test qua API, bỏ qua UI, nhanh/ổn định hơn), Record & Playback (ghi rồi phát lại thao tác, dễ hỏng khi UI đổi, không khuyến khích lâu dài), Visual regression testing (so sánh screenshot trước/sau), Synthetic monitoring (chạy E2E định kỳ trên production để giám sát uptime). Khái niệm liên quan: Page Object Model (POM), flaky test & retry logic (explicit wait thay vì sleep cứng), test data management, headless browser, smoke test (tập con nhỏ chỉ test luồng cực kỳ quan trọng sau mỗi lần deploy).

**Testing Pyramid:** Unit ở đáy (nhiều nhất, nhanh nhất, rẻ nhất) → Integration ở giữa → E2E ở đỉnh (ít nhất, chậm nhất, đắt nhất). Bảng so sánh 3 tiêu chí chính: tốc độ, độ tin cậy (ít flaky), chi phí bảo trì, loại lỗi phát hiện, khi nào ưu tiên dùng — nội dung này trùng với bảng tổng hợp ở Lượt 4 bên dưới nên không lặp lại ở đây.

---

## Lượt 2 — Hỏi thêm về nguồn tham khảo

**Prompt:**
> tôi cần thêm bạn đã lấy những kiến thức này ở trang web nào đưa tôi một số đường dẫn liên quan được không

### Trả lời của AI

AI làm rõ: câu trả lời trước được viết từ kiến thức huấn luyện sẵn (training data), không phải copy từ một trang cụ thể; sau đó đã search web để tìm nguồn uy tín xác nhận/bổ sung, đưa ra các đường dẫn:

- The Testing Pyramid — DevIQ (deviq.com): giải thích ngắn gọn 3 tầng Unit/Integration/E2E
- Testing Pyramid a Simple Way to Ensure Code Quality — dev.to: theo mô hình Testing Pyramid của Mike Cohn
- The Testing Pyramid Explained — thectoclub.com: áp dụng thực tế trong CI/CD
- Test Typology Wiki (Unit, Integration, Contract, E2E) — github.com: tổng hợp dựa trên Martin Fowler, phân biệt mockist vs classic và các loại test doubles
- Testing Strategy: Testing Pyramid — archman.dev: về Contract Testing và Testcontainers cho microservices

---

## Lượt 3 — Mở rộng: Cách nhận biết test bị "lẫn cấp độ"

**Prompt:**
> Làm sao để tôi có thể biết được mình đang viết test đúng cấp độ, không bị lẫn lộn hay không. Chẳng hạn như integration test nhưng lại mock hết mọi thứ y như unit test,... Cho tôi một vài dấu hiệu được không

### Trả lời của AI

**Bộ câu hỏi tự kiểm tra nhanh:**

| Câu hỏi | Nếu trả lời... |
|---|---|
| Test này có chạm vào I/O thật không (file, DB, network, subprocess)? | Không chạm gì cả → chắc chắn là unit test, dù đặt tên file là `*.integration.test.js` |
| Nếu xóa hết `jest.mock(...)` trong file, test còn chạy được không? | Không chạy được (mock đang giả toàn bộ dependency) → unit test đội lốt integration |
| Test có setup/teardown file thật, temp folder, container không? | Có → đúng chất integration. Không có gì dọn dẹp → nghi ngờ là unit |
| Nếu dependency thật đổi (schema DB đổi, API đổi) mà test vẫn xanh — có phát hiện ra không? | Không phát hiện được → đó là unit test (mock không tự cập nhật theo thực tế) |

**Triệu chứng cụ thể:**

*Integration test nhưng mock hết như unit test* — mock cả `storage` lẫn `validate` thì không còn gì được tích hợp thật, thực chất là unit test đặt sai tên:
```javascript
jest.mock('../src/storage');
jest.mock('../src/validate');
test('integration: create ticket', () => {
  storage.writeAll.mockReturnValue(true);
});
```
Cách sửa: integration test phải để ít nhất 1 lớp thật (storage/file/DB), chỉ mock thứ ngoài tầm kiểm soát (payment gateway, email service...).

*Unit test nhưng lại đụng file/DB thật* — validate logic không cần đọc file để test, làm vậy khiến test chậm, flaky, khó biết lỗi do logic hay do file system. Cách sửa: tách phần đọc file ra ngoài, chỉ truyền object thuần vào hàm cần test.

*E2E test nhưng "leo" vào kiểm tra chi tiết logic nội bộ* — E2E chỉ nên assert trên thứ user thấy được (stdout, exit code), không nên import module nội bộ để assert sâu như hàm `validateTicket` bên trong. Cách sửa: chỉ assert output text, exit code, file kết quả.

*Integration test cố cover mọi input/edge case như unit test* — vd viết 20 integration test cho `validateTicket` với file JSON thật là lãng phí, vì chạy chậm hơn 10-50 lần nhưng kết quả giống hệt unit test. Nên gom hết case validate thuần túy về unit, integration chỉ cần 1-2 case xác nhận validate được gọi đúng chỗ.

**Bảng "test signature" — nhận diện nhanh qua đặc điểm:**

| Đặc điểm quan sát được | Unit | Integration | E2E |
|---|---|---|---|
| Thời gian chạy 1 test | < 10ms | 10ms – vài trăm ms | vài trăm ms – vài giây |
| Có `beforeEach` tạo file/folder tạm không | Không | Có | Có |
| Có gọi subprocess không | Không | Không | Có |
| Số lượng `jest.mock()` | Nhiều, mock gần hết | Ít, chỉ mock thứ ở rìa (3rd-party) | Không mock gì |
| Assert vào đâu | Giá trị trả về của hàm | Trạng thái file/DB sau khi chạy | stdout/stderr/exit code/file cuối cùng |
| Góc nhìn code nội bộ | White-box (biết rõ) | Grey-box (biết một phần) | Black-box (không biết gì) |

**Quy tắc ngón tay cái:** số lượng mock tỉ lệ nghịch với cấp độ test — Unit mock nhiều nhất, Integration mock ít (chỉ thứ ngoài tầm kiểm soát), E2E gần như không mock gì. Nếu file integration test toàn `jest.mock()` giống hệt file unit test bên cạnh — đó là dấu hiệu rõ ràng nhất của việc lẫn cấp độ.

---

## Lượt 4 — Validation (chốt lại bằng bảng tổng hợp so sánh)

**Prompt:**
> Cuối cùng tôi cần bạn bảng tóm tắt lại so sánh 3 level test đó với một số các yếu tố cụ thể (tốc độ, phạm vi, khi nào nên dùng,..)

### Trả lời của AI — Bảng tóm tắt so sánh 3 cấp độ Test

| Yếu tố | Unit Test | Integration Test | End-to-End Test |
|---|---|---|---|
| Định nghĩa | Test một đơn vị code nhỏ nhất (hàm/class), cô lập hoàn toàn | Test sự tương tác giữa 2+ thành phần (module ↔ file/DB) | Test toàn bộ luồng nghiệp vụ từ góc nhìn người dùng |
| Phạm vi | Rất hẹp — 1 hàm/1 class | Vừa — ranh giới giữa các module | Rộng nhất — toàn hệ thống |
| Tốc độ chạy | Rất nhanh (< 10ms/test) | Trung bình (10ms – vài trăm ms) | Chậm (vài trăm ms – vài giây) |
| Số lượng trong dự án | Nhiều nhất (đáy pyramid) | Vừa phải (giữa pyramid) | Ít nhất (đỉnh pyramid) |
| Mock/Stub dùng | Mock/stub hầu hết dependency | Chỉ mock thứ ngoài tầm kiểm soát | Không mock (black-box hoàn toàn) |
| Đụng I/O thật | Không | Có (môi trường cô lập/tạm) | Có (chạy như production thật) |
| Độ tin cậy / flaky | Rất ổn định | Ổn định vừa phải | Dễ flaky nhất |
| Chi phí bảo trì | Thấp | Trung bình | Cao |
| Phát hiện loại lỗi | Bug logic trong hàm | Sai kết nối giữa module (schema, API contract) | Sai luồng nghiệp vụ, trải nghiệm người dùng |
| Góc nhìn | White-box | Grey-box | Black-box |
| Khi nào nên dùng | Logic nghiệp vụ, validate, thuật toán | Nhiều module/service giao tiếp, đọc-ghi DB/file | Luồng quan trọng nhất (critical path) |
| Ví dụ (Ticket CLI) | `validateTicket()`, merge logic update | `createTicket()` ghi vào file JSON thật | Chạy `create → list → update → show` qua subprocess |
| Chạy khi nào trong CI/CD | Mỗi commit/PR | Mỗi PR hoặc trước merge | Trước release / trên staging |

**Ghi nhớ nhanh:**
```
Unit         →  Nhiều nhất, nhanh nhất, mock nhiều nhất, test "1 hàm có đúng không"
Integration  →  Vừa phải, mock ít, test "2 module ghép lại có chạy đúng không"
E2E          →  Ít nhất, chậm nhất, không mock, test "người dùng dùng có được không"
```

**Tỷ lệ tham khảo phổ biến (Testing Pyramid):** khoảng 70% Unit – 20% Integration – 10% E2E, tùy độ phức tạp và rủi ro của hệ thống.

---
