---
title: 上下文系統：Git、CLAUDE.md 與提示詞
section: 核心機制
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/cc8
---

# 上下文系統：Git、CLAUDE.md 與提示詞

核心機制

00

# 上下文系統解析：Git、CLAUDE.md 與系統提示詞注入

## Claude Code 為什麼看起來“懂專案”

Claude Code 給人的一個強烈印象是：  
它不像在面對一段孤立程式碼，而像是在理解整個專案。

這背後最關鍵的原因之一，就是上下文系統。

`context.ts` 讓系統在對話開始前，就準備好一些高價值專案資訊，再注入到後續主迴圈裡。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## `getSystemContext()`：補充系統級背景

從原始碼看，`getSystemContext()` 至少會處理一類非常重要的資訊：**Git 狀態**。

它會嘗試收集：

- 當前分支
- 主分支
- 工作區狀態
- 最近提交
- Git 使用者資訊

這帶來的好處非常直接：

- 模型知道當前倉庫是不是髒的
- 模型知道你現在在哪條分支上
- 模型知道近期程式碼變化的大致方向

這對工程任務判斷非常重要。

### 對應原始碼片段

```
const [branch, mainBranch, status, log, userName] = await Promise.all([
  getBranch(),
  getDefaultBranch(),
  execFileNoThrow(gitExe(), ['--no-optional-locks', 'status', '--short'], {
    preserveOutputOnError: false,
  }).then(({ stdout }) => stdout.trim()),
  execFileNoThrow(
    gitExe(),
    ['--no-optional-locks', 'log', '--oneline', '-n', '5'],
    {
      preserveOutputOnError: false,
    },
  ).then(({ stdout }) => stdout.trim()),
  execFileNoThrow(gitExe(), ['config', 'user.name'], {
    preserveOutputOnError: false,
  }).then(({ stdout }) => stdout.trim()),
])
```

這裡一眼就能看出，Claude Code 不是隨便問一句“你當前分支是什麼”。  
它會主動並行收集：

- 當前分支
- 主分支
- 工作區狀態
- 最近提交
- Git 使用者名稱

這就是工程上下文。

## 從原始碼意圖上看，它在做“上下文壓縮”

原始工程資訊是極其雜亂的：

- 倉庫裡有很多檔案
- Git 狀態可能很長
- 記憶檔案可能很多

`context.ts` 的工作不是把一切原樣塞給模型，而是提煉出最值得注入的高訊號資訊。

## `getUserContext()`：補充使用者與專案約束

另一個重點是 `getUserContext()`。  
這裡會處理 `CLAUDE.md` 等記憶檔案，並注入當前日期。

`CLAUDE.md` 的意義非常大，你可以把它理解為專案級工作說明書，例如：

- 編碼規範
- 倉庫結構約束
- 特殊命令
- 團隊約定
- 某些需要避免的操作

有了這層記憶，Claude Code 對同一個專案的行為就會更穩定。

### 對應原始碼片段

```
const shouldDisableClaudeMd =
  isEnvTruthy(process.env.CLAUDE_CODE_DISABLE_CLAUDE_MDS) ||
  (isBareMode() && getAdditionalDirectoriesForClaudeMd().length === 0)

const claudeMd = shouldDisableClaudeMd
  ? null
  : getClaudeMds(filterInjectedMemoryFiles(await getMemoryFiles()))

setCachedClaudeMdContent(claudeMd || null)
```

這段程式碼說明 `CLAUDE.md` 並不是“固定總會注入”的。  
Claude Code 會根據模式和設定判斷是否啟用，然後再把結果快取給後續系統使用。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 這不是單純“拼 prompt”，而是上下文治理

表面上看，這像是在往系統提示詞里加文字。  
但從工程角度看，本質更接近 **上下文治理**：

- 什麼資訊值得長期注入
- 什麼資訊要快取
- 什麼資訊在 bare mode 下跳過
- 什麼資訊需要避免迴圈依賴
- 什麼資訊應該控制長度

這些都不是“寫個 prompt”那麼簡單，而是長期執行系統必須面對的問題。

### 對應原始碼片段

```
return {
  ...(gitStatus && { gitStatus }),
  ...(feature('BREAK_CACHE_COMMAND') && injection
    ? {
        cacheBreaker: `[CACHE_BREAKER: ${injection}]`,
      }
    : {}),
}
```

這段返回值看起來簡單，但它很能說明 Claude Code 的思路：  
上下文不是散落在系統裡的臨時字串，而是被統一整理成結構化上下文物件，再交給後續主迴圈使用。

## 為什麼 `CLAUDE.md` 這類檔案特別關鍵

因為它把專案經驗從“靠人臨時口述”變成了“可注入的系統知識”。

這會直接影響三件事：

- 輸出風格更穩定
- 修改更符合倉庫約定
- 不容易反覆犯同樣的錯誤

## Git 狀態為什麼這麼關鍵

很多人低估 Git 狀態的價值。  
實際上，對工程代理來說，Git 狀態是判斷風險和上下文邊界的重要訊號：

- 工作區乾淨還是髒
- 有沒有未提交修改
- 現在在功能分支還是主分支
- 最近改動和當前任務有沒有關係

Claude Code 把這些資訊納入系統上下文，意味著它不是把程式碼當靜態文字看，而是把倉庫當動態工程資產看。

## 這層設計帶來的直接結果

有了上下文系統，Claude Code 在很多場景下會表現得更像“進入專案現場”：

- 它更容易遵守倉庫約定
- 它更容易避免和當前工作區狀態衝突
- 它更容易理解為什麼當前任務要這麼做

這也是為什麼同一個模型，放在普通聊天框裡和放在 Claude Code 裡，體驗會差很多。

## 這層機制也有明顯邊界

上下文注入並不意味著 Claude Code 天然“全知”。  
它仍然會受到以下限制：

- 注入內容長度有限
- Git 狀態是快照，不會自動實時重新整理
- 記憶檔案質量決定了部分行為上限

所以“上下文系統”是增強器，不是萬能鑰匙。

## 小結

`context.ts` 告訴我們一件很重要的事：

> AI 程式設計工具之所以強，往往不是因為模型憑空更聰明，而是因為系統在模型開口前，就已經把“該知道的專案背景”準備好了。

Claude Code 的“懂專案”，很大一部分就來自這裡。