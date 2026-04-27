---
title: ExitPlanModeTool：退出 Plan Mode
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct15
---

# ExitPlanModeTool：退出 Plan Mode

Tools 工具組

00

# ExitPlanModeTool：退出 Plan Mode

## 它不是“結束規劃”，而是“提交規劃”

`ExitPlanModeTool` 的名字容易讓人誤會。  
它真正做的並不是簡單退出，而是：

> 當計劃已經寫好後，把計劃提交給使用者審批。

所以它代表的是 Plan Mode 的**交付節點**，不是隨便離開規劃態。

## 關鍵原始碼

`tools/ExitPlanModeTool/prompt.ts`：

```
Use this tool when you are in plan mode and have finished writing your plan
to the plan file and are ready for user approval.
```

這個 prompt 已經很明確了：

- 先寫 plan file
- 再呼叫這個工具
- 讓使用者審批

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 它和 `AskUserQuestionTool` 的邊界

這個邊界 Anthropic 寫得很死：

- 不確定需求：`AskUserQuestionTool`
- 計劃寫完求批准：`ExitPlanModeTool`

也就是說：

> 不要用普通提問工具去問“這個計劃可以嗎”

那是 `ExitPlanModeTool` 的職責。

## 它為什麼重要

如果沒有這個工具，Plan Mode 很容易退化成：

- 寫一堆計劃
- 再隨便發句自然語言

有了它之後，系統才能把“計劃已完成、待審批”當成一個正式狀態。

## 小結

`ExitPlanModeTool` 的意義在於：

> 它把規劃階段的收尾和審批流程，變成了 Claude Code 裡一個明確的狀態轉換點。