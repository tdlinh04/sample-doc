# Case Studies: Tối Ưu Request & Tách Planning/Execution trong GitHub Copilot

## Tóm tắt cho quản lý (Executive Summary)

Copilot tính phí theo **request** (và multiplier theo model), không theo tổng token. Vì vậy, chiến lược tối ưu là gửi prompt chất lượng cao để agent tự chạy nhiều bước (tool calls không bị tính thêm trong cùng phiên agentic). Đồng thời, với luồng dùng trong VS Code + GitHub Pull Requests extension, tài liệu chính thức cho thấy đây là kiến trúc **2 pha**: (1) scoping/planning cục bộ trong IDE, (2) execution trên GitHub Actions bởi cloud agent. Điểm quan trọng: hiện **không có bằng chứng chính thức** rằng pha planning này cố định dùng GPT-4o; ngược lại GPT-4o đã bị deprecated trong Copilot Chat từ 08/2025.

---

## Nguồn tham khảo đã sử dụng

| # | Nguồn | Ghi chú |
|---|--------|---------|
| S1 | [Requests in GitHub Copilot](https://docs.github.com/en/copilot/concepts/billing/copilot-requests) | Định nghĩa request/premium request; tool calls agentic không tính thêm |
| S2 | [About billing for Copilot in organizations](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises) | Cơ chế quota, overage, quản trị chi phí theo tổ chức |
| S3 | [GitHub Copilot features](https://docs.github.com/en/copilot/get-started/features) | Mô tả agent mode và hành vi tự động nhiều bước |
| S4 | [About GitHub Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli) | CLI mode, default model CLI, cơ chế tính request cho CLI |
| S5 | [Supported AI models](https://docs.github.com/en/copilot/reference/ai-models/supported-models) | Danh sách model + multipliers |
| S6 | [About Copilot auto model selection](https://docs.github.com/en/copilot/concepts/auto-model-selection) | Auto chọn model theo health/performance, discount 10% |
| S7 | [Base and LTS models](https://docs.github.com/en/copilot/concepts/fallback-and-lts-models) | Fallback khi hết premium requests |
| S8 | [Rate limits for GitHub Copilot](https://docs.github.com/en/copilot/concepts/rate-limits) | Ảnh hưởng bởi giới hạn tốc độ |
| S9 | [Hosting of models for GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/model-hosting) | Multi-provider hosting |
| S10 | [Changing the AI model for Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/changing-the-ai-model) | Entry points có model picker; nơi không có picker thì dùng Auto |
| S11 | [Asking Copilot to create a PR](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-a-pr) | Luồng delegate từ VS Code sang cloud agent |
| S12 | [VS Code docs - Planning with agents](https://code.visualstudio.com/docs/copilot/agents/planning) | Tách plan agent và implement step |
| S13 | [VS Code docs - Copilot settings](https://raw.githubusercontent.com/microsoft/vscode-docs/main/docs/copilot/reference/copilot-settings.md) | `chat.planAgent.defaultModel` và `implementAgent.model` |
| S14 | [Changelog - Start and track coding agent from VS Code](https://github.blog/changelog/2025-07-14-start-and-track-github-copilot-coding-agent-sessions-from-visual-studio-code/) | Có bước scoping cục bộ trước khi chuyển cloud agent |
| S15 | [Changelog - 1 premium request/session cho coding agent](https://github.blog/changelog/2025-07-10-github-copilot-coding-agent-now-uses-one-premium-request-per-session/) | Khẳng định cost theo session cho cloud agent |
| S16 | [Changelog - Deprecation of GPT-4o in Copilot Chat](https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/) | GPT-4o không còn là model chat kể từ 08/2025 |
| S17 | [Reddit - Copilot pricing change thread](https://reddit.com/r/GithubCopilot/comments/1l7flts/github_copilot_going_from_unlimited_to_004_per/) | Tín hiệu cộng đồng về confusion quota/usage tracking |
| S18 | [Reddit - 1 premium request/session thread](https://reddit.com/r/GithubCopilot/comments/1lwlp9r/github_copilot_coding_agent_now_uses_one_premium/) | Nhiều user nhầm session billing vs per-prompt billing |
| S19 | [Reddit - My wallet speaks for me](https://reddit.com/r/GithubCopilot/comments/1rhxap0/my_wallet_speaks_for_me_github_copilot_in_vs_code/) | Hành vi thực tế: pacing usage và so sánh chi phí |
| S20 | [Reddit - Copilot CLI context bloat](https://reddit.com/r/GithubCopilot/comments/1s4yng2/copilot_cli_is_unusable_with_large_global_skill/) | Pain point thực tế khi dùng nhiều global skills/plugins |
| S21 | [Reddit - Constant rate limiting at low monthly usage](https://reddit.com/r/GithubCopilot/comments/1qvoo47/whats_with_the_constant_rate_limiting_im_at_14_of/) | Phản ánh khác biệt giữa monthly quota và short-window limits |
| S22 | [Reddit - Usage optimization by model tiering](https://reddit.com/r/opencodeCLI/comments/1qttkzs/increased_usage_of_github_copilot_premium_requests/) | Pattern: dùng model đắt cho spec, model rẻ cho triển khai |

---

## SLIDE 1: Trang bìa

**Tiêu đề:** Case Studies: Tối Ưu Request & Tách Planning/Execution trong GitHub Copilot

- Case study 1: Tối ưu premium requests để Copilot làm nhiều việc nhất
- Case study 2: Chứng minh luồng VS Code extension là 2 pha (plan/scoping vs execute)
- Dựa trên docs/changelog chính thức GitHub + VS Code (cập nhật 04/2026)
- Mọi claim được gắn nhãn rõ: **Sự thật** vs **Giả thuyết**

**Speaker note:** Mục tiêu của deck là giúp team tối ưu chi phí sử dụng Copilot theo cơ chế billing hiện tại, đồng thời tránh hiểu sai về model chạy task khi delegate từ VS Code sang cloud agent.

---

## SLIDE 2: Cách Copilot tính phí — Premium Requests

**Tiêu đề:** Billing theo request, không theo token

- **Sự thật [S1]:** 1 prompt người dùng gửi = 1 premium request × model multiplier
- **Sự thật [S1]:** Với agentic features, chỉ prompt bạn gửi bị tính; tool calls tự động không bị tính thêm
- **Sự thật [S1][S5]:** Included models trên paid plans (GPT-4.1, GPT-4o, GPT-5 mini) có multiplier 0x
- **Sự thật [S5]:** Opus 4.6 = 3x, Opus 4.6 fast mode = 30x, Sonnet 4.5/4.6 = 1x
- **Sự thật [S2]:** Paid plans có quota premium request riêng cho các model premium

**Speaker note:** Core insight: tối ưu chi phí không nằm ở việc giảm token output, mà nằm ở việc giảm số lần gửi prompt và chọn đúng model multiplier.

---

## SLIDE 3: Case Study 1 — Tận dụng agent để "nén" requests

**Tiêu đề:** CS1: Ít prompt hơn, nhiều việc hơn

- **Sự thật [S3]:** Agent mode có thể tự xác định file, đề xuất edit, chạy terminal, tự iterate sửa lỗi
- **Sự thật [S1]:** Các bước tự động đó không phát sinh premium request mới nếu không có prompt mới từ user
- **Sự thật [S4]:** Copilot CLI cũng tính theo mỗi prompt (không theo số tool calls nội bộ)
- **Sự thật [S15]:** Copilot cloud agent dùng 1 premium request cho mỗi session (khi tạo/chỉnh PR)
- **Giả thuyết vận hành:** Prompt càng đầy đủ upfront, giá trị thu được trên mỗi request càng cao

**Speaker note:** Đây là đòn bẩy chính để "bắt Copilot làm càng nhiều càng tốt": gộp task thành một phiên agentic đủ rõ thay vì chat chia nhỏ nhiều lượt.

---

## SLIDE 4: Bảng so sánh chi phí

**Tiêu đề:** 10 prompt nhỏ vs 1 prompt agentic

- **Workflow A (10 prompts, Sonnet 4.5):** 10 × 1x = **10 premium requests**
- **Workflow B (1 prompt agent mode, Sonnet 4.5):** 1 × 1x = **1 premium request**
- **Workflow C (1 prompt, GPT-4.1):** 1 × 0x = **0 premium requests**
- **Workflow D (cloud agent 1 session):** **1 premium request/session** [S15]
- **Kết luận:** Tối ưu request đến từ kiến trúc workflow, không chỉ từ prompt wording

**Speaker note:** Một phiên agentic dài với nhiều thao tác có thể rẻ hơn nhiều so với nhiều lượt chat ngắn.

---

## SLIDE 5: Playbook triển khai cho team

**Tiêu đề:** Cách vận hành để giảm request thực tế

- Chuẩn hóa template prompt cho task lớn (scope, constraints, done criteria, test criteria)
- Ưu tiên agent mode/cloud agent cho task đa bước; tránh chia nhỏ thành nhiều prompt vụn
- Dùng included models cho task nhẹ; giữ premium models cho task reasoning sâu
- Dùng auto model selection khi không bắt buộc model cụ thể (được discount multiplier trên paid plans) [S6]
- Theo dõi usage theo user/team định kỳ để phát hiện pattern lãng phí [S2]

**Speaker note:** Tối ưu bền vững cần cả kỹ thuật prompt lẫn policy vận hành.

---

## SLIDE 6: Case Study 2 — "GPT-4o chỉ phân task?"

**Tiêu đề:** CS2: Sửa giả thuyết cho đúng với docs

- **Giả thuyết ban đầu:** Khi dùng GitHub extension trong VS Code, GPT-4o chỉ dùng để phân task, không chạy task
- **Sự thật [S16]:** GPT-4o đã bị deprecated trong Copilot Chat từ 08/2025
- **Sự thật [S14]:** Luồng VS Code có bước scoping/research cục bộ rồi mới chuyển sang cloud agent sau khi user xác nhận
- **Sự thật [S11]:** Sau khi delegate, task chạy trong môi trường cloud (GitHub Actions), mở PR và cập nhật session
- **Kết luận:** Có thể chứng minh **tách pha planning/scoping và execution**, nhưng không thể khẳng định planning model cố định là GPT-4o

**Speaker note:** Đây là điểm chỉnh quan trọng để slide không over-claim.

---

## SLIDE 7: Kiến trúc 2 pha cần trình bày

**Tiêu đề:** VS Code extension: Local scope -> Cloud execute

- **Pha 1 (local):** Copilot Chat trong IDE phân tích/scoping task, có thể hỏi lại và chuẩn hóa yêu cầu [S14]
- **Pha 2 (handoff):** User bấm Continue/Delegate để chuyển task sang Copilot cloud agent [S11][S14]
- **Pha 3 (cloud execution):** Agent chạy trên GitHub Actions, commit lên branch/PR và log tiến trình [S11][S14]
- **Billing cho cloud execution:** Tính theo session cloud agent [S1][S15]
- **Hệ quả:** Model chat local và model execution cloud có thể khác nhau

**Speaker note:** Đây là phần "chứng minh" quan trọng nhất cho audience kỹ thuật và quản lý.

---

## SLIDE 8: Bằng chứng model cho planning vs execution

**Tiêu đề:** Chúng ta chứng minh được gì, chưa chứng minh được gì?

- **Chứng minh được [S12][S13]:** VS Code có setting riêng cho plan agent và implementation step (`chat.planAgent.defaultModel`, `implementAgent.model`)
- **Chứng minh được [S10]:** Với cloud agent, nếu entry point không có model picker thì dùng **Auto**
- **Chứng minh được [S11][S14]:** Delegate từ VS Code sẽ đẩy execution sang cloud agent, không chạy toàn bộ ở local chat
- **Chưa chứng minh được:** Mapping cố định "planning luôn GPT-4o" cho mọi thời điểm và mọi org policy
- **Lưu ý [S16]:** Dùng GPT-4o làm luận điểm trung tâm cho chat sẽ lỗi thời với dữ liệu hiện tại

**Speaker note:** Slide này giúp phân biệt fact-based evidence và suy diễn.

---

## SLIDE 9: Protocol demo để thuyết phục trong buổi trình bày

**Tiêu đề:** Demo 5 bước xác thực thực nghiệm

- B1: Trong VS Code, dùng extension và khởi tạo task từ chat -> delegate sang cloud agent [S11]
- B2: Quan sát prompt phản hồi xác nhận handoff và link PR/session [S14]
- B3: Mở session logs để thấy execution diễn ra trên cloud agent, không ở local terminal [S14]
- B4: Đối chiếu usage metrics/premium counters theo session để map chi phí [S1][S15]
- B5: Nếu có quyền admin, đối chiếu model usage trong báo cáo để kiểm tra Auto resolution

**Speaker note:** Đây là demo có thể lặp lại, dễ thuyết phục stakeholder hơn tranh luận cảm tính về "văn phong model".

---

## SLIDE 10: Tránh nhầm 3 công cụ khác nhau

**Tiêu đề:** Copilot CLI vs GitHub CLI vs VS Code extension

- **Copilot CLI (`copilot`) [S4]:** Agent ở terminal, default model Sonnet 4.5, tính theo mỗi prompt
- **GitHub CLI (`gh agent-task`) [S11]:** Entry point gọi cloud agent session trên GitHub
- **VS Code + GitHub Pull Requests ext [S11][S14]:** Local scoping rồi handoff cloud execution
- **Cloud agent model selection [S10]:** Một số entry points có picker; nơi không có picker dùng Auto
- **Khuyến nghị:** Khi phân tích billing/model, luôn ghi rõ "surface" đang nói tới

**Speaker note:** Nhiều tranh cãi xuất phát từ việc trộn các bề mặt sản phẩm vào một nhóm.

---

## SLIDE 11: Rủi ro & khuyến nghị

**Tiêu đề:** Rủi ro thực tế và cách giảm thiểu

- **Rủi ro 1:** Billing surprise khi dùng model multiplier cao (đặc biệt Opus fast mode 30x) [S5]
- **Rủi ro 2:** Over-claim về model (ví dụ gắn cứng GPT-4o) làm sai quyết định kỹ thuật [S16]
- **Rủi ro 3:** Nhầm lẫn local chat với cloud execution dẫn tới sai cách đo hiệu quả
- **Khuyến nghị 1:** Chuẩn hóa workflow agentic + template prompt để giảm số requests
- **Khuyến nghị 2:** Tách rõ policy model cho planning và execution trong team docs
- **Khuyến nghị 3:** Audit usage và session logs hàng tháng để hiệu chỉnh playbook

**Speaker note:** Mục tiêu cuối cùng là quản trị được cả cost lẫn kỳ vọng chất lượng.

---

## SLIDE 12: Dữ liệu cộng đồng Reddit

**Tiêu đề:** User thực tế đang dùng Copilot như thế nào?

- **Pattern 1 [S19][S22]:** Nhiều user tối ưu bằng model tiering (model mạnh cho spec/reasoning, model rẻ hơn cho triển khai)
- **Pattern 2 [S19]:** User có xu hướng theo dõi nhịp tiêu hao premium requests theo tuần/tháng để tránh overage
- **Pain point 1 [S18]:** Nhầm lẫn giữa billing theo session (cloud agent) và billing theo prompt (chat/agent mode)
- **Pain point 2 [S21]:** Có thể gặp rate limiting dù quota tháng còn nhiều (do giới hạn theo cửa sổ ngắn)
- **Pain point 3 [S20]:** Với Copilot CLI, setup quá nhiều global skills/plugins có thể gây context bloat và lỗi token limit

**Speaker note:** Dữ liệu Reddit là dữ liệu hành vi thực tế, hữu ích để hiểu adoption và friction. Tuy nhiên đây là dữ liệu cộng đồng nên chỉ dùng để tạo giả thuyết vận hành, không thay thế tài liệu chính thức.

---

## SLIDE 13: Q&A

**Tiêu đề:** Hỏi & Đáp

- Billing requests: https://docs.github.com/en/copilot/concepts/billing/copilot-requests
- Copilot cloud agent model selection: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/changing-the-ai-model
- Delegate từ VS Code: https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-a-pr
- VS Code planning docs: https://code.visualstudio.com/docs/copilot/agents/planning
- GPT-4o deprecation in chat: https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/
- Reddit evidence index: https://reddit.com/r/GithubCopilot/comments/1rhxap0/my_wallet_speaks_for_me_github_copilot_in_vs_code/

**Speaker note:** Key takeaways: (1) Tối ưu request bằng workflow agentic đúng cách. (2) Có thể chứng minh tách planning/execution theo docs. (3) Không nên chốt luận điểm dựa trên giả định model đã lỗi thời.

---

## PHỤ LỤC: Bảng model multipliers tiêu biểu (tham chiếu nhanh)

| Model | Paid plan multiplier | Free plan multiplier |
|-------|---------------------|---------------------|
| Claude Haiku 4.5 | 0.33x | 1x |
| Claude Sonnet 4.5 | 1x | N/A |
| Claude Sonnet 4.6 | 1x | N/A |
| Claude Opus 4.6 | 3x | N/A |
| Claude Opus 4.6 (fast mode) | 30x | N/A |
| GPT-4.1 | 0x | 1x |
| GPT-4o | 0x | 1x |
| GPT-5 mini | 0x | 1x |
| GPT-5.4 | 1x | N/A |
| GPT-5.4 mini | 0.33x | N/A |
| Grok Code Fast 1 | 0.25x | 1x |
| Raptor mini | 0x | 1x |

*Nguồn: [S5] — danh sách có thể thay đổi theo thời gian, luôn kiểm tra bảng chính thức trước khi chốt policy nội bộ.*
