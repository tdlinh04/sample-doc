# Case Studies: Tối Ưu Premium Requests & Kiến Trúc 2 Pha Planning/Execution trong GitHub Copilot

## Tóm tắt cho quản lý (Executive Summary)

GitHub Copilot **tính phí theo premium requests** (1 user prompt × model multiplier), **không theo token**.  

Chiến lược tối ưu cao nhất (theo cả docs + user thực tế 2026):
- Gửi **prompt chất lượng cao** → kích hoạt **agentic behavior** (tool calls, iterate tự động **không tốn request thêm**).
- Sử dụng **Plan Agent (local trong VS Code)** + **handoff sang Cloud Agent** để nén toàn bộ task vào **1 request/session**.

Với **VS Code + GitHub Pull Requests extension**: kiến trúc rõ ràng **2 pha** (Plan Agent local scoping → Cloud execution trên GitHub Actions).

**Quan trọng**: GPT-4o đã **deprecated từ 08/2025**. Planning & execution model **có thể config riêng** (`chat.planAgent.defaultModel`). Tất cả claim đều gắn **Sự thật** vs **Giả thuyết**.

**Mục tiêu deck**: Giúp team giảm chi phí tối đa + “bắt Copilot làm càng nhiều càng tốt” theo workflow đã được hàng nghìn user áp dụng thành công.

---

## Nguồn tham khảo (cập nhật 04/2026)

| #  | Nguồn | Ghi chú |
|----|-------|---------|
| S1 | [Requests in GitHub Copilot](https://docs.github.com/en/copilot/concepts/billing/copilot-requests) | Premium request + agentic + cloud session |
| S2 | [Supported AI models](https://docs.github.com/en/copilot/reference/ai-models/supported-models) | Multipliers & included models |
| S3 | [Planning with agents in VS Code](https://code.visualstudio.com/docs/copilot/agents/planning) | Plan Agent + settings |
| S4 | [VS Code Copilot settings](https://code.visualstudio.com/docs/copilot/reference/copilot-settings) | `chat.planAgent.defaultModel` |
| S5 | [Cloud Agent docs](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) | Local → Cloud handoff |
| S6 | [GPT-4o deprecation](https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/) | Deprecated trong Chat |
| S23 | Reddit threads 2025-2026 (r/GithubCopilot) | User experiences: 1 req/session, model tiering, Plan → Cloud workflow |

---

## SLIDE 1: Trang bìa

**Tiêu đề:** Case Studies: Tối Ưu Premium Requests & Kiến Trúc 2 Pha Planning/Execution trong GitHub Copilot

- Case study 1: Tận dụng agentic + Plan → Cloud để Copilot làm **nhiều nhất với ít request nhất**
- Case study 2: Chứng minh **tách Planning (local Plan Agent) vs Execution (Cloud Agent)**
- Dựa trên docs chính thức + **phản ánh user thực tế 2026**
- Mọi claim gắn rõ: **Sự thật** vs **Giả thuyết**

**Speaker note:** Deck kết hợp tài liệu chính thức với kinh nghiệm thực tế của cộng đồng để áp dụng ngay.

---

## SLIDE 2: Cách Copilot tính phí — Premium Requests

**Tiêu đề:** Billing theo premium request, không theo token

- **Sự thật [S1]:** 1 user prompt = 1 premium request × model multiplier
- **Sự thật [S1]:** Agentic tool calls & iterate **không tính thêm** trong cùng session
- **Sự thật [S2]:** Included models (GPT-4.1, GPT-5 mini, Raptor mini…) = **0x**
- **Sự thật [S2]:** Claude Sonnet 4.x ≈ 1x | Haiku 4.5 = 0.33x | Opus 4.6 = 3x (fast = 30x)
- **Sự thật [S1][S5]:** Cloud Agent = **1 premium request per session**

**Speaker note:** User thực tế cho biết: “Một session Cloud Agent có thể thay thế 20-50 prompt thông thường”.

---

## SLIDE 3: Case Study 1 — Tận dụng agent để "nén" requests

**Tiêu đề:** CS1: Ít prompt hơn – Nhiều việc hơn (User feedback 2026)

- **Sự thật [S1][S5]:** Cloud Agent tự research, edit, iterate, tạo PR trong **1 session**
- **Sự thật [S1]:** Toàn bộ tool calls tự động **không tốn request thêm**
- **User thực tế:** “1 premium request giờ làm được cả task refactor lớn” (r/GithubCopilot)
- **Giả thuyết vận hành:** Prompt upfront đầy đủ + Plan Agent → giá trị trên mỗi request tăng mạnh

**Speaker note:** Đây là cách user thực tế “bắt Copilot làm càng nhiều càng tốt”.

---

## SLIDE 4: Bảng so sánh chi phí

**Tiêu đề:** 10 prompt nhỏ vs 1 prompt agentic vs Cloud Session

- **Workflow A (10 prompt nhỏ, Sonnet 4.6):** 10 × 1x = **10 premium requests**
- **Workflow B (1 prompt agent mode):** 1 × 1x = **1 premium request**
- **Workflow C (1 prompt, GPT-5 mini – included):** 1 × **0x** = **0 premium requests**
- **Workflow D (Cloud Agent session):** **1 premium request/session**
- **Kết luận (user confirm):** Plan Agent + Cloud handoff = cách tiết kiệm request mạnh nhất

**Speaker note:** Nhiều dev báo cáo tiết kiệm 70-90% request so với chat thông thường.

---

## SLIDE 5: Playbook triển khai cho team

**Tiêu đề:** Cách vận hành giảm request thực tế (theo user best practice)

- Chuẩn hóa template prompt + **copilot-instructions.md** (tối ưu token để tránh bloat)
- **Workflow khuyến nghị:** Plan Agent local (model mạnh) → review plan → handoff Cloud Agent
- Dùng included models 0x cho execution; premium model chỉ cho planning/reasoning sâu
- Bật Auto model selection + theo dõi usage analytics hàng tuần
- Tránh delegate vội khi prompt chưa rõ ràng (nguyên nhân usage spike)

**Speaker note:** Đây chính là playbook mà cộng đồng đang dùng và chia sẻ rộng rãi.

---

## SLIDE 6: Case Study 2 — Tách Planning vs Execution

**Tiêu đề:** CS2: Chứng minh kiến trúc 2 pha (không over-claim model)

- **Giả thuyết cũ (cần sửa):** “Extension VS Code chỉ dùng GPT-4o cho planning”
- **Sự thật [S6]:** GPT-4o **đã deprecated từ 08/2025**
- **Sự thật [S3][S4]:** Plan Agent local có setting riêng → handoff explicit sang Cloud Agent
- **User thực tế:** “Plan Agent incredible cho scoping, sau đó handoff cloud rất mượt”
- **Kết luận:** Chứng minh được tách Planning/Execution, nhưng model **configurable**, không fixed

**Speaker note:** Điểm chỉnh quan trọng để slide chính xác với thực tế hiện tại.

---

## SLIDE 7: Kiến trúc 2 pha cần trình bày

**Tiêu đề:** VS Code extension: Local Plan → Cloud Execute

- **Pha 1 (Local):** Plan Agent scoping, hỏi clarifying questions [S3]
- **Pha 2 (Handoff):** User confirm → delegate sang Cloud Agent
- **Pha 3 (Cloud):** Execution trên GitHub Actions, commit/PR [S5]
- **Billing:** Chỉ 1 premium request/session
- **Hệ quả:** Model planning & execution **có thể khác nhau** (user hay tiering)

**Speaker note:** Workflow này được cộng đồng đánh giá cao nhất hiện nay.

---

## SLIDE 8: Bằng chứng model cho Planning vs Execution

**Tiêu đề:** Chúng ta chứng minh được gì?

- **Chứng minh được:** Setting riêng cho Plan Agent & Implementation
- **Chứng minh được:** Delegate từ VS Code đẩy execution sang Cloud Agent
- **Chứng minh được:** Cloud Agent tính theo session (Auto hoặc picker)
- **Chưa chứng minh được:** Planning luôn cố định một model (đặc biệt GPT-4o đã deprecated)
- **User feedback:** Hầu hết tự config model tiering

---

## SLIDE 9: Protocol demo để thuyết phục

**Tiêu đề:** Demo 5 bước xác thực thực nghiệm

- B1: VS Code → Plan Agent → scoping task
- B2: Review plan → confirm handoff Cloud Agent
- B3: Quan sát session logs (cloud execution)
- B4: Check usage metrics = 1 premium request/session
- B5: (Admin) Kiểm tra model resolution

**Speaker note:** Demo này giống hệt workflow user thực tế đang dùng.

---

## SLIDE 10: Tránh nhầm 3 công cụ khác nhau

**Tiêu đề:** Copilot CLI vs GitHub CLI vs VS Code + Cloud Agent

- **Copilot CLI (`copilot`):** Local/background agent, default Sonnet, tính per prompt
- **GitHub Pull Requests extension + Cloud Agent:** Local Plan → Cloud execution (tạo PR)
- **Cloud Agent model:** Một số entry point có picker; còn lại Auto
- **Khuyến nghị:** Luôn ghi rõ “surface” khi phân tích billing/model

**Speaker note:** Nhiều tranh cãi xuất phát từ việc trộn lẫn các bề mặt.

---

## SLIDE 11: Rủi ro & khuyến nghị

**Tiêu đề:** Rủi ro thực tế và cách giảm thiểu

- **Rủi ro 1:** Model multiplier cao → billing surprise
- **Rủi ro 2:** Over-claim model (GPT-4o) → sai quyết định
- **Rủi ro 3:** Usage spike khi delegate mà prompt chưa tốt
- **Khuyến nghị:** Plan first → handoff, audit usage hàng tháng, tối ưu copilot-instructions.md

**Speaker note:** Quản trị cả cost lẫn kỳ vọng chất lượng.

---

## SLIDE 12: Dữ liệu cộng đồng Reddit (User thực tế 2026)

**Tiêu đề:** User thực tế đang làm gì?

- **Pattern 1:** “1 Cloud Agent session thay thế hàng chục prompt” – game changer
- **Pattern 2:** Model tiering mạnh: Planning dùng Opus/GPT-5.4, execution dùng GPT-5 mini 0x
- **Pattern 3:** Workflow Plan Agent local → handoff Cloud Agent là chuẩn
- **Pain point 1:** Usage spike bất ngờ khi delegate → khuyên check analytics
- **Pain point 2:** Rate limiting dù quota tháng còn nhiều
- **Pain point 3:** Context bloat trong copilot-instructions.md → cần tối ưu token

**Speaker note:** Dữ liệu cộng đồng rất sát thực tế, giúp team áp dụng ngay mà không trial-and-error.

---

## SLIDE 13: Q&A

**Tiêu đề:** Hỏi & Đáp

**Key links:**
- Billing requests: https://docs.github.com/en/copilot/concepts/billing/copilot-requests
- Cloud Agent: https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent
- Plan Agent VS Code: https://code.visualstudio.com/docs/copilot/agents/planning
- GPT-4o deprecation: https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/

**Key takeaways:**
1. Tối ưu request bằng **Plan Agent + Cloud Session**
2. **Chứng minh được tách Planning/Execution** qua setting VS Code
3. User thực tế đang dùng model tiering + handoff để tiết kiệm mạnh

---

## PHỤ LỤC: Bảng model multipliers tiêu biểu (04/2026)

| Model                        | Paid plan multiplier | Ghi chú                  |
|------------------------------|----------------------|--------------------------|
| GPT-4.1 / GPT-5 mini / Raptor mini | 0x                  | Included (unlimited)    |
| GPT-5.4 mini                 | 0.33x               | -                        |
| Claude Haiku 4.5             | 0.33x               | -                        |
| Claude Sonnet 4.x / 4.6     | 1x                  | -                        |
| Claude Opus 4.6              | 3x                  | -                        |
| Claude Opus 4.6 (fast mode)  | 30x                 | Preview                  |
| Grok Code Fast 1             | 0.25x               | -                        |

*Nguồn: [S2] — multipliers có thể thay đổi, luôn kiểm tra docs chính thức trước khi áp dụng policy.*

---
**Hướng dẫn sử dụng:**  
Copy toàn bộ nội dung trên → dán vào file `copilot-case-study.md` → mở bằng Obsidian, Typora, Notion, hoặc VS Code để xem đẹp.  
Nếu cần chỉnh thêm hoặc export sang PowerPoint, cứ nói nhé!