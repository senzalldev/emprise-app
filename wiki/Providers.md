# LLM Providers

emprise supports multiple LLM providers through a unified profile system. Each profile specifies a provider, endpoint, authentication, and model.

## Supported Providers

| Provider | Auth | Notes |
|----------|------|-------|
| **Ollama** | none | Local LLMs, no API key needed |
| **OpenAI** | api_key | GPT-4o, GPT-3.5, etc. |
| **Anthropic** | api_key | Claude Sonnet, Claude Opus, Claude Haiku |
| **GitHub Models** | token | Free with GitHub, uses Azure inference endpoint |
| **Custom** | any | Any OpenAI-compatible API (LM Studio, vLLM, etc.) |

## Setup Wizard

On first run, emprise walks you through provider setup:

```bash
emprise --setup
```

The wizard offers five choices:
1. **Ollama** — local, runs on your machine or a server
2. **GitHub Models** — free with GitHub Copilot
3. **OpenAI** — GPT-4o, needs API key
4. **Anthropic Claude** — needs API key
5. **Custom endpoint** — any OpenAI-compatible API

You can add a second profile during setup. After that, add more with `/profile` in the TUI or by editing `~/.emprise/config.yaml`.

## Ollama (Local)

Ollama runs LLMs locally with no API key required.

### Setup
1. Install Ollama: `brew install ollama` (macOS) or `curl -fsSL https://ollama.com/install.sh | sh` (Linux)
2. Pull a model: `ollama pull llama3.2:3b`
3. Start Ollama: `ollama serve`

### Config
```yaml
profiles:
  local:
    provider: ollama
    base_url: http://localhost:11434
    model: llama3.2:3b
```

### Remote Ollama
Point `base_url` to a remote server:
```yaml
profiles:
  server:
    provider: ollama
    base_url: http://192.168.1.100:11434
    model: qwen2.5:14b
```

### Recommended Models
- **llama3.2:3b** — fast, good for basic tasks (Basic tier)
- **llama3:8b** — balanced speed and capability (Standard tier)
- **qwen2.5:14b** — strong reasoning (Standard tier)
- **llama3.1:70b** — near-GPT-4 quality (Advanced tier)

## OpenAI

### Setup
1. Get an API key from [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Set it as an environment variable: `export OPENAI_API_KEY=sk-...`

### Config
```yaml
profiles:
  openai:
    provider: openai
    base_url: https://api.openai.com/v1
    api_key: ${OPENAI_API_KEY}
    model: gpt-4o
```

### Models
- **gpt-4o** — best overall (Advanced tier)
- **gpt-4o-mini** — fast and cheap (Standard tier)
- **o1-preview** — reasoning model

## Anthropic

### Setup
1. Get an API key from [console.anthropic.com](https://console.anthropic.com/settings/keys)
2. Set it: `export ANTHROPIC_API_KEY=sk-ant-...`

### Config
```yaml
profiles:
  claude:
    provider: anthropic
    api_key: ${ANTHROPIC_API_KEY}
    model: claude-sonnet-4-20250514
```

### Models
- **claude-sonnet-4-20250514** — best balance (Advanced tier)
- **claude-opus-4-20250514** — most capable (Advanced tier)
- **claude-haiku** — fastest (Standard tier)

## GitHub Models

Free with a GitHub account. Uses Azure inference endpoint.

### Setup
1. Generate a token at [github.com/settings/tokens](https://github.com/settings/tokens) (classic, no scopes needed)
2. Set it: `export GITHUB_TOKEN=ghp_...`

### Config
```yaml
profiles:
  github:
    provider: openai
    base_url: https://models.inference.ai.azure.com
    token: ${GITHUB_TOKEN}
    model: gpt-4o
```

During setup, emprise fetches the available model list from the endpoint when a valid token is provided.

## Custom / OAuth Endpoints

Any OpenAI-compatible API works with the `openai` provider.

### LM Studio / vLLM
```yaml
profiles:
  custom:
    provider: openai
    base_url: http://localhost:8080/v1
    model: default
```

### Azure OpenAI with OAuth
```yaml
profiles:
  azure:
    provider: openai
    base_url: https://my-instance.openai.azure.com/openai/deployments/gpt-4o
    model: gpt-4o
    auth:
      type: oauth
      client_id: ${AZURE_CLIENT_ID}
      client_secret: ${AZURE_CLIENT_SECRET}
      authority: https://login.microsoftonline.com/TENANT_ID
      scope: https://cognitiveservices.azure.com/.default
```

OAuth tokens are automatically cached and refreshed before expiry. You can also store credentials in `~/.emprise/secrets.json`:

```json
{
  "credentials": {
    "azure-llm": {
      "client_id": "...",
      "client_secret": "...",
      "authority": "https://login.microsoftonline.com/TENANT_ID",
      "scope": "https://cognitiveservices.azure.com/.default"
    }
  }
}
```

Then reference by name:
```yaml
profiles:
  azure:
    provider: openai
    base_url: https://my-instance.openai.azure.com/...
    credential: azure-llm
    model: gpt-4o
```

The secrets file must have `chmod 600` permissions.

## Test Connection

In the TUI, use the `/test` command to verify a profile works:

```
/test           # test current profile
/test github    # test a specific profile
```

The test sends a minimal request ("Say hello"), reports auth status, model, latency, and token usage.

In the profile editor (accessible via `/profile`), each profile has a **Test Connection** button that:
1. Resolves authentication (OAuth token acquisition, API key validation, etc.)
2. Creates a client and sends a test message
3. Reports success/failure, model, token count, and latency

## Model Capability Tiers

emprise automatically detects model capability and adjusts behavior:

| Tier | Models | Behavior |
|------|--------|----------|
| **Basic** | 1-3B models (llama3.2:3b, etc.) | One tool at a time, short responses, emphasis repeated 3x |
| **Standard** | 7-13B models (llama3, qwen2.5:7b, mistral, gpt-3.5) | Normal behavior, emphasis repeated 2x |
| **Advanced** | 70B+, GPT-4, Claude Sonnet/Opus | Multi-tool chaining, detailed responses, emphasis shown once |

## Multiple Profiles

You can define as many profiles as you want and switch between them:

```bash
emprise -p claude "explain this code"    # use a specific profile
```

In the TUI:
```
/profile use claude    # switch to claude profile
/profile               # list all profiles
```

## Advanced Profile Settings

Each profile supports fine-tuning via the TUI profile editor or config:

| Setting | Description |
|---------|-------------|
| `max_tokens` | Cap on response length |
| `temperature` | 0-2, lower = more deterministic |
| `top_p` | 0-1, nucleus sampling threshold |
| `context_size` | Override auto-detected context window |
| `timeout` | Request timeout in seconds (default 120) |
| `max_tool_rounds` | Max tool call iterations (default 10) |
| `stream` | Enable/disable streaming (default true) |
