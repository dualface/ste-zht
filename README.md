# ste-zht：簡化技術中文規範（繁體）

一個面向 AI Agent 的任務彙報規範 skill。借鑑航空與工業界嚴謹的 **ASD-STE100**（Simplified Technical English，簡化技術英語）原則，約束 Agent 用清晰、嚴密且無歧義的繁體中文彙報工作結果。

> 繁體中文版。簡體中文請用姊妹專案 [ste-zh](https://github.com/dualface/ste-zh)。本倉為整倉文法與術語改寫，不是簡繁機翻。


Agent 的日常彙報常常過程冗長、推測含糊，或者漏掉關鍵的驗證證據。開啟 `ste-zht` 後，Agent 將遵循受控繁體中文規範進行彙報：**結論先行、實證說話、狀態固定、一目瞭然**。

> 💡 本 skill 僅規範 Agent 向使用者彙報與討論時的表達方式，絕不影響專案程式碼、註釋、Commit 資訊及生成的檔案內容。

---

## 為什麼需要 ste-zht？

同一個 Bug 修復任務，常見彙報與規範化彙報的差異如下：

**常見 AI 彙報（冗長、推測、不報實證）：**

> 「好的，我先看了一下登入相關的程式碼，發現問題應該是出在 token 刷新的邏輯上，大概是併發的時候兩個請求同時去刷新導致的。然後我對 `auth/refresh.ts` 進行了修改，加了一個鎖來實現序列化處理，另外順手把一些日誌之類的也最佳化了一下。測試基本都跑過了，應該沒什麼問題，不過 iOS 那邊因為沒有環境暫時沒法驗證。你看要不要我再把重試邏輯也一起改了？」

**`ste-zht` 規範化彙報（結論先行、實證可查、便於決策）：**

> **已完成**：修復登入後偶發退出的問題。`npm test auth` 已驗證，iOS 端未驗證。
>
> **原因**：兩個請求同時重新整理 token，後完成的重新整理導致先獲取的 token 失效。  
> **證據**：`auth/refresh.ts:42` — 重新整理前未檢查是否已有進行中的重新整理。
>
> **改動**：
>
> - `auth/refresh.ts`：同一時間僅允許一個重新整理請求，其他請求等待該結果。
> - `auth/logger.ts`：刪除 3 處重複日誌（超出任務範圍）。
>
> **驗證**：
>
> - 已驗證：`npm test auth`（48 項測試全部透過）。
> - 未驗證：iOS 端（本機缺少構建環境）。
>
> **待確認**：  
> 是否修改重試邏輯？
>
> 1. 保持現狀（推薦）
> 2. 本次一同修改

---

## 核心特性

- **結論先行**：回覆首句直奔主題，多工並行時數秒內即可掃視完畢。
- **狀態固定**：僅使用「已完成」「部分完成」「未驗證」「阻塞」等明確狀態詞，杜絕「應該搞定了」這類模稜兩可的說辭。
- **必須標明驗證**：每項改動都寫明是否驗證、怎樣驗證，杜絕虛假彙報。
- **一詞一義與主動語態**：規範專用詞彙，消除虛化動詞與歧義長句，降低閱讀心智負擔。
- **選項化決策**：遇到需要人工定奪的決策點，自動整理為帶推薦傾向的編號選項，使用者輸入數字即可快速推進。

---

## 安裝

### 推薦：使用 `skills` CLI 安裝

[`skills`](https://github.com/vercel-labs/skills) 是 Vercel Labs 推出的 Agent Skill 管理工具，支援 Claude Code、Cursor、Codex 等主流環境。工具會自動按配置將本技能安裝至 `ste-zht` 目錄。

全域性安裝（所有專案通用）：

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

將本倉庫克隆至 Agent 的 skills 目錄下即可。由於本 skill 的標準名稱為 `ste-zht`，克隆時請確保目標目錄命名為 `ste-zht`：

**Claude Code（全域性）：**

```bash
git clone https://github.com/dualface/ste-zht.git ~/.claude/skills/ste-zht
```

其他支援 `SKILL.md` 規範的 Agent，參考各自文件將檔案放置在對應的 skills 路徑下。

---

## 使用方式

在對話中輸入以下任意指令，即可啟用本 skill：

- `/ste-zht`
- 「用繁中 STE 規範輸出」
- 「按 ASD-STE100 彙報」

啟用後，當前會話中的所有彙報都將嚴格按照本規範組織。如需退出，傳送「停止 ste-zht」或「stop ste-zht」即可。

> 當本 skill 與其他輸出風格存在衝突時，優先遵循本 skill。

### 注意事項

- **執行機制**：本 skill 靠 Agent 遵守上下文中的指令生效，沒有程式強制執行。
- **上下文衰減**：在極長會話或發生上下文壓縮（compaction）後，Agent 可能會遺忘指令規範。若發現輸出風格退化，隨時重新傳送 `/ste-zht` 即可恢復。
- **全域性常駐**：如希望每個會話預設開啟，可將「每個會話開始時載入 ste-zht skill」寫入 Agent 的全域性規則中（例如 Claude Code 的 `~/.claude/CLAUDE.md` 或 `~/.claude/rules/` 規則檔案）。

---

## 實戰配合：與 Kander 協同

作者在使用 [Kander](https://github.com/dualface/kander)（多 Agent 看板排程工具）並行排程多個任務時，重度依賴本 skill：

- **高效掃視**：看板上多張任務卡並行流轉，每張卡的彙報第一句就是核心結論，幾秒鐘即可過完所有任務進展。
- **實證門禁**：Kander 嚴格要求任務成果可核驗，配合本 skill 強制區分「已驗證」與「未驗證」，未驗證的結論無法混進「已完成」。
- **極簡決策**：需要人工確認的阻塞點被清晰格式化為編號選項，在 Agent 會話中回覆數字即可快速放行。

---

## 設計背景與 ASD-STE100 的關係

- **關於標準**：[ASD-STE100](https://www.asd-ste100.org/)（Simplified Technical English）由歐洲航空航天與國防工業協會（ASD）維護，原本是針對英文技術維護文件制定的受控語言規範，用於消除歧義與理解偏差。
- **中文改寫**：本專案提煉了 ASD-STE100 的核心原則，並結合 AI 互動特點重構為一套繁體中文實用規則體系。條目編號與內容均為獨立設計，並非原文直譯。
- **獨立宣告**：本專案為個人開源專案（繁體中文版），未收錄 ASD-STE100 原文與受控詞典，與 ASD 組織無隸屬關係。ASD-STE100 與 Simplified Technical English 的權利屬於 ASD。如需研讀 ASD-STE100 英文原版標準，可在其官網免費申請。

---

## 核心檔案

| 路徑                        | 說明                                                                              |
| --------------------------- | --------------------------------------------------------------------------------- |
| `SKILL.md`                  | skill 入口：生效機制、適用範圍、25 條核心規則、5 種標準化輸出模板及自檢清單       |
| `references/terminology.md` | 術語字典：ASD-STE100 關鍵詞譯法、受控情態詞、虛化動詞替換表、固定狀態詞與標點規範 |
| `examples/result-report.md` | 單任務結果彙報改寫示例（改寫前後對比）                                            |
| `examples/task-summary.md`  | 多工與迭代需求總結改寫示例（改寫前後對比）                                      |

---

## 許可證

基於 MIT 協議開源。詳見 [LICENSE](LICENSE)。

---

## 作者的其他專案

歡迎體驗作者 [dualface](https://github.com/dualface) 的其他專案：

- [Kander](https://github.com/dualface/kander)：規則驅動的多 Agent 看板排程工具，內建獨立稽核與自動化交付門禁。
- [Ullage](https://github.com/dualface/ullage-cli)：本地守護程序與 CLI 工具，檢視 Claude、ChatGPT、Grok、Cursor 等訂閱的用量。
- [QuickTUI](https://quicktui.ai/)：適用於各類編碼 Agent 的手機端完整終端，支援自託管直連，單臺主機免費。
