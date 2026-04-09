# Case Studies: Tối Ưu Request & Kiến Trúc Copilot trong VS Code

> **Cập nhật:** 04/2026 | **Nguồn:** docs.github.com + code.visualstudio.com (chính thức) + source code analysis
> 
> 🏷️ Mọi claim được gắn nhãn rõ: ✅ **Sự thật** | ⚠️ **Giả thuyết** | ❌ **Lỗi thời**

---

## SLIDE 1: Tóm Tắt cho Quản Lý (Executive Summary)

### 💡 3 Core Insights

| # | Insight | Ý nghĩa |
|---|---------|---------|
| 1 | **Tối ưu Request = Tối ưu Workflow** | 1 prompt Agent Mode thay thế 10-12 prompt chat nhỏ → tiết kiệm 10x request |
| 2 | **Included models (0x) = Miễn phí** | GPT-4.1, GPT-5 mini → 0 premium request trên paid plans. 70% tasks đủ dùng |
| 3 | **VS Code có Hidden Pipeline** | Intent Detection (gpt-4o-mini) chạy ẩn trước mọi request + Auto-fallback |

### 📈 Dữ liệu nghiên cứu (GitHub × Accenture, 450+ devs)

| Metric | Kết quả |
|--------|---------|
| Code speed | **55% nhanh hơn** |
| Code confidence | **85% tự tin hơn** |
| PR merge rate | **+15%** (code quality tốt hơn) |
| Successful builds | **+84%** |
| Job satisfaction | **90% hài lòng hơn** |

> *Nguồn: [github.blog — Accenture Research](https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-in-the-enterprise-with-accenture/)*

**Speaker note:** Slide overview 1 phút. 3 insights + data chứng minh ROI. Copilot không phải cost — là investment.

---

## SLIDE 2: Billing — Tính Phí theo Request, Không theo Token

**Nguồn:** [docs.github.com/copilot-requests](https://docs.github.com/en/copilot/concepts/billing/copilot-requests)

### Công thức

```
Chi phí = 1 premium request × model multiplier
```

### 5 Sự Thật Quan Trọng

1. ✅ Tool calls trong agent mode **KHÔNG tính thêm** — agent chạy 50+ bước vẫn chỉ 1 request
2. ✅ Included models (GPT-4.1, GPT-5 mini, GPT-4o) = **0x** trên paid plans → dùng không giới hạn
3. ✅ Auto model selection = **discount ~10%** multiplier (paid plans)
4. ✅ Inline completions (ghost text) có **quota riêng** — KHÔNG tính premium request
5. ✅ Khi hết premium request → vẫn dùng được included models (0x)

### Ví dụ: Cùng 1 task — chênh lệch 10x

> **Scenario:** Refactor auth module (5 file, ~800 dòng)
>
> | Cách làm | Model | Premium Requests |
> |----------|-------|------------------|
> | Chat 10 lần hỏi từng file | Sonnet 4.6 (1x) | **10 req** |
> | 1 prompt agent mode | Sonnet 4.6 (1x) | **1 req** |
> | 1 prompt agent mode | GPT-4.1 (0x) | **0 req** |

**Speaker note:** Tối ưu chi phí = giảm số prompt + chọn đúng model + tận dụng agent mode.

---

## SLIDE 3: Bảng Model Multipliers — Chọn Model Nào?

> ⚠️ Bảng thay đổi thường xuyên → luôn check [docs.github.com/supported-models](https://docs.github.com/en/copilot/reference/ai-models/supported-models)

| Nhóm | Model | Multiplier | Khi nào dùng | Ví dụ |
|------|-------|-----------|--------------|-------|
| 🟢 **FREE** | GPT-4.1 | **0x** | Task hàng ngày | Refactor, explain, generate code |
| 🟢 **FREE** | GPT-5 mini | **0x** | Fix bug, tests | Fix null pointer, generate test file |
| 🟢 **FREE** | GPT-4o | **0x** | Quick chat | "Giải thích regex này" |
| 🟡 **BUDGET** | Grok Code Fast 1 | 0.25x | Review nhanh | Review 10 files thay đổi |
| 🟡 **BUDGET** | Claude Haiku 4.5 | 0.33x | Agent đơn giản | Thêm validation cho 3 endpoints |
| 🟡 **BUDGET** | o3/o4 mini | 0.33x | Reasoning nhẹ | Tìm logic bug trong sort |
| 🔵 **STANDARD** | Sonnet 4.5/4.6 | 1x | Agent phức tạp | Multi-file refactor, full feature |
| 🔵 **STANDARD** | Gemini 3 Pro | 1x | Context dài | Analyze 50 files, find dead code |
| 🔴 **PREMIUM** | Opus 4.5/4.6 | **3x** | Architecture | Design event-driven system |
| ❌ **TRÁNH** | GPT-4.5 | **~50x** | ❌ Không dùng | Tuyệt đối tránh |

### Phân bổ budget khuyến nghị (team 5 người, 300 req/user/tháng)

| Hạng mục | % | Requests | Ví dụ |
|----------|---|----------|-------|
| Agent mode (Sonnet 1x) | 60% | 180 | 180 task phức tạp |
| Budget models (0.33x) | 20% | ~180 effective | Migration, validation |
| Planning (Opus 3x) | 10% | ~10 sessions | Architecture reviews |
| Buffer | 10% | 30 | Thử model mới, ad-hoc |

> 💡 **Kết hợp included models (0x)** cho task đơn giản → stretch budget thêm 2-3x

**Speaker note:** Slide này là reference card — in ra cho team dán bàn. Free models xử lý 70% tasks.

---

## SLIDE 4: Agent Mode — Cơ Chế Hoạt Động

### Agent tự làm gì mà KHÔNG tốn thêm request?

| Bước Agent tự chạy | Ví dụ | Tính request? |
|---------------------|-------|--------------|
| 🔍 Scan codebase | Tìm files liên quan `AuthService` | ❌ Không |
| ✏️ Tạo & edit files | Refactor 5 file, apply changes | ❌ Không |
| 🖥️ Chạy terminal | `npm run build && npm test` | ❌ Không |
| 🔄 Iterate sửa lỗi | TS error → sửa → build lại → pass | ❌ Không |
| 📝 Tạo file mới | Migration script, test files | ❌ Không |
| ✅ **Prompt user** | **Lệnh khởi đầu duy nhất** | ✅ **1 request** |

> 💡 Agent Mode = thuê junior dev: giao task đầy đủ 1 lần → nó tự làm → anh review kết quả cuối.
> Mỗi lần "chen ngang" chỉnh sửa = **thêm 1 request**.

> ✅ **Nguồn:** *"For agentic features, only the prompts you send count as premium requests; actions Copilot takes autonomously to complete your task, such as tool calls, do not."* — docs.github.com

**Speaker note:** Slide này giải thích WHY agent mode tiết kiệm. Key: tool calls = FREE.

---

## SLIDE 5: Case Study #1 — CRUD Invoice Module (12 req → 1 req)

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

## SLIDE 6: Case Study #2 — Database Migration (8 req → 1 req)

**📌 Bối cảnh:** SaaS app cần thêm `organizationId` vào 5 bảng chính → multi-tenant support. Ảnh hưởng 30+ files.

### ❌ Cách cũ: 8 prompts × 1x = 8 premium requests, ~35 phút

### ✅ 1 prompt Agent Mode:

```markdown
## Scope
Thêm multi-tenant: organizationId (FK → Organization) vào 5 bảng:
users, projects, invoices, tasks, comments.
## Chi tiết
- Prisma migration: thêm cột, index, default org cho data cũ
- Update TypeScript types/interfaces
- Update queries: thêm organizationId filter
- Update API: inject organizationId từ auth context
- Update seed: tạo 2 organizations + sample data
## Verify
Chạy `npx prisma migrate dev` → `npm run build && npm test`, tự fix nếu lỗi.
```

### 📊 Kết quả
- Sửa **34 file** (schema, types, services, routes, seeds)
- Migration chạy thành công, 47/47 tests pass
- **1 request, ~12 phút**

**Speaker note:** Migration là use case mạnh nhất vì ảnh hưởng nhiều file — agent tự handle hết.

---

## SLIDE 7: Case Study #3 — Refactor God File 1200 dòng (10 req → 1 req)

**📌 Bối cảnh:** `src/utils.ts` tích lũy 2 năm, 1200 dòng, 47 file import. Dev estimate: 1 ngày.

### ✅ 1 prompt Agent Mode:

```markdown
## Scope
Tách src/utils.ts thành modules:
- src/utils/string.ts, date.ts, validation.ts, api.ts
- src/utils/index.ts: barrel re-export (backward compatibility)
## Constraints
- KHÔNG thay đổi function signatures
- Update TẤT CẢ import paths (47 files)
## Verify
`npm run build && npm test && npm run lint` → ALL PASS.
```

### 📊 Kết quả
- Tạo 5 file mới, update 23 file imports (24 file dùng barrel → không cần sửa)
- Build: 0 errors. Test: 89/89 pass. Lint: pass
- **1 request, ~6 phút. Dev estimate cũ: 1 ngày**

**Speaker note:** Case study refactor — highlight sự khác biệt: 6 phút vs 1 ngày estimate.

---

## SLIDE 8: Case Study #4 — Bug Fix Cross-Service (6 req → 1 req)

**📌 Bối cảnh:** Production 500 error khi tạo order có promotion code. Chỉ xảy ra khi code expired. Team debug 2 giờ chưa tìm ra.

### ✅ 1 prompt Agent Mode:

```markdown
## Bug Report
- POST /api/orders → 500 khi có promotionCode
- Error: "Cannot read property 'discount' of undefined"
- Không lỗi khi bỏ promotionCode hoặc dùng valid code
## Yêu cầu
- Trace: route → controller → orderService → promotionService
- Fix: handle expired promotion → return 400 (not 500)
- Tests: valid promo, expired promo, invalid code, no promo
## Verify: `npm test`
```

### 📊 Kết quả
- Root cause: `promotionService.getByCode()` trả `null` khi expired → `promotion.discount` crash
- Fix: null check + 400 response "Promotion code expired or invalid"
- 4 test cases → all pass
- **1 request, ~5 phút. Team trước đó: 2 giờ chưa tìm ra**

**Speaker note:** Bug fix = high-value use case. Agent trace code nhanh hơn người vì scan toàn bộ call chain.

---

## SLIDE 9: Case Study #5 — Full-Stack Feature (1 req = backend + frontend + tests)

**📌 Bối cảnh:** Sprint task: User Settings page. Spring Boot API + ReactJS + JPA/Hibernate. Trước Copilot: 2 ngày.

### ✅ 1 prompt Agent Mode (Mega-Prompt):

```markdown
## Backend (Spring Boot)
- GET/PUT /api/settings (UserSettings entity, JPA Repository)
- Jakarta Validation (@Valid, @NotBlank, @Email), Spring Security auth guard
- DTO: UserSettingsRequest/Response, MapStruct mapper
- Exception handling: @ControllerAdvice, custom ErrorResponse
## Frontend (ReactJS)
- SettingsPage: 3 tabs (Profile, Security, Notifications)
- React Hook Form, Toast notifications (react-toastify), Loading skeleton
- Axios service: settingsApi.ts (GET/PUT /api/settings)
## Tests
- Backend: @SpringBootTest + MockMvc (endpoints, auth, validation)
- Frontend: @testing-library/react (form, tabs, API mock với MSW)
## Verify: `./mvnw test` (backend) && `npm test` (frontend), tự fix nếu lỗi.
```

### 📊 Kết quả
- Backend: 6 file (Entity, Repository, Service, Controller, DTO, Config) → Frontend: 6 file → Tests: 22 cases → ALL PASS
- **1 premium request, ~15 phút. Estimate cũ: 2 ngày**

**Speaker note:** Full-stack Spring Boot + React trong 1 prompt = đỉnh cao agent mode. Show prompt mẫu này cho team.

---

## SLIDE 10: Tổng Hợp 5 Case Studies — So Sánh Chi Phí

| # | Scenario | Cách cũ (Chat) | Agent Mode | Tiết kiệm |
|---|----------|---------------|------------|-----------|
| 1 | CRUD Invoice module | 12 req, ~45 phút | **1 req, ~8 phút** | **12x** |
| 2 | Multi-tenant migration | 8 req, ~35 phút | **1 req, ~12 phút** | **8x** |
| 3 | Refactor God File 1200 dòng | 10 req, ~1 ngày | **1 req, ~6 phút** | **10x** |
| 4 | Bug fix cross-service | 6 req, ~2 giờ | **1 req, ~5 phút** | **6x** |
| 5 | Full-stack Settings page | 15+ req, ~2 ngày | **1 req, ~15 phút** | **15x** |

### 🔑 Bí quyết chung

1. **Prompt đầy đủ Scope + Constraints + Done + Test** (template S.C.D.T)
2. **Kết thúc bằng verify command:** `npm run build && npm test`
3. **Tham khảo existing code:** `xem src/modules/product/ làm mẫu`
4. **Dùng included model (0x)** cho task đơn giản → hoàn toàn miễn phí

> 💡 **Kết luận:** Tối ưu request đến từ **kiến trúc workflow**, không chỉ từ cách viết prompt.

**Speaker note:** Slide tổng hợp — impact rõ ràng. Dùng bảng này để convince team/management.

---

## SLIDE 11: Kỹ Thuật Nâng Cao 1 — Mega-Prompt & Context Files

### 🚀 Mega-Prompt — Gộp cả sprint task vào 1 session

**Ví dụ — Search & Filter feature:**
```markdown
Implement search & filter cho Product listing:
1. Backend: GET /api/products?search=&category=&minPrice=&maxPrice=&page=
2. Frontend: SearchBar (debounce 300ms), FilterPanel, ProductGrid
3. Tests: search accuracy, filter combos, pagination, URL persistence
4. Chạy `npm run build && npm test`, tự fix nếu lỗi.
```
**Chi phí:** 1 premium request. **Output:** Full-stack feature hoàn chỉnh.

### 🚀 Context Files — "Dạy" Copilot 1 lần, dùng mãi

Tạo `.github/copilot-instructions.md` (auto-loaded cho mọi interaction):
```markdown
# Project: HealthTrack SaaS
- Stack: Express 5 + TypeScript 5.5 + Prisma 6 + React 19
- Test: Vitest + Supertest, Testing Library
- Naming: camelCase vars, PascalCase types
- Error: { success: false, error: { code, message } }
- Luôn chạy `npm run build && npm test` sau khi edit
```

> **Hiệu quả thực tế:** Team 8 devs — giảm ~40% follow-up prompts sau khi thêm context file.

**Speaker note:** Mega-prompt + context file = combo mạnh nhất. Đầu tư 30 phút viết instructions → tiết kiệm hàng giờ/tuần.

---

## SLIDE 12: Kỹ Thuật Nâng Cao 2 — Plan First & Free First

### 🚀 "Plan Then Execute" — Giảm waste

```
Bước 1 (Free): Plan Agent (GPT-4.1, 0x) → phân tích scope, liệt kê tasks
Bước 2 (Review): Dev review plan, chỉnh sửa (~2 phút)
Bước 3 (Execute): Agent Mode (Sonnet 1x) → implement theo plan
```

**Ví dụ — Payment Integration (Stripe):**
> - Plan Agent (0x) → 12 subtasks, 8 files cần tạo, 3 files cần sửa
> - Dev review → bỏ 2 subtask, thêm note webhook
> - Agent implement → **1 premium request**
>
> **Không plan:** Agent dùng sai flow → phải prompt lại → 3+ request thay vì 1

### 🚀 "Included Model First" — 0x trước, Premium sau

```
Task mới? → Thử GPT-4.1 (0x) TRƯỚC
  → Đủ tốt? → ✅ DONE (0 premium req)
  → Chưa đủ? → Switch Sonnet 4.6 (1x)
  → Cần deep reasoning? → Opus (3x), CHỈ cho architecture
```

**Kết quả thực tế (Team 5 devs, 1 tháng):**
- 70% tasks → GPT-4.1 (0x) = **0 premium requests**
- 25% tasks → Sonnet (1x)
- 5% tasks → Opus (3x)
- → **Tiết kiệm ~65% budget** so với dùng Sonnet cho mọi task

**Speaker note:** 2 chiến lược kiểm soát chi phí hiệu quả nhất. Plan = tránh waste. Free First = tối đa budget.

---

## SLIDE 13: Kỹ Thuật Nâng Cao 3 — Custom Agents & Auto Selection

### 🚀 Custom Agents (VS Code 2026) — .agent.md files

**Ví dụ — Pipeline Plan → Implement:**

`.github/agents/planner.agent.md`:
```yaml
---
name: Planner
tools: ['search/codebase', 'web/fetch']
model: ['GPT-4.1']  # Free, 0x
handoffs:
  - label: Implement Plan
    agent: implementer
---
# Analyze codebase, output structured plan. Do NOT edit code.
```

`.github/agents/implementer.agent.md`:
```yaml
---
name: Implementer
tools: ['edit', 'run/terminal', 'search/codebase']
model: ['Claude Sonnet 4.6']
---
# Follow the plan. Run build + test. Fix errors automatically.
```

**Kết quả:** Plan (0x, free) → review → implement (1x) = **chỉ 1 premium request**

### 🚀 Auto Model Selection — Discount 10%

| Scenario | Chọn | Lý do |
|----------|------|-------|
| Quick chat | Auto | Copilot chọn model rẻ, tiết kiệm 10% |
| Agent phức tạp | Manual (Sonnet) | Đảm bảo model mạnh |
| Architecture | Manual (Opus) | Cần reasoning mạnh nhất |

**Speaker note:** Custom Agents là tính năng mới 2026 — handoffs rất mạnh cho team workflow.

---

## SLIDE 14: Playbook — Template S.C.D.T + 3 Quy Tắc Vàng

### Template Prompt Chuẩn Hóa S.C.D.T

```markdown
## Scope       → [Task cần làm — càng cụ thể càng tốt]
## Constraints  → [Framework, conventions, files KHÔNG sửa]
## Done         → [Tiêu chí hoàn thành, checklist]
## Test         → [Commands verify: npm run build && npm test]
```

### Prompt xấu vs Prompt tốt

| ❌ Xấu (10+ req) | ✅ Tốt (1 req) |
|-------------------|----------------|
| "Thêm login" | **S:** POST /auth/login + register + refresh. **C:** JWT, bcrypt, rate limit. **D:** 3 endpoints hoạt động. **T:** `npm test` |
| "Fix bug" | **S:** 500 error GET /orders/:id khi không tồn tại. **C:** Giữ response format. **D:** 400 invalid, 404 not found. **T:** 3 test cases, `npm test` |
| "Refactor code" | **S:** Tách UserService 800 dòng → 3 services. **C:** Giữ public interface. **D:** 3 files, 0 TS errors. **T:** `npm run build && npm test` |

### 3 Quy Tắc Vàng

| # | Quy tắc | Metric |
|---|---------|--------|
| 1 | **1 Prompt = 1 Task Hoàn Chỉnh** | Trung bình > 3 prompts/task → cải thiện prompt quality |
| 2 | **Free First** | > 40% tasks dùng premium → review lại |
| 3 | **Plan → Review → Execute** | Plan step giảm > 50% re-prompts |

**Speaker note:** Print slide này cho team. Template S.C.D.T + 3 rules = framework bắt đầu ngay ngày mai.

---

## SLIDE 15: Playbook — Model Tiering & Governance

### Chiến Lược Chọn Model cho Team

| Giai đoạn | Model | Multiplier | Ví dụ task |
|-----------|-------|-----------|------------|
| 💬 Quick chat | GPT-4.1 | **0x** | "Giải thích regex này" |
| 🔧 Code gen | GPT-4.1 / GPT-5 mini | **0x** | CRUD boilerplate, generate types |
| 🤖 Agent đơn giản | Haiku 4.5 | **0.33x** | Validation, update imports |
| 🧠 Agent phức tạp | Sonnet 4.5/4.6 | **1x** | Multi-file refactor, full feature |
| 📐 Architecture | Opus 4.5/4.6 | **3x** | System design, critical decisions |
| 🔀 Không biết | Auto | **-10%** | Để Copilot tự chọn |

### Governance & Monitoring

| Hạng mục | Hành động | Tần suất | Phụ trách |
|----------|-----------|----------|-----------|
| Usage audit | GitHub Billing Analytics theo user/team | Hàng tháng | Tech Lead |
| Budget cap | Set spending limit cho overage | Khi setup | Admin |
| Cost alerts | Cảnh báo > 80% monthly quota | Tự động | Admin |
| Model policy | Document models cho từng task type | Hàng quý | Team |
| Rate limit | Đào tạo: rate limit ≠ monthly quota | 1 lần | Tech Lead |

**Ví dụ alert thực tế:**
> User A dùng 120/300 req tuần 1 (40%) — 🟡 Warning
> Root cause: Opus (3x) cho simple tasks → 40 interactions = 120 req
> Fix: GPT-4.1 (0x) cho 80% task → tuần 2 chỉ 15 req

**Speaker note:** Governance slide cho management. Monthly audit là must-have.

---

## SLIDE 16: VS Code Local — 3 Surfaces & Billing

> 🎯 **Phạm vi:** Copilot **cục bộ trong VS Code** — KHÔNG phải Cloud Agent

### 3 Surfaces + Billing

| Surface | Mô tả | Billing | Ví dụ |
|---------|-------|---------|-------|
| ✏️ **Inline Completions** | Ghost text khi gõ | Quota riêng (2k/tháng Free, ∞ Paid) | Tab accept suggestion |
| 💬 **Chat Mode** | Hỏi đáp, explain | 1 req × multiplier / message | "Giải thích hàm này" |
| 🤖 **Agent Mode** | Edit files, terminal, iterate | 1 req × multiplier / prompt | "Implement CRUD module" |

### Phân biệt 3 loại request

```
┌────────────────────────────────────┐
│ 1. Inline Completions              │
│    → Quota riêng, KHÔNG premium    │
│ 2. Chat / Ask Mode                 │
│    → 1 premium req × multiplier    │
│ 3. Agent Mode                      │
│    → 1 premium req × multiplier    │
│    → Tool calls = MIỄN PHÍ         │
└────────────────────────────────────┘
```

### Chiến lược: "Free First, Agent Always"

1. Chat đơn giản → GPT-4.1 (0x) = **miễn phí**
2. Agent mode cho edit code → GPT-4.1 (0x) = **miễn phí** + tự làm nhiều bước
3. Chưa đủ tốt → switch Sonnet 4.6 (1x)
4. Planning → Plan Agent (Sonnet/Opus) → review → implement model rẻ hơn

**Speaker note:** VS Code Local Agent Mode = surface tiết kiệm nhất. 0x model + agent = hoàn toàn free.

---

## SLIDE 17: Model Mismatch — Vấn Đề Phát Hiện

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
