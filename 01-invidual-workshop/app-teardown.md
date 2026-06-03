# Workshop — Mổ App AI Thật

**Thời gian:** 35-45 phút  
**Hình thức:** cá nhân trước, chia sẻ theo nhóm sau  
**Output:** finding note + sketch `as-is / to-be`

Mục tiêu không phải chấm "UI đẹp hay xấu". Mục tiêu là dùng sản phẩm thật như một bài needfinding: tìm chỗ product gãy trong workflow thật, rồi viết finding đó thành quyết định product.

## 1. Chọn một sản phẩm để dùng thử

| Sản phẩm | AI feature | Cách truy cập | Chọn |
|---|---|---|---|
| MoMo — Moni | Trợ thủ tài chính, phân tích chi tiêu, chatbot | App MoMo | |
| **Vietnam Airlines — NEO** | **Chatbot hỗ trợ vé, hành lý, khiếu nại** | **Website/Zalo VNA** | **[x] Chọn** |
| V-App — V-AI | Trợ lý voice/text, gợi ý theo ngữ cảnh | App V-App | |

---

## 2. Dùng thử: promise vs reality

Ghi nhanh:

- **Product hứa gì?**  
  Vietnam Airlines quảng bá trợ lý ảo NEO là một giải pháp thông minh tự động 24/7 giúp khách hàng tra cứu thông tin chuyến bay nhanh chóng, đặt vé máy bay, kiểm tra điều kiện hoàn/hủy vé, chính sách hành lý và các tiện ích gia tăng mà không cần phải chờ đợi điện thoại viên.
- **User nào được hứa sẽ được giúp?**  
  Hành khách của Vietnam Airlines hoặc khách hàng tiềm năng muốn tìm kiếm thông tin hành trình, mua vé, hoặc cần hỗ trợ dịch vụ sau bán hàng (hoàn vé, mua thêm hành lý, hỏi quy định an toàn).
- **Bạn kỳ vọng AI làm được task nào?**  
  1. Hiểu ngữ cảnh hội thoại liên tục (nhớ điểm đi/đến, ngày bay khi khách thay đổi ý định).  
  2. Xử lý được ngôn ngữ tự nhiên thực tế của người Việt (teencode, slang như "sì gòn", "ngày mốt", "tml/tomorrow", viết không dấu).  
  3. Phản hồi thông minh trước các yêu cầu phi lý hoặc mâu thuẫn (nhận diện tính bất khả thi và cảnh báo thay vì cố tìm kiếm).  
  4. Tính toán và tư vấn mức phí cụ thể cho các tổ hợp hành lý đặc biệt thay vì chỉ trích xuất tài liệu chung chung.  
  5. Quản lý trạng thái phiên hội thoại (session state) tốt, không bị kẹt luồng hoặc rò rỉ ý định cũ sang các câu hỏi mới.
- **Khi dùng thật, điểm gãy xuất hiện ở đâu?**  
  1. **Lệch múi giờ & Nhận diện thời gian tương đối sai:** Khi hỏi chuyến bay ngày mai ("tomorrow / tml") lúc nửa đêm (00:08 ngày 04/06/2026), bot lại nhận dạng ngày khởi hành là hôm nay (04/06/2026).
  2. **Kẹt luồng hoàn vé (Sticky Intent / Context Leakage):** Sau khi hỏi về hoàn vé ở lượt trước, bot bị kẹt cứng trong luồng này. Ở các câu hỏi sau về tình trạng chuyến bay hay thời tiết Tokyo, bot trả lời xong là lập tức nài nỉ hỏi mã đặt chỗ để hoàn vé tiếp.
  3. **Thiếu Validation ngày đi:** Chấp nhận ngày không tồn tại "31/04/2026" ở bước nhận diện slot, chỉ báo lỗi hệ thống chung chung sau khi khách đã nhập xong số hành khách.
  4. **Phớt lờ ràng buộc phi lý (No Constraint Checking):** Khi khách đòi bay thẳng từ Hà Nội đi New York dưới 5 tiếng, giá dưới 1 triệu, bot bỏ qua hoàn toàn và đề xuất chuyến nối chuyến Frankfurt dài hơn 20 tiếng, giá 126-148 triệu VND.
  5. **Khả năng tổng hợp kém (Low Synthesis):** Gặp câu hỏi phức tạp (em bé 2 tuổi + thú cưng 5kg + cello 15kg), bot không tính được phí mà ném ra hàng loạt link điều khoản dài dòng bắt khách tự đọc.

### Evidence cần có:

#### 1. Evidence 1: Khả năng giữ ngữ cảnh (Context Retention)
* **Screenshot:**
  ![Evidence 1: Context Retention](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/screenshot_1.png)
* **Prompt/Input đã thử:**
  * Lượt 1: *"Tôi muốn tra chuyến bay đi Tokyo vào ngày 15/10"*
  * Lượt 2: *"À đổi lại sang ngày 16 đi"*
  * Lượt 3: *"15/10/2025"*
  * Lượt 4: *"Hà Nội"*
* **Hành vi quan sát được:** Bot bị mất ngữ cảnh điểm đến "Tokyo" và ngày "15/10" ngay tại lượt 2 khi khách chỉ thay đổi ngày đi. Nó quay lại form nhập thô ban đầu. Đến khi nhận đủ thông tin ở lượt 4, bot lại tự ghép cả ngày 15/10/2025 và 16/10/2025 vào cùng một câu trả lời cực kỳ mâu thuẫn và lộn xộn.
* **Quote từ app:**
  * *"Để tra cứu chính xác các chuyến bay từ Hà Nội đi Tokyo vào ngày 15/10/2025, Quý khách vui lòng cung cấp thêm thông tin về ngày bay mới (ngày 16/10/2025) nếu Quý khách muốn thay đổi theo yêu cầu trước đó ạ."*

#### 2. Evidence 2: Logic nghiệp vụ chồng chéo (Complex Edge Cases)
* **Screenshot:**
  ![Evidence 2: Complex Edge Cases](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/screenshot_2.png)
* **Prompt/Input đã thử:**
  * *"Tôi bay từ Hà Nội đi Paris hạng phổ thông, đi cùng 1 em bé 2 tuổi, mang theo 1 con chó Poodle nặng 5kg và 1 cây đàn cello 15kg. Tôi phải trả thêm bao nhiêu tiền phí và thủ tục thế nào?"*
* **Hành vi quan sát được:** Bot tách được các thực thể (em bé 2 tuổi, chó 5kg, đàn cello 15kg) nhưng không hề thực hiện tính toán tổng hợp phí. Thay vì đưa ra câu trả lời trực tiếp về giá tiền hoặc thủ tục tích hợp, bot liệt kê danh sách link điều khoản của từng mục riêng lẻ và bắt người dùng tự nhấn vào để đọc và tính toán thủ công.
* **Quote từ app:**
  * *"Mức phí: Quý khách vui lòng tra cứu bảng phí vận chuyển thú cưng trên khoang hành khách tại: https://www.vietnamairlines.com/content/vna/vn/vi/travel-information/special-services/traveling-with-pet/petc-charge"*
  * *"Quý khách vui lòng tra cứu phí vận chuyển theo hành trình cụ thể tại: https://www.vietnamairlines.com/vn/vi/travel-information/baggage/baggage-calculator"*

#### 3. Evidence 3: NLP với Slang/Teencode & Lỗi kẹt luồng (Context Confusion)
* **Screenshot:**
  ![Evidence 3: NLP & Context Leakage](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/screenshot_3.png)
* **Prompt/Input đã thử:**
  * Lượt 1 (Teencode): *"tui mún book vé đi sì gòn ngày mốt, giá rẽ nhứt nhiu vại shop?"*
  * Lượt 2: *"cho hoi huy ve mat phi k"*
  * Lượt 3 (Slang): *"Check hộ mình cái flight status chuyến VN213 tml (tomorrow) với"* -> Xác nhận.
  * Lượt 4: *"Thời tiết ở điểm đến Tokyo ngày mai thế nào? Có cần mang áo khoác không?"*
* **Hành vi quan sát được:**
  * **NLP:** Nhận diện được teencode "sì gòn" -> SGN. Tuy nhiên, từ "tml" (tomorrow) bị parse sai múi giờ/thời điểm thành ngày hôm nay (04/06/2026).
  * **Context Leakage:** Sau lượt 2 hỏi về hủy vé, bot bị kẹt vĩnh viễn trong luồng hoàn vé (sticky intent). Kể cả sau khi hoàn thành việc tra cứu chuyến bay ở lượt 3 hay thời tiết ở lượt 4, bot vẫn chèn thêm câu đòi mã đặt chỗ để hoàn vé.
* **Quote từ app:**
  * *"NEO xin phép được hỗ trợ Quý khách về điều kiện hoàn vé. Vui lòng cung cấp mã đặt chỗ để NEO kiểm tra chi tiết."*

#### 4. Evidence 4: Xử lý sự mâu thuẫn (Contradiction & Constraints)
* **Screenshot:**
  ![Evidence 4: Contradiction](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/screenshot_4.png)
* **Prompt/Input đã thử:**
  * *"Tôi muốn mua vé bay thẳng từ Hà Nội đi New York, thời gian bay dưới 5 tiếng, hạng thương gia giá dưới 1 triệu. ngày 15/10/2026. 1 người lớn thôi"*
* **Hành vi quan sát được:** Bot hoàn toàn phớt lờ các ràng buộc không thể xảy ra (Hà Nội - New York bay thẳng < 5h, giá < 1M). Bot vẫn thu thập slot địa điểm và ngày, sau đó hiển thị 3 chuyến bay nối chuyến tại Frankfurt với giá từ 126 triệu - 148 triệu VND và thời gian bay > 20 giờ. Không hề cảnh báo user về việc các ràng buộc bị vi phạm.
* **Quote từ app:**
  * *"NEO đã tìm thấy 3 mức giá vé tốt nhất cho ngày 15/10/2026 với các chuyến bay sau... Giá vé: 126,538,000 VND... Điểm nối chuyến: Frankfurt"*

#### 5. Evidence 5: Bẫy dữ liệu phi lý (Data Validation)
* **Screenshot:**
  ![Evidence 5: Data Validation](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/screenshot_5.png)
* **Prompt/Input đã thử:**
  * Lượt 1: *"Tôi muốn tra vé từ Hà Nội đi Đà Nẵng vào ngày 31/04/2026."*
  * Lượt 2: *"1 người lớn"*
* **Hành vi quan sát được:** Ở lượt 1, bot không validate ngày thực tế (tháng 4 chỉ có 30 ngày), vẫn ghi nhận slot `Ngày đi: 31/04/2026`. Đến khi khách chọn số lượng người ở lượt 2 và bot gửi request tìm vé lên hệ thống, API trả về lỗi và bot phản hồi bằng thông báo lỗi chung chung, bắt người dùng tự liên hệ CSKH mà không chỉ rõ sai ở đâu.
* **Quote từ app:**
  * *"Thông tin vẫn chưa chính xác, vui lòng thử lại sau hoặc liên hệ tư vấn viên để được hỗ trợ."*

---

## 3. Vẽ 4 paths

| Path | Câu hỏi cần trả lời | Thực tế tại Vietnam Airlines — NEO |
|---|---|---|
| **Happy** | Khi AI đúng và tự tin, user thấy gì? | Khi người dùng nhập một câu lệnh cơ bản, không có nhiễu ("Cho tôi vé đi Phú Quốc từ Hải Phòng"), bot nhận diện đúng điểm đi (HPH), điểm đến (PQC), tự động ghi đè các thực thể cũ (Nha Trang, Đà Lạt) và chuyển tiếp đến bước hỏi ngày đi. |
| **Low-confidence** | Khi AI không chắc, hệ thống có hỏi lại, show options hoặc chuyển người không? | Khi gặp logic phức tạp ("nếu hạng thương gia đắt hơn 5 triệu thì đổi sang phổ thông đi Đà Nẵng") hoặc lúc không thể khôi phục ngữ cảnh đặt vé cũ, bot lập tức cung cấp thông tin liên hệ và hiển thị nút "Gặp tư vấn viên" để chuyển tiếp cuộc gọi. |
| **Failure** | Khi AI sai, user biết bằng cách nào và sửa thế nào? | **1. Nhập ngày 31/04:** Bot vẫn ghi nhận slot. Chỉ khi gửi request tìm vé lên server và API crash, bot mới báo lỗi hệ thống chung chung. User không được chỉ rõ lỗi sai ngày, phải tự bắt đầu lại. <br>**2. Chặng bay New York:** Bot gợi ý chuyến bay hoàn toàn sai lệch ràng buộc ban đầu. User phải tự nhìn thấy giá vé 126M để biết bot đã phớt lờ điều kiện của mình. |
| **Correction** | Khi user sửa, correction có được lưu/log/học lại không hay biến mất? | Bot hỗ trợ sửa thông tin (ghi đè thực thể) ngay trong phiên thoại (ví dụ: đổi ngày bay từ 15 sang 16). Tuy nhiên, session state không lưu giữ lịch sử sửa đổi này để tối ưu hóa mô hình, thông tin sửa đổi sẽ biến mất hoàn toàn khi phiên kết thúc. |

---

## 4. Viết finding thành quyết định

- **Finding 1: Kẹt luồng hoàn vé (Context Leakage / Stuck Session State)**  
  Khi user hỏi về hủy/hoàn vé trước đó, sau đó chuyển sang hỏi về lịch bay VN213 hoặc thời tiết Tokyo,  
  AI vẫn tiếp tục chèn thêm câu nài nỉ hỏi mã đặt chỗ hoàn vé ở cuối mỗi phản hồi,  
  hậu quả là cuộc hội thoại bị chia cắt, gây cảm giác phiền hà và làm rối loạn luồng thông tin người dùng đang thực sự quan tâm.  
  Lỗi thuộc layer: **UX recovery / State Management**.  
  Nên sửa bằng: **UX / Fallback**: Tự động reset hoặc giải phóng (clear) intent hoàn vé khi người dùng đã trigger một intent khác rõ ràng (như hỏi thời tiết hoặc xem lịch bay), hoặc đặt thời gian hết hạn (session timeout) cho luồng hoàn vé sau 2 lượt hội thoại không liên quan.

- **Finding 2: Thiếu Date Validation ở tầng NLU**  
  Khi user nhập ngày đi không tồn tại trong thực tế (ngày 31/04/2026),  
  AI vẫn ghi nhận ngày đi bình thường và tiếp tục hỏi số lượng khách, sau đó báo lỗi hệ thống chung chung ở bước tìm kiếm cuối,  
  hậu quả là user mất thời gian hoàn thành các slot khác chỉ để nhận một thông báo lỗi không rõ nguyên nhân ở cuối luồng.  
  Lỗi thuộc layer: **Data-tool / Validation**.  
  Nên sửa bằng: **Requirement / Fallback**: Tích hợp thư viện validate lịch pháp thực tế ngay khi user cung cấp ngày đi. Nếu ngày không hợp lệ, bot lập tức báo lỗi cụ thể: *"Tháng 4 chỉ có 30 ngày. Quý khách vui lòng cung cấp ngày đi hợp lệ."*

- **Finding 3: Phớt lờ ràng buộc phi lý và tạo ra ảo tưởng (Ignoring Constraints & Hallucination)**  
  Khi user yêu cầu đặt vé với các điều kiện phi thực tế (bay thẳng HAN-JFK dưới 5 tiếng, giá dưới 1 triệu đồng),  
  AI phớt lờ hoàn toàn các ràng buộc về thời gian bay, loại chuyến bay và giá tiền, vẫn tiến hành tìm kiếm và trả về chuyến bay nối chuyến Frankfurt giá 126 triệu VND,  
  hậu quả là bot đưa ra đề xuất hoàn toàn sai lệch nhu cầu của khách hàng, làm mất lòng tin của khách vào sự thông minh của trợ lý ảo.  
  Lỗi thuộc layer: **Intent / Data-tool**.  
  Nên sửa bằng: **Requirement / Fallback**: Thêm bước pre-flight check đối với các từ khóa ràng buộc (bay thẳng, thời gian < X, giá < Y). Nếu không có kết quả nào thỏa mãn, bot cần thông báo trực tiếp: *"Hiện tại Vietnam Airlines không khai thác đường bay thẳng HAN - JFK dưới 5 tiếng với giá dưới 1 triệu VND. Sau đây là các phương án nối chuyến tối ưu nhất để quý khách tham khảo."*

- **Finding 4: Lệch múi giờ khi xử lý thời gian tương đối (Relative Time Mismatch)**  
  Khi user nhắn tin lúc nửa đêm (00:08 ngày 04/06) hỏi chuyến bay "ngày mai" (tomorrow/tml),  
  AI nhận dạng nhầm ngày khởi hành là ngày hôm nay (04/06) thay vì ngày mai (05/06),  
  hậu quả là hệ thống tìm kiếm sai lịch trình, hành khách có thể đặt nhầm ngày bay nếu không kiểm tra kỹ thông tin xác nhận.  
  Lỗi thuộc layer: **Data-tool (Entity Parsing)**.  
  Nên sửa bằng: **Test case / Requirement**: Sử dụng múi giờ của sân bay khởi hành làm mốc tham chiếu cho các từ khóa thời gian tương đối ("ngày mai", "ngày mốt"), kết hợp cấu hình tham số tham chiếu thời gian động của thư viện NLU (như Duckling) để tránh lỗi lệch ngày khi chat vào khung giờ đêm muộn.

---

## 5. Sketch as-is / to-be

Dưới đây là các sơ đồ so sánh luồng trải nghiệm hiện tại gặp lỗi (As-is) và luồng cải tiến đề xuất (To-be) được vẽ dưới dạng sketch tay trên bảng trắng (whiteboard sketch) bằng Tiếng Việt:

### AS-IS FLOW (Sơ đồ hiện tại — Các điểm gãy nghiệp vụ)

#### 1. Sơ đồ gãy luồng 1: Kẹt luồng & Rò rỉ ngữ cảnh (Context Leakage)
Sơ đồ mô tả điểm gãy khi người dùng chuyển sang hỏi thông tin khác (Check chuyến bay) nhưng chatbot vẫn bị kẹt cứng ở ý định cũ (nài nỉ đòi mã đặt chỗ hoàn vé).
![Sơ đồ As-Is: Kẹt luồng](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/sketch_as_is_context.png)

#### 2. Sơ đồ gãy luồng 2: Lỗi Validation ngày & Bỏ qua ràng buộc (Ignore Constraints)
Sơ đồ mô tả việc bot chấp nhận ngày phi lý (31/04) và phớt lờ các ràng buộc khắt khe của người dùng (bay thẳng đi New York < 5h, giá < 1M) để đưa ra kết quả sai lệch.
![Sơ đồ As-Is: Lỗi Validation & Ràng buộc](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/sketch_as_is_logic.png)

---

### TO-BE FLOW (Sơ đồ đề xuất — Các cải tiến kỹ thuật)

#### 1. Sơ đồ sửa lỗi 1: Quản lý trạng thái & Reset Session (State Management)
Sơ đồ đề xuất giải pháp tự động giải phóng (reset) trạng thái hoàn vé cũ (`Refund State = NULL`) ngay khi NLU phát hiện ý định mới từ người dùng.
![Sơ đồ To-Be: Quản lý trạng thái](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/sketch_to_be_context.png)

#### 2. Sơ đồ sửa lỗi 2: Validate ngày & Ràng buộc ở Front-end
Sơ đồ đề xuất tích hợp Validation Engine và Constraint Checker để chủ động chặn dữ liệu phi lý (ngày không tồn tại) và cảnh báo người dùng về giới hạn chặng bay ngay từ bước nhập liệu.
![Sơ đồ To-Be: Validate ngày & Ràng buộc](file:///Users/anhtuan/Desktop/Vinuni/Lab5/01-invidual-workshop/images/sketch_to_be_logic.png)

---

## 6. Tự kiểm trước khi nộp

- [x] Có ít nhất 1 screenshot hoặc observation cụ thể.
- [x] Có đủ 4 paths hoặc nói rõ path nào chưa có trong product.
- [x] Finding được viết thành product decision, không chỉ là nhận xét.
- [x] Sketch có as-is và to-be.
- [x] Có một câu nói rõ finding này sẽ đổi gì trong SPEC.

> **Quyết định thay đổi SPEC cốt lõi:**  
> Bổ sung vào Product Spec quy định về **State Management Lifespan** (tự động xóa bỏ intent cũ sau 2 lượt thoại không liên quan để tránh Context Leakage) và tích hợp **Date & Constraint Validation Middleware** vào trước bước Booking API Call để chặn và xử lý thông tin phi lý ngay tại Front-end Chatbot.