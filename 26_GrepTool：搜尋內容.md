---
title: GrepTool：搜尋內容
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct7
---

# GrepTool：搜尋內容

Tools 工具組

00

# GrepTool：搜尋內容

## 它是 Claude Code 最常用的“找線索”工具之一

`GrepTool` 負責在檔案內容裡搜尋文字或正則模式。  
在真實使用裡，它常常是 Claude Code 排查問題、理解程式碼和定位實現的第一步。

如果你把 Claude Code 的常見動作拆開看，會發現很多輪對話其實都是：

1. 先 `Grep`
2. 再 `Read`
3. 再判斷要不要 `Edit`

所以 `GrepTool` 本質上是主迴圈裡的“線索發現器”。

## 看它的 schema，就知道它不是簡單字串搜尋

`tools/GrepTool/GrepTool.ts`：

```
const inputSchema = z.strictObject({
  pattern: z.string(),
  path: z.string().optional(),
  glob: z.string().optional(),
  output_mode: z.enum(['content', 'files_with_matches', 'count']).optional(),
  '-B': z.number().optional(),
  '-A': z.number().optional(),
  '-C': z.number().optional(),
  '-n': z.boolean().optional(),
  '-i': z.boolean().optional(),
  type: z.string().optional(),
  head_limit: z.number().optional(),
  offset: z.number().optional(),
  multiline: z.boolean().optional(),
})
```

這說明 `GrepTool` 不只是“搜某個詞”，它還支援：

- 搜檔名過濾
- 搜檔案型別過濾
- 只看命中檔案
- 看內容上下文
- 看匹配計數
- 分頁和截斷
- 多行模式

也就是說，它是一個被結構化封裝過的 `ripgrep` 搜尋器。

## 它的底層就是 ripgrep，但不是裸暴露

原始碼裡最關鍵的匯入是：

```
import { ripGrep } from '../../utils/ripgrep.js'
```

Anthropic 沒有讓模型直接去 Bash 裡跑 `rg`，而是把 `ripgrep` 包裝成了正式工具。  
這樣做的好處很直接：

- 許可權系統能識別它是“內容搜尋”
- 輸出模式更可控
- 上下文結果更容易裁剪
- UI 能做更合理展示

## 一張圖看它的常見鏈路


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它在設計上很重視“結果控量”

原始碼裡有一個非常關鍵的預設值：

```
const DEFAULT_HEAD_LIMIT = 250
```

註釋寫得很直白：  
無上限的 grep 很容易把上下文塞爆。

所以 `GrepTool` 天生就做了兩件事：

- 預設限制結果規模
- 支援 `offset` 分頁繼續看

這很重要，因為 Claude Code 本質上是在和有限上下文做博弈。  
搜尋工具如果不幫它控量，很快就會把對話拖垮。

## 它不只是“返回命中內容”，還區分三種模式

原始碼裡把輸出模式拆成：

- `content`
- `files_with_matches`
- `count`

這三種模式背後對應的是三種完全不同的工作目標：

- `files_with_matches`：我先知道哪些檔案相關
- `content`：我要看匹配周邊上下文
- `count`：我想知道影響範圍或匹配規模

這也是 Claude Code 比“讓 AI 跑 rg 然後自己讀終端輸出”更高階的地方。  
因為它不是隻有一個工具，而是讓工具內部支援不同意圖。

## 它和 `GlobTool` 的邊界

這兩個工具經常放在一起比較：

- `GlobTool`：按路徑/檔名找
- `GrepTool`：按內容找

你可以這麼理解：


```mermaid
flowchart LR
    A[需要查找檔案] --> B[GlobTool]
    B --> C[輸入 glob 模式]
    C --> D[遞迴掃描目錄]
    D --> E[回傳匹配檔案清單]
```


在真實任務裡，`GrepTool` 的使用頻率通常會更高。  
因為大多數時候你知道的是：

- 某個函式名
- 某個 API 路徑
- 某個報錯資訊
- 某個文案

而不是準確檔名。

## 它也是隻讀且可並行的

和 `GlobTool` 一樣，`GrepTool` 也明確宣告自己是：

```
isConcurrencySafe() {
  return true
}

isReadOnly() {
  return true
}
```

這意味著主執行緒可以更大膽地把多個搜尋請求並行發出去。  
這對 Claude Code 的探索效率很有幫助。

## 一次真實使用路徑

比如使用者說：

> 登入成功後，為什麼還會跳回登入頁？

Claude Code 很典型的路徑會是：

1. `GrepTool` 搜 `login`、`redirect`、路由保護邏輯
2. `FileReadTool` 讀取命中的幾個關鍵檔案
3. 再用 `GrepTool` 搜狀態來源
4. 最後定位 root cause

你會發現，`GrepTool` 在這裡不是輔助工具，而是調查主鏈的起點。

## 最容易誤解它的地方

### 誤解一：直接 Bash 跑 `rg` 就夠了

功能上也許能湊合，但系統層面不一樣。  
`GrepTool` 更可控、更節省上下文、更利於後續推理。

### 誤解二：Grep 只是“搜字串”

不對。  
它實際上支援不同輸出模式、上下文範圍、分頁和檔案過濾。

### 誤解三：Grep 只是前期探索工具

也不完全。  
它還經常被用於：

- 確認某次改動影響了哪些地方
- 看一個名字是否還殘留在專案裡
- 確認重構是否改全

## 小結

如果把 Claude Code 的搜尋能力壓縮成一句話：

> `GrepTool` 是主迴圈裡最常用的內容定位工具，它把 `ripgrep` 封裝成了可分頁、可裁剪、可結構化回注的正式搜尋能力。

很多原始碼分析任務，真正開始於 `GrepTool`。