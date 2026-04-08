# Case Studies: Tối Ưu Request & Tách Planning/Execution trong GitHub Copilot

> **Cập nhật:** 04/2026 | **Nguồn:** docs.github.com + code.visualstudio.com (chính thức)
> 
> 🏷️ Mọi claim được gắn nhãn rõ: ✅ **Sự thật** | ⚠️ **Giả thuyết** | ❌ **Lỗi thời**

---

## Tóm Tắt cho Quản Lý (Executive Summary)

### 💡 Core Insight #1 — Tối ưu Request
Copilot tính phí theo **request × model multiplier**, không theo tổng token.
Chiến lược tối ưu: gửi **1 prompt chất lượng cao** → agent tự chạy nhiều bước mà **không tốn thêm request**.

### 🏗️ Core Insight #2 — Kiến trúc 2 pha
VS Code + GitHub Pull Requests extension = kiến trúc **2 pha**:
1. **Scoping/planning** cục bộ trong IDE
2. **Execution** trên GitHub Actions bởi cloud agent

### ⚠️ Điểm chỉnh quan trọng
Hiện **không có bằng chứng chính thức** rằng pha planning dùng GPT-4o cố định.
GPT-4o đã **deprecated trong Copilot Chat từ 08/2025**.
Claim đúng: tách pha planning/execution — **KHÔNG phải** "GPT-4o làm planning".

---

## SLIDE 2: Billing theo Request, Không theo Token

**Nguồn:** [docs.github.com/copilot/concepts/billing/copilot-requests](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)

### Công thức cốt lõi
1 prompt gửi đi = 1 request × model multiplier
- ✅ **Sự thật:** Tool calls tự động trong cùng phiên agentic **KHÔNG bị tính thêm**
- ✅ **Sự thật:** Copilot cloud agent = **1 premium request/session** dù agent chạy hàng trăm bước
- ✅ **Sự thật:** Included models (GPT-4.1, GPT-4o, GPT-5 mini) có multiplier **0x** trên paid plans

### Bảng Model Multipliers (Paid Plans)

| Model | Multiplier | Ghi chú |
|-------|-----------|---------|
| GPT-4.1 | **0x** | Included — không tốn premium request |
| GPT-4o | **0x** | Included (deprecated Copilot Chat 08/2025) |
| GPT-5 mini | **0x** | Included |
| Raptor mini | **0x** | Included |
| Grok Code Fast 1 | 0.25x | Rẻ nhất trong premium |
| Claude Haiku 4.5 | 0.33x | Budget-friendly |
| Claude Sonnet 4.5/4.6 | 1x | Standard premium |
| Gemini 3 Pro | 1x | Standard premium |
| Claude Opus 4.5 | **3x** | ⚠️ Cân nhắc kỹ |
| GPT-4.5 | **~50x** | ❌ Tránh dùng thường xuyên |

> ⚠️ **Lưu ý:** Bảng thay đổi thường xuyên. Luôn kiểm tra
> [docs.github.com/en/copilot/reference/ai-models/supported-models](https://docs.github.com/en/copilot/reference/ai-models/supported-models)
> trước khi chốt policy nội bộ.

**Speaker note:** Core insight — tối ưu chi phí nằm ở việc giảm **số lượng prompt** và chọn đúng model, không phải giảm token output.

---

## SLIDE 3: Case Study 1 — Ít Prompt Hơn, Nhiều Việc Hơn

**Mục tiêu:** Tận dụng cách tính request để "nén" nhiều task vào 1 phiên agentic

### Tại sao agent mode tiết kiệm?

| Tính năng | Mô tả | Tính request? |
|-----------|-------|--------------|
| Agent tự xác định file | Scan codebase, tìm file liên quan | ❌ Không |
| Agent tự đề xuất edit | Tạo diff, apply changes | ❌ Không |
| Agent chạy terminal | Build, test, lint | ❌ Không |
| Agent iterate sửa lỗi | Loop tự fix sau mỗi error | ❌ Không |
| **Prompt của user** | Lệnh khởi đầu | ✅ **Có — 1 request** |

### So sánh chi phí 4 workflows

| Workflow | Model | Premium Requests | Output |
|----------|-------|-----------------|--------|
| A: 10 prompts nhỏ | Sonnet 4.5 (1x) | **10 req** | 10 đoạn rời rạc |
| B: 1 prompt agent mode | Sonnet 4.5 (1x) | **1 req** | Full task end-to-end |
| C: 1 prompt | GPT-4.1 (0x) | **0 req** | Full task, included model |
| D: Cloud agent session | Auto (cloud) | **1 req/session** | PR tự động trên GitHub |

> 💡 **Kết luận:** Tối ưu request đến từ **kiến trúc workflow**, không chỉ từ cách viết prompt.

**Speaker note:** Đòn bẩy chính = gộp task thành 1 phiên agentic đủ rõ thay vì chat nhỏ lẻ nhiều lượt.

---

## SLIDE 4: Playbook Triển Khai cho Team

### ✦ Prompt Engineering
- Chuẩn hóa template prompt: `scope` + `constraints` + `done criteria` + `test criteria`
- Gộp task đa bước vào 1 agentic session thay vì chia nhỏ
- Dùng context files, `.github/copilot-instructions.md` để giảm lượt làm rõ

### ✦ Model Selection
- **Task nhẹ:** GPT-4.1 / GPT-5 mini → 0x (miễn phí)
- **Task reasoning sâu:** Sonnet 4.5/4.6 → 1x
- **Auto model selection** → discount thêm ~10% multiplier trên paid plans

### ✦ Governance
- Audit usage theo user/team hàng tháng
- Set budget cap cho premium request overage
- Cảnh báo khi tiêu thụ cao bất thường (Opus, GPT-4.5)

### ✦ Model Tiering (Community Pattern)
| Giai đoạn | Model khuyến nghị | Lý do |
|-----------|-------------------|-------|
| Spec / Planning | Sonnet 4.5 (1x) | Reasoning tốt |
| Implementation | GPT-4.1 (0x) | Included, free |
| Review / Test | Grok Code Fast (0.25x) | Tiết kiệm |

**Speaker note:** Tối ưu bền vững cần cả kỹ thuật prompt lẫn policy vận hành team.

---

## SLIDE 5: Case Study 2 — "GPT-4o Chỉ Phân Task?" → Sửa Giả Thuyết

### ⚠️ Giả thuyết ban đầu (cần kiểm chứng)
> *"Khi dùng GitHub Pull Requests extension trong VS Code, GPT-4o chỉ dùng để phân task, không chạy task thực"*

### ✅ Chứng minh được (theo docs chính thức)

- **Kiến trúc 2 pha có thật:** local scoping → cloud execution là kiến trúc thực của GitHub Copilot coding agent
  - Nguồn: [changelog 2025-07-14](https://github.blog/changelog/2025-07-14-start-and-track-github-copilot-coding-agent-sessions-from-visual-studio-code/)
- **VS Code có settings riêng** cho từng pha:
  - `chat.planAgent.defaultModel` — model cho pha planning
  - `implementAgent.model` — model cho pha implement
  - Nguồn: [code.visualstudio.com/docs/copilot/agents/planning](https://code.visualstudio.com/docs/copilot/agents/planning)
- **Execution chạy trên GitHub Actions**, không phải local terminal
  - Nguồn: [docs.github.com/…/create-a-pr](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-a-pr)

### ❌ Chưa chứng minh được / Lỗi thời

- **"Planning luôn dùng GPT-4o"** — không có tài liệu chính thức nào khẳng định điều này
- **GPT-4o đã deprecated trong Copilot Chat từ 08/2025** — dùng làm luận điểm trung tâm sẽ lỗi thời
  - Nguồn: [changelog 2025-08-06](https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/)
- **Nếu entry point không có model picker → dùng Auto**, không cố định GPT-4o
  - Nguồn: [docs.github.com/…/changing-the-ai-model](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/changing-the-ai-model)

> 💡 **Kết luận đúng:** Claim chính xác là "VS Code extension tách planning (local) và execution (cloud)".
> **KHÔNG phải** "GPT-4o làm planning cố định".

---

## SLIDE 6: Kiến Trúc 2 Pha — VS Code → Cloud
┌─────────────────────┐ user ┌──────────────────────┐ async ┌────────────────────────┐
│ PHA 1 · LOCAL IDE │ ──confirm──> │ PHA 2 · HANDOFF │ ──run──────> │ PHA 3 · CLOUD EXEC │
│ │ │ │ │ │
│ - Copilot Chat │ │ - User bấm Continue │ │ - GitHub Actions env │
│ - Scoping/planning │ │ hoặc Delegate │ │ - Agent commit branch │
│ - Hỏi lại yêu cầu │ │ - Task → cloud agent │ │ - Mở/update PR │
│ │ │ │ │ - Run tests, builds │
│ Model: configured │ │ Model: Auto │ │ - Log trong session │
│ hoặc Auto │ │ hoặc picker │ │ │
│ Billing: chat req │ │ │ │ Billing: 1 req/session │
└─────────────────────┘ └──────────────────────┘ └────────────────────────┘

### Hệ quả quan trọng
- Model chat local (pha 1) và model execution cloud (pha 3) **có thể khác nhau**
- Không nên giả định cùng model cho cả 2 pha
- Billing cũng **khác nhau**: chat request (pha 1) vs session request (pha 3)

**Speaker note:** Đây là phần "chứng minh" quan trọng nhất cho audience kỹ thuật và quản lý.

---

## SLIDE 7: Bảng Bằng Chứng — Fact vs Hypothesis

| Luận điểm | Trạng thái | Nguồn |
|-----------|-----------|-------|
| VS Code có settings riêng cho plan agent và implement step | ✅ Sự thật | VS Code docs · agents/planning |
| Cloud agent entry points không có model picker → dùng Auto | ✅ Sự thật | docs.github.com/changing-the-ai-model |
| Delegate từ VS Code đẩy execution sang cloud agent (không local) | ✅ Sự thật | changelog 2025-07-14 · create-a-pr |
| GPT-4o đã deprecated trong Copilot Chat từ 08/2025 | ✅ Sự thật | changelog 2025-08-06 |
| "Planning LUÔN dùng GPT-4o" cho mọi org/thời điểm | ⚠️ Giả thuyết | Không có nguồn chính thức nào xác nhận |
| GPT-4o còn là model trung tâm Copilot Chat hiện tại | ❌ Lỗi thời | Đã deprecated — thay bằng GPT-4.1, Auto |

**Speaker note:** Slide này giúp phân biệt fact-based evidence và suy diễn — quan trọng cho uy tín của presenter.

---

## SLIDE 8: Demo 5 Bước Xác Thực Thực Nghiệm

> 🎯 **Mục tiêu:** Thuyết phục stakeholder bằng bằng chứng thực nghiệm, không bằng tranh luận lý thuyết

**B1 · KHỞI TẠO**
- Trong VS Code, dùng GitHub Pull Requests extension
- Khởi tạo task từ chat → delegate sang cloud agent

**B2 · QUAN SÁT HANDOFF**
- Quan sát prompt phản hồi xác nhận handoff
- Link PR/session xuất hiện — task không còn chạy ở local terminal

**B3 · VERIFY CLOUD EXECUTION**
- Mở session logs trong VS Code panel
- Thấy execution diễn ra trên cloud agent / GitHub Actions, không phải local machine

**B4 · MAP BILLING**
- Đối chiếu usage metrics / premium counters theo session
- Xác nhận: 1 premium request/session cho cloud agent

**B5 · MODEL AUDIT** *(nếu có quyền admin)*
- Đối chiếu model usage report trong GitHub organization settings
- Kiểm tra model nào thực sự được chọn (Auto resolution là gì)

**Speaker note:** Demo có thể lặp lại — dễ thuyết phục stakeholder hơn tranh luận cảm tính về "văn phong model".

---

## SLIDE 9: Tránh Nhầm 3 Công Cụ Khác Nhau

> ⚠️ Nhiều tranh cãi về billing/model xuất phát từ việc **trộn lẫn các "surface" khác nhau** vào cùng một nhóm

| Tiêu chí | Copilot CLI (`gh copilot`) | GitHub CLI Agent (`gh agent-task`) | VS Code + PR Ext |
|---------|--------------------------|-----------------------------------|-----------------|
| **Môi trường** | Terminal / Shell | GitHub CLI | IDE + GitHub |
| **Default model** | Sonnet 4.5 | Auto | Configured / Auto |
| **Billing** | Theo mỗi prompt | 1 req/session | Chat req + 1 req/session |
| **Context** | Local shell | GitHub repo | IDE + GitHub |
| **Có model picker?** | Có | Không | Có (pha 1) |
| **Pain point** | Global skills → context bloat | — | Nhầm local vs cloud billing |

> 💡 **Khuyến nghị:** Khi phân tích billing hay model, luôn ghi rõ **"surface" đang nói tới**.

---

## SLIDE 10: Rủi Ro Thực Tế & Khuyến Nghị

### ⚠️ Rủi ro

| # | Rủi ro | Mức độ | Mô tả |
|---|--------|--------|-------|
| R1 | Billing surprise | 🔴 Cao | Model multiplier cao bất ngờ (Opus 3x, GPT-4.5 ~50x). Không set budget cap → overage không giới hạn |
| R2 | Over-claim về model | 🟡 Trung bình | Gắn cứng "GPT-4o là planning model" làm sai quyết định kỹ thuật — GPT-4o deprecated 08/2025 |
| R3 | Nhầm Local vs Cloud | 🟡 Trung bình | Nhầm local chat với cloud execution → đo hiệu quả và billing sai |
| R4 | Rate limiting sớm | 🟡 Trung bình | Short-window rate limit kích hoạt dù quota tháng còn nhiều |

### ✅ Khuyến nghị

| # | Khuyến nghị | Hành động cụ thể |
|---|------------|-----------------|
| K1 | Agentic Workflow Standard | Chuẩn hóa template prompt + ưu tiên agent mode |
| K2 | Model Policy Documentation | Tách rõ policy model cho planning và execution trong team docs |
| K3 | Monthly Usage Audit | Audit usage + session logs hàng tháng, set budget alerts |

**Speaker note:** Mục tiêu cuối cùng là quản trị được cả cost lẫn kỳ vọng chất lượng model.

---

## SLIDE 11: Community Data — User Thực Tế Dùng Copilot Như Thế Nào?

> 📊 Nguồn: r/GithubCopilot, r/opencodeCLI (2025–2026)
> ⚠️ Dữ liệu cộng đồng — chỉ dùng để tạo giả thuyết vận hành, không thay thế tài liệu chính thức

### ✓ Patterns hiệu quả
- **Model Tiering:** Dùng model mạnh (1x) cho spec/reasoning, model rẻ (0x/0.25x) cho triển khai
- **Usage Pacing:** Theo dõi nhịp tiêu hao premium requests theo tuần/tháng để tránh overage cuối kỳ

### ⚠️ Pain points phổ biến
- **Nhầm Session vs Prompt Billing:** Nhiều user nhầm billing theo session (cloud agent) với billing theo từng prompt (chat mode)
- **Rate Limiting sớm:** Gặp rate limit dù quota tháng còn nhiều — do giới hạn burst window, không phải hết tháng
- **CLI Context Bloat:** Copilot CLI + nhiều global skills/plugins → context bloat → token limit errors

---

## SLIDE 12: Q&A + Key Takeaways

### 4 Key Takeaways

1. ✅ **Tối ưu request = workflow agentic đúng cách**, không phải giảm token
2. ✅ **Có thể chứng minh tách planning/execution** theo docs chính thức — KHÔNG thể chốt "GPT-4o làm planning cố định"
3. ✅ **3 tool surfaces** (CLI, gh CLI, VS Code ext) có billing và model behavior **khác nhau hoàn toàn**
4. ✅ **Luôn gắn nhãn Sự thật vs Giả thuyết** khi trình bày — uy tín quan trọng hơn sự chắc chắn giả

### Nguồn tham khảo chính

| Chủ đề | Link |
|--------|------|
| Billing / Requests | https://docs.github.com/en/copilot/concepts/billing/copilot-requests |
| Supported Models + Multipliers | https://docs.github.com/en/copilot/reference/ai-models/supported-models |
| Cloud Agent Model Selection | https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/changing-the-ai-model |
| Delegate từ VS Code | https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-a-pr |
| VS Code Planning Docs | https://code.visualstudio.com/docs/copilot/agents/planning |
| GPT-4o Deprecation | https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/ |
| 1 req/session Changelog | https://github.blog/changelog/2025-07-10-github-copilot-coding-agent-now-uses-one-premium-request-per-session/ |

---

## Phụ Lục: Bảng Model Multipliers Đầy Đủ

| Model | Paid Plan | Free Plan | Nhóm |
|-------|-----------|-----------|------|
| GPT-4.1 | 0x | 1x | Included |
| GPT-4o | 0x | 1x | Included (deprecated Chat) |
| GPT-5 mini | 0x | 1x | Included |
| Raptor mini | 0x | 1x | Included |
| Grok Code Fast 1 | 0.25x | N/A | Budget premium |
| Claude Haiku 4.5 | 0.33x | 1x | Budget premium |
| GPT-5.1-Codex-Mini | 0.33x | N/A | Budget premium |
| o3 mini / o4 mini | 0.33x | N/A | Budget premium |
| Gemini 3 Flash | 0.33x | N/A | Budget premium |
| Claude Sonnet 4.5/4.6 | 1x | N/A | Standard premium |
| GPT-5.1 / 5.2 / 5.3 / 5.4 | 1x | N/A | Standard premium |
| Gemini 3 Pro | 1x | N/A | Standard premium |
| o3 | 1x | N/A | Standard premium |
| Claude Opus 4.5 | **3x** | N/A | ⚠️ High cost |
| GPT-4.5 | **~50x** | N/A | ❌ Rất đắt |

*Nguồn: docs.github.com/en/copilot/reference/ai-models/supported-models — cập nhật 04/2026*