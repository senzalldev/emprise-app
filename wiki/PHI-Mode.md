# PHI Compliance Mode

emprise includes a PHI (Protected Health Information) compliance mode for healthcare and regulated environments. When enabled, it restricts data flow, validates LLM endpoints, disables web tools, and logs all tool calls for audit.

## What PHI Mode Does

| Protection | Description |
|-----------|-------------|
| **Endpoint validation** | Verifies the LLM endpoint is in the secured endpoints list before sending data |
| **Web tools disabled** | `web_search` and `web_fetch` are blocked to prevent data leakage |
| **Audit logging** | Every tool call is logged with timestamp, tool name, arguments, user, and allow/deny status |
| **Secured endpoint list** | Default trusted patterns plus your custom patterns |

## How to Enable

Add to `~/.emprise/config.yaml`:

```yaml
phi_mode: true
secured_endpoints:
  - "*.openai.azure.com"
  - "*.cognitiveservices.azure.com"
  - my-internal-llm.corp.com
```

## Default Secured Endpoints

These patterns are always trusted (in addition to your `secured_endpoints`):

| Pattern | Matches |
|---------|---------|
| `*.openai.azure.com` | Azure OpenAI deployments |
| `*.cognitiveservices.azure.com` | Azure Cognitive Services |
| `*.googleapis.com` | Google Cloud APIs |
| `bedrock*.amazonaws.com` | AWS Bedrock |
| `sagemaker*.amazonaws.com` | AWS SageMaker |
| `localhost` | Local services |
| `127.0.0.1` | Loopback |
| `0.0.0.0` | Local binding |

## Endpoint Validation

When PHI mode is on, emprise validates each profile's endpoint before allowing use:

### Ollama (Local)
- Localhost/127.0.0.1: **always allowed** ("local Ollama, no data leaves machine")
- Remote Ollama: must be in `secured_endpoints`

### OpenAI
- Azure OpenAI (`*.azure` in URL): **allowed** ("Azure OpenAI")
- Direct OpenAI API: **blocked** unless added to `secured_endpoints`

### Anthropic
- **Blocked** unless added to `secured_endpoints`

### Any endpoint
- If `base_url` matches a pattern in `secured_endpoints` or defaults: **allowed**
- Otherwise: **blocked** with a message suggesting to add it

## Restricted Tools

When PHI mode is active, these tools are blocked:
- `web_search` — could send PHI to search engines
- `web_fetch` — could send PHI to arbitrary URLs

Attempting to use them returns: `"tool X is disabled in PHI mode (data protection)"`

## Audit Logging

Every tool call is logged in memory with:

```json
{
  "time": "2024-03-15T10:30:00Z",
  "tool": "sql_query",
  "args": "{\"db\":\"patient\",\"query\":\"SELECT...\"}",
  "user": "",
  "allowed": true,
  "reason": ""
}
```

API calls made through `api_call` are also logged.

View the audit log with the `/phi` command:
```
/phi
```

This shows:
- PHI mode status (on/off)
- Endpoint validation result (secured or warning)
- Active protections
- Audit log entry count
- OAuth credentials (if any loaded)

## OAuth for LLM Endpoints

PHI environments often use OAuth (client credentials flow) for LLM access. emprise supports this natively:

### Inline in config
```yaml
profiles:
  phi-safe:
    provider: openai
    base_url: https://my-instance.openai.azure.com/...
    model: gpt-4o
    auth:
      type: oauth
      client_id: ${AZURE_CLIENT_ID}
      client_secret: ${AZURE_CLIENT_SECRET}
      authority: https://login.microsoftonline.com/TENANT_ID
      scope: https://cognitiveservices.azure.com/.default
```

### Via secrets.json
Store credentials in `~/.emprise/secrets.json` (must be `chmod 600`):

```json
{
  "credentials": {
    "azure-llm": {
      "client_id": "app-id-here",
      "client_secret": "secret-here",
      "authority": "https://login.microsoftonline.com/TENANT_ID",
      "scope": "https://cognitiveservices.azure.com/.default"
    }
  }
}
```

Reference in config:
```yaml
profiles:
  phi-safe:
    provider: openai
    base_url: https://my-instance.openai.azure.com/...
    credential: azure-llm
    model: gpt-4o
```

Tokens are cached and refreshed automatically (5 minutes before expiry).

## API Calls in PHI Mode

The `api_call` tool (for calling authenticated REST APIs like FHIR) works in PHI mode and logs every call to the audit trail:

```yaml
apis:
  fhir:
    base_url: https://fhir.example.com/api
    credential: azure-fhir
    headers:
      Accept: application/fhir+json
```

## Recommended PHI Setup

```yaml
# Full PHI-compliant config example
default_profile: azure-phi

profiles:
  azure-phi:
    provider: openai
    base_url: https://my-instance.openai.azure.com/openai/deployments/gpt-4o
    model: gpt-4o
    auth:
      type: oauth
      client_id: ${AZURE_CLIENT_ID}
      client_secret: ${AZURE_CLIENT_SECRET}
      authority: https://login.microsoftonline.com/TENANT_ID
      scope: https://cognitiveservices.azure.com/.default

  local:
    provider: ollama
    base_url: http://localhost:11434
    model: llama3.2:3b

phi_mode: true
secured_endpoints:
  - "*.openai.azure.com"
  - "*.cognitiveservices.azure.com"
  - fhir.example.com

apis:
  fhir:
    base_url: https://fhir.example.com/api
    credential: azure-fhir
```

This setup ensures:
- LLM traffic goes only to Azure OpenAI (BAA-covered)
- Local Ollama is allowed (data stays on machine)
- Web tools are disabled
- All tool calls are audited
- FHIR API access uses OAuth
