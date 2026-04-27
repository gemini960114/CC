---
title: NotebookEditTool：編輯 Notebook
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct8
---

# NotebookEditTool：編輯 Notebook

Tools 工具組

00

# NotebookEditTool：編輯 Notebook

## 它為什麼不是普通版 `FileEditTool`

`.ipynb` 檔案雖然本質上是 JSON，但語義上它不是普通文字檔案，而是：

- 一組有順序的 cell
- 混合了 code / markdown
- 帶輸出、後設資料、語言資訊

如果直接把 Notebook 當普通文字改，最容易出現兩類問題：

- 結構被破壞，檔案打不開
- 只想改一個 cell，卻誤傷整個 notebook

所以 Claude Code 給它單獨做了 `NotebookEditTool`。

## 關鍵原始碼

`tools/NotebookEditTool/NotebookEditTool.ts`：

```
export const inputSchema = z.strictObject({
  notebook_path: z.string(),
  cell_id: z.string().optional(),
  new_source: z.string(),
  cell_type: z.enum(['code', 'markdown']).optional(),
  edit_mode: z.enum(['replace', 'insert', 'delete']).optional(),
})
```

這幾個欄位說明它的操作粒度是 **cell 級**，不是整檔案字串級。

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 實現重點

它做了幾件普通檔案編輯工具不會做的事：

- 強制要求目標檔案是 `.ipynb`
- 支援 `replace / insert / delete` 三種 cell 操作
- `insert` 時要求明確 `cell_type`
- 校驗 cell 是否存在、ID 是否有效
- 也要求“先讀再改”，和普通檔案編輯鏈路一致

原始碼裡有一條很重要：

```
// Require Read-before-Edit (matches FileEditTool/FileWriteTool).
```

這說明 Anthropic 很強調一致性：  
即使 Notebook 是特殊物件，也不能繞開“讀快照 -> 再編輯”的時序約束。

## 一次典型使用路徑

1. 先用 `Read` 看 notebook 的 cell 結構和內容
2. 定位要改哪個 cell
3. 用 `NotebookEditTool` 做 replace / insert / delete
4. 再回到主執行緒繼續驗證結果

## 它和相鄰工具的關係


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 最容易誤解它的地方

### 誤解一：Notebook 反正是 JSON，直接 Edit 就行

從底層格式看也許可以，但從產品行為看不應該。  
Claude Code 明確把 Notebook 提升成了專門文件型別。

### 誤解二：它只是換了個副檔名

不是。  
它的編輯物件是 cell，不是整段原始 JSON。

## 小結

`NotebookEditTool` 的價值在於：

> Claude Code 沒把 Notebook 降級成普通文字，而是給這種“程式碼 + 文件”混合格式單獨做了一條受控編輯鏈路。