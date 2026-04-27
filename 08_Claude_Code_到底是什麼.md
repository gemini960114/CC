---
title: Claude Code 到底是什麼
section: 快速入門
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/cc1
---

# Claude Code 到底是什麼

快速入門

00

# 概述：Claude Code 到底是什麼

Claude Code 是Anthropic 推出的命令列 AI 程式設計搭檔，它直接執行在終端裡，能理解你的整個專案程式碼，並幫你編寫、修改、除錯程式碼，甚至執行命令和 Git 操作。

![Claude Code 官方介面示意](/images/tutorials/anthropic-claude-code.webp)

## 一張圖先建立整體感覺


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


> 類比一下：如果說 Google AI Studio 更像一個線上原型實驗場，那 Claude Code 更像一個真正駐紮在你電腦專案目錄裡的高階工程搭檔。

## 它最核心的能力是什麼

先不要急著看原始碼細節，先建立一個產品級認知。  
Claude Code 之所以強，最核心的能力可以概括成下面這幾項：

| 能力 | 說明 | 典型問題 |
| --- | --- | --- |
| **理解專案** | 讀取並分析整個程式碼倉庫 | “這個專案整體架構是怎樣的？” |
| **編寫和修改程式碼** | 直接建立、更新、重構檔案 | “給使用者列表頁加一個搜尋功能” |
| **除錯修復** | 定位報錯、修 bug、補驗證 | “這個介面為什麼返回 500？” |
| **執行命令** | 執行測試、構建、安裝依賴、Git | “幫我跑一下測試，看看哪裡掛了” |
| **遵守專案規範** | 結合 `CLAUDE.md` 和專案上下文工作 | “按這個倉庫現有風格改，不要亂來” |

所以它強的地方，並不只是“能生成程式碼”，而是能在真實專案環境裡持續推進任務。

## 它和普通 AI 聊天工具的本質區別

普通 AI 聊天，大多是：

1. 你提問
2. 模型回答
3. 回答結束

而 Claude Code 的核心模式是：

1. 你提出任務
2. 系統收集專案上下文
3. 模型決定要不要呼叫工具
4. 工具執行檔案讀寫、搜尋、Shell、MCP、LSP 等動作
5. 執行結果迴流給模型
6. 模型繼續下一步，直到任務完成

這就是為什麼它給人的感覺不像“問答機器人”，而像一個會動手幹活的工程搭檔。

## 一個典型工作流是什麼樣的

如果把它放進真實開發現場，一次典型任務往往是這樣推進的：


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


這裡最重要的一點是：  
Claude Code 的工作不是“回答完就結束”，而是會沿著讀取、修改、驗證、彙報這條鏈路持續推進。

## 用一句更工程化的話來定義它

如果用更偏架構的語言描述，Claude Code 可以定義為：

> 一個執行在終端中的會話級 Agent Runtime，它以大模型為決策核心，以工具系統為執行介面，以專案上下文為決策基礎。

這個定義裡有三個關鍵詞：

- **會話級**：不是一次問答，而是持續執行的會話
- **工具化**：不是隻生成文字，而是可以採取動作
- **上下文化**：不是面對孤立 prompt，而是面對整個專案環境

如果再說得更直白一點，它其實是把下面三樣東西綁在了一起：

1. 大模型的推理能力
2. 終端裡的工程工具鏈
3. 專案目錄裡的真實上下文

這三樣東西一旦放到一起，能力上限就明顯和普通聊天工具拉開了。

## 從原始碼看，它的核心入口在哪裡

在這份原始碼映象裡，最值得先記住的幾個檔案是：

- `main.tsx`：啟動與裝配入口
- `QueryEngine.ts`：對話與工具呼叫主迴圈
- `Tool.ts`：工具協議定義
- `tools.ts`：工具登錄檔
- `commands.ts`：斜槓命令系統
- `context.ts`：上下文注入
- `state/AppStateStore.ts`：終端 UI 狀態中心

你可以把這幾個檔案理解成 Claude Code 的主骨架。

## 原始碼片段：它一開始就不是“輕量 CLI”

下面這一小段 `main.tsx` 很能說明問題：

```
// These side-effects must run before all other imports:
// 1. profileCheckpoint marks entry before heavy module evaluation begins
// 2. startMdmRawRead fires MDM subprocesses ...
// 3. startKeychainPrefetch fires both macOS keychain reads ...
import { profileCheckpoint, profileReport } from './utils/startupProfiler.js';

profileCheckpoint('main_tsx_entry');
import { startMdmRawRead } from './utils/settings/mdm/rawRead.js';
startMdmRawRead();
import { ensureKeychainPrefetchCompleted, startKeychainPrefetch } from './utils/secureStorage/keychainPrefetch.js';
startKeychainPrefetch();
```

這段程式碼說明了三件事：

- 啟動效能被當成正式工程問題處理
- 一些系統讀取會在最早階段並行預熱
- Claude Code 啟動時就已經在做複雜執行時準備，而不是簡單等使用者輸入

也就是說，Claude Code 不是“命令列開啟後再慢慢載入”，而是從入口開始就在裝配一套執行環境。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它到底在解決什麼問題

Claude Code 解決的不是“寫一段函式”的問題，而是更完整的工程任務問題，比如：

- 在一個真實倉庫裡理解現有程式碼
- 修改多個檔案並保持風格一致
- 執行命令驗證結果
- 根據 Git 狀態和專案約束做決策
- 接入外部工具或資源

也就是說，它把“大模型”推進到了“工程工作流”內部，而不是停留在聊天框外面。

## `CLAUDE.md` 為什麼很重要

理解 Claude Code，不能漏掉 `CLAUDE.md`。  
你可以把它看成專案級記憶檔案，用來告訴 Claude Code：

- 這個專案是做什麼的
- 程式碼應該遵循什麼規範
- 常用命令有哪些
- 哪些目錄和檔案比較關鍵
- 哪些坑要避免

例如它可能長這樣：

```
# 專案說明

- 這是一個 Next.js + TypeScript 專案
- 樣式使用 Tailwind CSS
- 提交資訊使用中文

# 開發約定

- 優先複用現有元件
- 不要隨意引入新依賴
- 修改後先執行 build 驗證
```

Claude Code 每輪工作時都會參考這類專案記憶，所以它更容易保持長期一致性，而不是每次都像“第一次來到這個專案”。

## 它適合什麼場景

最適合 Claude Code 的，通常是這些場景：

- 中大型專案開發
- 跨檔案修改和重構
- 帶構建、測試、命令執行的工程任務
- 需要長期迭代的本地專案

不那麼適合它的場景則包括：

- 純聊天問答
- 完全沒有本地專案上下文的開放式探索
- 只想要簡單程式碼補全的場景

所以它和編輯器補全工具不是同一個定位。

## Claude Code 和其他 AI 程式設計工具的區別

如果粗略比較一下：

| 維度 | Claude Code | Google AI Studio | 編輯器內補全工具 |
| --- | --- | --- | --- |
| **宿主環境** | 終端 / 本地專案 | 瀏覽器 | 編輯器 |
| **上下文深度** | 整個倉庫 + 命令列環境 | 主要是當前原型或片段 | 多數偏當前檔案附近 |
| **可執行能力** | 強，可讀寫檔案並執行命令 | 較弱，偏原型與實驗 | 中等，通常偏編輯內操作 |
| **適合階段** | 正式開發與持續迭代 | 原型驗證、快速試驗 | 編碼補全、區域性協助 |

所以非常常見的一條路徑是：

1. 先在 Google AI Studio 裡快速驗證想法
2. 再把程式碼放到本地專案
3. 用 Claude Code 做正式迭代開發

## 為什麼研究它的原始碼有價值

就算你不打算自己做一個 Claude Code，研究它也至少有三層價值：

- 你能更清楚地理解 AI 程式設計工具為什麼強，不再把能力歸因於“模型突然變聰明瞭”
- 你能建立 Agent 系統的基本認知，知道真正關鍵的是主迴圈、工具、許可權、上下文
- 你能借鑑它的架構思想，去設計自己的命令列智慧體、開發助手或自動化系統

同時，這篇是專題裡的“入口文章”，重點是先幫你建立整體感覺。  
後面文章才會逐步進入：

- `main.tsx` 如何啟動
- `QueryEngine` 如何推進任務
- 工具和許可權是怎麼協作的
- 為什麼它能接入 MCP、LSP、外掛和多 Agent

## 這一專題會怎麼展開

這個專題不會停留在“怎麼裝、怎麼用”的層面，而是按照下面的路線逐步拆解：

1. 先建立整體認知
2. 再看高層架構
3. 然後進入主迴圈、工具、命令、上下文、狀態、許可權
4. 最後再看 MCP、LSP、外掛、Skills、子 Agent 這些擴充套件能力

如果你第一次接觸 Claude Code，建議直接按目錄順序閱讀。

## 閱讀這個專題時的推薦順序

不同讀者的推薦順序不一樣：

### 如果你是普通讀者

1. 先看這篇概述
2. 再看“為什麼更像工程搭檔”
3. 再看“原始碼架構總覽”
4. 之後按興趣進入 QueryEngine、工具、許可權

### 如果你是進階開發者

1. 先看“原始碼架構總覽”
2. 再看 “main.tsx 啟動流程”
3. 再看 “QueryEngine 主迴圈”
4. 然後看工具、上下文、許可權和擴充套件系統

## 小結

Claude Code 的本質，可以用一句話概括：

> 它是一個以大模型為決策核心、以工具系統為執行手腳、以終端工作流為宿主環境的工程型智慧體系統。

理解了這一點，後面很多原始碼細節就會更容易看懂。