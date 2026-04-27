# Claude Code — OpenAI Compatible API 設定

## OPENROUTER

### macOS / Linux（bash/zsh）

```bash
export OPENROUTER_API_KEY="sk-or-v1-"
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"
export ANTHROPIC_API_KEY=""  # 必須明確設為空字串

export ANTHROPIC_DEFAULT_OPUS_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
export ANTHROPIC_DEFAULT_SONNET_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
export CLAUDE_CODE_SUBAGENT_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
```

### Windows（PowerShell）

```powershell
$env:OPENROUTER_API_KEY = "sk-or-v1-"
$env:ANTHROPIC_BASE_URL = "https://openrouter.ai/api"
$env:ANTHROPIC_AUTH_TOKEN = $env:OPENROUTER_API_KEY
$env:ANTHROPIC_API_KEY = ""  # 必須明確設為空字串

$env:ANTHROPIC_DEFAULT_OPUS_MODEL  = "nvidia/nemotron-3-super-120b-a12b:free"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "nvidia/nemotron-3-super-120b-a12b:free"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL  = "nvidia/nemotron-3-super-120b-a12b:free"
$env:CLAUDE_CODE_SUBAGENT_MODEL     = "nvidia/nemotron-3-super-120b-a12b:free"
```

---

## OpenAI Compatible（自架端點，如 NCHC）

### macOS / Linux（bash/zsh）

```bash
export NCHC_API_KEY="sk-"
export ANTHROPIC_BASE_URL="https://inner-medusa.genai.nchc.org.tw"
export ANTHROPIC_AUTH_TOKEN="$NCHC_API_KEY"
export ANTHROPIC_API_KEY=""  # 必須明確設為空字串

export CLAUDE_CODE_DISABLE_THINKING=1
export LITELLM_DROP_PARAMS=true
export ANTHROPIC_DISABLE_THINKING=1

export ANTHROPIC_DEFAULT_OPUS_MODEL="Thanos3.5-397B-A17B"
export ANTHROPIC_DEFAULT_SONNET_MODEL="Thanos3.5-397B-A17B"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="Thanos3.5-397B-A17B"
export CLAUDE_CODE_SUBAGENT_MODEL="Thanos3.5-397B-A17B"
```

### Windows（PowerShell）

```powershell
$env:NCHC_API_KEY = "sk-"
$env:ANTHROPIC_BASE_URL = "https://inner-medusa.genai.nchc.org.tw"
$env:ANTHROPIC_AUTH_TOKEN = $env:NCHC_API_KEY
$env:ANTHROPIC_API_KEY = ""  # 必須明確設為空字串

$env:CLAUDE_CODE_DISABLE_THINKING = "1"
$env:LITELLM_DROP_PARAMS          = "true"
$env:ANTHROPIC_DISABLE_THINKING   = "1"

$env:ANTHROPIC_DEFAULT_OPUS_MODEL  = "Thanos3.5-397B-A17B"
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "Thanos3.5-397B-A17B"
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL  = "Thanos3.5-397B-A17B"
$env:CLAUDE_CODE_SUBAGENT_MODEL     = "Thanos3.5-397B-A17B"
```
