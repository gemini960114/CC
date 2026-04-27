---
title: FileReadTool：讀取檔案
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct3
---

# FileReadTool：讀取檔案

Tools 工具組

00

# FileReadTool：讀取檔案

## 它為什麼比 `cat` 更重要

`FileReadTool` 表面看只是“讀檔案”，但在 Claude Code 裡，它其實承擔了三層職責：

1. 給模型穩定讀取專案檔案的入口
2. 讓讀取結果結構化、可追蹤
3. 為後續編輯建立“已讀狀態”

第三點最容易被忽視。  
Claude Code 不是隨便改檔案的，它很強調：

> 先讀，再改

而 `FileReadTool` 就是這個鏈路的起點。

## 先看它的 prompt 怎麼定義自己

`tools/FileReadTool/prompt.ts`：

```
export const DESCRIPTION = 'Read a file from the local filesystem.'

return `Reads a file from the local filesystem.
- The file_path parameter must be an absolute path
- By default, it reads up to 2000 lines
- This tool can only read files, not directories`
```

這裡只是幾條規則，但背後是非常明確的產品設計：

- 路徑必須可確定
- 長檔案預設有上限
- 檔案和目錄分開處理

這避免了模型隨便讀、亂猜路徑或把“目錄瀏覽”和“檔案內容讀取”混在一起。

## 它不只讀文字，還讀多模態內容

同一個 prompt 檔案裡還有兩條很關鍵的說明：

```
- This tool allows Claude Code to read images
- This tool can read PDF files (.pdf)
- This tool can read Jupyter notebooks (.ipynb files)
```

這說明 `Read` 在 Claude Code 裡並不是“文字版 cat”，而是統一的**多模態讀取入口**。

也正因為如此，Claude Code 才會在 prompt 裡強調：

> 使用者給你截圖路徑時，必須用 `Read` 去看

## 一張圖看讀取鏈路


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它為什麼會影響後續寫入

雖然 `FileReadTool` 自己只是讀，但 Claude Code 的寫入工具會檢查：

- 這個檔案之前有沒有讀過
- 讀的時候的時間戳是什麼
- 檔案是不是後來又變了

所以 `Read` 的真實角色還包括：

> 給後續 `Edit` / `Write` 提供可信的“已讀快照”

這和很多粗糙 Agent 最大的不同是：  
Claude Code 明確把“讀過什麼、什麼時候讀的”納入執行時狀態。

## 預設讀 2000 行這件事很有意思

原始碼裡寫死了：

```
export const MAX_LINES_TO_READ = 2000
```

這不是隨便拍腦袋的限制。  
它體現的是 Claude Code 一貫的思路：

- 讓模型拿到足夠資訊
- 但又不要一口氣把超大檔案全塞進上下文

所以它同時提供了：

- 預設全檔案讀取
- 偏移量 `offset`
- 限制 `limit`

讓模型能從“粗讀”逐步走向“區域性精讀”。

## 一次典型使用路徑

比如 Claude Code 要改一個報錯：

1. 先用 `GrepTool` 命中可疑位置
2. 再用 `FileReadTool` 讀取目標檔案
3. 看函式、上下文、相鄰邏輯
4. 決定是否交給 `FileEditTool`

所以 `Read` 在執行時裡常常承擔“從搜尋到理解”的橋樑作用。

## 它和其他工具怎麼配合


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


- `GlobTool`：先找檔名，再讀檔案
- `GrepTool`：先搜關鍵詞，再讀區域性
- `LSPTool`：先做語義定位，再讀正文
- `FileEditTool`：讀完再改
- `BashTool`：改完再驗證

## 最容易誤解它的地方

### 誤解一：Read 只是為了方便展示

不對。  
它還參與了後續編輯合法性的判斷。

### 誤解二：既然有 Bash，就沒必要有 Read

恰恰相反。  
Claude Code 很明確地希望：

- 結構化讀取走 `Read`
- Shell 命令留給真正需要 shell 的場景

### 誤解三：Read 只適合原始碼檔案

不是。  
它還承擔圖片、PDF、Notebook 的多模態讀取。

## 小結

`FileReadTool` 的真正價值不是“能讀檔案”，而是：

> 它把 Claude Code 的檔案理解過程變成了結構化、可追蹤、可參與後續寫入校驗的正式執行時能力。

在整個工具鏈裡，它就是最重要的觀察入口。