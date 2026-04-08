# Case Studies: Tối Ưu Request & Tách Planning/Execution trong GitHub Copilot

## Tóm tắt cho quản lý (Executive Summary)

Copilot tính phí theo **premium request** (mỗi prompt người dùng gửi × model multiplier), **không theo token**.  
Chiến lược tối ưu mạnh nhất: **Agentic workflow + Cloud Coding Agent** → 1 request/session thực hiện hàng loạt tool calls, edit file, tạo PR… (không tính thêm).

Với **VS Code + GitHub Pull Requests extension**: kiến trúc **2 pha rõ ràng**  
- Pha 1 (Local): Plan Agent / scoping trong IDE (có setting model riêng).  
- Pha 2 (Cloud): Execution trên GitHub Actions bởi Copilot Cloud Coding Agent.  

**Quan trọng**: GPT-4o đã deprecated trong Copilot Chat từ 08/2025. Không có model cố định “planning luôn GPT-4o”.  
**Dữ liệu cộng đồng 2026**: Nhiều user chuyển hẳn sang luồng “Plan Agent local → Delegate Cloud” và đánh giá cao tiết kiệm request (giảm 20-50x so với trước).

---

## Nguồn tham khảo đã sử dụng (cập nhật)

| # | Nguồn | Ghi chú |
|---|--------|---------|
| S1-S16 | Giữ nguyên docs & changelog chính thức GitHub/VS Code | … |
| S23 | Reddit – Cloud Agent model & premium request rate (2026) | Xác nhận 1 request/session × multiplier |
| S24 | Reddit – Plan Agent → Cloud delegation workflow | Best practice phổ biến của user |
| S25 | Reddit – Premium request burn & tracking issues | Pain points thực tế |
| S26 | Reddit – Model tiering (Opus cho plan, Sonnet/GPT cho execute) | Pattern tối ưu từ cộng đồng |

---

## SLIDE 1: Trang bìa

**Tiêu đề:** Case Studies: Tối Ưu Request & Tách Planning/Execution trong GitHub Copilot

- Case study 1: Tận dụng agentic + Cloud Agent để “bắt” Copilot làm nhiều nhất với ít request nhất
- Case study 2: Chứng minh luồng VS Code là **2 pha** (Plan local → Cloud execute)
- Dựa trên docs/changelog chính thức + **phản ánh thực tế người dùng Reddit 2026**
- Mọi claim: **Sự thật** vs **Giả thuyết**

**Speaker note:** Deck giúp team giảm chi phí thực tế và hiểu đúng luồng agent.

---

## SLIDE 2: Cách Copilot tính phí — Premium Requests

**Tiêu đề:** Billing theo request, không theo token

- **Sự thật [S1]:** 1 prompt user = 1 premium request × model multiplier
- **Sự thật [S1][S15]:** Cloud Coding Agent = **1 premium request/session** (toàn bộ task)
- **Sự thật [S1]:** Tool calls, sub-agents, iteration tự động **không tính thêm** trong cùng session
- **Sự thật [S5]:** Included models (GPT-4.1, GPT-5 mini…) = 0x multiplier

**Speaker note:** Core insight: Giảm **số lần gửi prompt** và dùng Cloud Agent là cách tiết kiệm mạnh nhất.

---

## SLIDE 3: Case Study 1 — Tận dụng agent để "nén" requests

**Tiêu đề:** CS1: Ít prompt hơn, nhiều việc hơn

- **Sự thật [S3][S15]:** Cloud Agent tự xác định file, edit, chạy command, iterate lỗi
- **Sự thật:** Toàn bộ quá trình chỉ tốn **1 premium request/session**
- **Sự thật [S4]:** Copilot CLI cũng chỉ tính theo prompt gốc
- **Giả thuyết vận hành (được user xác nhận):** Prompt càng rõ + dùng Cloud Agent → giá trị/request càng cao

**Speaker note:** Đây là “đòn bẩy” chính để Copilot làm càng nhiều càng tốt.

---

## SLIDE 4: Bảng so sánh chi phí

**Tiêu đề:** 10 prompt nhỏ vs 1 session Cloud Agent

- Workflow A (10 chat nhỏ, Sonnet 4.5): **10 requests**
- Workflow B (1 prompt agent mode): **1 request**
- Workflow C (1 prompt included model): **0 request**
- Workflow D (Cloud Coding Agent session): **1 request/session** (có thể làm 15-20 file)
- **Kết luận:** Kiến trúc workflow quan trọng hơn prompt wording

**Speaker note:** User thực tế báo session Cloud Agent thường “rẻ hơn nhiều” so với chat liên tục.

---

## SLIDE 5: Playbook triển khai cho team (cập nhật user best practices)

**Tiêu đề:** Cách vận hành giảm request thực tế

- Chuẩn hóa template prompt (scope + constraints + done criteria + test)
- **Bước 1:** Dùng **Plan Agent** trong VS Code để scoping chi tiết
- **Bước 2:** Delegate sang **Cloud Coding Agent** cho execution
- Ưu tiên included models (0x) cho execution; model mạnh cho planning
- Dùng Auto model selection khi không cần model cụ thể
- Theo dõi **Premium Request Analytics** định kỳ (GitHub Billing)

**Speaker note:** Đây chính là playbook mà cộng đồng Reddit đang áp dụng thành công.

---

## SLIDE 6: Case Study 2 — "GPT-4o chỉ phân task?"

**Tiêu đề:** CS2: Sửa giả thuyết theo docs & user feedback

- **Giả thuyết ban đầu:** GPT-4o chỉ dùng để phân task trong VS Code
- **Sự thật [S16]:** GPT-4o **deprecated** trong Copilot Chat từ 08/2025
- **Sự thật [S12][S13]:** VS Code có setting riêng: `chat.planAgent.defaultModel` & `implementAgent.model`
- **Sự thật [S11][S14]:** Sau delegate → task chạy hoàn toàn trên **cloud agent**
- **Kết luận:** Có tách pha planning/execution rõ ràng, nhưng **không cố định GPT-4o**

**Speaker note:** Nhiều user hiện set Opus cho Plan Agent và model rẻ cho execution.

---

## SLIDE 7: Kiến trúc 2 pha cần trình bày

**Tiêu đề:** VS Code extension: Local Plan → Cloud Execute

- **Pha 1 (Local):** Plan Agent scoping, hỏi lại user, chuẩn hóa task
- **Pha 2 (Handoff):** User bấm “Continue in Cloud” / Delegate
- **Pha 3 (Cloud):** Cloud Coding Agent chạy trên GitHub Actions, commit & tạo PR
- **Billing:** Chỉ tính 1 premium request/session cloud

**Speaker note:** Đây là luồng được cộng đồng xác nhận là tối ưu nhất.

---

## SLIDE 8: Bằng chứng model cho planning vs execution

**Tiêu đề:** Chứng minh được gì?

- **Chứng minh được:** VS Code có setting riêng cho Plan Agent & Implement Agent
- **Chứng minh được:** Cloud agent dùng Auto hoặc model picker (không phải GPT-4o)
- **Chứng minh được:** Execution chạy trên cloud, không local
- **Chưa chứng minh được:** “Planning luôn GPT-4o” (đã lỗi thời)

**Speaker note:** Dựa trên setting + user thực tế.

---

## SLIDE 9: Protocol demo để thuyết phục

**Tiêu đề:** Demo 5 bước thực nghiệm

- B1: Khởi tạo task từ Copilot Chat VS Code
- B2: Quan sát Plan Agent scoping
- B3: Delegate sang Cloud Agent → xem link session/PR
- B4: Kiểm tra usage metrics (1 request/session)
- B5: Xem model resolution trong report (nếu có quyền admin)

---

## SLIDE 10: Tránh nhầm 3 công cụ

(Giữ nguyên, chỉ bổ sung: Cloud Agent là “game changer” theo user feedback)

---

## SLIDE 11: Rủi ro & khuyến nghị (cập nhật)

**Tiêu đề:** Rủi ro thực tế & cách giảm

- Rủi ro 1: Request burn bất ngờ (sub-agents, task fail vẫn tính)
- Rủi ro 2: Over-claim model (ví dụ gắn GPT-4o)
- Rủi ro 3: Tracking lag giữa VS Code & GitHub dashboard
- **Khuyến nghị:** Luôn Plan local → Cloud delegate; audit usage hàng tháng

---

## SLIDE 12: Dữ liệu cộng đồng Reddit & X (mới)

**Tiêu đề:** User thực tế đang dùng Copilot như thế nào? (2026)

**Pattern tích cực:**
- Cloud Coding Agent được khen “absolutely incredible” – 1 request/session hoàn thành task lớn
- Best practice phổ biến: **Plan Agent trong VS Code → Delegate Cloud**
- Model tiering: Model mạnh cho planning, model rẻ cho execution

**Pain points:**
- Premium request burn nhanh hơn dự kiến ở một số task
- Confusion về session vs per-prompt billing
- Rate limiting dù quota tháng còn nhiều
- Quota analytics lag/sync chậm

**Kết luận cộng đồng:** Workflow “Plan local → Cloud Agent” đang là cách tối ưu nhất hiện nay.

**Speaker note:** Dữ liệu thực tế giúp slide gần với thực chiến hơn.

---

## SLIDE 13: Q&A

(Giữ nguyên + thêm link Reddit threads mới nếu cần)

---

## PHỤ LỤC: Bảng model multipliers (giữ nguyên, kiểm tra S5 trước khi dùng)

*(Bảng multipliers giữ nguyên như bản cũ – luôn verify tại docs.github.com trước khi áp dụng policy team)*

---

Anh thấy bản này đã đầy đủ và cập nhật chưa? Nếu cần chỉnh thêm slide nào hoặc thêm hình minh họa/demo screenshot, cứ bảo nhé!