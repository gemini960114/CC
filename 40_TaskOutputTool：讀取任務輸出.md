---
title: TaskOutputTool：讀取任務輸出
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct21
---

# TaskOutputTool：讀取任務輸出

Tools 工具組

00

# TaskOutputTool：讀取任務輸出

## 它把後臺任務輸出統一成同一種可讀結果

`TaskOutputTool` 用來讀取後臺任務的輸出。  
它最重要的價值是統一：

- shell 任務輸出
- 本地 agent 輸出
- 遠端 agent 輸出

所以主執行緒不需要關心底層任務型別差異，只要關心統一返回結構。

## 關鍵原始碼

輸入定義：

```
const inputSchema = z.strictObject({
  task_id: z.string(),
  block: z.boolean().default(true),
  timeout: z.number().min(0).max(600000).default(30000),
})
```

統一輸出物件：

```
type TaskOutput = {
  task_id: string;
  task_type: TaskType;
  status: string;
  description: string;
  output: string;
}
```

## 它還有一個很有意思的現狀

原始碼 prompt 明寫了：

> DEPRECATED: Prefer using the Read tool on the task's output file path instead.

也就是說，它仍然存在，但 Anthropic 已經在把“讀任務輸出”引導到更通用的 `Read` 工具。

這對學習原始碼很有價值，因為你能看到 Claude Code 執行時能力如何逐步收斂。

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 小結

`TaskOutputTool` 是後臺任務鏈路裡的“統一讀口”，即使它未來可能被更通用的 `Read` 替代。