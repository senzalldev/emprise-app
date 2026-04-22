# Frequently Asked Questions

## Installation

### How do I install emprise?
```bash
curl -fsSL https://raw.githubusercontent.com/senzalldev/emprise-app/main/install.sh | sh
```

Or download the binary for your platform from the [releases page](https://github.com/senzalldev/emprise-app/releases).

### How do I update?
```bash
emprise update
```

This checks for a newer version and downloads it automatically. Use `emprise install` to force reinstall the current version.

### Where is the config file?
`~/.emprise/config.yaml`. Edit it directly or use the interactive TUI:
- `/config` — opens the config in your editor
- `/profile` — manage LLM profiles
- `/setup` — re-run the setup wizard from the TUI

You can also re-run setup from the command line: `emprise --setup`

### What platforms are supported?
macOS (Intel and Apple Silicon), Linux (amd64 and arm64), and Windows. The macOS distribution includes a signed and notarized DMG.

### Do I need an API key?
Not if you use Ollama (local). For cloud providers:
- **GitHub Models**: Free with a GitHub token (no scopes needed)
- **OpenAI**: Requires a paid API key
- **Anthropic**: Requires a paid API key

## Providers

### Which provider should I use?
- **Starting out / free**: Ollama with `llama3.2:3b` (local, no cost)
- **Free cloud**: GitHub Models with `gpt-4o` (free with GitHub)
- **Best quality**: OpenAI `gpt-4o` or Anthropic `claude-sonnet-4-20250514`
- **PHI/enterprise**: Azure OpenAI with OAuth

### Can I use multiple providers?
Yes. Define multiple profiles in your config and switch between them:
```bash
emprise -p claude "explain this code"    # command line
```
In the TUI: `/profile use claude`

### How do I add a local model?
1. Install Ollama: `brew install ollama`
2. Pull a model: `ollama pull llama3.2:3b`
3. Start the server: `ollama serve`
4. emprise will auto-detect it on `http://localhost:11434`

### The model is slow / running out of context
Adjust per-profile settings:
```yaml
prompts:
  profiles:
    local:
      context_size: 4096
      max_tools_per_turn: 1
      emphasis:
        be_concise: high
```

### How do I use Azure OpenAI?
See [Providers](Providers#custom--oauth-endpoints). Use OAuth auth type with your Azure AD client credentials.

## Safety

### What commands are blocked?
emprise has a hardcoded blocklist of dangerous commands that can never run:
- Disk destruction: `mkfs`, `fdisk`, `dd if=`, `dd of=/dev`
- Recursive system delete: `rm -rf /`, `rm -rf /*`, `rm -rf ~`
- Shutdown/reboot: `shutdown`, `reboot`, `halt`, `poweroff`
- Fork bombs, credential theft patterns
- Platform-specific: `diskutil eraseDisk` (macOS), `format c:` (Windows), `wipefs` (Linux)

### What commands show warnings?
These are allowed but show a confirmation prompt:
- `rm -rf`, `rm -r`, `sudo rm`
- `DROP TABLE`, `DROP DATABASE`, `DELETE FROM`
- `git push --force`, `git reset --hard`
- `docker rm`, `docker system prune`
- `kill -9`, `sudo`

### What file paths are protected?
Cannot write to or delete from:
- `/etc/`, `/usr/`, `/bin/`, `/sbin/`, `/boot/`, `/dev/`, `/proc/`, `/sys/`
- `/var/log/`, `/var/run/`
- `C:\Windows\`, `C:\Program Files\`
- `/System/`, `/Library/` (macOS)

Writing to `.ssh`, `.gnupg`, `.aws`, `.kube` directories triggers a warning.

### Can the LLM run commands without asking?
By default, "dangerous" tools (write_file, git_commit, run_command for risky commands) require confirmation. Options:
- Default: confirm dangerous operations
- `--yes` / `-y`: auto-approve safe operations
- `--yolo`: skip ALL confirmations (use with caution)

## PHI Mode

### What is PHI mode?
PHI (Protected Health Information) compliance mode for healthcare environments. It validates LLM endpoints against a trusted list, disables web tools, and audits all tool calls. See [PHI Mode](PHI-Mode).

### Which endpoints work with PHI mode?
Any endpoint matching the secured patterns: `*.openai.azure.com`, `*.cognitiveservices.azure.com`, `*.googleapis.com`, `bedrock*.amazonaws.com`, `sagemaker*.amazonaws.com`, localhost, plus your custom patterns.

### Can I use OpenAI/Anthropic with PHI mode?
Not by default — their endpoints are not in the trusted list. You can add them to `secured_endpoints` if your organization has a BAA with these providers.

### Is audit data persistent?
The audit log is in-memory for the current session. For persistent audit logging, use the `metrics` system or export conversation history.

## OAuth

### How does OAuth work?
emprise uses the OAuth2 client credentials flow (machine-to-machine). You provide `client_id`, `client_secret`, `authority` (Azure AD tenant URL), and `scope`. Tokens are cached and refreshed automatically.

### Where do I store OAuth secrets?
Two options:
1. **In config** using environment variables: `client_id: ${AZURE_CLIENT_ID}`
2. **In secrets.json**: `~/.emprise/secrets.json` (must be `chmod 600`)

### Token caching?
Tokens are cached in memory and refreshed 5 minutes before expiry. No token is written to disk.

## Troubleshooting

### "unknown profile" error
Your `default_profile` references a profile that does not exist in `profiles`. Check `~/.emprise/config.yaml` for typos.

### OAuth token acquisition fails
- Verify `client_id`, `client_secret`, `authority`, and `scope` are correct
- Check that `authority` is the Azure AD tenant URL (e.g., `https://login.microsoftonline.com/TENANT_ID`)
- Make sure the app registration has the correct API permissions
- Use `/test` in the TUI to see detailed error messages

### Ollama connection refused
- Make sure Ollama is running: `ollama serve`
- Check the URL in your config: default is `http://localhost:11434`
- For remote Ollama, verify the server allows external connections

### Config validation warnings
emprise validates config on load. Warnings include missing fields, invalid URLs, out-of-range values. Fix them in `~/.emprise/config.yaml` or re-run `/setup`.

### Tool calls not working
- Check `/tools` to see available tools and their enabled/disabled status
- Toggle tools with `/tools toggle <name>`
- Some tools require additional setup (databases need config, SSH needs host entries)

### Large file reading is slow
Use the `limit` parameter: the LLM calls `read_file(path: "big.log", limit: 50)` to read only the first 50 lines. For documents (PDF, DOCX, etc.), all parsers respect the limit.

## Comparison with Claude Code

| Feature | emprise | Claude Code |
|---------|---------|-------------|
| **Provider** | Any (Ollama, OpenAI, Anthropic, custom) | Anthropic only |
| **Local LLMs** | Yes (Ollama) | No |
| **Cost** | Free (local) or BYOK | Requires Anthropic API key |
| **PHI mode** | Yes | No |
| **SQL tools** | Built-in (SQLite, Postgres, MySQL, SQL Server) | No |
| **Document parsing** | PDF, DOCX, XLSX, PPTX, Jupyter | Limited |
| **OAuth auth** | Azure AD client credentials | No |
| **Dev inventory** | 40+ tools with install commands | No |
| **Prompt engineering** | 6-layer system with roles and emphasis | System prompt only |
| **Named conversations** | Yes (--id, --resume) | Yes |
| **Channel mode** | HTTP API, stdin | No |
| **Platform** | macOS, Linux, Windows | macOS, Linux |
| **Written in** | Go | TypeScript |

emprise is designed for teams that need multi-provider support, PHI compliance, or prefer local LLMs. Claude Code is optimized for the Anthropic ecosystem with deep IDE integration.
