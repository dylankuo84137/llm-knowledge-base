---
question: "llm-wiki.md 描述的概念、Cole Medin 實作、與 llm-knowledge-base 本地實作有何異同？"
asked_at: 2026-04-30
sources:
  - "[[llm-wiki]]"
  - "[[I Built Self-Evolving Claude Code Memory w Karpathy's LLM Knowledge Bases]]"
  - "[[CLAUDE.md]]"
  - "[[architecture]]"
  - "[[workflows]]"
---

## TL;DR

實作在概念文件的三層架構上，新增了 `brainstorming/` 與 `artifacts/` 兩層，並引入 `origin: self vs external` 的二元標記，讓個人經驗與外部研究在概念條目中並排對話。7 個具體指令則把模糊的三個操作（Ingest / Query / Lint）落地成可重複的工作流程。

---

## 結論

### 架構：3 層 → 4 層

| 概念文件 (llm-wiki.md) | 實作 (llm-knowledge-base) |
|---|---|
| Raw sources（原始資料） | `raw/`（同） |
| The Wiki（LLM 維護） | `wiki/`（同） |
| The Schema（CLAUDE.md） | `CLAUDE.md` + `AGENTS.md` |
| _(無)_ | `brainstorming/`（新增層） |
| _(無)_ | `artifacts/`（新增層） |

最大的架構擴充是拆出 `brainstorming/` 和 `artifacts/` 兩個層，解決了概念文件沒有處理的問題：

- `brainstorming/chat/` — 對話探索本身有價值，但不應直接進 wiki（太生、太暫時）
- `brainstorming/health/` — 健康檢查報告有時間序，適合獨立追蹤
- `artifacts/` — 使用者自己寫的作品也是知識來源，應被編譯進 wiki

### 操作：3 個 → 7 個指令

概念文件提出三個操作：Ingest、Query、Lint。實作將它們具體化並擴充：

| 概念文件 | 實作指令 | 說明 |
|---|---|---|
| Ingest | `/compile` | 自動偵測未編譯檔案，分批處理 |
| Query | `/thinking-partner` | 強調「先問再答」的探索模式，非單純查詢 |
| Query | `/write-partner` | 動筆前的素材整理，Query 的特化版 |
| Lint | `/health-check` | 比 Lint 更詳細（雙向連結、孤立摘要、Ghost entries） |
| _(無)_ | `/braindump` | 對話沉澱為素材或文章草稿 |
| _(無)_ | `/init-llm` | 互動式初始化，設定個人資料與語言 |
| _(無)_ | `/convert-to-md` | 格式轉換（EPUB / PDF / Facebook JSON） |

### 關鍵創新：`origin: self vs external`

概念文件假設所有來源都是外部材料。實作引入了 origin 二元標記：

- `origin: external`（文章、書、論文）→ 摘要強調「核心結論 / 關鍵證據 / 疑點」
- `origin: self`（自己寫的文章、筆記）→ 摘要強調「我的主張 / 實踐經驗 / 未解問題」

這個區分滲透整個系統：概念條目的結構因此拆成「我的實踐」vs「外部觀點」兩欄，並有專門的「張力與缺口」段落記錄兩者矛盾。這讓知識庫能追蹤個人經驗與外部研究之間的對話，是概念文件沒有設想到的。

### 索引設計

| 概念文件 | 實作 |
|---|---|
| 單一 `index.md`（來源 + 概念混合） | 拆成 `All-Sources.md` 和 `All-Concepts.md` 兩份 |
| `log.md`（時序紀錄） | `wiki/log.md`（同）+ `brainstorming/health/`（健康報告有時間戳） |

---

## 證據

概念文件原文提及三個操作，描述刻意保持抽象：「The exact directory structure, the schema conventions, the page formats, the tooling — all of that will depend on your domain.」

實作的 `docs/architecture.md` 明確把四層定義為「圖書館 / 百科全書 / 實驗筆記本 / 發表成果」的隱喻，讓分層有更清楚的認知框架。

---

## 不確定性

- 概念文件提到好的查詢答案應「filed back into wiki」，但實作的 `/braindump` 只寫進 `brainstorming/`，沒有直接進 wiki——這個缺口目前靠手動 `/compile` 補，是否需要更自動化的路徑尚未決定
- 概念文件提到 qmd 等 CLI 搜尋工具，實作目前靠 index 檔案 + Grep，規模變大後是否需要引入向量搜尋還沒有答案
- Marp / Dataview 等輸出格式實作未涵蓋，是否需要依領域補充尚未評估
