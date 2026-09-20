# Evidence: Layered Questioning — Nguyên tắc TDD & Thực thể Ticket

**Workflow:** Layered Questioning (Research → Brief Feature → Code Example → Validation) 

---

## Lượt 1 — Research

**Prompt:**
> Tôi đang hoàn thành một bài tập mang tính học thuyết nặng và cần được những thông tin chính xác nhất. Tôi cần bạn giải thích những một số khái niệm này cho tôi:
>
> TDD (Test-Driven Development) là gì?
> Giải thích chi tiết 3 bước Red - Green - Refactor: mỗi bước làm gì, và tại sao phải theo đúng thứ tự Red trước rồi mới Green

TDD (Test-Driven Development)
1. TDD là gì?
TDD là một phương pháp phát triển phần mềm, trong đó bài kiểm thử (test) được viết trước khi viết code chức năng (production code). Thay vì viết code xong rồi mới test, lập trình viên đảo ngược quy trình: xác định trước hành vi mong muốn bằng một test, sau đó viết code tối thiểu để test đó vượt qua.

Về bản chất, TDD không chỉ là "kỹ thuật kiểm thử" mà còn là một kỹ thuật thiết kế: việc viết test trước buộc lập trình viên phải suy nghĩ rõ ràng về interface, hành vi và yêu cầu của đoạn code trước khi hiện thực nó, giúp code có thiết kế sạch hơn, dễ kiểm thử hơn và ít lỗi hơn.

TDD được Kent Beck hệ thống hóa và phổ biến rộng rãi trong phong trào Extreme Programming (XP).

2. Chu trình Red - Green - Refactor
Bước 1: RED (Đỏ)
Viết một test case cho một chức năng chưa tồn tại hoặc chưa được hiện thực.
Chạy test này — nó phải thất bại (fail), thường hiển thị màu đỏ trên các công cụ test runner.
Mục đích: xác nhận rằng test thực sự đang kiểm tra đúng điều cần kiểm tra, và chức năng đó chưa có sẵn (tránh trường hợp test "giả vờ pass" mà không kiểm tra gì cả).
Bước 2: GREEN (Xanh)
Viết lượng code tối thiểu cần thiết để test vượt qua (pass), hiển thị màu xanh.
Ở bước này, code có thể chưa đẹp, chưa tối ưu, thậm chí "hard-code" — điều đó chấp nhận được, vì mục tiêu duy nhất là làm test pass.
Mục đích: đảm bảo có một giải pháp hoạt động đúng, được xác nhận bằng test.
Bước 3: REFACTOR (Tái cấu trúc)
Cải thiện chất lượng code (đặt tên rõ ràng, loại bỏ trùng lặp, tối ưu cấu trúc...) mà không thay đổi hành vi của nó.
Sau mỗi thay đổi nhỏ, chạy lại test để đảm bảo vẫn pass (vẫn "xanh").
Mục đích: có được code sạch, dễ bảo trì, trong khi vẫn giữ được sự an toàn nhờ bộ test đã có.

Sau đó, chu trình lặp lại từ đầu với một test mới cho hành vi tiếp theo.

3. Tại sao phải theo đúng thứ tự Red → Green?

Đây là điểm cốt lõi và hay bị hiểu sai nếu bỏ qua. Một số lý do chính:

a) Xác thực tính hiệu lực của test (Validate the test itself)
Nếu bạn viết code trước rồi mới viết test, và test pass ngay từ đầu, bạn không thể chắc chắn rằng test đó thực sự kiểm tra đúng điều cần kiểm tra — có thể test có lỗi logic, hoặc pass "một cách tình cờ" (ví dụ assertion sai, test không gọi đúng hàm...). Bước RED buộc bạn phải thấy test fail trước, để chứng minh rằng: test có khả năng phát hiện lỗi khi chức năng chưa đúng/chưa tồn tại.

b) Tránh test "giả" (false positive)
Một test luôn pass dù code đúng hay sai là một test vô dụng và nguy hiểm — nó tạo cảm giác an toàn giả tạo. Việc bắt buộc thấy RED trước đảm bảo bạn loại trừ được trường hợp này ngay từ đầu.

c) Định hình thiết kế trước khi hiện thực (Design-first thinking)
Khi viết test trước (ở trạng thái RED), bạn đang đóng vai người dùng của đoạn code đó — buộc phải nghĩ về API, tên hàm, tham số đầu vào/đầu ra một cách rõ ràng trước khi bị "cuốn" vào chi tiết hiện thực. Nếu viết code trước, tư duy thiết kế dễ bị chi phối bởi cách hiện thực có sẵn thay vì nhu cầu thực sự.

d) Giữ phạm vi thay đổi tối thiểu, có kiểm soát
GREEN chỉ nên là bước viết đủ code để pass đúng test vừa fail — không hơn. Nếu không có bước RED rõ ràng trước đó, ranh giới "đủ để pass" trở nên mơ hồ, dễ dẫn đến việc viết thừa code chưa được kiểm chứng bằng test nào.

e) Refactor an toàn nhờ có "lưới bảo hiểm"
Bước REFACTOR chỉ có ý nghĩa và an toàn khi đã có GREEN — tức là đã có ít nhất một test pass làm cơ sở để phát hiện nếu refactor làm hỏng hành vi. Không thể refactor an toàn nếu chưa từng có trạng thái "code hoạt động đúng" được xác nhận.

Tóm lại thứ tự logic: RED chứng minh test có giá trị → GREEN chứng minh code đáp ứng đúng yêu cầu tối thiểu → REFACTOR cải thiện chất lượng mà không phá vỡ điều đã đạt được. Đảo ngược thứ tự này (ví dụ viết code trước) sẽ làm mất đi cơ chế tự kiểm chứng vốn là giá trị cốt lõi của TDD.


**Prompt:**
> Nếu so sánh với quy trình truyền thống (test after) là viết xong code mới test thì so với TDD khác gì và nó có lợi thế và điểm yếu gì của từng loại so với cái còn lại.

So sánh TDD (Test-First) và Test-After trong bối cảnh Ticket Manager CLI
1. Khác biệt cốt lõi về quy trình
	TDD (Test-First)	Test-After
Thứ tự	Test → Code → Refactor	Code → Test
Vai trò của test	Công cụ thiết kế (design driver)	Công cụ xác minh (verification) sau khi đã có giải pháp
Tư duy khi viết	"Tôi cần API/hành vi như thế nào để giải quyết việc này?"	"Tôi đã viết cái này, giờ kiểm tra xem nó có đúng không?"
Vòng phản hồi	Rất ngắn (từng hành vi nhỏ, vài phút)	Thường dài hơn (viết xong cả hàm/module mới test)

Ví dụ cụ thể với updateTicket: ở vòng lặp trước, test updateTicket không cho phép đổi id được viết trước khi biết code sẽ merge object thế nào — nhờ đó việc lọc field id được thiết kế ngay từ đầu. Nếu làm test-after, rất có thể bạn viết {...ticket, ...changes} trước, chạy thử thấy "hoạt động", rồi mới viết test — và nếu không tự nhớ ra edge case đó, test-after sẽ không bao giờ phát hiện ra lỗ hổng này, vì bạn chỉ có xu hướng viết test cho cái mình đã biết code làm được.

2. Ưu điểm của TDD

a) Đảm bảo mọi dòng code đều có lý do tồn tại (test-driven scope)
Vì code chỉ được viết để làm test pass, bạn khó viết code thừa, không dùng đến hoặc "phòng hờ" không cần thiết — như trong ví dụ loadAll(), ban đầu không xử lý file hỏng vì chưa có test yêu cầu, tránh over-engineering sớm.

b) Test suite có độ phủ cao một cách tự nhiên
Vì mỗi dòng code sinh ra từ một test, tỷ lệ code không được test gần như bằng 0. Với test-after, người viết dễ bỏ sót các nhánh nhỏ (ví dụ file JSON rỗng, id không tồn tại) vì tâm lý "code chạy được rồi, test qua loa cho có".

c) Test đóng vai trò tài liệu thiết kế API
Nhìn vào bộ test của ticketService.js, người đọc hiểu ngay createTicket cần tham số gì, trả về gì — vì test được viết ở góc nhìn người dùng hàm, trước khi bị "nhiễm" bởi chi tiết cài đặt bên trong.

d) Phản hồi nhanh, phát hiện lỗi thiết kế sớm
Nếu ở vòng lặp 2, việc inject storage làm tham số khiến code khó test → bạn phát hiện vấn đề thiết kế ngay lúc viết test, trước khi module phình to. Test-after chỉ phát hiện vấn đề này sau khi toàn bộ module đã được viết theo kiến trúc khó test — chi phí sửa lại lúc đó cao hơn nhiều.

e) An toàn khi refactor
Vì luôn có bộ test bao phủ từ đầu, việc tái cấu trúc (như tách idCounter ra khỏi biến toàn cục) được thực hiện với "lưới bảo hiểm" ngay lập tức.

3. Nhược điểm của TDD

a) Chi phí thời gian ban đầu cao hơn, đặc biệt với người mới
Việc phải nghĩ ra test trước khi biết chính xác code sẽ trông ra sao đòi hỏi một tư duy trừu tượng hóa nhất định. Với CLI đơn giản như Ticket Manager, nhiều lập trình viên mới sẽ cảm thấy chậm hơn đáng kể so với việc code thẳng.

b) Khó áp dụng khi chưa rõ thiết kế/API (exploratory coding)
Ở giai đoạn khám phá — ví dụ chưa chắc chắn cấu trúc file JSON nên là mảng [{...}] hay object {tickets: [...], meta: {...}} — viết test trước cho một thiết kế chưa ổn định dễ dẫn đến viết đi viết lại test liên tục. Nhiều đội chọn "spike" (viết code thử nghiệm nhanh, không test, rồi vứt đi) trước khi TDD hóa phiên bản thật.

c) Rủi ro test giả tạo, gắn chặt vào implementation
Nếu người viết TDD không cẩn thận (ví dụ mock quá nhiều, hoặc test cả các hàm private), test dễ trở nên gắn chặt với cách hiện thực (implementation detail) thay vì hành vi thực sự — khiến refactor sau này làm vỡ hàng loạt test dù hành vi bên ngoài không đổi. Đây là lỗi thường gặp, không phải lỗi bản chất của TDD, nhưng dễ xảy ra khi làm sai.

d) Không đảm bảo chất lượng nếu người viết test kém
TDD chỉ tốt bằng chất lượng tư duy của người viết test. Nếu chỉ viết test hời hợt để "cho có Red-Green", TDD không mang lại giá trị hơn test-after là bao.

4. Ưu điểm của Test-After

a) Linh hoạt hơn trong giai đoạn thiết kế chưa chắc chắn
Cho phép viết code thăm dò (prototype) nhanh để tìm ra giải pháp trước, tránh mất thời gian viết test cho một thiết kế sẽ bị vứt bỏ.

b) Tốc độ ban đầu nhanh hơn với logic đơn giản, quen thuộc
Với các hàm rất đơn giản (ví dụ một hàm format ngày tháng cho output CLI), việc code thẳng rồi test nhanh gọn hơn, không cần thiết phải qua chu trình Red-Green đầy đủ.

c) Dễ tiếp cận hơn cho người mới học kiểm thử
Không đòi hỏi tư duy "thiết kế từ test" — chỉ cần biết cách viết assertion cho code đã có, ngưỡng học tập thấp hơn.

5. Nhược điểm của Test-After

a) Thiên vị xác nhận (confirmation bias)
Người viết test sau khi đã có code thường vô thức viết test để khớp với những gì code đã làm, thay vì kiểm tra xem code có đúng với yêu cầu hay không. Ví dụ: nếu updateTicket lỡ cho phép đổi id, người viết test-after rất dễ không nghĩ đến việc test trường hợp đó, vì họ đang nhìn từ góc độ "code đã chạy đúng luồng chính".

b) Độ phủ test thường thấp hơn, đặc biệt ở edge case
Sau khi code đã chạy được, tâm lý phổ biến là viết test cho luồng chính (happy path) rồi dừng, bỏ qua các nhánh lỗi/biên như file JSON hỏng, id không tồn tại.

c) Thiết kế khó thay đổi khi phát hiện vấn đề qua test
Nếu đến lúc viết test mới phát hiện ticketService.js khó test vì phụ thuộc trực tiếp vào fs (không inject storage), việc sửa lại kiến trúc lúc này tốn kém hơn nhiều so với việc phát hiện ngay từ khi viết test đầu tiên (như TDD).

d) Dễ bị bỏ qua/trì hoãn dưới áp lực deadline
Vì test được xem là bước "phụ" sau khi tính năng đã "xong", trong thực tế test-after thường bị cắt giảm hoặc bỏ hẳn khi deadline gấp — trong khi TDD, test là một phần bắt buộc để có code, nên khó bị lược bỏ theo cách tương tự.

6. Tổng kết cho bối cảnh bài tập Ticket Manager CLI
Tiêu chí	TDD phù hợp hơn khi	Test-After phù hợp hơn khi
Logic nghiệp vụ rõ ràng (create, update ticket)	✅ Có	
Cần thiết kế interface storage sạch, dễ mock	✅ Có	
Thăm dò cấu trúc file JSON ban đầu, chưa chắc chắn		✅ Có
Hàm tiện ích đơn giản (format output ra console)		✅ Có
Cần đảm bảo edge case (file hỏng, id sai) không bị bỏ sót	✅ Có	

Trong thực tế, nhiều đội áp dụng kết hợp: dùng code thăm dò không test để chốt thiết kế ban đầu (đặc biệt với cấu trúc lưu JSON), sau đó quay lại dùng TDD nghiêm ngặt để hiện thực chính thức từng hành vi — tận dụng được sự linh hoạt của test-after ở giai đoạn khám phá và độ chặt chẽ của TDD ở giai đoạn hiện thực.

**Prompt:**
> TDD có được chia ra làm nhiều cấp độ hay không  và nếu có thì chỉ ra sự khác biệt của nó


A. Phân loại theo phạm vi: Double-Loop TDD
1. Acceptance TDD (ATDD) — vòng lặp ngoài
Test được viết ở mức hành vi người dùng cuối (ví dụ: "Khi tôi chạy ticket create --title 'Fix bug', hệ thống phải in ra ticket vừa tạo với id"), thường dùng công cụ như Cucumber, hoặc đơn giản là test end-to-end gọi CLI thật.
Mục đích: xác nhận tính năng đáp ứng đúng yêu cầu nghiệp vụ, không quan tâm chi tiết bên trong.
Vòng lặp Red-Green ở đây rất chậm — có thể mất nhiều commit/nhiều unit test con mới làm test này pass.

Ví dụ với Ticket Manager:

js
test('[Acceptance] CLI create ticket và lưu vào file JSON thực tế', () => {
  execSync(`node cli.js create --title "Fix bug"`);
  const data = JSON.parse(fs.readFileSync('tickets.json'));
  expect(data[0].title).toBe('Fix bug');
});
2. Unit TDD (Developer TDD) — vòng lặp trong
Đây chính là các ví dụ Red-Green-Refactor chi tiết ở các câu trả lời trước: test cho từng hàm nhỏ như createTicket, loadAll, updateTicket.
Mục đích: đảm bảo từng đơn vị logic nhỏ hoạt động đúng, phản hồi cực nhanh.
Cách hai cấp độ này phối hợp (Double-Loop)

Quy trình thực tế: viết một test Acceptance (RED, thất bại vì tính năng chưa tồn tại) → bên trong đó, chạy nhiều vòng lặp Unit TDD nhỏ (Red-Green-Refactor cho createTicket, save, parseArgs...) → khi tất cả unit ghép lại đủ, test Acceptance ở vòng ngoài mới chuyển GREEN. Đây là lý do gọi là "double-loop" — một vòng lặp lớn bao ngoài nhiều vòng lặp nhỏ.

	Acceptance TDD	Unit TDD
Tốc độ phản hồi	Chậm (giây đến phút)	Rất nhanh (mili-giây)
Phạm vi	Toàn bộ tính năng/luồng	Một hàm/class
Vai trò	Xác nhận "đúng yêu cầu" (đúng cái cần làm)	Xác nhận "đúng cách làm" (code chạy đúng)
Ví dụ trong dự án	Test chạy CLI thật, kiểm tra file JSON	Test createTicket(), loadAll()
B. Phân loại theo triết lý: Classicist (Detroit) vs Mockist (London)

Đây là sự khác biệt quan trọng hơn về mặt tư duy thiết kế, xuất phát từ hai trường phái lịch sử của TDD.

1. Classicist / Detroit School (Kent Beck)
Triết lý: test hành vi (state-based testing) — kiểm tra kết quả đầu ra/trạng thái cuối cùng của đối tượng, hạn chế tối đa việc dùng mock.
Chỉ mock các phụ thuộc thực sự "khó kiểm soát" (I/O, network, thời gian hệ thống) — còn lại dùng object thật.
Cách tiếp cận: Inside-Out — xây từ các thành phần nhỏ nhất (domain logic) trước, rồi lắp ráp dần lên các lớp cao hơn.

Ví dụ với Ticket Manager theo phong cách Classicist — test updateTicket bằng cách kiểm tra state thật trả về, không mock gì thêm:

js
test('[Classicist] updateTicket trả về ticket đã cập nhật status', () => {
  const storage = createInMemoryStorage(); // fake thật, không phải mock có assertion
  const t = createTicket({ title: 'A' }, storage);
  const updated = updateTicket(t.id, { status: 'done' }, storage);
  expect(updated.status).toBe('done'); // kiểm tra STATE, không kiểm tra cách gọi hàm
});
2. Mockist / London School (Steve Freeman, Nat Pryce)
Triết lý: test tương tác (interaction-based testing) — kiểm tra xem một object có gọi đúng phương thức, với đúng tham số đến các collaborator (dependency) của nó hay không, dùng mock có xác nhận lời gọi (expect(mock).toHaveBeenCalledWith(...)).
Mock mọi dependency của đơn vị đang test, kể cả các object domain khác — không chỉ I/O.
Cách tiếp cận: Outside-In — bắt đầu từ lớp giao diện ngoài cùng (ví dụ CLI command), giả lập (mock) toàn bộ các lớp bên dưới chưa tồn tại, rồi "đào sâu" dần xuống, viết TDD cho từng lớp mock đó theo thứ tự.

Ví dụ theo phong cách Mockist — test rằng createTicket gọi đúng storage.save với đúng tham số:

js
test('[Mockist] createTicket gọi storage.save với đúng ticket vừa tạo', () => {
  const mockStorage = { save: jest.fn() };
  const ticket = createTicket({ title: 'A' }, mockStorage);
  expect(mockStorage.save).toHaveBeenCalledWith(
    expect.objectContaining({ title: 'A', status: 'open' })
  );
});

Đây chính là ví dụ ở vòng lặp 2 trong câu trả lời trước — thực chất đã mang phong cách Mockist (test cách createTicket tương tác với storage, chứ không test state cuối cùng của file).

So sánh hai trường phái
	Classicist (Detroit)	Mockist (London)
Kiểm tra gì	Trạng thái/kết quả cuối (state)	Cách các object tương tác với nhau (interaction)
Dùng mock khi nào	Chỉ cho I/O, thời gian, network...	Cho hầu hết mọi collaborator
Hướng phát triển	Inside-Out (từ lõi ra ngoài)	Outside-In (từ ngoài vào lõi)
Ưu điểm	Test ít gắn với chi tiết cài đặt → refactor thoải mái hơn	Thiết kế rõ ràng ngay từ đầu về cách các class giao tiếp; hỗ trợ tốt cho thiết kế hướng đối tượng phức tạp
Nhược điểm	Có thể chậm phát hiện vấn đề thiết kế về sự phối hợp giữa các object	Test dễ gắn chặt vào implementation detail (mock quá nhiều) → refactor dễ làm vỡ hàng loạt test dù hành vi không đổi
Phù hợp với dự án	Logic nghiệp vụ tương đối đơn giản, ít lớp trừu tượng — như ticketService.js trong ví dụ	Hệ thống nhiều lớp, nhiều object cộng tác phức tạp (thường gặp ở kiến trúc lớn, nhiều service)
Áp dụng vào Ticket Manager CLI

Với quy mô dự án này (CLI đơn giản, ít lớp trừu tượng), phong cách phù hợp hơn thường là Classicist pha trộn có kiểm soát: dùng fake/in-memory storage thật (không phải mock có assertion) cho phần lớn test service, chỉ dùng mock/spy (kiểu Mockist) ở những chỗ thực sự cần xác nhận hành vi gọi — ví dụ đảm bảo save() được gọi đúng một lần khi update, tránh ghi file thừa. Đây cũng là thực tế phổ biến: rất ít đội theo tuyệt đối một trường phái, mà chọn linh hoạt theo từng ngữ cảnh test cụ thể.


**Prompt:**
> TDD có được chia ra làm nhiều cấp độ hay không  và nếu có thì chỉ ra sự khác biệt của nó


Các cấp độ / trường phái của TDD

Có, TDD thường được phân loại theo hai trục khác nhau: (A) theo phạm vi/độ chi tiết của kiểm thử (Acceptance vs Unit level — thường gọi là "Double-Loop TDD"), và (B) theo triết lý thiết kế/cách dùng test double (Classicist/Detroit School vs Mockist/London School). Đây là hai cách phân loại độc lập, không loại trừ nhau.

A. Phân loại theo phạm vi: Double-Loop TDD
1. Acceptance TDD (ATDD) — vòng lặp ngoài
Test được viết ở mức hành vi người dùng cuối (ví dụ: "Khi tôi chạy ticket create --title 'Fix bug', hệ thống phải in ra ticket vừa tạo với id"), thường dùng công cụ như Cucumber, hoặc đơn giản là test end-to-end gọi CLI thật.
Mục đích: xác nhận tính năng đáp ứng đúng yêu cầu nghiệp vụ, không quan tâm chi tiết bên trong.
Vòng lặp Red-Green ở đây rất chậm — có thể mất nhiều commit/nhiều unit test con mới làm test này pass.

Ví dụ với Ticket Manager:

js
test('[Acceptance] CLI create ticket và lưu vào file JSON thực tế', () => {
  execSync(`node cli.js create --title "Fix bug"`);
  const data = JSON.parse(fs.readFileSync('tickets.json'));
  expect(data[0].title).toBe('Fix bug');
});
2. Unit TDD (Developer TDD) — vòng lặp trong
Đây chính là các ví dụ Red-Green-Refactor chi tiết ở các câu trả lời trước: test cho từng hàm nhỏ như createTicket, loadAll, updateTicket.
Mục đích: đảm bảo từng đơn vị logic nhỏ hoạt động đúng, phản hồi cực nhanh.
Cách hai cấp độ này phối hợp (Double-Loop)

Quy trình thực tế: viết một test Acceptance (RED, thất bại vì tính năng chưa tồn tại) → bên trong đó, chạy nhiều vòng lặp Unit TDD nhỏ (Red-Green-Refactor cho createTicket, save, parseArgs...) → khi tất cả unit ghép lại đủ, test Acceptance ở vòng ngoài mới chuyển GREEN. Đây là lý do gọi là "double-loop" — một vòng lặp lớn bao ngoài nhiều vòng lặp nhỏ.

	Acceptance TDD	Unit TDD
Tốc độ phản hồi	Chậm (giây đến phút)	Rất nhanh (mili-giây)
Phạm vi	Toàn bộ tính năng/luồng	Một hàm/class
Vai trò	Xác nhận "đúng yêu cầu" (đúng cái cần làm)	Xác nhận "đúng cách làm" (code chạy đúng)
Ví dụ trong dự án	Test chạy CLI thật, kiểm tra file JSON	Test createTicket(), loadAll()
B. Phân loại theo triết lý: Classicist (Detroit) vs Mockist (London)

Đây là sự khác biệt quan trọng hơn về mặt tư duy thiết kế, xuất phát từ hai trường phái lịch sử của TDD.

1. Classicist / Detroit School (Kent Beck)
Triết lý: test hành vi (state-based testing) — kiểm tra kết quả đầu ra/trạng thái cuối cùng của đối tượng, hạn chế tối đa việc dùng mock.
Chỉ mock các phụ thuộc thực sự "khó kiểm soát" (I/O, network, thời gian hệ thống) — còn lại dùng object thật.
Cách tiếp cận: Inside-Out — xây từ các thành phần nhỏ nhất (domain logic) trước, rồi lắp ráp dần lên các lớp cao hơn.

Ví dụ với Ticket Manager theo phong cách Classicist — test updateTicket bằng cách kiểm tra state thật trả về, không mock gì thêm:

js
test('[Classicist] updateTicket trả về ticket đã cập nhật status', () => {
  const storage = createInMemoryStorage(); // fake thật, không phải mock có assertion
  const t = createTicket({ title: 'A' }, storage);
  const updated = updateTicket(t.id, { status: 'done' }, storage);
  expect(updated.status).toBe('done'); // kiểm tra STATE, không kiểm tra cách gọi hàm
});
2. Mockist / London School (Steve Freeman, Nat Pryce)
Triết lý: test tương tác (interaction-based testing) — kiểm tra xem một object có gọi đúng phương thức, với đúng tham số đến các collaborator (dependency) của nó hay không, dùng mock có xác nhận lời gọi (expect(mock).toHaveBeenCalledWith(...)).
Mock mọi dependency của đơn vị đang test, kể cả các object domain khác — không chỉ I/O.
Cách tiếp cận: Outside-In — bắt đầu từ lớp giao diện ngoài cùng (ví dụ CLI command), giả lập (mock) toàn bộ các lớp bên dưới chưa tồn tại, rồi "đào sâu" dần xuống, viết TDD cho từng lớp mock đó theo thứ tự.

Ví dụ theo phong cách Mockist — test rằng createTicket gọi đúng storage.save với đúng tham số:

js
test('[Mockist] createTicket gọi storage.save với đúng ticket vừa tạo', () => {
  const mockStorage = { save: jest.fn() };
  const ticket = createTicket({ title: 'A' }, mockStorage);
  expect(mockStorage.save).toHaveBeenCalledWith(
    expect.objectContaining({ title: 'A', status: 'open' })
  );
});

Đây chính là ví dụ ở vòng lặp 2 trong câu trả lời trước — thực chất đã mang phong cách Mockist (test cách createTicket tương tác với storage, chứ không test state cuối cùng của file).

So sánh hai trường phái
	Classicist (Detroit)	Mockist (London)
Kiểm tra gì	Trạng thái/kết quả cuối (state)	Cách các object tương tác với nhau (interaction)
Dùng mock khi nào	Chỉ cho I/O, thời gian, network...	Cho hầu hết mọi collaborator
Hướng phát triển	Inside-Out (từ lõi ra ngoài)	Outside-In (từ ngoài vào lõi)
Ưu điểm	Test ít gắn với chi tiết cài đặt → refactor thoải mái hơn	Thiết kế rõ ràng ngay từ đầu về cách các class giao tiếp; hỗ trợ tốt cho thiết kế hướng đối tượng phức tạp
Nhược điểm	Có thể chậm phát hiện vấn đề thiết kế về sự phối hợp giữa các object	Test dễ gắn chặt vào implementation detail (mock quá nhiều) → refactor dễ làm vỡ hàng loạt test dù hành vi không đổi
Phù hợp với dự án	Logic nghiệp vụ tương đối đơn giản, ít lớp trừu tượng — như ticketService.js trong ví dụ	Hệ thống nhiều lớp, nhiều object cộng tác phức tạp (thường gặp ở kiến trúc lớn, nhiều service)
Áp dụng vào Ticket Manager CLI

Với quy mô dự án này (CLI đơn giản, ít lớp trừu tượng), phong cách phù hợp hơn thường là Classicist pha trộn có kiểm soát: dùng fake/in-memory storage thật (không phải mock có assertion) cho phần lớn test service, chỉ dùng mock/spy (kiểu Mockist) ở những chỗ thực sự cần xác nhận hành vi gọi — ví dụ đảm bảo save() được gọi đúng một lần khi update, tránh ghi file thừa. Đây cũng là thực tế phổ biến: rất ít đội theo tuyệt đối một trường phái, mà chọn linh hoạt theo từng ngữ cảnh test cụ thể.

trên mạng tôi đọc được chỉ có thông tin về mục A Double-Loop TDD chưa thấy mục B bạn có thể đưa tôi link trang web minh chứng được không

Có, đây là chủ đề đã được viết khá nhiều — mục B (London/Mockist vs Detroit-Chicago/Classicist) là một cách phân loại có thật, phổ biến trong cộng đồng, chỉ là ít phổ biến hơn Double-Loop TDD ở các nguồn nhập môn tiếng Việt/beginner. Dưới đây là các nguồn gốc và bài viết uy tín để bạn trích dẫn cho bài học thuật:

Nguồn gốc chính thống (sách gốc):

Freeman, Steve & Pryce, Nat — "Growing Object-Oriented Software, Guided by Tests" (2009). Đây là cuốn sách được xem là văn bản gốc định hình trường phái London/Mockist. Cách tiếp cận Mockist bản chất xoay quanh các ý tưởng trình bày trong cuốn sách này, chuyển từ kiểm chứng trạng thái sang kiểm chứng tương tác thông qua mock. 
control-plane

Bài viết/thảo luận có thể trích dẫn:

London school TDD — testdouble/contributing-tests wiki
Giải thích nguồn gốc trường phái London, xuất phát từ cộng đồng Extreme Programming ở London, đối lập với trường phái Detroit phổ biến hơn trước đó.
GitHub
Detroit and London Schools of Test-Driven Development
Bài phân tích chi tiết lịch sử và khác biệt kỹ thuật giữa hai trường phái, có đề cập Kent Beck (Detroit) và Robert C. Martin.
DevGenius (Medium)
"Classic" versus "Mockist" TDD, Distinction Real?
Bài viết năm 2009 tường thuật cuộc thảo luận gốc giữa Steve Freeman, Nat Pryce, Michael Feathers về ranh giới giữa hai trường phái.
InfoQ
Making TDD Work for You, Part 2: crossing TDD's tribal lines
Giải thích rõ vì sao trường phái London ra đời (tránh phá vỡ tính đóng gói/encapsulation khi phải thêm getter để test).
control-plane.io
London vs Chicago: The two ways of TDD
So sánh trực quan ưu nhược điểm hai trường phái, kèm bảng tổng hợp Inside-Out vs Outside-In.
Substack

Vài lưu ý học thuật quan trọng khi trích dẫn:

Đây không phải là một "cấp độ" chính thức trong lý thuyết TDD gốc của Kent Beck (cuốn Test-Driven Development: By Example, 2003) — Beck không dùng thuật ngữ "Detroit school". Tên gọi "Detroit" và "London" là cách người ta hồi cố đặt tên để phân biệt hai cách thực hành khác nhau đã hình thành theo thời gian, không phải hai "level" được định nghĩa từ đầu như Double-Loop TDD.
Tên gọi khác cũng hay gặp: "Chicago school" thường được dùng thay cho "Detroit school" trong nhiều bài viết gần đây (cùng một khái niệm, khác tên do nguồn gốc lịch sử không hoàn toàn thống nhất) — bạn sẽ thấy điều này khi đọc các nguồn trên, nên nếu trích dẫn trong bài học thuật, nên ghi chú cả hai tên để tránh gây nhầm lẫn cho người chấm.
Nếu cần nguồn học thuật/sách chính thống nhất để trích dẫn: ưu tiên trích Freeman & Pryce, "Growing Object-Oriented Software, Guided by Tests" (2009) — đây là tài liệu gốc, có ISBN, phù hợp hơn để đưa vào danh mục tài liệu tham khảo (references) so với các bài blog, vốn phù hợp hơn cho phần đọc thêm/thảo luận.


---

## Lượt 2 — Brief Feature → Code Example → Validation (gộp trong 1 lượt hỏi-đáp)

**Prompt:**
> Tôi vừa đọc lại code example của bạn thì tôi cần bạn bổ sung về thực thể Ticket bao gồm những attribute nào và nó sẽ có những edge case kiểm tra validation (field rỗng, ticket trùng ID, thiếu field bắt buộc phải điền,... ) chỉ cần gen example cho create ticket


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

**Prompt:**
> Vậy thì trong khi áp dụng TDD và thực tế thì thường các developer sẽ gặp những lỗi thông thường gì và giải pháp để giải quyết vấn đề đó ra sao

Các lỗi thường gặp khi áp dụng TDD trong thực tế và giải pháp
1. Viết test quá lớn, bao quát nhiều hành vi cùng lúc

Biểu hiện: Thay vì test từng hành vi nhỏ, developer viết một test kiểm tra luôn cả "tạo ticket, lưu file, đọc lại, update status" trong một test() block.

js
// SAI - test quá to
test('toàn bộ luồng ticket', () => {
  const t = createTicket({title: 'A'}, storage);
  const updated = updateTicket(t.id, {status: 'done'}, storage);
  const all = listTickets(storage);
  expect(all).toHaveLength(1);
  expect(updated.status).toBe('done');
  // ...còn nhiều assertion khác
});

Hậu quả: Khi test fail, khó biết chính xác dòng nào gây lỗi. Vi phạm nguyên tắc RED phải rõ ràng — một RED nên chỉ vì một lý do duy nhất.

Giải pháp: Áp dụng nguyên tắc mỗi test chỉ assert một hành vi/khái niệm logic (không nhất thiết là một expect(), nhưng phải cùng một ý nghĩa). Tách thành các test độc lập: test('createTicket trả về status mặc định open'), test('updateTicket đổi status thành done'), v.v. Nếu thấy tên test có chữ "và" (ví dụ "tạo và lưu và đọc"), đó là dấu hiệu cần tách.

2. Bỏ qua bước RED — không xác nhận test thực sự fail trước

Biểu hiện: Viết test và code gần như cùng lúc, không chạy test riêng ở trạng thái chưa có code để xem nó fail.

Hậu quả: Không phát hiện được test "giả" — ví dụ quên await, quên gọi đúng hàm export, hoặc assertion sai logic (như expect(true).toBe(true)) khiến test luôn pass dù code có đúng hay không.

Giải pháp: Luôn chạy test ngay sau khi viết, trước khi viết bất kỳ dòng code chức năng nào, và đọc kỹ thông báo lỗi — đảm bảo nó fail vì lý do đúng (thiếu implementation), không phải vì lỗi cú pháp/import sai. Với Ticket Manager, một mẹo kiểm tra: thử tạm sửa assertion thành sai (ví dụ status).toBe('closed') thay vì 'open') — nếu test vẫn pass, chứng tỏ assertion có vấn đề.

3. Viết thừa code ở bước GREEN (over-engineering sớm)

Biểu hiện: Test chỉ yêu cầu createTicket trả về object có id, title, status, nhưng developer nhân tiện viết luôn validate title rỗng, giới hạn độ dài, thêm createdAt, priority... dù chưa có test nào yêu cầu.

Hậu quả: Code không được test bảo vệ (vì viết trước khi có test tương ứng), dễ có lỗi tiềm ẩn, và làm chậm tiến độ vì giải quyết vấn đề chưa cần thiết.

Giải pháp: Kỷ luật "chỉ viết đủ để pass test hiện tại" (nguyên tắc "Fake it till you make it" của Kent Beck ở giai đoạn đầu). Nếu nghĩ ra thêm yêu cầu trong lúc code, ghi lại thành TODO/backlog, rồi quay lại viết test riêng cho nó ở vòng lặp sau — không chèn ngang vào GREEN hiện tại.

4. Bỏ qua hoặc lơ là bước REFACTOR

Biểu hiện: Chu trình trở thành "Red-Green-Red-Green..." liên tục, không bao giờ dừng lại dọn code, vì tâm lý "test đang pass rồi, để sau".

Hậu quả: Code tích lũy trùng lặp, đặt tên xấu, cấu trúc rối — ví dụ trong Ticket Manager, hàm updateTicket dần dần chứa logic merge phức tạp copy-paste từ createTicket mà không được tách hàm chung.

Giải pháp: Coi REFACTOR là bước bắt buộc, không phải tùy chọn — đặt quy tắc cá nhân: sau mỗi 2-3 vòng Green, dừng lại rà soát code trùng lặp. Dùng test suite hiện có làm lưới an toàn để refactor tự tin, chạy lại toàn bộ test sau mỗi thay đổi nhỏ.

5. Test gắn chặt vào chi tiết cài đặt (Mock quá mức / Over-mocking)

Biểu hiện: Test kiểm tra cách hàm gọi nội bộ ra sao (ví dụ assert rằng save() được gọi đúng 1 lần, với đúng thứ tự tham số bên trong) thay vì kiểm tra kết quả/hành vi bên ngoài.

js
// Dễ vỡ khi refactor dù hành vi thực tế không đổi
expect(mockStorage.save).toHaveBeenCalledTimes(1);
expect(mockStorage.loadAll).toHaveBeenCalledBefore(mockStorage.save);

Hậu quả: Đây là cạm bẫy phổ biến của phong cách Mockist khi lạm dụng — chỉ cần đổi thứ tự gọi hàm nội bộ (mà kết quả cuối vẫn đúng), hàng loạt test vỡ dù chương trình chạy đúng. Điều này làm nản lòng việc refactor — đi ngược lại chính mục đích của TDD.

Giải pháp: Ưu tiên test trạng thái/kết quả cuối cùng (state-based, phong cách Classicist) cho phần lớn logic nghiệp vụ; chỉ dùng mock có assertion về lời gọi khi thực sự cần xác nhận side-effect quan trọng (ví dụ: đảm bảo file chỉ được ghi đúng 1 lần để tránh mất dữ liệu, không phải để kiểm tra thứ tự gọi hàm tùy tiện).

6. Test phụ thuộc lẫn nhau / không cô lập (Test Pollution)

Biểu hiện: Trong Ticket Manager, biến idCounter toàn cục hoặc file JSON dùng chung giữa các test khiến test A chạy trước làm ảnh hưởng kết quả test B.

js
// Lỗi: test này pass hay fail phụ thuộc thứ tự chạy
test('ticket đầu tiên có id = 1', () => {
  const t = createTicket({title: 'A'});
  expect(t.id).toBe(1); // fail nếu có test khác chạy trước làm tăng idCounter
});

Hậu quả: Test flaky (lúc pass lúc fail tùy thứ tự chạy), rất khó debug vì lỗi không nhất quán.

Giải pháp: Đảm bảo mỗi test có state riêng biệt — dùng beforeEach/afterEach để reset state (in-memory storage mới, file tạm mới mỗi test, generate id bằng UUID thay vì counter toàn cục). Nguyên tắc: mỗi test phải chạy độc lập theo bất kỳ thứ tự nào và cho kết quả giống nhau.

7. Viết test sau khi thấy code "chắc chắn đúng" (Skip TDD khi tự tin)

Biểu hiện: Với các hàm tưởng như đơn giản (ví dụ parseArgs), developer code thẳng trước "vì dễ mà", rồi viết test sau — âm thầm quay về test-after mà không nhận ra.

Hậu quả: Đây thường là nơi bug ẩn náu nhiều nhất, vì "tưởng đơn giản" — ví dụ quên xử lý --title có khoảng trắng, hoặc thiếu --title hoàn toàn.

Giải pháp: Áp dụng kỷ luật đều đặn bất kể độ khó cảm nhận — chính những hàm "tưởng đơn giản" lại hưởng lợi nhiều từ việc liệt kê edge case ra dưới dạng test trước khi code (thiếu tham số, tham số sai định dạng, tham số dư thừa).

8. Coverage cao nhưng chất lượng test thấp (Test theo kiểu "cho có")

Biểu hiện: Viết test chỉ để đạt % coverage, không thực sự assert điều gì có ý nghĩa.

js
// Coverage tăng nhưng vô nghĩa
test('createTicket chạy không lỗi', () => {
  createTicket({title: 'A'}, storage);
  expect(true).toBe(true);
});

Hậu quả: Con số coverage đẹp nhưng không phản ánh chất lượng thực — đây là hiện tượng "Goodhart's Law" áp dụng vào testing: khi coverage trở thành mục tiêu, nó ngừng là thước đo tốt.

Giải pháp: Đánh giá test bằng câu hỏi "nếu tôi cố tình làm hỏng code (mutate một điều kiện, đổi một giá trị), test này có bắt được không?" — kỹ thuật này gọi là mutation testing, có thể áp dụng thủ công hoặc bằng công cụ (Stryker cho JS) để kiểm tra độ "sắc" thực sự của test suite, chứ không chỉ dừng ở coverage phần trăm.

Tổng kết: nguyên nhân gốc rễ chung

Phần lớn các lỗi trên xuất phát từ một điểm chung: đối xử với TDD như một thủ tục hình thức (chạy đúng 3 bước cho có) thay vì một kỷ luật tư duy thiết kế. Giải pháp bền vững nhất không phải là ghi nhớ danh sách lỗi này, mà là liên tục tự hỏi ở mỗi bước: "Test này có đang thực sự ép tôi thiết kế đúng và phát hiện đúng lỗi hay không?" — nếu câu trả lời mơ hồ, đó là dấu hiệu cần dừng lại xem xét lại cách viết test, trước khi tiếp tục vòng lặp tiếp theo.

thông qua những lý thuyết trên thì tôi cần lưu ý những key word nào để nắm rõ được về TDD

Bảng từ khóa (Keywords) cốt lõi về TDD
Nhóm 1: Khái niệm nền tảng
Từ khóa	Ý nghĩa cốt lõi
TDD (Test-Driven Development)	Phương pháp viết test trước code chức năng
Test-First	Cách gọi khác nhấn mạnh thứ tự: test → code
Test-After	Đối lập với TDD: code → test
Kent Beck	Người hệ thống hóa và phổ biến TDD, tác giả Test-Driven Development: By Example
Extreme Programming (XP)	Phong trào Agile mà TDD là một thực hành cốt lõi
YAGNI (You Aren't Gonna Need It)	Nguyên tắc không viết code/tính năng chưa có test/yêu cầu đòi hỏi
Nhóm 2: Chu trình Red-Green-Refactor
Từ khóa	Ý nghĩa cốt lõi
Red	Viết test cho hành vi chưa có, xác nhận nó fail
Green	Viết code tối thiểu để test pass
Refactor	Cải thiện code mà không đổi hành vi, có test bảo vệ
Fake it till you make it	Kỹ thuật ở bước Green: hard-code tạm để pass nhanh, rồi tổng quát hóa sau
Regression	Lỗi cũ tái xuất hiện do thay đổi mới — bộ test giúp phát hiện sớm
False positive (test)	Test pass dù code sai — hậu quả của việc bỏ qua bước Red
Nhóm 3: Phạm vi/cấp độ kiểm thử
Từ khóa	Ý nghĩa cốt lõi
Double-Loop TDD	Mô hình 2 vòng lặp: Acceptance (ngoài) bao Unit (trong)
Acceptance TDD (ATDD)	Test ở mức hành vi người dùng cuối, chạy chậm hơn
Unit TDD	Test từng hàm/class nhỏ, chạy rất nhanh
End-to-end test	Test toàn bộ luồng thật (ví dụ chạy CLI thật, đọc file JSON thật)
Nhóm 4: Trường phái thiết kế (rất quan trọng, hay bị bỏ sót)
Từ khóa	Ý nghĩa cốt lõi
Classicist / Detroit School / Chicago School	Test trạng thái (state-based), hạn chế mock, đi từ trong ra ngoài (Inside-Out)
Mockist / London School	Test tương tác (interaction-based), mock nhiều collaborator, đi từ ngoài vào trong (Outside-In)
State-based testing	Kiểm tra kết quả/trạng thái cuối cùng của object
Interaction-based testing	Kiểm tra object có gọi đúng phương thức/tham số đến collaborator không
Inside-Out	Xây từ domain logic lõi trước, ghép dần ra ngoài
Outside-In	Bắt đầu từ giao diện ngoài (API/CLI), mock các lớp bên dưới, đào dần vào trong
Freeman & Pryce	Tác giả Growing Object-Oriented Software, Guided by Tests — văn bản gốc của London School
Nhóm 5: Test double (kỹ thuật cô lập phụ thuộc)
Từ khóa	Ý nghĩa cốt lõi
Test double	Thuật ngữ chung cho mọi loại đối tượng giả dùng trong test
Mock	Test double có xác nhận lời gọi (assert đã được gọi thế nào)
Stub	Test double trả về giá trị giả định sẵn, không assert lời gọi
Fake	Cài đặt giả nhưng hoạt động thật (ví dụ in-memory storage thay file thật)
Spy	Ghi lại thông tin lời gọi để kiểm tra sau, thường kết hợp object thật
Nhóm 6: Lỗi thường gặp / anti-pattern
Từ khóa	Ý nghĩa cốt lõi
Over-mocking	Lạm dụng mock khiến test gắn chặt vào chi tiết cài đặt
Test pollution / Flaky test	Test phụ thuộc lẫn nhau hoặc cho kết quả không ổn định
Over-engineering (ở Green)	Viết thừa code vượt quá yêu cầu của test hiện tại
Goodhart's Law (áp dụng vào coverage)	Khi coverage % trở thành mục tiêu, nó ngừng phản ánh chất lượng thực
Mutation testing	Kỹ thuật cố tình làm hỏng code để kiểm tra test có phát hiện được không
Nhóm 7: Nguyên tắc thiết kế liên quan
Từ khóa	Ý nghĩa cốt lõi
Design pressure	Áp lực từ việc viết test trước, buộc code phải dễ test → dễ tách rời (decoupled)
Dependency Injection	Truyền phụ thuộc (như storage) qua tham số thay vì hard-code, giúp dễ test
Encapsulation	Lý do lịch sử ra đời London School — tránh phải thêm getter phá vỡ đóng gói khi test Classicist

Gợi ý cách ôn tập: Với 7 nhóm trên, nhóm 2 (Red-Green-Refactor) và nhóm 4 (Classicist vs Mockist) là hai phần dễ bị hỏi sâu nhất trong bài học thuật, vì chúng thể hiện được cả hiểu biết về quy trình lẫn tư duy thiết kế — hai khía cạnh giám khảo thường muốn phân biệt ở người thực sự hiểu TDD so với người chỉ thuộc "3 bước".