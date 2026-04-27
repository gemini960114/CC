---
title: FileWriteTool：寫入檔案
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct5
---

# FileWriteTool：寫入檔案

Tools 工具組

00

# FileWriteTool：寫入檔案

## 它解決的是“整檔案寫入”問題

`FileWriteTool` 適合兩種典型場景：

- 新建一個檔案
- 用一整塊新內容覆蓋一個現有檔案

如果說 `FileEditTool` 更像外科手術，  
那 `FileWriteTool` 更像“整塊材料重新鋪設”。

Claude Code 明確把這兩件事拆成了不同工具，而不是統統交給 shell。  
這就是它工程設計上的一個細節點。

## 原始碼裡最關鍵的輸入定義

`tools/FileWriteTool/FileWriteTool.ts`：

```
const inputSchema = z.strictObject({
  file_path: z.string().describe('The absolute path to the file to write'),
  content: z.string().describe('The content to write to the file'),
})
```

它的輸入非常乾淨：

- 寫到哪裡
- 寫什麼內容

這說明 `Write` 的職責就是**完整寫入**，而不是做複雜的區域性替換邏輯。

## 它的輸出比你想的更豐富

`outputSchema` 裡不僅返回最終內容，還返回：

```
type: z.enum(['create', 'update'])
structuredPatch: z.array(hunkSchema())
originalFile: z.string().nullable()
gitDiff: gitDiffSchema().optional()
```

也就是說，`Write` 並不是“靜默覆蓋然後就沒了”。  
它依然會保留：

- 這是建立還是更新
- 原檔案是什麼
- 差異補丁是什麼
- Git diff 視角下變化如何

這很關鍵，因為它讓整檔案寫入依然處於 Claude Code 的可審計鏈路裡。

## 一張圖看寫入鏈路


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它也會檢查檔案是否過期

和 `Edit` 一樣，`Write` 也不是無腦覆蓋。  
原始碼裡有一段很重要的校驗邏輯：

```
if (!readTimestamp || readTimestamp.isPartialView) {
  return {
    result: false,
    message: 'File has not been read yet. Read it first before writing to it.',
  }
}
```

這說明對於已存在檔案，Claude Code 很強調：

- 先讀
- 再寫

這能防止基於過期內容覆蓋別人剛改過的檔案。

## 它為什麼還要接入 diff 和 Git

原始碼裡可以看到：

```
import { countLinesChanged, getPatchForDisplay } from '../../utils/diff.js'
import { fetchSingleFileGitDiff } from '../../utils/gitDiff.js'
```

這說明即使是整檔案寫入，Claude Code 也不把它當作“黑盒操作”，而是會繼續：

- 統計改動行數
- 生成可展示 patch
- 關聯 Git 視角的變化

這樣前端和後續模型都能更清楚地理解這次寫入發生了什麼。

## 它和 `FileEditTool` 的分工

這個區別值得反覆強調：

- `Edit`：區域性修改已有內容
- `Write`：建立檔案或整檔案重寫

如果模型只是想改一個函式，`Write` 往往會顯得過重；  
但如果是生成新配置檔案、新元件骨架、新文件，`Write` 就非常合適。

## 一次典型使用路徑


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 最容易誤解它的地方

### 誤解一：Write 只是“更方便的 echo > file”

不是。  
它接入了許可權、檔案時序、diff 和 Git 檢視。

### 誤解二：Write 比 Edit 更強，所以更應該優先用

也不對。  
Claude Code 更強調職責清晰，而不是“越萬能越好”。

### 誤解三：Write 只適合新檔案

不完全對。  
它也能更新已有檔案，但更適合整塊重寫，而不是區域性小修。

## 小結

`FileWriteTool` 真正值得學的地方是：

> Claude Code 連“整檔案寫入”都沒有降級成簡單 shell 重定向，而是仍然納入了受控、可審計、可 diff 的工具系統。

這也是 Claude Code 整體設計非常“工程化”的地方。