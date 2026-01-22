# Getting Started

## Installation (5 minutes)

### 1. Clone Repository
```bash
git clone https://github.com/eddiebe147/claude-automation-pipeline.git
cd claude-automation-pipeline
```

### 2. Copy Files to Development
```bash
# Scripts
mkdir -p ~/Development/scripts
cp *.sh ~/Development/scripts/
cp dev-execute ~/Development/scripts/

# Config
mkdir -p ~/Development/config
cp *.json ~/Development/config/
cp -r config/launchd/* ~/Development/config/

# Skills
mkdir -p ~/Development/id8/skills-registry/skills/stackshack
cp -r skills/* ~/Development/id8/skills-registry/skills/stackshack/

# Documentation
cp *.md ~/Development/
```

### 3. Set API Key
```bash
# Get your key from https://console.anthropic.com
export CLAUDE_API_KEY="sk-..."

# Make permanent (add to ~/.zshrc or ~/.bash_profile)
echo 'export CLAUDE_API_KEY="sk-..."' >> ~/.zshrc
source ~/.zshrc
```

### 4. Enable Automation
```bash
bash ~/Development/scripts/setup-automation.sh --enable
```

### 5. Verify
```bash
bash ~/Development/scripts/setup-automation.sh --status
# Should show all 3 agents LOADED
```

## First Run (Manual)

Test before automation kicks in tomorrow:

```bash
# Run collection
~/Development/scripts/collect-product-health.sh

# Run analysis
~/Development/scripts/analyze-report.sh

# Generate report
~/Development/scripts/generate-morning-report.sh
```

## Daily Workflow

Tomorrow at **7:00 AM**:
1. Morning report opens in your editor
2. Read priorities and recommendations
3. Execute tasks: `dev-execute <task-id>`
4. System logs outcomes and learns

## Troubleshooting

### Reports not generating?
```bash
tail -f ~/Development/reports/launchd-*.log
```

### API key not working?
```bash
# Test directly
export CLAUDE_API_KEY="sk-..."
~/Development/scripts/analyze-report.sh
```

### Disable temporarily?
```bash
bash ~/Development/scripts/setup-automation.sh --disable
```

## Configuration

Edit per-project quality requirements:
```bash
nano ~/Development/config/quality-gates.json
```

Adjust automation settings:
```bash
nano ~/Development/config/automation.json
```

---

Questions? Check `AUTOMATION_PIPELINE.md` for detailed documentation.
