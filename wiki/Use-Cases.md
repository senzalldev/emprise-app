# Use Cases

Real-world examples organized by role. All examples work as single-shot commands or in the interactive TUI.

## Developer

### Code Exploration
```bash
emprise "what does the handleAuth function do in this project?"
emprise "find all TODO comments in the codebase"
emprise "explain the architecture of this Go project"
emprise "list all exported functions in internal/tools/"
```

### Code Generation
```bash
emprise "create a REST API handler for user CRUD in Go"
emprise "write unit tests for the config parser"
emprise "add error handling to main.go"
emprise "create a Dockerfile for this Go project"
```

### Code Review
```bash
emprise "review the latest PR diff"
emprise "check for security issues in the auth code"
emprise "are there any unused imports in this project?"
```

### Git Workflow
```bash
emprise "what changed since last commit?"
emprise "create a feature branch for the new login page"
emprise "write a commit message for the staged changes"
emprise "show me the diff for PR 42"
```

### Dependency Management
```bash
emprise "check for outdated Go dependencies"
emprise "are there any npm vulnerabilities?"
emprise "update the go.mod to latest patch versions"
```

### Debugging
```bash
emprise "why is this test failing?" < test_output.txt
emprise "explain this stack trace" < error.log
emprise "find where the nil pointer dereference could happen in server.go"
```

## DevOps

### Docker / Containers
```bash
emprise "list running containers and their resource usage"
emprise "create a multi-stage Dockerfile for this Node.js app"
emprise "why is my container using so much memory?"
emprise "write a docker-compose.yml for postgres + redis + the app"
```

### CI/CD
```bash
emprise "create a GitHub Actions workflow for Go tests"
emprise "add a deploy step to the CI pipeline"
emprise "why did the last CI run fail?"
```

### Infrastructure
```bash
emprise "write a Terraform module for an S3 bucket with versioning"
emprise "create an Ansible playbook to install nginx"
emprise "generate a Kubernetes deployment for this app"
```

### Monitoring
```bash
emprise "check disk usage on all mounts"
emprise "show the top 10 processes by memory"
emprise "are there any zombie processes?"
emprise "check if port 8080 is in use"
```

## Cloud Engineer

### AWS
```bash
emprise "list all EC2 instances in us-east-1"
emprise "create a CloudFormation template for a VPC"
emprise "check my AWS IAM permissions"
emprise "estimate the cost of running 3 t3.large instances"
```

### Azure
```bash
emprise "list all resource groups"
emprise "create an ARM template for a web app"
emprise "check Azure AD app registrations"
```

### GCP
```bash
emprise "list GKE clusters"
emprise "create a Cloud Run service definition"
```

### Kubernetes
```bash
emprise "show pods in the production namespace"
emprise "why is this pod in CrashLoopBackOff?"
emprise "scale the web deployment to 5 replicas"
emprise "create a Helm chart for this microservice"
```

## Data Engineer

### SQL Analysis
```bash
emprise "what tables are in the analytics database?"
emprise "describe the users table schema"
emprise "show the top 10 customers by order count"
emprise "find duplicate email addresses in the users table"
```

### CSV Analysis
```bash
emprise "analyze this sales CSV — what are the trends by region?"
emprise "import customers.csv and find the most common states"
emprise "how many rows have missing email addresses?"
```

### Data Pipeline
```bash
emprise "write a Python ETL script to transform this CSV"
emprise "create a SQL migration to add an index on users.email"
emprise "validate the schema of this JSON file against the API spec"
```

### Document Processing
```bash
emprise "summarize this PDF report"
emprise "extract the data from this Excel spreadsheet"
emprise "what's in this Jupyter notebook?"
emprise "parse this PPTX and list the key points"
```

## Security

### Code Security
```bash
emprise "check this code for SQL injection vulnerabilities"
emprise "are there any hardcoded secrets in this repo?"
emprise "review the auth middleware for common vulnerabilities"
emprise "check file permissions on the config directory"
```

### System Security
```bash
emprise "check for open ports on this machine"
emprise "list SSH authorized keys"
emprise "are there any world-writable files in /etc?"
emprise "check TLS certificate expiry for example.com"
```

### Compliance
```bash
emprise "is this code HIPAA-compliant for handling patient data?"
emprise "check if our API follows OWASP top 10 guidelines"
emprise "audit the npm dependencies for known vulnerabilities"
```

## Admin

### System Administration
```bash
emprise "check system uptime and load average"
emprise "show disk usage by directory"
emprise "list all cron jobs"
emprise "find large files over 100MB"
emprise "check memory usage breakdown"
```

### User Management
```bash
emprise "list all system users with shell access"
emprise "create a new user with SSH key access"
```

### Backup and Recovery
```bash
emprise "create a backup script for the postgres database"
emprise "write a cron job to clean logs older than 30 days"
```

### Network
```bash
emprise "check DNS resolution for example.com"
emprise "trace the route to 8.8.8.8"
emprise "list all listening services"
```

## Pipe Mode

Pipe data into emprise for analysis:

```bash
# Explain error logs
tail -100 /var/log/app.log | emprise "what errors are happening?"

# Review code
cat main.go | emprise "review this code for issues"

# Analyze command output
kubectl get pods -A | emprise "are any pods unhealthy?"

# Process data
curl -s api.example.com/data | emprise "summarize this JSON response"

# Git analysis
git log --oneline -20 | emprise "summarize recent changes"
```

## Named Conversations

Keep context across multiple interactions:

```bash
# Start a named conversation
emprise --id "server migration" "we need to migrate the database to a new server"

# Continue later with more context
emprise --id "server migration" "the new server is at 10.0.1.50, postgres 16"

# Resume interactively
emprise --resume "server migration"

# Continue the most recent conversation
emprise -c
```

## Channel Mode

Run emprise as a service:

```bash
# HTTP API server on port 8090
emprise --channel http:8090

# Stdin mode (for piping from other tools)
emprise --channel stdin
```
