---
title: Plan Mode 在架構裡的位置
section: 深入研究
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/cc15
---

# Plan Mode 在架構裡的位置

深入研究

00

# Claude Code 的 Plan Mode 在架構裡處於什麼位置

## Plan Mode 不是一個普通“模式切換按鈕”

很多人會把 Plan Mode 理解成：“先寫計劃，不直接寫程式碼”的產品互動。  
但從原始碼看，它遠不只是一個 UI 狀態，而是貫穿：

- 工具系統
- 許可權系統
- 會話狀態
- 審批流

的一套架構能力。

## 先看它在工具層的存在方式

在 `tools.ts` 裡，Plan Mode 不是神秘隱藏能力，而是明確的工具：

```
export function getAllBaseTools(): Tools {
  return [
    ...
    ExitPlanModeV2Tool,
    ...
    EnterPlanModeTool,
    ...
  ]
}
```

這說明 Plan Mode 的進入和退出，都是系統正式建模過的操作，而不是臨時插進去的布林開關。

## 先看整體關係圖


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## `ExitPlanModeV2Tool` 很能說明它的真正定位

來看關鍵定義：

```
export const ExitPlanModeV2Tool: Tool<InputSchema, Output> = buildTool({
  name: EXIT_PLAN_MODE_V2_TOOL_NAME,
  searchHint: 'present plan for approval and start coding (plan mode only)',
  shouldDefer: true,
  isReadOnly() {
    return false
  },
  requiresUserInteraction() {
    if (isTeammate()) {
      return false
    }
    return true
  },
})
```

這段程式碼非常重要，因為它說明：

- 退出 Plan Mode 是一個正式工具呼叫
- 它預設要求使用者互動
- 在 teammate 場景下行為又不同

也就是說，Plan Mode 並不是一個“提示詞建議”，而是系統級審批節點。

## 為什麼它和許可權系統綁得這麼緊

再看它的許可權檢查邏輯：

```
async checkPermissions(input, context) {
  if (isTeammate()) {
    return {
      behavior: 'allow' as const,
      updatedInput: input,
    }
  }

  return {
    behavior: 'ask' as const,
    message: 'Exit plan mode?',
    updatedInput: input,
  }
}
```

這段程式碼說明，Plan Mode 的退出並不是模型自己說了算。  
它是一個需要系統許可權流承認的動作。

## 狀態層也為它預留了語義

在許可權上下文中還能看到 `prePlanMode` 這樣的欄位，說明系統會記住進入 Plan Mode 前的許可權狀態，以便退出後恢復。

這意味著 Plan Mode 不是孤立狀態，而是對整個許可權語義有影響。


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 為什麼 Claude Code 要把 Plan Mode 做到這個程度

因為工程任務裡最危險的，不是“模型不會寫”，而是“模型太快開始寫”。  
Plan Mode 本質上是在系統層引入一個暫停點：

- 先整理方案
- 再給人稽核
- 稽核透過後再進入執行

這對高風險修改、團隊協作、多 Agent 場景都非常重要。

## 它在多 Agent 場景裡更關鍵

原始碼裡還能看到 teammate / approval / mailbox 相關邏輯，這說明：

- 對主執行緒來說，Plan Mode 是使用者審批點
- 對子 Agent 來說，Plan Mode 可能變成團隊領導審批點

也就是說，Plan Mode 是協作工作流的一部分，不只是單使用者互動功能。

## 小結

從架構上說，Plan Mode 更準確的定位是：

> Claude Code 在自動化執行鏈路中人為插入的一個“計劃審批閘門”。

它透過工具、許可權和狀態系統協同工作，確保“從規劃到實現”的切換是可控的。