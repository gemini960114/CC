---
title: GlobTool：查找檔案
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct6
---

# GlobTool：查找檔案

Tools 工具組

00

# GlobTool：查詢檔案

## 這個工具看起來簡單，但位置非常關鍵

`GlobTool` 做的事很直接：按檔名或通配模式找檔案。  
但在 Claude Code 的主迴圈裡，它其實承擔的是：

> 把“我大概知道要找什麼檔案”變成“我已經定位到候選路徑”。

很多複雜任務的第一步，都不是直接讀檔案，而是先縮小範圍。  
`GlobTool` 就是這一步的標準入口。

## 先看它的輸入定義

`tools/GlobTool/GlobTool.ts`：

```
const inputSchema = z.strictObject({
  pattern: z.string().describe('The glob pattern to match files against'),
  path: z.string().optional().describe('The directory to search in'),
})
```

這個 schema 很簡單，但設計上很清楚：

- `pattern`：用來表達“想找什麼”
- `path`：用來限制搜尋範圍

也就是說，Claude Code 希望模型不要預設全倉庫亂掃，而是學會縮小搜尋半徑。

## 它是讀操作，而且是併發安全的

原始碼裡這幾個宣告很值得注意：

```
isConcurrencySafe() {
  return true
}

isReadOnly() {
  return true
}

isSearchOrReadCommand() {
  return { isSearch: true, isRead: false }
}
```

這說明系統從一開始就把 `GlobTool` 定義成：

- 只讀
- 可並行
- 明確屬於“搜尋”類工具

這對主迴圈排程和 UI 展示都很重要。

## 一張圖看它在搜尋鏈裡的位置


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它不是簡單的 `find`

`GlobTool` 內部並不是直接把 Bash `find` 暴露給模型，而是走自己的檔案搜尋實現：

```
import { glob } from '../../utils/glob.js'
```

這意味著：

- 搜尋結果結構化
- 許可權系統可感知
- 返回值更適合後續模型處理

這和 Bash 命令最大的不同是：  
系統知道“你剛剛是在做檔案發現”，而不是只看到一串 shell 輸出。

## 它會校驗 path 不是亂填的

`validateInput()` 裡有一段很實際的邏輯：

```
if (path) {
  const absolutePath = expandPath(path)
  ...
  if (!stats.isDirectory()) {
    return {
      result: false,
      message: `Path is not a directory: ${path}`,
    }
  }
}
```

這說明 `GlobTool` 很明確地區分：

- 路徑是目錄
- 路徑不存在
- 路徑寫錯了

甚至還會嘗試給出 cwd 下的建議路徑。  
這類細節很像一個真正面向產品的工具，而不只是 SDK demo。

## 它還會主動限制結果規模

呼叫邏輯裡有一個預設限制：

```
const limit = globLimits?.maxResults ?? 100
```

這說明 Claude Code 很清楚一個問題：

> 檔案搜尋如果不控量，很容易一次返回幾百上千條結果，把上下文浪費掉。

所以 `GlobTool` 的目標不是“搜得越多越好”，而是“給主迴圈足夠用的候選集”。

## 它和 `GrepTool` 的分工

這是最值得記住的一點：

- `GlobTool`：我知道檔案大概叫什麼
- `GrepTool`：我知道檔案裡大概有什麼文字

這兩個工具經常被混著看，但從工程角度上它們代表的是兩種不同搜尋策略。

## 典型使用路徑


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 最容易誤解它的地方

### 誤解一：有 Bash 的 `find`，Glob 就沒必要

不對。  
`GlobTool` 的優勢恰恰在於結構化、可控、對主迴圈更友好。

### 誤解二：Glob 只是 UI 更好看

也不對。  
它會影響許可權判斷、上下文控制和後續工具選擇。

### 誤解三：它只是小工具，不重要

搜尋工具往往看起來簡單，但在 Claude Code 這種系統裡，  
“先找到正確檔案”本身就是一條非常關鍵的主鏈路。

## 小結

`GlobTool` 的價值可以概括成一句話：

> 它把“按檔案路徑模式發現目標檔案”做成了 Claude Code 的標準、只讀、結構化搜尋入口，是很多工真正開始的第一步。