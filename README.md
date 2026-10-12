# ste-zht：簡化技術中文規範（繁體）

**語言 / Language：** [简体中文](https://github.com/dualface/ste-zh) | 繁體中文

一個面向 AI Agent 的任務回報規範 skill。借鑑航空與工業界嚴謹的 **ASD-STE100**（Simplified Technical English，簡化技術英語）原則，約束 Agent 用清晰、嚴密且無歧義的繁體中文回報工作結果。

Agent 的日常回報常常過程冗長、推測含糊，或者漏掉關鍵的驗證證據。開啟 `ste-zht` 後，Agent 將遵循受控繁體中文規範進行回報：**關鍵資訊先行、實證說話、狀態固定、一目瞭然**。

> 💡 本 skill 僅規範 Agent 向使用者回報與討論時的表達方式，絕不影響專案程式碼、註解、Commit 資訊及生成的檔案內容。


> 繁體中文版。簡體中文請用姊妹專案 [ste-zh](https://github.com/dualface/ste-zh)。用語依貢獻者 [laihenyi](https://github.com/laihenyi) 在 ste-zh#1 整理的台灣繁體術語表；本倉為獨立 skill，不是簡繁機翻。

---

## 為什麼需要 ste-zht？

同一個 Bug 修復任務，常見回報與規範化回報的差異如下。以下摘錄以 [結果回報範例](examples/result-report.md) 為準，範例中的來源事實見該檔案。

**常見 AI 回報（冗長、推測、不報實證）：**

> 「好的，我先看了一下登入相關的程式碼，發現問題應該是出在 token 更新的邏輯上，大概是併發的時候兩個請求同時去更新導致的。然後我對 `auth/refresh.ts` 進行了修改，改為同一時間只發一個更新請求，其他請求等待這次更新的結果，另外順手把一些日誌之類的也最佳化了一下。測試基本都跑過了，應該沒什麼問題，不過 iOS 那邊因為沒有環境暫時沒法驗證。你看要不要我再把重試邏輯也一起改了？」

**`ste-zht` 規範化回報（結論先行、實證可查、便於決策）：**

> 已完成：登入更新邏輯的程式碼修改。iOS 端未驗證。
>
> **原因**：兩個請求同時更新 token。後完成的更新使先拿到的 token 失效。
>
> **證據**：
>
> - `auth/refresh.ts:42` — 更新前不檢查是否已有更新在進行。
>
> **改動**
>
> - `auth/refresh.ts`：同一時間只發一個更新請求。其他請求等待這次更新的結果。
> - `auth/logger.ts`：刪除 3 處重複日誌。這項改動不在原範圍內。
>
> **驗證**
>
> - 已驗證：`npm test auth` — 48 項通過，0 項失敗。包含新增併發更新測試 `auth/refresh.test.ts:88`。
> - 未驗證：iOS 端 — 本機沒有 iOS 建置環境。
>
> **待確認**
>
> - 是否保留 `auth/logger.ts` 的改動：
>   1. 撤回（推薦）— 本次提交只含登入修復。
>   2. 保留 — 日誌清理和登入修復一起提交。

---

## 核心特性

- **關鍵資訊先行**：首句按回覆類型寫。結果寫狀態與完成範圍，進度寫當前動作，需求總結寫總結範圍。普通解釋或建議不強套模板。
- **狀態固定**：工作狀態與驗證狀態分別表述，且各自限定範圍。「已完成」不自動表示「已驗證」。完整定義見 [術語表第 4 節](references/terminology.md#4-狀態詞)。
- **如實寫驗證**：修復效果、運作行為和檢查結果寫明檢查方法、結果與涵蓋範圍。進度、建議、需求摘要和待決事項不強加驗證狀態；其中的實際效果斷言仍必須寫驗證。資料內容證據不能代替運作驗證。
- **一詞一義與主動語態**：規範專用詞彙，消除虛化動詞與歧義長句，降低閱讀心智負擔。
- **選項化決策**：遇到需要人工定奪的決策點，自動整理為帶推薦傾向的編號選項，使用者輸入數字即可快速推進。

---

## 安裝

### 推薦：使用 `skills` CLI 安裝

[`skills`](https://github.com/vercel-labs/skills) 是 Vercel Labs 推出的 Agent Skill 管理工具，支援 Claude Code、Cursor、Codex 等主流環境。工具會自動按設定將本技能安裝至 `ste-zht` 目錄。

全域安裝（所有專案通用）：

```bash
npx skills add dualface/ste-zht -g
```

安裝到當前專案：

```bash
npx skills add dualface/ste-zht
```

僅為指定 Agent 安裝（例如 Claude Code）：

```bash
npx skills add dualface/ste-zht -g -a claude-code
```

更新與解除安裝：

```bash
npx skills update ste-zht -g
npx skills remove --global ste-zht
```

### 手動安裝

將本儲存庫複製至 Agent 的 skills 目錄下即可。由於本 skill 的標準名稱為 `ste-zht`，複製時請確保目標目錄命名為 `ste-zht`：

**Claude Code（全域）：**

```bash
git clone https://github.com/dualface/ste-zht.git ~/.claude/skills/ste-zht
```

其他支援 `SKILL.md` 規範的 Agent，參考各自文件將檔案放置在對應的 skills 路徑下。

---

## 使用方式

在對話中輸入以下任意指令，即可啟用本 skill：

- `/ste-zht`
- 「用繁中 STE 規範輸出」
- 「按 ASD-STE100 用繁體中文回報」

啟用後，當前工作階段中的所有回報都將嚴格按照本規範組織。如需退出，傳送「停止 ste-zht」或「stop ste-zht」即可。

> 更高優先級指令和使用者明確指定的輸出格式優先。同級風格規則衝突且使用者未明確指定風格時，按本 skill 寫。

### 注意事項

- **執行機制**：本 skill 靠 Agent 遵守上下文中的指令生效，沒有程式強制執行。
- **上下文衰減**：在極長工作階段或發生上下文壓縮（compaction）後，Agent 可能會遺忘指令規範。若發現輸出風格退化，隨時重新傳送 `/ste-zht` 即可恢復。
- **全域常駐**：如希望每個工作階段預設開啟，可將「每個工作階段開始時載入 ste-zht skill」寫入 Agent 的全域規則中（例如 Claude Code 的 `~/.claude/CLAUDE.md` 或 `~/.claude/rules/` 規則檔案）。

---

## 實戰配合：與 Kander 協同

作者在使用 [Kander](https://github.com/dualface/kander)（多 Agent 看板排程工具）並行排程多個任務時，重度依賴本 skill：

- **高效掃視**：看板上多張任務卡並行流轉，每張卡的回報首句給出當前關鍵資訊，便於查看任務進展。
- **完成與驗證分開**：程式碼修改執行完畢、運作檢查未執行時，可以寫「已完成：程式碼修改。運作效果未驗證。」驗收條件來自使用者要求、適用的專案要求和任務目標。必需檢查未執行或失敗時，不得宣稱整項任務已完成。已知失敗不能改寫成「已完成，未驗證」。
- **極簡決策**：需要人工確認的阻塞點被清晰格式化為編號選項，在 Agent 工作階段中回覆數字即可快速放行。

---

## 設計背景與 ASD-STE100 的關係

- **關於標準**：[ASD-STE100](https://www.asd-ste100.org/)（Simplified Technical English）由歐洲航太、安全與國防工業協會（ASD）維護，原本是針對英文技術維護文件制定的受控語言規範，用於消除歧義與理解偏差。
- **中文改寫**：本專案提煉了 ASD-STE100 的核心原則，並結合 AI 互動特點重構為一套繁體中文實用規則體系。條目編號與內容均為獨立設計，並非原文直譯。
- **獨立宣告**：本專案為個人開源專案，未收錄 ASD-STE100 原文與受控詞典，與 ASD 組織無隸屬關係。ASD-STE100 與 Simplified Technical English 的權利屬於 ASD。如需研讀 ASD-STE100 英文原版標準，可在其官網免費申請。

---

## 核心檔案

| 路徑                        | 說明                                                                              |
| --------------------------- | --------------------------------------------------------------------------------- |
| `SKILL.md`                  | skill 入口：生效機制、適用範圍、25 條核心規則、5 種標準化輸出模板及自檢清單       |
| `references/terminology.md` | 術語字典：ASD-STE100 關鍵詞譯法、受控情態詞、虛化動詞替換表、固定狀態詞與標點規範 |
| `examples/result-report.md` | 單任務結果回報改寫示例（改寫前後對比）                                            |
| `examples/task-summary.md`  | 多工與迭代需求總結改寫示例（改寫前後對比）                                      |

---

## 授權

基於 MIT 授權開源。詳見 [LICENSE](LICENSE)。

---

## 作者的其他專案

歡迎體驗作者 [dualface](https://github.com/dualface) 的其他專案：

- [Kander](https://github.com/dualface/kander)：規則驅動的多 Agent 看板排程工具，內建獨立稽核與自動化交付門禁。
- [Ullage](https://github.com/dualface/ullage-cli)：本地守護程序與 CLI 工具，檢視 Claude、ChatGPT、Grok、Cursor 等訂閱的用量。
- [QuickTUI](https://quicktui.ai/)：適用於各類編碼 Agent 的手機端完整終端，支援自託管直連，單台主機免費。