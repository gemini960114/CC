---
title: LSPTool：語言服務接入
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct25
---

# LSPTool：語言服務接入

Tools 工具組

00

# LSPTool：語言服務接入

## 它讓 Claude Code 獲得 IDE 級程式碼智慧

`LSPTool` 的意義不在於“又多了一個搜尋工具”，而在於：

> Claude Code 不再只依賴文字搜尋，而是能呼叫語言伺服器做語義級理解。

這讓它可以做：

- go to definition
- find references
- hover
- document symbol
- call hierarchy

也就是說，它已經接近 IDE 裡的“程式碼智慧”層。

## 關鍵原始碼

`tools/LSPTool/LSPTool.ts`：

```
const inputSchema = z.strictObject({
  operation: z.enum([
    'goToDefinition',
    'findReferences',
    'hover',
    'documentSymbol',
    'workspaceSymbol',
    'goToImplementation',
    'prepareCallHierarchy',
    'incomingCalls',
    'outgoingCalls',
  ]),
  filePath: z.string(),
  line: z.number().int().positive(),
  character: z.number().int().positive(),
})
```

這說明 `LSPTool` 並不是一個單一 API，而是一整組程式碼智慧操作的統一入口。

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它為什麼比 `GrepTool` 高一級

兩者的本質區別是：

- `GrepTool`：字串匹配
- `LSPTool`：語言語義

舉例來說，找一個函式定義時：

- `GrepTool` 可能會匹配到註釋、字串、相似名字
- `LSPTool` 能真正問語言伺服器“這個符號定義在哪裡”

所以它更像 IDE 的能力，而不是終端搜尋能力。

## 它也有嚴格的輸入和檔案校驗

原始碼裡不只是看引數格式，還會檢查：

- 檔案是否存在
- 路徑是否是 regular file
- 大檔案限制
- 當前 LSP 是否已連線

這說明它不是“儘量呼叫一下”，而是一個比較嚴謹的語義工具入口。

## 一張圖看它和其他搜尋工具的關係


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 小結

`LSPTool` 最值得學的地方是：

> Claude Code 已經不滿足於文字搜尋，而是把 IDE 裡的語言伺服器能力正式接進了 Agent 工具鏈。