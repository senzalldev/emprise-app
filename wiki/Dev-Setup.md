# Developer Environment Setup

The `/dev` command provides a comprehensive inventory of your development environment. It checks for 40+ tools across 12 categories with platform-aware install commands.

## Usage

```
/dev                    # opens the dev setup screen
/dev inventory          # show full inventory as text
/dev inventory Cloud    # filter to one section
```

## Categories and Tools

### Core
| Tool | Check | Install (macOS) | Install (Linux) |
|------|-------|-----------------|-----------------|
| **Git** | `git --version` + email config | `brew install git` | `sudo apt install -y git` |
| **SSH Key** | Checks `~/.ssh/` for ed25519/rsa/ecdsa | `ssh-keygen -t ed25519` | `ssh-keygen -t ed25519` |
| **Homebrew** | `brew --version` | Install script | Install script |

### Languages
| Tool | Install (macOS) | Install (Linux) |
|------|-----------------|-----------------|
| **nvm** | `curl` install script | `curl` install script |
| **Node.js** | `nvm install --lts` | `nvm install --lts` |
| **Go** | `brew install go` | `sudo apt install -y golang` |
| **Rust** | `rustup` install script | `rustup` install script |
| **Python** | `brew install python` | `sudo apt install -y python3` |
| **Java** | `brew install openjdk` | `sudo apt install -y default-jdk` |

### AI Tools
| Tool | Install (macOS/Linux) |
|------|----------------------|
| **Claude Code** | `npm install -g @anthropic-ai/claude-code` |
| **GitHub Copilot** | `gh extension install github/gh-copilot` |
| **OpenAI Codex** | `npm install -g @openai/codex` |
| **Ollama** | `brew install ollama` / `curl install script` |
| **OpenCode** | `go install github.com/opencode-ai/opencode@latest` |

### Editors
| Tool | Install (macOS) |
|------|-----------------|
| **VS Code** | `brew install --cask visual-studio-code` |
| **JetBrains Toolbox** | `brew install --cask jetbrains-toolbox` |
| **JetBrains IDEs** | Detects IDEA, GoLand, WebStorm, PyCharm, Rider, CLion |

### Cloud
| Tool | Install (macOS) | Install (Linux) |
|------|-----------------|-----------------|
| **AWS CLI** | `brew install awscli` | `sudo apt install -y awscli` |
| **Azure CLI** | `brew install azure-cli` | `curl` install script |
| **GCP CLI** | `brew install google-cloud-sdk` | `sudo snap install google-cloud-cli` |
| **Terraform** | `brew install terraform` | `sudo apt install -y terraform` |
| **kubectl** | `brew install kubectl` | `sudo snap install kubectl` |
| **Helm** | `brew install helm` | `sudo snap install helm` |

### Containers
| Tool | Install (macOS) | Install (Linux) |
|------|-----------------|-----------------|
| **Podman** | `brew install podman` | `sudo apt install -y podman` |
| **Docker** | `brew install --cask docker` | `sudo apt install -y docker.io` |

### Dev Tools
| Tool | Install (macOS) | Install (Linux) |
|------|-----------------|-----------------|
| **GitHub CLI** | `brew install gh` | `sudo apt install -y gh` |
| **Make** | `xcode-select --install` | `sudo apt install -y build-essential` |
| **jq** | `brew install jq` | `sudo apt install -y jq` |
| **curl** | `brew install curl` | `sudo apt install -y curl` |
| **ripgrep** | `brew install ripgrep` | `sudo apt install -y ripgrep` |
| **npm** | Comes with Node.js | Comes with Node.js |
| **pip** | `brew install python` | `sudo apt install -y python3-pip` |
| **ffmpeg** | `brew install ffmpeg` | `sudo apt install -y ffmpeg` |
| **Caddy** | `brew install caddy` | `sudo apt install -y caddy` |
| **.NET SDK** | `brew install dotnet-sdk` | `sudo apt install -y dotnet-sdk-8.0` |
| **Postman** | `brew install --cask postman` | `sudo snap install postman` |

### Security
| Tool | Install (macOS) |
|------|-----------------|
| **1Password** | `brew install --cask 1password 1password-cli` |
| **LastPass** | `brew install lastpass-cli` |

### Browsers
| Tool | Install (macOS) |
|------|-----------------|
| **Chrome** | `brew install --cask google-chrome` |
| **Edge** | `brew install --cask microsoft-edge` |

### Databases
| Tool | Install (macOS) | Install (Linux) |
|------|-----------------|-----------------|
| **PostgreSQL** | `brew install postgresql@16` | `sudo apt install -y postgresql` |
| **Redis** | `brew install redis` | `sudo apt install -y redis` |
| **SQLite** | `brew install sqlite` | `sudo apt install -y sqlite3` |

### Networking
| Tool | Install (macOS) | Install (Linux) |
|------|-----------------|-----------------|
| **ngrok** | `brew install ngrok` | `sudo snap install ngrok` |

### Communication
| Tool | Install (macOS) |
|------|-----------------|
| **Slack** | `brew install --cask slack` |
| **Teams** | `brew install --cask microsoft-teams` |

## How It Works

Each tool check runs a version command (e.g., `git --version`) and reports:
- Green checkmark if found, with version info
- Red X if missing, with install command

The inventory is platform-aware:
- **macOS**: Prefers `brew` and `brew install --cask`
- **Linux**: Uses `apt`, `snap`, or `curl` install scripts
- **Windows**: Uses `winget` where available

Some tools are platform-specific (e.g., GitHub Desktop on Windows only, PowerShell on Windows only).

## Recommended Tools

The following tools are marked as recommended (highlighted in the inventory):
Git, SSH Key, Homebrew, nvm, Node.js, Go, Python, Claude Code, Ollama, VS Code, Podman, GitHub CLI, Make, jq, curl, ripgrep, Nerd Font, Starship, zsh

## /dev inventory

The text-mode inventory (`/dev inventory`) is useful for piping or scripting. Filter to a section:

```
/dev inventory Languages
/dev inventory Cloud
/dev inventory "AI Tools"
/dev inventory Containers
```
