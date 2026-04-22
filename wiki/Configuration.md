# Configuration Reference

emprise stores its configuration in `~/.emprise/config.yaml`. This page documents every field.

## Quick Start

On first run, emprise launches an interactive setup wizard that creates the config file for you. You can re-run it anytime:

```bash
emprise --setup
```

## Full Config Structure

```yaml
default_profile: local

profiles:
  local:
    provider: ollama
    base_url: http://localhost:11434
    model: llama3.2:3b

  github:
    provider: openai
    base_url: https://models.inference.ai.azure.com
    token: ${GITHUB_TOKEN}
    model: gpt-4o

  claude:
    provider: anthropic
    api_key: ${ANTHROPIC_API_KEY}
    model: claude-sonnet-4-20250514

  azure-gpt4:
    provider: openai
    base_url: https://my-instance.openai.azure.com/openai/deployments/gpt-4o
    auth:
      type: oauth
      client_id: ${AZURE_CLIENT_ID}
      client_secret: ${AZURE_CLIENT_SECRET}
      authority: https://login.microsoftonline.com/TENANT_ID
      scope: https://cognitiveservices.azure.com/.default

tools:
  shell:
    confirm: true
  ssh:
    hosts:
      enigma: steve@192.168.50.2
      web: deploy@web.example.com
  mcp:
    - name: my-server
      url: http://localhost:3000
      token: secret
  databases:
    local:
      driver: sqlite
      path: ~/data/app.db
    prod:
      driver: postgres
      dsn: postgres://user:pass@db.example.com:5432/mydb
  plugins_dir: ~/.emprise/plugins
  paste_threshold: 3

history:
  db: ~/.emprise/history.db
  max_conversations: 100

team:
  config_url: https://example.com/team-config.yaml
  name: My Team
  require_sso: false
  sso_provider: https://login.microsoftonline.com/TENANT_ID

prompts:
  user_name: Steve
  role: developer
  rules:
    - Always use feature branches
    - Prefer Go for new services
  emphasis:
    write_to_file: high
    branch_workflow: high
    test_after_change: medium
  tool_emphasis:
    run_command:
      rules:
        - Always explain what the command does before running
  model_overrides:
    llama3.2:3b:
      max_tools_per_turn: 1
      max_response_len: 200
      extra_rules:
        - Keep it very short
      disable_tools:
        - web_search
    gpt-4o:
      max_tools_per_turn: 5
  profiles:
    local:
      context_size: 4096
      max_tools_per_turn: 1
      emphasis:
        be_concise: high
      extra_rules:
        - Summarize long outputs
    github:
      context_size: 128000
      max_tools_per_turn: 5

phi_mode: false
secured_endpoints:
  - "*.openai.azure.com"
  - "*.cognitiveservices.azure.com"
  - my-internal-llm.corp.com

apis:
  fhir:
    base_url: https://fhir.example.com/api
    credential: azure-fhir
    headers:
      Accept: application/fhir+json
  internal:
    base_url: https://api.internal.corp.com
    credential: corp-oauth

api_tools:
  patient-lookup:
    name: patient_lookup
    description: Look up patient records
    base_url: https://fhir.example.com/api
    auth:
      type: oauth
      client_id: ${FHIR_CLIENT_ID}
      client_secret: ${FHIR_SECRET}
      authority: https://login.microsoftonline.com/TENANT
      scope: https://fhir.example.com/.default
```

## Profile Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `provider` | string | — | **Required.** `ollama`, `openai`, `anthropic`, or `gemini` |
| `base_url` | string | varies | API endpoint URL |
| `api_key` | string | — | API key (or `${ENV_VAR}`) |
| `token` | string | — | Bearer token (or `${ENV_VAR}`) |
| `model` | string | — | **Required.** Model name |
| `auth` | object | — | Auth configuration (see below) |
| `enabled` | bool | true | Enable/disable this profile |
| `stream` | bool | true | Enable streaming responses |
| `max_tokens` | int | default | Max response tokens |
| `temperature` | float | default | 0.0-2.0, lower = more precise |
| `top_p` | float | default | 0.0-1.0, nucleus sampling |
| `context_size` | int | auto | Context window size in tokens |
| `timeout` | int | 120 | Request timeout in seconds |
| `max_tool_rounds` | int | 10 | Max tool call iterations per turn |

## Auth Types

### `none` (default for Ollama)
No authentication. Used for local Ollama instances.

### `api_key`
Standard API key authentication. The key is sent as a header.

```yaml
auth:
  type: api_key
  api_key: ${OPENAI_API_KEY}
```

Or use the shorthand:
```yaml
api_key: ${OPENAI_API_KEY}
```

### `token`
Bearer token authentication (e.g., GitHub personal access tokens).

```yaml
auth:
  type: token
token: ${GITHUB_TOKEN}
```

### `oauth`
OAuth2 client credentials flow. Used for Azure OpenAI, enterprise endpoints, etc.

```yaml
auth:
  type: oauth
  client_id: ${CLIENT_ID}
  client_secret: ${CLIENT_SECRET}
  authority: https://login.microsoftonline.com/TENANT_ID
  scope: https://cognitiveservices.azure.com/.default
```

Tokens are automatically cached and refreshed 5 minutes before expiry.

## Environment Variables

Any string value can reference an environment variable using `${VAR_NAME}` syntax:

```yaml
api_key: ${OPENAI_API_KEY}
token: ${GITHUB_TOKEN}
```

This is expanded at load time. If the variable is not set, the literal string is kept.

## Database Configuration

```yaml
tools:
  databases:
    mydb:
      driver: sqlite|postgres|mysql|sqlserver
      path: /path/to/file.db    # sqlite only
      dsn: connection_string     # postgres/mysql/sqlserver
```

Supported drivers: `sqlite`, `postgres`, `mysql`, `sqlserver` (also `mssql`).

## Config Validation

emprise validates your config at load time and prints warnings for:
- `default_profile` referencing a non-existent profile
- Profiles missing `provider` or `model`
- OAuth profiles missing `client_id`, `client_secret`, `authority`, or `scope`
- API key profiles with no key configured
- Invalid `base_url` format
- `temperature` outside 0-2 range
- `top_p` outside 0-1 range
- Databases missing `driver` or both `path`/`dsn`

## File Locations

| File | Purpose |
|------|---------|
| `~/.emprise/config.yaml` | Main configuration |
| `~/.emprise/secrets.json` | OAuth credentials (must be `chmod 600`) |
| `~/.emprise/history.db` | Conversation history (SQLite) |
| `~/.emprise/plugins/` | Plugin scripts |
