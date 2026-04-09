# Case Studies: Tối Ưu Request & Kiến Trúc Copilot trong VS Code

## SLIDE 1: Billing — Tính Phí theo Request, Không theo Token

**Nguồn:** [docs.github.com/copilot-requests](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)

### Công thức

```
Chi phí = 1 premium request × model multiplier
```

### Sự Thật Quan Trọng

Tool calls trong agent mode **KHÔNG tính thêm** — agent chạy 50+ bước vẫn chỉ 1 request

### Ví dụ: Cùng 1 task — chênh lệch 10x

> **Scenario:** Refactor auth module (10 file, ~1200 dòng)
>
> | Cách làm | Model | Premium Requests |
> |----------|-------|------------------|
> | Chat 10 lần hỏi từng file | Sonnet 4.6 (1x) | **10 req** |
> | 1 prompt agent mode | Sonnet 4.6 (1x) | **1 req** |
---

## SLIDE 2: Agent Mode — Cơ Chế Hoạt Động

### Agent tự làm gì mà KHÔNG tốn thêm request?

| Bước Agent tự chạy | Ví dụ | Tính request? |
|---------------------|-------|--------------|
| 🔍 Scan codebase | Tìm files liên quan `AuthService` | ❌ Không |
| ✏️ Tạo & edit files | Refactor 10 file, apply changes | ❌ Không |
| 🖥️ Chạy terminal | `npm run build && npm test` | ❌ Không |
| 🔄 Iterate sửa lỗi | TS error → sửa → build lại → pass | ❌ Không |
| 📝 Tạo file mới | Migration script, test files | ❌ Không |
| ✅ **Prompt user** | **Lệnh khởi đầu duy nhất** | ✅ **1 request** |

> 💡 Agent Mode = thuê junior dev: giao task đầy đủ 1 lần → nó tự làm → anh review kết quả cuối.
> Mỗi lần "chen ngang" chỉnh sửa = **thêm 1 request**.

> ✅ **Nguồn:** *"For agentic features, only the prompts you send count as premium requests; actions Copilot takes autonomously to complete your task, such as tool calls, do not."* — docs.github.com

---

## SLIDE 3: Case Study #1 — CRUD Invoice Module (12 req → 1 req)

**📌 Bối cảnh:** Team 4 dev, e-commerce (Express + TypeScript + Prisma). Cần thêm module Invoice.

### ❌ Cách cũ — Chat tuần tự: 12 requests, ~45 phút

```
Prompt 1: "Tạo Invoice model"          → 1 req
Prompt 2: "Thêm zod validation"        → 1 req
Prompt 3: "Tạo controller"             → 1 req
Prompt 4: "Thêm routes"               → 1 req
Prompt 5: "Viết unit tests"           → 1 req
Prompt 6-8: Fix import, error, auth    → 3 req
Prompt 9: "Update swagger docs"        → 1 req
Prompt 10-12: Fix build, lint, types   → 3 req
───────────────────────────────────────
Tổng: 12 req × 1x = 12 premium requests
```

### ✅ Cách mới — 1 prompt Agent Mode: 1 request, ~8 phút

```markdown
## Scope
Tạo module Invoice API cho e-commerce backend.
## Yêu cầu
- Prisma Model: Invoice { id, customerId, items[], total, status (DRAFT/SENT/PAID/CANCELLED) }
- CRUD: GET (list + detail), POST, PUT, DELETE (soft delete)
- Validation: zod, customerId tồn tại, items[] không rỗng
- Error handling: 400, 404, 409, 500
- Auth: wrap routes với authMiddleware
- Tests: vitest, cover endpoints + edge cases
- Tham khảo src/modules/product/ cho conventions
## Verify
Chạy `npx prisma generate && npm run build && npm test`, tự fix nếu lỗi.
```

### 📊 Kết quả
- Tạo 8 file → fix 2 TS errors → fix 2 test assertions → **16/16 tests pass**
- **1 request, ~8 phút, 0 lần dev can thiệp**

**Speaker note:** Case study kinh điển nhất. 12x savings. Show prompt mẫu chi tiết.

---

## SLIDE 4: Case Study #2 — Bug Investigation (Workflow tốt = AI tự điều tra, 1 request)

**📌 Bối cảnh:** Bug production: "Checkout 500 lỗi ngẫu nhiên ~5% requests". Spring Boot + ReactJS + JPA/Hibernate + VNPay API. Trước Copilot: dev điều tra 4-6 giờ.

### ❌ Cách cũ — Hỏi từng bước: 8+ requests

```
Prompt 1: "Tìm file xử lý checkout"                    → 1 req
Prompt 2: "Xem logic gọi VNPay API"                    → 1 req
Prompt 3: "Check error handling ở đây"                   → 1 req
Prompt 4: "Tìm race condition giữa inventory và payment" → 1 req
Prompt 5: "Tại sao chỉ lỗi 5%?"                         → 1 req
Prompt 6: "Sửa lỗi race condition"                       → 1 req
Prompt 7: "Viết test cho case này"                       → 1 req
Prompt 8: "Fix test fail"                                → 1 req
───────────────────────────────────────────────────────────
Tổng: 8 req — dev vẫn phải "lái" từng bước
```

> ⚠️ **Vấn đề:** Mỗi lần hỏi = 1 premium request. Dev trở thành "người dẫn đường" cho AI, tốn request y như tự debug nhưng chậm hơn.

### ✅ Cách mới — 1 prompt Agent Mode + Workflow tự động: 1 request

```markdown
## Bug Report
Checkout endpoint POST /api/checkout trả 500 error ngẫu nhiên ~5% requests.
Lỗi không reproduce được khi test manual. Chỉ xảy ra dưới concurrent load.

## Workflow điều tra (agent tự chạy theo thứ tự)
### Bước 1: Thu thập context
- Tìm tất cả files liên quan checkout: Controller, Service, Payment, Inventory
- Đọc và hiểu flow: request → validate → reserve inventory → charge VNPay → confirm order

### Bước 2: Phân tích nguyên nhân
- Tìm race condition, missing locks, hoặc thiếu @Transactional
- Kiểm tra error handling: có catch đúng VNPay errors không?
- Check thứ tự operations: inventory reserve vs payment charge — ai trước?
- Nếu inventory reserve TRƯỚC payment → rollback khi VNPay fail?
- Nếu payment TRƯỚC inventory → oversell khi concurrent?

### Bước 3: Fix
- Wrap checkout flow trong @Transactional (Spring JPA transaction)
- Thêm idempotency key cho VNPay payment (tránh double charge)
- Thêm proper error handling + rollback logic
- Đảm bảo atomic: hoặc cả hai thành công, hoặc cả hai rollback

### Bước 4: Verify
- Viết unit test: happy path + concurrent checkout + VNPay failure + inventory insufficient
- Chạy `./mvnw test` (backend), tự fix nếu lỗi
- Chạy `npm test` (frontend) để verify React components không lỗi
```

### 📊 Kết quả
- Agent tự scan 12 files → phát hiện race condition (inventory reserve không có @Transactional) → fix 3 files (Service, Controller, PaymentGateway) → thêm 8 test cases (@SpringBootTest + MockMvc) → **ALL PASS**
- **1 premium request, ~6 phút, 0 lần dev can thiệp**

### 💡 Bài học: Workflow là chìa khóa

```
┌─────────────────────────────────────────────────────────┐
│  Muốn tiết kiệm request = Thiết kế WORKFLOW cho agent  │
│                                                         │
│  ❌ SAI: Hỏi từng bước → AI chờ → bạn đọc → hỏi tiếp  │
│     (8 req, bạn làm "GPS" cho AI)                       │
│                                                         │
│  ✅ ĐÚNG: Cho agent CẢ quy trình điều tra              │
│     Bước 1 → 2 → 3 → 4 tự chạy liên tục               │
│     (1 req, AI tự "lái" theo workflow)                  │
│                                                         │
│  🔑 Mấu chốt: Bạn không cần biết đáp án,              │
│     chỉ cần biết CÁCH ĐIỀU TRA (workflow)               │
└─────────────────────────────────────────────────────────┘
```

> 💡 **Nguyên tắc:** Đừng cho AI "cá" (đáp án từng bước) — hãy cho AI "cần câu" (workflow điều tra). Agent tự scan code, tự suy luận, tự fix — tất cả trong 1 request.

---


## SLIDE 5: Model Mismatch — Vấn Đề Phát Hiện

> 🔬 **Nguồn:** Source code `vscode-copilot-chat`, GitHub Issues, Chat Debug logs

### ⚠️ Tình huống thực tế

- User chọn: **GPT-5.3-Codex**
- Chat Debug hiện:
  ```
  model: gpt-4o-mini
  completion_tokens: 8
  prompt_tokens: 1628
  duration: 1911ms
  ```
- ❓ Copilot dùng `gpt-4o-mini` hay `GPT-5.3-Codex`?

### 🔍 Nguyên nhân: Two-Stage Pipeline

```
USER PROMPT
    ↓
┌────────────────────────────┐
│ STAGE 1: Intent Detection  │
│ Model: gpt-4o-mini         │
│ (hardcoded, "copilot-fast")│
│ Output: ~8 tokens          │
│ Billing: KHÔNG premium     │
└──────────┬─────────────────┘
           ↓
┌────────────────────────────┐
│ STAGE 2: Main Execution    │
│ Model: GPT-5.3-Codex       │
│ (model user đã chọn)       │
│ Output: nhiều tokens       │
│ Billing: CÓ premium × mul │
└────────────────────────────┘
```

> **Kết luận:** Cái thấy trong debug (gpt-4o-mini, 8 tokens) = **Stage 1 Intent Detection**, KHÔNG phải main task.

**Speaker note:** Đây là phát hiện quan trọng — giải thích tại sao debug logs "khác" với model đã chọn.

---

## SLIDE 18: Model Mismatch — Bằng Chứng & Auto-Fallback

### Bằng chứng từ Source Code

| Nguồn | Phát hiện |
|-------|-----------|
| `intentDetector.tsx` (line 217) | `getChatEndpoint('copilot-fast')` — endpoint riêng cho intent |
| `GPT4OIntentDetectionPrompt` class | GPT-4O family dùng cho intent detection |
| `CopilotLanguageModelWrapper` | Central proxy cho TẤT CẢ model requests |
| docs.github.com/auto-model-selection | Auto-fallback khi model không available |
| GitHub Issue #14165 | GPT-5.3-Codex không accessible trên một số env |

### Auto-Fallback — Model chọn ≠ Model chạy

Copilot tự chuyển model dựa trên:
- Real-time system health metrics
- Model availability & capacity
- Rate limiting status
- Regional/subscription constraints

> ⚠️ `resolved model` trong debug = model **THỰC SỰ** được dùng (có thể khác model đã chọn)

### Bảng Fact vs Hypothesis

| Luận điểm | Trạng thái |
|-----------|-----------|
| VS Code dùng 2-stage pipeline | ✅ Sự thật (source code) |
| Intent detection dùng `copilot-fast` endpoint | ✅ Sự thật (source code) |
| Copilot có auto-fallback | ✅ Sự thật (docs) |
| GPT-5.3-Codex có issues availability | ✅ Sự thật (Issues #14165) |
| gpt-4o-mini là router model chính thức | ⚠️ Giả thuyết (không document công khai) |
| Model user chọn luôn được dùng | ❌ Không chắc (auto-fallback) |

**Speaker note:** Slide bằng chứng kỹ thuật. Nhấn mạnh: source code + GitHub Issues xác nhận.

---

## SLIDE 19: So Sánh Local Copilot vs Copilot CLI trong VS Code

> ⚠️ Hai surface này KHÁC NHAU mặc dù đều chạy trong VS Code

| Tiêu chí | Chat/Agent (Local) | Copilot CLI (Terminal) |
|---------|-------------------|----------------------|
| **Nơi chạy** | IDE panel | Integrated terminal |
| **Agent type** | Local (built-in) | Background agent |
| **Billing** | 1 req × multiplier / prompt | 1 req × multiplier / prompt |
| **Tool calls** | Không tính thêm | Không tính thêm |
| **Intent detection** | ⚠️ Có (ẩn) | ⚠️ Có (ẩn) |
| **Model mismatch risk** | 🟡 Ít gặp | 🔴 Cao hơn (Codex issues) |

### Rủi ro "Model Phantom"

| Rủi ro | Mức độ | Mô tả |
|--------|--------|-------|
| Model Phantom | 🔴 Cao | Chọn model A → thực tế dùng model B (intent detection + auto-fallback) |
| Debug Confusion | 🟡 TB | Chat Debug trộn intent detection với main execution |
| Codex Availability | 🟡 TB | GPT-5.3-Codex không available trên mọi môi trường |

**Speaker note:** Clarify sự khác biệt 2 surfaces. Codex CLI có rủi ro cao hơn.

---

## SLIDE 20: Kiến Trúc Tổng Quan — 3 Surfaces + Hidden Layer

```
┌────────────────────────────────────────────────────────────────────┐
│                         VS CODE IDE                                │
│                                                                    │
│  ┌──────────────┐  ┌───────────────────┐  ┌──────────────────┐   │
│  │ 1. INLINE    │  │ 2. CHAT / AGENT   │  │ 3. COPILOT CLI  │   │
│  │ COMPLETIONS  │  │    MODE (LOCAL)    │  │   (BACKGROUND)  │   │
│  │              │  │                    │  │                  │   │
│  │ Ghost text   │  │ Chat + Agent edit  │  │ Terminal agent   │   │
│  │ Quota riêng  │  │ 1 req/prompt      │  │ 1 req/prompt    │   │
│  │ Ko premium   │  │ Tool calls FREE   │  │ Tool calls FREE │   │
│  └──────────────┘  └───────────────────┘  └──────────────────┘   │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │ ẨN: Intent Detection (gpt-4o-mini via "copilot-fast")       │ │
│  │ → Chạy TRƯỚC mọi surface → Classify: ask/write/edit/help    │ │
│  │ → KHÔNG document chính thức                                  │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

| Surface | Best For | Billing |
|---------|----------|---------|
| Inline Completions | Gõ code real-time | Quota riêng (free trên Paid) |
| Chat Mode | Hỏi đáp, explain, review | 1 req × multiplier / message |
| Agent Mode | Multi-file, phức tạp | 1 req × multiplier / prompt |

**Speaker note:** Bức tranh toàn cảnh — 3 surfaces + 1 hidden layer. Print cho team.

---

## SLIDE 21: Demo — Cách Verify Model Thực Tế

### 5 Bước Verify

**B1.** VS Code → Command Palette → `Developer: Open Chat Debug`

**B2.** Gửi 1 prompt đơn giản, chọn model cụ thể (ví dụ: GPT-5.3-Codex)

**B3.** Kiểm tra Chat Debug — tìm **tất cả requests** trong 1 interaction:
- Request nhỏ (~8 completion tokens) = **Intent Detection** (gpt-4o-mini)
- Request lớn (nhiều completion tokens) = **Main Task** (model đã chọn)

**B4.** So sánh `resolved model` vs model đã chọn:
- Giống → ✅ OK
- Khác → ⚠️ Auto-fallback đã xảy ra

**B5.** Verify billing: Intent detection = KHÔNG premium. Main task = CÓ premium × multiplier.

**Speaker note:** Demo 3 phút. Rất hữu ích cho team kỹ thuật — chứng minh bằng data.

---

## SLIDE 22: Rủi Ro & Khuyến Nghị Tổng Hợp

### ⚠️ Rủi ro

| # | Rủi ro | Mức độ | Mô tả |
|---|--------|--------|-------|
| R1 | Billing surprise | 🔴 Cao | Opus 3x, GPT-4.5 ~50x. Không set budget → overage |
| R2 | Model Phantom | 🔴 Cao | Chọn model A → Copilot dùng model B (fallback) |
| R3 | Nhầm Surface | 🟡 TB | Chat vs Agent vs CLI → billing khác nhau |
| R4 | Rate limit sớm | 🟡 TB | Burst window limit dù quota tháng còn |
| R5 | Codex Availability | 🟡 TB | GPT-5.3-Codex không available mọi env |

### ✅ Khuyến nghị

| # | Khuyến nghị | Hành động |
|---|------------|-----------|
| K1 | Chuẩn hóa workflow | Template S.C.D.T + Agent Mode |
| K2 | Verify model | Chat Debug → check `resolved model` |
| K3 | Monthly audit | Usage analytics + budget alerts |
| K4 | Surface docs | Tách rõ 3 surfaces trong team docs |
| K5 | Fallback training | Team hiểu auto-fallback + cách verify |

**Speaker note:** Quản trị = cost + model quality + kỳ vọng output. 5 khuyến nghị áp dụng ngay.

---

## SLIDE 23: Community Data & Nghiên Cứu Bổ Sung

### 📈 Data chính thức (Accenture, 2024)

| Metric | Kết quả | Ý nghĩa |
|--------|---------|---------|
| PR throughput | **+8.69%** / dev | Nhiều feature deliver hơn |
| Build success | **+84%** | Ít break CI/CD |
| Adoption | **96% dùng ngay ngày đầu** | Onboarding cực nhanh |
| Usage | **67% dùng ≥5 ngày/tuần** | Tool thiết yếu |
| Code retention | **88% giữ generated code** | Output chất lượng |

### Patterns hiệu quả (cộng đồng)
- **Model Tiering:** Model mạnh cho spec, model rẻ cho implement
- **Chat Debug Audit:** Verify model thực tế — phát hiện intent detection
- **Context File Investment:** 30 phút viết instructions → tiết kiệm hàng giờ/tuần

### Pain points phổ biến
- **Model Mismatch:** gpt-4o-mini trong debug → confusion
- **Rate Limiting sớm:** Burst window ≠ monthly quota
- **Prompt Quality Gap:** Chưa quen agentic prompt → tốn 5-10x request

**Speaker note:** Data Accenture cho executive buy-in. Pain points giúp team tránh bẫy.

---

## SLIDE 24: Q&A + Key Takeaways

### 5 Key Takeaways

1. ✅ **Agent Mode = tiết kiệm 10-12x** — 1 prompt thay 12 prompt chat nhỏ
2. ✅ **Included models (0x) = vũ khí bí mật** — 70% tasks miễn phí hoàn toàn
3. ✅ **Model picker ≠ Model thực tế** — Intent detection + auto-fallback ẩn
4. ✅ **Template S.C.D.T + 3 Quy Tắc Vàng** — Framework áp dụng ngay
5. ✅ **Data chứng minh ROI** — 55% nhanh hơn, +84% builds, 90% hài lòng

### Nguồn tham khảo

| Chủ đề | Link |
|--------|------|
| Billing / Requests | docs.github.com/copilot/concepts/billing/copilot-requests |
| Supported Models | docs.github.com/copilot/reference/ai-models/supported-models |
| Auto Model Selection | docs.github.com/copilot/concepts/auto-model-selection |
| VS Code Chat Modes | code.visualstudio.com/docs/copilot/chat/chat-modes |
| Accenture Research | github.blog — research-quantifying-copilots-impact |
| Agent Mode Blog | github.blog — copilot-the-agent-awakens |
| Codex Issue #14165 | github.com/microsoft/vscode-copilot-release/issues/14165 |

**Speaker note:** Kết thúc mạnh với 5 takeaways. Mỗi takeaway map tới 1 nhóm slides.

---

## Phụ Lục A: Bảng Model Multipliers Đầy Đủ

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

---

## Phụ Lục B: Two-Stage Pipeline — Minh Họa Kỹ Thuật

```
User chọn GPT-5.3-Codex trong Model Picker
           ↓
┌──────────────────────────────────┐
│  STAGE 1: Intent Detection       │
│  Model: gpt-4o-mini (hardcoded) │
│  Endpoint: copilot-fast          │
│  ~8 completion tokens            │
│  Billing: KHÔNG tính premium     │
└──────────────────────────────────┘
           ↓
┌──────────────────────────────────┐
│  STAGE 2: Main Task Execution    │
│  Model: GPT-5.3-Codex (hoặc     │
│         auto-fallback model)     │
│  Proxy: copilotLanguageModel     │
│         Wrapper                  │
│  Nhiều completion tokens         │
│  Billing: CÓ tính premium × mul │
└──────────────────────────────────┘
           ↓
     Response → User
```

**Cách phân biệt trong Chat Debug:**
- Request có ~8 completion tokens, ~1600 prompt tokens → **Intent Detection** (Stage 1)
- Request có nhiều completion tokens → **Main Task** (Stage 2)
- Check field `resolved model` trong Stage 2 để verify model thực tế

---

## Phụ Lục C: Bảng Bằng Chứng Tổng Hợp — Fact vs Hypothesis

| Luận điểm | Trạng thái | Nguồn |
|-----------|-----------|-------|
| Agent Mode: tool calls không tính premium | ✅ Sự thật | docs.github.com/copilot-requests |
| Inline completions có quota riêng | ✅ Sự thật | docs.github.com/copilot-requests |
| Auto model selection discount 10% | ✅ Sự thật | docs.github.com/auto-model-selection |
| VS Code dùng `copilot-fast` cho intent detection | ✅ Sự thật | Source code intentDetector.tsx |
| gpt-4o-mini dùng cho intent detection | ⚠️ Giả thuyết mạnh | Source code + Chat Debug evidence |
| Copilot có auto-fallback | ✅ Sự thật | docs.github.com/auto-model-selection |
| GPT-5.3-Codex có issues availability | ✅ Sự thật | GitHub Issues #14165, #14167 |
| GPT-4o deprecated trong Copilot Chat | ✅ Sự thật | changelog 2025-08-06 |
| Model user chọn luôn là model chạy | ❌ Không chắc | Auto-fallback có thể thay thế |

*Phát hiện dựa trên: source code analysis + Chat Debug observation + community reports*
