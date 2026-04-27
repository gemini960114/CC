---
title: ReadMcpResourceTool：讀取 MCP 資源
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct24
---

# ReadMcpResourceTool：讀取 MCP 資源

Tools 工具組

00

# ReadMcpResourceTool：讀取 MCP 資源

## 它是 MCP 世界裡的遠端 `Read`

`ReadMcpResourceTool` 的職責非常明確：  
給定 `server + uri`，把遠端 MCP 資源正文讀回來。

## 關鍵原始碼

輸入定義：

```
export const inputSchema = z.object({
  server: z.string().describe('The MCP server name'),
  uri: z.string().describe('The resource URI to read'),
})
```

實際讀取走 MCP SDK：

```
const result = await connectedClient.client.request(
  {
    method: 'resources/read',
    params: { uri },
  },
  ReadResourceResultSchema,
)
```

## 二進位制資源也能處理

原始碼裡有一段很關鍵：

```
if (!('blob' in c) || typeof c.blob !== 'string') { ... }
const persisted = await persistBinaryContent(...)
```

這說明它不僅能讀文字資源，也能讀二進位制 blob，並把內容儲存到磁碟路徑再返回。

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 小結

`ReadMcpResourceTool` 把外部上下文讀入主迴圈，是 MCP 整合真正落地的一環。