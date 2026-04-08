
bash -lc sed -n '1,260p' /mnt/data/copilot_case_study_revised_2026-04-07.md
# Case Study: Tối Ưu Premium Requests & Tách Local Planning / Cloud Execution trong GitHub Copilot

> Phiên bản đã rà soát theo docs/changelog công khai đến ngày **2026-04-07**.
> 
> Mục tiêu: 
> 1. Chỉ ra cách tận dụng cơ chế tính **premium requests** để “bắt Copilot làm nhiều nhất có thể trên mỗi request”.
> 2. Chứng minh bằng tài liệu chính thức rằng trong luồng **VS Code + GitHub Pull Requests extension**, có tách biệt giữa **local planning/scoping** và **cloud execution**.
> 3. Tránh over-claim rằng “GPT-4o luôn là model planning” — vì hiện tại tài liệu chính thức **không chứng minh được** điều đó.

---

## Tóm tắt cho quản lý (Executive Summary)

- **Sự thật [S1]:** Đơn vị khan hiếm cần tối ưu trong Copilot là **request/session theo từng surface**, nhân với **model multiplier**; Copilot **không tính tiền theo tổng token** trong cách anh em thường tranh luận nội bộ.
- **Sự thật [S1]:** Với các tính năng agentic, **tool calls tự động không bị tính thành request riêng**; vì vậy leverage lớn nhất là gom đủ context ngay từ prompt đầu để agent tự chạy nhiều bước.
- **Sự thật [S1]:** Với **Copilot cloud agent**, cách tính mới nhất là **1 premium request / session × model multiplier**; tuy nhiên **mỗi steering comment real-time** trong session đang chạy cũng tiêu tốn thêm **1 premium request / session × model multiplier**.
- **Sự thật [S9][S4]:** Trong VS Code với GitHub Pull Requests extension, GitHub mô tả một luồng có **research/scoping cục bộ trong IDE**, sau đó mới **handoff sang cloud agent chạy trên GitHub Actions**.
- **Kết luận chứng minh được [S5][S10][S2]:** Deck nên chốt luận điểm là **“tách local planning/scoping và cloud execution”**, không nên chốt **“GPT-4o chỉ dùng để phân task”**. Lý do: GPT-4o đã bị deprecated khỏi **Copilot Chat** từ **2025-08-06**, còn danh sách model hiện tại cho **cloud agent** cũng **không có GPT-4o**.

---

## Nguồn tham khảo chính

| ID | Nguồn | Dùng để chứng minh |
|---|---|---|
| S1 | [Requests in GitHub Copilot](https://docs.github.com/en/copilot/concepts/billing/copilot-requests) | Cách tính request, steering comments, SKU tách riêng |
| S2 | [Supported AI models in GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/supported-models) | Multipliers, model list hiện tại |
| S3 | [About Copilot auto model selection](https://docs.github.com/en/copilot/concepts/auto-model-selection) | Auto selection, discount 10% trong Chat |
| S4 | [About GitHub Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent) | Cloud agent chạy trên GitHub Actions |
| S5 | [Changing the AI model for GitHub Copilot cloud agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/coding-agent/changing-the-ai-model) | Model picker, Auto, supported models cloud agent |
| S6 | [Asking GitHub Copilot to create a pull request](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-a-pr) | Delegate từ VS Code, GitHub CLI |
| S7 | [Tracking GitHub Copilot's sessions](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/track-copilot-sessions) | View Session, logs, token usage |
| S8 | [About GitHub Copilot CLI](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/agents/copilot-cli/about-copilot-cli) | Copilot CLI default model, request per prompt |
| S9 | [Start and track GitHub Copilot coding agent sessions from VS Code](https://github.blog/changelog/2025-07-14-start-and-track-github-copilot-coding-agent-sessions-from-visual-studio-code/) | Local scoping -> cloud handoff |
| S10 | [Deprecation of GPT-4o in Copilot Chat](https://github.blog/changelog/2025-08-06-deprecation-of-gpt-4o-in-copilot-chat/) | GPT-4o không còn là chat model |
| S11 | [Planning with agents in VS Code](https://code.visualstudio.com/docs/copilot/agents/planning) | Settings riêng cho planning/implementation |
| S12 | [Using agents in Visual Studio Code](https://code.visualstudio.com/docs/copilot/agents/overview) | Local / Copilot CLI / Cloud agent và handoff |
| S13 | [Monitoring your GitHub Copilot usage and entitlements](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/monitor-premium-requests) | Premium request analytics |
| S14 | [Rate limits for GitHub Copilot](https://docs.github.com/en/copilot/concepts/rate-limits) | Quota tháng khác rate limit cửa sổ ngắn |
| S15 | [Data available in Copilot usage metrics](https://docs.github.com/en/copilot/reference/copilot-usage-metrics/copilot-usage-metrics) | Model usage per chat mode, agent-initiated changes per model |

---

## SLIDE 1: Trang bìa

**Tiêu đề:** Case Study: Tối Ưu Premium Requests & Tách Local Planning / Cloud Execution trong GitHub Copilot

- Case study 1: Tối đa hóa lượng việc Copilot hoàn thành trên mỗi request/session
- Case study 2: Chứng minh luồng VS Code là **local scoping -> cloud execution**, không phải “chat local tự chạy hết”
- Deck này ưu tiên **fact-based claims** từ docs/changelog chính thức
- Cập nhật theo dữ liệu công khai đến **2026-04-07**

**Speaker note:** Mục tiêu của deck là giúp team tối ưu chi phí dùng Copilot theo cơ chế billing hiện tại, đồng thời sửa một số giả định chưa đủ bằng chứng về model dùng cho planning.

---

## SLIDE 2: Copilot tính phí theo gì?

**Tiêu đề:** Request/session là đơn vị tối ưu, không phải token

- **Sự thật [S1]:** Mỗi prompt user gửi vào Copilot Chat tính là **1 premium request × model multiplier**
- **Sự thật [S1]:** Với tính năng agentic, **tool calls do Copilot tự thực hiện không bị tính thành request riêng**
- **Sự thật [S1]:** **Copilot CLI** tính theo **mỗi prompt**
- **Sự thật [S1]:** **Copilot cloud agent** tính theo **mỗi session**, và **mỗi steering comment real-time** trong session cũng tiêu tốn thêm request
- **Sự thật [S1]:** Từ **2025-11-01**, **Copilot cloud agent** và **Spark** được theo dõi ở **SKU tách riêng**

**Speaker note:** Đây là phần cần sửa rõ so với deck cũ: “1 request/session” là đúng nhưng chưa đủ, vì steering comments trong cloud session vẫn có chi phí riêng.

---

## SLIDE 3: Case Study 1 — Cách “nén” nhiều việc vào ít requests hơn

**Tiêu đề:** CS1: Tối đa hóa “work completed per request”

- **Ý tưởng vận hành:** Không tối ưu theo token; hãy tối ưu theo **khối lượng việc agent tự làm sau một prompt**
- Đưa đủ context ngay từ đầu: **scope, constraints, acceptance criteria, test/lint commands, non-goals**
- Chọn surface cho đúng việc: để agent **tự research -> edit -> test -> iterate** trong cùng một request/session [S1][S4][S8]
- Tránh chia nhỏ một task lớn thành nhiều prompt “vụn” chỉ vì thói quen chat tuần tự
- Với cloud agent, tránh steering comment không cần thiết vì có thể phát sinh thêm cost [S1]

**Speaker note:** “Bắt Copilot làm càng nhiều càng tốt” nghĩa là gom task đủ rõ để agent chạy trọn vòng lặp, thay vì liên tục vi chỉnh bằng nhiều prompt nhỏ.

---

## SLIDE 4: So sánh chi phí theo workflow

**Tiêu đề:** 10 prompt nhỏ thường đắt hơn 1 prompt agentic tốt

- **Workflow A:** 10 prompt ngắn trên Claude Sonnet 4.5 => **10 × 1x = 10 premium requests** [S1][S2]
- **Workflow B:** 1 prompt Copilot CLI đủ rõ trên Claude Sonnet 4.5 => **1 × 1x = 1 premium request** [S1][S8]
- **Workflow C:** 1 cloud-agent session để tạo PR => **1 session × model multiplier**, trừ khi anh tiếp tục steering trong lúc session chạy [S1]
- **Workflow D:** Trên paid plan, các included/0x models như **GPT-4.1** hoặc **GPT-5 mini** không ăn premium requests; nhưng **model availability phụ thuộc surface** và có thể thay đổi theo thời gian [S1][S2]
- **Kết luận:** Tối ưu request đến từ **thiết kế workflow**, không chỉ từ prompt wording

**Speaker note:** Bài học quan trọng là gom context và chuyển đúng surface có thể tiết kiệm hơn rất nhiều so với tối ưu câu chữ theo kiểu micro.

---

## SLIDE 5: Playbook triển khai cho team

**Tiêu đề:** Cách vận hành để giảm request thực tế

- Dùng **Plan / Ask local** để làm rõ spec, edge cases, acceptance criteria trước [S11][S12]
- Sau khi đủ rõ, **handoff đúng 1 lần** sang surface thực thi:
  - **Copilot CLI** nếu muốn background work **trên máy của mình** [S12]
  - **Cloud agent** nếu muốn kết quả đi thẳng vào **branch/PR/logs** [S4][S6]
- Dùng **Auto** trong **Copilot Chat** khi không bắt buộc model cụ thể; GitHub nói Auto có **discount 10% multiplier trong Chat** [S3]
- Giữ model multiplier cao cho task thực sự khó; task routine nên dùng included/low-multiplier models [S2][S13]
- Theo dõi usage trong IDE và billing analytics định kỳ [S13]

**Speaker note:** Quy trình tốt nhất thường là: spec local -> implement background -> review PR, thay vì chat từng bước cho đến lúc xong.

---

## SLIDE 6: Các “bẫy” làm team hiểu sai cost

**Tiêu đề:** 4 caveat quan trọng deck cũ nên sửa

- **Caveat 1 [S3]:** Auto discount **chỉ áp dụng cho Copilot Chat**, không phải mọi surface
- **Caveat 2 [S14]:** **Quota tháng còn** không đồng nghĩa với **không bị rate limit**; rate limit là cơ chế theo cửa sổ thời gian ngắn
- **Caveat 3 [S2]:** **Model multipliers và model list có thể đổi**, không nên hard-code policy quá lâu
- **Caveat 4 [S4]:** Nếu nhìn tổng chi phí, cloud agent còn dùng **GitHub Actions minutes** bên cạnh premium requests

**Speaker note:** Khi team tranh luận về “sao tháng này vẫn bị chặn dù quota còn”, thường là đang nhầm giữa budget/quota và rate limits.

---

## SLIDE 7: Giả thuyết cần sửa lại cho đúng

**Tiêu đề:** Từ “GPT-4o chỉ phân task?” sang claim chứng minh được

- **Giả thuyết ban đầu:** Trong VS Code, GPT-4o chỉ dùng để phân task, không chạy task
- **Tài liệu chính thức chứng minh được [S9][S4]:** Có **local research/scoping trong IDE**, rồi mới **handoff sang cloud agent**
- **Tài liệu chính thức chứng minh được [S11][S12]:** VS Code có khái niệm **planning** và **implementation** riêng, thậm chí có setting model riêng cho hai pha
- **Tài liệu chính thức chứng minh được [S5]:** Với **cloud agent**, nơi nào **không có model picker** thì **Auto** sẽ được dùng tự động
- **Kết luận:** Claim an toàn là **“planning/scoping và execution được tách pha”**, không phải **“planning luôn là GPT-4o”**

**Speaker note:** Đây là thay đổi quan trọng nhất để deck không over-claim và vẫn rất thuyết phục về mặt kỹ thuật.

---

## SLIDE 8: Bằng chứng kiến trúc 2 pha

**Tiêu đề:** VS Code: Local scope -> Handoff -> Cloud execute

- **Sự thật [S12]:** VS Code định nghĩa rõ nhiều loại agent: **Local**, **Copilot CLI**, **Cloud**
- **Sự thật [S12]:** **Plan** là built-in agent tạo kế hoạch trước khi code, rồi handoff sang implementation agent
- **Sự thật [S9]:** Với GitHub Pull Requests extension, Copilot sẽ **research/scoping locally in your IDE**, sau đó hỏi xác nhận trước khi chuyển sang background execution
- **Sự thật [S9][S4]:** Sau khi user bấm Continue, cloud agent mở draft PR và chạy trong môi trường **GitHub Actions-powered**
- **Sự thật [S7]:** Kết quả có thể theo dõi qua **View Session** và session logs

**Speaker note:** Nếu mục tiêu là “chứng minh bằng docs”, đây là slide mạnh nhất của cả deck.

---

## SLIDE 9: Chúng ta chứng minh được gì về model?

**Tiêu đề:** Fact, inference, và điều chưa thể khẳng định

- **Chứng minh được [S11]:** Local planning và implementation trong VS Code có thể dùng **setting model riêng** (`chat.planAgent.defaultModel`, `github.copilot.chat.implementAgent.model`)
- **Chứng minh được [S5]:** Danh sách model hiện tại cho **Copilot cloud agent** là: **Auto, Claude Sonnet 4.5, Claude Opus 4.5, Claude Opus 4.6, GPT-5.2-Codex**
- **Suy luận mạnh [S5][S6]:** Luồng delegate từ **VS Code** không được docs mô tả có model picker; GitHub nói nơi không có picker thì **Auto** được dùng tự động
- **Không chứng minh được:** “Planning trong VS Code luôn dùng GPT-4o”
- **Phản chứng quan trọng [S10]:** GPT-4o đã bị deprecated khỏi **Copilot Chat** từ **2025-08-06**; changelog chỉ nói GPT-4o vẫn còn cho **code completions**, tức là **một surface khác**

**Speaker note:** Nếu audience bắt bẻ “thế có chắc là GPT-4o planning không?”, câu trả lời nên là: docs hiện tại không cho phép khẳng định điều đó.

---

## SLIDE 10: Protocol demo để thuyết phục trong buổi trình bày

**Tiêu đề:** Demo 5 bước — dễ lặp lại, dễ thuyết phục

- **B1 [S6][S9]:** Trong VS Code, mở Copilot Chat và delegate task bằng GitHub Pull Requests extension
- **B2 [S9]:** Quan sát việc Copilot **scope locally** trước khi xin xác nhận handoff
- **B3 [S7]:** Sau khi PR/session xuất hiện, mở **View Session** để xem logs ngay trong VS Code
- **B4 [S7]:** Trong session logs, cho audience thấy Copilot dùng tool gì, validation ra sao, tiến độ và token usage ở đâu
- **B5 [S13][S15]:** Đối chiếu billing analytics / usage metrics để xem **model usage per chat mode** và **agent-initiated code changes per model**

**Speaker note:** Demo thực nghiệm này mạnh hơn rất nhiều so với suy đoán bằng “văn phong model” hay cảm giác chủ quan.

---

## SLIDE 11: Đừng trộn 4 surface vào một rổ

**Tiêu đề:** Local Agent vs Copilot CLI vs Cloud Agent vs GitHub CLI

- **Local Agent / Plan trong VS Code [S12]:** Chạy tương tác trong editor, hữu ích cho explore/spec/planning
- **Copilot CLI [S8][S12]:** Chạy background **trên máy của bạn**; default model hiện tại là **Claude Sonnet 4.5**; mỗi prompt tính 1 premium request × model
- **Cloud Agent [S4][S6]:** Chạy remote trên **GitHub Actions**, gắn với branch/PR/session logs
- **GitHub CLI (`gh agent-task`) [S6][S7]:** Là **entry point dòng lệnh** để tạo và theo dõi **cloud-agent sessions**; nó **không đồng nghĩa** với Copilot CLI
- **Khuyến nghị:** Khi nói về billing/model, luôn ghi rõ đang nói tới **surface nào**

**Speaker note:** Phần lớn hiểu nhầm nội bộ xuất phát từ việc trộn “Copilot CLI”, “GitHub CLI”, “agent mode trong VS Code” và “cloud agent” thành cùng một thứ.

---

## SLIDE 12: Kết luận & khuyến nghị

**Tiêu đề:** Kết luận chốt deck

- **Kết luận 1:** Muốn tối ưu request, hãy tối ưu **workflow** để Copilot nhận ít prompt hơn nhưng tự làm nhiều bước hơn [S1][S8]
- **Kết luận 2:** Với VS Code + GitHub Pull Requests extension, có đủ bằng chứng để nói **local scoping/planning tách khỏi cloud execution** [S9][S4][S7]
- **Kết luận 3:** Không nên dùng luận điểm “GPT-4o chỉ phân task” làm claim trung tâm; docs hiện tại **không đủ để chốt** và còn có dữ liệu phản hướng [S10][S5]
- **Khuyến nghị:** Chuẩn hóa **prompt template**, **model policy theo surface**, và **monthly analytics review** [S13][S15]

**Speaker note:** Bản deck mạnh nhất không phải bản nói “chắc chắn model X làm planning”, mà là bản chỉ ra đúng đơn vị chi phí và đúng điểm handoff trong kiến trúc.

---

## Q&A / Backup notes

- Nếu bị hỏi “Vậy GPT-4o có còn tồn tại trong Copilot không?” -> Trả lời: **Có trong một số surface/reference và vẫn còn cho code completions theo changelog**, nhưng **không còn là Copilot Chat model** từ 2025-08-06 [S10][S2]
- Nếu bị hỏi “Cloud agent có chắc chạy trên local không?” -> Trả lời: **Không**; docs nói rõ nó chạy trong môi trường **GitHub Actions-powered** [S4][S9]
- Nếu bị hỏi “VS Code delegate có picker model không?” -> Trả lời: docs hiện tại **không mô tả picker cho path đó**; GitHub nói nơi không có picker thì **Auto** được dùng [S5][S6]

---

## PHỤ LỤC: Snapshot multipliers nên dùng để tham chiếu nhanh

> Nguồn: [S2]. Danh sách dưới đây chỉ là snapshot theo docs ngày 2026-04-07. **Luôn re-check trước khi chốt policy nội bộ.**

| Model | Paid plan multiplier | Copilot Free |
|---|---:|---:|
| Claude Haiku 4.5 | 0.33x | 1x |
| Claude Opus 4.5 | 3x | N/A |
| Claude Opus 4.6 | 3x | N/A |
| Claude Opus 4.6 (fast mode) | 30x | N/A |
| Claude Sonnet 4 | 1x | N/A |
| Claude Sonnet 4.5 | 1x | N/A |
| Claude Sonnet 4.6 | 1x | N/A |
| Gemini 3 Flash | 0.33x | N/A |
| GPT-4.1 | 0x | 1x |
| GPT-4o | 0x | 1x |
| GPT-5 mini | 0x | 1x |
| GPT-5.1 | 1x | N/A |
| GPT-5.1-Codex-Mini | 0.33x | N/A |
| GPT-5.2 | 1x | N/A |
| GPT-5.2-Codex | 1x | N/A |
| GPT-5.3-Codex | 1x | N/A |
| GPT-5.4 | 1x | N/A |
| GPT-5.4 mini | 0.33x | N/A |
| Grok Code Fast 1 | 0.25x | 1x |
| Raptor mini | 0x | 1x |

**Speaker note:** Có thể giữ GPT-4o trong appendix để phản ánh đúng bảng multiplier hiện hành, nhưng nên nói rất rõ rằng **GPT-4o không còn là Copilot Chat model** theo changelog; đừng dùng nó làm suy luận cho planning/chat surfaces.