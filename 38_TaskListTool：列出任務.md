---
title: TaskListTool：列出任務
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct19
---

# TaskListTool：列出任務

Tools 工具組

00

# TaskListTool：列出任務

## 它給主執行緒提供全域性任務檢視

`TaskListTool` 的作用是讓模型重新看到“現在有哪些任務、誰卡住了誰、哪些還沒完成”。  
在多步流程中，它相當於任務系統的總覽面板。

## 關鍵原始碼

```
const allTasks = (await listTasks(taskListId)).filter(
  t => !t.metadata?._internal,
)
```

它還會過濾已解決阻塞項，讓輸出更貼近“當前可行動狀態”。

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 小結

`TaskListTool` 讓 Claude Code 在複雜任務流裡具備了“抬頭看全域性”的能力。