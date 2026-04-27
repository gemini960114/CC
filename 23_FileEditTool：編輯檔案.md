---
title: FileEditTool：編輯檔案
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct4
---

# FileEditTool：編輯檔案

Tools 工具組

00

# FileEditTool：編輯檔案

## 這個工具解決的不是“能改檔案”，而是“怎麼安全地改”

`FileEditTool` 負責對已有檔案做定點修改。  
在 Claude Code 裡，它的價值從來不只是“替換一段字串”，而是：

- 先確認檔案讀過
- 再確認檔案沒被別人改過
- 再確認當前編輯許可權允許
- 最後才生成 patch 並寫回

也就是說，`FileEditTool` 體現的是 Claude Code 的**受控編輯模型**。

## 看原始碼，它的依賴明顯比想象中重

`tools/FileEditTool/FileEditTool.ts`：

```
import { countLinesChanged } from '../../utils/diff.js'
import { fetchSingleFileGitDiff } from '../../utils/gitDiff.js'
import { checkWritePermissionForTool } from '../../utils/permissions/filesystem.js'
import { readFileSyncWithMetadata } from '../../utils/fileRead.js'
import { getPatchForEdit } from './utils.js'
```

這些匯入已經說明它不是一個簡單替換器，而是同時關注：

- diff
- Git 視角
- 許可權
- 檔案後設資料
- patch 生成

## 先看工具本體定義

```
export const FileEditTool = buildTool({
  name: FILE_EDIT_TOOL_NAME,
  searchHint: 'modify file contents in place',
  strict: true,
  ...
})
```

這裡的 `strict: true` 很重要。  
它意味著 Anthropic 對這個工具的輸入結構和行為約束更嚴格，不希望模型隨便糊弄引數。

## 一張圖看編輯鏈路


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它最關鍵的一點：要求先讀檔案

`FileEditTool` 在校驗邏輯裡，會檢查這個檔案是否已經讀過。  
這是 Claude Code 特別重要的一條工程約束。

你從它的寫入鄰居 `FileWriteTool` 就能看到類似邏輯，而 `Edit` 這邊更強調區域性修改場景。  
Anthropic 這麼做的目的很明確：

- 避免模型盲改
- 避免基於過期內容改
- 避免使用者和模型同時修改導致衝突

這和“讓 AI 隨便改程式碼再說”的工具思路很不一樣。

## 它不是直接覆蓋，而是 patch 驅動

原始碼裡有幾個很有代表性的工具函式：

```
import { getPatchForEdit } from './utils.js'
import { areFileEditsInputsEquivalent, findActualString } from './utils.js'
```

這說明 `FileEditTool` 更像是在做：

- 找到要改的舊片段
- 生成區域性補丁
- 保留儘可能多的原始結構

而不是：

> 把整個檔案重新生成一遍後覆蓋

這也是為什麼 `Edit` 通常比 `Write` 更適合已有檔案的小中型改動。

## 它還會盯著檔案是否被外部修改過

原始碼裡有一個非常關鍵的錯誤常量：

```
FILE_UNEXPECTEDLY_MODIFIED_ERROR
```

對應的場景就是：

- Claude 讀過檔案
- 使用者或 linter 後來又改了檔案
- Claude 還想基於舊快照繼續編輯

這時系統會阻止它繼續寫。  
這類設計很重要，因為真實專案裡：

- 儲存時自動格式化
- IDE 自動修復
- 你和 Claude 同時改檔案

都非常常見。

## 一次典型使用路徑


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它和 `FileWriteTool` 的區別

很多人會把 `Edit` 和 `Write` 混在一起。  
兩者分工其實非常清楚：

- `FileEditTool`：已有檔案，區域性替換，強調 patch
- `FileWriteTool`：整檔案建立或覆蓋，強調完整內容寫入

所以如果一個檔案已經存在，而且你只想改其中一小塊邏輯，`Edit` 更合理。

## 最容易誤解它的地方

### 誤解一：Edit 和 Bash `sed` 沒本質區別

有很大區別。  
`Edit` 直接接入了許可權系統、檔案讀狀態和 diff 跟蹤。

### 誤解二：Edit 是文字工具，不涉及工程狀態

不對。  
它會聯動：

- Git diff
- LSP 診斷
- 檔案歷史
- 技能目錄啟用

### 誤解三：Edit 只是“更安全的替換”

更準確一點：

> 它是 Claude Code 的受控增量編輯器。

## 它和相鄰工具的關係


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 小結

`FileEditTool` 最值得學的點，不是“怎麼替換字串”，而是：

> Claude Code 如何把檔案編輯做成一個帶前置閱讀、衝突檢測、許可權控制和 patch 結果的工程化操作。

這正是它比“AI 幫你執行 sed 命令”高階的地方。