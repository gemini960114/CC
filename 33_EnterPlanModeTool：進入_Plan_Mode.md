---
title: EnterPlanModeTool：進入 Plan Mode
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct14
---

# EnterPlanModeTool：進入 Plan Mode

Tools 工具組

00

# EnterPlanModeTool：進入 Plan Mode

## 它的本質是狀態切換，不是提示詞切換

`EnterPlanModeTool` 的作用不是“讓模型多想一會兒”，而是把當前會話切到一種新的執行狀態：`plan`。

這意味著系統會同時改變：

- 許可權模式
- 會話目標
- 使用者互動預期

所以它本質上是一個 **執行時狀態切換工具**。

## 關鍵原始碼

`tools/EnterPlanModeTool/EnterPlanModeTool.ts`：

```
import { handlePlanModeTransition } from '../../bootstrap/state.js'
import { applyPermissionUpdate } from '../../utils/permissions/PermissionUpdate.js'
import { prepareContextForPlanMode } from '../../utils/permissions/permissionSetup.js'
```

真正的核心邏輯在 `call()`：

```
context.setAppState(prev => ({
  ...prev,
  toolPermissionContext: applyPermissionUpdate(
    prepareContextForPlanMode(prev.toolPermissionContext),
    { type: 'setMode', mode: 'plan', destination: 'session' },
  ),
}))
```

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 為什麼它不能在 agent context 裡隨便用

原始碼裡有一條硬限制：

```
if (context.agentId) {
  throw new Error('EnterPlanMode tool cannot be used in agent contexts')
}
```

這說明 Anthropic 不希望子 Agent 隨便把自己切進 Plan Mode，  
而是把這種流程控制留給主執行緒。

## 它和 `AskUserQuestionTool` 的關係

Plan Mode 不是閉門寫計劃。  
如果需求沒搞清楚，仍然要用 `AskUserQuestionTool` 先補資訊，再繼續計劃。

## 小結

`EnterPlanModeTool` 的價值在於：

> 它把“先規劃再編碼”從一種提示習慣，升級成了 Claude Code 執行時裡的正式狀態機切換。