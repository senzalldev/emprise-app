# emprise

> *A bold undertaking* — AI-powered CLI assistant with 43+ tools, interactive menus, and prompt engineering.

[![Download](https://img.shields.io/github/v/release/senzalldev/emprise-app?label=download)](https://github.com/senzalldev/emprise-app/releases/latest)
[![Website](https://img.shields.io/badge/website-emprise.dev-58d1c9)](https://emprise.dev)

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/senzalldev/emprise-app/main/install.sh | sh
```

Or download from [Releases](https://github.com/senzalldev/emprise-app/releases/latest): macOS DMG (signed + notarized), Linux binary, Windows EXE.

## Features

- **43+ built-in tools** — system info, files, git, shell, web, Docker, SSH, SQL, JSON, document parsing
- **4 LLM providers** — Ollama (local), OpenAI, Anthropic, GitHub Models + OAuth endpoints
- **Interactive menus** — arrow-key navigation for profiles, tools, models, history, prompts, config
- **Prompt engineering** — 6 role presets, 40+ emphasis rules, per-model overrides, test prompt
- **Developer environment** — `/dev` audits 40+ tools, one-click install across 12 categories
- **Document parsing** — PDF, DOCX, XLSX, PPTX, ZIP, Jupyter, CSV, JSON, XML
- **SQL tools** — SQLite, PostgreSQL, MySQL, SQL Server (Azure AD auth)
- **PHI compliance** — disable web tools, validate endpoints, audit logging, OAuth
- **Diff view** — line numbers, syntax highlighting, +/- coloring
- **Self-update** — `emprise update` with auto-restart

## Quick Start

```bash
emprise                          # Interactive chat
emprise "what is my disk usage?" # Single-shot
emprise -c                       # Continue last conversation
cat file.log | emprise "explain" # Pipe mode
```

## Commands

Type `/help` in emprise for the full list. Highlights:

| Command | Description |
|---------|-------------|
| `/profile` | Switch LLM profile |
| `/model` | Switch model |
| `/tools` | Toggle tools by category |
| `/dev` | Developer environment audit |
| `/prompts` | Prompt engineering config |
| `/stats` | Usage heat map |
| `/config` | Settings |
| `/setup` | Add LLM provider (wizard) |

## Documentation

- [Website](https://emprise.dev) — features, commands, FAQ
- [Wiki](https://github.com/senzalldev/emprise-app/wiki) — installation, configuration, guides
- [Discussions](https://github.com/senzalldev/emprise-app/discussions) — community, Q&A
- [Issues](https://github.com/senzalldev/emprise-app/issues) — bug reports, feature requests

## Configuration

```yaml
# ~/.emprise/config.yaml
default_profile: local
profiles:
  local:
    provider: ollama
    base_url: http://localhost:11434
    model: llama3.2:3b
    auth:
      type: none
prompts:
  user_name: Your Name
  role: developer
```

See the [Configuration Guide](https://github.com/senzalldev/emprise-app/wiki/Configuration) for OAuth, PHI mode, SQL databases, and API tools.

## Update

```bash
emprise update
```

## License

Proprietary. Built by Steve Sparks.

