---
title: AppStateStore 狀態管理
section: 深入研究
source: https://www.xuanyuancode.com/learn-claude-code/tutorials/cc9
---

# AppStateStore 狀態管理

深入研究

00

# 狀態管理解析：AppStateStore 裡到底儲存了什麼

## Claude Code 不是一次性指令碼，而是長期執行介面

只要看 `state/AppStateStore.ts`，你就會立刻意識到一件事：  
Claude Code 絕不是一個“執行一下就退出”的普通 CLI。

它內部維護了大量 UI 與會話態資料，這說明它本質上是一個終端應用。

## 這裡儲存的狀態遠比你想象的多

`AppState` 裡能看到很多型別的資訊：

- 當前 settings、model、verbose、狀態列文字
- 工具許可權上下文
- 任務列表與前臺任務
- MCP 客戶端、工具、命令、資源
- 外掛啟用狀態和錯誤
- 通知佇列與 elicitation 佇列
- 遠端連線狀態
- prompt suggestion、thinking、session hooks
- Todo、attribution、file history
- Companion、tmux、browser panel 等介面狀態

這已經不是簡單的“頁面狀態”，而是完整執行時狀態。

## 為什麼終端程式會需要這麼重的狀態

因為 Claude Code 不只是顯示文字，它還要同時協調很多非同步物件：

- 模型響應流
- 工具執行
- 背景任務
- 許可權彈窗
- MCP 連線
- 外掛重新整理
- 遠端橋接
- 通知與提示

如果沒有統一狀態中心，這類系統會非常容易失控。

## `AppStateStore` 的真正作用

可以把它理解為 Claude Code 互動層的匯流排：

- 上層 UI 從這裡讀當前狀態
- 下層工具和服務往這裡寫狀態變化
- 某些跨模組能力透過它共享會話級資訊

也就是說，它不是一個被動資料容器，而是終端應用穩定運轉的基礎設施。

## 這也解釋了 Claude Code 為什麼能做複雜互動

很多高階互動能力，本質上都依賴統一狀態管理：

- 同時存在多個任務和前後臺切換
- 在 REPL 裡顯示不同面板
- 遠端會話連線狀態實時更新
- 外掛安裝狀態可追蹤
- MCP 資源、工具、命令動態變化

這些體驗的背後，都離不開 `AppState` 的組織。

## 小結

`AppStateStore.ts` 傳達出的一個重要訊號是：

> Claude Code 不是“命令列裡的模型呼叫器”，而是一個狀態複雜、互動豐富、長期駐留的終端應用。

理解這一點，很多看似“為什麼這裡這麼重”的設計就都說得通了。