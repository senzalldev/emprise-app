# Prompt Engineering

emprise uses a layered prompt system that adapts to the model, user role, and per-profile configuration. This lets small local models stay focused while giving advanced models full freedom.

## The 6 Prompt Layers

The system prompt is assembled in order. Each layer adds to or overrides previous layers.

### Layer 0: User Identity
Set your name and role so the LLM tailors responses to your expertise.

```yaml
prompts:
  user_name: Steve
  role: developer
```

If `user_name` is set, the LLM addresses you by name occasionally. If `role` is set, responses are tailored to that workflow.

### Layer 1: Model Capability Tier
Automatically detected from the model name. Adds instructions appropriate for the model's ability:

- **Basic** (1-3B): "Keep responses SHORT. Only call ONE tool at a time."
- **Standard** (7-13B): No additional instructions (baseline).
- **Advanced** (70B+, GPT-4, Claude): "You can chain multiple tool calls, plan multi-step solutions."

### Layer 2: User Global Rules
Custom rules that apply to every conversation:

```yaml
prompts:
  rules:
    - Always use feature branches
    - Prefer Go for new services
    - Never commit directly to main
```

These appear as "USER RULES" in the system prompt.

### Layer 3: Model-Specific Overrides
Tune behavior for specific models:

```yaml
prompts:
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
```

Model matching uses case-insensitive contains, so `llama3` matches `llama3.2:3b`, `llama3:8b`, etc.

### Layer 4: Per-Profile Overrides
Different settings per LLM profile (useful when one profile is a small local model and another is GPT-4):

```yaml
prompts:
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
```

### Layer 5: Tool-Specific Emphasis
Add rules that apply only when a specific tool is being used:

```yaml
prompts:
  tool_emphasis:
    run_command:
      rules:
        - Always explain what the command does before running
    write_file:
      rules:
        - Include a comment header with date and purpose
```

### Layer 6: Emphasis Rules
The emphasis system is the main way emprise controls LLM behavior. Rules have priority levels that determine how strongly they are reinforced.

## Role Presets

Setting `role` in your config activates a preset emphasis profile:

### developer
`write_to_file: high`, `use_tools_first: high`, `no_secrets: high`, `check_before_change: medium`, `minimal_changes: medium`, `error_handling: medium`, `type_safety: medium`, `branch_workflow: medium`, `commit_messages: medium`

### devops
`write_to_file: high`, `use_tools_first: high`, `no_secrets: high`, `infra_as_code: high`, `env_awareness: high`, `docker_security: medium`, `idempotent: medium`, `backup_first: medium`, `dry_run: medium`

### cloud
`write_to_file: high`, `use_tools_first: high`, `no_secrets: high`, `check_resources: high`, `env_awareness: high`, `infra_as_code: medium`, `least_privilege: medium`, `backup_first: medium`

### data
`write_to_file: high`, `use_tools_first: high`, `no_secrets: high`, `validate_data: high`, `sql_safety: high`, `pipeline_idempotent: medium`, `schema_changes: medium`, `data_privacy: high`

### security
`write_to_file: high`, `use_tools_first: high`, `no_secrets: high`, `least_privilege: high`, `input_validation: high`, `no_hardcoded_creds: high`, `audit_trail: medium`, `encrypt_at_rest: medium`, `dependency_audit: medium`

### admin
`write_to_file: high`, `use_tools_first: high`, `no_secrets: high`, `backup_first: high`, `dry_run: high`, `log_commands: medium`, `prefer_read_only: medium`, `check_disk: medium`

## Emphasis Levels

| Level | Effect on Basic models | Effect on Standard | Effect on Advanced |
|-------|----------------------|-------------------|--------------------|
| **high** | Repeated 3 times in prompt | Repeated 2 times | Shown once |
| **medium** | Shown once | Shown once | Omitted |
| **off** | Removes the rule entirely | Removes the rule | Removes the rule |

This means small models get heavy repetition of critical rules, while advanced models get a clean prompt.

## All Available Emphasis Keys

### Code Practices
| Key | Instruction |
|-----|-------------|
| `write_to_file` | Always write code to files, never paste code blocks |
| `no_inline_code` | Never put code blocks in response |
| `use_tools_first` | Always use tools before answering |
| `be_concise` | Keep responses concise, use bullet points |
| `one_tool` | Call only one tool at a time |
| `ask_first` | Ask clarifying questions before acting |

### Developer Practices
| Key | Instruction |
|-----|-------------|
| `check_before_change` | Read a file before modifying it |
| `git_safety` | Never force push or reset --hard without asking |
| `test_after_change` | Suggest running tests after code changes |
| `explain_changes` | Briefly explain what changed and why |
| `no_secrets` | Never write API keys/tokens to files |
| `minimal_changes` | Make the smallest change that solves the problem |

### Admin/Ops Practices
| Key | Instruction |
|-----|-------------|
| `backup_first` | Suggest creating a backup before modifying configs |
| `dry_run` | Show what would happen before destructive operations |
| `check_disk` | Check disk space before writing large files |
| `log_commands` | Explain each shell command before running it |
| `prefer_read_only` | Default to read-only operations |

### DevOps/Cloud Practices
| Key | Instruction |
|-----|-------------|
| `infra_as_code` | Prefer IaC approaches (Terraform, Ansible) |
| `check_resources` | Check existing resources before provisioning |
| `use_namespaces` | Always specify Kubernetes namespace |
| `docker_security` | Use specific image tags, non-root containers |
| `env_awareness` | Confirm environment (dev/staging/prod) before changes |
| `idempotent` | Prefer idempotent operations |

### Developer Workflow
| Key | Instruction |
|-----|-------------|
| `branch_workflow` | Work on feature branches, never commit to main |
| `commit_messages` | Write clear commit messages explaining why |
| `code_review` | Suggest creating a PR for review |
| `error_handling` | Always include error handling in generated code |
| `type_safety` | Use strong typing, avoid any/interface{} |

### Data Engineering
| Key | Instruction |
|-----|-------------|
| `validate_data` | Validate schemas and types before processing |
| `sql_safety` | Use parameterized queries |
| `pipeline_idempotent` | Data pipelines must be safe to re-run |
| `schema_changes` | Schema migrations must be backwards-compatible |
| `data_privacy` | Mask PII in logs and output |

### Security
| Key | Instruction |
|-----|-------------|
| `least_privilege` | Request minimum permissions |
| `audit_trail` | Log security-relevant actions |
| `input_validation` | Validate and sanitize all external input |
| `encrypt_at_rest` | Suggest encryption for sensitive data |
| `dependency_audit` | Flag outdated or vulnerable dependencies |
| `no_hardcoded_creds` | Never hardcode credentials |

## Overriding Emphasis

Your config can override role defaults:

```yaml
prompts:
  role: developer
  emphasis:
    branch_workflow: off      # remove this rule
    docker_security: high     # add from another role
    custom_rule: medium       # custom key (shown literally)
```

Profile-level emphasis overrides global emphasis:

```yaml
prompts:
  profiles:
    local:
      emphasis:
        be_concise: high      # override for this profile
```

## The /prompts Command

In the TUI, use `/prompts` to open the prompt editor. This provides an interactive view of the current prompt configuration.

## The /system Command

Use `/system` to see a summary of the active system prompt:
- Identity (model, provider)
- Capability tier
- Role
- User name
- Number of active emphasis rules
- All active rules with their levels

Use `/system raw` to see the full system prompt text as sent to the LLM.

## Testing Prompts

The `/test` command sends a minimal test message through the current profile, which exercises the full prompt pipeline including all layers.
