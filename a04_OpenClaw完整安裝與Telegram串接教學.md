---
title: OpenClaw 完整安裝與 Telegram 串接教學
section: 補充資料
source: custom
author: Elponcrab
date: 2026-04-02
---

# OpenClaw（龍蝦 AI）完整安裝教學 2026：從零架設個人 AI 助理，串接 Telegram

> **作者**：Elponcrab  
> **日期**：2026/4/2

OpenClaw（龍蝦）是目前 GitHub 成長速度最快的 AI Agent 開源專案，讓你在自己的電腦上架設一個永遠在線的私人 AI 助理，並透過 Telegram、WhatsApp、Discord 等你平常就在用的通訊 App 和它對話。

這篇教學從零開始，帶你完成安裝、設定到發送第一則訊息，全程約 10 分鐘。

---

## 目錄
1. [OpenClaw 是什麼？和 ChatGPT 有什麼不同？](#openclaw-是什麼和-chatgpt-有什麼不同)
2. [你需要用到「終端機」，不用怕](#你需要用到終端機不用怕)
3. [安裝前準備](#安裝前準備)
4. [第一步：安裝 OpenClaw](#第一步安裝-openclaw)
5. [第二步：執行 Onboarding 向導](#第二步執行-onboarding-向導)
6. [第三步：確認 Gateway 正在運行](#第三步確認-gateway-正在運行)
7. [第四步：串接 Telegram（最快上手的方式）](#第四步串接-telegram最快上手的方式)
8. [常見問題 FAQ](#常見問題-faq)
9. [接下來可以做什麼](#接下來可以做什麼)

---

## OpenClaw 是什麼？和 ChatGPT 有什麼不同？

ChatGPT 是問答工具，你問它一個問題，它給你一個回答，對話完成後就結束了。OpenClaw 是 **AI Agent**：你交付一個任務，它會自己思考、呼叫工具、執行動作——查你的 Gmail、看你的日曆、搜尋網路、寫程式——然後把結果告訴你。

更關鍵的差異是：**OpenClaw 跑在你自己的電腦上，資料完全由你掌控**，不會上傳到任何第三方伺服器。它本身不包含 AI 模型，需要你自己提供 API Key（Anthropic Claude、OpenAI GPT、Google Gemini 等皆可），模型費用直接由你付給供應商。

## 你需要用到「終端機」，不用怕

這份教學需要在電腦上輸入幾行指令。如果你之前沒用過，先花 30 秒了解怎麼打開它：

- **macOS**：按 `Command + 空白鍵` 開啟 Spotlight 搜尋，輸入「終端機」，按 Enter 開啟。
- **Windows**：按 `Win + R`，輸入 `powershell`，按 Enter；或在開始選單搜尋「PowerShell」。
- **Linux**：在應用程式選單找 Terminal 或 Konsole。

打開後，把本文的指令直接複製貼上，再按 Enter 執行，就這樣。遇到要你輸入密碼的時候，打字時畫面不會顯示，但其實有輸入，打完按 Enter 就好。

---

## 安裝前準備

### 系統需求
- macOS、Linux，或 Windows（建議使用 WSL2，穩定性更高）
- **Node.js 22.14 以上**（推薦 Node 24）
- 至少一組 AI 模型的 API Key

### 安裝 Node.js
Node.js 是 OpenClaw 需要的執行環境，就像 App 需要作業系統才能跑一樣。先確認電腦上有沒有、版本夠不夠：

```bash
node -v
```

若版本低於 `v22.14` 或尚未安裝，依你的電腦系統選擇：

- **macOS 用戶**：最簡單的方式是直接前往 [nodejs.org](https://nodejs.org/) 下載安裝包，點兩下執行即可。若你有安裝 Homebrew（Mac 的套件管理工具），也可以在終端機執行：
  ```bash
  brew install node
  ```
- **Windows 用戶**：同樣可以直接去 [nodejs.org](https://nodejs.org/) 下載 Windows 安裝包，雙擊執行，一路 Next 就好。

安裝完後，重新打開終端機，再輸入一次 `node -v` 確認版本顯示 v22 以上即代表成功。

### 取得 AI 模型 API Key
OpenClaw 支援多種模型供應商，以下是最常用的三個選擇：

1. **Anthropic Claude**：前往 [console.anthropic.com](https://console.anthropic.com/) 建立帳號，在 API Keys 頁面新增金鑰。Claude Sonnet 4.x 是目前最主流的選擇。
2. **OpenAI GPT**：前往 [platform.openai.com](https://platform.openai.com/)，建立 API Key。
3. **Google Gemini**：前往 [aistudio.google.com](https://aistudio.google.com/)，可免費取得 API Key。

把 API Key 複製好備用，等等安裝時會需要貼入。

---

## 第一步：安裝 OpenClaw

打開終端機，複製以下指令貼上並按 Enter：

**macOS / Linux：**
```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

**Windows（在 PowerShell 執行）：**
```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

安裝完成後，確認指令可以使用：
```bash
openclaw --version
```

---

## 第二步：執行 Onboarding 向導

執行以下指令，啟動互動式設定向導，同時讓 OpenClaw 以背景服務形式隨系統啟動：

```bash
openclaw onboard --install-daemon
```

向導會依序詢問幾個問題：
1. **選擇模型供應商**：選你剛才取得 API Key 的那家（Anthropic、OpenAI、Google 等）
2. **貼入 API Key**：把剛才複製的金鑰貼進去
3. **選擇模型**：建議選最新的主力模型（如 `claude-sonnet-4-5`）
4. **選擇安裝模式**：新手選 **QuickStart** 即可，會用預設值完成所有設定

整個過程約 2 分鐘，大部分選項按 Enter 接受預設值就好。

---

## 第三步：確認 Gateway 正在運行

```bash
openclaw gateway status
```
若看到 Gateway 監聽在 `port 18789`，代表安裝成功。

開啟瀏覽器控制介面試試：
```bash
openclaw dashboard
```
瀏覽器會打開 Control UI，你可以在這裡直接傳訊息給 AI 助理，確認一切運作正常。

---

## 第四步：串接 Telegram（最快上手的方式）

Telegram 是 OpenClaw 官方支援最完整的通訊管道，設定也最簡單，只需要一組 Bot Token。

### 4-1 向 BotFather 申請 Bot Token
1. 打開 Telegram，搜尋並開啟 **@BotFather**（確認旁邊有官方藍勾）。
2. 傳送 `/newbot`。
3. 依提示輸入 Bot 名稱（顯示名）和 Bot 帳號（需以 `bot` 結尾，如 `my_ai_assistant_bot`）。
4. BotFather 會給你一組 Token，格式類似 `123456789:AAGxxxxxxxxxxxxxxxxx`，把它複製下來。

### 4-2 設定 Token
接著要編輯 OpenClaw 的設定檔。檔案在你的家目錄下的 `.openclaw` 資料夾，路徑是 `~/.openclaw/config.json5`。

最簡單的開啟方式：
- **macOS / Linux**（用 TextEdit/nano 開啟）：
  ```bash
  open -e ~/.openclaw/config.json5
  ```
- **Windows**：
  ```powershell
  notepad $env:USERPROFILE\.openclaw\config.json5
  ```

在檔案中找到（或加入）以下 Telegram 設定：
```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "在這裡貼入你的 Token",
      dmPolicy: "pairing",
    },
  },
}
```

### 4-3 重啟 Gateway 並配對

```bash
openclaw gateway restart
openclaw pairing list telegram
```

接著打開 Telegram，找到你剛建立的 Bot（搜尋你設定的帳號名稱），傳任何一則訊息給它。這時 Bot 不會回應，但終端機那邊會出現一組**配對碼**。把那組碼複製下來，執行：

```bash
openclaw pairing approve telegram <配對碼>
```

完成後，在 Telegram 傳訊息給你的 Bot，就會收到 AI 助理的回覆了！🎉

---

## 常見問題 FAQ

### Node 版本錯誤怎麼辦？
執行 `node -v` 確認版本。若低於 `v22.14`，依照上方表格重新安裝 Node.js。macOS 用戶若有安裝 Homebrew，執行 `brew upgrade node` 即可升級。

### Gateway 無法啟動？
執行 `openclaw gateway status` 查看錯誤訊息。最常見的原因是 port `18789` 被佔用，或 API Key 格式錯誤。可以執行 `openclaw configure` 重新設定。

### 如何更換 API Key？
執行 `openclaw configure`，選擇 Model/Auth 步驟重新設定即可。

### Token 會一直燒嗎？
預設情況下，OpenClaw 只在收到訊息時才呼叫 API，不會無故消耗 Token。但若你啟用了某些自動化 Skill 或排程任務，建議在模型供應商的後台設定每月用量上限，避免意外超支。

---

## 接下來可以做什麼

- 串接 Gmail、Google Calendar，讓助理幫你管理信件與行程。
- 安裝 Skills，擴充助理的能力（網路搜尋、程式執行、記憶管理等）。
- 雲端部署：透過 Cloudflare Tunnel 讓龍蝦 24 小時在線（進階教學，即將推出）。
