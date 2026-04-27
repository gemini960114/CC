---
title: ListMcpResourcesTool：列出 MCP 資源
section: Tools 工具組
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/ct23
---

# ListMcpResourcesTool：列出 MCP 資源

Tools 工具組

00

# ListMcpResourcesTool：列出 MCP 資源

## 它讓模型先獲得“資源發現能力”

`ListMcpResourcesTool` 的職責很像遠端世界裡的目錄瀏覽。  
如果模型已經接上了 MCP server，但不知道那邊具體暴露了哪些資源，第一步就應該用它先列出來。

## 關鍵原始碼

`tools/ListMcpResourcesTool/ListMcpResourcesTool.ts`：

```
const inputSchema = z.object({
  server: z.string().optional().describe('Optional server name to filter resources by'),
})
```

核心邏輯是：

```
const results = await Promise.all(
  clientsToProcess.map(async client => {
    const fresh = await ensureConnectedClient(client)
    return await fetchResourcesForClient(fresh)
  }),
)
```

這說明它不是靜態讀快取，而是圍繞 MCP client 連線狀態工作的。

## 呼叫鏈


```mermaid
flowchart LR
    A[開始] --> B[Claude Code 處理]
    B --> C[Tool 執行]
    C --> D[結果輸出]
    D --> E[完成]
```


## 小結

`ListMcpResourcesTool` 解決的是資源發現問題，沒有它，模型只能盲猜 URI。