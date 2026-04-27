---
title: Claude Code 啟動流程
section: 架構總覽
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/cc4
---

# Claude Code 啟動流程

架構總覽

00

# 啟動流程解析：main.tsx 到 REPL 是怎麼串起來的

## 為什麼 `main.tsx` 值得重點看

很多專案的入口檔案只是薄薄一層，但 Claude Code 的 `main.tsx` 明顯不是。  
從匯入規模和初始化動作就能看出來，它承擔的是“系統裝配器”的角色。

它做的事情至少包括：

- 啟動早期效能預熱
- 解析命令列引數
- 載入設定、策略和環境變數
- 初始化認證與實驗開關
- 收集命令與工具
- 啟動互動式 REPL 或其他執行模式


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 一開頭就在搶啟動時間

檔案最前面的幾個 side effect 很有代表性：

- `profileCheckpoint`
- `startMdmRawRead()`
- `startKeychainPrefetch()`

這說明 Claude Code 團隊已經把啟動效能當成正式問題來最佳化。  
也就是說，入口檔案不只是“能跑起來”，而是在儘量把一些 I/O 提前並行化。

### 對應原始碼片段

```
profileCheckpoint('main_tsx_entry');
import { startMdmRawRead } from './utils/settings/mdm/rawRead.js';
startMdmRawRead();
import { ensureKeychainPrefetchCompleted, startKeychainPrefetch } from './utils/secureStorage/keychainPrefetch.js';
startKeychainPrefetch();
```

這段程式碼的重點不是語法，而是時機：

- `profileCheckpoint` 先打點
- MDM 讀取儘早啟動
- keychain 預取儘早啟動

這說明入口層是在主動縮短後續等待時間。

## 為什麼入口層會有這麼多 side effect

從一般編碼習慣看，入口層副作用越少越好。  
但 Claude Code 這裡恰恰相反，因為它面對的是一個真實 CLI 產品：

- 啟動時間要快
- 配置必須儘早生效
- 某些系統狀態要在後續匯入前可用

所以這裡不是“寫得隨意”，而是明確在為執行時體驗服務。

## 它先決定“這次啟動是什麼會話”

啟動後要先弄清幾個問題：

- 當前是不是互動模式
- 有沒有遠端會話或橋接模式
- 要不要恢復舊會話
- 模型、許可權、提示風格、工作目錄是什麼

這些資訊會影響後續整個系統裝配結果。  
所以 `main.tsx` 的很多邏輯，實際上是在決定“這次 session 的執行形態”。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 命令、工具、設定是在這裡彙總的

從匯入關係能看出來，`main.tsx` 會把幾類核心資源拉到一起：

- `getCommands()` 獲取命令系統
- `getTools()` 獲取工具集合
- `getSystemContext()` / `getUserContext()` 獲取上下文
- 各種設定與 feature gate 決定哪些能力啟用

也就是說，真正把 Claude Code 拼裝成一個可執行系統的地方，不在某個單獨 service，而就在入口層。

### 對應原始碼片段

```
import { getSystemContext, getUserContext } from './context.js';
import { filterCommandsForRemoteMode, getCommands } from './commands.js';
import { getTools } from './tools.js';
import { launchRepl } from './replLauncher.js';
```

這幾行程式碼幾乎已經把入口層的主線交代清楚了：

- 先拿上下文
- 再拿命令
- 再拿工具
- 最後進入 REPL

也就是說，REPL 看到的不是“裸模型”，而是已經裝配好的執行時。

## 入口層最像什麼

如果一定要類比，`main.tsx` 很像後端系統裡的“應用裝配根”：

- 它不負責所有細節實現
- 但負責決定細節如何被拼起來
- 任何關鍵子系統最終都要在這裡接上線

## 它不是隻啟動 REPL，還會先初始化外部能力

原始碼裡還能看到很多外圍系統在入口層就被拉起：

- MCP 相關初始化
- LSP Server Manager 初始化
- 外掛與技能初始化
- 遠端會話配置
- 遙測與限制策略

原因也很直接：  
這些能力都可能影響當前 session 的工具清單、命令清單和介面狀態，所以必須在進入主迴圈前先準備好。

### 對應原始碼片段

```
import { initializeLspServerManager } from './services/lsp/manager.js';
import { getMcpToolsCommandsAndResources, prefetchAllMcpResources } from './services/mcp/client.js';
import { initBuiltinPlugins } from './plugins/bundled/index.js';
import { initBundledSkills } from './skills/bundled/index.js';
```

這組匯入說明入口層天然就是能力匯流點。  
LSP、MCP、外掛、Skills 這些都不是後面臨時發現，而是在會話建立階段就已經納入考慮。

## REPL 只是表層，真正重要的是“執行態裝配完成”

很多人看到終端介面，直覺會把 Claude Code 當成一個 React Ink 程式。  
這個理解只對一半。

REPL 當然重要，但它更像是表現層。  
真正關鍵的是：在 REPL 出現之前，系統已經把下面這些東西準備好了：

- 會話設定
- 模型選擇
- 工具集合
- 命令集合
- 上下文資料
- MCP / LSP / 外掛狀態
- AppState 初始值

因此 REPL 並不是起點，而是“系統已經裝配完成後的互動外殼”。

## 從使用者視角看，它的啟動鏈路是什麼


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 啟動流程可以粗略理解為 4 步

```
入口預熱 -> 配置與環境解析 -> 能力裝配 -> 進入互動或執行模式
```

如果把它展開一點，就是：

1. 先做效能預熱和必要的早期副作用
2. 解析 CLI 引數、載入 settings、策略和環境
3. 初始化命令、工具、上下文、MCP、LSP、外掛、Skills
4. 根據模式決定啟動 REPL、恢復會話、遠端連線或非互動執行

## 從原始碼閱讀角度，這篇之後該看什麼

理解了啟動層後，最自然的下一篇就是 `QueryEngine.ts`。  
因為入口層解決的是“怎麼啟動”，而 `QueryEngine` 解決的是“啟動之後怎麼持續推進任務”。

## 閱讀這一層時要關注什麼

看 `main.tsx`，不建議陷入每一段細節。  
更應該關注的是：

- 哪些初始化屬於“全域性能力”
- 哪些初始化會影響當前 session
- 哪些能力是透過 feature flag 控制的
- 哪些結果最終流向 REPL 或 QueryEngine

只要抓住這四點，入口層就會變得很清楚。

## 小結

`main.tsx` 的價值，不在於它實現了什麼具體業務，而在於它回答了一個更重要的問題：

> Claude Code 在真正開始和你對話前，到底把哪些能力裝進了這次會話裡？

理解了啟動流程，後面看主迴圈和工具系統就會輕鬆很多。