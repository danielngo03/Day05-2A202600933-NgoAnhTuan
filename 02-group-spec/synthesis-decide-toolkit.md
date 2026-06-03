# Synthesis Decide Toolkit

> File này dùng sau Evidence Pack và Thin SPEC để chốt build slice đủ nhỏ cho Day 06.  

---

## 1. Gom evidence thành cụm

Gom theo **workflow/pain**, không gom theo tên feature.

### Cụm 1 — Thông tin phòng/tiện ích bị rải rác, user phải tự đọc và tự ghép

**Evidence liên quan:**

- Self-use trên OTA/website khách sạn cho thấy thông tin phòng thường bị tách ở nhiều nơi: room card, facilities, policies, gallery, FAQ, nearby places.
- Một câu hỏi tự nhiên như “phòng nào cho 2 người, có bồn tắm, từ 35m2 trở lên?” buộc user phải tự kiểm tra nhiều field: sức chứa, diện tích, tiện ích, ảnh, giá tham khảo.
- Travel apps/OTA có nhiều dữ liệu nhưng thường trình bày theo tab/filter/card, chưa tối ưu cho việc hỏi đáp theo nhu cầu tự nhiên.

**Pain pattern:**

```text
User không thiếu dữ liệu.
User bị quá tải vì dữ liệu bị rải nhiều nơi và phải tự ghép thành quyết định.
```

**Điểm gãy trong workflow:**

1. User mở trang khách sạn.
2. User đọc mô tả chung.
3. User mở từng hạng phòng.
4. User đối chiếu tiện ích/phòng/ảnh/giá/chính sách.
5. User vẫn không chắc phòng nào phù hợp nhất.

**Path liên quan:** Happy / Low-confidence

**SPEC implication:**

- Build slice phải ưu tiên `room_search`, `room_compare`, `room_detail` hơn FAQ chung chung.
- Data cần có `room_types`, `area_m2`, `max_guests`, `bed_type`, `room_amenities`, `room_images`, `price_from`.
- Bot cần trả lời có lý do match, không chỉ đưa tên phòng.

---

### Cụm 2 — User dễ hiểu nhầm ảnh, tiện ích, phí và điều kiện sử dụng

**Evidence liên quan:**

- Self-use cho thấy ảnh chung của khách sạn và ảnh từng phòng dễ bị lẫn nếu gallery không map rõ theo hạng phòng.
- Câu hỏi như “ảnh này thuộc phòng nào?”, “bữa sáng có miễn phí không?”, “hồ bơi/gym có mất phí không?” thường cần thông tin chi tiết hơn là chỉ biết tiện ích có tồn tại.
- Public evidence/review trong hospitality cho thấy complaint thường liên quan tới room, bathroom, facilities, breakfast, location, reservation/promotion hoặc kỳ vọng trước khi ở không khớp trải nghiệm thực tế.

**Pain pattern:**

```text
User không chỉ muốn biết khách sạn “có tiện ích hay không”.
User cần biết tiện ích đó thuộc phòng nào, miễn phí hay tính phí, có điều kiện gì và nguồn thông tin ở đâu.
```

**Điểm gãy trong workflow:**

1. User thấy tiện ích/ảnh trong listing.
2. User không biết tiện ích áp dụng cho toàn khách sạn hay riêng một hạng phòng.
3. User không rõ miễn phí/tính phí/điều kiện dùng.
4. User hình thành kỳ vọng sai trước khi đặt.

**Path liên quan:** Low-confidence / Correction

**SPEC implication:**

- Data cần có field `is_free`, `fee_note`, `applies_to`, `source_note`, `image_type`, `room_id`.
- Bot phải nói rõ “mình chưa có dữ liệu phí/điều kiện” nếu field thiếu.
- Bot không được tự suy diễn kiểu “miễn phí”, “view đẹp nhất”, “phù hợp ngắm hoàng hôn” nếu source không ghi.

---

### Cụm 3 — User thường trộn câu hỏi thông tin với câu hỏi booking/availability real-time

**Evidence liên quan:**

- Self-use cho thấy khi tìm hiểu khách sạn, user rất dễ hỏi tiếp: “ngày này còn phòng không?”, “đặt luôn được không?”, “giá tối nay bao nhiêu?”.
- Đây là câu hỏi cần availability, pricing, booking engine hoặc OTA/PMS integration; không thể trả lời đúng nếu chỉ có Google Sheet/JSON tĩnh.
- Thin SPEC đã xác định bot không xử lý đặt phòng, hủy/đổi, hoàn tiền, check ngày cụ thể hoặc giá real-time.

**Pain pattern:**

```text
User không phân biệt rõ “hỏi thông tin khách sạn” và “thực hiện giao dịch đặt phòng”.
Nếu bot không có boundary, bot rất dễ tạo kỳ vọng sai.
```

**Điểm gãy trong workflow:**

1. User hỏi thông tin phòng.
2. Bot trả lời phòng phù hợp.
3. User hỏi tiếp availability/booking.
4. Nếu bot trả lời bừa, user tưởng thông tin đã được xác nhận.

**Path liên quan:** Failure

**SPEC implication:**

- Prototype bắt buộc có intent detection cho `out_of_scope_booking`, `out_of_scope_availability`, `out_of_scope_real_time_price`.
- Failure path phải được demo rõ, không được bỏ qua.
- Bot phải fallback sang booking link/lễ tân/OTA, không nhận đặt phòng và không khẳng định còn phòng.

---

### Cụm 4 — AI travel hữu ích cho hỏi đáp tự nhiên nhưng rủi ro nếu không grounded vào source

**Evidence liên quan:**

- Competitor/analog như Booking.com AI Trip Planner, Expedia in ChatGPT, Marriott Help, Hilton/IBM Connie cho thấy travel user có nhu cầu hỏi bằng hội thoại tự nhiên.
- Tuy nhiên, các case booking/giá/availability vẫn cần hệ thống transaction hoặc nguồn chính thức.
- Với dữ liệu khách sạn tĩnh, LLM phù hợp để hiểu câu hỏi, map intent và tóm tắt có cấu trúc; không phù hợp để tự quyết hoặc tự hành động như agent booking.

**Pain pattern:**

```text
AI có thể làm trải nghiệm hỏi đáp nhanh hơn,
nhưng nếu AI trả lời tự tin mà không dẫn nguồn thì rủi ro còn lớn hơn việc user tự đọc.
```

**Điểm gãy trong workflow:**

1. User hỏi câu tự nhiên.
2. AI hiểu intent nhưng không retrieve đúng data.
3. AI suy diễn từ kiến thức chung.
4. User tin nhầm câu trả lời không có source.

**Path liên quan:** Low-confidence / Failure / Correction

**SPEC implication:**

- Không build full agent.
- Chọn LLM feature + structured retrieval/RAG.
- Câu trả lời phải có source/field nguồn.
- Cần confidence threshold và fallback khi thiếu dữ liệu.

---

## 2. Viết insight

Form:

```text
User [segment] không chỉ cần [surface need].
Họ thật ra cần [deeper need],
vì [evidence pattern].
```

### Insight chính

```text
Khách đang tìm hiểu khách sạn trước khi đặt không chỉ cần xem danh sách phòng và tiện ích.
Họ thật ra cần một cách hỏi nhanh, đáng tin và có nguồn để biến dữ liệu rời rạc thành quyết định chọn phòng phù hợp,
vì evidence từ self-use cho thấy thông tin phòng, ảnh, tiện ích, phí và chính sách bị phân tán ở nhiều tab/card, còn review/complaint khách sạn cho thấy kỳ vọng sai về phòng và facilities có thể dẫn tới thất vọng hoặc khiếu nại.
```

### Insight phụ 1 — Trust quan trọng hơn trả lời nhiều

```text
Khách không chỉ cần bot trả lời nhanh.
Họ cần biết câu trả lời đến từ đâu và phần nào chưa chắc,
vì các câu hỏi về phí tiện ích, ảnh phòng, chính sách và availability nếu bị trả lời sai sẽ tạo kỳ vọng sai trước khi đặt.
```

### Insight phụ 2 — Scope phải hẹp để demo tốt

```text
Nhóm không chỉ cần một chatbot du lịch rộng.
Nhóm cần một slice đủ hẹp để chứng minh AI xử lý tốt một workflow cụ thể,
vì Day 06 chỉ có thể demo trong thời gian ngắn và các tác vụ booking/availability cần dữ liệu real-time nằm ngoài khả năng của prototype M1.
```

### Insight phụ 3 — AI phù hợp ở bước hiểu câu hỏi và tổng hợp, không phù hợp để tự giao dịch

```text
User không chỉ cần filter cứng như trên OTA.
Họ cần hỏi bằng tiếng Việt tự nhiên và nhận câu trả lời đã tổng hợp,
vì evidence self-use cho thấy cùng một nhu cầu có thể được diễn đạt rất đời thường như “phòng nào cho gia đình 4 người có bếp không”.
```

---

## 3. Viết opportunity

Form:

```text
Cơ hội là dùng AI để [augment/automate hành động hẹp],
giúp user [kết quả],
trong khi vẫn kiểm soát [failure/risk].
```

### Opportunity chính

```text
Cơ hội là dùng AI để tự động hiểu câu hỏi tự nhiên của khách và truy xuất đúng dữ liệu khách sạn từ Google Sheet/JSON,
giúp user nhanh chóng biết phòng/tiện ích nào phù hợp với nhu cầu của họ,
trong khi vẫn kiểm soát rủi ro hallucination bằng source, confidence threshold, fallback khi thiếu dữ liệu và chặn các câu hỏi booking/availability/giá real-time.
```

### Opportunity cụ thể cho prototype

```text
Cơ hội là build một chatbot Q&A hẹp cho khách sạn,
cho phép user hỏi các câu như “phòng nào có bồn tắm cho 2 người?”, “hồ bơi có miễn phí không?”, “gần khách sạn có chỗ ăn không?”,
và bot trả lời bằng dữ liệu có cấu trúc kèm source/ảnh/ghi chú thiếu dữ liệu,
thay vì bắt user tự mở từng tab, từng room card và tự suy luận.
```

### Opportunity không làm trong Day 06

```text
Không dùng AI để đặt phòng, giữ phòng, hủy phòng, báo giá real-time hoặc xác nhận còn phòng,
vì các tác vụ đó cần hệ thống booking/PMS/OTA và có rủi ro cao nếu prototype trả lời sai.
```

---

## 4. Chọn build slice

Build slice tốt phải qua 5 câu hỏi:

| Câu hỏi | Đạt khi | Đánh giá cho dự án | Kết luận |
|---|---|---|---|
| User cụ thể chưa? | Nói được ai dùng, trong bối cảnh nào. | User là khách du lịch/khách chuẩn bị đặt khách sạn, đang tìm hiểu phòng và tiện ích trước khi quyết định đặt. Bối cảnh là họ đang xem website/OTA/chatbot của một khách sạn cụ thể. | Đạt |
| Task đủ hẹp chưa? | Demo được trong 3–5 phút. | Task chỉ là hỏi đáp thông tin khách sạn/phòng/tiện ích/nearby places từ Google Sheet/JSON. Demo 4 path bằng 4–6 câu hỏi là đủ. | Đạt |
| AI decision rõ chưa? | AI gợi ý/tự làm một việc cụ thể. | AI tự động hiểu intent, lọc dữ liệu, tổng hợp câu trả lời và đưa source. AI không tự booking, không tự thanh toán, không tự xác nhận availability. | Đạt |
| Failure path rõ chưa? | Có một case AI không chắc hoặc sai để test. | Failure nguy hiểm nhất là user hỏi ngày còn phòng/giá real-time/đặt phòng, AI có thể hallucinate. Prototype sẽ detect và fallback. | Đạt |
| Có evidence không? | Có bằng chứng từ self-use/review/user/competitor. | Có self-use trên app/web travel, evidence review/complaint hospitality và competitor/analog AI travel. | Đạt |

### Build slice cuối cùng

```text
Hotel Amenities Q&A Chatbot:
Một chatbot hỏi đáp thông tin khách sạn giúp khách trước khi đặt có thể hỏi bằng tiếng Việt tự nhiên về hạng phòng, tiện ích, ảnh, giá tham khảo nếu có, chính sách đã biết và địa điểm xung quanh; bot trả lời từ Google Sheet/JSON có source, đồng thời từ chối/fallback với booking, hủy/đổi, availability theo ngày và giá real-time.
```

## 5. Quyết định: giữ, giảm scope, hay đổi hướng?

| Tình huống | Quyết định | Áp dụng cho dự án |
|---|---|---|
| Evidence yếu, user mơ hồ | Dừng build sâu; quay lại research 20 phút. | Không chọn. Evidence hiện đủ để build slice hẹp: self-use, review/complaint, competitor/analog. |
| Ý tưởng quá rộng | Giữ domain, cắt xuống một flow. | Có áp dụng. Ban đầu domain Travel & Hospitality rộng, đã cắt xuống một flow: hỏi đáp tiện ích/phòng khách sạn trước khi đặt. |
| AI không cần thiết | Dùng rule/manual prototype; ghi rõ vì sao không dùng AI sâu. | Không hoàn toàn. Rule có thể xử lý FAQ đơn giản, nhưng AI cần thiết để hiểu câu hỏi tự nhiên, nhiều điều kiện, tiếng Việt không chuẩn và tổng hợp từ nhiều bảng. |
| Rủi ro cao | Chọn augmentation hoặc conditional automation. | Có áp dụng. Chọn Conditional automation: AI tự trả lời case hẹp có source, fallback case rủi ro. |
| Không demo được trong 1 ngày | Đưa phần lớn vào backlog, giữ một path nhỏ. | Có áp dụng. Booking, availability real-time, PMS/OTA integration, payment, live staff handoff được đưa vào backlog. |

### Quyết định cuối

**Giữ domain, giảm scope mạnh, build slice hẹp.**

```text
Nhóm không build “AI hotel concierge” rộng.
Nhóm build “Hotel Amenities Q&A Chatbot” hẹp, tập trung vào hỏi đáp thông tin phòng/tiện ích/ảnh/nearby places từ dữ liệu tĩnh có cấu trúc.
```

### Lý do

- Evidence chỉ ra pain rõ nhất là user phải tự đọc và tự ghép thông tin khách sạn/phòng/tiện ích.
- Day 06 cần demo nhanh, nên không nên chọn luồng booking/availability phức tạp.
- AI có giá trị rõ trong việc hiểu câu hỏi tự nhiên và tổng hợp câu trả lời từ dữ liệu có cấu trúc.
- Rủi ro hallucination cao, nên cần source, confidence và fallback thay vì full automation.

### SPEC sau quyết định

**User chính:** Khách đang tìm hiểu khách sạn trước khi đặt.  
**Pain chính:** Khó tìm/so sánh thông tin phòng, tiện ích, ảnh, phí và chính sách vì dữ liệu rải rác.  
**Build slice:** Q&A chatbot trên Google Sheet/JSON khách sạn.  
**AI decision:** Conditional automation.  
**Boundary:** Không booking, không availability real-time, không giá theo ngày, không hủy/đổi/hoàn tiền.  
**Failure path bắt buộc:** User hỏi “ngày X còn phòng không/đặt giúp tôi”.  

---

## 6. Câu chốt cuối

Điền câu này trước khi rời lớp:

```text
Dựa trên evidence từ self-use trên các app/web travel, review/complaint khách sạn và competitor/analog AI travel,
nhóm sẽ build Hotel Amenities Q&A Chatbot,
cho khách đang tìm hiểu khách sạn trước khi đặt,
để giải quyết pain phải tự đọc và tự ghép thông tin phòng, tiện ích, ảnh, phí và chính sách bị rải rác,
bằng cách AI tự động hiểu câu hỏi tiếng Việt tự nhiên, retrieve dữ liệu từ Google Sheet/JSON và trả lời có source,
và sẽ test failure path user hỏi availability/booking/giá real-time như “Ngày 15/06 còn phòng Deluxe không? Đặt luôn giúp tôi”.
```

---

## 7. Backlog

Những thứ **không build trong Day 06**:

- Booking/đặt phòng trực tiếp trong chatbot.
- Check availability theo ngày, theo số đêm, theo số khách real-time.
- Giá real-time theo ngày, mùa, khuyến mãi hoặc OTA.
- Hủy phòng, đổi ngày, hoàn tiền, xử lý booking đã có.
- Tích hợp PMS/channel manager/OTA/booking engine.
- Thanh toán, giữ phòng, đặt cọc.
- Tự động gửi email/SMS/Zalo xác nhận booking.
- Live handoff cho lễ tân trong production.
- Tự crawl dữ liệu từ website/OTA theo thời gian thực.
- Gợi ý itinerary phức tạp theo thời tiết, giờ mở cửa, giao thông real-time.
- Cá nhân hóa theo lịch sử khách hàng, loyalty tier hoặc CRM.
- Multi-hotel search hoặc so sánh nhiều khách sạn cùng lúc.
- Voice chatbot/call center.
- Đa ngôn ngữ ngoài tiếng Việt/tiếng Anh cơ bản.
- Admin dashboard hoàn chỉnh để khách sạn tự sửa data.

### Backlog ưu tiên trong Day 06

| Ưu tiên | Item | Lý do để sau |
|---|---|---|
| P1 | Admin chỉnh sửa Google Sheet/JSON dễ hơn | Cần sau khi prototype chứng minh được Q&A flow. |
| P1 | Log unanswered/correction thành bảng riêng | Quan trọng để cải thiện data, nhưng M1 chỉ cần log đơn giản. |
| P2 | Booking link handoff theo room type | Có ích nhưng vẫn chưa phải booking automation. |
| P2 | Multi-language answer | Có thể mở rộng sau nếu user quốc tế. |
| P3 | PMS/OTA integration | Phức tạp, cần API/permission, không phù hợp Day 06. |
| P3 | Real-time nearby recommendations | Cần dữ liệu giờ mở cửa/khoảng cách real-time, ngoài scope M1. |

---