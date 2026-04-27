---
title: Claude Code 源碼架構總覽
section: 架構總覽
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/cc3
---

# Claude Code 源碼架構總覽

架構總覽

00

# Claude Code 原始碼架構總覽

## 先看整體圖

如果只從原始碼目錄去看，很容易被大量檔案嚇住。  
但從主幹關係看，Claude Code 的架構並不亂，它大致可以抽象成下面這張圖：


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 再看一張更貼近原始碼目錄的分層圖


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 第一層：啟動與裝配

`main.tsx` 的職責非常重，它不像普通 CLI 那樣只是簡單解析引數後執行一個函式。  
它會在啟動階段做很多裝配工作：

- 預熱效能敏感模組
- 載入配置與託管設定
- 初始化認證、遙測、策略限制
- 初始化 MCP、LSP、外掛、Skills
- 彙總命令和工具
- 根據模式啟動 REPL、非互動流程或遠端會話

所以 `main.tsx` 更像一個系統引導器。

### 對應原始碼片段

```
import { getSystemContext, getUserContext } from './context.js';
import { launchRepl } from './replLauncher.js';
import { getTools } from './tools.js';
import { filterCommandsForRemoteMode, getCommands } from './commands.js';
import { initializeLspServerManager } from './services/lsp/manager.js';
import { initBuiltinPlugins } from './plugins/bundled/index.js';
import { initBundledSkills } from './skills/bundled/index.js';
```

這段匯入列表本身就很有資訊量。  
它說明入口層不是隻拉一個 REPL，而是在同時裝配：

- 上下文系統
- 工具系統
- 命令系統
- LSP
- 外掛
- Skills

所以 `main.tsx` 在架構中的真實定位就是“裝配根”。

### 這一層最重要的工程意義

啟動層的價值不是“把東西 import 進來”，而是統一決定：

- 當前 session 是什麼形態
- 哪些能力要啟用
- 哪些狀態要預裝
- 哪些資源要在進入主迴圈前準備好

## 第二層：QueryEngine 主迴圈

`QueryEngine.ts` 是 Claude Code 的心臟。  
它負責把一次使用者任務轉化為連續推進的執行過程。

它管理的核心物件包括：

- 訊息歷史
- 工具可用性
- 許可權拒絕記錄
- 檔案快取
- token 與成本統計
- 中斷控制
- 會話級狀態延續

如果沒有這一層，Claude Code 就會退化成“帶一些工具描述的大模型呼叫器”。

### 對應原始碼片段

```
export class QueryEngine {
  private config: QueryEngineConfig
  private mutableMessages: Message[]
  private abortController: AbortController
  private permissionDenials: SDKPermissionDenial[]
  private totalUsage: NonNullableUsage
  private readFileState: FileStateCache

  constructor(config: QueryEngineConfig) {
    this.config = config
    this.mutableMessages = config.initialMessages ?? []
    this.abortController = config.abortController ?? createAbortController()
    this.permissionDenials = []
    this.readFileState = config.readFileCache
    this.totalUsage = EMPTY_USAGE
  }
}
```

只看這些欄位就能知道，`QueryEngine` 管的不只是“發請求給模型”，還包括：

- 歷史訊息
- 中斷控制
- 許可權拒絕
- 檔案快取
- usage 統計

這就是典型的會話級執行時，而不是一次性請求處理器。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 第三層：工具系統

`Tool.ts` 定義工具協議，`tools.ts` 負責註冊和篩選工具。

這層的作用，是把底層能力統一包裝成模型可呼叫的工具介面，例如：

- 讀寫檔案
- Bash / PowerShell
- 搜尋與 glob
- MCP 資源讀取
- LSP 能力呼叫
- AskUserQuestion
- Agent / Team / Task 相關工具

你可以把它理解為 Claude Code 的“行動層”。

### 對應原始碼片段

```
export function getAllBaseTools(): Tools {
  return [
    AgentTool,
    TaskOutputTool,
    BashTool,
    ...(hasEmbeddedSearchTools() ? [] : [GlobTool, GrepTool]),
    FileReadTool,
    FileEditTool,
    FileWriteTool,
    WebFetchTool,
    TodoWriteTool,
    WebSearchTool,
    AskUserQuestionTool,
    SkillTool,
    EnterPlanModeTool,
    ...(isEnvTruthy(process.env.ENABLE_LSP_TOOL) ? [LSPTool] : []),
    ListMcpResourcesTool,
    ReadMcpResourceTool,
  ]
}
```

這段程式碼直接說明 Claude Code 的能力不是抽象想象，而是明確註冊出來的工具集合。

從這裡你能非常直觀地看到：

- 檔案能力
- Shell 能力
- 搜尋能力
- 互動能力
- Skill 能力
- LSP 與 MCP 能力

## 第四層：命令系統

除了模型可呼叫工具，Claude Code 還有大量顯式命令。  
`commands.ts` 聚合了很多斜槓命令，例如：

- 配置類
- 會話類
- 審查類
- 外掛類
- MCP 類
- 計劃類
- 狀態與統計類

命令系統服務的是“使用者顯式控制”，工具系統服務的是“模型隱式執行”，兩者職能不同。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 第五層：上下文與狀態

這套系統之所以能“像懂專案”，關鍵不只是工具，還包括上下文與狀態：

- `context.ts` 負責準備 Git 狀態、`CLAUDE.md`、日期等上下文
- `AppStateStore.ts` 管理 REPL、任務、通知、遠端連線、MCP、外掛等 UI 與會話狀態

一個負責“給模型看什麼”，一個負責“介面和會話當前處於什麼狀態”。

### 對應原始碼片段

```
export const getSystemContext = memoize(async (): Promise<{ [k: string]: string }> => {
  const gitStatus =
    isEnvTruthy(process.env.CLAUDE_CODE_REMOTE) ||
    !shouldIncludeGitInstructions()
      ? null
      : await getGitStatus()

  return {
    ...(gitStatus && { gitStatus }),
  }
})
```

這裡能看出一個關鍵事實：  
Claude Code 會主動把 Git 這樣的工程上下文注入到後續對話裡，這就是它“看起來懂專案”的重要原因之一。

## 第六層：擴充套件能力

從目錄結構能看到，Claude Code 早就不是一個封閉工具，而是平臺化形態：

- `services/mcp/*`
- `services/lsp/*`
- `plugins/*`
- `skills/*`
- `tools/AgentTool/*`
- `remote/*`

這些模組意味著 Claude Code 不只是執行內建工具，而是在持續向“可擴充套件工程智慧體平臺”演化。

## 目錄體量為什麼會這麼大

當你看到這個倉庫目錄很多時，先不要急著認為它“設計很亂”。  
更合理的理解是：Claude Code 同時承擔了四類系統職責：

- 終端互動應用
- Agent 執行時
- 工具與命令平臺
- 外部擴充套件整合層

把這四類職責疊起來，目錄自然不會小。

## 架構主線與次要分支怎麼區分

閱讀時，建議把目錄分成兩類：

### 主幹檔案

- `main.tsx`
- `QueryEngine.ts`
- `Tool.ts`
- `tools.ts`
- `commands.ts`
- `context.ts`
- `state/AppStateStore.ts`

### 延展檔案

- `services/*`
- `components/*`
- `commands/*`
- `tools/*`
- `plugins/*`
- `skills/*`
- `remote/*`

先抓住主幹，再看延展，效率會高很多。

## 閱讀原始碼時的正確順序

如果你直接從海量目錄亂翻，很容易迷失。  
更推薦的順序是：

1. `main.tsx`
2. `QueryEngine.ts`
3. `Tool.ts`
4. `tools.ts`
5. `commands.ts`
6. `context.ts`
7. `state/AppStateStore.ts`
8. 再進入 `services/mcp`、`services/lsp`、`plugins`、`skills`

這條線更接近系統真正的骨架。

## 小結

Claude Code 的原始碼架構可以概括成一句話：

> 用 `main.tsx` 把配置、命令、工具、上下文和擴充套件能力裝配起來，再由 `QueryEngine` 驅動整個工程任務迴圈。

後面幾篇文章，我們會沿著這條主幹逐步往裡拆。