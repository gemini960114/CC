# claude openai compatiable api

## OPENROUTER 
```bash
export OPENROUTER_API_KEY="sk-or-v1-"
export ANTHROPIC_BASE_URL="https://openrouter.ai/api"
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"
export ANTHROPIC_API_KEY="" # Important: Must be explicitly empty

export ANTHROPIC_DEFAULT_OPUS_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
export ANTHROPIC_DEFAULT_SONNET_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
export CLAUDE_CODE_SUBAGENT_MODEL="nvidia/nemotron-3-super-120b-a12b:free"
```


## OPENAI Comaptiable
```bash
export OPENROUTER_API_KEY="sk-"
export ANTHROPIC_BASE_URL="https://inner-medusa.genai.nchc.org.tw"
export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"
export ANTHROPIC_API_KEY=""

export CLAUDE_CODE_DISABLE_THINKING=1
export LITELLM_DROP_PARAMS=true
export ANTHROPIC_DISABLE_THINKING=1

export ANTHROPIC_DEFAULT_OPUS_MODEL="Thanos3.5-397B-A17B"
export ANTHROPIC_DEFAULT_SONNET_MODEL="Thanos3.5-397B-A17B"
export ANTHROPIC_DEFAULT_HAIKU_MODEL="Thanos3.5-397B-A17B"
export CLAUDE_CODE_SUBAGENT_MODEL="Thanos3.5-397B-A17B"
```

